# Agent Tools

Small deterministic tools that help agents navigate Arc without broad,
unguided repository search.

## Repo Map

Use `repo-map` before starting work on a known feature area:

```bash
./tools/agent/repo-map checkout
./tools/agent/repo-map product-import
./tools/agent/repo-map multi-currency
./tools/agent/repo-map realtime-chat
```

It prints the relevant docs, API surfaces, web surfaces, guidance files, and
verification commands for the topic.

Modes:

- `./tools/agent/repo-map --list` prints the topic names.
- `./tools/agent/repo-map --check` exits non-zero when a topic names a missing
  path or an unknown `./scripts/verify` target. It runs as part of
  `./scripts/verify architecture` and in CI, so moving a doc or renaming a
  verify target fails loudly instead of surfacing as `(missing)` mid-task.

To add a topic, define a `topic_<name>()` function; the usage list and
`--list` derive from those functions, and `--check` picks it up automatically.
Map committed files only: CI checks out the repository without gitignored paths
such as `.scratch/`, so a topic must not name them.
