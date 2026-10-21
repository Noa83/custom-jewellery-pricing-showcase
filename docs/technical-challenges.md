# Technical Challenges & Engineering Decisions

## Overview

Custom Jewellery Pricing is a custom WordPress/WooCommerce plugin designed to calculate jewellery prices using material-specific rules, configurable expressions, product metadata and variation attributes.

The following case study distinguishes **mechanisms identified in the private source repository** from **risks and improvements that still require validation**. It does not claim that the plugin has passed a full security or performance audit.

## 1. Configurable business calculations

### Challenge

Jewellery prices vary with raw materials, weight, labour time, coefficients and optional components. Hard-coding a separate price for each WooCommerce variation would make catalogue maintenance expensive and error-prone.

### Implemented approach

The plugin stores shared inputs using WordPress options and reads product-specific formulas from post metadata. The calculation layer resolves the variation's material category, loads applicable variables, selects a formula and produces a rounded price.

The formula-selection hierarchy is:

| Material category | Preferred formula | Fallback |
|---|---|---|
| Gold | Product-specific gold formula | Global formula |
| Palladium-containing gold | Product-specific gold formula | Global formula |
| Silver | Product-specific silver formula | Global formula |

### Trade-offs and validation

A flexible formula format increases configurability, but also increases the need for validation. Arithmetic expressions are currently filtered and then executed using PHP `eval()`. **This is a security review priority**; replacing it with a restricted arithmetic parser would provide a clearer execution boundary and easier testing.

The code also contains a predefined calculation fallback when formula evaluation fails. Testing should confirm whether such fallback behavior is appropriate for every product category.

## 2. Mapping WooCommerce variation attributes to pricing inputs

### Challenge

A single product may have numerous variations. Material labels and other attributes may be stored in different forms, but the pricing engine needs consistent numeric inputs.

### Implemented approach

Helper and calculation modules normalize material identifiers, inspect variation attributes and can persist a normalized material category to variation metadata. Data-access routines retrieve variation information for product-wide calculations.

The engine supports categories including gold, silver and palladium-containing gold. It also reads weight, labour-related product metadata and configurable option prices.

### Trade-offs and validation

Normalizing heterogeneous attributes avoids repeatedly embedding attribute parsing into formulas. However, implicit defaults may conceal missing or ambiguous product data. Regression tests should cover mixed-material variations, unknown labels, changed taxonomy values and stale cached metadata.

## 3. Preventing incorrect price updates

### Challenge

Missing weight or labour information could result in incorrect commercial prices if the plugin silently continues.

### Implemented approach

The calculation path checks for missing or non-positive labour time and records an error rather than returning a valid price. The variation-batch loop skips missing or non-positive weights and logs the reason.

A custom error-log table records the problem type, affected product/variation and contextual information. Administrative tooling exposes diagnostic information.

### Trade-offs and validation

The current per-variation processing path deserves particular attention when a calculation returns `null`: the batch result collection should not pass invalid values to price persistence. A stronger boundary would validate every calculated amount before preparing SQL updates.

A useful test suite should cover missing inputs, malformed numeric values, invalid formula expressions, zero/negative results, rounding and partially valid batches.

## 4. Efficient updates across large variation catalogues

### Challenge

Updating each WooCommerce variation individually can be costly when products have many attribute combinations.

### Implemented approach

A bulk update function groups updates to `_price` and `_regular_price` metadata through direct SQL, adds missing metadata records, then invalidates caches and synchronizes the parent variable product.

The AJAX administration layer provides both single-product and multi-product update actions, including counts of processed and skipped variations and elapsed processing time.

A profiler provides measurements such as query counts, SQL execution durations and variation-level processing times.

### Trade-offs and validation

Direct SQL can reduce application overhead, but it may bypass WooCommerce data-store hooks, plugin integrations and other lifecycle effects. Behaviour must be tested against the WooCommerce version actually deployed, including caches, product visibility, sale-price interactions and dependent extensions.

The AJAX handlers also raise PHP execution and memory limits for long-running requests. For very large catalogues, chunked processing, persistent progress state, retry-safe jobs and resumable operations are potential improvements rather than implemented guarantees.

## 5. Administration operations and security

### Challenge

Pricing controls are commercially sensitive: unauthorized recalculation or parameter changes could affect an entire product catalogue.

### Implemented approach

The price-update AJAX actions check the `manage_woocommerce` capability and validate basic input identifiers. The administrative interface relies on WordPress capabilities and settings mechanisms. Import/export forms include WordPress nonce fields with corresponding checks.

### Trade-offs and validation

**The two price-update AJAX handlers inspected do not perform an explicit nonce check.** Whether a separate layer enforces CSRF protection across the entire request flow must be verified. Capability checks do not replace nonce validation for state-changing browser requests.

Recommended review areas include all AJAX handlers, import validation, escaping, stored settings, error exposure and permissions for background operations. These are review recommendations, not claims of confirmed exploits.

## 6. Asynchronous weight updates and operational reliability

### Challenge

Weight metadata may require recalculation or adjustment across numerous product variations. Long synchronous requests can time out or leave incomplete results.

### Implemented approach

The plugin contains AJAX actions for weight management and a WordPress scheduled-hook mechanism for deferred weight updates, with status checking and cleanup functions.

### Trade-offs and validation

Scheduled WordPress execution depends on the host's cron configuration and request activity. Operational testing should verify execution, recovery from interrupted jobs, prevention of duplicate processing, timeout behavior and progress reporting.

## 7. Configuration portability and diagnostic tools

### Challenge

Business users need to understand configuration changes and investigate failures without editing PHP files.

### Implemented approach

The plugin includes a WordPress settings interface, JSON import/export for selected configuration values, a dedicated error log and performance profiling tools.

### Trade-offs and validation

JSON import requires strict schema validation and explicit handling of missing, unexpected and malformed values. Diagnostic output should avoid exposing customer-specific prices or sensitive information to unauthorized users.

Performance metrics from the built-in profiler are useful for investigation, but they are not equivalent to a reproducible benchmark or a guarantee of a side-effect-free simulation without additional verification.

## 8. Testing and maintainability

### Challenge

A formula engine combines many interacting rules. Small changes to attribute interpretation or rounding can affect numerous commercial prices.

### Evidence and priorities

The project's Composer configuration declares PHPUnit as a development dependency. **This alone does not establish current automated coverage**.

High-value tests would include:

- Selection and fallback of material-specific formulas.
- Parsing and validation of arithmetic expressions.
- Variation classification, including mixed and missing material values.
- Input validation for weight, labour and options.
- Rounding and invalid/negative price behavior.
- Bulk persistence and WooCommerce cache synchronization.
- Authorization and CSRF protection of admin actions.
- Interruption/retry behavior for deferred weight operations.

A longer-term maintainability improvement would be separating pure pricing calculations from WordPress-specific data access, which would make deterministic unit tests easier to write.

## Conclusion

Custom Jewellery Pricing illustrates a recurring software-engineering challenge: embedding configurable, domain-specific business logic into an existing platform without compromising data consistency, security, compatibility or operational performance.

**Confirmed implementation areas:** configurable formulas, material classification, variation-level calculations, bulk price updates, WordPress/WooCommerce integration, diagnostics and deferred weight processing.

**Open review areas:** safe formula evaluation, validation of all AJAX mutation paths, handling of invalid calculation results, WooCommerce lifecycle compatibility, large-catalogue reliability and automated test coverage.

## Confidentiality

This case study deliberately excludes proprietary source code, real customer price lists, production configuration and identifying customer data. Examples in companion documents use fictitious values.
