# Seller Views Product Pricing In Seller Dashboard

This flow describes how seller dashboard reads show seller-authored base pricing.

```mermaid
sequenceDiagram
    actor Seller
    participant Web as Seller Dashboard
    participant API as Seller Product API
    participant Catalog as Catalog Pricing

    Seller->>Web: Open product pricing view
    Web->>API: Load seller product draft or detail
    API->>Catalog: Resolve canonical base pricing per inventory row
    Catalog-->>API: Base pricing snapshots
    API-->>Web: Product response with current base pricing
    Web-->>Seller: Display seller-authored pricing
```

Summary:

- seller dashboard currently exposes the base pricing snapshot used by the product draft summary
- market override rows may exist in persistence, but they are not surfaced through the normal seller pricing write/read flow today
