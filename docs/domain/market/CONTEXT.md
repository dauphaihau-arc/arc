# Arc Market

Arc Market covers the buyer's commercial context: the Markets the storefront sells into, their enabled state, currencies and locales, and the single preview country each enabled Market declares. It supplies the country that Shipping interprets as the Preview Destination; Shipping owns that concept and every shipping decision derived from it.

## Language

**Market**:
The commercial configuration for a geography, identified by values such as `United States` and `Vietnam`. It owns its code, enabled state, currency set, default content locale, supported locales, and market-specific rules.
_Avoid_: Country, storefront, shipping zone

**Enabled Market**:
A Market buyers may select. Only an enabled Market must declare a preview country, and its configuration is validated at application startup and in CI.
_Avoid_: Active market, live market

**Preview Country Code**:
The Market's one `preview_country_code`: a valid ISO 3166-1 alpha-2 country that Shipping uses as the Preview Destination for that Market. Shipping owns the Preview Destination concept, its matcher, and its consequences; Market stores only this configuration value.
_Avoid_: Preview Destination (owned by Shipping), default shipping country, Market country

**Market Preference**:
The buyer's selected Market, stored as the `region` preference for an authenticated buyer and in guest preference storage otherwise. It selects a Market; it is not a shipping address, a Storefront Language, or a currency.
_Avoid_: Locale, country, delivery region

## Relationships

- An enabled Market declares exactly one `preview_country_code`, and it must be a valid ISO 3166-1 alpha-2 country.
- Configuration validation rejects an enabled Market whose preview country is missing, malformed, or unknown at startup and in CI. Disabled Markets are not validated because buyers cannot select them.
- The marketplace configuration response exposes `preview_country_code` per Market. The storefront never keeps its own Market-to-country map.
- Shipping interprets the preview country as the Preview Destination and owns Shipping Coverage and Shipping Preview. Market never resolves shipping, prices shipping, or decides whether a Product ships.
- A buyer's Market Preference selects the Market whose Preview Destination drives product detail, catalog badges and filtering, cart pre-checks, and suggestion and recommendation filtering.
- A Market change never rewrites a saved address, an accepted quote, or a confirmed Order; Checkout prices from the real shipping address.
- A Market does not constrain Storefront Language: every enabled Market can render every supported Storefront Language. Localization owns Locale, which pairs a Storefront Language with a Market region.
- The buyer preference field `region` carries the Market's display name and the `X-Market-Code` header carries its code. `preview_country_code` is a separate value and the client never derives it.
- The United States Market maps to `US` and the Vietnam Market maps to `VN`. The contract does not assume a one-to-one name match, so future Markets may declare any valid country.

## Flagged Ambiguities

- "Market" is a commercial context, not a delivery address. The Preview Destination is for browsing and pre-checkout estimates only; Checkout uses the shipping address.
- "Region" in buyer preferences and the wire field `region` means Market, not a sub-country shipping region or zone.
- `code` and `preview_country_code` may hold the same value today; they remain independent values.
- Market does not own shipping vocabulary. See Shipping for Preview Destination, Shipping Coverage, Shipping Preview, and Destination Rate.
