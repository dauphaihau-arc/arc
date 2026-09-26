# Arc Checkout

Arc Checkout determines whether a buyer may reserve and purchase a current seller offer.

## Language

**Purchase Eligibility**:
The requirement that a Product and any selected Product Variant are active, the Inventory Item is reservable through its default seller Stock Pool, and Available Quantity covers the requested amount.
_Avoid_: Catalog Visibility, Search Availability

**Delivery Option**:
A buyer-selectable delivery arrangement for a Fulfillment Group. It is not a warehouse, Fulfillment Method, or stock source.
_Avoid_: Shipping Method, Fulfillment Method, Warehouse

**Delivery Promise**:
The delivery window and Shipping Charge accepted by the buyer for a Fulfillment Group at checkout. Later estimates do not overwrite it or increase the charge.
_Avoid_: Estimate, Delivery Date

**Shipping Charge**:
The amount charged for delivering a Fulfillment Group. The buyer sees the combined Order shipping charge, and later partial Shipments do not increase it.
_Avoid_: Shipping Fee, Delivery Cost

## Relationships

- Checkout reserves the requested quantity from the Inventory Item's one default seller Stock Pool. There is no source selection, priority, or splitting.
- The quote fingerprint and quote reuse do not depend on a stock source, because each Inventory Item has exactly one.
- Confirmation preserves the confirmed fulfillment assignment; later Product configuration changes do not silently change confirmed Orders or alter historical purchase facts.
- Purchase Eligibility requires current Seller Catalog lifecycle state, a reservable default seller Stock Pool, and current Inventory state.
- The buyer chooses a Delivery Option, not a warehouse or stock source.
- Checkout establishes a Delivery Promise and Shipping Charge per Fulfillment Group and presents the combined Order shipping charge.
- Partial Shipments must respect the buyer's accepted Delivery Promise and cannot increase the agreed Shipping Charge.
- Confirmed fulfillment assignment and physical execution belong to Fulfillment, not Checkout.
- A projected or searchable Product is not necessarily eligible for purchase.
- An inactive or removed Product or Product Variant invalidates its unpaid quotes and causes their reservations to be released.
- A confirmed Order is not changed by later Product or Product Variant lifecycle transitions.
- An Inventory Shortage denies new reservations.
- During an Inventory Shortage, an existing reservation is consumed only when its complete quantity remains physically available; consumption is never partial.

## Flagged Ambiguities

- "Available in the catalog" means discoverable, not Purchase Eligibility.
- "Available" for Purchase Eligibility means the Inventory Item's default seller Stock Pool covers the quantity, not merely a non-zero aggregate On-hand Quantity.
- "Source" means the Inventory Item's default seller Stock Pool, not a warehouse or a Fulfillment Method.
- Buyer-chosen delivery options, per-shop delivery promises, and shop-wide Shipping Charges are agreed target contracts (tickets 08-09); current checkout pricing and delivery estimation do not yet implement them. Multi-location or provider stock sourcing is explicitly deferred and is not represented by runtime abstractions.
- "Reserved" does not guarantee purchase after a truthful physical count exposes an Inventory Shortage.
