# Local gems — path overrides for shared libraries

Easy guide for **path-vs-published** shared gems in a multi-app cluster.

Bundler can resolve a gem from a **local checkout** while the Gemfile still pins
the **published** version (Nexus / rubygems / git). That is what we use for
library gems edited next to the apps. **Not** Shopify bootboot (dual lockfiles).

| Tool | Role |
|------|------|
| `config/apps.yml` → `shared_gems` | Register gem name + path under the cluster root |
| `bin/local-gem` | **status / enable / disable** + shell env |
| `bin/local-gem-env` | Emit `export BUNDLE_LOCAL__…=path` (used by warm / docker-app) |

---

## 30-second version

```bash
# 1. Register the checkout (once)
# config/apps.yml
# shared_gems:
#   - name: wizardry_shared
#     path: wizardry_shared

# 2. Ensure local overrides are on (default)
task local-gem -- status
task local-gem -- enable          # if you previously disabled

# 3. Warm / up already apply overrides inside containers
task warm
task up:all

# Host shell for this terminal only
eval "$(bin/local-gem env)"
cd fred && bundle install
```

Turn **off** (use published / lock source again):

```bash
task local-gem -- disable         # persists for this machine (flag file)
# or one shot:  LOCAL_GEMS=0 task warm
```

---

## How it works

1. **Gemfile** pins the published form only (version, Nexus, or git **branch** — not tag-only).
2. **`shared_gems`** lists checkouts under the cluster root (usually submodules).
3. Dev sets Bundler **local override**:

   | Concept | Example |
   |---------|---------|
   | Config name | `local.wizardry_shared` |
   | Environment variable | `BUNDLE_LOCAL__WIZARDRY_SHARED=/path/to/checkout` |

4. **warm** and **docker-app** `eval` `bin/local-gem-env /work` when local gems are **enabled**.
5. Deploy / CI never set `BUNDLE_LOCAL__*` — lock + published source win.

We use **ENV**, not `bundle config set --local local.*` in `.bundle/config`, so host
paths and container `/work/…` do not fight.

---

## Register a gem

```yaml
# config/apps.yml
shared_gems:
  - name: wizardry_shared   # gem name (Bundler local.<name>)
    path: wizardry_shared   # directory under cluster root
```

Checkout that path (submodule or clone). Gemfile in each app:

```ruby
gem "wizardry_shared", "0.1.0",
  git: "https://github.com/org/wizardry_shared.git",
  branch: "master"   # branch required for local.* with git sources
```

Or a pure version pin against Nexus / rubygems when you have that source.

---

## Enable / disable

```bash
bin/local-gem status              # what is registered + on/off + paths
bin/local-gem enable              # remove disable flag (default is on)
bin/local-gem disable             # write .local-gems.disabled (gitignored)
bin/local-gem env                 # same as local-gem-env (exports)
bin/local-gem env --print         # KEY=value only
bin/local-gem env /work           # container paths
```

| Mechanism | Scope |
|-----------|--------|
| **Default** (no flag) | Local overrides **on** whenever `shared_gems` is non-empty |
| **`bin/local-gem disable`** | Writes **`.local-gems.disabled`** in the cluster root — warm / docker-app / setup skip overrides |
| **`bin/local-gem enable`** | Deletes that flag |
| **`LOCAL_GEMS=0`** | One-shot off for that process (also accepts `false` / `off`) |
| **`LOCAL_GEMS=1`** | Force on even if the disable flag exists |

`.local-gems.disabled` is machine-local (listed in `.gitignore` by `wire`). It does
**not** change Gemfiles or lockfiles.

After **enable** or **disable**, re-run **`task warm`** (and restart app containers)
so installs and boots pick up the new mode.

---

## Host vs container

| Context | Command |
|---------|---------|
| Host shell | `eval "$(bin/local-gem env)"` |
| Inside `/work` (compose) | warm / docker-app call `local-gem-env /work` for you |
| Inspect | `bin/local-gem status` / `bin/local-gem env --print` |

---

## Checklist

- [ ] Standalone cluster clone (not nested submodule — gitdir must live under `/work`)
- [ ] Shared gem on a **real branch** (submodules often detach HEAD)
- [ ] Gemfile uses **branch:** for git sources (or pure version pin)
- [ ] Gemspec version matches Gemfile pin
- [ ] `shared_gems` entry present
- [ ] Local mode **enabled** (`bin/local-gem status`)
- [ ] No `BUNDLE_LOCAL__*` in production / CI images

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| Still fetching remote | Overrides disabled, or process never got ENV — `status`, then `enable` + warm |
| `branch is not specified` | Use `branch:` in the Gemfile git source |
| `git rev-parse` failed under `/work` | Nested monorepo mount; use standalone cluster |
| Version conflict | Align gemspec VERSION with Gemfile pin |
| Deploy looks for a path | `BUNDLE_LOCAL__*` leaked into image or CI — never bake enable into prod |

---

## Related

- Consumer deep-dive (demo cluster): [docker-mise-cluster SHARED-GEMS](https://github.com/Ruby-on-Rails-Wizardry/docker-mise-cluster/blob/master/docs/SHARED-GEMS.md)
- [Bundler: local git repos](https://bundler.io/guides/git.html#local-git-repos)
- [docs/CONTRACT.md](CONTRACT.md) — `shared_gems` on the consumer
