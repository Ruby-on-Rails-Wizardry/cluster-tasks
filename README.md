# cluster-tasks

Reusable **host orchestration** for multi-app Docker Rails **dev** clusters (the “overhead” next to your apps): **mise**, **Task**, and `bin/*` for warm, compose, setup, shared-gem path overrides, and db helpers.

| | |
|--|--|
| **Product** | Tooling library / submodule consumers drop into a `work/` (or similar) tree |
| **Not** | App code, `compose.yml` topology, nginx configs, or production Kamal |
| **Reference consumer** | [docker-mise-cluster](https://github.com/Ruby-on-Rails-Wizardry/docker-mise-cluster) |
| **Base image contract** | [ubuntu-mise](https://github.com/Ruby-on-Rails-Wizardry/ubuntu-mise) → `ubuntu-mise:dev`, project at **`/work`**, cache volume **`cache` → `/cache`** |

This repo is **early scaffolding**. Implementation will be extracted from docker-mise-cluster once the plan in [docs/PLAN.md](docs/PLAN.md) is executed. Track progress in [docs/TODO.md](docs/TODO.md).

## Goals

1. **One toolbox** for any multi-app cluster: apps listed only in **`config/apps.yml`**.
2. **No hard-coded app names** (`fred` / `ron` / …) inside the library.
3. **Easy adopt:** submodule (or vendor) + thin project `Taskfile` / `bin` wrappers.
4. Keep **path-vs-published shared gems** (`BUNDLE_LOCAL__*` / `bin/local-gem-env`) and warm/cache behavior portable.

## Intended consumer layout

```text
work/                         # your cluster (Compose project = directory name)
├── .cluster-tasks/           # submodule → this repo (name TBD)
├── Taskfile.yml              # includes cluster-tasks Taskfile
├── bin/                      # thin wrappers OR PATH to .cluster-tasks/bin
├── config/apps.yml           # apps + shared_gems (project-owned)
├── compose.yml               # topology (project-owned)
├── nginx/                    # (project-owned)
├── <app>/…                   # app submodules
└── <shared_gem>/…            # optional library submodules
```

Day-to-day (from the project root, once adopted):

```bash
task warm
task up:all                 # or: task up -- app1 app2
task compose -- ps
task db:reset -- app1
```

## Documentation

| Doc | Purpose |
|-----|---------|
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

## Status

**v0.0.0 — planning only.** No installable `bin/` yet; do not submodule into production clusters until **v0.1.0**.

## License

Same org conventions as docker-mise-cluster (private template tooling; no license asserted yet).
