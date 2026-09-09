# Add Values Or Expand Dimensions

See the [design](../README.md) and [flow index](../flow.md).

```mermaid
flowchart TD
    A[Build target option matrix] --> B{Existing combination unchanged?}
    B -- Yes --> C[Keep variant and inventory IDs]
    B -- No --> D[Create replacement or new combination]
    D --> E[Require seller-reviewed count and price]
    C --> F[Acknowledge every superseded variant]
    E --> F
    F --> G{Superseded inventory has active reservations?}
    G -- Yes --> H[Reject without moving stock or reservations]
    G -- No --> I[Commit complete target configuration]
```

- Adding Green to a Blue/Red option preserves Blue and Red combinations.
- Adding Size to a published Color-only Product changes the old combinations. Color-only identities remain historical; Color/Size combinations receive new identities.
- Counts are not copied into every generated row. The seller must review replacement inventory explicitly.
- An unwanted target combination remains present as inactive; leaving it out would create an incomplete matrix.
