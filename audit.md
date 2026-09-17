# Dashboard Semantic Audit

**Status values:** `Identified` · `Investigating` · `Fixed`

## Codex model-catalog requests misclassified as model-operation health

**Status:** Fixed

**Preservation scope:** Keep both panel IDs, titles, layouts, chart/table types, units, short legends, and native-health framing. Change only the Codex population filter, measurement label, and explanatory hint required to make that framing truthful.

**Affected panels:**

- Native Operation Success / Failure
- Native Operation Failures Over Time

**Finding:** The Codex branches select every `codex.api_request` event. That population includes `/models` model-catalog refresh attempts as well as `/responses` model-serving attempts, so the panels do not currently represent model-operation health.

**Evidence (30-day window):**

- All 83 `codex_cli_rs` failures target `/models`, have no conversation or model identity, and occurred within one nine-minute interval on 2026-09-14.
- Retry attempts 0–4 occur 17/17/17/16/16 times, so approximately 17 logical catalog refreshes became 83 charted failures.
- Related `list_models` spans report no terminal span errors because Codex catches online refresh failures and continues with its bundled or cached catalog.
- Codex `/models` population: app-server 7,127 success / 174 failure; exec 1,121 success / 0 failure; CLI 0 success / 83 failure.
- Codex `/responses` population: app-server 29 success / 1 failure; exec and CLI have no qualifying records.

**Impact:** The apparent Codex CLI 100% failure rate and the largest Codex failure peaks describe catalog-refresh transport attempts, not failed user model operations.

**Recommended correction:** Restrict the Codex branches in both affected panels to `attributes_string['endpoint'] = '/responses'`. Keep `/models` discovery health separate if it is useful operationally. P95 Native Operation Duration is unaffected because it uses sampling spans rather than `codex.api_request` logs.

**Known evidence gap:** The exact network cause of the `/models` send failures is unavailable; the telemetry contains a generic transport error and no HTTP status.

**Validated sustainable fix:**

- Restrict Codex outcome records to `attributes_string['endpoint'] = '/responses'`.
- Require nonempty `conversation.id` and `model` so the population has model-operation identity.
- Preserve the current fallback outcome rule: use `attributes_bool['success']` when present; otherwise treat an absent `error.message` as success. Current `/responses` records omit the boolean, so boolean-only logic would falsely mark all 30 records as failures.
- Rename the measurement to `Codex response attempt` and disclose that it is attempt-level.
- Treat exec and CLI as unavailable/no data when they emit no qualifying `/responses` records; do not render them as zero failures.
- Keep `/models` catalog-refresh health separate.

**Validated 30-day result after correction:** app-server 29 success / 1 failure; exec and CLI no qualifying data.

## Oh My Pi usage sources disagree across panels

**Status:** Fixed

**Preservation scope:** Keep every affected panel, layout, chart type, unit, and established token/cache arithmetic. Replace only the legacy Oh My Pi source branch and the minimum query structure needed to union logs and spans consistently.

**Affected panels:**

- Input Tokens
- Output Tokens
- Cached Input
- Uncached Input
- Input Cache Utilization
- Daily Token Consumption
- Daily Input Cache Utilization
- Cache Efficiency by Harness
- Cache Efficiency by Model

**Finding:** These nine aggregate panels use legacy Oh My Pi `api_request` logs, while the four session-usage panels use authoritative native `chat` spans. The two sources have materially different populations and totals.

**Evidence (30-day window):**

- Aggregate panels include 45 legacy Oh My Pi requests, last observed on 2026-09-13: 2.93M input, 2.61M cached input, and 38.9K output tokens.
- Session panels include 51,518 native chat spans through 2026-09-16: 6.680B input, 3.216B cached input, and 20.577M output tokens.
- Current dashboard totals versus totals using the authoritative Oh My Pi source:
  - Input: 373.9M versus 7.049B (`-94.7%`).
  - Output: 1.713M versus 22.246M (`-92.3%`).
  - Cached input: 314.4M versus 3.527B (`-91.1%`).
  - Uncached input: 59.5M versus 3.522B (`-98.3%`).
  - Cache utilization: 84.1% versus 50.0%.

**Impact:** Aggregate KPIs, daily charts, and cache tables disagree with Session Details and materially understate Oh My Pi activity.

**Required correction:** Use the native Oh My Pi chat-span population consistently in all usage panels.

**Validated sustainable fix:**

- Replace every legacy Oh My Pi `api_request` branch with the native chat-span boundary already used by Session Details:
  - `serviceName = 'oh-my-pi'`
  - `gen_ai.operation.name = 'chat'`
  - nonempty conversation and model
  - token-bearing, nonzero usage
- Normalize total input as direct input + cache creation input + cache-read input; cached input is cache-read input; output is output tokens.
- Exclude `invoke_agent` and child aggregate spans. Current qualifying span IDs are unique and every row satisfies cached input <= total input.
- Normalize all producers into one canonical `usage_events` schema: timestamp, harness, native session, provider, model, input, cached input, output, and completed operation.
- Derive every usage KPI, daily chart, cache table, and session panel from that schema. Apply the same reviewed `usage_events` fragment directly to each affected widget in one atomic JSON update; do not introduce a separate generator without an established repository contract.
- Add parity checks requiring KPI totals to equal daily, cache-table, and session-detail totals for the same time window.

**Expected visual consequence:** aggregate totals will increase substantially but will become internally consistent with Session Details.

## Cache tables count tool results as Codex model calls

**Status:** Fixed

**Preservation scope:** Keep both cache tables and every existing usage/cache column. Change only the invalid operation-count source, its column label, and the producer-boundary hint.

**Affected panels:**

- Cache Efficiency by Harness
- Cache Efficiency by Model

**Finding:** The `Model calls` column uses `count_distinct(call_id)` for Codex, but every nonempty Codex `call_id` in the measured population belongs to `codex.tool_result` events rather than completed model responses.

**Evidence (30-day window):**

- App-server: 2,706 unique tool-result `call_id` values versus 1,576 token-bearing completed SSE rows.
- Exec: 4,605 unique tool-result `call_id` values versus 4,157 token-bearing completed SSE rows.
- Token-bearing SSE completion rows have empty `call_id`.
- Oh My Pi contributes only 45 legacy request rows to the table rather than 51,518 native chat spans because of the source mismatch above.

**Impact:** The column labelled `Model calls` is primarily a Codex tool-result count and is not comparable with Claude Code or Oh My Pi model-operation counts.

**Required correction:** Count the accepted token-bearing model-usage event boundary for each producer rather than Codex `call_id`.

**Validated sustainable fix:**

- Set `completed_operations = 1` on every accepted canonical usage record and sum it consistently in cache tables and session panels.
- Codex counts token-bearing `codex.sse_event` completion records; Claude Code counts `api_request` records; Oh My Pi counts terminal chat spans.
- Rename `Model calls` to `Completed operations` or `Completed model operations`.
- Explain in the information hint that this is a producer-native completed-usage boundary, not a shared provider request identifier.
- Do not deduplicate Codex by payload: current log IDs are unique, and three distinct records with identical payload tuples may be legitimate.
- Assert that cache-table completed-operation totals equal Session Details totals for the same window.

## Daily Token Consumption stacks overlapping quantities

**Status:** Fixed

**Preservation scope:** Keep the panel ID, title, stacked chart, layout, units, daily bucketing, and blank-under-one-day behavior. Change only the overlapping series definitions and the hint that explains stack height.

**Affected panel:** Daily Token Consumption

**Finding:** The graph is stacked, but `Cached input` is already a subset of `Input tokens`.

**Impact:** Stack height double-counts cached tokens and visually presents input and cached input as additive categories.

**Required correction:** Either disable stacking or display mutually exclusive series such as uncached input, cached input, and output.

**Validated sustainable fix:**

- Keep the stacked visual, but use three mutually exclusive series from canonical `usage_events`: uncached input, cached input, and output.
- Define uncached input as total input minus cached input. The stack height then represents total processed tokens (input + output) without overlap.
- Preserve the explicit blank result for ranges shorter than one day.
- Update the information hint to state that the stack is mutually exclusive and that its height is total processed tokens.
- Assert for every day that uncached input + cached input = total input, and that daily sums equal the corresponding KPI totals.

**Validated 30-day result:** 12 daily points with zero partition violations.

## Daily cache utilization suppresses valid Codex days

**Status:** Fixed

**Preservation scope:** Keep the panel ID, title, percentage line, layout, units, 0–100 interpretation, and blank-under-one-day behavior. Change only the query composition needed to retain every valid usage-bearing day.

**Affected panel:** Daily Input Cache Utilization

**Finding:** The formula combines separately grouped Codex and Claude/Oh My Pi series. SigNoz aligns the series only where every operand exists, so days without legacy Claude/Oh My Pi `api_request` rows disappear even when Codex has valid input and cache data.

**Evidence (30-day window):**

- Codex has qualifying daily input/cache data on 12 days.
- The panel returns only four daily points.
- Those four dates exactly match the four dates containing Claude/Oh My Pi legacy `api_request` rows.

**Impact:** Eight valid Codex days are omitted, making the daily trend appear sparser and potentially hiding cache-utilization changes.

**Required correction:** Aggregate all producer branches in SQL before calculating the daily ratio, so missing producer rows contribute zero rather than removing the date.

**Validated sustainable fix:**

- Replace the multi-query builder formula with one direct SQL query over canonical `usage_events`.
- Aggregate daily input and cached input across all producer branches first, then calculate `100 * cached_input / nullIf(input_tokens, 0)`. This produces a correctly weighted ratio and prevents missing producer rows from suppressing dates.
- Keep the percentage line visual with a 0–100 domain.
- Preserve the explicit blank result for ranges shorter than one day.
- State in the hint that each point is the cache-read share of total normalized input for that day.
- Assert that every usage-bearing day appears once, cached input never exceeds input, daily sums match KPI totals, and sub-day ranges return no data.

**Validated result:** 12 daily points over 30 days and zero points over six hours.

## Distinct Sessions descriptions omit usage qualification

**Status:** Fixed

**Preservation scope:** Keep both panel IDs, layouts, visual types, adaptive bucketing, and usage-qualified query population. Change only the titles and hints needed to state what the existing calculation actually measures.

**Affected panels:**

- Distinct Sessions
- Distinct Sessions Over Time

**Finding:** The panels count only sessions containing qualifying token-usage events, while their descriptions claim to count distinct native sessions generally.

**Evidence (30-day window):**

- Observed native sessions: 2,691.
- Usage-qualified sessions: 2,234.
- Excluded by producer: Claude Code 258, Oh My Pi 178, app-server 4, exec 13, CLI 4.

**Impact:** The displayed value is valid for usage-active sessions but understates all observed native sessions by 457.

**Required correction:** Either rename/describe the measure as usage-active sessions or broaden the query to the full native-session population.

**Validated sustainable fix:**

- Keep the usage-qualified population; broad lifecycle/session records are not a comparable model-usage population across producers.
- Rename the panels to `Usage-Active Sessions` and `Usage-Active Sessions Over Time`.
- Define the KPI as distinct `(harness, native_session_id)` pairs with at least one accepted usage event in the selected range.
- Define each time-series point as distinct pairs with activity in that bucket. It is neither session starts nor concurrency, and one session may appear in multiple buckets.
- Disclose identity keys in the hints: Codex `conversation.id`, Claude Code `session.id`, and Oh My Pi `gen_ai.conversation.id`; identity is scoped by harness.
- Assert that the KPI equals distinct harness/session pairs in Session Details, every bucket is a subset of the range-level KPI, IDs are nonempty, and harness scoping prevents collisions.

## Cross-cutting implementation contract

**Status:** Fixed

- Use one reviewed canonical `usage_events` SQL fragment as the implementation source for every affected direct-SQL widget in `caching-all-model-usage.json`.
- Remove mixed builder-formula and hand-divergent SQL paths for the affected usage panels.
- Validate exact query results through SigNoz and enforce cross-panel parity for totals, completed operations, daily coverage, and session populations.
- Keep producer-boundary differences in information hints while ensuring each visual combines only quantities that are mathematically and semantically compatible.
- None of these fixes has been applied to the dashboard yet.

## Recommended implementation sequence

**Status:** Fixed

1. Prepare and validate the canonical `usage_events` SQL fragment and each direct-widget query in a temporary implementation harness; keep `caching-all-model-usage.json` as the sole maintained artifact.
2. Migrate all token, cache, daily, cache-table, and session queries in one atomic cutover so no panel remains on the legacy Oh My Pi source.
3. In that cutover, replace `Model calls` with `Completed operations`, use mutually exclusive daily token series, calculate daily cache utilization in direct SQL, and rename the two session-count panels.
4. Independently constrain the two native-outcome panels to identified Codex `/responses` attempts and update their measurement labels and information hints.
5. Validate direct ClickHouse results, every SigNoz query response, cross-panel parity, 30-day rendering, six-hour blank daily charts, information hints, and exact deployed/local equality before marking any anomaly `Fixed`.
