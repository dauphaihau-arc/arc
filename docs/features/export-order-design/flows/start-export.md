# Start Export

See the [design](../README.md) and [flow index](../flow.md).

```mermaid
sequenceDiagram
    actor Seller
    participant Web as Seller Web
    participant API as Shop Order Export API

    Seller->>Web: Choose range, timezone, and columns
    Web->>Web: Convert calendar range to timestamps
    Web->>Web: Build request from current order filters
    Web->>API: POST /shops/:shop_id/orders/exports
    API->>API: Authorize seller for shop
    API->>API: Store export request snapshot
    API-->>Web: Export status response
```

The request includes existing order filters, export metadata, and either the default column preset or selected custom column IDs.
