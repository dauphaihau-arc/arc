# Coupon Expiry Projection

This flow describes start and end date projection for auto-sale coupons.

```mermaid
sequenceDiagram
    participant Scheduler as Projection Scheduler
    participant Jobs as Job Dispatcher
    participant Projector as Catalog Projection Job
    participant Coupon as CouponAutoSaleProjectionReader
    participant Catalog as Catalog Documents

    Scheduler->>Jobs: Schedule projection at startDate
    Scheduler->>Jobs: Schedule projection at endDate
    Jobs->>Projector: Run at date boundary
    Projector->>Coupon: Find current best auto-sale for each product
    Coupon-->>Projector: Current active coupon or none
    Projector->>Catalog: Save current display pricing
```

Summary:

- future auto-sales need a start-date projection so discounts appear when active
- active auto-sales need an end-date projection so discounts disappear when expired
- projection recomputes from current coupon state instead of trusting the originally scheduled coupon
- overlapping coupons are resolved by the current best matching auto-sale at projection time
