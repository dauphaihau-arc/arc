# Authenticated Buyer

This flow describes checkout when buyer identity comes from the active user session.

```mermaid
sequenceDiagram
    actor Buyer
    participant Web as Storefront Web
    participant Quote as Checkout Quote API
    participant Order as Order Creation API

    Buyer->>Web: Choose saved address
    Web->>Quote: Create quote with user_address_id
    Quote->>Quote: Bind quote to authenticated user context
    Quote-->>Web: quote_id
    Web->>Order: Create order with quote_id and payment_type
    Order->>Quote: Validate user owns quote
    Order-->>Web: Checkout session URL or order shops
```

Summary:

- authenticated checkout uses the current user session for buyer identity
- the frontend sends address identity instead of a full shipping address
- the quote is bound to the authenticated user context
- order creation validates that the current user owns the quote
