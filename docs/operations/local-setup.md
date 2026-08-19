# Local Setup

Use this guide to run Arc on a local machine with seeded demo data.

## Prerequisites

- Git with submodule support
- Docker and Docker Compose
- `just`
- Node.js `22.x` and PNPM `9.x` for the API
- Node.js `20.20.2` and PNPM `9.x` for the web workspace

## Clone And Submodules

```bash
git clone --recurse-submodules https://github.com/dauphaihau-arc/arc.git
cd arc
git submodule update --init --recursive
```

## Local URLs

| App | URL |
| --- | --- |
| API | `http://localhost:3000` |
| Storefront | `http://localhost:4000` |
| Seller | `http://localhost:4001` |
| Grafana | `http://localhost:3001` |

## API Setup

Run API commands from `apps/api`.

```bash
cd apps/api
just infra-up
just api-install
cp api/.env.example api/.env
just db-migration-up
just seed-full
```

## Run API

Start the API and worker in separate terminals.

```bash
cd apps/api
just api-up-observability
```

```bash
cd apps/api
just api-worker-up-observability
```

Useful local endpoints after startup:

- API docs: `http://localhost:3000/docs`
- OpenAPI JSON: `http://localhost:3000/docs/openapi.json`
- Readiness: `http://localhost:3000/health/ready`
- Queues: `http://localhost:3000/ops/queues`
- Grafana: `http://localhost:3001`

## Web Setup

Run web commands from `apps/web`.

```bash
cd apps/web
pnpm install
```

If an app needs local API access, copy the app environment example first:

```bash
cp apps/storefront/.env.example apps/storefront/.env
cp apps/seller/.env.example apps/seller/.env
```

## Run Web Apps

Start the storefront:

```bash
cd apps/web
pnpm dev:storefront
```

Open `http://localhost:4000`.

Start the seller app in a separate terminal:

```bash
cd apps/web
pnpm dev:seller
```

Open `http://localhost:4001`.

## Seeded Local Accounts

The demo seed includes these useful accounts in `apps/api/seed-data/auth-users.tsv`:

| Role | Email | Password |
| --- | --- | --- |
| Admin | `admin@example.com` | `Password123!` |
| Customer | `member@example.com` | `Password123!` |
| Customer | `buyer.aiden@example.com` | `Password123!` |
| Seller | `maker.olive@example.com` | `Password123!` |
| Seller | `maker.mason@example.com` | `Password123!` |

Use local-only seed files for extra local data instead of editing shared TSV files. See `apps/api/seed-data/README.md`.

## Quick Run Check

- API readiness endpoint returns healthy.
- Storefront home, search, product detail, cart, checkout start, account, orders, messages, and notifications load.
- Seller sign-in, shop dashboard, product management, inventory, orders, promotions, and messages load.
- Auth state survives refresh and redirects protected routes correctly.
- Seeded images render from the configured storage backend.
- Currency, shipping, coupon, and checkout calculations are consistent between UI and API responses.

## Common Issues

- Missing API env: copy `apps/api/api/.env.example` to `apps/api/api/.env`.
- Empty catalog or missing images: rerun `just seed-full` from `apps/api`.
- Port conflicts: stop the existing process or change the app port in the app-specific environment.
- Stale database state: run `just db-fresh-demo` from `apps/api`.
- Storage state out of sync: run `just storage-fresh` from `apps/api`.
