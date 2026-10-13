# Category Identity and Navigation

Arc's category context owns category hierarchy, stable public identities, current slugs, and historical path ownership.

## Language

**Category Public Identity**: The public identity of a category, distinct from its internal database identity. It survives renames, moves, and unavailability and is never reassigned.

**Category Slug**: The readable current name segment used by canonical category URLs. A display-name change does not implicitly change it.

**Category Canonical Path**: The flat category URL `/c/:slug-c:publicId`. The `c` marker is URL-layer only and is never persisted; the public ID disambiguates duplicate slugs, so a slug change alters the path without changing identity.

**Category Resolution**: Authoritative lookup of a category public identity returning its current slug and its ordered root-to-current ancestry. Breadcrumbs derive from this ancestry; session state is never a prerequisite.

**Category Subtree Scope**: The set of a category together with its descendants, used to scope storefront listings and facets.

**Legacy Category Path**: A historical hierarchical path permanently associated with the category identity it originally addressed. It is never reassigned; an unavailable owner remains unavailable rather than sending the path to another category.

**Legacy Path Conflict**: A historical path with more than one candidate category owner. Ownership must be resolved explicitly before the path can be used for routing.

## Relationships

- A Category Public Identity is separate from the internal category key and its relationships.
- A Category Slug may be shared by multiple categories; identity disambiguates URLs.
- A Legacy Category Path identifies its original category, not a destination URL.
- Category hierarchy and effective storefront availability are distinct concepts.

## Supported storefront operation

The public boundary uses positive decimal identities as strings; internal
category UUIDs remain persistence relationships and are never accepted as a
public-ID fallback. Display labels are not identity, and duplicate slugs are
valid. API and clients speak category identity generation `2`.

The hierarchy is seeded before traffic and fixed during supported operation.
Listings and facets scope eligible Products to the Category Subtree Scope;
facets exclude only their own active filter when calculating alternatives.
The optional taxonomy-subtree cache lasts five minutes. Unsupported hierarchy
changes can leave cached facets stale relative to uncached listings for that
period; no immediate mutation consistency is promised.

An accessible category with no eligible Products remains a `200` empty page.
Category availability does not inherit Product lifecycle states. Category
moves, new category availability controls/ancestor propagation, and explicit
admin slug editing are deferred; existing hiding/deletion behavior is preserved.

Historical paths resolve their permanent owner to its current flat URL in one
HTTP `301`, preserving the query. Unknown paths do not redirect to a parent;
backend failures remain errors rather than fabricated not-found results.
