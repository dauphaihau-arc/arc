# Main Flow

See the [design](../README.md) and [flow index](../flow.md).

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
