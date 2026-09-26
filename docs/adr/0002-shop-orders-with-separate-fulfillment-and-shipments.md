---
status: accepted
---

# Keep shop Orders while separating fulfillment assignment from physical Shipments

Arc keeps one Order per shop, as today, and represents fulfillment separately: Order Item quantities are assigned to Fulfillment Groups by method and operator, and physical dispatch is represented by one or more Shipments within a group. This preserves the buyer's single shop purchase and the seller's commercial accountability when a future Provider Fulfillment operator fulfills part of an Order, and it allows one Order Item's quantity to be divided without changing its identity. The group and Shipment model is implemented for Seller Fulfillment: confirmed Orders receive a persisted seller group, and preparation, dispatch, transit, and delivery are recorded per Shipment. Order-level shipping details and tracking are retained only as immutable legacy evidence for pre-cutover Orders, and only Seller Fulfillment operates.

## Considered alternatives

- **Split Orders by fulfillment source.** Creating a separate Order per operator would make fulfillment routing part of Order identity, fragment the buyer's single shop purchase across records, and force Order Item identity or quantity duplication for one purchased line. Confirmed purchase facts such as the Order Item Snapshot, shipping charge, and delivery promise would no longer have one natural owner.
- **Keep one Order and one shipment.** The current one-status/one-tracking assumption cannot represent two parcels dispatched on different days or a mix of seller- and provider-fulfilled goods. Parcel-level progress would be misreported, and partial delivery of one consignment would imply delivery of everything.

## Consequences

- Quantity accounting spans two levels: Fulfillment Group assignments partition an Order Item's ordered quantity, and Shipments consume assigned quantities. Neither level may exceed the Order Item's ordered quantity or count a unit twice.
- A Shipment may exist before carrier handover, so Shipment creation does not by itself mean dispatch.
- Fulfillment Progress is a quantity roll-up over groups and the Order, distinct from a Shipment's journey and from the commercial Order state.
- Inventory remains the authority for stock quantities and movements; Fulfillment references stock identities rather than owning stock.
- A Stock Pool is the internal accounting source for one Inventory Item; Seller Fulfillment uses exactly one default seller Stock Pool per Inventory Item, and it does not create new Product Variant, Inventory Item, or SKU identities.
- Order-level shipping details and tracking for pre-cutover Orders are retained as immutable legacy evidence and are never inferred into group or Shipment membership. Active legacy Orders require an explicit seller reconciliation of remaining quantities before new quantity-sensitive mutation.

## Verification

Future behavioral verification should use the public order and fulfillment APIs as the preferred cross-context seam: exercise commands and observe returned or subsequently retrieved buyer/seller state and quantity effects across Checkout, Ordering, Fulfillment, and Inventory. The `commerce` and `fulfillment` integration suites are prior art; `OrderCheckoutService` unit tests mock dependencies and do not prove cross-context inventory correctness. A lower-level inventory/concurrency seam is justified only when a public scenario cannot reliably exercise the race or stock-commitment invariant.
