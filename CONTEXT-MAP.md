# Context Map

Arc has multiple domain contexts. Each context owns its glossary and relationships; architectural choices and tradeoffs live in the relevant ADR directory.

## Contexts

- [Seller Catalog](./docs/domain/seller-catalog/CONTEXT.md) - Seller-managed Products, Product Variants, and commercial lifecycle.
- [Categories](./docs/domain/categories/CONTEXT.md) - Category hierarchy, stable public identities, current slugs, and historical path ownership.
- [Inventory](./docs/domain/inventory/CONTEXT.md) - Stable stock identities, one internal default seller Stock Pool per Inventory Item, SKUs, physical quantities, reservations, and inventory history.
- [Location](./docs/domain/location/CONTEXT.md) - Canonical Countries and first-level Administrative Areas referenced by addresses and destination pickers.
- [Checkout](./docs/domain/checkout/CONTEXT.md) - Current-offer Purchase Eligibility, default-pool reservation, and reservation outcomes.
- [Shipping](./docs/domain/shipping/CONTEXT.md) - Reusable, shop-owned Shipping Profiles, their Destination Rates, Shipping Coverage, Shipping Preview, and the Default Shipping Profile.
- [Market](./docs/domain/market/CONTEXT.md) - Buyer commercial contexts, enabled Markets, currencies, supported locales, and each enabled Market's single preview country.
- [Ordering](./docs/domain/ordering/CONTEXT.md) - Confirmed Order facts, historical identity, and retention.
- [Fulfillment](./docs/domain/fulfillment/CONTEXT.md) - Fulfillment responsibility, assignments, groups, physical Shipments, progress, and exceptions for confirmed Order quantities.
- [Product Imports](./docs/domain/product-imports/CONTEXT.md) - Asynchronous XLSX submissions that create Product drafts with row-level outcomes.
- [Promotions](./docs/domain/promotions/CONTEXT.md) - Seller Sales, Checkout Discounts, Promo Codes, and their discoverability.
- [Finance](./docs/domain/finance/CONTEXT.md) - Journal Entries, Ledger Accounts, Seller Payable, Platform Commission, and Payouts for confirmed Order money.
- [Localization](./docs/domain/localization/CONTEXT.md) - Storefront Language, Locale, Buyer Language Preference, UI Translation, Localized Content ownership, and outbound-message language.

## Relationships

- **Seller Catalog -> Inventory**: Seller Catalog references stable Inventory Item identities and publishes lifecycle changes that alter reservability and SKU ownership.
- **Checkout -> Seller Catalog**: Checkout asks Seller Catalog for current Product and Product Variant lifecycle state.
- **Seller Catalog -> Shipping**: Seller Catalog assigns at most one Shipping Profile per Product; a published Product must hold a Checkout-ready one.
- **Checkout -> Shipping**: Checkout resolves each purchased Product's assigned Shipping Profile to a Destination Rate for the buyer's real shipping address; any selected line without a rate rejects the whole quote with `CHECKOUT_SHIPPING_UNAVAILABLE`. The Preview Destination never prices a checkout.
- **Market -> Shipping**: Each enabled Market declares one `preview_country_code`; Shipping interprets it as the Preview Destination for live Shipping Preview, catalog Shipping Coverage and badges, cart pre-checks, and suggestion/recommendation filtering. Market never resolves shipping itself.
- **Market -> Checkout**: Checkout uses the real shipping address, never the Market's Preview Destination; a supported address remains purchasable even when the preview country is unsupported, and Checkout notes when the address country differs from the Preview Destination.
- **Checkout -> Inventory**: Checkout reserves, releases, and consumes quantities in each Inventory Item's default seller Stock Pool.
- **Checkout -> Ordering**: A successful final Purchase Eligibility check produces a confirmed Order and its Order Item Snapshots.
- **Checkout -> Fulfillment**: Checkout establishes the Delivery Promise and Shipping Charge and holds Checkout Reservations; confirmation creates the seller Fulfillment Group with its exact quantities, and Fulfillment owns physical execution.
- **Ordering -> Fulfillment**: Ordering retains confirmed purchase facts, including the confirmed assignment and accepted Delivery Promise; Fulfillment records groups, Shipments, Fulfillment Progress, and exceptions.
- **Fulfillment -> Inventory**: Fulfillment references Inventory quantities for execution; Inventory remains the authority for stock quantities, reservations, and movements.
- **Ordering -> Seller Catalog / Inventory**: Ordering retains purchase-time facts and historical references that constrain physical deletion.
- **Product Imports -> Seller Catalog**: Product Imports creates Product drafts through Seller Catalog rules.
- **Product Imports -> Inventory**: Product Imports checks SKU conflicts and creates Inventory Items through Inventory authority.
- **Categories -> Seller Catalog**: Seller Catalog products reference Categories; Category identity and hierarchy are managed separately from Product lifecycle.
- **Ordering -> Finance**: an Order's confirmed money facts and its Order Refunds are the source of Finance Journal Entries; Finance never re-prices an Order.
- **Finance -> Ordering**: Finance reads confirmed purchase facts and never writes Order facts.
- **Finance -> Payment**: Payouts leave through the payment provider adapter; Finance records provider outcomes rather than deciding whether a payment succeeds.
- **Localization -> Market**: a Locale pairs a Storefront Language with a Market's region; Market never constrains Storefront Language.
- **Localization -> Seller Catalog / Categories**: Localized Content is owned by the context that owns the record; the storefront selects a Locale and falls back to English per field.
- **Localization -> Auth**: Buyer Language Preference persists through `user_preferences.language` and guest preference storage.

These relationships describe the agreed target model. Current behavior records shipping details and tracking per shop Order as legacy evidence, exposes only Seller Fulfillment, and implements quantity-based Fulfillment Progress with partial Shipments. Seller Fulfillment is single-source: one internal default seller Stock Pool per Inventory Item. Reusable Shipping Profiles and their Product assignment exist, including the Checkout-ready designation. Market Shipping Preview and Shipping Coverage are implemented: each enabled Market declares a valid preview country, product detail resolves a live Shipping Preview, catalog documents carry projected coverage and accept the browse-only `catalog_ship_to` override, cart marks unsupported selected lines unavailable and blocks checkout, and checkout prices from the real shipping address. Buyer-chosen delivery options and multi-location or provider sourcing are not implemented.

## Architecture Decisions

- Repo-wide ADRs live in `docs/adr/`.
- API architecture ADRs live in `apps/api/api/docs/adrs/`.
- Context-specific ADRs may live beside their context docs.
