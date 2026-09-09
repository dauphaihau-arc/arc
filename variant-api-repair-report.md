# Variant API repair report

Outcome: succeeded.

Changed API behavior:
- Normalized variant configuration is now the only product variant write/read model in API source paths; legacy product variant aliases and old order/checkout/cart wire fields were removed from source mappers.
- Create-product draft facade accepts seller create payloads with `options`, `variants`, `inventory`, and `pricing`, then drives the canonical variant-configuration command.
- Checkout/order/cart snapshots now carry `selectedOptions`/`selected_options` instead of legacy variant name/group label fields.
- Migration `Migration20260908100000` now backfills normalized option/value/selection rows, rejects ambiguous legacy data, drops old product/variant/snapshot columns, and updates the MikroORM snapshot.
- Product command repository now prevalidates duplicate variant selections and reserved removals before mutation, persists new inventory before inserting movement rows, and populates variant inventory for command transitions.
- Seeds and fixtures no longer read removed entity fields for orders; product seed variants use normalized `combinationKey`.

Verification:
- `DB_HOST=127.0.0.1 DB_PORT=55432 DB_USER=postgres DB_PASSWORD=variant-test-only NODE_ENV=test pnpm test:int -- product-variant-normalization.int-spec.ts` passed: 4 tests, true PostgreSQL DB proof.
- `pnpm typecheck` passed in `apps/api/api`.
- Source legacy-field scan passed for `variant_name`, `variant_group_name`, `variant_sub_group_name`, `variantLabels`, `variantName`, `variantGroupName`, `variantSubGroupName`, `optionValue1`, and `optionValue2` under `apps/api/api/src/domains/**/*.ts` excluding specs.

Notes:
- Formatter/lint was intentionally not run; coordinator owns final formatting/validation.
- Existing historical migrations and legacy seed input TSV/SQL fixtures still mention old columns where they model pre-migration input.
