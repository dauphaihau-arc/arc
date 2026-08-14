# Guest Buyer

This flow describes checkout when buyer identity comes from guest shipping and contact inputs.

```mermaid
sequenceDiagram
    actor Buyer
    participant Web as Storefront Web
    participant Quote as Checkout Quote API
    participant Order as Order Creation API
    participant GuestLookup as Guest Order Lookup

    Buyer->>Web: Enter shipping address and email
    Web->>Quote: Create quote with shipping address and presentment currency
    Quote->>Quote: Bind quote to guest checkout context
    Quote-->>Web: quote_id
    Web->>Order: Create order with quote_id, payment_type, and guest email
    Order-->>Web: Checkout session URL or order shops
    Web-->>Buyer: Success or payment redirect
    Buyer->>GuestLookup: Track order with session id or guest lookup details
```

Summary:

- guest checkout sends a full shipping address during quote creation
- guest checkout sends buyer email during order creation
- the quote is bound to guest checkout context
- guest checkout must preserve enough tracking context after order creation for the buyer to find the created order shops
