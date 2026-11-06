# Arc Location Reference Data

Arc Location owns the canonical Country and first-level Administrative Area values that addresses and destination pickers refer to. It does not own buyer addresses, shipping eligibility, address formatting, or city data.

## Language

**Country**:
A sovereign state or territory identified by its ISO 3166-1 alpha-2 code. The code is the canonical identity; a display name is presentation.
_Avoid_: Country name as identity, nationality

**Country Display Name**:
The human-readable name shown for a Country. It may change without changing Country identity.
_Avoid_: Country code

**Administrative Area**:
A first-level subdivision of a Country, such as a state, province, region, or department. It is identified by an ISO 3166-2 code where the dataset supplies one, and otherwise by a stable country-scoped code.
_Avoid_: State, Province, Region (as generic domain terms)

**Location Reference Data**:
Arc-owned, versioned Country and Administrative Area data served by Arc rather than fetched from a third-party provider at request time.
_Avoid_: Address metadata, geo API data

**Legacy Address Text**:
A country or subdivision string preserved on an existing address because it identifies no seeded Country or Administrative Area. It is displayed and retained, but it is never treated as canonical identity.
_Avoid_: Unknown country, fallback country

**Address Format Rules**:
The per-country field set, labels, order, requiredness, and postal-code format used to render an address form. Formats are owned by the API: generated library defaults are merged with Arc overrides in `country_address_rules`, served per country, and used for both rendering and validation.
_Avoid_: Address metadata

## Relationships

- A Country has zero or more Administrative Areas. Arc never assumes every Country has a first-level subdivision.
- An Administrative Area belongs to exactly one Country.
- A User Address refers to a Country by canonical code and, when one applies, to an Administrative Area by canonical code.
- New and updated User Addresses use codes as the only write identity; display names are derived by the API.
- An existing User Address whose text cannot be identified keeps Legacy Address Text rather than receiving a guessed code. Legacy text is storage-only and is never accepted as a new write identity.
- Destination Rates match a buyer destination by Country identity, never by display name.
- City is free text and is not Location Reference Data in the current model.
- Location Reference Data never decides Purchase Eligibility or whether a shop ships to a destination.

## Flagged Ambiguities

- "State" in address HTTP/API fields and `state_code` means Administrative Area, not a US state.
- Buyer-address country identity is always the alpha-2 code; display names are response-only.
- "Address metadata" conflates Location Reference Data with Address Format Rules; they are separate concerns.
- Countries without a trusted Administrative Area list accept free-text subdivision rather than a required empty dropdown.
