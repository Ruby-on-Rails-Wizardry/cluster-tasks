# cluster-tasks

Reusable **host orchestration** for multi-app Docker Rails **dev** clusters (the “overhead” next to your apps): **mise**, **Task**, and `bin/*` for warm, compose, setup, shared-gem path overrides, and db helpers.

| | |
|--|--|
| **Product** | Tooling library checked out as a **sibling** of each cluster tree |
| **Not** | App code, `compose.yml` topology, nginx configs, or production Kamal |
| **Reference consumer** | [docker-mise-cluster](https://github.com/Ruby-on-Rails-Wizardry/docker-mise-cluster) |
| **Base image contract** | [ubuntu-mise](https://github.com/Ruby-on-Rails-Wizardry/ubuntu-mise) → `ubuntu-mise:dev`, project at **`/work`**, cache volume **`cache` → `/cache`** |

This repo is **early scaffolding**. Implementation will be extracted from docker-mise-cluster once the plan in [docs/PLAN.md](docs/PLAN.md) is executed. Track progress in [docs/TODO.md](docs/TODO.md).

## Goals

1. **One toolbox** for any multi-app cluster: apps listed only in **`config/apps.yml`**.
2. **No hard-coded app names** (`fred` / `ron` / …) inside the library.
3. **Easy adopt:** **sibling clones** + thin project `Taskfile` / `bin` wrappers (not nested as a submodule of the cluster).
4. Keep **path-vs-published shared gems** (`BUNDLE_LOCAL__*` / `bin/local-gem-env`) and warm/cache behavior portable.

## Intended layout (siblings)

```text
Ruby-on-Rails-Wizardry/          # or any parent folder
├── ubuntu-mise/                 # optional sibling — build IMAGE
├── cluster-tasks/               # THIS repo (standalone clone)
└── docker-mise-cluster/         # consumer cluster (or work/)
    ├── Taskfile.yml             # includes ../cluster-tasks/task/Taskfile.yml
    ├── bin/                     # thin wrappers → ../cluster-tasks/bin/*
    ├── config/apps.yml          # apps + shared_gems (project-owned)
    ├── compose.yml              # topology (project-owned)
    ├── nginx/
    ├── <app>/…                  # app submodules
    └── <shared_gem>/…           # optional library submodules
```

Discovery: `CLUSTER_TASKS_ROOT` or default **`../cluster-tasks`** relative to the cluster root.  
Do **not** nest `cluster-tasks` inside the cluster git tree (avoids submodule gitdir pain and keeps versions independent).

Day-to-day (from the **cluster** project root, once adopted):

```bash
task warm
task up:all                 # or: task up -- app1 app2
task compose -- ps
task db:reset -- app1
```

## Documentation

| Doc | Purpose |
|-----|---------|
| [docs/LOCAL-GEMS.md](docs/LOCAL-GEMS.md) | **Shared library gems** — path overrides, enable/disable |
| [docs/ENV.md](docs/ENV.md) | **Env files** — shared vs vault secrets, dry compose |
| [docs/PLAN.md](docs/PLAN.md) | Design, boundaries, extraction phases |
| [docs/TODO.md](docs/TODO.md) | Checklist of work remaining |
| [docs/CONTRACT.md](docs/CONTRACT.md) | What the consumer must provide |
| [AGENTS.md](AGENTS.md) | Agent/maintainer rules for this repo |

## Remotes

```bash
git remote add github git@github.com:Ruby-on-Rails-Wizardry/cluster-tasks.git
git remote add gitlab git@gitlab.com:ruby-on-rails-wizardry/cluster-tasks.git
git remote add ami    git@ami:Ruby-on-Rails-Wizardry/cluster-tasks.git

git push github && git push gitlab && git push ami
```

## Wire into a cluster (sibling)

```bash
# Layout
#   parent/cluster-tasks
#   parent/docker-mise-cluster   # or work/

cd ../docker-mise-cluster
../cluster-tasks/bin/wire --yes
task doctor
task warm
task up:all
```

`wire` is **idempotent**: re-run after updating cluster-tasks to refresh wrappers and materialized in-container scripts (`docker-app`, `apps`, `local-gem-env`).

## Status

**v0.1.1** — bins + `wire` + doctor + `bootboot:nuke`; smoke-tested with sibling [docker-mise-cluster](https://github.com/Ruby-on-Rails-Wizardry/docker-mise-cluster) branch `cluster-tasks-phase1` (warm + up:all). Pin a sibling checkout at tag **v0.1.1** (or **v0.1.0**) or track `master`.

## License

Same org conventions as docker-mise-cluster (private template tooling; no license asserted yet).
