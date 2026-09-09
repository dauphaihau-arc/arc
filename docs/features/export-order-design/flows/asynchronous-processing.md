# Asynchronous Processing

See the [design](../README.md) and [flow index](../flow.md).

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
