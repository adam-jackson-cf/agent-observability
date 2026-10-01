# Query validation workflow

## Objective

Prove every panel query runs and returns plausible results against the live backend, without endangering the shared SigNoz instance.

## Required actions

1. Execute each panel query read-only against the backend.
   - Substitute the time range and variables exactly as SigNoz would: lists as tuples, text boxes as quoted strings.
   - Use the representative range plus one shorter than any minimum-range guard.
2. Bound every validation query: set a per-query memory cap, a thread limit, and a time window. Run queries one at a time.
3. For anti-joins or `IN (subquery)` over large identifier sets, disable skip-index use. Bloom-filter index analysis on large sets can exhaust server memory.
4. Avoid creating background work on the shared server. Staging tables must not merge, or must be small.
5. Check results for plausibility:
   - Totals reconcile across panels for the same window.
   - Ratios stay in range.
   - Producers that report nothing show not reported rather than zero.
6. If the server restarts or a query exceeds its limits, stop. Find the cause before running more queries.

## Done when

- Every panel query succeeds over the representative and guard ranges.
- Cross-panel reconciliation checks pass, or their differences are documented in the signal reference.
