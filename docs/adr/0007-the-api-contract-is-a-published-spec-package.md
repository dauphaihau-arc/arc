---
status: accepted
---

# The API contract is a published spec package, not a runtime document

The API repository is the only author of the HTTP contract. It generates `openapi.json` by booting Nest in preview mode, where no database, Redis, or Mongo connection is dialed, and commits the file beside the API source; the document no longer exists only behind `/docs/openapi.json` on a running server. A publish workflow copies the committed spec into `@dauphaihau-arc/api-spec` and pushes it to GitHub Packages on merge to `production`. The web repository installs that package at a pinned version and runs `openapi-typescript` over it to emit committed types, so both repositories read one file instead of two hand-maintained shapes. Each side refuses a stale artifact in CI: the API fails when `openapi.json` disagrees with the code, and the web fails when the generated types disagree with the installed spec.

## Versioning

The package version is hand-written semver in the manifest: patch for descriptions and examples, minor for additive fields or endpoints, major for breaking or newly required fields. CI refuses to publish a version that already exists, so a release cannot overwrite what a consumer has pinned. Prereleases publish under the `next` tag and are never resolved by `latest`, and breaking changes are gated by `oasdiff` before a version ships. Renovate opens the upgrade PR in the web repository, where a major bump is reviewed as the type change it is.

## Enum ownership

`@arc/enums` stays hand-written. The spec's inline enum tuples are not generated into the frontend; a conformance test asserts that every enum-shaped tuple in the spec matches an `@arc/enums` value set, and value sets with no frontend counterpart are allowlisted with a reason. The frontend keeps its enum ergonomics — exhaustive switches, named members, reverse mappings — rather than generated string unions, and a spec enum that gains or loses a value fails the test instead of silently widening a union.

## Considered alternatives

- **Keep hand-maintained zod API types in the frontend.** It is what the repo did, but nothing tied those shapes to the server, so a renamed field was discovered at runtime in the browser rather than by a compiler.
- **Fetch the spec from `/docs/openapi.json` and generate types from it.** It needs a running, seeded API to typecheck a frontend, and it makes the contract depend on whatever revision a developer happened to have booted. A committed file makes the contract diffable and reviewable in the same pull request as the change.
- **Generate types at install or build time without committing them.** Cheaper to maintain, but then a contract change is invisible in review and any build resolves a different document depending on the lockfile and network. Committing the generated output makes the diff part of the change.
- **Generate enums from the spec instead of maintaining `@arc/enums`.** It removes the conformance test, but it turns every enum into a union of strings and drops the exhaustive-switch and named-member ergonomics the frontend code depends on.

## Consequences

- A backend field rename or a newly required field now fails the frontend build from generated types, instead of surfacing as a runtime error.
- The web repository needs GitHub Packages credentials with `read:packages` to install; CI supplies a token and local setup documents `.npmrc`.
- The committed spec and generated types make a contract change reviewable as a diff, at the cost of regenerating both and re-running the checks whenever the API contract moves.
- The API must publish a new version for the frontend to see a change, so a merged API change does not reach the web repository until the released package is bumped and Renovate opens the upgrade.
- Preview-mode generation is a hard requirement: the contract job must not acquire a dependency on PostgreSQL, Redis, or Mongo, or the spec cannot be produced in CI.
- The enum allowlist is a review surface, not a dumping ground; every entry documents why the frontend has no matching value set.
