# Conversation Open And Product Reference

This flow describes how storefront opens a buyer-shop chat conversation and preserves product context as a generated message.

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
- storefront reads render product-card money in buyer presentment currency, while seller reads render seller-authored base catalog pricing
