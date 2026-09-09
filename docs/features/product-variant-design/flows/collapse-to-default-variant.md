# Collapse To A Default Variant

See the [design](../README.md) and [flow index](../flow.md).

1. Choose Remove options in the seller editor.
2. The editor shows default inventory inputs without selecting an old row or summing its stock.
3. Enter the default price, On-hand Quantity, and optional SKU.
4. Submit `options: []`, one new zero-selection variant, and every superseded variant ID in `removed_variant_ids`.
5. The server rejects the operation if a superseded item has active reservations or any other validation fails.
6. On success, the new default inventory contains only the explicitly entered count. Historical inventory is not reassigned to it.
7. Reload and confirm one default variant and the reviewed inventory values.

Before saving, choosing Add options again restores the unsaved option configuration. Validation from the hidden option editor must not change the current default-variant intent during submission.

Collapsing two dimensions to one follows the same identity and reviewed-inventory rules for affected published combinations; it is not an automatic stock merge.
