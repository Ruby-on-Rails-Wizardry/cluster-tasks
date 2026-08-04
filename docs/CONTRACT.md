# Consumer contract

What a cluster project **must** provide for cluster-tasks to work (v0.1+ target).

## Required

| Path / fact | Meaning |
|-------------|---------|
| **Project root** | Directory from which `task` / `bin/*` are run |
| **`config/apps.yml`** | List of apps (`name`, `path`, `port`, …) and optional `shared_gems` |
| **`compose.yml`** | Dev stack; service names should match app `name`s for `compose up <name>` |
| **Docker** | Daemon available; prefer `bin/compose` wrapper |
| **Image** | `ubuntu-mise:dev` (or `IMAGE=…`) built for this host UID |
| **Mount** | Compose bind-mounts project root to **`/work`** |
| **Cache** | Named volume (default `cache`) mounted at **`/cache`** |

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
| `shared_gems` + path checkouts | `bin/local-gem-env` → `BUNDLE_LOCAL__*` |
| `config/bundler-flags.yml` | Seed for `ensure-bundle-config` |
| Per-app `Taskfile.yml` | `task <app> -- …` delegates into app tree |
| `nginx/` | Front door; doctor may check `nginx/nginx.conf` if present |

## Environment knobs (planned)

| Variable | Default | Role |
|----------|---------|------|
| `IMAGE` | `ubuntu-mise:dev` | App/dev container image |
| `CACHE_VOLUME` | `cache` | Docker volume name for `/cache` |
| `UBUNTU_MISE_ROOT` | sibling `../ubuntu-mise` | Where to build base image |
| `CLUSTER_ROOT` | auto | Override project root discovery |
| `WARM_*` | see warm | Isolate bundle, skip next lock, force cache |

## Explicitly out of contract

- Kamal / production deploy files
- App business logic
- Exact app count or names
- Nesting the consumer as a submodule of an unrelated monorepo (breaks gitdir for `local.*` shared gems)

## Standalone clone

Consumers that use **Bundler `local.*`** for shared gems must be a **standalone git clone** (recurse-submodules), not a nested submodule of another repo. See docker-mise-cluster `docs/SHARED-GEMS.md`.
