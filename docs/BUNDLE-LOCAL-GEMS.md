# Bundler: use a local directory for a gem

**Use case:** The Gemfile already pins a gem (rubygems, private server, or git). You want Bundler to load that gem from a **local directory** on disk. **Do not change the Gemfile.**

The pin stays the source of truth for deploy and for the lockfile. Local override only affects processes where you enable it.

## TL;DR

Gem name `my_lib`, directory `/path/to/my_lib` (gem root with the `.gemspec`).
Run `bundle install` after turning an override on or off.

**On — config** (writes app `.bundle/config`):

```bash
bundle config set --local local.my_lib /path/to/my_lib
```

**On — environment** (this shell / process only):

```bash
export BUNDLE_LOCAL__MY_LIB=/path/to/my_lib
```

**Off — config:**

```bash
bundle config unset --local local.my_lib
```

**Off — environment:**

```bash
unset BUNDLE_LOCAL__MY_LIB
```

Hyphenated gem names: `some-gem` → `local.some-gem` / `BUNDLE_LOCAL__SOME_GEM`.

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

- Local directory override via **`bundle config set … local.GEM_NAME`**:  
  [Bundler guide — Local git repos](https://bundler.io/guides/git.html#local-git-repos)  
  (same mechanism; path is a local checkout directory)

- **Environment variable** form of any config key (including `local.*`):  
  [bundle-config — Build flags / Environment Variables](https://bundler.io/v2.4/man/bundle-config.1.html#BUILD-FLAGS)  
  and the RubyGems command reference:  
  [bundle config — Environment Variables](https://guides.rubygems.org/command-reference/bundle-config/)  

  Those pages document that config keys map to env vars by uppercasing and
  replacing `.` with `__`. Example given there: `local.rack` →
  **`BUNDLE_LOCAL__RACK`**. That is the documented source for the
  `BUNDLE_LOCAL__*` method (not a separate feature from `local.*` config).
