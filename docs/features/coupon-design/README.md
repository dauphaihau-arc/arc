# Coupon Design

## Overview

Coupons let shops offer discounts through two distinct paths:

- manual coupon redemption during checkout, by entering a code or selecting a listed shop coupon
- auto-sale coupons that affect storefront display pricing without buyer input

Manual coupons are checkout adjustments. Auto-sale coupons are catalog pricing projections. They share coupon data, but they have different correctness boundaries.

## Goals

- Keep checkout totals owned by backend quote and order creation logic.
- Let sellers configure manual and auto-sale coupons from shop coupon management.
- Show auto-sale discounts in storefront product cards and product details.
- Reproject affected catalog products when auto-sale coupons become active, expire, are created, or are deleted.
- Keep coupon persistence behind repository/query ports for ordinary reads and writes.

## Scope

In scope:

- Shop coupon creation and deletion.
- Percentage and fixed-amount coupon data model behavior.
- Auto-sale percentage coupon display pricing.
- Product-scoped and all-products coupon targeting.
- Catalog projection scheduling for auto-sale coupon lifecycle changes.
- Manual coupon redemption at checkout.

Out of scope:

- Payment provider behavior.
- Seller payout or accounting policy.
- Marketing campaign analytics.
- Notification campaigns.
- Tax policy.

## Vocabulary

- Manual coupon: a coupon applied by the buyer during checkout, either by code or from the shop coupon picker.
- Auto-sale coupon: a coupon marked `isAutoSale` that is applied to storefront display pricing automatically.
- Display pricing: projected catalog price shown before checkout.
- Checkout pricing: quote-owned totals used for order creation.
- Product scope: coupon targeting through `appliesTo=ALL` or `appliesTo=SPECIFIC` plus `appliesProductIds`.
- Coupon visibility: `public` makes an eligible manual coupon discoverable in checkout; `code_only` permits redemption by code without listing it. Visibility does not change redemption eligibility.
- Coupon currency: the currency a Coupon's monetary fields (`amountOff`, `minOrderValue`) are denominated in. It is snapshotted from the owning Shop at creation, so a Coupon's face value never silently changes with the buyer's presentment currency.
- Manual coupon slots: each shop can apply one `FREE_SHIP` coupon and one `FIXED_AMOUNT` or `PERCENTAGE` coupon. Fixed-amount and percentage coupons share a slot and cannot stack.
- Projection: rebuilding catalog product/search price documents from source product data and active auto-sale coupons.

## Design Boundaries

Auto-sale display pricing must not become checkout authority. It can show buyers the expected sale price, but checkout still recomputes totals from current product, coupon, shipping, and buyer context.

Manual redemption must not rely on catalog projections. It validates the entered code against coupon rules at quote creation.

The checkout picker lists the shop's public manual coupons, including ones the current cart cannot use yet. Each listed coupon carries `is_eligible` and, when ineligible, the single `ineligible_reason` that currently blocks it, so the storefront can show what a coupon requires rather than hiding it. `code_only` coupons and automatic-sale coupons are never listed. Only the server decides eligibility: the storefront renders an ineligible coupon as a disabled card and never enables or hides one on its own. Applying a coupon still validates the resulting selection, so an ineligible code is rejected on apply even though it is visible in the list. Both public and code-only coupons use the same redemption validation; applying a new coupon replaces the existing coupon in its slot only after the resulting selection validates, and a failure preserves the buyer's previous selection.

Coupon monetary fields are authored in one canonical currency and compared against checkout-currency money only after conversion through the shared currency-conversion seam. A missing rate is never resolved by treating the coupon's face value as the checkout currency: the coupon is omitted from the picker and rejected on apply, so a buyer can never see or accept a discount the pricing path cannot honor.

Existing coupons migrate to `code_only` to avoid publishing previously unlisted codes. The seller creation form explicitly defaults new coupons to `public`. Seed data records both visibility and canonical currency explicitly in `apps/api/seed-data/coupons.tsv`; the loader and seed mapper preserve those values, and each row's currency must match its shop.

The coupon query used by catalog projection is represented by `CouponAutoSaleProjectionReader`. The MikroORM implementation owns direct `EntityManager` access; application services depend on the port.

## Main Components

- `CouponAutoSaleProjectionReader`: app-level port for finding the best active auto-sale coupon for a product.
- `MikroOrmCouponAutoSaleProjectionReader`: infra repository implementation that queries `CouponEntity`.
- `StorefrontIndexedPriceProjectionService`: applies the selected auto-sale discount to base and indexed storefront prices.
- Shop coupon create/delete use cases: persist coupon changes and schedule catalog reprojection jobs.
- `catalog.project-shop-products`: background job that refreshes catalog product projections for a shop or product subset.

## Main Flow

1. Seller creates an auto-sale coupon.
2. API persists the coupon.
3. API schedules catalog projection for the affected products and coupon date boundaries.
4. Projection loads products and asks `CouponAutoSaleProjectionReader` for the best auto-sale coupon per product.
5. Pricing projection applies the highest matching auto-sale percentage to storefront display prices.
6. Catalog product/search documents store the projected display price and auto-sale metadata.
7. Storefront reads catalog documents and displays the discounted price.
8. Checkout still creates a backend quote and validates coupon rules independently.

## Detailed Flows

See [flow.md](flow.md) for the high-level lifecycle map.

Scenario flows:

- [Auto-Sale Display Pricing](flows/auto-sale-display-pricing.md)
- [Manual Coupon Redemption](flows/manual-coupon-redemption.md)
- [Coupon Create Projection](flows/coupon-create-projection.md)
- [Coupon Delete Projection](flows/coupon-delete-projection.md)
- [Coupon Expiry Projection](flows/coupon-expiry-projection.md)
- [Coupon Product Scope](flows/coupon-scope-products.md)

## Open Questions

- Should future fixed-amount auto-sales be supported for display pricing?
- Should overlapping auto-sales always choose highest `percentOff`, or should priority be explicit?
- Should coupon update use cases schedule projection with the same lifecycle guarantees as create/delete?
