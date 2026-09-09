# Save A Complete Configuration

See the [design](../README.md) and [flow index](../flow.md).

```mermaid
sequenceDiagram
    actor Seller
    participant Web as Seller Editor
    participant API as Configuration Command
    participant DB as Product and Inventory Database
    participant Projection as Catalog Projection

    Web->>API: Load Product detail
    API-->>Web: Options, selections, inventory, versions
    Seller->>Web: Edit complete intended configuration
    Web->>API: PUT variant-configuration + Idempotency-Key
    API->>API: Authorize shop and validate target matrix
    API->>DB: Begin transaction and lock Product and inventory
    API->>DB: Validate versions, ownership, SKUs, reservations
    alt Target is valid
        API->>DB: Persist configuration, inventory, prices, mutation records
        API->>DB: Increment Product Version once and commit
        API->>Projection: Dispatch catalog projection
        API-->>Web: Updated normalized detail
        Web-->>Seller: Rehydrate saved state
    else Conflict or invalid target
        API->>DB: Roll back transaction
        API-->>Web: Error with current Product for configuration conflicts
        Web-->>Seller: Preserve intent and show recovery state
    end
```

The transaction is the configuration boundary, not the whole multi-section seller form. Projection follows the committed canonical state; it is not a substitute for current purchase-eligibility checks.
