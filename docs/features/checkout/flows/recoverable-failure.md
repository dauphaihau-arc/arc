# Recoverable Failure Flow

This flow describes checkout invalidation failures that can be recovered by refreshing cart data.

```mermaid
sequenceDiagram
    actor Buyer
    participant Web as Storefront Web
    participant Quote as Checkout Quote API
    participant Order as Order Creation API
    participant Cart as Cart Query

    Buyer->>Web: Complete order
    Web->>Quote: Create quote
    Quote-->>Web: quote_id
    Web->>Order: Create order with quote_id
    alt Quote expired
        Order-->>Web: CHECKOUT_QUOTE_EXPIRED
        Web->>Cart: Refresh cart data
        Web-->>Buyer: Show quote expired recovery copy
    else Cart changed
        Order-->>Web: CHECKOUT_QUOTE_CART_CHANGED
        Web->>Cart: Refresh cart data
        Web-->>Buyer: Show cart changed recovery copy
    else Stock unavailable
        Order-->>Web: CHECKOUT_QUOTE_RESERVATION_OUT_OF_STOCK
        Web->>Cart: Refresh cart data
        Web-->>Buyer: Show stock recovery copy
    else Reservation unavailable
        Order-->>Web: CHECKOUT_QUOTE_RESERVATION_UNAVAILABLE
        Web->>Cart: Refresh cart data
        Web-->>Buyer: Show reservation recovery copy
    end
```

Summary:

- recoverable checkout failures refresh cart data because the buyer needs the latest selected items, totals, and availability before retrying
- expired quotes show quote-expired recovery copy
- changed carts show cart-changed recovery copy
- stock and reservation failures show inventory-specific recovery copy
