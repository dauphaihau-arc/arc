# Coupon Product Scope

This flow describes how coupon product targeting affects projection.

```mermaid
flowchart TD
    Start[Project one product price] --> Load[Load active auto-sale percentage coupons for the shop]
    Load --> Each{For each coupon}
    Each --> All{appliesTo = ALL?}
    All -->|Yes| Candidate[Coupon matches product]
    All -->|No| Specific{appliesTo = SPECIFIC and product id is listed?}
    Specific -->|Yes| Candidate
    Specific -->|No| Ignore[Ignore coupon for this product]
    Candidate --> Positive{percentOff > 0?}
    Positive -->|Yes| Keep[Keep candidate]
    Positive -->|No| Ignore
    Keep --> Pick[Pick kept coupon with highest percentOff]
    Ignore --> Pick
    Pick --> Found{Any kept coupon?}
    Found -->|Yes| Sale[Apply sale display price]
    Found -->|No| Normal[Use normal active product price]
```

Summary:

- `appliesTo=ALL` affects every product in the shop
- `appliesTo=SPECIFIC` affects only listed product ids
- non-matching product-scoped coupons are ignored even if they have a higher percentage
- when no coupon matches, projected display pricing uses the product's normal active price
