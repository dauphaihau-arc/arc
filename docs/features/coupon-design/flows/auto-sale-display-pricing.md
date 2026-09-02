# Auto-Sale Display Pricing

This flow describes how an active auto-sale coupon becomes storefront display pricing.

```mermaid
sequenceDiagram
    actor Buyer
    participant Storefront
    participant Catalog as Catalog Documents
    participant Projector as Catalog Product Projector
    participant Pricing as StorefrontIndexedPriceProjectionService
    participant Coupon as CouponAutoSaleProjectionReader

    Projector->>Pricing: Project product prices
    Pricing->>Coupon: findBestAutoSaleForProduct(shop_id, product_id)
    Coupon-->>Pricing: coupon_id and percent_off
    Pricing->>Pricing: Discount base and indexed prices
    Pricing-->>Projector: Display price summary and inventory prices
    Projector->>Catalog: Save projected price documents
    Buyer->>Storefront: Open listing or detail page
    Storefront->>Catalog: Read projected product price
    Storefront-->>Buyer: Show sale price and original price
```

Summary:

- auto-sale display pricing is computed during catalog projection
- `findBestAutoSaleForProduct` returns the highest active percentage auto-sale matching the product
- the projected price carries `autoSale` metadata with `couponId` and `percentOff`
- checkout does not trust the displayed catalog price as final order authority
