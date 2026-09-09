# Product Variant Design

## Overview

Product Variant Configuration is the complete intended set of Product Options, Product Option Values, and purchasable combinations for one Product. Labels describe identities; they do not define them.

The normalized model is implemented by `Migration20260908100000` and the versioned seller configuration command. A Product without options has one Default Product Variant with zero selections and a linked Inventory Item. It is not a separate direct Product-to-Inventory case.

For the decision and rejected alternatives, see [Product option redesign and inventory identity](../../adr/0001-product-option-redesign.md). For workflows, see [flow.md](flow.md).

## Scope

- No-option, one-option, and two-option Products.
- Stable option/value identities and explicit variant selections.
- Complete matrices, including inactive combinations.
- Label corrections, value additions, structural expansion, and collapse.
- Atomic configuration, lifecycle, SKU, quantity, and base-price edits.
- Version conflicts, reservation protection, and idempotent request replay.
- Normalized seller/storefront reads, Product Import integration, and historical identity retention.

Current API limits are two options, up to 100 values per option, and at most 100 target variants. The complete matrix must fit the variant limit. The relational model does not encode two fixed label columns.

Out of scope: automatic stock transfer or consolidation, reservation reassignment, importing configurable matrices, and seller-authored market price overrides.

## Identity And Data Model

| Concept | Representation | Identity rule |
| --- | --- | --- |
| Product Option | `product_options` | Belongs to one Product; `name` and `position` are presentation data. |
| Product Option Value | `product_option_values` | Belongs to one option; `value` and `position` can change without replacing it. |
| Product Variant | Existing variant entity plus explicit selections | Selects exactly one value from each target option. |
| Variant selection | `product_variant_option_values` | Links the variant, option, and value with ownership constraints. |
| Inventory Item | Inventory linked by `product_variant_id` | Represents physical stock for one variant, including the default. |

A complete matrix contains the Cartesian product of the target option values. With no options, the matrix contains one zero-selection default variant. Inactive combinations remain in the matrix; omission means removal, not temporary unavailability.

Raw product detail responses expose `product_version`, `options`, `variants[].selections`, and inventory with `product_variant_id`. The legacy `variant_type`, group-name fields, and `option_value_1`/`option_value_2` are not authoritative API fields. Seller form adapters may derive a presentation mode and grouped rows from normalized data.

## Transformation Rules

- Correct labels for the same goods using existing option, value, variant, and inventory IDs. Reordering affects presentation, not physical identity.
- Preserve unchanged combinations when adding values. New combinations receive new identities and reviewed inventory.
- After first publication, an existing variant cannot be repurposed into a different selection combination. Expansion or collapse creates replacements and logically removes superseded identities. Making the Product inactive does not undo its publication history.
- New inventory requires explicit `on_hand_quantity`, `amount_minor`, and `currency`. Zero is a seller-entered count, not permission to infer stock from other rows.
- Never copy, distribute, sum, or multiply old quantities into replacements. Superseded inventory retains its history.
- A superseded Inventory Item with active Checkout Reservations blocks the configuration change. Reservations are never moved to replacement identities.
- `active` and `inactive` are target lifecycle states. Restoring a removed variant requires explicit acknowledgement and revalidation of its original identity and SKU ownership.
- SKU ownership remains shop-scoped among non-removed inventory. A SKU collision rejects the configuration rather than silently clearing another item's SKU.

## Configuration Command

`PUT /v1/shops/:shop_id/products/:product_id/variant-configuration`

Requires an authorized shop manager and an `Idempotency-Key` header. The body describes the complete target configuration, not a list of isolated row patches.

| Field | Meaning |
| --- | --- |
| `product_version` | Version of the Product the seller edited. |
| `options` | Target options and their values, including labels and one-based positions. |
| `variants` | Every target combination, its selections, lifecycle state, and optional inventory edits. |
| `removed_variant_ids` | Exact acknowledgement of the existing variants superseded or omitted by this target. Empty when nothing is removed. |
| `restore_variant_ids` | Explicit acknowledgement of removed variants being restored. |

Options, values, and variants use exactly one existing `id` or new `client_ref`. New references are request-wide unique. Selections use one `option_id` or `option_ref`, and one `value_id` or `value_ref`; references must resolve within the correct Product and option.

Inventory edits use `on_hand_quantity`, `expected_on_hand_version`, `sku`, `amount_minor`, and `currency`. Existing count edits require the expected On-hand Version. Omitted inventory fields leave the corresponding existing data unchanged; they are not resets. Use explicit `sku: null` to clear a SKU. Prices are minor-unit amounts, not display decimals.

The configuration transaction locks the Product and its inventory, validates identities and conflicts, and persists the target configuration with inventory, pricing, and associated mutation records. A successful configuration increments Product Version once. Failure rolls back the configuration; it must not leave a label edit committed while its inventory edit fails. Catalog projection is dispatched after the command succeeds.

This atomic boundary does not include the entire seller form: basic information, images, details, and configuration can be saved as separate sections. Nor does it make the whole Product creation facade one configuration transaction.

## Errors And Recovery

| Response | Meaning | Client action |
| --- | --- | --- |
| `400` | Invalid request shape or DTO bounds. | Correct the request before retrying. |
| `403` / `404` | Authorization or Product/shop scope failure. | Do not retry as a version conflict. |
| `409` | Product version, On-hand Version, SKU, reservation, or idempotency conflict. | Inspect the error code and resolve the conflict before resubmitting. |
| `422` | Invalid matrix, identity/reference relationship, or removal/restore acknowledgement. | Correct the complete target configuration. |

Configuration conflict responses include `current_product`; inventory-related conflicts also identify affected IDs, and SKU conflicts can include row-level conflict details. Do not automatically overwrite newer server state with an old matrix.

Retry an uncertain request with the same key and identical body. A successful replay returns the prior result without repeating effects. A revised intent is a new request with a new key and current versions.

## Seller UI And API Boundaries

The seller editor supports option/value editing, complete matrix generation, inventory editing, adding/removing dimensions, and removing all options. It carries stable IDs through label changes and requires replacement counts and prices rather than inheriting stock on structural changes.

Removing options and then adding them back before saving restores the unsaved option editor state. Hidden option-editor validation must not overwrite a selected default-variant configuration. Saved results are rehydrated from the returned detail response and survive reload.

The command supports explicit inactive combinations and restoration acknowledgements. That API capability does not imply the current editor exposes a dedicated control for every lifecycle operation. Restoration cannot be inferred simply by matching a label to a historical row.

## Migration And Rollout

Apply `Migration20260908100000` before running the normalized application against an existing database. The migration backfills option/value identities and selections, adds default variants for no-option Products, and removes legacy option columns.

Normalization preserves existing variant and inventory identifiers where present. Creating a missing default variant does not authorize replacement of the old Inventory Item. Ambiguous or malformed legacy combinations must fail migration rather than be guessed into a new matrix; repair the data deliberately and retry.

Verify the migration on a disposable copy before rollout. Do not reconstruct purchase-time labels, SKUs, or quantities from today's catalog when historical facts are unavailable.

## Verification Priorities

- Rename and reorder preserve variant and inventory identities.
- Expansion preserves unchanged combinations and does not multiply stock.
- Collapse creates a default with explicitly reviewed inventory and survives save/reload.
- Stale versions, duplicate SKUs, and reserved removals produce no partial writes.
- Exact idempotent replay produces no duplicate effects.
- Inactive rows remain part of the matrix but cannot be newly purchased.
- Existing Orders retain purchase-time facts after catalog edits.

## Related Documents

- [Seller Catalog vocabulary](../../domain/seller-catalog/CONTEXT.md)
- [Inventory vocabulary](../../domain/inventory/CONTEXT.md)
- [Ordering vocabulary](../../domain/ordering/CONTEXT.md)
- [Product Import](../import-product-design/README.md)
- [Multi-Currency Pricing](../multi-currency-pricing/README.md)
- [Checkout](../checkout/README.md)
