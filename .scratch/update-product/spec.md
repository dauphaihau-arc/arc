# Stable Product Mutation, Inventory, and Removal

Status: ready-for-agent

## Problem Statement

Sellers currently edit a Product through several independent requests, but the UI presents them as one save and reports only a generic failure. A successful early request can remain persisted while a later request fails, leaving the seller unable to tell which section changed or how to recover.

Published Product Variants and Inventory Items are currently replaced as complete collections. Replacement changes their identities and can violate references held by carts, Checkout Reservations, and Orders. Seller-entered `stock` is also treated as Available Quantity even though sellers mean a physical On-hand Quantity. Reservation activity and seller counts therefore compete over one value without optimistic concurrency.

Product lifecycle changes are authoritative in Seller Catalog but their catalog projection and Checkout effects are not durably coupled to the source transaction. A removed offer can remain discoverable during projection lag, and an existing cart or unpaid quote can reach a purchase path without one authoritative Purchase Eligibility decision. Order history also lacks the complete purchase-time SKU, variant, image, and commercial facts needed to survive later catalog changes.

## Solution

Give Seller Catalog and Inventory explicit ownership and stable identities. Seller Catalog owns Products, Product Variants, pricing, shipping, and commercial lifecycle. Inventory owns Inventory Items, SKUs, On-hand Quantity, Reserved Quantity, Available Quantity, reservations, and Inventory Movements. NestJS authorizes seller activity and accesses either the local or remote Inventory implementation through one application contract.

Replace published-collection updates with targeted, versioned, idempotent commands. Retain full replacement only for Products that have never been published. Separate the seller edit screen into independently saved sections, show section-specific success and failure, retain unsaved input, and make stale changes an explicit reapply flow.

Make lifecycle changes durable with transactional outbox events. Enforce Purchase Eligibility synchronously at cart mutation, quote creation, reservation, and final Order creation so projection or cleanup lag cannot authorize a purchase. Preserve complete purchase-time Order Item Snapshots and retain referenced identities and images according to Historical Retention.

Cut over the API, seller UI, Inventory implementations, and persisted data together. Reconcile active reservations in a maintenance window, convert current available `stock` to On-hand Quantity, and remove obsolete destructive mutation paths rather than maintaining compatibility wrappers.

## User Stories

1. As a seller, I want the quantity editor to represent physical On-hand Quantity, so that the number I enter matches the units I can count.
2. As a seller, I want to see On-hand Quantity, Reserved Quantity, and Available Quantity separately, so that active Checkout Reservations are not mistaken for missing stock.
3. As a seller, I want a truthful physical count accepted even when it reveals an Inventory Shortage, so that the system does not force me to enter false inventory.
4. As a seller, I want an Inventory Shortage shown explicitly, so that I can investigate the deficit without seeing negative Available Quantity.
5. As a seller, I want concurrent physical counts detected, so that one browser session cannot silently overwrite another seller's accepted count.
6. As a seller, I want reservation activity not to create false physical-count conflicts, so that normal buyer traffic does not make inventory editing unusable.
7. As a seller, I want a stale count response to include the current quantities and On-hand Version, so that I can decide whether to reapply my intended count.
8. As a seller, I want successful retry of the same inventory command to return its original result, so that a lost network response does not create a duplicate change.
9. As a seller, I want reuse of an idempotency key with different input rejected, so that an old command identity cannot apply a new mutation accidentally.
10. As a seller, I want every count and correction recorded as an Inventory Movement, so that I can trace why inventory changed.
11. As an operator, I want reservations, releases, sales, and corrections recorded as Inventory Movements, so that balance investigations have one immutable history.
12. As an operator, I want Inventory Movements to include cause, actor or system identity, command identity, and before-and-after quantities, so that changes are auditable.
13. As a seller, I want an optional note on a physical correction, so that unusual counts can carry business context without requiring notes for routine work.
14. As a seller, I want each published Product Variant to keep its identity when I edit labels, pricing, SKU, or availability, so that historical and active references remain valid.
15. As a seller, I want each published Inventory Item to keep its identity when I change SKU or quantities, so that carts, reservations, and Orders keep referencing the same stock identity.
16. As a seller, I want to replace a complete Product Variant configuration while the Product has never been published, so that draft setup remains simple.
17. As a seller, I want destructive collection replacement rejected after first publication, so that a temporarily inactive Product cannot lose published identities.
18. As a seller, I want an active Product Variant to be made inactive without removal, so that I can pause it and reactivate it later.
19. As a seller, I want to remove a Product Variant without physically erasing it, so that history retains the original Product Variant and Inventory Item identities.
20. As a seller, I want to restore a Removed Product Variant explicitly, so that accidental removal is recoverable without inventing a new identity.
21. As a seller, I want a restored Product Variant to remain non-purchasable until explicitly activated, so that I can review its SKU and commercial configuration safely.
22. As a seller, I want Product Variant restoration rejected when its former SKU has been reused, so that two non-removed Inventory Items cannot own one shop SKU.
23. As a seller, I want SKU uniqueness enforced only among non-removed Inventory Items in my shop, so that a legitimately retired SKU can be reused.
24. As a seller, I want Product inactivation to be reversible, so that I can pause an offer without deleting its identity or configuration.
25. As a seller, I want Product removal to be irreversible through seller controls, so that removal is clearly distinct from a temporary pause.
26. As a seller, I want removing a Product to logically remove its Product Variants and Inventory Items, so that no child offer remains available independently.
27. As a seller, I want removing a Product to release its child SKUs after Inventory processes the lifecycle change, so that retired identifiers can be reused.
28. As an administrator, I want to recover a Removed Product as inactive, so that exceptional recovery does not automatically republish it.
29. As an administrator, I want Product recovery to preserve removed child states, so that recovery cannot silently resurrect Product Variants or Inventory Items.
30. As a seller, I want each recovered child restored explicitly, so that SKU conflicts and current configuration are reviewed before publication.
31. As a seller, I want one Product Version across metadata, pricing, shipping, and Product Variant lifecycle, so that concurrent edits cannot silently overwrite accepted catalog changes.
32. As a seller, I want stale Product changes rejected with current state and Product Version, so that I can reapply my intent knowingly.
33. As a seller, I want retried lifecycle commands to return their original results, so that network ambiguity does not repeat a removal or activation.
34. As a seller, I want bulk lifecycle retries to preserve the original per-Product outcome, so that retrying a partially successful request is safe.
35. As a seller, I want Product details, pricing, shipping, Product Variants, media, attributes, and inventory presented as independently saved sections, so that the interface matches real transaction boundaries.
36. As a seller, I want a successful section to remain visibly saved if another section fails, so that I do not repeat accepted changes.
37. As a seller, I want a failed section identified precisely, so that I know what still needs attention.
38. As a seller, I want unsaved input retained after a section failure, so that a transient error does not erase my work.
39. As a seller, I want conflict details shown in the affected section, so that I can compare my edit with current authoritative state.
40. As a seller, I want the interface to stop claiming the complete Product form is one atomic save, so that partial success is not hidden behind a generic error.
41. As a buyer, I want cart additions and quantity updates rejected for an inactive or removed Product, so that I cannot start purchasing an unavailable offer.
42. As a buyer, I want cart additions and quantity updates rejected for an inactive or removed Product Variant, so that stale catalog data cannot add an unavailable option.
43. As a buyer, I want quote creation to check current Seller Catalog and Inventory state, so that search projection lag cannot authorize my purchase.
44. As a buyer, I want reservation creation to require sufficient Available Quantity, so that the system does not intentionally oversell.
45. As a buyer, I want final Order creation to recheck Purchase Eligibility, so that lifecycle changes after quote creation are respected.
46. As a buyer, I want an unpaid quote invalidated when its Product or Product Variant becomes inactive or removed, so that payment cannot complete against a withdrawn offer.
47. As a buyer, I want the reservation for an invalid unpaid quote released, so that unavailable checkout attempts do not hold inventory indefinitely.
48. As a buyer, I want a clear eligibility failure when an offer changed, so that I can return to the cart instead of receiving an unrelated payment or server error.
49. As a buyer with an existing reservation during an Inventory Shortage, I want my complete requested quantity either consumed or rejected, so that I never receive an accidental partial Order.
50. As a buyer, I want a confirmed Order unaffected by later Product or Product Variant removal, so that completed purchase history remains stable.
51. As a buyer, I want my Order to retain the purchase-time Product title and variant labels, so that later seller edits do not rewrite what I bought.
52. As a buyer, I want my Order to retain the purchase-time SKU, so that support and fulfillment can identify the sold Inventory Item accurately.
53. As a buyer, I want my Order to retain purchase-time price, discount, and currency, so that historical totals do not depend on current pricing.
54. As a buyer, I want my Order to retain an Order-safe Image Reference, so that its visual history survives Product image changes.
55. As a support agent, I want historical Order views and reports to use Order Item Snapshots, so that current catalog data cannot alter purchase evidence.
56. As a support agent, I want unreconstructable legacy SKU or image values shown as unknown, so that current values are not misrepresented as purchase-time facts.
57. As a compliance operator, I want referenced catalog identities and Order-safe Image References retained while legal, financial, refund, review, or dispute obligations exist, so that required history is not erased.
58. As an operator, I want retention to default to indefinite until a policy is configured, so that missing configuration cannot cause destructive cleanup.
59. As an operator, I want lifecycle state and its outbox event committed together, so that catalog projection and cleanup work cannot be lost after a successful source mutation.
60. As an operator, I want lifecycle consumers to be idempotent and retryable, so that duplicate or delayed delivery converges on the authoritative state.
61. As an operator, I want catalog and search projections removed or restored from durable lifecycle events, so that discovery eventually matches Seller Catalog.
62. As an operator, I want synchronous Purchase Eligibility independent of projection completion, so that consumer lag cannot make an unavailable offer purchasable.
63. As an operator, I want local and remote Inventory implementations to follow the same command semantics, so that deployment configuration does not change business behavior.
64. As an operator, I want active Checkout Reservations reconciled before inventory migration, so that the new On-hand and Reserved quantities preserve real commitments.
65. As an operator, I want current available `stock` converted to `on_hand = stock + reserved`, so that the migration does not understate physical inventory.
66. As an operator, I want inventory writes paused during reconciliation and conversion, so that concurrent mutations cannot corrupt the migration baseline.
67. As an operator, I want migration invariants verified before writers resume, so that cutover failures are detected before accepting new commerce traffic.
68. As an operator, I want a declared rollback point before migration starts, so that a failed cutover can return to a coherent prior state.
69. As a developer, I want obsolete replacement and hard-delete client paths removed during cutover, so that the codebase has one mutation convention.
70. As a developer, I want Seller Catalog to reference Inventory Items through an application contract instead of sharing ownership of persistence records, so that each capability has one writer.

## Implementation Decisions

- **Capability ownership**: Seller Catalog owns Products, Product Variants, metadata, pricing, shipping, attributes, media association, and commercial lifecycle. Inventory owns Inventory Item identity, SKU lifecycle and uniqueness, quantity balances, reservations, and Inventory Movements. Catalog code must not directly replace live Inventory persistence.
- **Inventory port**: NestJS authenticates and authorizes seller commands, then calls one Inventory application contract. Both the local NestJS implementation and remote Go implementation must implement the same requests, conflicts, idempotency, lifecycle, and result shapes.
- **Stable identities**: Persist an immutable indication that a Product has been published at least once. Complete Product Variant and Inventory Item replacement is allowed only before that point. Current `DRAFT` or `INACTIVE` state alone does not reopen replacement after prior publication.
- **Product lifecycle**: Preserve Product states for draft, active, inactive, and removed behavior. Inactive is seller-reversible. Removed is a retained seller-irreversible tombstone with removal time; there is no seller hard-delete contract.
- **Product Variant lifecycle**: Add active, inactive, and removed lifecycle state plus removal metadata. Published Product Variants are mutated by stable ID. Restore reuses the original Product Variant and Inventory Item identities and returns the child to a non-purchasable state until explicit activation.
- **Product removal cascade**: A Product removal commits the Product state and lifecycle outbox message in one Seller Catalog transaction. The lifecycle consumer logically removes child Product Variants and asks Inventory idempotently to remove their Inventory Items and release their SKUs. Source removal succeeds independently of consumer availability; final Purchase Eligibility blocks immediately from authoritative Product state.
- **Administrative recovery**: Administrative Product Recovery changes only the Removed Product to inactive. It does not restore, activate, or republish children. Each child is restored explicitly, and publication remains blocked until required children and conflicts are resolved.
- **Child restoration**: Inventory validates restoration of the original Inventory Item and former SKU. A SKU conflict rejects restoration without automatic renaming. Retry-safe orchestration must remain non-purchasable after any partial infrastructure failure and converge when retried.
- **SKU constraint**: SKU is scoped to a shop and unique among non-removed Inventory Items. Removal releases uniqueness; restoration reacquires the former SKU or returns a conflict.
- **Quantity model**: Replace the overloaded `stock` balance with On-hand Quantity and Reserved Quantity. Available Quantity is derived as the non-negative `max(0, on_hand - reserved)` and is not independently writable.
- **Inventory Shortage**: A seller may set On-hand Quantity below Reserved Quantity. The command succeeds, Available Quantity becomes zero, and the Inventory Item enters an observable shortage condition. Existing reservations are not silently canceled.
- **Reservation transitions**: Reserving increases Reserved Quantity. Releasing decreases Reserved Quantity. Successful sale consumption atomically decreases both Reserved Quantity and On-hand Quantity for the complete reservation quantity. A consumption that cannot be physically satisfied fails without partial consumption or negative On-hand Quantity. No FIFO ordering guarantee is introduced.
- **On-hand concurrency**: Each Inventory Item has an On-hand Version changed only by accepted On-hand Quantity mutations. Seller count commands supply the expected On-hand Version. Reservation, release, and consumption activity uses its own monotonic balance or event sequence and does not cause seller count conflicts.
- **Product concurrency**: Each Product has one Product Version guarding metadata, pricing, shipping, and Product Variant lifecycle mutations. Every accepted guarded mutation increments it once. A stale expected version returns HTTP conflict with the current Product Version and enough current section state for explicit reapplication; the server does not auto-merge.
- **Idempotency**: Seller lifecycle and inventory mutations require an idempotency key. The key is scoped to the authenticated actor, command kind, and target. Repeating the same key with the same canonical payload returns the original status and body without another state transition or Inventory Movement. Reusing the key with a different payload returns a conflict. A bulk lifecycle request treats the complete ordered target/version/action payload as the idempotent command and replays its original per-item result.
- **Inventory Movement ledger**: Append an immutable Inventory Movement for each accepted physical count, reservation, release, sale, and correction. Record movement kind, Inventory Item, quantity delta, before-and-after On-hand and Reserved quantities, cause, actor or system identity, command or event identity, timestamp, and optional seller note. Retries cannot append duplicates.
- **Seller command boundaries**: Provide independently transactional commands for Product details, pricing, shipping, Product Variant lifecycle/configuration, media, attributes, and Inventory counts. Do not add a distributed transaction over the complete edit form.
- **Seller HTTP contracts**: Return the resulting section state and current relevant version from mutation responses. Return domain-specific validation, not-found, lifecycle, SKU, shortage, stale-version, and idempotency conflicts through the existing layered error model. Remove the obsolete seller Product hard-delete client contract rather than aliasing it.
- **Draft replacement contract**: Existing complete variant/inventory configuration may remain only for never-published Products. Published Products use targeted create/update/inactivate/remove/restore commands. No compatibility wrapper may translate a published replacement request into destructive persistence.
- **Bulk lifecycle behavior**: Preserve per-Product success and failure reporting, but route every publish, deactivate, and remove result through the same version checks, audit behavior, and transactional lifecycle outbox used by single-Product commands.
- **Seller UI workflow**: Replace the chained generic save with section-level dirty, pending, success, conflict, and error state. Save only changed sections. Preserve successful server results, refresh relevant versions, retain failed section input, and present a deliberate compare-and-reapply action after a conflict.
- **Lifecycle outbox**: Product and Product Variant publication, activation, inactivation, removal, and restoration persist a durable outbox event in the same transaction as source lifecycle state. Consumers are idempotent and update catalog/search projections plus reservation cleanup and Inventory lifecycle effects as applicable.
- **Projection semantics**: Catalog and search are discovery projections. They eventually add, update, or remove documents from durable lifecycle events, including bulk mutations, but their presence never grants Purchase Eligibility.
- **Purchase Eligibility service**: Provide one authoritative application seam that evaluates current Seller Catalog lifecycle and Inventory state. A purchase requires an active Product, an active selected Product Variant when present, a reservable non-removed Inventory Item, and sufficient Available Quantity for the complete request.
- **Purchase gates**: Apply the same Purchase Eligibility policy at cart add/update, quote creation, reservation, and immediately before final Order creation. A stale cart or projected Product cannot bypass the policy.
- **Quote invalidation**: Inactivation or removal of a Product or Product Variant makes affected unpaid quotes invalid. Durable lifecycle handling releases their reservations idempotently. Final Order creation rejects the quote synchronously even if asynchronous release has not completed. Confirmed Orders are unchanged.
- **Order Item Snapshot**: At confirmed Order creation, persist title, SKU, selected variant labels, Order-safe Image Reference, unit price, discount allocation, currency, and existing required commercial totals. Order history and reporting read this snapshot instead of mutable Seller Catalog or Inventory state.
- **Order-safe images**: The snapshot must refer to immutable or retention-protected image content. Replacing or removing current Product media cannot break the Order view during Historical Retention.
- **Legacy Order history**: Migration may backfill only facts supported by historical evidence. Unknown SKU or image facts remain nullable or explicitly unknown. Current catalog values must not be labeled as purchase-time values.
- **Historical Retention**: Retained Products, Product Variants, Inventory Items, Order Item Snapshots, and Order-safe Image References are not physically erased until a configured policy permits it and no Order, legal, financial, refund, review, dispute, or other blocking reference remains. Until policy configuration exists, retention is indefinite. Automated physical purge is not introduced by this cutover.
- **Data migration**: Use a bounded maintenance window. Pause every seller and checkout inventory writer, identify the configured authoritative Inventory implementation, reconcile active reservations, set Reserved Quantity from those reservations, and initialize On-hand Quantity as existing available `stock + reserved`. Initialize versions and lifecycle fields deterministically, verify invariants, then switch all readers and writers before traffic resumes.
- **Migration validation**: Prove each active reservation maps to an Inventory Item, Available Quantity is non-negative, explicit shortages are identified, stable IDs and referenced foreign keys remain intact, SKU uniqueness holds among non-removed items, and Order snapshots are honest. Establish a database and Inventory rollback point before mutation.
- **Clean cutover**: Deploy schema, application contracts, local and remote Inventory behavior, Seller Catalog commands, Checkout gates, Ordering snapshots, lifecycle consumers, and seller UI as one coordinated compatibility boundary. Remove old `stock` writes, published full-replacement behavior, legacy client calls, obsolete aliases, and tests that assert destructive replacement semantics.

## Testing Decisions

- Tests assert observable domain and transport behavior, not repository call order, private helpers, field forwarding, or source text. A good test fails if stable identity, lifecycle, concurrency, idempotency, balance, Purchase Eligibility, history, or seller recovery behavior regresses.
- The primary acceptance seam is the existing commerce HTTP integration suite running the real Nest application against isolated PostgreSQL. Extend that seam to configure and publish a Product, mutate it through seller contracts, operate a cart/quote/Order, and inspect externally visible results. This is the highest existing seam that crosses Seller Catalog, local Inventory, Checkout, Ordering, persistence, transport mapping, and inline outbox execution.
- At the primary seam, prove that a published Product Variant and Inventory Item keep their IDs across targeted edits; published replacement is rejected; inactive and removed offers fail cart, quote, and Order attempts; confirmed Order history remains unchanged; and bulk lifecycle mutations update eventual projections through the outbox path.
- At the primary seam, prove Product Version and On-hand Version conflicts return current state, a same-key retry returns the original response without duplicate effects, and a changed payload under the same key is rejected.
- At the primary seam, prove Product removal is a retained tombstone, child offers become non-purchasable, SKU ownership is released after lifecycle consumption, Administrative Product Recovery returns only the Product as inactive, and child restoration detects SKU reuse.
- At the primary seam, prove an Order Item Snapshot retains purchase-time title, SKU, variant labels, image reference, price, discount, and currency after subsequent Seller Catalog edits or removal.
- Inventory requires one reusable black-box contract suite executed against both configured implementations. The contract covers reserve, validate, release, consume, seller count, On-hand Version conflict, idempotency conflict, SKU lifecycle, Inventory Movement deduplication, shortage creation, non-negative Available Quantity, and atomic no-partial consumption. Existing local checkout reservation service tests and Go reservation service tests are the prior art.
- Use focused concurrent database tests for races that sequential HTTP tests cannot prove: two seller counts with one expected On-hand Version, competing full-reservation consumptions during shortage, duplicate command delivery, and SKU restoration racing with SKU reuse. Assert one coherent outcome and persisted balances rather than lock or SQL implementation details.
- Use focused Seller Catalog application tests for state-transition rules, never-published replacement eligibility, Product Version conflicts, child restoration prerequisites, and transactionally persisted lifecycle outbox records. Existing publish, bulk mutation, and catalog projector use-case tests are the prior art.
- Use focused Checkout application tests for the shared Purchase Eligibility policy at cart, quote, reservation, and final Order creation. Exercise stale projected data explicitly and assert the authoritative rejection plus reservation release behavior. Existing cart-item, checkout-quote, checkout reservation, and create-order use-case tests are the prior art.
- Use seller Vitest/Nuxt tests for the behavior not visible from API integration: independently saved sections, retained unsaved input after failure, persistent success state for earlier sections, specific failed-section messaging, version refresh, and compare-and-reapply conflict flow. Existing update-product submission tests are prior art; tests asserting replacement call order must be replaced because that behavior is intentionally removed.
- Add migration verification against a production-shaped isolated database fixture containing available stock, active and expired reservations, published and inactive Products, existing Orders, reused candidate SKUs, and incomplete historical snapshots. Prove conversion arithmetic, identity retention, shortage classification, honest null history, rerun safety where supported, and rollback readiness.
- Do not add broad browser coverage merely to duplicate the commerce integration suite. Add a seller browser journey only if section persistence and conflict recovery cannot be demonstrated through the existing Nuxt test seam. Existing storefront checkout Playwright coverage remains focused on payment-critical browser behavior.
- Run focused API, seller, storefront, Inventory, and architecture verification for their touched boundaries, then run the root cross-area verification harness because the cutover spans all areas.

## Out of Scope

- A distributed all-or-nothing transaction for the complete seller Product form.
- Compatibility wrappers, deprecated aliases, or dual writes for the old published variant/inventory replacement contracts.
- Automatic merging of concurrent seller edits.
- Automatic SKU renaming during restoration.
- Automatic child restoration or republishing during Administrative Product Recovery.
- Partial reservation consumption or a FIFO reservation scheduler during Inventory Shortage.
- Intentional negative On-hand Quantity or Available Quantity.
- Treating catalog/search projection presence as purchase authorization.
- Rewriting confirmed Orders after later Product or Product Variant lifecycle changes.
- Fabricating purchase-time SKU, image, or pricing facts from current catalog state.
- Choosing a legal/business retention duration. Retention remains indefinite until a separate policy is configured.
- Automated physical purge of retained catalog, inventory, order, or image records.
- Product Import update-by-SKU behavior; Product Imports continues to create draft Products only.
- New notification, analytics, or forecasting features beyond the required Inventory Movement and existing audit behavior.

## Further Notes

- This specification implements the confirmed model in ADR-005 through ADR-011 and uses the Seller Catalog, Inventory, Checkout, and Ordering glossaries.
- Current `stock` represents sellable/available quantity because reservations decrement it and releases restore it. The migration formula must therefore be `on_hand = stock + reserved`; renaming the field without reconciliation would corrupt inventory meaning.
- Lifecycle projection is intentionally eventually consistent. Safety comes from synchronous Purchase Eligibility against authoritative Seller Catalog and Inventory state, not from waiting for search or catalog document cleanup.
- Inventory lifecycle effects caused by Product removal may complete after the Seller Catalog source transaction. Until the idempotent consumer releases a SKU, reuse may temporarily report a conflict; this lag cannot make the removed offer purchasable.
- The idempotency retention duration is an operational configuration, but it must cover the maximum supported client retry window and must be identical in behavior for local and remote Inventory implementations.
- Migration planning must include the selected Inventory driver, a complete writer inventory, a traffic-pause mechanism, invariant queries, observability for outbox backlog, and an explicit rollback decision point before irreversible cleanup.
