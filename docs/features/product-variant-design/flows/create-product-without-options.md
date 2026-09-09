# Create A Product Without Options

See the [design](../README.md) and [flow index](../flow.md).

1. Create a Product draft using the Product creation workflow.
2. The Product has no options and one Default Product Variant with no selections.
3. Its Inventory Item references that default variant, not the Product alone.
4. Enter inventory and price through the creation facade or configuration workflow.
5. Save and reload: the same default and inventory identities represent the offer.

Product Import creates this same no-option shape for each successful row. It does not import configurable matrices.
