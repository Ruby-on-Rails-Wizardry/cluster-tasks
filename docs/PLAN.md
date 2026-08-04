# Plan — extract cluster host tooling

## Problem

[docker-mise-cluster](https://github.com/Ruby-on-Rails-Wizardry/docker-mise-cluster) embeds useful host UX (`bin/*`, Task, mise) for multi-app Docker Rails **dev**. That tooling should work for **any** cluster-shaped tree (weasily, future `work/` clones), not only the demo apps.

Today most scripts already read **`config/apps.yml`**, but:

- Taskfile still has **per-app hard-coded** tasks (`up:fred`, …)
- A few banners/fallbacks still name demo apps
- Copy-pasting the whole cluster repo to get Task/bin is the wrong unit of reuse

## Non-goals

- Shipping a multi-app **compose** or nginx template (stays in the consumer)
- Production Kamal / app Dockerfiles
- Replacing ubuntu-mise
- Bootboot dual-boot (shared gems use **`local.*`** / `BUNDLE_LOCAL__*`)

## Boundaries

```text
  parent/
  ├── cluster-tasks/          ← this repo (standalone git clone)
  ├── docker-mise-cluster/    ← consumer (standalone git clone)
  └── ubuntu-mise/            ← optional sibling for image builds

  Consumer Taskfile / bin wrappers point at ../cluster-tasks
  (or $CLUSTER_TASKS_ROOT). No nesting cluster-tasks inside the cluster repo.
```

```text
┌─────────────────────────────────────────────────────────┐
│  cluster-tasks (sibling checkout)                       │
│  bin/*, generic Taskfile, mise defaults, contracts      │
└──────────────────────────▲──────────────────────────────┘
                           │ PATH / Task include / thin wrappers
┌──────────────────────────┴──────────────────────────────┐
│  Consumer cluster (docker-mise-cluster, work/, …)       │
│  config/apps.yml, compose.yml, nginx/, apps, gems       │
└─────────────────────────────────────────────────────────┘
                           │ image contract
┌──────────────────────────┴──────────────────────────────┐
│  ubuntu-mise:dev  ·  /work mount  ·  volume cache→/cache│
└─────────────────────────────────────────────────────────┘
```

## Design decisions (proposed)

| Decision | Choice |
|----------|--------|
| Distribution | **Sibling clone** next to the cluster (`../cluster-tasks` or `CLUSTER_TASKS_ROOT`) — **not** a nested submodule of the cluster |
| Config SSOT for apps | Consumer **`config/apps.yml`** only |
| Compose | Consumer-owned; tasks shell out to `bin/compose` against project root |
| Task UX | Generic `warm`, `setup`, `compose`, `up`, `up:all`, `db:reset`; optional **generated** `up:<name>` later |
| App list for `up:all` | `bin/apps names` |
| Image | Default `ubuntu-mise:dev`; override via env / `.mise.env` |
| Cache volume | Default name `cache` → `/cache` |
| Shared gems | `shared_gems` in apps.yml + `bin/local-gem-env` |
| Versioning | semver tags on cluster-tasks; consumers pin via checkout branch/tag/SHA of the **sibling** clone |

## Extraction source

Primary source of truth today: **docker-mise-cluster** `bin/` (~1.3k LOC) + `Taskfile.yml` + patterns from `docs/SHARED-GEMS.md`.

Rough map:

| Source path | Destination |
|-------------|-------------|
| `bin/lib.sh` | `bin/lib.sh` (env knobs, no demo fallbacks) |
| `bin/apps` | `bin/apps` |
| `bin/compose`, `warm`, `setup`, `docker-app`, … | same names under `bin/` |
| `Taskfile.yml` | `task/Taskfile.yml` (generic) |
| `mise.toml` / `.mise.env` patterns | `mise.toml` + docs for consumer merge |
| SHARED-GEMS concepts | `docs/SHARED-GEMS.md` (ported/adapted) |

## Phases

### Phase 0 — Repo + docs (this commit)

- Create repo, remotes, PLAN/TODO/CONTRACT, empty layout placeholders
- No consumer adoption yet

### Phase 1 — Make docker-mise-cluster fully generic (in place)

- Remove hard-coded app names from `lib.sh`, `doctor`, `setup` messages
- `up:all` / compose always driven by `bin/apps`
- Optional: generate Task snippet from apps.yml  
- **Prove** genericity without a second repo consumer

### Phase 2 — Extract into cluster-tasks

- Copy generalized `bin/*` + Taskfile into this repo
- Add discovery:
  - **Consumer root:** `CLUSTER_ROOT` or cwd / walk to `config/apps.yml`
  - **Tooling root:** `CLUSTER_TASKS_ROOT` or sibling `../cluster-tasks`
- Document thin wrappers / Task `includes` with `dir: .` and path to sibling Taskfile
- Tag **v0.1.0**

### Phase 3 — Adopt in docker-mise-cluster (sibling)

- Assume layout: `…/cluster-tasks` next to `…/docker-mise-cluster`
- Thin `bin/*` wrappers calling `$CLUSTER_TASKS_ROOT/bin/…` (default `../cluster-tasks`)
- Taskfile includes sibling `../cluster-tasks/task/Taskfile.yml`
- Delete duplicated script **bodies** from cluster (wrappers only)
- CI / manual smoke: `task warm`, `task up:all` (or single app)
- Document clone of **both** repos side by side

### Phase 4 — Second consumer / polish

- Minimal fixture or weasily-style adopt notes (sibling layout)
- `bin/gen-tasks` if per-app Task shortcuts are still wanted
- Release process + CHANGELOG discipline

## Risks

| Risk | Mitigation |
|------|------------|
| Compose still project-specific | Never generate full compose in v1 |
| Broken ROOT when invoked from wrong cwd | Document “run from cluster root”; fail fast if no apps.yml |
| Missing sibling checkout | Fail with clear “clone cluster-tasks next to this project” |
| Nested monorepo gitdir + local gems | Consumer must be standalone tree (see SHARED-GEMS) |
| Divergent forks of bin/ | One release train; consumers pin sibling SHA/tag |

## Success criteria (v0.1.0)

- [ ] Sibling checkouts: `cluster-tasks` + consumer; consumer can `task warm` via wrappers
- [ ] docker-mise-cluster uses sibling cluster-tasks (no duplicate bin implementations)
- [ ] No hard-coded demo app names in cluster-tasks
- [ ] Docs: adopt (sibling), contract, remotes (github + gitlab + ami)
