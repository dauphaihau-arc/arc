# Message Creation And Delivery

This flow describes how REST message creation fans out through realtime delivery.

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
- product-reference events should cause clients to refetch active message history because one event payload cannot carry both buyer-presentment and seller-base product-card pricing
