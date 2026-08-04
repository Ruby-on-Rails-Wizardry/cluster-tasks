# Agent guide — cluster-tasks

## Purpose

Reusable host orchestration for multi-app Docker **dev** clusters.  
**Not** a compose stack and **not** app code.

Read [docs/PLAN.md](docs/PLAN.md) and [docs/TODO.md](docs/TODO.md) before large changes.

## Rules

1. **No hard-coded consumer app names** in library code (no `fred` / `ron` as defaults).
2. **apps.yml is the list** — all multi-app loops go through `bin/apps` (or successor).
3. **Consumer owns compose + nginx + apps**; this repo owns warm/compose-wrapper/setup/task UX.
4. Keep the **ubuntu-mise / `/work` / `/cache`** contract stable; document breaking changes in CHANGELOG.
5. Prefer **minimal diffs**; do not reintroduce Yarn Berry or dual host `.cache` trees.
6. Remotes: **github** (canonical), **gitlab**, **ami** — push all three after meaningful commits.
7. Until **v0.1.0**, do not tell consumers to submodule this repo for production use.

## Phrase shortcuts

| Phrase | Means |
|--------|--------|
| **send it** / **ship it** | When a release process exists: tests/docs → version → tag → push branch+tag → `gh release create` → push ami |

## Related

- [docker-mise-cluster](https://github.com/Ruby-on-Rails-Wizardry/docker-mise-cluster)
- [ubuntu-mise](https://github.com/Ruby-on-Rails-Wizardry/ubuntu-mise)
