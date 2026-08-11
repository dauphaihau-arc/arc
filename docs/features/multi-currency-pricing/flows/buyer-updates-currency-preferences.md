# Buyer Updates Storefront Currency Preferences

This flow describes how buyer region and currency preferences change active storefront pricing context.

```mermaid
sequenceDiagram
    actor Buyer
    participant Storefront
    participant Dialog as Preferences Dialog
    participant User as Current User API
    participant Market as Market Store
    participant Query as Storefront Query Cache
    participant ProductAPI as Storefront Product API
    participant Pricing as Price Resolution

    Buyer->>Storefront: Open market preferences control
    Storefront->>Dialog: Show current region and currency preferences
    Dialog->>Dialog: Load enabled markets and supported currencies
    Buyer->>Dialog: Select region or currency and save
    opt Buyer is authenticated
        Dialog->>User: Persist updated preferences
        User-->>Dialog: Return updated user preferences
    end
    Dialog->>Market: Store active guest preferences locally
    Dialog->>Query: Invalidate active market-sensitive queries
    Query->>ProductAPI: Refetch products, product detail, cart, and checkout session
    ProductAPI->>Pricing: Resolve prices with updated market/currency context
    Pricing-->>ProductAPI: Updated display pricing
    ProductAPI-->>Storefront: Product and commerce responses in selected currency
    Storefront-->>Buyer: Product prices update across active storefront views
```

Summary:

- buyer can change region and currency from the storefront preferences dialog
- available currencies are constrained by the selected enabled market
- authenticated buyers persist preferences through the current user API
- guest preferences are stored locally and become the active storefront market context
- active market-sensitive storefront queries are invalidated after save
- refetched products, product detail, cart, and checkout session data resolve prices with the selected currency
- product display prices update across active storefront views after the refetch completes
