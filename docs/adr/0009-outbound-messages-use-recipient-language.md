---
status: accepted
---

# Outbound transactional messages use recipient language and repo-owned catalogs

Arc currently sends buyer transactional email and Web Push with English copy embedded in jobs and notification builders. The copy cannot follow the buyer's Storefront Language, guest orders carry no reliable language after checkout, and Seller/Admin notifications share the same English path. PDFs/invoices and SMS do not exist yet, so this decision deliberately scopes them out.

## Decision

Outbound buyer messages are language-specific and repository-owned.

- **Order-bound email uses an Order Language snapshot.** Confirmed Orders store `language` (`en` or `vi`) captured from the checkout URL's `X-Locale` on every order-creation path, authenticated or guest. Existing Orders backfill to `en`; no language is inferred from Market or currency. Future order SMS/PDF follow the same snapshot.
- **Live in-app and Web Push use the current Buyer Language Preference.** Order-linked in-app notifications render at read time and Web Push renders at dispatch time in the preference active then; changing language updates live surfaces without rewriting historical email delivery.
- **Account messages use Buyer Language Preference.** Welcome and password-reset emails use `user_preferences.language` for accounts that hold only customer access. Any Seller/Admin role keeps account messages English because those applications are not localized.
- **Buyer scope only.** Buyer transactional emails, buyer in-app notifications, and buyer Web Push are localized. Seller/Admin notifications, seller order-update emails, and payout emails remain English. SMS and PDFs/invoices are deferred.
- **Copy is repository-owned.** English is the source, Vietnamese is reviewed, and no TMS or machine translation runs at send time. Catalogs are typed TypeScript modules owned by the domain that owns the event; shared catalog/render contracts live in platform code. CI enforces `en`/`vi` key parity.
- **Notification rendering is keyed by `type` and `data`.** In-app text renders at read time and Web Push at dispatch time, both in the recipient's current Buyer Language Preference. Order-bound email renders at dispatch using the Order Language. Stored `title`/`body` remain required and hold an English fallback for legacy readers and unknown types.
- **Fallback is whole-message.** If Vietnamese copy is incomplete, the entire message uses English rather than mixing languages. If an existing notification lacks the data required by its localized template, the stored English `title`/`body` is used. New notification types must include the complete template data contract.
- **Email presentation is shared.** One responsive HTML layout plus a text alternative carries localized content. Money formats with `Intl.NumberFormat` using recipient language and the Order currency; no currency conversion occurs.
- **Provider failure text stays internal.** Buyer refund messages do not interpolate raw provider errors. They use localized safe copy; the raw reason remains in logs.

## Considered alternatives

- **Resolve every message from current user preference.** Rejected for order-bound email: a buyer can change language after ordering, and guests have no preference. Live in-app/push intentionally use the current preference.
- **Infer language from `market_code`.** Rejected: Market is commercial context, not an explicit Storefront Language choice.
- **Translate at send time with a provider.** Rejected: inconsistent copy, provider latency in jobs, and no reviewed Vietnamese source.
- **Store rendered per-language notification title/body.** Rejected: historical language would freeze and readers cannot follow a later language change.
- **Centralize all copy in the notification domain.** Rejected: account copy belongs to Auth, order copy to Ordering, and notification infrastructure should not own business copy.
- **Localize Seller/Admin messages now.** Rejected: those applications and their generated emails remain English; adding a language contract without localized surfaces creates false precision.
- **Include SMS and invoice PDFs now.** Deferred: neither channel exists, and invoice requires legal/tax and numbering decisions that are separate from message language.

## Consequences

- Ordering gains a language snapshot column and must set it in cart, buy-now, guest, and authenticated checkout paths.
- Existing Orders backfill to `en`; historical rows without a row-level language fall back to English.
- Notification readers gain a renderer that uses catalog `type` plus `data`, while retaining stored copy as compatibility fallback.
- Buyer notification builders must put all interpolation fields into `data`, including order number where the template needs it.
- Buyer message renderers share formatters for money and text; seller/admin messages remain on their current English path.
- SMS consent/phone handling and invoice generation remain separate future decisions.
