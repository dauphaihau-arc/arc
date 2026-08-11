# Connection Authentication

This flow describes how the realtime gateway authenticates websocket connections.

```mermaid
sequenceDiagram
    participant Storefront
    participant Realtime as Realtime Gateway
    participant Session as Session Auth

    Storefront->>Realtime: Connect to /ws
    Realtime->>Session: Validate session cookie
    alt Session is missing, expired, or inactive
        Realtime-->>Storefront: Unauthorized error
        Realtime-xStorefront: Disconnect
    else Session is active
        Realtime-->>Storefront: Connected
    end
```

Summary:

- unauthenticated clients cannot keep a realtime connection open
- successful connections are associated with the authenticated user
- the user association is reused for later room subscription checks
