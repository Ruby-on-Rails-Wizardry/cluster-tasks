# Bundler: use a local directory for a gem

**Use case:** The Gemfile already pins a gem (rubygems, private server, or git). You want Bundler to load that gem from a **local directory** on disk. **Do not change the Gemfile.**

The pin stays the source of truth for deploy and for the lockfile. Local override only affects processes where you enable it.

---

## One idea

| | |
|--|--|
| Gemfile | Unchanged |
| Override | “For gem *name*, use this **directory** instead of downloading/cloning” |
| Directory | Must be the gem root (contains the `.gemspec`) |
| Version | Gemspec version must still satisfy the Gemfile requirement |

---

## Option A — config (`local.<name>`)

From the application directory (the one with the Gemfile):

```bash
bundle config set --local local.GEM_NAME /absolute/path/to/gem/directory
bundle install
```

Example: gem `wizardry_shared` lives in `/home/you/src/wizardry_shared`:

```bash
bundle config set --local local.wizardry_shared /home/you/src/wizardry_shared
bundle install
```

Inspect:

```bash
bundle config get local.wizardry_shared
```

Turn off:

```bash
bundle config unset --local local.wizardry_shared
bundle install
```

`--local` writes the app’s `.bundle/config`. The path is stored there until you unset it.

---

## Option B — environment variable (`BUNDLE_LOCAL__*`)

Same override, no config file. Mapping:

```text
local.GEM_NAME  →  BUNDLE_LOCAL__GEM_NAME
```

- Prefix: `BUNDLE_LOCAL__` (two underscores)
- Gem name: **UPPERCASE**; hyphens become underscores

```bash
export BUNDLE_LOCAL__WIZARDRY_SHARED=/absolute/path/to/gem/directory
bundle install
```

Turn off:

```bash
unset BUNDLE_LOCAL__WIZARDRY_SHARED
bundle install
```

Only processes that inherit this environment use the local directory.

---

## Config vs environment

| | Config (`local.*`) | Environment (`BUNDLE_LOCAL__*`) |
|--|--------------------|----------------------------------|
| Set | `bundle config set --local local.NAME /path` | `export BUNDLE_LOCAL__NAME=/path` |
| Clear | `bundle config unset --local local.NAME` | `unset BUNDLE_LOCAL__NAME` |
| Lasts | Until unset (in `.bundle/config`) | Only while the env is set |
| Gemfile | Unchanged | Unchanged |

Use **one** of them for a given gem in a given process; they implement the same Bundler feature.

---

## After enabling

```bash
bundle install    # resolve/install with the local directory
bundle exec …     # run with that resolution
```

---

## Git pins and the Gemfile

If the **existing** Gemfile line uses a **git** source, Bundler requires that line to specify a **`branch:`** for a local directory override to work. You still do not change the Gemfile for path work itself—but if the pin is tag-only, Bundler will refuse the override until the pin includes a branch. Pure version pins (no git) do not need a branch.

---

## Failures specific to this use case

| Symptom | Cause |
|---------|--------|
| Override ignored | Wrong gem name, or config/env not active in this process |
| `branch is not specified` | Git pin without `branch:` |
| Version error | Directory gemspec version does not match the Gemfile requirement |
| Path errors | Path is not the gem root, or is not absolute / not readable |

---

## Reference

[Bundler: local git repos / local overrides](https://bundler.io/guides/git.html#local-git-repos)
