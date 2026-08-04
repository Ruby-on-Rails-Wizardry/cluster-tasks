# TODO — cluster-tasks

Living checklist. Check items off in PRs; keep [PLAN.md](PLAN.md) for design narrative.

## Phase 0 — Repository bootstrap

- [x] Create local git repo + initial docs
- [x] Create GitHub repo `Ruby-on-Rails-Wizardry/cluster-tasks`
- [x] Create GitLab project `ruby-on-rails-wizardry/cluster-tasks`
- [x] Create ami bare `Ruby-on-Rails-Wizardry/cluster-tasks`
- [x] Push master to github + gitlab + ami
- [ ] Optional: GitHub description + topics

## Phase 1 — Genericize source (in docker-mise-cluster first)

- [x] Audit `bin/*` for hard-coded app names / ports / paths
- [x] Replace doctor/setup banners with `bin/apps` output
- [x] Remove `fred ron harry george` fallbacks in `lib.sh`
- [x] Ensure `up:all` / multi-app compose args only use `bin/apps names`
- [x] Decide: drop per-app Task targets vs generate them
- [ ] Smoke: warm + one-app up + all-up on docker-mise-cluster (partial: doctor/apps/compose)

## Phase 2 — Extract into this repo

- [x] Add directory layout: `bin/`, `task/`, `docs/`, `templates/`
- [x] Port `bin/lib.sh` with `CLUSTER_ROOT` (apps.yml) + `CLUSTER_TASKS_ROOT` (sibling tools)
- [x] Port `apps`, `compose`, `warm`, `setup`, `docker-app`, `doctor`, `db-reset`
- [x] Port `cache-*`, `cache-env`, `ensure-bundle-config`, `local-gem-env`, `mise-host-env.sh`
- [x] Port/adapt generic `task/Taskfile.yml`
- [x] `bin/wire` — idempotent sibling wiring (wrappers + materialize + BUNDLE_CLEAN + doctor)
- [x] Doctor: sibling path, BUNDLE_CLEAN=false, materialized docker-app
- [ ] Port/adapt mise defaults + document consumer `.mise.env`
- [ ] Port SHARED-GEMS notes (or link to cluster doc until extract)
- [x] Add `templates/apps.yml.example`
- [x] README install/adopt section (**sibling** + `bin/wire`)
- [ ] Tag **v0.1.0** when smoke passes with sibling consumer tree

## Phase 3 — docker-mise-cluster becomes consumer (siblings)

- [x] Document required layout: `…/cluster-tasks` next to `…/docker-mise-cluster`
- [x] Wire Taskfile `includes` → `../cluster-tasks/task/Taskfile.yml` (`flatten: true`)
- [x] Thin `bin/*` wrappers → sibling; materialize in-container scripts (on branch `cluster-tasks-phase1`)
- [ ] Update cluster AGENTS/ADOPT/SHARED-GEMS for sibling tooling (phase1 branch)
- [ ] Full smoke: warm + up on phase1 branch
- [ ] Keep **master** of docker-mise-cluster untouched until ready
- [ ] CHANGELOG + merge/release cluster when ready
- [ ] Triple-push cluster (github, gitlab, ami)

## Phase 4 — Polish

- [ ] Second minimal consumer fixture (optional CI) as sibling pair
- [ ] `bin/gen-tasks` (optional) for `up:<app>` shortcuts from apps.yml
- [ ] Version policy in CHANGELOG / RELEASE notes
- [ ] setup-remotes or doc snippet for triple remotes on this repo
- [ ] Consider mise task aliases mirroring Taskfile

## Open questions

- [x] Layout: **siblings** (`cluster-tasks` beside the cluster) — not nested submodule
- [ ] Default env name: `CLUSTER_TASKS_ROOT` only, or also support `../cluster-tasks` auto-detect?
- [ ] Should `bin/compose` live here if compose file is always consumer-owned? (yes — wrapper only)
- [ ] Generate nginx snippets later or never?
- [ ] Relationship to ubuntu-sample host UX — share any scripts?

## Done log

| Date | Item |
|------|------|
| 2026-08-04 | Repo scaffold + PLAN/TODO/CONTRACT docs |
| 2026-08-04 | Phase 1 on docker-mise-cluster branch `cluster-tasks-phase1` |
| 2026-08-04 | Locked layout: **siblings**, not nest cluster-tasks under cluster |
