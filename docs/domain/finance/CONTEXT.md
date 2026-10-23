# Arc Finance

Arc Finance records the money consequences of confirmed Orders: what Arc owes each shop, what Arc keeps, and what has been paid out.

## Language

**Money Event**:
A confirmed fact about money the payment provider has already moved — a paid Order, an Order Refund, or a Manual Adjustment — from which a Journal Entry is recorded.
_Avoid_: Transaction, Payment

**Journal Entry**:
The immutable record of one Money Event's money consequences, identified by the Money Event that caused it.
_Avoid_: Transaction, Ledger Row

**Posting**:
One line of a Journal Entry moving an amount into or out of one Ledger Account. A Journal Entry's Postings sum to zero in their currency.
_Avoid_: Entry Line

**Ledger Account**:
A named position that Postings accumulate into, such as platform clearing, a shop's Seller Payable, Platform Commission, shipping revenue, or refunds.
_Avoid_: Wallet, Bucket

**Seller Payable**:
The amount Arc owes a shop, in the shop's Settlement Currency, from its confirmed Orders after Platform Commission and Order Refunds.
_Avoid_: Balance, Earnings, Wallet

**Available Seller Payable**:
The part of a shop's Seller Payable whose Payout Availability has been recorded, and the only part a Payout Run may pay. Everything else is not-yet-available Seller Payable.
_Avoid_: Cleared Balance, Withdrawable, Matured Balance

**Settlement Currency**:
The shop's currency at the time a Money Event is recorded, held as a dated history. Seller Payable accrues in it.
_Avoid_: Wallet Currency, Payout Currency, Base Currency

**Cutover Rate**:
The recorded rate at which a retired Settlement Currency converts into the shop's new one. Reversals of Money Events recorded before the change convert through it.
_Avoid_: Migration Rate, Change Rate

**Platform Commission**:
The platform's share of an Order, fixed when the Order's Money Event is recorded and never recomputed from later commission rules.
_Avoid_: Fee, Take Rate, Cut

**Commission Schedule**:
The effective-dated, per-shop record that Platform Commission is resolved from when a Money Event is recorded.
_Avoid_: Fee Table, Rate Card

**Order Refund**:
Money returned to a buyer against one Order, in an amount no greater than that Order's paid amount. A refund is partial or full; it is never negative and never exceeds what was paid.
_Avoid_: Reversal, Credit Note

**Payout**:
The transfer of a shop's Available Seller Payable, in its Settlement Currency, to that shop's Payout Account.
_Avoid_: Withdrawal, Disbursement, Settlement

**Payout Run**:
One scheduled execution that pays every shop whose Seller Payable is available in it.
_Avoid_: Batch, Payout Cycle

**Payout Availability**:
Finance's own decision that an Order's Seller Payable may be paid, recorded when it happens; it follows delivery of every required unit of the Order and is not read from the Order's commercial status.
_Avoid_: Maturity, Clearing Date

**Payout Minimum**:
The amount below which a Payout Run does not pay a shop; the unpaid amount is carried into the next run.
_Avoid_: Threshold, Floor

**Payout Account**:
The Finance-owned record of where a shop's Payouts are sent: its type, provider reference, holder, bank name, country, and the currency, which is the shop's Settlement Currency. A shop has at most one active Payout Account, and a Payout keeps pointing at the account it paid.
_Avoid_: Bank Account, Payout Method, Payout Profile, Payout Destination

**Blocked Payout**:
A Payout a run skipped because the shop has no usable Payout Account. It is not a failed Payout, and it leaves Seller Payable unchanged.
_Avoid_: Failed Payout, Held Payout

**Manual Adjustment**:
A money fact recorded by an operator instead of derived from an Order, carrying the operator's identity and a mandatory reason. It is the only way a Reconciliation Exception is resolved.
_Avoid_: Correction, Journal Fix, Write-off

**Provider Fee**:
What the payment provider charges for moving money, recorded as a platform expense from the provider's own record. It never reduces a Seller Payable.
_Avoid_: Processing Cost, Transaction Fee

**Reconciliation Exception**:
A recorded disagreement between a Ledger Account and the provider's record. It stays visible until a Manual Adjustment with a reason closes it; a Posting is never edited to close one.
_Avoid_: Discrepancy, Mismatch

**Reconciliation**:
The recurring comparison of Ledger Account positions against the payment provider's own record of money moved.
_Avoid_: Audit, Sync

## Relationships

- An Order's money is converted once, into the shop's Settlement Currency, when its Money Event is recorded, at a rate frozen on the Journal Entry with its source and effective time.
- An Order paid in its shop's Settlement Currency records no rate and no conversion.
- An Order Refund reverses at the rates frozen on the original Journal Entry, so a refund never moves a seller's balance by an exchange rate.
- Platform Commission is charged on an Order's discounted item subtotal only; the shipping charge passes to Seller Payable in full, all in the Settlement Currency.
- An Order Refund reverses the same share of Platform Commission that it reverses of the Order.
- Every discount is borne by the seller until funding is decided.
- A shop may change its Settlement Currency only while it holds no Seller Payable in any state, available or not-yet-available, and no negative position, and no Payout is pending.
- A change is allowed at most once per 90 days per shop, and it turns automatic payouts off until a Payout Account in the new currency is recorded and verified.
- Every Settlement Currency change is recorded with its effective date and Cutover Rate; recorded money is never re-denominated, and prices are never relabelled.
- Exchange-rate movement between recording and payout sits with the platform and is recorded when a Payout converts collected money.
- A Payout Run happens at one platform time for every shop, never at each shop's local time.
- One shop's Payout outcome never determines another's within the same Payout Run.
- A Ledger Account is one kind in one currency, and a shop has one Seller Payable account in its current Settlement Currency.
- A Payout Run produces one Payout per shop, in its Settlement Currency; nothing is converted at payout time.
- A shop has at most one active Payout Account at a time, and a past Payout keeps pointing at the Payout Account it paid.
- A seller sees Available Seller Payable and the not-yet-available remainder as separate figures.
- A Reconciliation Exception is closed only by a Manual Adjustment; Postings are never edited or deleted.
- One Order Refund records one provider refund, and several partial Order Refunds may belong to one Order.
- Payout Availability is decided per Order, never per Order Item or per shop.
- A Payout Run pays a shop at most once per day, and only when the shop's Available Seller Payable reaches its Payout Minimum.
- Order Refunds can make Seller Payable negative, and a Payout Run nets a negative position before paying anything.
- Reconciliation compares Ledger Accounts against the provider's record and never rewrites a Journal Entry.
- An Order's confirmed money facts are the source of its Journal Entries; Finance never re-prices an Order.
- One Money Event produces exactly one Journal Entry, and exactly one Journal Entry records a given Money Event.
- Every Posting belongs to exactly one Ledger Account, and a Journal Entry's Postings balance to zero in each currency they are denominated in.
- Seller Payable is the sum of a shop's Postings; it is projected from them and never stored as an authority.
- Platform Commission is fixed into a Posting and is not recalculated when commission rules change.
- A Payout discharges the Seller Payable it pays; a failed Payout leaves Seller Payable unchanged.
- Finance records money the payment provider has already moved; it never decides whether a payment succeeds.

## Flagged Ambiguities

- "Balance" always means the projected Seller Payable. There is no stored balance that can disagree with the Postings.
- "Transaction" is not used in Finance: the payment provider already owns that word, and Ordering owns confirmed purchase facts.
- Which party funds a discount is undefined. Until it is decided, every discount is borne by the seller: Platform Commission is charged on the discounted subtotal and the whole discount reduces Seller Payable, with no distinction between a seller-funded Sale and a platform-funded Promotion.
- "Order Refund" is the money returned to a buyer; the Order's own refund state remains an Ordering fact.
- "Completed" is an Ordering status derived from delivery. Payout Availability is Finance's own recorded decision and is not the same fact as that status.
- The shop's currency serves as both the currency a seller types prices in and its Settlement Currency; no separate settlement setting exists.
- "Account" is never written bare in Finance: a Ledger Account holds Postings, while a Payout Account is where a Payout is sent.
