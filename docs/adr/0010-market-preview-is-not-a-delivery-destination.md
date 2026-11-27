---
status: accepted
---

# A Market Preview Destination is commercial context, not a delivery destination

Each enabled Market declares exactly one validated ISO alpha-2 `preview_country_code`. The storefront uses that country as the Market's Preview Destination: the country Product-detail Shipping Preview prices against, the default destination for catalog coverage and availability badges, and the country assumed by Cart pre-checkout availability, suggestions, recommendations, and rail filtering. Because one prominent country now travels through every browsing surface, it is tempting to let it stand in for the buyer's shipping address. That would make a commercial browsing decision authoritative over a delivery fact the buyer has not supplied yet.

This decision separates the two. A Preview Destination describes the Market a buyer is browsing. The shipping address entered at Checkout describes where an Order is actually delivered. Only the latter may decide whether a selected line can be purchased and what shipping it is charged.

## Decision

A Market's Preview Destination is commercial and browse context, not a delivery destination.

- **Each enabled Market declares exactly one preview country.** Every enabled Market requires a validated ISO alpha-2 `preview_country_code`, exposed through marketplace configuration. Shipping interprets that server-owned Market configuration as the Preview Destination; it is not a per-request, per-buyer, or per-Product value.
- **The Preview Destination is the default commercial lens.** It drives the Product-detail Shipping Preview, default catalog coverage and availability badges, Cart pre-checkout availability, and Product suggestions, recommendations, and rail filtering.
- **Shipping Coverage is derived from the assigned Shipping Profile.** A Product is covered for a country when that country is named by one of the Profile's Destination Rates, or when the Profile carries `everywhere_else` (worldwide). Digital Products bypass physical shipping and are exempt from physical coverage. Coverage is projected into catalog documents and verified and backfilled from the Profile rather than authored separately.
- **Shipping Preview is an estimate for the Preview Destination, not an acceptance.** It reports live availability, the reason when unavailable, a converted one-item fee, and a delivery estimate. The unavailable reasons are `missing_assignment`, `profile_not_found`, `profile_not_ready`, `unsupported_destination`, and `unsupported_currency`.
- **Browse may override the Preview Destination, and only browse.** Catalog requests may pass `catalog_ship_to=<ISO>` to evaluate coverage and badges for a different country, or `catalog_ship_to=all` to browse without a destination filter. The override never changes Product-detail Shipping Preview, Cart availability, or Checkout.
- **Add to Cart warns instead of blocking.** A buyer may add a line even when the Preview Destination is unsupported, because the real delivery address may still be covered. The warning tells the buyer the estimate is unavailable for the Market's Preview Destination; it does not reject the selection.
- **Cart preserves buyer selections.** An unsupported selected line stays in the Cart. The Cart reports the unsupported quantity as `unavailable_quantity`, excludes those lines from money totals, and blocks progression to Checkout while they remain selected. It does not silently deselect or delete them.
- **Checkout is authoritative from the real shipping address.** The shipping address entered at Checkout is the sole delivery authority for quoting shipping, applying coverage, and snapshotting the Order. The Preview Destination is never substituted for it. If any selected line is unsupported for the real address, the whole quote is rejected with `CHECKOUT_SHIPPING_UNAVAILABLE`; Checkout never quotes or confirms a partial selection.
- **Accepted shipping facts are immutable.** Quotes and Orders snapshot the accepted shipping facts. Later changes to a Market's Preview Destination, a Shipping Profile, Shipping Coverage, or the catalog do not rewrite a quote that was accepted or an Order that was confirmed.
- **`Destination Rate` is the canonical term.** There are no zones. Coverage and fees are expressed through Destination Rates plus `everywhere_else`.

## Considered alternatives

- **Treat the Preview Destination as the buyer's address.** Using the Market's country as if it were a delivery destination would price and filter against a fact the buyer never supplied. It would also make the browsing Market decide purchasability for a buyer whose real address is elsewhere, and would have to be undone at Checkout when the real address arrives.
- **Silently deselect unsupported Cart lines.** Removing lines the buyer explicitly selected hides a conflict instead of surfacing it, and the buyer may reach payment believing the whole Cart is being purchased. Keeping the lines with an explicit `unavailable_quantity` and a checkout block preserves intent and makes the remedy visible.
- **Partial Checkout.** Quoting and confirming only the supported lines would silently drop the rest, split an Order the buyer did not ask to split, and leave money and fulfillment records that disagree with the selection the buyer saw. Rejecting the whole quote with `CHECKOUT_SHIPPING_UNAVAILABLE` keeps one selection and one quote.
- **Block Add to Cart when the Preview Destination is unsupported.** This confuses a browse-time estimate with purchase eligibility. A buyer browsing an unsupported Market may hold a supported real address, so blocking the selection prevents a legitimate purchase. A warning preserves the signal without taking the decision away from the buyer.
- **Use the Preview Destination for Checkout shipping.** Quoting, coverage checks, and Order snapshots would then describe delivery to the Market's country rather than the buyer's address. That misprices shipping, can reject a purchasable address, and records a delivery destination the buyer never chose.
- **Rewrite accepted quotes and Orders when coverage changes.** Retroactively recomputing a quote or Order would alter an agreed price and delivery promise, invalidate financial records, and make a confirmed Order disagree with what the buyer accepted. Accepted facts are snapshotted and left immutable; new coverage applies only to new quotes and Orders.

## Consequences

- A Market cannot be enabled without a valid ISO alpha-2 `preview_country_code`; an invalid or missing value is a configuration error rather than a fallback to a currency, locale, or shop country.
- The marketplace config response exposes the Preview Destination as the single server-owned country for every browser surface, so buyer-facing applications do not invent a per-view or per-Product destination.
- Product-detail Shipping Preview renders availability and reason, a converted one-item fee, and a delivery estimate for the Preview Destination, and shows an explicit unavailable state for the enumerated reasons.
- Catalog documents carry projected Shipping Coverage, and default catalog coverage and availability badges evaluate that coverage against the Preview Destination. The projection is verified and backfilled from assigned Shipping Profiles.
- Digital Products do not claim or require physical Shipping Coverage and are not filtered or badged as physically uncovered.
- Catalog browsing can override the country used for coverage and badges with `catalog_ship_to=<ISO>`, or remove the coverage filter with `catalog_ship_to=all`; that override has no effect on Product-detail Shipping Preview, Cart, or Checkout.
- Add to Cart succeeds with a warning when the Preview Destination is unsupported; the warning is not a purchase block.
- The Cart keeps unsupported selected lines, reports `unavailable_quantity`, excludes their money from Cart totals, and blocks Checkout until the buyer changes the selection or the lines become supported.
- Checkout quotes and confirms from the real shipping address. Any unsupported selected line rejects the entire quote with `CHECKOUT_SHIPPING_UNAVAILABLE`; a supported real address remains purchasable even when the Market's Preview Destination is unsupported.
- Checkout shows an informational note when the real address country differs from the Preview Destination, so a difference between the browse-time estimate and the Checkout quote is explained rather than presented as an error.
- Quotes and Orders snapshot accepted destination, coverage, fees, and delivery estimates. Later Market, Shipping Profile, Shipping Coverage, or catalog changes never rewrite those accepted facts; new quotes and Orders use current configuration.
