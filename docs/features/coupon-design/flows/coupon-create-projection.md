# Coupon Create Projection

This flow describes seller coupon creation and catalog projection scheduling.

```mermaid
sequenceDiagram
    actor Seller
    participant API as Create Shop Coupon Use Case
    participant Coupon as Coupon Repository
    participant Scheduler as Projection Scheduler
    participant Jobs as Job Dispatcher
    participant Projector as Catalog Projection Job

    Seller->>API: Create coupon
    API->>API: Validate shop ownership and coupon input
    API->>Coupon: Persist coupon
    API->>Scheduler: scheduleShopCouponCatalogProjection(coupon)
    alt Active auto-sale now
        Scheduler->>Jobs: Dispatch immediate projection
    else Future auto-sale
        Scheduler->>Jobs: Dispatch delayed start projection
    end
    Scheduler->>Jobs: Dispatch delayed end projection
    Jobs->>Projector: Reproject affected shop products
```

Summary:

- all coupons are persisted before projection scheduling
- only active or scheduled auto-sale percentage coupons need catalog display-price projection
- `appliesTo=ALL` schedules shop-wide projection
- `appliesTo=SPECIFIC` schedules projection for the listed product ids
