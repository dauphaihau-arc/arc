# Arc Shipping

Arc Shipping covers the reusable, shop-owned Shipping Profiles that Products are assigned to, the destination rates and handling times those profiles carry, and the buyer-facing Preview Destination, Shipping Coverage, and Shipping Preview derived from them.

## Language

**Shipping Profile**:
A reusable, shop-owned set of destination rates and handling times that a Product is assigned to for delivery pricing. A Product holds at most one, and a profile may be assigned to many Products.
_Avoid_: Shipping Template, Rate Group, Shipping Method

**Destination Rate**:
One destination scope and its fees and delivery time within a Shipping Profile. A profile carries several; a Product's assigned profile supplies all of them.
_Avoid_: Shipping Option, Zone, Tier

**Checkout-ready Shipping Profile**:
A Shipping Profile that can price a checkout right now: it is active, named, carries at least one Destination Rate, and has complete valid processing and delivery time ranges.
_Avoid_: Valid profile, Usable profile, Complete profile

**Default Shipping Profile**:
The one Shipping Profile a shop designates to be offered for new Products, so creating a Product does not require choosing a profile every time. Only a Checkout-ready Shipping Profile may be the default, and a shop has at most one.
_Avoid_: Primary profile, Preferred profile, Main profile

**Preview Destination**:
The single country an enabled Market declares for buyer-facing shipping before a delivery address is known. Shipping resolves Shipping Coverage and Shipping Preview against it on product detail, catalog, cart, and suggestion surfaces. It is not a delivery address and never overrides the checkout shipping address.
_Avoid_: Market country, Default destination, Shipping country, Zone

**Shipping Coverage**:
The destinations a Product can be delivered to, derived from its assigned Shipping Profile: each `country` Destination Rate contributes that country, and an `everywhere_else` rate contributes worldwide coverage. It is projected into catalog documents as `ships_to_country_codes` and `ships_worldwide`; a missing or non-Checkout-ready Shipping Profile contributes none. Digital Products bypass physical shipping and carry no Shipping Coverage.
_Avoid_: Shipping zones, Delivery regions, Serviceable area

**Shipping Preview**:
The live buyer-facing result for one Product and one Preview Destination: whether it can ship, a stable reason when it cannot, the converted one-item fee in the buyer's display currency, and the total delivery estimate combining Processing Time and Delivery Time. A Digital Product reports its digital fact instead of physical availability. Unavailable reasons are stable enums: `missing_assignment`, `profile_not_found`, `profile_not_ready`, `unsupported_destination`, and `unsupported_currency`.
_Avoid_: Shipping quote (reserved for checkout), Shipping estimate, Delivery promise

## Relationships

- A Shipping Profile belongs to exactly one shop; Products of other shops cannot be assigned to it.
- A Product holds at most one Shipping Profile, and may hold none while unpublished.
- A published Product must hold a Checkout-ready Shipping Profile; an unpublished Product may hold one that is not yet Checkout-ready.
- A Shipping Profile may only be archived when no published Product references it.
- A shop designates at most one of its Shipping Profiles as its Default Shipping Profile.
- A Shipping Profile that stops being Checkout-ready — archived, or edited below readiness — loses the designation; it is never kept ineligible.
- The Default Shipping Profile is a creation-time convenience: it is offered when a Product is created, and it does not assign itself to any Product.
- Checkout never substitutes the Default Shipping Profile for a Product that holds no Shipping Profile.
- Editing a Shipping Profile changes future delivery pricing for every Product assigned to it; it never rewrites delivery pricing already accepted at checkout.
- Every enabled Market declares exactly one Preview Destination through its `preview_country_code`. Market owns that configuration value; Shipping owns the Preview Destination concept, its matcher, and everything derived from it.
- A Market change changes the Preview Destination and every Shipping Preview, catalog badge, or cart pre-check derived from it. It never rewrites a saved address, an accepted quote, or a confirmed Order.
- Destination Rate remains the only destination model. Shipping Coverage is derived from `country` and `everywhere_else` rates; zones, Destination Groups, and sub-country destinations are not part of the model.
- A Digital Product bypasses physical shipping: it has no Shipping Coverage or fee and remains available in every Preview Destination.
- Product detail resolves Shipping Preview live from the current Product assignment and Shipping Profile. Catalog badges read projected Shipping Coverage and may lag until a profile or assignment change is reprojected; catalog coverage is backfilled and verified before coverage filtering is enabled.
- Catalog browse may override the destination only with `catalog_ship_to=<ISO 3166-1 alpha-2>`. `catalog_ship_to=all` disables coverage filtering, and an absent value follows the Market's Preview Destination. This override is browse-only: it never changes product detail, cart, or checkout.
- Cart lines carry the same availability and reason for the Preview Destination. An unsupported selected line stays in the cart, is reported in `unavailable_quantity`, is excluded from money totals, and blocks checkout; adding it to the cart remains allowed with a warning.
- Checkout uses the real shipping address as the delivery authority, never the Preview Destination. Any selected line without a Destination Rate for that address rejects the whole quote with `CHECKOUT_SHIPPING_UNAVAILABLE`; a supported address remains purchasable even when the Preview Destination is unsupported.
- Checkout notes when the shipping address country differs from the Preview Destination. Quotes and confirmed Orders snapshot the accepted shipping facts and are never rewritten by later profile, assignment, or Market changes.

## Flagged Ambiguities

- "Preview" means buyer-facing availability before an address exists; it is not a quote or a delivery promise.
- "Ships to" in catalog browsing means Shipping Coverage for the selected destination, not Purchase Eligibility.
- Add to cart succeeding does not mean checkout will accept the destination; the shipping address decides.
- "Zone" is not a model. The destination scopes are `country` and `everywhere_else`, and Destination Rate is canonical.
