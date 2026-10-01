# SigNoz Operations Workflow

Use this guide to keep a running SigNoz instance's telemetry complete and durable: setting retention, stopping and restarting, and upgrading the pinned version.

## Purpose

The dashboards read raw rows from SigNoz's ClickHouse database. Lost data parts, expired rows, or duplicated inserts change dashboard numbers without any query changing. Duplicated inserts already happened once in this repository's history (see `audit.md`, "Retried trace and metric inserts duplicated rows"). This guide condenses the `setup-signoz-telemetry` skill's retention, safe-operation, and troubleshooting workflows (`.agents/skills/setup-signoz-telemetry/references/`).

## Preconditions

- SigNoz deployed as described in `ops/signoz/README.md`, including its three local adjustments.
- Access to the SigNoz settings API or UI and to the ClickHouse container.
- Explicit approval before deleting volumes, data directories, or detached parts.

## Steps

### Set retention

1. Read the current logs, traces, and metrics retention. Defaults can be short, for example 15 days for logs and traces.
2. Set retention through SigNoz's supported setting, not by editing tables directly.
3. Confirm that existing data carries the new expiry, not just new rows. Logs can store retention per row, and existing parts keep the expiry calculated when they were written. Recalculate part expiry if needed.
4. Before recalculating expiry, confirm no part carries an expiry that has already passed. Merging such parts can drop live rows.

### Stop or restart

1. Before stopping the stack, its container runtime, or the host: stop database merges, wait for running merges to finish, flush logs, and sync filesystem writes.
2. Keep the host awake during long ingestion or maintenance. Sleep can suspend the runtime mid-write, especially on external storage.
3. After restart, confirm merges resume and no replication entries are stuck waiting for missing parts.

### Upgrade SigNoz

1. Change the tags in `ops/signoz/versions.env`.
2. Re-run query validation for every dashboard.
3. Apply the upgrade deliberately and record the change in the commit. Never upgrade as a side effect of setup.

## Durability Settings

| Setting | Use it? | Why |
| --- | --- | --- |
| `min_rows_to_fsync_after_merge = 1` in ClickHouse `config.xml` | Yes | Merged parts reach disk, so a stop cannot lose them |
| `fsync_after_insert`, `fsync_part_directory` | No, on slow or external storage | They slowed trace inserts to about 8.6 s at p95, close to the collector's 9 s timeout. Retried batches were written twice and inflated span-based panels 2–7× |
| `max_allowed_data_age_days: 3650` on `clickhouselogsexporter` | Yes | Accepts delayed or backfilled logs that the default would drop |

## What To Check

- Each signal's retention setting matches what you requested, and existing parts carry consistent expiries.
- After a restart: no lost or stuck parts, merges running, and no new duplicate span IDs.
- Running image tags match `ops/signoz/versions.env`.
- If data looks missing, check in dependency order: container health and restart counts, listeners and synthetic ingestion, retention, the replication queue, then producer configuration. Repeated ClickHouse restarts usually mean memory exhaustion.

## Related Docs

- `README.md`: install and first run
- `ops/signoz/README.md`: the version pin and local adjustments
- [`dashboard-change-workflow.md`](dashboard-change-workflow.md): re-validating dashboards after an upgrade
