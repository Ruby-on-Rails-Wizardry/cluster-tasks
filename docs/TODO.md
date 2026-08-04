# TODO — cluster-tasks

Living checklist. Check items off in PRs; keep [PLAN.md](PLAN.md) for design narrative.

## Phase 0 — Repository bootstrap

- [x] Create local git repo + initial docs
- [ ] Create GitHub repo `Ruby-on-Rails-Wizardry/cluster-tasks`
- [ ] Create GitLab project `ruby-on-rails-wizardry/cluster-tasks`
- [ ] Create ami bare `Ruby-on-Rails-Wizardry/cluster-tasks`
- [ ] Push master to github + gitlab + ami
- [ ] Optional: GitHub description + topics

## Phase 1 — Genericize source (in docker-mise-cluster first)

- [ ] Audit `bin/*` for hard-coded app names / ports / paths
- [ ] Replace doctor/setup banners with `bin/apps` output
- [ ] Remove `fred ron harry george` fallbacks in `lib.sh`
- [ ] Ensure `up:all` / multi-app compose args only use `bin/apps names`
- [ ] Decide: drop per-app Task targets vs generate them
- [ ] Smoke: warm + one-app up + all-up on docker-mise-cluster

## Phase 2 — Extract into this repo

- [ ] Add directory layout: `bin/`, `task/`, `docs/`, `templates/`
- [ ] Port `bin/lib.sh` with `CLUSTER_ROOT` / apps.yml discovery
- [ ] Port `apps`, `compose`, `warm`, `setup`, `docker-app`, `doctor`, `db-reset`
- [ ] Port `cache-*`, `cache-env`, `ensure-bundle-config`, `local-gem-env`, `mise-host-env.sh`
- [ ] Port/adapt generic `task/Taskfile.yml`
- [ ] Port/adapt mise defaults + document consumer `.mise.env`
- [ ] Port SHARED-GEMS notes (or link to cluster doc until extract)
- [ ] Add `templates/apps.yml.example`
- [ ] README install/adopt section (submodule steps)
- [ ] Tag **v0.1.0** when smoke passes on a throwaway consumer tree

## Phase 3 — docker-mise-cluster becomes consumer

- [ ] Add submodule pin to cluster-tasks
- [ ] Wire Taskfile `includes` + dir = project root
- [ ] Thin wrappers or remove duplicated `bin/*`
- [ ] Update cluster AGENTS/ADOPT/SHARED-GEMS to point at cluster-tasks
- [ ] CHANGELOG + release cluster as needed
- [ ] Triple-push cluster (github, gitlab, ami)

## Phase 4 — Polish

- [ ] Second minimal consumer fixture (optional CI)
- [ ] `bin/gen-tasks` (optional) for `up:<app>` shortcuts from apps.yml
- [ ] Version policy in CHANGELOG / RELEASE notes
- [ ] setup-remotes or doc snippet for triple remotes on this repo
- [ ] Consider mise task aliases mirroring Taskfile

## Open questions

- [ ] Submodule path name: `.cluster-tasks` vs `cluster-tasks` vs `vendor/cluster-tasks`?
- [ ] Should `bin/compose` live here if compose file is always consumer-owned? (yes — wrapper only)
- [ ] Generate nginx snippets later or never?
- [ ] Relationship to ubuntu-sample host UX — share any scripts?

## Done log

| Date | Item |
|------|------|
| 2026-08-04 | Repo scaffold + PLAN/TODO/CONTRACT docs |
