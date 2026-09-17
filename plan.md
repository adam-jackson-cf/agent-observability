# Dashboard Anomaly Remediation Plan

## Objective

Correct the six confirmed data-selection and semantic anomalies without redesigning the dashboard. Preserve the visual framing reached through prior iteration and change only the source boundaries, series definitions, labels, and hints required to make each existing visual truthful.

## Source of truth

- Canonical dashboard: `caching-all-model-usage.json`
- Evidence and issue status: `audit.md`
- Target dashboard: `01a0a00b-9c1d-7d86-89c2-d7ee7bc0c150`
- Separate Codex dashboard `01a09fae-7dda-794d-bbdb-35d73d72d624` must remain unchanged.

## Change budget

- No new panels or panel removals.
- No layout reflow.
- No chart-type changes.
- No new telemetry or runtime infrastructure.
- No checked-in generator or second dashboard source of truth.
- No changes to panels that passed the audit.
- Every proposed edit must map directly to one anomaly in `audit.md`.

## Qualification approach

- Different producers may use different authoritative events, spans, and schemas to surface the same operational scenario.
- Producer-native health signals remain separate series and are interpreted against their own history, not as a benchmark.
- Producers may be aggregated only when values are mathematically compatible after normalization.
- Missing coverage is no data, never an inferred zero.
- Information hints disclose producer boundaries where they differ.

## Mandatory KISS review gate

Before modifying `caching-all-model-usage.json`:

1. Prepare a widget-ID keyed allowlist mapping every anomaly to the exact query, label, and hint fields that may change; all other fields are immutable.
2. Prepare the concrete query and copy changes for those allowed fields.
3. Invoke the read-only `kiss` agent to review that exact proposal for unnecessary abstractions, scope expansion, duplicate machinery, and changes not required by `audit.md`.
4. Remove or simplify every challenged element unless repository evidence demonstrates it is necessary.
5. Implementation may begin only after this gate is complete.

## Planned corrections

### Codex model-catalog misclassification

- Preserve both native-health panels and their visual framing.
- Restrict Codex outcomes to `/responses` with nonempty conversation and model identity.
- Retain the error-message fallback when the success boolean is absent.
- Rename only the Codex series/measurement boundary to `Codex response attempt`; preserve both panel titles and disclose app-server-only current coverage and attempt semantics in the hint.
- Keep `/models` traffic out of model-operation health.

### Oh My Pi source mismatch

- Preserve all affected visuals, units, layouts, and token/cache arithmetic.
- Replace legacy Oh My Pi `api_request` rows with authoritative nonzero native chat spans.
- Apply one reviewed normalized `usage_events` SQL shape directly to every affected widget in the same cutover.
- Keep `caching-all-model-usage.json` as the only maintained artifact.

### Invalid model-operation count

- Preserve both cache tables and their usage/cache columns.
- Count one completed operation per accepted normalized usage record.
- Rename `Model calls` to `Completed operations` and document producer-native completion boundaries.

### Overlapping daily token stack

- Preserve the stacked chart and its sub-day blank behavior.
- Use mutually exclusive uncached input, cached input, and output series.
- Explain that stack height represents total processed tokens.

### Incomplete daily cache trend

- Preserve the percentage line, units, domain, and sub-day blank behavior.
- Aggregate all normalized producer usage per day before calculating cache utilization.
- Return one point for every usage-bearing day.

### Ambiguous session labels

- Preserve both query populations, layouts, and visual types.
- Rename the panels to `Usage-Active Sessions` and `Usage-Active Sessions Over Time`.
- Explain identity keys, harness scoping, and per-bucket activity semantics in hints.

## Implementation sequence

1. Confirm deployed/local JSON equality, then define a widget-ID keyed allowlist of editable fields.
2. Prepare the exact affected-widget proposal and complete the mandatory KISS review gate.
3. Correct the two native-outcome queries, series labels, and hints.
4. In one atomic edit, apply the normalized usage source, completed-operation counting, mutually exclusive daily token series, daily cache aggregation, and session title/hint corrections across every `usage_events`-dependent widget.
5. Validate direct ClickHouse results, every changed SigNoz query, and every cross-panel parity relation named in `audit.md`.
6. Deploy the exact canonical JSON through the established deployment path and require a fresh GET structurally equal to the local artifact.
7. Verify the affected panels and information hints, then require the canonical JSON diff to contain only allowlisted widget fields.
8. Mark each `audit.md` issue `Fixed` independently only after its own acceptance criteria pass.

## Acceptance criteria

### Preservation

- Widget IDs and layouts are unchanged.
- Panel count remains 16.
- Existing chart/table types and units are unchanged.
- The canonical JSON diff contains only allowlisted widget fields.

### Data and semantic integrity

- Codex outcome panels contain no `/models` records.
- OMP usage comes exclusively from qualifying native chat spans.
- KPI totals equal the corresponding daily, cache-table, and session-detail totals for the same window.
- Completed-operation totals agree between cache tables and Session Details.
- Cached input never exceeds total input.
- Daily uncached input plus cached input equals daily total input.
- Every usage-bearing day appears exactly once in the daily cache trend.
- Usage-active session KPI equals distinct harness/session pairs represented by Session Details.
- Missing producer coverage renders as no data, not zero.

### Runtime and visual verification

- Every changed deployed query returns HTTP 200, success, and no warnings or errors over 30 days.
- Both daily charts remain blank over six hours.
- Information hints exactly describe producer-specific boundaries.
- No red/yellow hazards, query errors, clipped tables, or misleading legends appear.
- Deployed JSON structurally equals the canonical local artifact.

