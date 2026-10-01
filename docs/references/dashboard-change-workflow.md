# Dashboard Change Workflow

Use this guide when you add, change, or fix a dashboard panel. It keeps the dashboard, its signal reference, the running SigNoz copy, and `audit.md` consistent with each other.

## Purpose

Each dashboard has a signal reference that defines what every panel means. Dashboards drift when panels are edited without that contract, or when a running SigNoz copy diverges from the repository file. This workflow is the path the `develop-signoz-dashboard` skill (`.agents/skills/develop-signoz-dashboard/`) follows.

## Preconditions

- A running SigNoz at the version pinned in `ops/signoz/versions.env`, with live producer data for the signals you are changing.
- The SigNoz URL and credentials, supplied at run time. Never commit them.
- The dashboard JSON and the signal reference whose `Source of truth:` line names it.

## Steps

1. **Bind the scope.** Name the one dashboard file and its signal reference. If a requested signal has no contract in the reference, stop and define one first.
2. **Update the contract first.** Record new or changed producer fields, operation boundaries, formulas, dimensions, and empty-state meaning in the signal reference. Confirm each field exists in live data.
3. **Design the panel.** Specify visualization, formula, grouping, filters, variables, unit, legend, and no-data behavior. Reject any cross-producer comparison whose operation boundaries do not match.
4. **Edit the JSON.** Keep the IDs of unchanged panels stable, and give replaced panels new IDs. Leave out instance-specific values: URLs, dashboard route IDs, paths, credentials.
5. **Validate offline, then against data.** Check structure, IDs, layout, variables, and portability. Run every panel query against the backend with bounded, read-only execution.
6. **Apply only when asked.** Find the persisted dashboard by its document `uuid`. Back up the persisted response outside the repository, then update that route in place. Never import a copy.
7. **Verify in SigNoz.** Run every changed panel through SigNoz with explicit values for every variable. Report visual checks separately from query checks.

## Recording Defects In `audit.md`

Each finding in `audit.md` is one `##` section with:

- `**Status:**`: one of `Identified`, `Investigating`, `Fixed`
- `**Preservation scope:**`: what the fix must not change (panel IDs, titles, layouts, chart types, units)
- `**Affected panels:**`, `**Finding:**`, `**Evidence:**`, `**Impact:**`
- The validated fix and its validation result

Mark a finding `Fixed` only after its own validation passes against live data. Fix data incidents at their cause, not by deduplicating inside individual panels, so that a recurrence stays visible. The duplicated-insert incident in `audit.md` is an example.

## What To Check

- The persisted dashboard equals the repository file, and its route ID, `uuid`, and the total dashboard count are unchanged.
- Cross-panel parity holds: KPI totals equal the matching daily, cache-table, and session-detail totals for the same window.
- Missing producer coverage shows as no data, never as zero.
- Information hints describe each producer's operation boundary where producers differ.

## Related Docs

- `README.md`: project purpose and setup
- [`dashboard-catalog.md`](dashboard-catalog.md): which dashboard and producer supply each signal
- `.agents/skills/develop-signoz-dashboard/references/`: the detailed step workflows
