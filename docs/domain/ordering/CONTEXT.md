# Arc Ordering

Arc Ordering preserves the purchase-time facts and historical identities of confirmed Orders.

## Language

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

- Every confirmed Order Item retains one Order Item Snapshot.
- Historical display and reporting use the Order Item Snapshot rather than current Seller Catalog or Inventory values.
- An Order-safe Image Reference remains immutable for Historical Retention.
- Referenced catalog and inventory identities cannot be physically erased before Historical Retention ends and all blocking references are gone.
- Historical facts that cannot be reconstructed remain explicitly unknown rather than being copied from current catalog state.

## Flagged Ambiguities

- "Product details on an Order" means the Order Item Snapshot, not current Product details.
- "Order image" means an Order-safe Image Reference, not the Product's current image.
- "Backfill" does not permit representing current SKU or image values as purchase-time facts.
