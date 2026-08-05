# Configurable project mount (`WORK_MOUNT`)

By default the project bind-mounts at **`/work`** inside containers. You can
change that path with **`WORK_MOUNT`** (alias: **`WORKSPACE`**).

## Default

| | |
|--|--|
| Host path | Cluster root (compose `.`) or ubuntu-mise `PROJECT` |
| Container path | **`/work`** |

## Override

```bash
# one-shot
WORK_MOUNT=/app task warm
WORK_MOUNT=/app task up:all

# persistent (cluster or ubuntu-mise)
# .mise.env
WORK_MOUNT=/app
```

`bin/compose` writes `WORK_MOUNT` into generated `.env` so compose volumes and
`working_dir` stay aligned.

## What must match

| Place | Uses |
|-------|------|
| Compose volume target | `${WORK_MOUNT:-/work}` |
| Compose `working_dir` | `${WORK_MOUNT:-/work}` or `…/fred` |
| Container env | `WORK_MOUNT` / `WORKSPACE` |
| `bin/docker-app` | resolves cluster root from `WORK_MOUNT` |
| `bin/warm` | mounts and crawls `WORK_MOUNT` |
| mise trust | `MISE_TRUSTED_CONFIG_PATHS` should include the mount (image defaults to `/work`; override when you change mount) |

## Example compose (cluster)

```yaml
x-app: &app
  working_dir: ${WORK_MOUNT:-/work}
  environment:
    WORK_MOUNT: ${WORK_MOUNT:-/work}
    WORKSPACE: ${WORK_MOUNT:-/work}
  volumes:
    - .:${WORK_MOUNT:-/work}:cached
    - cache:/cache
  command: ["bash", "-lc", "exec $${WORK_MOUNT}/bin/docker-app"]
```

## Notes

- Default remains **`/work`** — existing projects need no change.
- Do not use a trailing slash (`/work/` vs `/work`).
- Shared gem `local-gem-env` paths use the mount root (`/app/wizardry_shared` when `WORK_MOUNT=/app`).
