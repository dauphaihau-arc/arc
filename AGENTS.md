# Arc Agent Instructions

This file is the repository-level router. Keep it short; load area-specific
guidance before editing.

## Repository areas

- API app: `apps/api/api/`
- Inventory service: `apps/api/inventory-service/`
- Web monorepo: `apps/web/`
- Seller web app: `apps/web/apps/seller/`
- Storefront web app: `apps/web/apps/storefront/`

## Before Editing

1. Determine which application or package owns the requested behavior.
2. Read the nearest `AGENTS.md` for that area.
3. For domain-sensitive work, read `CONTEXT-MAP.md`, then the relevant context doc.
4. For architecture-sensitive work, read the relevant area docs before changing code.

## Agent skills

### Issue tracker

Issues and specs are tracked as local markdown under `.scratch/<feature-slug>/`. See `docs/agents/issue-tracker.md`.

### Domain docs

This repo uses a multi-context domain doc layout rooted at `CONTEXT-MAP.md`. See `docs/agents/domain.md`.

### API guidance

For API work, read `apps/api/api/AGENTS.md`, then the focused files linked
from `apps/api/api/agent-skills/README.md`.

### Web guidance

For seller or storefront work, read `apps/web/AGENTS.md`, then the focused
files linked from `apps/web/agent-skills/README.md`.

### Repo map

For known feature areas, use `./tools/agent/repo-map <topic>` before broad
search. Available topics include `checkout`, `product-import`,
`multi-currency`, `realtime-chat`, `order-export`, `inventory`, and `catalog`.

## Verification

Use the root verification harness when the change spans areas:

```bash
./scripts/verify
```

For focused checks:

```bash
./scripts/verify api
./scripts/verify web
./scripts/verify seller
./scripts/verify storefront
./scripts/verify inventory
./scripts/verify architecture
```
