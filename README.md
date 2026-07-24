# Arc
A ecommerce marketplace workspace where people come together to make, sell, buy, and collect unique items.

## Product Capabilities

- **Multi-vendor marketplace** - multiple sellers can operate shops and sell products through one customer storefront
- **Multi-currency pricing** - catalog and checkout flows support market-aware prices, presentment currency, and exchange-rate metadata
- **Product search** - customers can search products with suggestions, filters, facets, and category navigation
- **Product recommendations** - storefront flows surface best sellers, trending products, recently viewed items, and related products
- **Product reviews** - customers can rate purchased products, write reviews, and upload review images
- **Cart and checkout** - customers can manage carts, apply coupons, choose shipping, and place cart or buy-now orders
- **Payments and refunds** - customers can complete purchases and request refunds when needed
- **Customer account management** - users can manage profiles, addresses, notifications, orders, reviews, and guest order tracking
- **Notifications** - customers and sellers can receive commerce events such as order updates, shipment changes, support activity, and low-stock alerts
- **Seller shop operations** - sellers can create shops, manage products, publish listings, update inventory, and configure shipping
- **Promotion management** - sellers can create promo codes and sale campaigns for selected products
- **Order fulfillment workflows** - sellers can review orders, update shipment state, handle cancellations, and process refunds
- **Buyer-seller messaging** - customers and sellers can chat with conversation history, unread counts, and realtime delivery

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

## Contact

For any inquiries or feedback, feel free to contact [me](mailto:dauphaihau@gmail.com).
