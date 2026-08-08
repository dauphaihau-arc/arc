# Multi-Currency Pricing

## Overview

Multi-Currency Pricing defines how ARC owns, presents, quotes, stores, and charges money across catalog browsing and checkout.

The feature keeps seller-authored catalog prices separate from buyer-facing presentment currency and backend-owned checkout currency. This prevents storefront display, checkout, orders, and payment providers from becoming competing pricing authorities.

For lifecycle examples, see [flow.md](flow.md). For quote lifecycle details, see [checkout-quote-design.md](../../../apps/api/api/docs/checkout-quote-design.md). For field-level catalog price semantics, see [variant-prices-table.md](../../../apps/api/api/docs/multi-currency/variant-prices-table.md).

## Goal

The goal is to make pricing consistent from catalog through order:

- sellers author canonical catalog prices
- buyers can browse with market and currency context
- checkout creates a persisted pricing quote before order creation
- orders store immutable charged-currency snapshots
- payment providers consume backend-quoted totals and do not reprice

## Scope

In scope:

- seller-authored base catalog prices
- optional market-specific price overrides when present
- storefront display pricing for market and currency context
- indexed storefront price projections for configured market/currency pairs
- fallback live price resolution for non-indexed pairs
- persisted checkout quotes
- order money snapshots
- pricing provenance for FX-derived amounts

Out of scope:

- seller UX for managing market-specific override rows
- payment-provider currency selection outside backend checkout policy
- post-purchase repricing
- accounting, tax remittance, and refund policy design
- full exchange-rate administration UI

## Core Concepts

### Catalog Price

Catalog price is the seller-authored source price for a sellable inventory item.

Rules:

- every sellable item has one active base price
- optional market override prices may exist for specific markets
- catalog prices are not derived from cart, checkout, order, or payment state
- catalog pricing is the source for merchandise pricing before quote creation

### Presentment Currency

Presentment currency is the buyer-facing currency context used while browsing and reviewing checkout.

Rules:

- it is a display concern
- it may differ from the final checkout currency
- it must not redefine who owns catalog price truth
- any conversion is backend-derived from catalog pricing

### Checkout Currency

Checkout currency is the currency selected by backend policy for quote, order, and payment.

Rules:

- one quote has exactly one checkout currency
- all quote item totals use that checkout currency
- payment providers receive the checkout currency and quoted totals

### Order Snapshot

An order stores the final purchased amounts as immutable money snapshots.

Rules:

- order totals do not change when catalog prices later change
- order item currency matches order currency
- refunds and support workflows use stored order amounts, not live catalog pricing

## Pricing Model

The system separates pricing into four layers:

1. catalog/base price
2. optional market override price
3. storefront display price
4. checkout quote

Interpretation:

- catalog/base price is canonical by default
- market override price is canonical for that market when present
- storefront display price is a derived browsing view
- checkout quote is the transactional pricing checkpoint before order creation

## Price Resolution

Storefront and checkout price resolution follow the same ownership boundary but serve different moments in the buyer journey.

Browsing:

1. Resolve the active catalog price for the inventory item.
2. Prefer an active market override when one exists for the target market.
3. Use indexed storefront pricing when the market/currency pair is precomputed.
4. Fall back to live canonical price resolution and FX conversion when needed.
5. Return backend-resolved display pricing to the storefront.

Checkout:

1. Resolve the selected cart or buy-now items.
2. Select one allowed checkout currency by backend policy.
3. Persist quoted item amounts, totals, and pricing provenance.
4. Create orders from the persisted quote instead of recalculating from mutable cart or display state.

Important boundary:

- display conversion may happen during catalog projection or on demand before purchase
- transactional pricing becomes authoritative only after quote creation

## API Contract

Seller pricing APIs should express catalog pricing explicitly:

- base price
- compare-at price
- optional market-specific prices when the seller-facing surface supports them

Checkout APIs should create a quote before order creation. A quote response should include:

- `quote_id`
- `checkout_currency`
- quoted item amounts
- quoted totals
- expiration information

Order creation should consume a `quote_id`, not raw client-authored totals or arbitrary currency input.

## User-Visible Behavior

Seller behavior:

- sellers author product prices in the seller dashboard
- current seller pricing writes update base pricing
- market override rows may exist in the model, but they are not exposed through the normal seller pricing write flow today

Buyer behavior:

- buyers browse products with market and currency context
- storefront display currency can differ from checkout currency
- checkout displays backend-quoted totals before order creation
- payment amount must match the accepted quote

## Invariants

Catalog:

- every sellable inventory row has exactly one active base price
- active price rows must not overlap for the same inventory item and market
- amounts are stored in minor units

Quote:

- one quote has exactly one checkout currency
- quote totals equal quoted items plus shipping and discounts
- quote expiration is enforced before order creation
- quote amounts are persisted and not recalculated during payment creation

Order:

- order currency is the actual charged currency
- order item currency matches order currency
- order prices remain unchanged after purchase

Payment:

- payment totals exactly match persisted quote totals
- payment creation does not perform repricing
- payment creation does not rebuild totals from mutable cart state

## Design Decisions

### Catalog price is canonical

Catalog pricing belongs to the seller and should be authored once, then reused consistently across browse, quote, order, and payment.

### Presentment currency is separate from checkout currency

Browsing needs flexible display context, but purchase needs one backend-controlled currency for correctness, auditability, and payment integration.

### Quote persistence is required

Without a persisted quote, checkout depends on mutable catalog, cart, and FX state. A quote creates a stable pricing contract for the purchase attempt.

### Payment cannot reprice

If payment providers recalculate totals, the system loses its single source of truth and can charge a different amount than the buyer accepted.

## Related Documents

- [Flow](flow.md)
- [Variant Prices Table](../../../apps/api/api/docs/multi-currency/variant-prices-table.md)
- [Checkout Quote Design](../../../apps/api/api/docs/checkout-quote-design.md)
- [ADR-001: Canonical Catalog Pricing With Quote-Based Multi-Currency Checkout](../../../apps/api/api/docs/adrs/001-adopt-canonical-catalog-pricing-with-quote-based-multi-currency-checkout.md)
