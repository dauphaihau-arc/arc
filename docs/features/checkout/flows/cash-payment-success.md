# Cash Payment Success

This flow describes success-page routing after cash order creation.

```mermaid
sequenceDiagram
    participant Order as Order Creation API
    participant Web as Storefront Web
    actor Buyer

    Order-->>Web: Created order shops
    Web->>Web: Store order shops in checkout state
    alt Authenticated buyer
        Web-->>Buyer: Open /success
        Buyer->>Web: View authenticated orders
    else Guest buyer
        Web-->>Buyer: Open /success with guest email, ZIP, and order numbers
        Buyer->>Web: Track guest orders
    end
```

Summary:

- cash success currently depends on order shops returned during order creation
- the storefront stores returned order shops in checkout state before routing to success
- authenticated buyers can continue to their authenticated orders route
- guest routing carries lookup context in query parameters
