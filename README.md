# Arc
A ecommerce marketplace workspace where people come together to make, sell, buy, and collect unique items.

![Arc storefront homepage](./assets/screenshots/storefront-homepage.png?v=fcd84f15d5d2)
[See more storefront screenshots](./docs/screenshots/storefront.md)

![Arc seller dashboard](./assets/screenshots/seller-dashboard.png?v=5fa3ec7558d3)
[See more seller screenshots](./docs/screenshots/seller.md)

## Demo
| App | URL |
| --- | --- |
| Storefront | https://arc.hautran.me |
| Seller | https://seller.arc.hautran.me |
| Admin | https://admin.arc.hautran.me |
| API Docs | https://api.arc.hautran.me/docs |
- [Demo guide](./docs/operations/demo-guide.md) - cross-environment demo accounts, smoke checks, and test flows.

## Product Capabilities

- **Multi-vendor marketplace** - multiple sellers can create shops and sell products through one customer storefront
- **Multi-currency pricing** - catalog and checkout flows support market-aware prices, presentment currency, exchange-rate metadata, and a seller settlement currency
- **Product catalog and inventory** - sellers manage physical or digital products, option-based variants and SKUs, images, stock levels, and AI-assisted product descriptions
- **Bulk product import** - sellers create product drafts in bulk from an XLSX template with row-level validation and a downloadable import report
- **Shipping profiles** - sellers define reusable shipping profiles with destination rates and delivery times, then assign a profile to each physical product
- **Product search** - customers can search products with suggestions, filters, facets, and category navigation
- **Product recommendations** - storefront flows surface best sellers, trending products, recently viewed items, and related products
- **Product reviews** - customers can rate purchased products, write reviews, and upload review images
- **Cart and checkout** - guest and signed-in customers can manage carts, apply promo codes, review shipping, and place cart or buy-now orders with card or cash-on-delivery payment
- **Payments and refunds** - card payments run through Stripe; customers can cancel before shipment, and sellers or admins can issue and retry refunds
- **Promotion management** - sellers can run scheduled percentage sales and create promo codes with product scope, redemption limits, and visibility
- **Customer account management** - users can manage profiles, addresses, notifications, orders, and reviews, and guests can track orders
- **Buyer-seller messaging** - customers and sellers can chat with conversation history, unread counts, and realtime delivery
- **Notifications** - customers and sellers receive in-app notifications for orders, fulfillment changes, refunds, support requests, payout outcomes, and completed exports; storefront customers can also enable web push
- **Seller dashboard** - sellers can track revenue, orders, units sold, average order value, recent orders, and top products over selectable periods
- **Order fulfillment workflows** - sellers can prepare, update, void, and reconcile shipments, handle cancellations, and process refunds
- **Order export** - sellers can export filtered orders to CSV, with direct downloads for small exports and background exports with progress and private download links
- **Seller finance and payouts** - sellers can view available and pending Seller Payable, onboard a payout account, review transactions and payout history, receive scheduled or manual payouts, and manage settlement currency
- **Admin back office** - role-gated operators can manage shops and shop-level finance, update order status, refunds, and support notes, administer users and roles, and review an audit trail

For technical implementation details, see:

- [Backend implemented patterns and capabilities](./apps/api/README.md#implemented-patterns-and-capabilities)
- [Frontend implemented patterns and capabilities](./apps/web/README.md#implemented-patterns-and-capabilities)

## Tech stack

#### Frontend:
- [Nuxt.js](https://nuxt.com/) - Vue Framework
- [Vue Query](https://tanstack.com/query/latest/docs/framework/vue/overview) - Managing and caching asynchronous data
- [Pinia](https://pinia.vuejs.org/) - State management
- [Typescript](https://www.typescriptlang.org/) - Static Type Checking
- [Tailwind](https://tailwindcss.com/) - Utility-first CSS framework
- [Zod](https://zod.dev/) - TypeScript-first schema declaration and validation
- [Vitest](https://vitest.dev/) - Testing Framework

#### Backend:
- [NestJS](https://nestjs.com/) - Node.js backend framework
- [PostgreSQL](https://www.postgresql.org/) - Primary database
- [MikroORM](https://mikro-orm.io/) - ORM and migrations
- [JWT](https://jwt.io/) - Authentication
- [BullMQ](https://bullmq.io/) - Background jobs and workers
- [Redis](https://redis.io/) - Cache, queues, and rate limiting
- [MongoDB Atlas Search](https://www.mongodb.com/products/platform/atlas-search) - Optional catalog read-model search and recommendations
- [AWS S3](https://aws.amazon.com/s3/) - Object storage
- [Stripe](https://stripe.com/) - Payments
- [Typescript](https://www.typescriptlang.org/) - Static type checking
- [Zod](https://zod.dev/) - Schema validation

## Workspace Structure

This repository is the parent workspace for the Arc project. The application code lives in separate Git submodules so each app keeps its own history and remote repository.

- Parent workspace: [dauphaihau-arc/arc](https://github.com/dauphaihau-arc/arc)
- Frontend app: [apps/web](apps/web) - [dauphaihau-arc/web](https://github.com/dauphaihau-arc/web)
- Backend app: [apps/api](apps/api) - [dauphaihau-arc/api](https://github.com/dauphaihau-arc/api)

## Setup
```bash
git clone --recurse-submodules https://github.com/dauphaihau-arc/arc.git
cd arc
git submodule update --init --recursive
```

- [Local setup](./docs/local-setup.md) - install dependencies, seed demo data, and run the API, storefront, and seller app locally.

## Contact

For any inquiries or feedback, feel free to contact [me](mailto:hi@hautran.me).
