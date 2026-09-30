# Arc Promotions

Arc Promotions defines seller offers that reduce product prices or provide discounts at checkout.

## Language

**Promotion**:
A seller offer that provides a Sale or a Checkout Discount.
_Avoid_: Coupon as the umbrella term

**Sale**:
An automatic, scheduled percentage reduction in a Product's price with no redemption code, minimum order spend, minimum purchase quantity, or buyer-specific redemption limit.
_Avoid_: Auto-sale Coupon, Promo Code

**Checkout Discount**:
An offer applied during checkout whose benefit may depend on eligibility conditions rather than being an unconditional product-price reduction.
_Avoid_: Sale

**Promo Code**:
A code that activates a Checkout Discount, whether entered by the buyer or applied by selecting a publicly listed offer.
_Avoid_: Sale Name

**Promo Code Visibility**:
Whether a Promo Code is publicly discoverable in checkout or unlisted and shared directly; visibility does not change redemption eligibility.
_Avoid_: Eligibility, Automatic Application

**Promotion Product Scope**:
The Products eligible for a Promotion: either all of the shop's Products, including future Products, or explicitly selected Products. An eligible Product includes all its purchasable Product Variants.
_Avoid_: Variant Targeting

**Sale Price**:
The current regular Product price after the highest matching active Sale percentage is applied, without compounding overlapping Sales or freezing the regular price. An eligible product-discount Promo Code further discounts this price at checkout.
_Avoid_: Final Checkout Total

**Product-discount Promo Code**:
A Promo Code granting a percentage or fixed-amount Checkout Discount. A shop permits one such code alongside one free-shipping Promo Code.
_Avoid_: Sale

**Eligible Merchandise Subtotal**:
The value of a Promotion's targeted merchandise within its shop after Sale discounts and before Promo Code discounts, excluding shipping and tax. This is the basis for its minimum-spend condition and the maximum merchandise value a product-discount Promo Code can discount.
_Avoid_: Order Total

**Minimum Eligible Quantity**:
The minimum number of targeted merchandise units required for a Promo Code, counting quantities rather than distinct Products.
_Avoid_: Minimum Product Count

**Free-shipping Promo Code**:
A shop-wide Promo Code that waives the owning shop's Shipping Charge, subject to any configured minimum on that shop's eligible merchandise. It does not target individual Products or waive other shops' Shipping Charges.
_Avoid_: Product Sale

**Promotion Period**:
The interval during which a Promotion may apply, including its start instant and excluding its end instant. Adding a Product to a cart does not preserve eligibility beyond this period.
_Avoid_: Cart Price Lock

**Promotion Currency**:
The currency in which a Promotion's monetary values were defined at creation; a later change to the shop's currency does not redefine those values.
_Avoid_: Buyer Currency

**Promo Code Redemption**:
The consumption of a Promo Code's allowance when an Order is successfully committed, not when the code is selected or a quote is obtained. Cancellation or refund does not restore the allowance, and retrying the same committed Order does not consume it again.
_Avoid_: Code Selection, Redemption Reservation

**Per-buyer Redemption Limit**:
The maximum number of redemptions permitted for one authenticated buyer account. Browser identity or unverified email does not establish the buyer for this limit.
_Avoid_: Per-device Limit

**Promotion Cancellation**:
The irreversible stopping of a scheduled Promotion before it starts, retaining its definition and references.
_Avoid_: Promotion Deletion, Pause

**Early Promotion Ending**:
The irreversible stopping of an active Promotion for subsequent purchases without changing committed Order prices or removing its history.
_Avoid_: Promotion Deletion, Pause

**Exhausted Promo Code**:
A Promo Code whose total redemption allowance has been consumed, regardless of whether its Promotion Period is still active.
_Avoid_: Ended Promotion

**Zero-benefit Promo Code**:
A Promo Code that provides no monetary saving for the current checkout, including after rounding. It is ineligible and cannot consume a redemption.
_Avoid_: Successful Redemption
