# Dashboard scope workflow

## Objective

Resolve the target dashboard, the requested change, and its authoritative signal contract.

## Required actions

1. Decide whether the request creates a new dashboard or updates an existing one.
2. For an update, find the dashboard JSON at the repository root and the signal reference whose `Source of truth` line names it.
3. For a new dashboard, choose a new root-level JSON name and a matching `<topic>-signal-reference.md`. Both are created in this change.
4. List the requested producers, signals, and any panels to add, change, or retire.
5. Stop if a requested signal has no documented producer contract and cannot be confirmed from live data in Step 2.

## Done when

- One dashboard and one signal reference are selected or planned.
- Missing contracts are reported as explicit blockers.
