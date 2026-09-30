# Context Map

Arc has multiple domain contexts. Each context owns its glossary and relationships; architectural choices and tradeoffs live in the relevant ADR directory.

## Contexts

- [Seller Catalog](./docs/domain/seller-catalog/CONTEXT.md) - Seller-managed Products, Product Variants, and commercial lifecycle.
- [Inventory](./docs/domain/inventory/CONTEXT.md) - Stable stock identities, one internal default seller Stock Pool per Inventory Item, SKUs, physical quantities, reservations, and inventory history.
- [Checkout](./docs/domain/checkout/CONTEXT.md) - Current-offer Purchase Eligibility, default-pool reservation, and reservation outcomes.
- [Shipping](./docs/domain/shipping/CONTEXT.md) - Reusable, shop-owned Shipping Profiles, their Destination Rates, and the Default Shipping Profile.
- [Ordering](./docs/domain/ordering/CONTEXT.md) - Confirmed Order facts, historical identity, and retention.
- [Fulfillment](./docs/domain/fulfillment/CONTEXT.md) - Fulfillment responsibility, assignments, groups, physical Shipments, progress, and exceptions for confirmed Order quantities.
- [Product Imports](./docs/domain/product-imports/CONTEXT.md) - Asynchronous XLSX submissions that create Product drafts with row-level outcomes.
- [Promotions](./docs/domain/promotions/CONTEXT.md) - Seller Sales, Checkout Discounts, Promo Codes, and their discoverability.

## Relationships

- **Seller Catalog -> Inventory**: Seller Catalog references stable Inventory Item identities and publishes lifecycle changes that alter reservability and SKU ownership.
- **Checkout -> Seller Catalog**: Checkout asks Seller Catalog for current Product and Product Variant lifecycle state.
- **Seller Catalog -> Shipping**: Seller Catalog assigns at most one Shipping Profile per Product; a published Product must hold a Checkout-ready one.
- **Checkout -> Shipping**: Checkout resolves each purchased Product's assigned Shipping Profile to a Destination Rate for the buyer's destination.
- **Checkout -> Inventory**: Checkout reserves, releases, and consumes quantities in each Inventory Item's default seller Stock Pool.
- **Checkout -> Ordering**: A successful final Purchase Eligibility check produces a confirmed Order and its Order Item Snapshots.
- **Checkout -> Fulfillment**: Checkout establishes the Delivery Promise and Shipping Charge and holds Checkout Reservations; confirmation creates the seller Fulfillment Group with its exact quantities, and Fulfillment owns physical execution.
- **Ordering -> Fulfillment**: Ordering retains confirmed purchase facts, including the confirmed assignment and accepted Delivery Promise; Fulfillment records groups, Shipments, Fulfillment Progress, and exceptions.
- **Fulfillment -> Inventory**: Fulfillment references Inventory quantities for execution; Inventory remains the authority for stock quantities, reservations, and movements.
- **Ordering -> Seller Catalog / Inventory**: Ordering retains purchase-time facts and historical references that constrain physical deletion.
- **Product Imports -> Seller Catalog**: Product Imports creates Product drafts through Seller Catalog rules.
- **Product Imports -> Inventory**: Product Imports checks SKU conflicts and creates Inventory Items through Inventory authority.

These relationships describe the agreed target model. Current behavior records shipping details and tracking per shop Order as legacy evidence, exposes only Seller Fulfillment, and implements quantity-based Fulfillment Progress with partial Shipments. Seller Fulfillment is single-source: one internal default seller Stock Pool per Inventory Item. Reusable Shipping Profiles and their Product assignment exist, including the Checkout-ready designation; checkout-time Shipping Charges and buyer-chosen delivery options (tickets 08-09) and multi-location or provider sourcing are not implemented.

## Architecture Decisions

- Repo-wide ADRs live in `docs/adr/`.
- API architecture ADRs live in `apps/api/api/docs/adrs/`.
- Context-specific ADRs may live beside their context docs.
