# Technical Architecture — Custom Jewellery Pricing

## Overview

Custom Jewellery Pricing is a WordPress plugin that extends WooCommerce with a configurable pricing engine for jewellery products and their variations. It combines WordPress administration screens, WooCommerce product data, business-specific calculation rules, AJAX operations, and diagnostic tools.

The architecture below is a **conceptual map of the components confirmed in the private repository**, not a complete runtime or deployment diagram.

![Custom Jewellery Pricing — technical architecture](../assets/architecture.svg)

## 1. Technology and integration points

| Area | Implementation |
|---|---|
| Runtime | PHP plugin loaded by WordPress |
| Commerce platform | WooCommerce products, variable products, variations and metadata |
| Administration | WordPress admin menu, Settings API and plugin-specific pages |
| Browser interactions | JavaScript and authenticated WordPress AJAX actions |
| Persistence | WordPress options, post metadata and custom error-log table through `$wpdb` |
| Scheduled operations | WordPress hooks / scheduled events for background weight updates |
| Diagnostics | Error logging, import/export, profiler and maintenance screens |
| Testing | PHPUnit development dependency; test coverage not assessed |

The project uses a functional PHP module organization rather than a Symfony-style service/container architecture. Its plugin entry point loads dedicated PHP modules and initializes WooCommerce-dependent behavior after plugin loading.

## 2. Plugin bootstrap and administration

The main plugin file registers activation and deactivation hooks, loads the modules, checks the WooCommerce dependency, and initializes the error-log table when needed.

The administration layer is organized across specialized files for settings, product management, diagnostic tools, error logs, import/export, profiling and maintenance. The settings interface stores shared pricing parameters as WordPress options; product-specific formulas and labour inputs are obtained from product metadata.

These are architectural responsibilities, not evidence that every administration action has completed security and end-to-end validation.

## 3. Pricing engine

The calculation module coordinates the price calculation for each product variation.

1. Retrieve relevant variation attributes and stored values.
2. Determine the material category (gold, silver or palladium-containing gold).
3. Read configurable material prices, labour information, multipliers and option prices.
4. Select a product-specific gold or silver formula when available, otherwise use the global formula.
5. Substitute numeric values into the arithmetic expression, evaluate and round the result.
6. Return the price to the batch update process.

Missing or non-positive labour time prevents a price calculation and triggers a logged error. The batch variation loop skips variations with missing or non-positive weight.

**Security caveat:** arithmetic expressions are currently evaluated using filtered input followed by PHP `eval()`. Filtering should not be described as equivalent to a safe arithmetic parser; this deserves a separate security review.

For more detail, see [Pricing Engine](pricing-engine.md).

## 4. WooCommerce data access and persistence

Variation data is retrieved through optimized database access routines, alongside WooCommerce APIs and WordPress metadata operations.

Price persistence uses grouped SQL updates to `_price` and `_regular_price` post metadata. The plugin subsequently clears relevant caches/transients and synchronizes the variable parent product.

This is a deliberate performance-oriented design, with a trade-off: direct metadata writes may bypass some WooCommerce data-store and lifecycle behaviors. Compatibility and consistency should be verified against the deployed WooCommerce version.

## 5. Product weight management

A dedicated weight-management module provides authenticated AJAX actions for updating variation weights. It also registers a WordPress hook for scheduled background weight updates and includes a status endpoint.

Weight data is business-critical: a missing or invalid weight can prevent reliable price calculation. The scheduled mechanism is **WordPress hook-based**, not an external queue or separately deployed worker.

## 6. Administrative update workflows

The plugin exposes AJAX handlers for updating the prices of a single variable product or multiple selected products. The handlers verify the `manage_woocommerce` capability and report counts of updated and skipped variations, plus execution times.

**Validation item:** the inspected price-update AJAX handlers do not themselves show nonce checks. Authorization and CSRF protection are separate controls; the complete request path should be reviewed before claiming comprehensive request security.

## 7. Diagnostics and configuration exchange

A dedicated logger writes pricing and weight-related errors to a plugin-owned database table, including product and variation context. Administrative tools provide filtered log views.

The plugin also includes JSON import/export for selected settings and a profiler that measures execution times and SQL query behavior. These facilities support troubleshooting, but do not constitute a verified benchmark or complete observability platform.

## 8. Key architectural risks and improvement opportunities

- Replace dynamic `eval()` with a restricted arithmetic expression parser.
- Verify nonce checks for all state-changing requests.
- Handle unsuccessful calculations explicitly before batch persistence.
- Test direct SQL updates against WooCommerce caching and persistence behavior.
- Consider chunking/background processing for very large catalogues.
- Expand tests for formula selection, missing input, rounding, materials and variation updates.
- Validate access permissions for configuration import/export and diagnostics.

These are **review priorities**, not statements that the current implementation is fully hardened or that the proposed changes are already complete.

## Source code and confidentiality

This architecture case study is based on selected components of the private plugin repository. It does not publish proprietary source code, actual client price lists or confidential commercial formulas.
