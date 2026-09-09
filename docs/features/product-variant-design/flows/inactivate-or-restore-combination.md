# Inactivate Or Restore A Combination

See the [design](../README.md) and [flow index](../flow.md).

The configuration API can retain a combination with `lifecycle_state: inactive`. Its identity remains stable and the complete matrix still includes it. Returning that same combination to active is distinct from restoring a removed identity.

Restoration submits the removed variant's original ID in the target and acknowledges it in `restore_variant_ids`. The server revalidates its selection identity and SKU ownership. A label match alone must not resurrect a removed variant. These are API behaviors; a dedicated restoration UI is not assumed.
