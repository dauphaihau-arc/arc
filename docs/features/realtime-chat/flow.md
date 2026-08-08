# Realtime Chat Flow

This document captures the user-visible and cross-surface flow for realtime chat delivery.

For feature scope, event shape, and implementation file references, see [README.md](README.md).

## Main Flow

```mermaid
sequenceDiagram
    actor User
    participant Storefront
    participant Realtime as Realtime Gateway
    participant Chat as Chat API
    participant Delivery as Realtime Delivery

    User->>Storefront: Open a chat conversation
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
```

Summary:

- the storefront uses REST for message creation and websocket delivery for realtime updates
- the websocket connection uses the same session boundary as authenticated API requests
- conversation subscription is authorized before a client can receive conversation-scoped events
- newly created messages are pushed to subscribed clients without waiting for polling

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
    participant SenderWeb as Sender Storefront
    participant Chat as Chat API
    participant Delivery as Realtime Delivery
    participant RecipientWeb as Recipient Storefront

    Sender->>SenderWeb: Send message
    SenderWeb->>Chat: Create message
    Chat->>Chat: Store message and update conversation summary
    Chat-->>SenderWeb: Message created
    Chat-->>Delivery: Publish new message
    Delivery-->>SenderWeb: Push message event
    Delivery-->>RecipientWeb: Push message event
```

Summary:

- REST remains the authoritative write path for sending messages
- realtime delivery fans out the created message after the write succeeds
- both sender and recipient surfaces can receive the same created-message event

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
