# SigNoz deployment

The repository targets a vanilla SigNoz instance, built from the official deployment files
at the release pinned in [`versions.env`](versions.env). Nothing machine-specific is stored
here. You choose the deployment directory, data location, and credentials when you set it
up.

## Set up

1. Clone `https://github.com/SigNoz/signoz` at `SIGNOZ_DEPLOY_REF` into a directory of your
   choice. Use its `deploy/docker` files.
2. Export the variables from `versions.env`, so the compose file's `VERSION` and
   `OTELCOL_TAG` resolve to the pinned images.
3. Copy [`docker-compose.override.example.yaml`](docker-compose.override.example.yaml) to
   `deploy/docker/docker-compose.override.yaml`. Supply its variables from a secret store or
   the shell.
4. Apply the local adjustments below, then start the stack with `docker compose up -d` from
   `deploy/docker`.
5. Follow the `setup-signoz-telemetry` skill: set retention, prove synthetic ingestion, then
   configure producers.

## Local adjustments

These are the changes the repository's dashboards and workflows were validated with.

| File (relative to the SigNoz checkout) | Change | Why |
|---|---|---|
| `deploy/docker/docker-compose.yaml`, `otel-collector` command | Run `/signoz-otel-collector --config=/etc/otel-collector-config.yaml` without `--manager-config` and `--copy-path` | Manager mode can rewrite the effective collector configuration |
| `deploy/docker/otel-collector-config.yaml`, the `clickhouselogsexporter` exporter | Add `max_allowed_data_age_days: 3650` | Accepts delayed or backfilled logs that the default would drop |
| `deploy/common/clickhouse/config.xml` | Add `<merge_tree><min_rows_to_fsync_after_merge>1</min_rows_to_fsync_after_merge></merge_tree>` | Syncs merged parts to disk, so stopping the runtime does not lose them |

Do not enable insert-path fsync (`fsync_after_insert`, `fsync_part_directory`) on slow or
external storage. It can push inserts past the collector's timeout, and the collector's
retries then duplicate telemetry.

## Upgrading

Change `versions.env`, re-run query validation for every dashboard, and apply the upgrade
deliberately. Never upgrade as a side effect of setup.
