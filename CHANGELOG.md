# Changelog

All notable changes to **cluster-tasks** are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- `bin/nuke-bootboot` / `task bootboot:nuke` — remove installed bootboot artifacts
  (per-app `.bundle/plugin`, isolate `/cache/bundle/<app>`, bootboot gem trees;
  optional `--locks` for `Gemfile_next.lock`). Dry-run by default; pass `-y`.

### Changed

### Fixed

### Security

## [0.1.0] - 2026-08-04

First installable release. Clone as a **sibling** of a multi-app cluster and run `bin/wire`.

### Added

- Repository scaffold: PLAN, TODO, CONTRACT, README, AGENTS
- Ported host `bin/*` + `task/Taskfile.yml` from docker-mise-cluster (apps.yml-driven)
- **`bin/wire`** — idempotent wire into a sibling cluster (wrappers, materialize, `BUNDLE_CLEAN=false`, Task include, doctor)
- Doctor checks: `CLUSTER_TASKS_ROOT`, `BUNDLE_CLEAN=false`, materialized `docker-app`
- `templates/apps.yml.example`
- Discovery: `CLUSTER_TASKS_ROOT` or auto-detect **`../cluster-tasks`** relative to the consumer

### Changed

- Adoption model: **sibling clones** (`cluster-tasks` beside the cluster), not nested submodule

### Fixed

- `bin/docker-app`: wait for Postgres with `pg_isready` against the maintenance DB (`postgres`), not ActiveRecord against the app database — fixes first boot on a fresh `pgdata` volume before `db:prepare`

### Security

## [0.0.0] - 2026-08-04

### Added

- Initial planning-only repository (no installable bins yet)
