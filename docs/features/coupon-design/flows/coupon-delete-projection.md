# Coupon Delete Projection

This flow describes catalog cleanup after a seller deletes one or more coupons.

```mermaid
sequenceDiagram
    actor Seller
    participant API as Delete Coupon Use Case
    participant Coupon as Coupon Repository
    participant Jobs as Job Dispatcher
    participant Projector as Catalog Projection Job
    participant Catalog as Catalog Documents

    Seller->>API: Delete coupon
    API->>Coupon: Load coupon with shop owner
    API->>API: Validate shop ownership
    API->>Coupon: Remove coupon and flush
    alt Deleted coupon was active auto-sale
        API->>Jobs: Dispatch affected product projection
        Jobs->>Projector: Recompute prices without deleted coupon
        Projector->>Catalog: Replace projected price documents
    else Manual or inactive coupon
        API-->>Seller: Delete complete without projection
    end
```

Summary:

- deletion flushes before projection dispatch
- active auto-sale deletion must reproject affected products so stale sale prices disappear
- manual coupon deletion does not require catalog projection
- bulk deletion can combine product ids and dispatch one projection job
