# Recover From A Conflict

See the [design](../README.md) and [flow index](../flow.md).

1. Two editors load Product Version N.
2. Editor A saves successfully, producing N+1.
3. Editor B submits N and receives a conflict with current Product state instead of overwriting A.
4. Compare B's intent against the current options, identities, counts, and versions.
5. Reapply the intended changes deliberately. If replacement rows are needed, review their counts and prices again.
6. Submit the revised target with current versions and a new idempotency key.

For a SKU conflict, choose a valid SKU or resolve ownership. For a reservation conflict, do not transfer the reservation or automatically retry a destructive transformation. Resolve it through the reservation lifecycle before rebuilding the intended configuration.
