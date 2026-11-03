---
status: accepted
---

# Carrier evidence drives Shipment status after dispatch

The seller no longer sets `dispatched`, `in_transit`, or `delivered`; the Shipment journey after handover is derived only from Carrier Evidence delivered over a signed webhook. The seller keeps exactly one authority: a Handover Declaration that records dispatch and notifies the Carrier, because offline or unsupported carriers cannot scan. Every Shipment Update still records its actor and source, so seller declarations, carrier scans, and any future buyer confirmation stay distinct facts.

## Considered alternatives

- **Keep seller-reported journey transitions.** It is what the code does today (`POST .../shipments/:id/journey`), but it makes the seller the author of delivery, which also releases the seller's payout. The party being paid would attest to the event that pays them.
- **Accept only carrier evidence, including dispatch.** Cleanest model, but an unsupported carrier or an offline handover could never dispatch or deliver, so the order would never complete and the seller would never be paid.

## Consequences

- The mock carrier service is required for development: without carrier events, shipments stop at dispatch and Orders never complete.
- `delivered` remains the trigger for Order completion and payout availability, so losing carrier webhooks stalls money, not just tracking.
- A seller declaration is evidence, not a status override: it may set dispatch once, and any later transit or delivery comes from the Carrier.
- Platform-tracked status can disagree with a seller's claim; the recorded history keeps both rather than overwriting either.
- Removing the seller journey endpoint requires rewriting the fulfillment integration scenarios that currently assert seller-set transitions.
