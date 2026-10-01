# Safe operation workflow

## Objective

Stop, restart, or relocate the stack without losing recently written telemetry.

## Required actions

1. Before stopping the stack, its container runtime, or the host:
   - Stop database merges.
   - Wait for running merges to finish.
   - Flush logs.
   - Sync filesystem writes.
   An abrupt stop can lose the newest data parts and leave the database expecting parts that no longer exist.
2. Keep the host awake during long ingestion, recovery, or maintenance work. Sleep can suspend the runtime mid-write, especially when its data sits on external storage.
3. Sync merged parts to disk (see `ops/signoz/README.md`) so stops cannot lose them, and verify the setting after a restart. Do not enable insert-path sync on slow or external storage: it can push inserts past the collector's timeout, and the collector's retries then duplicate telemetry.
4. Confirm merges resume after restart, and that no replication entries are stuck waiting for missing parts.
5. Never delete telemetry volumes, data directories, or set-aside parts without explicit approval.

## Done when

- The stack stops and restarts with no newly lost or stuck data parts.
- Any durability setting or quiesce step used is reported.
