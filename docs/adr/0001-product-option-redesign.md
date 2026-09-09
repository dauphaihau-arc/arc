---
status: accepted
---

# Product option redesign and inventory identity

Arc models Product Options and Product Option Values with stable identities and explicit Product Variant selections, rather than treating option labels as identities. Products without options have one default Product Variant with zero selections. Migration `Migration20260908100000` backfills normalized identities and removes the legacy option columns; application reads and seller mutations use the normalized configuration.

## Transformation policy

- Correcting labels for the same physical goods preserves Product Variant and Inventory Item identities; reordering changes presentation only.
- Published structural transformations preserve unchanged combinations, create new identities for new or replacement combinations, and logically remove superseded identities. A single target combination does not implicitly authorize identity reuse.
- Expansion and collapse never automatically transfer, sum, or multiply On-hand Quantity. New Inventory Items require seller-reviewed counts; superseded items retain their history and quantities.
- Transformations that supersede Inventory Items with active Checkout Reservations are blocked initially. Additions and edits that leave those reserved identities intact are not blocked by this rule. Reservations are never silently reassigned.
- The intended option matrix remains complete. Unwanted combinations are represented through explicit Product Variant lifecycle state, not omitted selections.
- Supported mixed option, value, and inventory edits are validated as one intended result and committed atomically. Stale state is rejected rather than silently overwritten; retries must not duplicate effects.
- Historical Order references and purchase-time snapshots remain intact. SKU ownership follows existing non-removed Inventory Item uniqueness rules.

## Consequences

The default-variant model requires backfilling no-option Products and migrating affected inventory, checkout, ordering, import, and projection contracts. Existing Product Variant and Inventory Item identifiers must not be regenerated merely to normalize the schema; migration of a no-option Product adds a default variant while retaining its existing Inventory Item identity. Structural editing after cutover follows the transformation policy above.

Seller configuration mutations use `PUT /v1/shops/:shop_id/products/:product_id/variant-configuration` with a Product Version and an Idempotency-Key. Options, selections, lifecycle changes, and reviewed inventory edits are validated and persisted as one transaction. The seller editor preserves identities on label corrections and requires explicit replacement inventory when collapsing options. No migration may silently move reservations or reinterpret historical purchases.

## Considered alternatives

Keeping two label columns is cheaper but leaves identity dependent on labels and limits the option model. Retaining direct Product inventory for no-option Products avoids a backfill but perpetuates separate mutation paths. Automatically choosing the first surviving row or summing stock during collapse is rejected because option order does not establish physical stock equivalence.
