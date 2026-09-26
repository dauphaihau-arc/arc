---
status: accepted
---

# Default Shipping Profile is a creation-time affordance, not a server-side fallback

A shop may designate one Checkout-ready Shipping Profile as its Default Shipping Profile. The seller product-creation form pre-selects it; the API never assigns a profile on its own. Creating a draft with no `shipping_profile_id` still leaves the Product unassigned, exactly as today.

## Considered alternatives

- **Server-side fallback in create-draft.** When a create request omitted `shipping_profile_id`, the API would assign the shop's Default Shipping Profile. This would silently change create-draft semantics for every caller, including Product Imports, which has its own row-level outcome model and must not inherit a profile the seller never chose. It would also make "assigned" and "defaulted" indistinguishable after the fact.
- **Backfill unassigned Products.** Assigning the default to every Product that currently holds no profile would rewrite seller intent for drafts that are deliberately unassigned, and would have to re-run the publish-readiness guard against Products the seller never published.
- **Derive the default instead of storing it.** Picking, say, the oldest active profile removes the need for a stored designation, but makes the pre-selection change underneath the seller whenever profiles are created or archived.
- **Bump the profile's configuration version when the designation changes.** `version` exists to reject stale configuration replacement. Folding a shop-level flag into it would surface spurious version conflicts on unrelated edits and invalidate an in-flight editor for a change that touched no rate or handling time.

## Consequences

- The Default Shipping Profile is a stored, shop-scoped designation with at most one per shop, enforced by a database constraint as well as by the transition guard.
- Only a Checkout-ready Shipping Profile may hold the designation, so the pre-selection never offers a profile that cannot price a checkout.
- A profile that stops being Checkout-ready — archived, or edited below readiness — loses the designation instead of blocking the transition. Archiving keeps only its existing published-reference guard.
- A shop starts with no Default Shipping Profile; the seller designates one explicitly. Nothing is inferred from profile age or usage.
- The designation is not itself an assignment: no Product becomes assigned because a profile is the default.
- Checkout never substitutes the default for a Product with no assignment; that stays an unavailable shipping reason.
- Setting and clearing the designation is a dedicated, naturally idempotent command, so it carries no idempotency key and never bumps the profile's configuration version.
- Clearing the designation rides in the same transaction as the archive or readiness-degrading edit that removed eligibility.
- The profile editor dialog never sets the designation; it is set only from the Shipping Settings profiles table, and the product-creation picker marks the default so the pre-fill is explained.
- A shop with no Default Shipping Profile keeps today's behaviour, where the seller chooses a profile explicitly.
- The edit form for an existing Product never re-derives the selection from the default; it shows the Product's own assignment.
