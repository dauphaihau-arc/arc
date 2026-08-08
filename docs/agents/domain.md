# Domain Docs

How the engineering skills should consume this repo's domain documentation when exploring the codebase.

## Layout

This repo uses a multi-context layout.

- `CONTEXT-MAP.md` at the repo root points to the relevant context docs.
- `docs/adr/` contains repo-wide architecture decisions.
- Context-specific `CONTEXT.md` and `docs/adr/` files may live under app or package directories.

## Before exploring, read these

- `CONTEXT-MAP.md` at the repo root, then read each referenced `CONTEXT.md` relevant to the topic.
- `docs/adr/` for repo-wide decisions that touch the area you're about to work in.
- Context-scoped ADR directories when present.

If any of these files don't exist, proceed silently. Don't flag their absence; don't suggest creating them upfront. The domain-modeling skills create them lazily when terms or decisions actually get resolved.

## Use the glossary's vocabulary

When your output names a domain concept, use the term as defined in the relevant `CONTEXT.md`. Don't drift to synonyms the glossary explicitly avoids.

If the concept you need isn't in the glossary yet, either reconsider whether you're inventing language the project doesn't use, or note the gap for domain-modeling.

## Flag ADR conflicts

If your output contradicts an existing ADR, surface it explicitly rather than silently overriding.
