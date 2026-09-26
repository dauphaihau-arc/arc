# Arc Shipping

Arc Shipping covers the reusable, shop-owned Shipping Profiles that Products are assigned to, and the destination rates and handling times those profiles carry.

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
