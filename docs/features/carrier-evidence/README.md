# Carrier Evidence

## Overview

Carrier Evidence is how a Shipment's status becomes credible after the seller hands
the parcel over. Sellers choose a platform-known carrier, prepare a Shipment with
quantities and tracking (or buy a Label in the app), and declare handover once.
Everything after handover comes from signed carrier events; the seller never writes
in-transit or delivered.

The feature spans four areas:

- **Fulfillment** — carrier catalogue, Shipment preparation, Handover Declaration,
  Carrier Evidence ingestion, Labels, Rate Quotes, refunds.
- **Order** — fulfillment projection, Order completion on carrier delivery, and
  buyer/seller order views.
- **Finance** — Label Charges and their reversal as ledger money events.
- **Mock Carrier** — a development service that emits the same signed webhook a real
  carrier would.

## Invariants

- A Shipment is `prepared → dispatched → in_transit → delivered`, with `voided`
  terminal. Preparation and label purchase never dispatch.
- The seller's only status authority is the Handover Declaration; it sets dispatch
  once.
- Carrier Evidence owns every later status. Duplicate events are acknowledged,
  unknown tracking is ignored, and stale evidence never regresses status.
- Tracking is validated against the carrier pattern; `other` is Unverified Tracking
  and produces no Carrier Evidence.
- A Label is `purchased` until the first real carrier scan marks it `used`. A refund
  before that scan voids the Label and Shipment, releases quantities, and reverses
  the charge exactly once.
- Prices and Label Charges use the shop's Settlement Currency; a Label Charge debits
  available Seller Payable immediately and never re-prices Order money.
- Delivery is the only trigger for Order completion and Payout Availability.

## Diagrams

| Diagram | Answers |
|---|---|
| [Shipment lifecycle](diagrams/shipment-lifecycle.html) | Who may change a Shipment status, and what the seller can still do after handover. |
| [Shipment and label schema](diagrams/carrier-evidence-schema.html) | The rows and keys behind Shipment evidence, updates, and purchased labels; ledger linkage is by money-event key. |
| [Carrier evidence ingestion](diagrams/carrier-evidence.html) | Signature, dedupe, routing, evidence storage, Label used-marking, projection, and notifications. |
| [Label purchase](diagrams/label-purchase.html) | Quote → purchase → PDF storage → prepared Shipment → Label Charge via the outbox. |
| [Label refund](diagrams/label-refund.html) | The refund decision tree, carrier-first ordering, local void, and one idempotent reversal. |

## Scope

In scope: the seller fulfillment journey from preparation through carrier delivery,
manual tracking and bought labels, Carrier Evidence ingestion, buyer/seller order
visibility, and Label money.

Out of scope: real carrier integrations, carrier rate zones or customs fields,
multi-package label batches, and admin/support shipment tooling.
