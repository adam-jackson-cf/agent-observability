# End-to-end validation workflow

## Objective

Prove fresh telemetry from each configured producer reaches SigNoz with its expected schema.

## Required actions

1. Restart or launch a fresh producer process.
2. Generate one bounded operation.
3. Query records newer than the verification start time.
4. Check service identity, signal type, event boundary, and required fields.

## Done when

- Fresh producer-specific telemetry is observed.
- Missing or mismatched fields are explicit.
