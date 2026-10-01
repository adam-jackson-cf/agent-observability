# Retention workflow

## Objective

Set each signal's retention deliberately, and prove that existing data keeps it.

## Required actions

1. Read the current retention for logs, traces, and metrics through SigNoz's settings API or UI. Default retention can be short, for example 15 days for logs and traces.
2. Set the requested retention through the supported setting, not direct table edits, whenever SigNoz manages that table.
3. Verify existing data as well as the setting:
   - Logs can store retention per row. A changed setting may apply only to new rows, and the per-row value can be part of the partition key.
   - Existing parts keep the expiry calculated when they were written. Check part-level expiry metadata, and recalculate it if needed.
4. Before recalculating expiry or clearing stuck replication entries, confirm no data part carries an expiry that has already passed. A merge of such parts can drop live rows.
5. Report tables that SigNoz settings do not manage, such as attribute-key and metadata tables. Change them only with approval.
6. Record the expected storage growth for the new retention.

## Done when

- Every signal's retention setting matches the request.
- Existing data parts carry expiries consistent with the new retention.
