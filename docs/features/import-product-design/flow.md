# Import Product Flow

## Main Flow

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
            Worker->>Catalog: Create draft product
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

## Progress Flow

```mermaid
sequenceDiagram
    participant Web as Seller Web
    participant API as Shop Product Import API
    participant Worker as Product Import Worker

    Web->>API: Subscribe to product import events
    Worker->>API: Publish queued or processing event
    API-->>Web: Progress payload
    Web->>Web: Update processed, created, failed, total, unprocessed

    alt Events continue
        Worker->>API: Publish completed or failed event
        API-->>Web: Terminal payload
        Web->>API: GET /shops/:shop_id/products/imports/:import_id
        API-->>Web: Final Product Import Job status
    else Events become stale
        Web->>API: GET /shops/:shop_id/products/imports/:import_id
        API-->>Web: Current Product Import Job status
    end
```

## Template Validation Failure

```mermaid
sequenceDiagram
    actor Seller
    participant Web as Seller Web
    participant API as Shop Product Import API

    Seller->>Web: Start import
    Web->>API: POST /shops/:shop_id/products/imports
    API->>API: Run Template Validation
    API-->>Web: 400 template error
    Web-->>Seller: Show upload error and retry path
```

Template Validation failures stop the import before a Product Import Job creates rows.

## Row Failure

```mermaid
sequenceDiagram
    participant Worker as Product Import Worker
    participant Catalog as Product Catalog
    participant Report as Import Report

    loop Each Import Row
        Worker->>Worker: Run Row Validation
        alt Row passes
            Worker->>Catalog: Create draft product
            Worker->>Report: Record created row
        else Row fails
            Worker->>Report: Record failed row with error code and message
        end
    end
```

Row Validation failures affect only the invalid Import Row. A Completed Import can still include failed rows.

## Retry Flow

```mermaid
sequenceDiagram
    participant Queue as Job Queue
    participant Worker as Product Import Worker
    participant Catalog as Product Catalog
    participant Report as Import Report

    Queue->>Worker: Retry Product Import Job
    Worker->>Report: Read existing row outcomes
    loop Each Import Row
        alt Row already created
            Worker->>Worker: Skip row
        else Row not created
            Worker->>Catalog: Attempt draft creation
            Worker->>Report: Record outcome
        end
    end
```

Worker retries must not recreate drafts for rows already recorded as created.
