---
name: shell-skill
description: Use when writing or reviewing shell scripts, Bash helpers, Docker cron scripts, backup wrappers, env-file-driven automation, or shell-based operational workflows. Do not use for one-off terminal commands unless they are being turned into reusable scripts.
---

# Shell Skill

Write shell scripts as small, explicit helper scripts. Prefer boring command wrappers that are easy to read in one pass.

## Core Rules

- Use `#!/usr/bin/env bash`.
- Use `set -Eeuo pipefail`.
- Keep scripts simple and direct.
- Do not add clever shell machinery unless it is clearly needed.
- Do not add defaults for required env vars.
- Do not duplicate variable definitions across env files.
- Do not introduce script arguments when the value already exists in env.
- Do not hide behavior behind generic helper layers.
- Do not print secrets.
- Run `bash -n` and `shellcheck` after edits.

## Env Variables

Configuration should come from env files, not from script defaults.

Good:

```bash
echo "BARMAN_RETENTION_POLICY: ${BARMAN_RETENTION_POLICY}"

exec /usr/local/bin/barman-cloud-backup-delete \
  --retention-policy "${BARMAN_RETENTION_POLICY}" \
  "${BARMAN_DESTINATION_URL}" \
  "${BARMAN_SERVER_NAME}"
```

Avoid:

```bash
retention_policy="${RETENTION_POLICY:-RECOVERY WINDOW OF 14 DAYS}"
barman-cloud-delete-old.sh "${retention_policy}"
```

If a script knows a value from env, read it from env directly. Do not pass it as `$1`.

Allowed special env namespaces:

- `AWS_*` for AWS/boto/S3-compatible SDK behavior.
- `POSTGRES_*` for PostgreSQL connection settings.
- `PATH` when cron or entrypoints need it.
- Domain-specific config should use the domain prefix, for example `BARMAN_*`.

For `.env.barman`, project-owned variables should be `BARMAN_*`. AWS SDK variables may stay `AWS_*`.

## Script Arguments

Use script arguments only for values that are naturally one-off runtime input and are not already configured in env.

Good argument use:

```bash
WAL_PATH="$1"
```

because PostgreSQL passes the WAL path to `archive_command`.

Avoid argument use:

```bash
RETENTION_POLICY="$1"
BACKUP_NAME="$1"
```

when cron or env can set `BARMAN_RETENTION_POLICY` or `BARMAN_BACKUP_NAME`.

## Command Style

Prefer direct commands over arrays and incremental argument building.

Good:

```bash
# shellcheck disable=SC2086
PGPASSWORD="${POSTGRES_PASSWORD}" exec /usr/local/bin/barman-cloud-backup \
  --name "${BARMAN_BACKUP_NAME}" \
  --cloud-provider "${BARMAN_CLOUD_PROVIDER}" \
  --read-timeout "${BARMAN_READ_TIMEOUT}" \
  --addressing-style "${BARMAN_ADDRESSING_STYLE}" \
  --jobs "${BARMAN_BACKUP_JOBS}" \
  --host "${POSTGRES_HOST}" \
  --port "${POSTGRES_PORT}" \
  --user "${POSTGRES_USER}" \
  --dbname "${POSTGRES_DB}" \
  --aws-region "${AWS_DEFAULT_REGION}" \
  --endpoint-url "${BARMAN_ENDPOINT_URL}" \
  --verbose \
  ${BARMAN_BACKUP_COMPRESSION} \
  "${BARMAN_DESTINATION_URL}" \
  "${BARMAN_SERVER_NAME}"
```

Avoid:

```bash
args=()
args+=("--cloud-provider" "${BARMAN_CLOUD_PROVIDER}")
args+=("--read-timeout" "${BARMAN_READ_TIMEOUT}")
exec some-command "${args[@]}"
```

Only use arrays when there is a real correctness need that cannot be expressed cleanly as a direct command.

## Logging

Echo useful non-secret config before running important commands.

Good:

```bash
echo "BARMAN_CLOUD_PROVIDER: ${BARMAN_CLOUD_PROVIDER}"
echo "BARMAN_ENDPOINT_URL: ${BARMAN_ENDPOINT_URL}"
echo "BARMAN_DESTINATION_URL: ${BARMAN_DESTINATION_URL}"
echo "BARMAN_SERVER_NAME: ${BARMAN_SERVER_NAME}"
```

Never echo:

- passwords
- access keys
- secret keys
- tokens
- cookies
- private URLs containing credentials

## Cron Scripts

Cron scripts should be explicit job scripts, not a generic scheduler framework.

Good structure:

```text
postgres-barman-cron-backup.sh
postgres-barman-cron-cleanup.sh
postgres-barman-cron-lib.sh
```

Rules:

- The cron schedule file should call focused scripts.
- Shared lock logic can live in a small lib script.
- Job config should come from env files.
- If cron creates a runtime value, export it with the domain prefix:

```bash
BARMAN_BACKUP_NAME="${BARMAN_BACKUP_NAME_PREFIX}-$(date -u +%Y%m%dT%H%M%SZ)"
export BARMAN_BACKUP_NAME
```

## Safety

Keep safety checks that prevent data loss.

Good:

```bash
if [[ -d "${RESTORE_DIR}" ]] && [[ -n "$(find "${RESTORE_DIR}" -mindepth 1 -maxdepth 1 2>/dev/null)" ]]; then
  echo "ERROR: restore directory is not empty: ${RESTORE_DIR}" >&2
  echo "Refusing to overwrite existing PGDATA." >&2
  exit 1
fi
```

Avoid removing validation just to make the script shorter.

## Helper Scripts

Helper scripts should do one job:

- backup
- cleanup
- list
- restore
- archive WAL
- run one cron job

Do not turn helper scripts into generic command routers.

Avoid:

```bash
case "$1" in
  backup) ... ;;
  cleanup) ... ;;
  restore) ... ;;
esac
```

Prefer separate scripts with direct names.

## Existing Script Edits

When editing existing `scripts/*.sh` files in this repo, do not delete existing content unless explicitly asked. Prefer appending or extending helpers. For newly added infrastructure scripts, keep the file small and focused from the start.

## Verification

After changing shell scripts, run:

```bash
bash -n path/to/script.sh
shellcheck path/to/script.sh
```

For Docker or cron scripts, also verify the image or Compose service that installs them:

```bash
docker compose -f infra/docker/docker-compose.yml -f infra/docker/docker-compose.user.yml config --services
just docker-build cron
```
