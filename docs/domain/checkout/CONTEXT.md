# Arc Checkout

Arc Checkout determines whether a buyer may reserve and purchase a current seller offer.

## Language

**Purchase Eligibility**:
The requirement that a Product and any selected Product Variant are active, the Inventory Item is reservable, and Available Quantity covers the requested amount.
_Avoid_: Catalog Visibility, Search Availability

## Relationships

- Purchase Eligibility requires current Seller Catalog lifecycle state and current Inventory state.
- A projected or searchable Product is not necessarily eligible for purchase.
- An inactive or removed Product or Product Variant invalidates its unpaid quotes and causes their reservations to be released.
- A confirmed Order is not changed by later Product or Product Variant lifecycle transitions.
- An Inventory Shortage denies new reservations.
- During an Inventory Shortage, an existing reservation is consumed only when its complete quantity remains physically available; consumption is never partial.

## Flagged Ambiguities

- "Available in the catalog" means discoverable, not Purchase Eligibility.
- "Reserved" does not guarantee purchase after a truthful physical count exposes an Inventory Shortage.
