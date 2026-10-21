# Custom Jewellery Pricing

### Moteur de calcul tarifaire configurable pour WordPress et WooCommerce

**PHP · WordPress · WooCommerce · JavaScript · AJAX · MySQL**

Plugin WordPress conçu sur mesure pour automatiser le calcul des prix de bijoux en fonction des matériaux, du poids, du temps de fabrication, des options et de formules configurables.

[🇬🇧 English](README.md) | 🇫🇷 Français

> Étude de cas technique publique uniquement. Le code source original du plugin reste privé.

---

## Présentation du projet

La tarification des bijoux dépend de nombreux paramètres : matière, poids, temps de fabrication, tarif horaire, coefficients et options. Le projet répondait au besoin de mettre à jour les prix de plusieurs produits WooCommerce, dont certains pouvaient comporter jusqu’à **4 500 variations**, rendant toute actualisation manuelle longue et source d’incohérences.

Custom Jewellery Pricing intègre un moteur de calcul configurable dans l'administration WooCommerce et permet le recalcul groupé des prix des variations.

## Mon rôle

**Développeuse PHP — Création de plugin WordPress et WooCommerce sur mesure**

Ce projet illustre le développement de calculs métier en PHP, l'intégration aux produits WooCommerce, la conception d'interfaces d'administration, la gestion d'actions AJAX, les outils de diagnostic et les optimisations de traitement.

## Fonctionnalités principales

- **Formules configurables :** formule globale et formules propres aux produits selon le métal, avec règles de repli.
- **Tarification selon la matière :** or, argent et or palladié.
- **Gestion des variations :** exploitation des attributs et métadonnées WooCommerce.
- **Recalcul individuel et multiple :** opérations AJAX permettant de mettre à jour un ou plusieurs produits.
- **Validation et journalisation :** détection de poids ou de temps de fabrication manquants, avec suivi des anomalies.
- **Import et export :** échange d'une partie des paramètres au format JSON.
- **Diagnostic :** journal des erreurs, outils de profilage et mesures de durée de traitement.
- **Gestion des poids :** opérations AJAX et traitements différés par hooks planifiés WordPress.

## Stack technique

| Domaine | Technologies |
|---|---|
| Back-end | PHP, API de plugins WordPress |
| E-commerce | WooCommerce, produits, variations et métadonnées |
| Base de données | API WordPress (`$wpdb`), MySQL |
| Administration | WordPress Settings API, PHP, JavaScript, AJAX |
| Traitements différés | Hooks planifiés WordPress |
| Échange de données | JSON |
| Dépendance de test | PHPUnit (déclaré en développement ; couverture non vérifiée) |

## Architecture

![Architecture de Custom Jewellery Pricing](assets/architecture.svg)

Le plugin relie l'administration WordPress, le moteur de calcul, les données des variations, la persistance WooCommerce et les outils de diagnostic. Ce schéma est conceptuel et ne constitue pas un audit de toutes les intégrations.

Consulter la [documentation de l'architecture](docs/architecture.md).

## Défis techniques

Les principaux enjeux concernent :

- La configuration des règles métier sans intervention manuelle sur chaque variation.
- L'interprétation des matériaux et attributs des produits.
- Le contrôle des données manquantes avant mise à jour des prix.
- L'optimisation des recalculs tout en préservant la cohérence des métadonnées et des caches.
- La sécurisation des expressions configurables et des opérations d'administration.
- La fiabilité des opérations longues et des traitements différés.

Voir les [défis techniques et choix d'implémentation](docs/technical-challenges.md).

## Documentation technique

- [Architecture technique](docs/architecture.md) — Composants WordPress/WooCommerce et interactions.
- [Moteur de tarification](docs/pricing-engine.md) — Variables, formules, calculs par variation et enregistrement des prix.
- [Défis techniques](docs/technical-challenges.md) — Compromis, points à vérifier et évolutions possibles.

*Les documents techniques détaillés sont rédigés en anglais.*

## Confidentialité

Cette étude de cas ne publie ni le code source privé, ni les prix réels de la cliente, ni les paramètres commerciaux confidentiels, ni des informations permettant d'identifier ses clients.
