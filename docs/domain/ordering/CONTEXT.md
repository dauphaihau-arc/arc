# Arc Ordering

Arc Ordering preserves the purchase-time facts and historical identities of confirmed Orders.

## Language

**Order**:
A confirmed purchase of one or more Order Items from one shop. It remains one Order regardless of how its quantities are fulfilled.
_Avoid_: Purchase, Transaction

**Order Item**:
One Product Variant and quantity within an Order. Its identity is stable, and its quantity may be divided across Fulfillment Groups and Shipments without duplicating the obligation.
_Avoid_: Line Item, Order Line

**Order-safe Image Reference**:
An immutable image reference retained by an Order Item Snapshot for the required historical retention period.
_Avoid_: Current Product Image

**Order Item Snapshot**:
The purchase-time title, SKU, variant labels, image reference, price, discount, and currency retained for an Order Item. Historical display and reporting use this snapshot rather than current catalog values.
_Avoid_: Current Product Details

**Historical Retention**:
The configured period during which referenced Products, Product Variants, Inventory Items, Order Item Snapshots, and Order-safe Image References cannot be physically erased. Retention is indefinite until a policy is configured, and blocking legal, financial, refund, review, or dispute references always extend it.
_Avoid_: Removal Period, Archive Window

## Relationships

- One Order belongs to one shop and contains one or more Order Items.
- Every confirmed Order Item retains one Order Item Snapshot.
- A shop Order may be fulfilled through multiple Fulfillment Groups and Shipments without fragmenting the Order or changing its commercial identity.
- An Order Item's quantity may be split across Fulfillment Groups and Shipments without changing the Order Item's identity or duplicating its obligation.
- Shipment quantities never exceed an Order Item's assigned, uncanceled quantity.
- The fulfillment assignment selected at confirmation, the accepted Delivery Promise, and the Shipping Charge are confirmed purchase facts; later Product configuration changes, estimates, or reassignments do not silently reroute or overwrite them.
- Canceled quantities remain visible as canceled and are never counted as delivered; an Order whose units are all canceled is canceled, not fulfilled.
- Dispatch history, Shipment Updates, and recorded exceptions are retained; a replacement does not reset previously dispatched quantities.
- Commercial Order completion remains a separate policy: it is currently automatic on an order-level delivered update, and quantity-based Fulfillment Progress does not itself redefine payment, refund, or commercial completion.
- Fulfillment assignment and dispatch do not change confirmed purchase facts or Order Item Snapshots.
- Historical display and reporting use the Order Item Snapshot rather than current Seller Catalog or Inventory values.
- An Order-safe Image Reference remains immutable for Historical Retention.
- Referenced catalog and inventory identities cannot be physically erased before Historical Retention ends and all blocking references are gone.
- Historical facts that cannot be reconstructed remain explicitly unknown rather than being copied from current catalog state.

## Flagged Ambiguities

- "Product details on an Order" means the Order Item Snapshot, not current Product details.
- "Order image" means an Order-safe Image Reference, not the Product's current image.
- "Backfill" does not permit representing current SKU or image values as purchase-time facts.
