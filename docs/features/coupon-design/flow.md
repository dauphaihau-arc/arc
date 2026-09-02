# Coupon Flow

This document is the entry point for end-to-end coupon lifecycle examples across auto-sale display pricing, manual redemption, product targeting, and catalog reprojection.

For scope, vocabulary, boundaries, and implementation notes, see [README.md](README.md).

## Main Flow

```mermaid
sequenceDiagram
    actor Seller
    actor Buyer
    participant ShopAPI as Shop Coupon API
    participant Coupon as Coupon Repository
    participant Jobs as Catalog Projection Jobs
    participant Projector as Product Projector
    participant Pricing as Storefront Price Projection
    participant Storefront as Storefront Catalog
    participant Checkout as Checkout Quote API

    Seller->>ShopAPI: Create auto-sale coupon
    ShopAPI->>Coupon: Persist coupon
    ShopAPI->>Jobs: Schedule affected product projection
    Jobs->>Projector: Project shop products
    Projector->>Pricing: Resolve indexed prices
    Pricing->>Coupon: Find best auto-sale for product
    Coupon-->>Pricing: coupon_id and percent_off
    Pricing-->>Projector: Sale-adjusted display prices
    Projector->>Storefront: Save catalog price documents
    Buyer->>Storefront: Browse product listing or detail
    Storefront-->>Buyer: Show discounted display price
    Buyer->>Checkout: Create checkout quote
    Checkout->>Coupon: Validate manual code or current coupon rules
    Checkout-->>Buyer: Quote-owned totals
```

Summary:

- auto-sale coupons affect catalog/storefront display prices through projection
- manual coupons affect checkout quotes through redemption validation
- catalog display pricing is not the source of checkout truth
- the best auto-sale for a product is the highest active percentage coupon matching the product scope

## Detailed Flows

- [Auto-Sale Display Pricing](flows/auto-sale-display-pricing.md)
- [Manual Coupon Redemption](flows/manual-coupon-redemption.md)
- [Coupon Create Projection](flows/coupon-create-projection.md)
- [Coupon Delete Projection](flows/coupon-delete-projection.md)
- [Coupon Expiry Projection](flows/coupon-expiry-projection.md)
- [Coupon Product Scope](flows/coupon-scope-products.md)
