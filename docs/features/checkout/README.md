# Checkout

## Overview

Checkout lets a buyer turn selected storefront items into one or more orders through a backend-owned quote, then complete payment by card or cash on delivery.

The feature covers both cart checkout and buy-now checkout. It supports authenticated buyers and guest buyers, with different identity and address inputs but the same quote-first order creation boundary.

Checkout is buyer-facing order creation. It is not seller order management, post-purchase fulfillment, refund handling, payment reconciliation, or catalog price authoring.

## Goals

- Let buyers review shipping, payment, shop notes, promo codes, and final totals before order creation.
- Keep final money amounts owned by the backend checkout quote.
- Create orders from a `quote_id`, not from client-authored totals.
- Route card payments to a payment-provider checkout session.
- Confirm cash orders immediately after local order creation.
- Give guest buyers enough information to track the created order shops.
- Recover clearly when the quote expires, the cart changes, or inventory can no longer be reserved.

## Scope

In scope:

- Storefront cart checkout from selected cart items.
- Storefront buy-now checkout from a temporary cart id.
- Authenticated checkout with a saved user address.
- Guest checkout with a shipping address and email.
- Checkout quote creation before order creation.
- Card and cash payment choices.
- Success page behavior for card return and cash order confirmation.
- Checkout failure handling for expired quotes, changed carts, and inventory reservation failures.

Out of scope:

- Seller-facing order management.
- Order cancellation, support requests, fulfillment, and review workflows after purchase.
- Payment webhook processing details.
- Stripe session creation internals.
- Quote persistence schema details.
- Catalog pricing and FX administration.
- Tax remittance and accounting policy.

## Users

- Authenticated buyer: checks out using account identity and a saved address.
- Guest buyer: checks out using shipping address and email, then tracks orders through guest lookup.

## Entry Points

- Cart page to cart checkout: `/cart` -> `/cart/checkout`.
- Buy-now action to buy-now checkout: `/checkout?c=:cart_id`.
- Card payment return to success page: `/success?session_id=:checkout_session_id`.
- Cash order confirmation to success page with in-memory order shops for authenticated buyers.
- Guest cash order confirmation to success page with guest email, ZIP, and order numbers.

## Main Flow

1. Buyer enters checkout from cart or buy-now.
2. Storefront collects or confirms shipping address.
3. Storefront collects payment type.
4. Storefront shows review and confirmation.
5. Storefront submits a checkout quote request.
6. API resolves checkout currency, totals, shipping, discounts, and item snapshots.
7. API returns `quote_id`, quoted totals, quoted items, checkout currency, and expiration.
8. Storefront submits order creation with `payment_type` and `quote_id`.
9. For card payment, API returns a payment checkout session URL and storefront redirects externally.
10. For cash payment, API returns created order shops and storefront routes to success.
11. Success page displays created order shops and gives the buyer a view or track order action.

## Checkout Modes

### Cart Checkout

Cart checkout starts from selected cart items.

The quote request may include:

- saved user address id for authenticated buyers
- guest shipping address for guest buyers
- guest presentment currency
- shop-level promo codes
- shop-level notes

The current storefront route is `/cart/checkout`.

### Buy-Now Checkout

Buy-now checkout starts from a temporary cart id passed as `c` in the checkout route.

The quote request may include:

- temporary cart id
- saved user address id for authenticated buyers
- guest shipping address for guest buyers
- guest presentment currency
- promo codes
- note

The current storefront route is `/checkout?c=:cart_id`.

### Authenticated Checkout

Authenticated checkout uses the current user session and a saved address id. The order creation request does not need guest identity.

After cash order creation, the buyer goes to the success page and can view orders from the authenticated orders route.

### Guest Checkout

Guest checkout sends a full shipping address during quote creation and guest email during order creation.

After cash order creation, the success page preserves guest email, ZIP, and order numbers so the buyer can track created orders through guest order lookup.

## Payment Behavior

### Card Payment

Current storefront behavior expects order creation to return `checkout_session_url` for card payment. The storefront redirects the buyer to that external URL.

The API response schema also allows `checkout_pending`, which aligns with the target transactional outbox design where card checkout session creation can become asynchronous.

The frontend and API contract need one explicit product decision:

- keep a synchronous facade that returns `checkout_session_url`
- or adopt asynchronous card checkout preparation with a pending state and session-readiness polling

### Cash Payment

Cash payment creates local orders without a payment-provider checkout session.

After order creation succeeds, the storefront stores returned order shops in checkout state and routes to the success page.

## UI States

### Default

- Checkout shows stepper navigation for address, payment, and review.
- Buyer can continue only after a checkout address is selected or entered.
- Review step submits final order creation.

### Loading

- Create order button shows pending state while quote and order creation are in progress.
- Success page shows a loading state when resolving order shops by card checkout session id.
- Success page shows a server wake-up state for transient `502`, `503`, or `504` responses while loading session order shops.

### Empty

- Cart checkout should not proceed when there are no selected or available cart items.
- Success page returns a not-found error when it has neither a checkout session id nor in-memory order shops.

### Error

- Expired quote shows quote-expired copy and refreshes cart data.
- Changed cart shows cart-changed copy and refreshes cart data.
- Stock unavailable shows stock failure copy and refreshes cart data.
- Released reservation shows reservation failure copy and refreshes cart data.
- Unknown backend failures show backend error text when available.

### Success

- Card success resolves order shops from checkout session id.
- Cash success displays order shops returned from order creation.
- Authenticated buyers can navigate to orders.
- Guest buyers can navigate to guest order tracking.

## Key Invariants

- Checkout quote is the pricing checkpoint between mutable cart state and immutable order state.
- Storefront may request a quote, but it does not author final totals.
- Order creation must consume a `quote_id`.
- Payment creation must use quote-owned totals.
- A quote belongs to the correct authenticated user or guest session context.
- Expired quotes cannot create orders.
- Cart checkout order creation must reject cart changes that invalidate the quoted selection.
- Card checkout must not mark success until created orders can be resolved from payment session or local order state.

## Edge Cases

- Quote expires after review but before order creation.
- Buyer changes selected cart items, quantities, promo codes, notes, or shipping inputs after quote creation.
- Inventory is no longer available or reservation is released.
- Card checkout session URL is missing from a synchronous card response.
- Payment provider redirects back before success page can resolve order shops.
- Guest buyer loses browser state after cash order creation.
- Backend wakes up slowly while the success page resolves card order shops.
- Buyer refreshes success page after in-memory cash order shops are cleared.

## Responsive And Accessibility Notes

- Checkout steps should remain usable on mobile, tablet, and desktop.
- Primary actions should have disabled and loading states that do not shift layout.
- Payment options should be keyboard reachable and screen-reader understandable.
- Error toasts should use specific recovery copy and should not rely only on color.
- Success actions should clearly distinguish authenticated order viewing from guest order tracking.

## Implementation Notes

- Storefront checkout routes are CSR according to the storefront Nuxt route rules.
- Cart checkout uses `/checkout/cart/quote` then `/checkout/cart` for guest APIs, and the corresponding authenticated `me` order mutations for signed-in buyers.
- Buy-now checkout uses `/checkout/buy-now/quote` then `/checkout/buy-now` for guest APIs, and the corresponding authenticated `me` order mutations for signed-in buyers.
- Quote responses include `quote_id`, presentment currency, checkout currency, totals, expiration, and item snapshots.
- Order responses may include `checkout_session_url`, `checkout_pending`, and created `order_shops`.
- Backend quote ownership is documented in [Checkout Quote Design](../../../apps/api/api/docs/features/checkout-quote-design.md).
- Backend card side-effect durability is documented in [Transactional Outbox For Checkout](../../../apps/api/api/docs/features/checkout-transactional-outbox.md).
- Pricing boundaries are documented in [Multi-Currency Pricing](../multi-currency-pricing/README.md).

## Acceptance Criteria

- [ ] Cart checkout creates a quote before creating an order.
- [ ] Buy-now checkout creates a quote before creating an order.
- [ ] Authenticated checkout sends saved address identity during quote creation.
- [ ] Guest checkout sends shipping address during quote creation and email during order creation.
- [ ] Order creation sends `quote_id` and `payment_type`.
- [ ] Card checkout redirects only when a checkout session URL is available, or shows an explicit pending state if asynchronous checkout is adopted.
- [ ] Cash checkout routes to success after order shops are returned.
- [ ] Guest success path preserves enough lookup context for order tracking.
- [ ] Quote-expired, cart-changed, stock-unavailable, and reservation-unavailable failures show specific recovery copy.
- [ ] Cart data refreshes after recoverable checkout invalidation failures.
- [ ] Success page can resolve card-created order shops from checkout session id.

## Open Questions

- [ ] Should card checkout expose asynchronous `CHECKOUT_PENDING` behavior to the storefront, or keep the current synchronous `checkout_session_url` facade?
- [ ] If asynchronous card checkout is adopted, which endpoint should the storefront poll for checkout session readiness?
- [ ] Should quote totals be rendered from the quote response on review, rather than only relying on cart display totals before submission?
- [ ] How long should guest success lookup context remain available after browser state is cleared?
- [ ] Should checkout support order recovery when a guest cash success page is refreshed without query parameters?
