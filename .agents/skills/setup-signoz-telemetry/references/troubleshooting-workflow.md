# Troubleshooting workflow

## Objective

Diagnose telemetry failures in dependency order without weakening safety controls.

## Required actions

1. Check the container runtime, service health, and restart counts. Repeated database restarts usually mean memory exhaustion; find the query or background merge responsible before running more work.
2. Check listeners, collector pipelines, and synthetic ingestion.
3. Check retention: data that seems to be missing may simply have expired.
4. Check the database replication queue for stuck entries waiting on missing parts. Inspect set-aside (detached) parts before clearing anything, and recalculate part expiry first (see the retention workflow).
5. Check producer syntax, endpoint compatibility, and fresh-process state.
6. Query recent backend evidence with bounded, read-only queries, and report the first failed boundary.

## Done when

- The failure is narrowed to one boundary.
- No security, retention, or telemetry-content safeguard was bypassed.
