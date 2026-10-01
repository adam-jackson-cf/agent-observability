# Model usage signal reference

Source of truth: `caching-all-model-usage.json` (`All Model Usage`, dashboard version `v5`).

This document records the producer-specific telemetry contracts and calculations behind every dashboard signal. Use the exact source field paths below when building or changing dashboards; similarly named fields from different producers are not interchangeable.

## 1. Scope and normalized event contract

Thirteen of the sixteen panels first normalize producer events to this logical row:

| Normalized field | Meaning |
|---|---|
| `timestamp_nano` | Event time in nanoseconds. |
| `harness` | Producer/service identity. |
| `native_session_id` | Producer-native conversation or session identifier. |
| `provider` | Normalized model provider label. |
| `model` | Producer-native model identifier. |
| `input_tokens` | Total input under the dashboard's normalized input definition. |
| `cached_input` | Input served by a cache. |
| `output_tokens` | Generated output tokens. |
| `completed_operations` | Constant `1` per accepted usage event; this is not a cross-producer request ID. |

The other three panels—native operation outcomes, failure rate over time, and P95 duration—use producer-native operation boundaries rather than this usage row.

All queries apply the selected dashboard time range and `$service_name` filter. Log timestamps are already nanoseconds. Trace timestamps are converted with `toUInt64(toUnixTimestamp64Nano(timestamp))` where a nanosecond integer is needed.

## 2. Producer telemetry contracts

### 2.1 Codex (`codex-app-server`, `codex_exec`, `codex_cli_rs`)

#### Usage and cache signals

- Signal type: logs.
- Table: `signoz_logs.distributed_logs_v2`.
- Accepted event: `attributes_string['event.name'] = 'codex.sse_event'`.
- Harness: ``resource.`service.name`::String``.
- Provider: constant `'OpenAI'`.
- Session: `attributes_string['conversation.id']`; must be non-empty.
- Model: `attributes_string['model']`; must be non-empty.
- Input tokens:
  ```sql
  if(
    mapContains(attributes_number, 'input_token_count'),
    attributes_number['input_token_count'],
    toFloat64OrZero(attributes_string['input_token_count'])
  )
  ```
  At least one of `attributes_number['input_token_count']` or `attributes_string['input_token_count']` must exist.
- Cached input:
  ```sql
  if(
    mapContains(attributes_number, 'cached_token_count'),
    attributes_number['cached_token_count'],
    toFloat64OrZero(attributes_string['cached_token_count'])
  )
  ```
- Output tokens:
  ```sql
  if(
    mapContains(attributes_number, 'output_token_count'),
    attributes_number['output_token_count'],
    toFloat64OrZero(attributes_string['output_token_count'])
  )
  ```
- Completed operation: `toUInt64(1)` for each accepted `codex.sse_event`.

The string fallback uses `toFloat64OrZero`, so a missing or non-numeric string becomes zero.

#### Native success and failure

- Signal type: logs.
- Table: `signoz_logs.distributed_logs_v2`.
- Operation boundary: one accepted response attempt.
- Filters:
  - `attributes_string['event.name'] = 'codex.api_request'`
  - `attributes_string['endpoint'] = '/responses'`
  - `attributes_string['conversation.id'] != ''`
  - `attributes_string['model'] != ''`
- Success calculation:
  ```sql
  if(
    mapContains(attributes_bool, 'success'),
    attributes_bool['success'],
    NOT (
      mapContains(attributes_string, 'error.message')
      AND attributes_string['error.message'] != ''
    )
  )
  ```
- Failure is `NOT succeeded`.

When `attributes_bool['success']` is absent, the event is considered successful unless a non-empty `attributes_string['error.message']` exists.

#### Native duration

- Signal type: traces.
- Table: `signoz_traces.distributed_signoz_index_v3`.
- Operation boundary: span `name = 'run_sampling_request'`.
- Harness: `serviceName`, restricted to the three Codex services.
- Model guard: `attributes_string['model'] != ''`.
- Provider: constant `'OpenAI'`.
- Duration seconds: `duration_nano / 1000000000`.

This span measures a Codex sampling turn, including retry handling, response processing, and in-flight tool draining. It is not the same boundary as the `/responses` attempt used for success/failure.

### 2.2 Claude Code (`claude-code`)

#### Usage and cache signals

- Signal type: logs.
- Table: `signoz_logs.distributed_logs_v2`.
- Service filter: ``resource.`service.name`::String = 'claude-code'``.
- Accepted event: `attributes_string['event.name'] = 'api_request'`.
- Harness: constant `'claude-code'`.
- Provider: constant `'Anthropic'`.
- Session: `attributes_string['session.id']`; must be non-empty.
- Model: `attributes_string['model']`; must be non-empty.
- Direct input: `attributes_number['input_tokens']`.
- Cache creation input: `attributes_number['cache_creation_tokens']`.
- Cache-read input: `attributes_number['cache_read_tokens']`.
- Normalized input:
  ```sql
  attributes_number['input_tokens']
  + attributes_number['cache_creation_tokens']
  + attributes_number['cache_read_tokens']
  ```
- Cached input: `attributes_number['cache_read_tokens']`.
- Output tokens: `attributes_number['output_tokens']`.
- Completed operation: `toUInt64(1)` for each accepted `api_request` event.
- Therefore, uncached input is effectively `input_tokens + cache_creation_tokens` after aggregation.

#### Native success, failure, and duration

- Signal type: traces.
- Table: `signoz_traces.distributed_signoz_index_v3`.
- Service filter: `serviceName = 'claude-code'`.
- Operation boundary: terminal span `name = 'claude_code.llm_request'`.
- Provider: constant `'Anthropic'`.
- Success: `NOT (has_error OR status_code_string = 'Error')`.
- Failure: `has_error OR status_code_string = 'Error'`.
- Duration seconds: `duration_nano / 1000000000`.

### 2.3 Oh My Pi (`oh-my-pi`)

#### Usage and cache signals

- Signal type: traces.
- Table: `signoz_traces.distributed_signoz_index_v3`.
- Service filter: `serviceName = 'oh-my-pi'`.
- Accepted operation: `attributes_string['gen_ai.operation.name'] = 'chat'`.
- Harness: constant `'oh-my-pi'`.
- Session: `attributes_string['gen_ai.conversation.id']`; must be non-empty.
- Model: `attributes_string['gen_ai.request.model']`; must be non-empty.
- Provider source: `attributes_string['gen_ai.provider.name']`, normalized as:
  ```sql
  multiIf(
    lower(attributes_string['gen_ai.provider.name']) LIKE '%anthropic%', 'Anthropic',
    lower(attributes_string['gen_ai.provider.name']) LIKE '%openai%', 'OpenAI',
    attributes_string['gen_ai.provider.name']
  )
  ```
- Normalized input: `attributes_number['gen_ai.usage.input_tokens']`; this field must exist.
- Cached input:
  ```sql
  if(
    mapContains(attributes_number, 'gen_ai.usage.cache_read.input_tokens'),
    attributes_number['gen_ai.usage.cache_read.input_tokens'],
    0
  )
  ```
- Output tokens: `attributes_number['gen_ai.usage.output_tokens']`.
- Completed operation: `toUInt64(1)` for each accepted chat span.
- Activity guard: either `attributes_number['gen_ai.usage.input_tokens'] > 0` or `attributes_number['gen_ai.usage.output_tokens'] > 0`.

The dashboard treats `gen_ai.usage.input_tokens` as total normalized input. It has no separate Oh My Pi cache-creation field; only cache-read input is split out.

#### Native success, failure, and duration

- Signal type: traces.
- Table: `signoz_traces.distributed_signoz_index_v3`.
- Service filter: `serviceName = 'oh-my-pi'`.
- Operation boundary: `attributes_string['gen_ai.operation.name'] = 'chat'`.
- Provider: normalized from `attributes_string['gen_ai.provider.name']` using the mapping above.
- Success: `NOT (has_error OR status_code_string = 'Error')`.
- Failure: `has_error OR status_code_string = 'Error'`.
- Duration model guard: `attributes_string['gen_ai.request.model'] != ''`.
- Duration seconds: `duration_nano / 1000000000`.

## 3. Signal catalog

In the formulas below, `Σ` means aggregation over all accepted normalized usage events remaining after the time and service filters, unless a grouping is stated.

| # | Dashboard signal | Calculation | Producer fields / dimensions | Important behavior |
|---:|---|---|---|---|
| 1 | **Input Tokens** | `Σ input_tokens` | Codex `input_token_count`; Claude `input_tokens + cache_creation_tokens + cache_read_tokens`; OMP `gen_ai.usage.input_tokens` | Normalized total input, including cache-read input. |
| 2 | **Output Tokens** | `Σ output_tokens` | Codex `output_token_count`; Claude `output_tokens`; OMP `gen_ai.usage.output_tokens` | Total generated output. |
| 3 | **Cached Input** | `Σ cached_input` | Codex `cached_token_count`; Claude `cache_read_tokens`; OMP `gen_ai.usage.cache_read.input_tokens` | Counts cache-read input only. |
| 4 | **Uncached Input** | `Σ input_tokens - Σ cached_input` | Input and cached-input fields above | For Claude this includes direct input plus cache-creation input. |
| 5 | **Input Cache Utilization** | `100 × Σ cached_input / nullIf(Σ input_tokens, 0)` | Input and cached-input fields above | Token-weighted across selected producers; it is not an average of producer percentages. No input produces `NULL`, not zero. |
| 6 | **Usage-Active Sessions** | `uniqExact(tuple(harness, native_session_id))` | Codex `service.name` + `conversation.id`; Claude `'claude-code'` + `session.id`; OMP `'oh-my-pi'` + `gen_ai.conversation.id` | A session is counted only if it has an accepted usage event. Harness scoping prevents ID collisions. |
| 7 | **Daily Token Consumption** | Per day: `Σ input_tokens - Σ cached_input`, `Σ cached_input`, and `Σ output_tokens` | Usage token fields; day from `timestamp_nano` | Three mutually exclusive stacked series: uncached input, cached input, output. Query intentionally returns no rows when the selected range is shorter than `86400000000000` ns (24 hours). |
| 8 | **Daily Input Cache Utilization** | Per day: `100 × Σ cached_input / nullIf(Σ input_tokens, 0)` | Usage input and cached-input fields; day from `timestamp_nano` | Token-weighted within each day. Intentionally blank for ranges shorter than 24 hours. |
| 9 | **Cache Efficiency by Harness** | Group by `harness`; emit `Σ input_tokens`, `Σ cached_input`, `Σ completed_operations`, `Σ input_tokens - Σ cached_input`, and `round(100 × Σ cached_input / nullIf(Σ input_tokens, 0), 1)` | Normalized harness and usage fields | Sorted by input descending. Completed operations are accepted usage-event counts, not a common provider request boundary. |
| 10 | **Cache Efficiency by Model** | Same measures as #9, grouped by `model` | Producer model fields and usage fields | Identical model strings from different harnesses/providers are combined because provider and harness are not grouping keys. |
| 11 | **High-Volume Session Cache Analysis** | Same token/cache measures plus `Σ output_tokens`; group by `harness, native_session_id, provider, model` | All normalized usage fields | Sorted by input descending; limited to 20 groups. A session using multiple models yields multiple rows. |
| 12 | **Usage-Active Sessions Over Time** | Per time bucket: `uniqExact(tuple(harness, native_session_id))` | Usage timestamp, harness, and native session fields | Bucket size is `max(1, floor((range_seconds) / 120))` seconds. Measures activity, not starts or concurrency; one session may appear in multiple buckets. |
| 13 | **Native Operation Success / Failure** | Per `provider, harness, measurement`: `countIf(succeeded)`, `countIf(NOT succeeded)`, `round(100 × countIf(NOT succeeded) / count(), 1)` | Codex `success` / `error.message`; Claude and OMP `has_error` / `status_code_string`; producer-native filters from section 2 | Boundaries differ: Codex response attempt, Claude terminal LLM request, OMP terminal chat. Use as per-harness health trends, not comparable request totals. |
| 14 | **Native Operation Failure Rate Over Time** | Per harness and bucket: `100 × countIf(failed) / nullIf(count(), 0)` | Same outcome fields as #13 plus producer timestamps | Bucket size is `max(1, floor((range_seconds) / 120))` seconds. Series labels: `Codex App`, `Codex Exec`, `Codex CLI`, `Claude Code`, `OMP`. |
| 15 | **P95 Native Operation Duration** | Per harness and bucket: `quantileExact(0.95)(duration_nano / 1000000000)` | Trace `duration_nano`; Codex `name`; Claude `name`; OMP `gen_ai.operation.name`; model guards from section 2 | Uses the same 120-target bucket calculation. Native operation boundaries differ, so absolute duration is not a cross-harness benchmark. |
| 16 | **Session Details** | Same measures as #11; group by `harness, native_session_id, provider, model` | All normalized usage fields | Sorted by input descending; limited to 5,000 groups. |

## 4. Reusable calculation definitions

Use these definitions rather than reinterpreting producer fields in each panel:

```text
uncached_input = input_tokens - cached_input
cache_utilization_percent = 100 * sum(cached_input) / nullIf(sum(input_tokens), 0)
completed_operations = count(accepted producer-native usage events)
usage_active_sessions = exact distinct count of (harness, native_session_id)
native_failure_rate_percent = 100 * failed native operations / all native operations
native_p95_duration_seconds = exact 95th percentile of duration_nano / 1,000,000,000
bucket_seconds = max(1, floor(selected_range_seconds / 120))
```

Aggregate token numerators and denominators before division. Averaging per-event, per-session, or per-producer cache percentages produces a different signal from the dashboard.

## 5. Interpretation constraints

1. **Usage events are not a universal request boundary.** `completed_operations` is one per accepted producer usage event. Do not compare it as if every producer emits one event at the same lifecycle point.
2. **Reliability and duration use native boundaries.** Codex outcomes use `/responses` attempts while Codex duration uses sampling turns; Claude uses terminal LLM-request spans; OMP uses chat spans.
3. **Cache creation is not cache reuse.** Claude `cache_creation_tokens` contributes to normalized and uncached input, not cached input.
4. **Cache utilization is token-weighted.** High-volume producers, models, or sessions dominate combined percentages.
5. **Missing cache fields become zero for Codex string fallback and OMP cache read.** This can mean either no cache use or absent telemetry; the dashboard cannot distinguish those cases.
6. **Model aggregation can cross producers.** The model panel groups only by the raw normalized `model` string.
7. **Session identity is harness-scoped.** Never deduplicate on the native session field alone.
8. **Daily panels require a 24-hour selected range.** This is a query guard, not a data-availability indication.
