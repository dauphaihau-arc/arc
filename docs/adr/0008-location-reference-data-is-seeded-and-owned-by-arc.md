---
status: accepted
---

# Location Reference Data is seeded and owned by Arc

Both web applications currently call `countriesnow.space` directly for country and subdivision options, while Arc stores buyer-address countries and states as display names and shipping destination rates as ISO alpha-2 codes. That leaves address options dependent on a third-party service with no SLA, duplicates the integration in two apps, and forces name-to-code resolution at the shipping boundary. Arc therefore owns Location Reference Data: countries and first-level administrative areas are imported into Postgres from a pinned dr5hn dataset export, served by the API, and referenced by canonical codes. Cities remain free text, and address-format rules remain deferred.

## Dataset and license

The import takes dr5hn's country records and level-1 subdivision records only: 250 countries and 3,863 administrative areas from a pinned release. dr5hn is licensed under ODbL 1.0: Arc must attribute the source, and a derivative database must be offered under the same license. The dataset release version is recorded with the imported data, and the attribution notice travels with the repository and published API documentation.

Vietnam is the one explicit override: dr5hn's VN rows are replaced by the pinned `open-admin-data/vietnam-administrative-divisions` dataset (CC-BY-4.0, Decision 19/2025/QĐ-TTg), using official numeric codes prefixed `VN-`. Arc treats VN as a core market, where stale first-level data is a correctness defect rather than a cosmetic one.

No provider is called at request time; refreshes are explicit, versioned imports. The API exposes `/v1/locations/countries` and `/v1/locations/countries/:code/administrative-areas`, caches them publicly with an ETag derived from the dataset version, and returns active rows only. Refreshes upsert and deactivate missing rows rather than deleting rows referenced by addresses.

## Considered alternatives

- **Keep direct frontend calls.** Rejected: CORS and rate limits depend on the provider, both Nuxt apps duplicate the integration, and there is no place to normalize names to codes or join against shipping destinations.
- **Proxy the provider through the API with a cache.** Rejected: countries and subdivisions change rarely, so a runtime proxy still couples checkout availability to a free service while adding latency and cache complexity.
- **Build the seed from ISO/CLDR data instead of dr5hn.** This avoids ODbL share-alike, but the team chose dr5hn for its first-level subdivision coverage and accepted the attribution and share-alike obligations.
- **Seed countries, subdivisions, and cities.** Rejected: city datasets are incomplete, informal place names are common, and a city dropdown is worse than free text for the supported markets.

## Rollout

The contract moves as expand/contract rather than one hard cutover: add code columns and endpoints, backfill, switch both web apps and every location consumer, then remove legacy name-only request handling in a following release. Checkout and new order snapshots carry both codes and display names, so guest checkout and saved addresses follow one canonical path without discarding human-readable history.

## Consequences

- Buyer addresses persist canonical country and subdivision codes; display names remain available for presentation and migration.
- Existing address rows are backfilled where a seeded match exists; unmatched values stay as explicit legacy text rather than being guessed.
- Shipping destination matching no longer needs to translate arbitrary display names at quote time.
- Arc now carries an ODbL attribution obligation and must keep the imported reference database separable enough to satisfy share-alike when it is publicly used.
- Refreshing the data is an operational task with a recorded dataset version, not a background dependency on a provider.
- Address-format rules, city reference data, and autocomplete are explicitly outside this decision.
