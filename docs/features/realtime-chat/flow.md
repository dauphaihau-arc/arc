# Realtime Chat Flow

This document captures the end-to-end user-visible and cross-surface flow for realtime chat delivery across storefront and seller chat surfaces.

For feature scope, event shape, and implementation file references, see [README.md](README.md).

## Main Flow

```mermaid
sequenceDiagram
    actor User
    participant Storefront
    participant Realtime as Realtime Gateway
    participant Chat as Chat API
    participant Delivery as Realtime Delivery
    participant Seller as Seller Inbox

    User->>Storefront: Open or create a chat conversation
    Storefront->>Chat: Create or get buyer-shop conversation
    alt Product context is provided
        Chat->>Chat: Create product_reference message if needed
        Chat-->>Delivery: Publish product reference message
    end
    Chat-->>Storefront: Conversation summary
    Storefront->>Realtime: Connect to /ws with session cookie
    alt Session is invalid
        Realtime-->>Storefront: Unauthorized connection error
        Realtime-xStorefront: Disconnect
    else Session is valid
        Realtime-->>Storefront: Connected
    end

    Storefront->>Realtime: Subscribe to conversation
    Realtime->>Chat: Check conversation access
    alt User cannot access conversation
        Realtime-->>Storefront: Subscription forbidden
    else User can access conversation
        Realtime-->>Storefront: Subscription accepted
    end

    User->>Storefront: Send message
    Storefront->>Chat: Create message
    Chat-->>Storefront: Message created
    Chat-->>Delivery: New chat message is available
    Delivery-->>Storefront: Push new message event
    Delivery-->>Seller: Push new message event
```

Summary:

- storefront and seller surfaces use REST for authoritative state and websocket delivery for realtime updates
- storefront opens a buyer-shop conversation through REST before subscribing to its realtime room
- product context is sent as a generated `product_reference` message, not as a conversation-level product field
- product-reference snapshot metadata is stored as fallback context; REST message reads provide viewer-specific product-card pricing
- the websocket connection uses the same session boundary as authenticated API requests
- conversation subscription is authorized before a client can receive conversation-scoped events
- newly created messages are pushed to subscribed clients without waiting for polling

## Detailed Flows

- [Conversation Open And Product Reference](flows/conversation-open.md)
- [Message History Pagination](flows/message-history.md)
- [Connection Authentication](flows/connection-auth.md)
- [Conversation Subscription](flows/conversation-subscription.md)
- [Conversation Unsubscribe](flows/conversation-unsubscribe.md)
- [Message Creation And Delivery](flows/message-delivery.md)
- [Mark Conversation Read](flows/mark-read.md)
- [Cross-Instance Delivery](flows/cross-instance-delivery.md)
