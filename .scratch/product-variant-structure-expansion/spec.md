# Product Variant Structure Expansion

Status: ready-for-agent

## Problem Statement

Sellers can currently edit Product Variants through lower-level product update flows, but adding an option that changes the purchasable Product Variant matrix is not modeled as one seller-visible operation. A seller expects “add Size” to create reviewed purchasable choices with copied prices, generated or blank editable SKUs, and safe On-hand Quantity defaults. The current prototype direction can orchestrate multiple stable mutation primitives from the client, which risks partial live states and does not fully encode the Product Variant Structure Expansion domain rule.

Published Product Variants and Inventory Items may already be referenced by carts, Checkout Reservations, Orders, reviews, analytics, promotions, and Inventory Movements. The system must not destroy or regenerate those identities when a seller expands the matrix. It also must not copy old On-hand Quantity into every generated variant, because that would multiply physical inventory.

## Solution

Add Product Variant Structure Expansion as a controlled seller operation. The seller can expand a no-option Product into one or two option dimensions, add a second option dimension to a one-option Product, or add option values to an existing one/two-option Product. The UI previews the full generated matrix before save. Prices are copied from the relevant existing offer, SKUs are generated only when a clear base SKU exists and remain editable/optional, and generated On-hand Quantity defaults to `0` but can be edited before save.

Saving expansion calls one atomic backend command with the seller-reviewed full target matrix, expected Product Version, and idempotency key. The backend validates the matrix, preserves unchanged active Product Variant and Inventory Item identities, creates new identities for new combinations, logically removes superseded published identities, optionally physically replaces never-published unreferenced draft identities, persists initial values for new rows, emits one expansion event, increments Product Version once, and returns refreshed Product detail.

Published storefront cutover is atomic. Old purchasable choices disappear, new active choices appear immediately, and choices with zero Available Quantity display as unavailable/out of stock. Existing carts or Checkout Reservations that reference removed Product Variants become not checkoutable and require buyer reselect; completed Orders and historical facts remain stable.

## User Stories

1. As a seller, I want to add an option to a no-option Product, so that I can start selling explicit choices such as Size.
2. As a seller, I want to add two option dimensions to a no-option Product in one operation, so that I can create a complete matrix such as Color × Size without artificial intermediate saves.
3. As a seller, I want to add a second option dimension to a one-option Product, so that existing choices such as Color can become Color × Size.
4. As a seller, I want to add a new option value to an existing one-option Product, so that I can add a new purchasable choice without changing existing Product Variant identities.
5. As a seller, I want to add a new option value to an existing two-option Product, so that the missing combinations are created while current combinations remain stable.
6. As a seller, I want Product Variant Structure Expansion to work while the Product is Published, so that I do not have to hide or draft the Product before editing options.
7. As a seller, I want Product Variant Structure Expansion to work while an ever-published Product is Inactive, so that I can prepare its next matrix before reactivation.
8. As a seller, I want Removed Products to reject expansion, so that removed catalog identities cannot be edited through normal seller controls.
9. As a seller, I want expansion to preserve existing active Product Variant identities, so that existing references remain meaningful.
10. As a seller, I want expansion to preserve existing Inventory Item identities for unchanged active rows, so that SKU history, reservations, and Inventory Movements remain attached to the right stock identity.
11. As a seller, I want superseded published Product Variants to become Removed Product Variants, so that old identities remain available for history without staying purchasable.
12. As a seller, I want the old no-option offer identity retained after publication, so that order and cart references to the former no-option Product remain coherent.
13. As a seller, I want never-published unreferenced draft identities to be replaceable, so that draft setup stays simple before marketplace identity exists.
14. As a seller, I want referenced draft identities not to be hard-deleted, so that unexpected references do not become dangling records.
15. As a seller, I want adding Size to a no-option Product to generate rows `S`, `M`, and `L`, so that no fake `Default` option leaks into the seller or buyer experience.
16. As a buyer, I want historical no-option purchases to display without invented option labels, so that purchase history reflects what I actually bought.
17. As a seller, I want adding Size to a no-option Product to copy the existing price to every generated size, so that I do not have to re-enter a safe commercial default.
18. As a seller, I want adding Color × Size to a no-option Product to copy the existing price to every generated combination, so that the new matrix starts from the current offer price.
19. As a seller, I want adding a second option dimension to copy price from each parent existing option value, so that `Red` children inherit Red's price and `Blue` children inherit Blue's price.
20. As a seller, I want adding a value to an existing second dimension to copy price from the first existing sibling under the same unchanged dimensions, so that new sibling combinations get deterministic defaults.
21. As a seller, I want expansion to require price for every active generated Product Variant, so that no active offer has ambiguous commercial pricing.
22. As a seller, I want generated On-hand Quantity to default to `0`, so that old physical quantity is not multiplied across new variants.
23. As a seller, I want to edit generated On-hand Quantity before save, so that I can enter counts I already know while still saving one expansion.
24. As a seller, I want superseded Inventory Items to retain their existing On-hand Quantity, so that inventory history is not discarded or redistributed by catalog editing.
25. As a seller, I want the UI to warn that existing quantity is not copied, so that I understand why generated rows start with zero Available Quantity.
26. As an operator, I want expansion not to transfer quantity automatically, so that inventory accounting remains explicit through Inventory Movement behavior.
27. As a seller, I want generated SKUs to be editable, so that I can correct or blank suggested values before save.
28. As a seller, I want SKU to remain optional, so that expansion does not force SKU entry when my shop does not use SKUs.
29. As a seller, I want no-option SKU `SHIRT` expanded to Size to suggest `SHIRT-S`, `SHIRT-M`, and `SHIRT-L`, so that generated SKUs are useful when a clear base exists.
30. As a seller, I want no-option SKU `SHIRT` expanded to Color × Size to suggest values such as `SHIRT-RED-S`, so that generated SKUs are unique across the full matrix.
31. As a seller, I want one-option SKU `TS-RED` expanded with Size `S` to suggest `TS-RED-S`, so that existing parent SKU remains the base.
32. As a seller, I want generated SKUs to stay blank when the parent SKU is blank, so that the system does not invent opaque identifiers.
33. As a seller, I want adding values to an already-combined matrix to leave SKUs blank unless a clear base exists, so that heuristic suffix replacement does not create misleading SKUs.
34. As a seller, I want SKU suffixes generated with predictable sanitization, so that labels like `Extra Large` become readable SKU parts such as `EXTRA-LARGE`.
35. As a seller, I want generated SKU length overflow and collisions flagged before save, so that I can review exactly what will be persisted.
36. As a seller, I want duplicate present SKUs rejected among non-removed Inventory Items, so that my shop does not have ambiguous active identifiers.
37. As a seller, I want the same released SKU to be reusable by a new Inventory Item during expansion, so that a removed old identity does not permanently reserve a seller identifier.
38. As a seller, I want target rows to preserve seller-visible option order, so that generated matrix rows are predictable and not alphabetically rearranged.
39. As a seller, I want one-option to two-option expansion to preserve the existing option dimension order and append the new option, so that Color remains the first dimension when I add Size.
40. As a seller, I want option group names to be explicit, so that buyer-facing dimensions do not receive silent names such as `Option 1`.
41. As a seller, I want duplicate option group names rejected case-insensitively after trimming, so that buyers do not see two confusing dimensions with the same name.
42. As a seller, I want option values within one dimension to be duplicate-checked case-insensitively after trimming, so that accidental duplicates such as `M` and ` m ` are caught.
43. As a seller, I want display casing for option labels preserved, so that the storefront uses the labels I typed after validation.
44. As a seller, I want duplicate labels across different option dimensions allowed, so that group names disambiguate unusual but valid labels.
45. As a seller, I want the target matrix to be the complete Cartesian product of option dimensions, so that no combinations are silently missing.
46. As a seller, I want sparse or unwanted combinations handled by separate lifecycle editing, so that expansion remains one clear matrix operation.
47. As a seller, I want all generated rows to start Active, so that current lifecycle editing remains separate from matrix expansion.
48. As a seller, I want active zero-Available Quantity rows to remain visible as unavailable/out of stock, so that the product page communicates current availability honestly.
49. As a seller, I want zero Available Quantity not to block publication or reactivation, so that catalog readiness is not confused with inventory availability.
50. As a buyer, I want a published Product with all zero-available variants to remain visible but marked unavailable, so that I understand the Product exists but cannot be checked out now.
51. As a buyer, I want removed old variants to disappear from current storefront choices after expansion, so that I only choose current Product Variants.
52. As a buyer with an old cart line, I want the cart to show the item as no longer available, so that I know I must reselect current options.
53. As a buyer with an old cart line, I want the old captured or last-known price shown for context, so that the unavailable cart line remains understandable.
54. As a buyer, I do not want the system to auto-map my old no-option cart item to the first generated size, so that I am not assigned an option I did not choose.
55. As a checkout system, I want existing reservations on superseded Inventory Items to follow normal release, expiry, or consume rules, so that expansion does not abruptly mutate reservation accounting.
56. As a seller, I want expansion allowed even when the old Inventory Item has Reserved Quantity, so that buyer reservations do not block catalog editing.
57. As a seller, I want a warning when known Reserved Quantity exists on superseded choices, so that I understand some buyers may need to reselect.
58. As a seller, I want current storefront structure to remain unchanged while I preview expansion locally, so that unfinished edits do not leak live.
59. As a seller, I want storefront cutover only after successful save, so that buyers see either the old matrix or the complete new matrix, never a partial state.
60. As a seller, I want failed expansion validation or conflict to leave the Product unchanged, so that a rejected save does not alter a live listing.
61. As a seller, I want expansion save to be idempotent, so that double-clicks and network retries do not create duplicate Product Variants or events.
62. As a seller, I want reuse of the same idempotency key to return the same final detail, so that retry after timeout is safe.
63. As a seller, I want stale Product Version conflicts to include current Product detail and conflict reason, so that I can reapply the expansion intentionally.
64. As a seller, I want existing row value drift to be treated as a conflict, so that the saved result never differs silently from my preview.
65. As a seller, I want Product Version incremented once for expansion, so that one confirmed seller action is one catalog revision.
66. As an operator, I want one `product.variant_structure_expanded` event per accepted expansion, so that audit and projections understand the business action.
67. As an operator, I want the expansion event to include preserved anchors, created IDs, removed IDs, actor, idempotency key, Product Version, and seller-reviewed row values, so that before/after state is reconstructable.
68. As a developer, I want expansion to be a named use case, so that command validation, persistence, audit, and response behavior are not scattered across lower-level primitives.
69. As a developer, I want existing stable mutation primitives to remain useful internally or for narrower actions, so that expansion does not remove targeted variant lifecycle and inventory commands.
70. As a developer, I want the current client-orchestrated prototype treated as non-final for this edge case, so that implementation moves toward the accepted atomic command decision.
71. As a seller, I want option value rename separate from expansion, so that label correction does not accidentally become inventory conversion.
72. As a seller, I want option group rename separate from expansion, so that dimension naming changes do not mix with matrix changes.
73. As a seller, I want reducing option dimensions handled outside expansion, so that contraction gets its own deliberate rules.
74. As a seller, I want normal edit not to restore removed Product Variant identities automatically, so that historical identity reuse is explicit and reviewable.
75. As a seller, I want reusing a removed label in normal edit to create a new identity, so that historical removed variants do not silently reappear.
76. As a seller, I want inactive Product Variants excluded from normal expansion, so that hidden choices do not silently spawn new active combinations.
77. As a seller, I want active zero-available Product Variants included in future expansion, so that lifecycle state, not quantity, defines the offered matrix.
78. As a seller, I want expansion blocked when no active offered row exists, so that there is no unclear inactive or removed history to expand from.
79. As a seller, I want normal seller variant tables to hide removed superseded rows, so that current editing stays focused on current sellable structure.
80. As an administrator or operator, I want removed superseded rows available in history/admin/inventory surfaces later, so that investigations retain identity evidence.
81. As a promotions manager, I want variant-scoped promotions on removed old variants not to auto-copy to generated variants, so that unintended discounts are not created.
82. As a buyer, I want product-level promotion behavior to remain product-scoped after expansion, so that existing product-level rules continue through their own scope.
83. As an analytics user, I want product-level aggregates to remain on the Product after expansion, so that product reporting remains continuous.
84. As an analytics user, I want variant-specific history to stay attached to the removed old variant, so that data is not invented for new sizes.
85. As a developer, I want API responses after expansion to return refreshed Product detail with current Product Version, variants, inventory, prices, and lifecycle states, so that the seller UI can continue from authoritative state.

## Implementation Decisions

- **Domain operation**: Implement Product Variant Structure Expansion as a first-class seller command. It is not Product Variant Rename, Product Variant deletion, generic create variant, or complete replacement after publication.
- **Atomicity**: Expansion commits as one logical operation. It either succeeds completely or leaves Product state unchanged. It must not expose partially created variants, prices, lifecycle changes, or Inventory Items to the storefront.
- **Command coverage**: One canonical command handles no-option to one-option, no-option to two-option, one-option to two-option, adding option values to one-option Products, and adding option values to two-option Products.
- **Published identity rule**: Ever-published Products preserve published Product Variant and Inventory Item identities regardless of current visibility state. Current `Published` versus `Inactive` does not reopen destructive replacement.
- **Draft rule**: Never-published Products may physically replace or delete only unreferenced draft identities. If references exist, implementation must retain identities rather than hard-delete.
- **Removed Product rule**: Removed Products reject seller expansion.
- **No-option model**: A no-option Product has one implicit active offered row backed by an Inventory Item. When expanded after publication, that implicit offer identity is superseded and retained as removed history.
- **Target matrix contract**: The request carries option dimension names, option values, full target rows, expected Product Version, and idempotency key. External request and response fields follow the API snake_case contract; internal use-case inputs remain camelCase.
- **Full Cartesian validation**: Backend recalculates and validates that target rows exactly match the Cartesian product of option dimensions. Sparse matrices are out of scope for expansion.
- **Dimension limit**: Backend enforces the app limit of at most two option dimensions.
- **Dimension and value validation**: Each option dimension must have at least one non-empty value. Duplicate group names and duplicate values within a dimension are rejected using trim and case-insensitive comparison while preserving display casing.
- **Combination validation**: Duplicate option combinations are rejected. SKU differences do not make duplicate commercial choices valid.
- **Dimension order**: Existing dimension order is preserved. Adding a second dimension appends it after the existing one. No alphabetical backend normalization.
- **No rename mixing**: Existing option value rename and option group rename are separate operations. Expansion rejects target matrices that modify existing active labels while also expanding.
- **No contraction mixing**: Reducing option dimensions or shrinking the matrix is not Product Variant Structure Expansion and requires a separate controlled operation if implemented later.
- **Active anchor rule**: Expansion operates over active/current offered rows. Active rows with zero Available Quantity are included. Inactive and Removed Product Variants are not included and are not restored or expanded by this operation.
- **No-active-row rule**: Expansion is blocked when no active/current offered row exists.
- **Existing row anchors**: Existing rows included in the full target matrix anchor identity and current values. Their Product Variant ID, Inventory Item ID, price, SKU, On-hand Quantity, On-hand Version, lifecycle, and display labels must match current authoritative state or produce a conflict.
- **Existing row mutation rule**: Expansion does not update price, SKU, or On-hand Quantity for existing rows. Existing-row edits must use existing pricing, inventory, SKU, or Product Variant Rename commands.
- **New row persistence**: Expansion creates new Product Variant and Inventory Item identities for new combinations and persists seller-reviewed initial price, optional SKU, and On-hand Quantity.
- **New row lifecycle**: New generated Product Variants start Active. Hiding generated combinations is a separate Product Variant lifecycle operation.
- **Price rule**: Price is required for every active generated Product Variant. No active generated row may be saved with ambiguous price.
- **Price inheritance**: No-option expansion copies the old single offer price to every generated row. One-option to two-option expansion copies from the parent old option row. Adding a value to an existing second dimension copies from the first existing sibling under unchanged dimensions by seller-visible order.
- **Quantity rule**: Generated On-hand Quantity defaults to `0` but can be edited before save. Superseded Inventory Items retain their existing On-hand Quantity. Expansion never transfers, distributes, zeros, or multiplies old On-hand Quantity.
- **On-hand Version rule**: New Inventory Items start with On-hand Version `1` after creation/count initialization.
- **SKU optionality**: SKU is optional. Uniqueness applies only to present SKUs among non-removed Inventory Items in one shop.
- **SKU generation**: UI suggests SKUs only when a clear base SKU exists. No-option SKU `SHIRT` plus Size `S` suggests `SHIRT-S`; no-option SKU plus Color × Size suggests `SHIRT-RED-S`; one-option parent SKU plus new second dimension suggests `TS-RED-S`. Blank parent SKU yields blank generated SKUs.
- **No SKU template now**: Do not introduce seller-configurable SKU templates in this feature. For already-combined matrix value-adds where a base cannot be safely derived, generated SKU remains blank and editable.
- **SKU sanitization**: Generated suffixes are uppercase, trimmed, replace non-alphanumeric runs with `-`, collapse dashes, and remove leading/trailing dashes. Overflow is handled deterministically in UI and remains backend-validated; backend must not silently mutate submitted SKUs.
- **SKU conflict behavior**: Generated or submitted duplicate present SKUs are flagged in UI when detectable and rejected by backend if submitted. Backend must not silently auto-number or rewrite SKUs.
- **SKU reuse**: A SKU released by a superseded removed Inventory Item may be reused by one new non-removed Inventory Item during expansion if uniqueness holds after the atomic transition.
- **Idempotency**: Repeating the same expansion command identity returns the same final Product detail and does not create duplicate Product Variants, Inventory Items, Inventory Movements, or expansion events. Reusing the idempotency key with a different canonical payload returns conflict.
- **Conflict response**: Stale Product Version or existing-row drift returns a conflict with current Product detail, current Product Version, and reason/code for UI reapply.
- **Product Version**: A successful expansion increments Product Version once, regardless of the number of Product Variants created or removed.
- **Eventing**: Accepted expansion emits `product.variant_structure_expanded` with before/after matrix context, preserved anchor IDs, created IDs, removed IDs, actor, idempotency key, Product Version, and seller-reviewed row values.
- **Projection and storefront cutover**: Published Product cutover is atomic from the seller command perspective. Storefront sees the old matrix before success and the new complete matrix after success. Zero Available Quantity renders as unavailable/out of stock, not hidden.
- **Cart and Checkout behavior**: Removed superseded Product Variants are not eligible for new cart or checkout operations. Existing cart lines remain readable but not checkoutable and prompt buyer reselect. Active Checkout Reservations are not auto-cancelled or transferred by expansion.
- **Promotion behavior**: Variant-scoped promotions attached to superseded variants do not auto-copy to generated variants. Product-scoped promotions continue through product scope.
- **History behavior**: Product-level aggregates remain product-level. Variant-specific history remains attached to superseded removed variants and is excluded from current choice lists.
- **Response contract**: Successful expansion returns refreshed shop Product detail containing new Product Version, current active matrix, inventory facts, prices, and lifecycle states.
- **Current prototype relationship**: Existing stable commands for create variant, update variant lifecycle, set On-hand Quantity, and set pricing remain useful primitives, but frontend orchestration of those primitives is not the accepted final behavior for Product Variant Structure Expansion.

## Testing Decisions

- Good tests assert observable seller, buyer, API, domain, persistence, concurrency, idempotency, and projection behavior. They must not assert private helper call order, repository internals, field forwarding, or source text.
- The primary acceptance seam should be one high-level API/integration seam that runs the real application against isolated persistence. It should exercise the seller expansion command and observe refreshed Product detail, storefront/catalog visibility, cart/checkout rejection for removed choices, identity retention, Product Version behavior, idempotency, and events. This is the preferred single seam because expansion crosses Seller Catalog, Inventory, pricing, lifecycle, HTTP contracts, persistence, and purchase eligibility.
- Use Product application/use-case tests for business rules that are cheaper and sharper than full integration: matrix validation, published versus draft identity behavior, active anchor selection, conflict detection, one Product Version increment, and event payload shape.
- Use repository behavior tests where persistence rules matter: preserving unchanged identities, creating new Product Variant and Inventory Item identities, logically removing superseded published identities, physically replacing only unreferenced never-published draft identities, SKU uniqueness across the atomic transition, and no hard-delete of referenced identities.
- Use API/controller contract tests for request validation and response shape: snake_case external fields, optional SKU, required price, max two dimensions, duplicate detection errors, stale Product Version conflict body, idempotency replay, and refreshed Product detail response.
- Use seller web unit or Nuxt tests for preview behavior not visible from backend alone: immediate matrix generation, row ordering, price inheritance, On-hand Quantity defaulting/editing, SKU generation/sanitization/collision flags, confirmation copy, local-only preview before save, and section conflict display.
- Use cart/checkout application tests for buyer-facing effects: old cart line becomes unavailable after expansion, checkout cannot proceed against removed Product Variant, active zero-Available Quantity generated variants are visible but not purchasable, and existing reservations are not auto-transferred.
- Use projection/storefront tests only at the highest existing seam needed to prove post-save storefront output: old choices disappear, new choices appear, zero Available Quantity choices display unavailable/out of stock, and product list remains visible but unavailable when every generated row has zero Available Quantity.
- Prior art includes existing seller update-product submission tests, shop product controller tests, product use-case tests for variant and pricing behavior, product command repository tests, cart/checkout use-case tests, and existing storefront checkout/catalog tests. Tests that currently assert client-orchestrated stable command call order should be replaced with behavior tests once the atomic expansion command exists.
- Verification should run focused API, seller, storefront, and architecture checks for touched boundaries, then the root cross-area verification harness if implementation touches API and web together.

## Out of Scope

- Product Variant Structure Contraction, such as Color × Size → Color.
- Product Variant Rename or option group rename inside expansion.
- Existing row price, SKU, or On-hand Quantity edits inside expansion.
- Sparse matrices or hiding generated combinations inside the expansion preview.
- Seller-configurable SKU templates.
- Heuristic SKU derivation from already-combined SKUs.
- Automatic SKU renaming or backend auto-numbering.
- Forced SKU entry; SKU remains optional.
- Automatic quantity transfer, allocation, distribution, or zeroing of superseded Inventory Items.
- Inventory Movement transfer/correction tooling for moving old quantity into new variants.
- Automatic restoration of Removed Product Variants through normal product edit.
- Expansion of Inactive or Removed Product Variants.
- Editing Removed Products through seller controls.
- Auto-mapping old cart lines or reservations to generated variants.
- Auto-copying variant-scoped promotions to generated variants.
- Broad pricing model redesign, such as product-level base price with variant overrides.
- Switching Published Products back to Draft for option editing.
- Browser E2E coverage that duplicates cheaper deterministic seams unless existing test seams cannot prove the seller-visible behavior.
- Implementation work in this step; this spec records the agreed behavior only.

## Further Notes

This spec follows the Seller Catalog and Inventory domain glossary. Use **On-hand Quantity**, **Reserved Quantity**, and **Available Quantity** instead of overloaded “stock” in implementation-facing language. Seller-facing UI copy may use simpler phrasing, but must not imply old quantity is copied, distributed, or erased.

The accepted ADR is that Product Variant Structure Expansion uses an atomic command. Existing lower-level stable mutation primitives are still valuable for narrower edits, but they are not the final seam for this matrix-expansion edge case.

Recommended seller confirmation copy: “Adding this option will create new variants. Prices are copied from current variants. New variants start with 0 available quantity unless you enter quantities now. Generated SKUs are optional and editable. Existing product history is preserved. Existing buyers may need to reselect options before checkout.”
