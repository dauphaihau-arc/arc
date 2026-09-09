# Arc Inventory

Arc Inventory covers stable seller stock identities, SKU ownership, physical quantities, reservations, and inventory history.

## Language

**Inventory Item**:
The seller-owned stock identity for one Product Variant, including a Default Product Variant when the Product has no Product Options. It remains stable across quantity and SKU changes.
_Avoid_: Inventory Row, Stock Record

**SKU**:
The seller-defined identifier of an Inventory Item. It is unique among non-removed Inventory Items in a shop and may be reused after removal.
_Avoid_: Product Code, Variant Code

**On-hand Quantity**:
The physical units recorded for an Inventory Item, including units currently reserved.
_Avoid_: Stock, Available Stock

**Reserved Quantity**:
The units of an Inventory Item held by active Checkout Reservations and unavailable to new buyers.
_Avoid_: Held Stock

**Available Quantity**:
The non-negative units of an Inventory Item eligible for a new Checkout Reservation after accounting for Reserved Quantity.
_Avoid_: Stock

**Inventory Shortage**:
The condition where Reserved Quantity exceeds On-hand Quantity. Available Quantity remains zero while the deficit is handled as an operational exception.
_Avoid_: Negative Stock

**On-hand Version**:
The monotonic revision of an Inventory Item's On-hand Quantity used to reject seller counts based on stale physical-count state. Reservation activity does not change it.
_Avoid_: Inventory Version, Updated Time, Last Modified

**Inventory Movement**:
An immutable record of a count, reservation, release, sale, or correction, including its cause, actor, command identity, and resulting quantities.
_Avoid_: Inventory Log, Stock Edit

**Published Variant Removal**:
Logical removal of the Inventory Item attached to a Product Variant after the Product has been published. The SKU is released for reuse, but the Inventory Item remains for reservations, movements, and Order history.
_Avoid_: Inventory Deletion

## Relationships

- An Inventory Item represents one Product Variant, including a Default Product Variant for a Product with no Product Options.
- An Inventory Item retains its identity across quantity and SKU changes.
- Renaming a Product Variant does not create a new Inventory Item or move On-hand Quantity.
- SKU is unique among non-removed Inventory Items in one shop.
- An Inventory Item may have no SKU; SKU uniqueness applies only to present SKUs on non-removed Inventory Items in one shop.
- Available Quantity is `max(0, On-hand Quantity - Reserved Quantity)`.
- Checkout reservation transitions change Reserved Quantity but not On-hand Quantity or On-hand Version.
- A seller count changes On-hand Quantity and On-hand Version.
- Reserved Quantity greater than On-hand Quantity creates an Inventory Shortage rather than a negative Available Quantity.
- Every count, reservation, release, sale, and correction creates an Inventory Movement.
- Never-published unreferenced draft Inventory Items may be physically deleted with their draft Product Variant.
- Product Variant Structure Expansion does not transfer or multiply On-hand Quantity; superseded Inventory Items retain their existing On-hand Quantity, and new Inventory Items start from seller-reviewed counts.
- Product Variant Structure Collapse does not sum or transfer On-hand Quantity; replacement Inventory Items require seller-reviewed counts.
- A configuration change cannot supersede an Inventory Item with active Checkout Reservations.
- Published Variant Removal releases the SKU; explicit restoration reuses the original Inventory Item identity and fails if another non-removed Inventory Item owns that SKU.
- Referenced Inventory Items are never hard-deleted while reservations, Inventory Movements, or Order Item Snapshots may still point at them.

## Flagged Ambiguities

- "Stock" is rejected because it can mean On-hand Quantity or Available Quantity.
- "Inventory Version" means On-hand Version for seller count conflicts, not reservation activity.
- "Negative stock" means Inventory Shortage; Available Quantity remains zero.
- "SKU validation" means rejecting duplicate present SKUs among non-removed Inventory Items; SKU is optional unless a specific workflow says otherwise.
- "Reset stock to zero" during Product Variant Structure Expansion means generated Inventory Items default to On-hand Quantity `0`; it does not zero or distribute the superseded Inventory Item's On-hand Quantity.
- "Rename variant" means correcting Product Variant labels and keeping the same Inventory Item, not converting physical stock.
