# Export Order Flow

## Main Flow

```mermaid
sequenceDiagram
    actor Seller
    participant Web as Seller Web
    participant API as Shop Order Export API
    participant Worker as Export Worker
    participant Storage as Private Storage
    participant Notifications

    Seller->>Web: Open seller orders
    Seller->>Web: Open export dialog
    Web->>Web: Capture filters, date range, timezone, and columns
    Web->>API: Start export
    API->>API: Authorize shop management
    alt Small export
        API->>API: Generate CSV
        API-->>Web: CSV download response
        Web-->>Seller: Download CSV
    else Larger export
        API->>API: Create export job snapshot
        API-->>Web: Export queued
        API-->>Worker: Process export job
        Worker->>API: Publish queued or processing progress
        Worker->>Storage: Store completed CSV
        Worker->>Notifications: Create completion notification
        Worker->>API: Publish completed progress
        Web->>API: Download completed export
        API->>Storage: Read private CSV
        API-->>Web: CSV download response
        Web-->>Seller: Download CSV
    end
```

## Start Export

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

## Progress Flow

```mermaid
sequenceDiagram
    participant Web as Seller Web
    participant API as Shop Order Export API
    participant Worker as Export Worker

    Web->>API: Subscribe to user events
    Web->>API: Poll export status
    Worker->>API: Publish queued or processing progress
    API-->>Web: Progress payload
    Web->>Web: Update percentage and row count

    alt Export completes
        Worker->>API: Publish completed event
        API-->>Web: Completed payload
        Web->>API: Download export
    else Export fails
        Worker->>API: Publish failed event
        API-->>Web: Failed payload
        Web-->>Web: Show failure state
    else Events are stale
        Web->>API: GET /shops/:shop_id/orders/exports/:export_id
        API-->>Web: Current export status
    end
```

The seller app uses events and polling together so progress can continue when one delivery path is unavailable.

## Asynchronous Processing

```mermaid
sequenceDiagram
    participant Worker as Export Worker
    participant Orders as Order Data
    participant Storage as Private Storage
    participant API as Shop Order Export API
    participant Notifications

    Worker->>API: Mark export processing
    Worker->>API: Publish initial progress
    loop Export row batches
        Worker->>Orders: Read seller-visible order rows
        Orders-->>Worker: Matching rows
        Worker->>Worker: Append CSV rows
        Worker->>API: Publish progress
    end
    Worker->>Storage: Store completed CSV
    Worker->>API: Mark export completed
    Worker->>Notifications: Create completion notification
    Worker->>API: Publish completed progress
```

Asynchronous processing restores the stored filter and column snapshot instead of reading the seller's current browser state.

## Download Flow

```mermaid
sequenceDiagram
    actor Seller
    participant Web as Seller Web
    participant API as Shop Order Export API
    participant Storage as Private Storage

    Seller->>Web: Download completed export
    Web->>API: GET /shops/:shop_id/orders/exports/:export_id/download
    API->>API: Recheck shop access
    API->>API: Validate export belongs to shop and is completed
    API->>Storage: Read private CSV
    Storage-->>API: CSV file
    API-->>Web: CSV attachment
    Web-->>Seller: Browser downloads CSV
```

The completed in-app notification stores enough routing data for the seller to download the same export later.

## Failure Flow

```mermaid
sequenceDiagram
    participant Worker as Export Worker
    participant API as Shop Order Export API
    participant Web as Seller Web

    Worker->>Worker: Export processing fails
    Worker->>API: Mark export failed
    Worker->>API: Publish failed progress
    API-->>Web: Failed payload or failed poll response
    Web-->>Web: Show failure state and close active progress connection
```

Temporary files are removed whether export processing succeeds or fails.
