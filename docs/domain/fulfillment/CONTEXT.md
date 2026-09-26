# Arc Fulfillment

Arc Fulfillment describes who is responsible for physically fulfilling confirmed Order quantities and how those quantities are consigned, without changing commercial ownership or the buyer's confirmed purchase facts. Seller Fulfillment is operational: confirmed Orders are assigned to a persisted seller Fulfillment Group and fulfilled through quantity-based Shipments.

## Language

### Responsibility

**Seller**:
The commercial party accountable to the buyer under Arc policies for an Order, including cancellation, refund, and dispute resolution. Seller accountability does not move with the fulfillment operator.
_Avoid_: Fulfiller, Vendor, Provider

**Fulfillment Method**:
The arrangement responsible for preparing and dispatching an Order Item quantity: Seller Fulfillment or Provider Fulfillment.
_Avoid_: Delivery Method, Shipping Method

**Seller Fulfillment**:
Fulfillment performed by the seller, which holds and dispatches the goods itself. It is the only Fulfillment Method operational initially.
_Avoid_: Self-fulfillment, Merchant Fulfillment, FBA

**Provider Fulfillment**:
Fulfillment performed by a party other than the seller under a provider arrangement. It is an agreed target capability, not an available product option.
_Avoid_: FBA, Third-party Fulfillment, Outsourced Fulfillment

**Fulfillment by Amazon (FBA)**:
Amazon's provider fulfillment model. FBA is one possible Provider Fulfillment arrangement, not a generic label for Arc-operated or other provider fulfillment.
_Avoid_: Provider Fulfillment, Platform Fulfillment

**Fulfillment Operator**:
The party responsible for the physical fulfillment of an Order Item quantity, whether the seller or a provider. A carrier transporting a consignment is not automatically the Fulfillment Operator.
_Avoid_: Carrier, Shipper, Fulfiller

### Order structure

**Fulfillment Group**:
Order Item quantities within one shop Order assigned to one Fulfillment Method and one Fulfillment Operator. A Fulfillment Group is defined by method and operator; splitting quantities into several Shipments does not create another group.
_Avoid_: Package, Parcel, Warehouse Group

**Shipment**:
A physical consignment carrying specified Order Item quantities with its own recorded origin evidence, destination, carrier, tracking details, and journey. A Shipment may exist before carrier handover, and one Fulfillment Group may require multiple Shipments.
_Avoid_: Package, Parcel, Fulfillment Group

**Fulfillment Progress**:
The quantity-based progress of a Fulfillment Group or Order. It is distinct from a Shipment's own journey and from the commercial Order state.
_Avoid_: Shipment Status, Order Status, Delivery Status

### Journey

**Shipment Preparation**:
Assembling and labeling a Shipment before carrier handover. Preparation, including label creation, does not mean the Shipment is Dispatched.
_Avoid_: Dispatch, Shipped

**Dispatched**:
The point at which a Shipment's goods are handed to the carrier. A prepared Shipment or printed label is not Dispatched.
_Avoid_: Shipped, In Transit, Delivered

**Shipment Update**:
A recorded change to a Shipment's journey that identifies its reporting actor and source. Seller reports, carrier or provider evidence, and buyer confirmation remain distinct facts, and conflicting history is retained.
_Avoid_: Order Status, Fulfillment Progress

**Revised Estimate**:
A later forecast of a Fulfillment Group's delivery that does not replace the buyer's accepted Delivery Promise.
_Avoid_: Delivery Promise, ETA

### Cancellation

**Cancellation Request**:
A request, from the buyer or seller, to cancel quantities. A request alone does not remove quantities from fulfillment obligations.
_Avoid_: Cancellation, Cancelled Order

**Confirmed Cancellation**:
An accepted cancellation that removes the affected quantities from fulfillment obligations. It is mutually exclusive with dispatch for the same units.
_Avoid_: Cancellation Request, Refund

### Exceptions and recovery

**Fulfillment Exception**:
A recorded condition preventing a Fulfillment Group or Shipment from completing as planned, such as unavailable replacement stock or a failed delivery. It is surfaced as an exception, never as successful fulfillment.
_Avoid_: Cancellation, Delivery

**Delivery Exception**:
A Fulfillment Exception where a Shipment fails to deliver along its expected journey, such as a loss or refused delivery. Dispatch and delivery evidence are preserved when it is recorded.
_Avoid_: Return, Cancellation

**Return to Sender**:
A Shipment returned to its origin by the carrier instead of being delivered. It is not a Buyer Return.
_Avoid_: Buyer Return, Refund

**Buyer Return**:
Goods returned by the buyer after delivery. It is distinct from a Return to Sender and a Delivery Exception.
_Avoid_: Return to Sender, Refund

**Replacement Fulfillment**:
Fulfillment of an authorized replacement for a failed obligation, explicitly linked to the original obligation. It does not reset previously dispatched quantities.
_Avoid_: Re-ship, Duplicate Shipment

## Relationships

- One shop Order may contain multiple Fulfillment Groups; every Fulfillment Group belongs to exactly one Order.
- A Fulfillment Group uses exactly one Fulfillment Method and one Fulfillment Operator.
- A Fulfillment Group accounts for Order Item quantities; one Order Item's quantity may be divided across Fulfillment Groups without changing the Order Item's identity.
- A Fulfillment Group may require multiple Shipments.
- A Shipment carries assigned quantities and never exceeds an Order Item's assigned, uncanceled quantity.
- Provider compensation is a provider arrangement and does not change the seller's accountability to the buyer.
- The Order records its origin evidence once at confirmation; a Shipment copies that evidence, so later configuration changes never change an Order that was already accepted. The seller never selects a Warehouse, Stock Pool, or source.
- Partial Shipments must remain consistent with the buyer's accepted Delivery Promise.
- A Shipment's recorded origin evidence is immutable; later Product shipping configuration affects only future quotes and Orders.
- In the target lifecycle, dispatch precedes transit within a Shipment's journey.
- Cancellation eligibility is determined when the outcome is accepted, not from the state when cancellation was requested.
- A Shipment's journey does not complete a Fulfillment Group or Order while assigned quantities remain outstanding.
- Fulfillment Progress is complete only when every non-canceled unit is delivered; canceled units remain canceled and are never counted as delivered.
- An Order whose units are all canceled is canceled, not fulfilled.
- Recording an exception, return, or replacement does not reset dispatched quantities or erase dispatch history.
- Fulfillment Progress counts quantities; it is not a Shipment's journey and does not by itself complete the commercial Order.

## Example Dialogue

> **Dev:** "A buyer orders a mug and a shirt from one shop. The mug ships from the seller and the shirt from a provider. Are those two Orders?"
> **Domain expert:** "No. It is one shop Order with two Fulfillment Groups: one Seller Fulfillment group and one Provider Fulfillment group. The seller remains the buyer's commercial party."
>
> **Dev:** "The buyer orders three identical units, one seller-held and two provider-held. Does that create two Order Items?"
> **Domain expert:** "No. It is one Order Item whose quantity is divided across two Fulfillment Groups: one unit in the seller group and two in the provider group, with no unit counted twice."
>
> **Dev:** "The seller dispatches two parcels on different days. Is that two Fulfillment Groups?"
> **Domain expert:** "No. It is one seller Fulfillment Group with two Shipments, each with its own tracking and quantities."
>
> **Dev:** "The seller moves premises and changes their Product shipping configuration. Do shipped Orders change?"
> **Domain expert:** "No. Each Shipment keeps the origin evidence recorded with its Order; later configuration applies only to future quotes and Orders."
>
> **Dev:** "The seller holds stock in more than one place. Does the seller choose where a parcel ships from?"
> **Domain expert:** "No. Seller Fulfillment is single-source: every Inventory Item is backed by one internal default seller Stock Pool, and a Shipment carries the origin evidence recorded with its Order."
>
> **Dev:** "The buyer prints nothing and the seller just prints a label. Is the parcel Dispatched?"
> **Domain expert:** "No. A prepared Shipment or printed label is not Dispatched; only carrier handover is, and a label alone does not prevent a Confirmed Cancellation."
>
> **Dev:** "The buyer received two units, one is confirmed canceled, and one is lost. Is the Order fulfilled?"
> **Domain expert:** "No. Fulfillment Progress requires every non-canceled unit delivered; the canceled unit stays canceled and the lost unit is a Delivery Exception, not a fulfilled quantity."
>
> **Dev:** "A shop Order's groups have different delivery windows. How is that shown?"
> **Domain expert:** "Each Fulfillment Group has its own Delivery Promise, and the buyer sees the combined Order Shipping Charge."
>
> **Dev:** "A provider delay produces a new delivery forecast."
> **Domain expert:** "That is a Revised Estimate. The buyer's accepted Delivery Promise and agreed Shipping Charge are retained unchanged."
>
> **Dev:** "Cancellation is requested, then the seller dispatches those same units."
> **Domain expert:** "Confirmed Cancellation and dispatch are mutually exclusive for the same units, and eligibility is decided when the outcome is accepted."
>
> **Dev:** "Every unit in the Order is confirmed canceled."
> **Domain expert:** "Then the Order is canceled, not fulfilled."
>
> **Dev:** "The seller reports delivery and a carrier report later disagrees."
> **Domain expert:** "Each Shipment Update keeps its actor and source, and the conflicting history is retained rather than overwritten."
>
> **Dev:** "A parcel comes back to the seller, or the seller sends a replacement after a loss."
> **Domain expert:** "A Return to Sender is not a Buyer Return; an authorized Replacement Fulfillment links to the original obligation and does not reset dispatched quantities."

## Flagged Ambiguities

- "Fulfillment" is not shipment status. A Shipment's journey describes one consignment; Fulfillment Progress accounts quantities across a group or Order.
- "FBA" is Amazon's provider fulfillment specifically, not Arc-operated fulfillment or provider fulfillment in general.
- "Fulfilled" is not the commercial Order state. Fulfillment Progress does not decide payment, refund, or commercial completion, and an all-canceled Order is canceled rather than fulfilled.
- Provider Fulfillment is a target capability: only Seller Fulfillment is operational initially, and no provider selection is currently exposed.
- A carrier is not automatically a Fulfillment Operator; providing carriage alone does not establish fulfillment responsibility.
- "Reserved" is not "dispatched": a Checkout Reservation is a pre-confirmation Inventory hold, while dispatch is physical carrier handover.
- A "delivery option" is not a warehouse or a stock source.
- "Replacement" does not reset dispatched quantities to unshipped.
- Current per-Order shipping details and tracking are retained as immutable legacy evidence only. Fulfillment Groups with quantity-based Shipments are implemented for Seller Fulfillment: newly confirmed Orders receive a persisted seller group, and one Order Item's quantity can be split across multiple Shipments, with preparation, dispatch, transit, and delivery recorded as per-Shipment journey updates. Quantity progress reports prepared, dispatched, delivered, and canceled units against the remaining obligation, and Confirmed Cancellation is mutually exclusive with dispatch for the same units. Seller Fulfillment is single-source: each Inventory Item has one internal default seller Stock Pool, so no Warehouse, allocation, source selector, or reassignment exists. Product shipping behavior is configured through reusable Shipping Profiles (tickets 08-09), not a shop-level dispatch address. Provider Fulfillment, multi-location sourcing, reassignment, and exceptions/estimates are explicitly deferred and are not represented by runtime abstractions.
