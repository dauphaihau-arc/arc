# Message History Pagination

This flow describes how clients load the latest message window and page backward through older messages.

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
- REST message responses are the authoritative source for product-reference card display pricing
- conversation list pagination remains page-based
