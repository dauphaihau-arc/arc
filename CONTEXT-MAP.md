# Context Map

Arc has multiple domain contexts. Each context owns its glossary and relationships; architectural choices and tradeoffs live in the relevant ADR directory.

## Contexts

- [Seller Catalog](./docs/domain/seller-catalog/CONTEXT.md) - Seller-managed Products, Product Variants, and commercial lifecycle.
- [Inventory](./docs/domain/inventory/CONTEXT.md) - Stable stock identities, SKUs, physical quantities, reservations, and inventory history.
- [Checkout](./docs/domain/checkout/CONTEXT.md) - Current-offer Purchase Eligibility and reservation outcomes.
- [Ordering](./docs/domain/ordering/CONTEXT.md) - Confirmed Order facts, historical identity, and retention.
- [Product Imports](./docs/domain/product-imports/CONTEXT.md) - Asynchronous XLSX submissions that create Product drafts with row-level outcomes.

## Relationships

- **Seller Catalog -> Inventory**: Seller Catalog references stable Inventory Item identities and publishes lifecycle changes that alter reservability and SKU ownership.
- **Checkout -> Seller Catalog**: Checkout asks Seller Catalog for current Product and Product Variant lifecycle state.
- **Checkout -> Inventory**: Checkout asks Inventory to reserve, release, and consume quantities.
- **Checkout -> Ordering**: A successful final Purchase Eligibility check produces a confirmed Order and its Order Item Snapshots.
- **Ordering -> Seller Catalog / Inventory**: Ordering retains purchase-time facts and historical references that constrain physical deletion.
- **Product Imports -> Seller Catalog**: Product Imports creates Product drafts through Seller Catalog rules.
- **Product Imports -> Inventory**: Product Imports checks SKU conflicts and creates Inventory Items through Inventory authority.

## Architecture Decisions

- Repo-wide ADRs live in `docs/adr/`.
- API architecture ADRs live in `apps/api/api/docs/adrs/`.
- Context-specific ADRs may live beside their context docs.
