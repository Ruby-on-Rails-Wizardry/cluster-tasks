# Changelog

All notable changes to **cluster-tasks** are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- Repository scaffold: PLAN, TODO, CONTRACT, README, AGENTS
- Work plan to extract host tooling from docker-mise-cluster
- Ported host `bin/*` + `task/Taskfile.yml` from docker-mise-cluster (apps.yml-driven)
- **`bin/wire`** — idempotent wire into a sibling cluster (wrappers, materialize, BUNDLE_CLEAN, Task include, doctor)
- Doctor checks: `CLUSTER_TASKS_ROOT`, `BUNDLE_CLEAN=false`, materialized `docker-app`

### Changed

- Adoption model: **sibling clones** (`cluster-tasks` beside the cluster), not nested submodule


### Fixed

### Security

## [0.0.0] - 2026-08-04

### Added

- Initial planning-only repository (no installable bins yet)
