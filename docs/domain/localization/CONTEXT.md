# Localization and Storefront Language

Arc's storefront serves the same catalog under more than one UI language. This
context owns the vocabulary for distinguishing a buyer's UI language, a market's
commercial context, and the locale used to render and format a page. It does not
own localized catalog content, which belongs to the context that owns each record.

## Language

**Market**: The commercial configuration for a geography, identified by values
such as `United States` and `Vietnam`. It owns the currency set, default content
locale, and market-specific rules.
_Avoid_: Region, country, storefront

**Storefront Language**: The language of buyer-facing interface copy, identified
by an ISO 639-1 code (`en`, `vi`). It is a buyer choice and is not constrained by
Market.
_Avoid_: Market language, lang

**Locale**: The BCP-47 pairing of a Storefront Language with a Market's region
(`en-US`, `vi-US`, `vi-VN`). It selects UI copy and controls date, number, and
text formatting.
_Avoid_: Language, when the region is part of the value

**Buyer Language Preference**: The Storefront Language a buyer or guest has
explicitly chosen, persisted to `user_preferences.language` for authenticated
buyers and to guest preference storage otherwise. Guest preference storage may
also carry auto-detected Market values (region and currency); its language value
is always an explicit choice. It may influence later navigation but never
overrides an explicit URL.
_Avoid_: Locale preference, browser language

**UI Translation**: Developer-maintained key/value interface copy. Owned by the
storefront.
_Avoid_: Localization, content translation

**Localized Content**: Seller- or admin-authored text for Products, Categories,
and Shops per Locale. Owned by the context that owns the record, not by the
storefront.
_Avoid_: UI translation, static content

**Language Fallback**: The English UI Translation or Localized Content value
rendered when the selected language has no value.
_Avoid_: Default locale

## Relationships

- A Locale is a Storefront Language plus a Market region; changing either changes
  the Locale.
- A Market does not constrain Storefront Language: every enabled Market can
  render every supported Storefront Language.
- A Buyer Language Preference names a Storefront Language. It is not a Market or
  a currency choice.
- UI Translation is owned by the storefront; Localized Content is owned by Seller
  Catalog or Categories. The storefront selects a Locale; it does not author
  content translations.
- The URL language prefix identifies the Storefront Language and is the rendering
  source of truth for UI Translation.

## Supported storefront operation

- Supported Storefront Languages are `en` and `vi`. English is the default and
  is served without a URL prefix; Vietnamese is served under `/vi/**`.
- A first visit renders English. Vietnamese is rendered only after an explicit
  choice or an explicit `/vi` URL.
- An explicit URL always wins over a Buyer Language Preference. A stored `vi`
  preference client-redirects the bare `/` only; no other path is redirected.
- Every storefront route exists in both locales, including cart, checkout,
  account, and orders. Product, Category, and Shop slugs are identity and are
  never translated.
- Requests carry the Locale in `X-Locale`; every enabled Market accepts both
  Storefront Languages.
- The previous `la` and `fr` preference values are retired and migrate to `en`.
- UI Translation keys have parity between `en` and `vi`. A missing key falls back
  to English at runtime, and CI fails on missing or extra keys.
- UI Translation copy is repository-owned: English is the source key set, and
  Vietnamese is authored and reviewed by a native speaker. No TMS or runtime
  machine translation.
- Known API error codes are translated. An unknown code shows a generic localized
  message; the raw server message and `request_id` stay in logs and never reach
  the buyer.
- Backend-produced emails, notifications, and invoices remain English in this
  delivery.
- Shared `@arc/ui` components stay locale-agnostic; the storefront supplies their
  copy through props and slots. Seller and Admin are not localized by this work.
- The first delivery translates storefront UI only. Product, Category, and Shop
  content remain English until Localized Content exists.
- Currency is a Market preference. Changing Storefront Language never converts or
  changes currency.
- Dates and numbers follow the Locale. Money is formatted for the selected
  currency, not converted. UI messages are keyed by Storefront Language; date,
  number, and text formatting use the full Locale.
- Locale persistence has one owner: Buyer Language Preference. The i18n runtime
  keeps no second locale cookie.
- A guest's Buyer Language Preference applies only while unauthenticated. An
  authenticated Buyer Language Preference wins after sign-in.
- Market auto-detection may seed a guest's region and currency. It never selects
  Vietnamese; Storefront Language stays English until an explicit choice.
- Each locale is self-canonical and exposes language-level `hreflang` alternates
  (`en`, `vi`) with `x-default` on English; the language switcher emits crawlable
  links. Page titles and meta descriptions are translated. No sitemap is added in
  this delivery.
