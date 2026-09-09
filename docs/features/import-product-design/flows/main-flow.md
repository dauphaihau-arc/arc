# Main Flow

See the [design](../README.md) and [flow index](../flow.md).

```mermaid
sequenceDiagram
    actor Seller
    participant Web as Seller Web
    participant API as Shop Product Import API
    participant Storage as Private Storage
    participant Queue as Job Queue
    participant Worker as Product Import Worker
    participant Catalog as Product Catalog

    Seller->>Web: Open Products > Import Products
    Seller->>Web: Download Import Template
    Web->>API: GET /shops/:shop_id/products/imports/template
    API-->>Web: XLSX Import Template

    Seller->>Web: Select filled XLSX
    Web->>Web: Build Import Preview
    Web-->>Seller: Show advisory validation for first 10 preview rows

    Seller->>Web: Start import
    Web->>API: POST /shops/:shop_id/products/imports
    Note over Web,API: Original XLSX multipart file + idempotency key
    API->>API: Authorize shop management
    API->>API: Run Template Validation
    API->>Storage: Store Import Source File
    API->>Queue: Enqueue Product Import Job
    API-->>Web: Product Import Job queued

    Queue->>Worker: Process Product Import Job
    Worker->>Storage: Read Import Source File
    Worker->>API: Publish progress events
    loop Each Import Row
        Worker->>Worker: Run Row Validation
        alt Row is valid
            Worker->>Catalog: Create no-option draft with default variant and inventory
            Worker->>Worker: Record created outcome
        else Row is invalid
            Worker->>Worker: Record failed outcome
        end
        Worker->>API: Publish progress event
    end
    Worker->>Storage: Store Import Report CSV
    Worker->>API: Mark job completed
    API-->>Web: Completion event or status response
    Web-->>Seller: Show result counts

    Seller->>Web: Download report
    Web->>API: GET /shops/:shop_id/products/imports/:import_id/report
    API->>Storage: Read Import Report
    API-->>Web: CSV attachment
```

Each successful row creates one Default Product Variant with zero selections and one Inventory Item containing the row's reviewed price, count, and optional SKU. The normalized model does not add option-matrix columns to the XLSX template. See [Product Variant Design](../../product-variant-design/README.md).
