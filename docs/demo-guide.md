# Demo Guide

Use this guide to test Arc demo environments, including the deployed demo sites and locally seeded demo data. Keep checks focused, reversible, and safe for shared data and real integrations.

## Sites

| App | Deployed | Local |
| --- | --- | --- |
| Storefront | https://arc.hautran.me | http://localhost:4000 |
| Seller | https://seller.arc.hautran.me | http://localhost:4001 |

## Suggested Test Accounts

Use these seeded demo accounts when testing the deployed demo sites. All listed accounts use password `Password123!`.
Each row lists more than one account where possible, so testers can switch accounts if one account state was changed during testing.

### Seller Accounts

Use these accounts on seller site.

| Accounts | Shops | Currencies | Best For |
| --- | --- | --- | --- |
| `bulk.catalog@example.com` | Bulk Catalog Lab | SGD | Large table testing: product list, coupon list, pagination, filtering, sorting, and CRUD flows. Has 50 draft products and 50 coupons. |
| `maker.olive@example.com`<br>`maker.mason@example.com`<br>`maker.sage@example.com` | Olive Atelier, Reed Workshop, Sage Studio | USD, EUR, JPY | Dashboard overview, revenue/order widgets, active product state, messages, reviews, and normal seller smoke checks. |
| `maker.sage@example.com`<br>`maker.juno@example.com` | Sage Studio, Juno Console | JPY, AUD | Currency-specific testing. Use Sage Studio for JPY formatting and zero-decimal currency assumptions; use Juno Console as a second non-USD currency check. |
| `maker.mason@example.com`<br>`maker.olive@example.com`<br>`maker.juno@example.com` | Reed Workshop, Olive Atelier, Juno Console | EUR, USD, AUD | Messages, reviews, and order checks across shops. Reed Workshop and Olive Atelier have stronger review coverage; Juno Console has buyer message coverage. |

### Storefront Customer Accounts

Use these accounts on storefront site.

| Accounts | Best For |
| --- | --- |
| `member@example.com`<br>`buyer.aiden@example.com`<br>`buyer.luna@example.com` | Main customer smoke checks. `member@example.com` has seeded cart items, completed demo orders, conversations, and reviews across multiple shops. |
| `buyer.aiden@example.com`<br>`buyer.luna@example.com`<br>`buyer.mason@example.com` | Olive Atelier review and purchase-eligibility checks, especially Canvas Market Tote. |
| `buyer.sophia@example.com`<br>`buyer.ethan@example.com`<br>`buyer.chloe@example.com` | Reed Workshop review and purchase-eligibility checks for electronics products. |
| `buyer.jack@example.com`<br>`buyer.zoe@example.com`<br>`buyer.nolan@example.com` | Sage Studio review and purchase-eligibility checks, useful when testing JPY product display from the storefront side. |
| `buyer.seed0001@example.com` through `buyer.seed0050@example.com` | Pool of lightweight review/order eligibility accounts generated around seeded product review coverage. Use these when you need many distinct customer accounts or a clean backup. |
| `seed.buyer.0001@example.com` through `seed.buyer.0320@example.com` | Pool of generated bulk-order customers. Use for order-history volume and bestseller/order-data coverage if these accounts are present in the deployed environment. |

### Guest Testing

No login account is needed for guest checks.

Use guest mode to test:

- Anonymous browsing
- Product search and product detail
- Guest cart add, update, and remove
- Cart persistence after refresh
- Guest checkout start
- Guest order lookup when you have an order email and order number from a created guest checkout

Seeded product view history includes guest-session traffic for trending and recommendation checks, but those guest sessions are backend seed data, not login accounts.

## Test Rules

### Safety Rules

- Do not test destructive admin actions against real shops, customers, orders, or payments.
- Do not send bulk notifications, bulk emails, or push campaigns.
- Do not run database seed, clear, migration rollback, or storage reset commands against shared or production-like environments.
- Use Stripe test cards only when the environment is configured for Stripe test mode; never use real cards for routine demo testing.
- Use sandbox payment credentials unless the release explicitly requires a live-payment verification.
- Use clearly labeled test products, test shops, and test orders when the environment allows write checks.
- Clean up test-only content after the release check if it is visible to users.

## Test Checklists

### Smoke Checklist

- Storefront loads without console-visible runtime errors.
- Seller app loads and redirects unauthenticated users to sign-in.
- API readiness endpoint is healthy.
- API docs or OpenAPI endpoint is reachable only as intended for the environment.
- Images and static assets load from the configured host.
- Locale, currency, and market settings resolve correctly.
- Login, logout, session refresh, and protected-route redirects work.
- Error tracking and logs receive new errors with enough request context to debug.

### Storefront Flow

- Browse home, category, search, product detail, and shop pages.
- Add an item to cart, update quantity, remove item, and reload the page.
- Sign in as a customer and confirm cart/account state.
- Continue with the dedicated checkout checklist when testing purchase behavior.
- Submit or view a product review only on approved test products.
- Send a buyer-to-seller message and confirm it appears in the conversation.

### Checkout Flow

- Add one item from a single shop to cart.
- Add multiple items from the same shop to cart.
- Confirm quantity update, item removal, subtotal, shipping, discount, and total.
- Apply a valid coupon when available.
- Try an invalid or expired coupon and confirm the error state.
- Start checkout as a guest.
- Start checkout as a signed-in customer.
- Use Stripe test card `4242 4242 4242 4242` with any future expiry date, any CVC, and any postal code when the environment uses Stripe test mode.
- Complete payment only with sandbox or approved test payment credentials.
- Confirm success page loads after payment.
- Confirm the new order appears in the customer order list.
- Confirm the new order appears in the seller order list.
- Confirm inventory or reservation behavior looks correct after order creation.
- Confirm order notification or email behavior if enabled for the target environment.

### Seller Flow

- Sign in as a seller.
- Open dashboard, product list, product detail, inventory, orders, promotions, and messages.
- Create or edit only an approved test product.
- Confirm validation errors for required product fields.
- Update inventory for a test listing.
- Review an order and confirm status, shipping, refund, and cancellation controls are visible according to permissions.
- Reply to a buyer message from the seller app.

### Integration Checks

- Payments use the expected live or sandbox mode.
- Email delivery is either sandboxed or approved for the test recipients.
- Object storage serves new and existing images.
- Search and recommendations return fresh enough product data.
- Queues process expected background jobs.
- WebSocket or SSE notifications connect and receive events.
