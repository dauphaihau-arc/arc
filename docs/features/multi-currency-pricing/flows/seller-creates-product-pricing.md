# Seller Creates Product Pricing For Storefront Display

This flow describes how an existing Product's seller-authored base pricing becomes backend-resolved storefront display pricing. Product creation uses its creation workflow; later seller inventory and base-price edits use the complete Product Variant Configuration command.

```mermaid
sequenceDiagram
    actor Seller
    participant Web as Seller Web
    participant API as Product Variant Configuration API
    participant Catalog as Catalog Pricing
    participant Projection as Catalog Projection
    participant FX as FX Service
    participant Storefront as Storefront Reads

    Seller->>Web: Edit inventory or product base price
    Web->>API: PUT variant-configuration with versions and Idempotency-Key
    API->>API: Authorize and validate complete configuration
    API->>Catalog: Commit configuration and base-price changes atomically
    Note over API,Catalog: Close superseded active base price and insert replacement when changed
    API-->>Web: Return normalized Product detail with current base prices
    API-->>Projection: Dispatch projection after successful command
    Projection->>Catalog: Resolve active base or market price
    opt Indexed market or currency needs conversion
        Projection->>FX: Resolve rate and rounding policy
    end
    Projection->>Projection: Write indexed storefront price snapshots
    Storefront->>Projection: Read configured market/currency pair
    Projection-->>Storefront: Backend-resolved display price
```

Summary:

- seller authors canonical base pricing
- each priced Inventory Item references a Product Variant, including the zero-selection default
- configuration edits use `PUT /v1/shops/:shop_id/products/:product_id/variant-configuration`; this is not a separate per-row pricing mutation endpoint
- rejected configuration edits must not partially apply their price changes
- the current seller pricing write API does not expose market override creation
- catalog projection precomputes storefront pricing for configured indexed market/currency pairs
- backend decides whether to use an existing market override or base-price conversion
- storefront receives backend-resolved display pricing

See [Product Variant Design](../../product-variant-design/README.md) for identity, replacement inventory, version conflicts, and retry semantics. Canonical price selection, market overrides, and FX resolution remain owned by [Multi-Currency Pricing](../README.md).
