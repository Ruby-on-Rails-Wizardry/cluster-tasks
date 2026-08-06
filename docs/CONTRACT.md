# Consumer contract

What a cluster project **must** provide for cluster-tasks to work (v0.1+ target).

## Required

| Path / fact | Meaning |
|-------------|---------|
| **Project root** | Directory from which `task` / `bin/*` are run (the **cluster** clone) |
| **Tooling** | **`bin/install --yes`** (standalone copies under consumer `bin/`) **or** sibling `../cluster-tasks` + `wire` |
| **`config/apps.yml`** | List of apps (`name`, `path`, `port`, …) and optional `shared_gems` |
| **`compose.yml`** | Dev stack; service names should match app `name`s for `compose up <name>` |
| **Docker** | Daemon available; prefer `bin/compose` wrapper |
| **Image** | `ubuntu-mise:dev` (or `IMAGE=…`) built for this host UID |
| **Mount** | Compose bind-mounts project root to **`WORK_MOUNT`** (default **`/work`**; see [WORK-MOUNT.md](WORK-MOUNT.md)) |
| **Cache** | Named volume (default `cache`) mounted at **`/cache`** |

**Do not** nest `cluster-tasks` as a git submodule of the cluster. Keep both repos as **siblings** (same pattern as standalone `docker-mise-cluster` next to `ubuntu-mise`).

### Minimal `config/apps.yml`

```yaml
apps:
  - name: myapp
    path: myapp
    port: 3001
    url_root: /myapp
    database: myapp_development
    redis_db: 0

shared_gems: []
# shared_gems:
#   - name: my_shared
#     path: my_shared
```

## Optional but supported

| Path | Meaning |
|------|---------|
| Root `Gemfile` | Tooling only; warm may install it |
| Optional `<app>/bin/after-docker-prepare` | App-specific hook after DB prepare (executable) |
| `shared_gems` + path checkouts | `bin/local-gem` / `local-gem-env` → `BUNDLE_LOCAL__*` ([LOCAL-GEMS.md](LOCAL-GEMS.md)) |
| `config/bundler-flags.yml` | Seed for `ensure-bundle-config` |
| `config/shared.env` | Committed non-secret app env (see docs/ENV.md) |
| `config/env.yml` | Manifest for shared vs vault secret files (`bin/env-files`) |
| `config/cache.env` | Optional stub only — **not** package-manager ENV (image provides `/cache`) |
| Per-app `Taskfile.yml` | `task <app> -- …` delegates into app tree |
| `nginx/` | Front door; doctor may check `nginx/nginx.conf` if present |

## Environment knobs (planned)

| Variable | Default | Role |
|----------|---------|------|
| `IMAGE` | `ubuntu-mise:dev` | App/dev container image |
| `CACHE_VOLUME` | `cache` | Docker volume name for `/cache` |
| `UBUNTU_MISE_ROOT` | sibling `../ubuntu-mise` | Where to build base image |
| `CLUSTER_ROOT` | auto | Override project (consumer) root discovery |
| `CLUSTER_TASKS_ROOT` | sibling `../cluster-tasks` | Where this tooling lives |
| `WARM_*` | see warm | Isolate bundle, skip next lock, force cache |

## Explicitly out of contract

- Kamal / production deploy files
- App business logic
- Exact app count or names
- Nesting the consumer as a submodule of an unrelated monorepo (breaks gitdir for `local.*` shared gems)

## Standalone clone

Consumers that use **Bundler `local.*`** for shared gems must be a **standalone git clone** (recurse-submodules), not a nested submodule of another repo. See docker-mise-cluster `docs/SHARED-GEMS.md`.
