# Realtime Chat Flow

This document captures the user-visible and cross-surface flow for realtime chat delivery across storefront and seller chat surfaces.

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
- the websocket connection uses the same session boundary as authenticated API requests
- conversation subscription is authorized before a client can receive conversation-scoped events
- newly created messages are pushed to subscribed clients without waiting for polling

## Conversation Open And Product Reference

```mermaid
sequenceDiagram
    actor Buyer
    participant Storefront
    participant Chat as Chat API
    participant Delivery as Realtime Delivery
    participant Seller as Seller Inbox

    Buyer->>Storefront: Open chat from shop or product detail
    Storefront->>Chat: Create or get conversation
    alt Product id is provided and not already referenced
        Chat->>Chat: Store product_reference message with snapshot metadata
        Chat->>Chat: Update last_message and unread counts
        Chat-->>Delivery: Publish chat.message.created
        Delivery-->>Seller: Push product_reference message
    else Product is absent or already referenced
        Chat->>Chat: Reuse existing conversation without duplicate product reference
    end
    Chat-->>Storefront: Conversation summary
```

Summary:

- a buyer has one conversation per shop, even when opening chat from multiple products
- product context is preserved as a message with `message_type: product_reference`
- duplicate product reference messages are avoided for the same conversation and product
- the generated product reference increments the seller unread count and becomes the latest message

## Message History Pagination

```mermaid
sequenceDiagram
    participant Client as Storefront or Seller
    participant Chat as Chat API

    Client->>Chat: List messages with limit
    Chat-->>Client: Latest window in ascending display order
    alt Older messages exist
        Chat-->>Client: page_info.has_more_before=true and before_cursor
        Client->>Chat: List messages with before cursor
        Chat-->>Client: Older window in ascending display order
    else No older messages exist
        Chat-->>Client: page_info.has_more_before=false
    end
```

Summary:

- message history uses cursor pagination instead of page numbers
- `before` points to the oldest loaded message window boundary
- clients merge pages, deduplicate by message id, and keep display order ascending
- conversation list pagination remains page-based

## Connection Authentication

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

## Conversation Subscription

```mermaid
sequenceDiagram
    participant Storefront
    participant Realtime as Realtime Gateway
    participant Chat as Chat API

    Storefront->>Realtime: Subscribe to conversation
    Realtime->>Chat: Verify user can access conversation
    alt Buyer, seller, or admin access is valid
        Realtime-->>Storefront: Subscription accepted
    else Access is not valid
        Realtime-->>Storefront: Subscription forbidden
    end
```

Summary:

- knowing a conversation id is not enough to receive realtime messages
- subscription access is valid for buyers in the conversation, sellers who own the conversation shop, and admins
- forbidden subscriptions fail without joining the conversation delivery channel

## Conversation Unsubscribe

```mermaid
sequenceDiagram
    participant Storefront
    participant Realtime as Realtime Gateway

    Storefront->>Realtime: Unsubscribe from conversation
    Realtime-->>Storefront: Conversation delivery stopped
```

Summary:

- storefront unsubscribes when the active conversation changes or the chat surface unmounts
- leaving a conversation delivery channel does not require another permission check

## Message Creation And Delivery

```mermaid
sequenceDiagram
    actor Sender
    participant SenderWeb as Sender Client
    participant Chat as Chat API
    participant Delivery as Realtime Delivery
    participant RecipientWeb as Recipient Client

    Sender->>SenderWeb: Send message
    SenderWeb->>Chat: Create message
    Chat->>Chat: Store message, latest preview, read marker, and unread counts
    Chat-->>SenderWeb: Message created
    Chat-->>Delivery: Publish new message
    Delivery-->>SenderWeb: Push message event
    Delivery-->>RecipientWeb: Push message event
```

Summary:

- REST remains the authoritative write path for sending messages
- realtime delivery fans out the created message after the write succeeds, including `message_type` and metadata
- sender and recipient surfaces can receive the same created-message event
- clients patch active message lists, conversation previews, and unread badges until the next REST refetch

## Mark Conversation Read

```mermaid
sequenceDiagram
    participant Client as Storefront or Seller
    participant Chat as Chat API

    Client->>Chat: Mark conversation read
    Chat->>Chat: Update actor-side read timestamp and unread count
    Chat-->>Client: Updated conversation summary
```

Summary:

- storefront clears `buyer_unread_count` for the buyer side
- seller clears `seller_unread_count` for the seller side
- read state remains REST-authoritative; websocket events only move counts forward for new messages

## Cross-Instance Delivery

```mermaid
sequenceDiagram
    participant APIA as API Instance A
    participant Broker as Realtime Broker
    participant APIB as API Instance B
    participant Recipient as Recipient Storefront

    Recipient->>APIB: Connected and subscribed to conversation
    APIA->>APIA: Message is created
    APIA->>Broker: Publish conversation event
    Broker-->>APIB: Forward conversation event
    APIB-->>Recipient: Push new message event
```

Summary:

- realtime delivery must work when different users are connected to different API instances
- the broker carries room events across instances
- conversation delivery remains addressed by the logical conversation subscription, not by a specific server process
