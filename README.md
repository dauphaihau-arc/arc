# Arc
A ecommerce marketplace workspace where people come together to make, sell, buy, and collect unique items.

This repository is the parent workspace for the Arc project. The application code lives in separate Git submodules so each app keeps its own history and remote repository.

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
- [AWS S3](https://aws.amazon.com/s3/) - Object storage
- [Stripe](https://stripe.com/) - Payments
- [Typescript](https://www.typescriptlang.org/) - Static type checking
- [Zod](https://zod.dev/) - Schema validation

## Features
- Marketplace web application
- Separate frontend and backend repositories managed as Git submodules
- Frontend app in [apps/web](./apps/web)
- Backend app in [apps/api](./apps/api)

## Repositories
- Parent workspace: [dauphaihau-arc/arc](https://github.com/dauphaihau-arc/arc)
- Frontend: [dauphaihau-arc/web](https://github.com/dauphaihau-arc/web)
- Backend: [dauphaihau-arc/api](https://github.com/dauphaihau-arc/api)

## Setup
```bash
git clone --recurse-submodules https://github.com/dauphaihau-arc/arc.git
cd arc
git submodule update --init --recursive
```

## Contact

For any inquiries or feedback, feel free to contact [me](mailto:dauphaihau@gmail.com).
