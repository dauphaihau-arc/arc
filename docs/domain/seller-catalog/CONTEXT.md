# Arc Seller Catalog

Arc Seller Catalog covers seller-managed Products, Product Variants, and their commercial lifecycle.

## Language

**Product Option**:
A seller-defined selectable dimension of a Product, such as Size or Color. Its name labels the dimension, not a purchasable combination.
_Avoid_: Variant Group, Variant Subgroup

**Product Option Value**:
A seller-defined choice belonging to one Product Option, such as M for Size or Red for Color. A value is not itself a Product Variant.
_Avoid_: Subvariant, Variant Value

**Product Variant**:
A purchasable option combination under a Product. After the Product has been published, the Product Variant keeps a stable identity even when its labels, SKU, or availability change.
_Avoid_: Variation

**Default Product Variant**:
The sole Product Variant of a Product with no Product Options. It has no option selections and represents the Product's purchasable offer.
_Avoid_: No-variant Inventory, Implicit Variant

**Product Variant Configuration**:
The complete intended set of Product Options, Product Option Values, and purchasable combinations under a Product, including their commercial lifecycle states.
_Avoid_: Variant Group Configuration, Option Matrix Patch

**Product Variant Structure Collapse**:
A change that removes one or more Product Option dimensions, including a change to a Default Product Variant. Superseded published combinations retain their historical identities; replacement offers do not implicitly inherit their physical quantities.
_Avoid_: Stock Merge, Automatic Consolidation

**Product Version**:
The monotonic revision used to reject seller mutations based on stale Product, pricing, shipping, or Product Variant lifecycle state.
_Avoid_: Updated Time, Last Modified

**Inactive Product**:
A previously published Product temporarily unavailable for purchase and reactivatable by its seller.
_Avoid_: Disabled Product, Archived Product

**Inactive Product Variant**:
A Product Variant temporarily unavailable for purchase while retaining its Inventory Item and remaining reactivatable by the seller.
_Avoid_: Out-of-stock Variant, Removed Variant

**Removed Product**:
A seller-deleted Product that cannot be reactivated by its seller or purchased, but remains retained for history and possible administrative recovery. Removal logically removes its Product Variants and Inventory Items and releases their SKUs.
_Avoid_: Deleted Product, Archived Product

**Removed Product Variant**:
A Product Variant no longer offered for new cart or checkout operations but retained to preserve historical identity. A seller may explicitly restore it using its original Product Variant and Inventory Item identities.
_Avoid_: Deleted Variant, Archived Variant

**Draft Product Variant Deletion**:
Physical deletion of an unreferenced Product Variant while its Product has never been published. The Product Variant has not yet acquired marketplace identity.
_Avoid_: Removed Variant

**Product Variant Rename**:
A correction to Product Variant labels that keeps the same Product Variant and Inventory Item identities. It does not mean the seller converted one physical stock pool into another.
_Avoid_: Variant Replacement, Stock Conversion

**Product Variant Structure Expansion**:
A controlled change that increases or reshapes the purchasable Product Variant matrix, such as adding a second option dimension to a previously single-option or no-option Product. For a published Product, expansion creates new Product Variant identities and removes superseded Product Variants instead of converting or regenerating historical identities.
_Avoid_: Create Variant, Replace Variants, Option Edit

**Administrative Product Recovery**:
The restoration of a Removed Product to the inactive state for seller review before republishing. Existing child lifecycle states remain unchanged, and conflicts must be resolved before publication.
_Avoid_: Product Reactivation, Automatic Republish

## Relationships

- A Product Variant has a stable identity after its Product is first published.
- A Product Option belongs to one Product; a Product Option Value belongs to one Product Option.
- A Product Variant selects one Product Option Value for each applicable Product Option.
- A Product with no Product Options has one Default Product Variant with no selections.
- A configuration change that supersedes an Inventory Item with active Checkout Reservations is blocked; reservations are not reassigned to replacement variants.
- A never-published Product may physically replace or delete unreferenced draft Product Variants.
- An Inactive Product or Inactive Product Variant may be reactivated by its seller.
- A Removed Product cannot be restored by its seller; Administrative Product Recovery returns only the Product as inactive.
- Removing a Product logically removes its Product Variants and Inventory Items.
- A Removed Product Variant retains its historical identity and requires explicit restoration.
- Renaming a Product Variant preserves its identity, Inventory Item, SKU history, reservations, and Order Item Snapshots.
- Product publication requires all child restoration and SKU conflicts to be resolved.
- Product Variant Structure Expansion is not Product Variant Rename; it creates new purchasable identities and preserves superseded published identities for history.
- Product Variant Structure Expansion may occur while a Product is Published or Inactive; current storefront visibility does not determine whether published identities must be preserved.
- Product Variant Structure Expansion creates the full intended purchasable matrix; hiding unwanted combinations is a separate Product Variant lifecycle change.

## Flagged Ambiguities

- "Inactive" means a reversible seller pause, not removal or an out-of-stock condition.
- "Delete Product" means create a Removed Product, not physically erase its identity.
- "Delete Variant" means Draft Product Variant Deletion only before first publication; after first publication it means a Removed Product Variant.
- "Rename Variant" means Product Variant Rename, not creating replacement physical stock.
- "Add Option" adds a Product Option dimension; "Add Option Value" adds a choice within an existing Product Option. Changes to the purchasable matrix are Product Variant Structure Expansion, not simple label edits.
- "Recover Product" means Administrative Product Recovery to inactive; it does not republish the Product or restore its children.
