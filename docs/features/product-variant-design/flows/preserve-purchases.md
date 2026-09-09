# Preserve Purchases

See the [design](../README.md) and [flow index](../flow.md).

Cart and storefront selection resolve the currently selected variant's Inventory Item. Checkout validates current eligibility and reserves that inventory identity.

A later seller rename changes current catalog labels, not purchase-time Order Item Snapshots. Structural replacement retains historical references; Orders must not be relinked to a newly generated combination. See [Checkout](../../checkout/README.md) for quote and reservation lifecycle behavior.
