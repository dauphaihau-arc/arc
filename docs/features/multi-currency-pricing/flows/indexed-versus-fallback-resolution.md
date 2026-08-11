# Indexed Versus Fallback Price Resolution

This flow describes how storefront price resolution chooses between projected indexed prices and canonical fallback resolution.

```mermaid
flowchart TD
    A[Storefront request includes market and currency] --> B{Market/currency pair is indexed?}
    B -- Yes --> C[Read projected Mongo or Atlas storefront price]
    B -- No --> D[Load active canonical price from variant_prices]
    D --> E{Market override exists?}
    E -- Yes --> F[Use market override as source price]
    E -- No --> G[Use active base price]
    F --> H{Display currency matches source currency?}
    G --> H
    H -- Yes --> I[Return resolved display amount]
    H -- No --> J[Apply FX conversion and rounding policy]
    J --> K[Cache fallback result for short TTL when configured]
    K --> I
    C --> I
```

Summary:

- indexed pairs use precomputed storefront price documents for read performance
- fallback pairs continue to resolve from canonical `variant_prices`
- market override rows win over base-price conversion when present
- FX conversion is a browsing concern until quote creation persists transactional amounts
