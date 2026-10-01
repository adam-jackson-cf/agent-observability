# Reasoning effort signal reference

Source of truth: `reasoning-effort-effectiveness.json` (`Reasoning Effort Effectiveness`, dashboard version `v5`).

This dashboard tests whether heavier reasoning (also called thinking) earns its cost. It
covers Codex, Oh My Pi (OMP) and Claude Code, and uses only data a vanilla SigNoz install
holds. It sets price-weighted cost, model steps, and time against friction-based outcomes for
each requested effort level, and it records producer-specific contracts in the same way as
`model-usage-signal-reference.md`. Similarly named fields from different producers are not
interchangeable.

## 1. Logical rows

### 1.1 `reasoning_usage`: one row per model call (reasoning spend)

Codex `codex.sse_event` token-bearing rows and OMP `chat` spans: harness, provider, model,
effort (`unset` when none was sent), output tokens, reasoning tokens, and one model call.
Claude Code is not included, because it reports no reasoning tokens.

### 1.2 `task_calls`: one row per model call inside a task (cost)

| Field | Meaning |
|---|---|
| `harness`, `task_id` | Owning task. |
| `agent` | OMP parent span (main agent or subagent); empty for Codex and Claude Code. |
| `ts` | Call time in nanoseconds. |
| `inp`, `cin` | Total input and cache-read input. |
| `outp`, `rsn` | Output, and reasoning (`NULL` when the producer does not report it). |
| `dur_s` | Call duration: OMP chat spans and Claude Code `api_request`. Codex model time comes from turn sampling spans instead (section 2.1). |
| `agent_first_input` | Input on the agent's first call in the task: the context it inherited. |
| `input_price` | Blended price per input token: `((inp - cin) × price_input + cin × price_cached_input) / inp`. |

### 1.3 `task_outcomes`: one row per user-initiated task

| Field | Meaning |
|---|---|
| `start_nano`, `end_nano`, `duration_seconds` | Task wall-clock span. |
| `harness`, `native_session_id`, `task_id`, `model`, `effort`, `effort_rank` | Identity. `effort_rank` runs low = 1 to xhigh = 4; unset and mixed are 0. |
| `model_steps` | Token-bearing model calls in the task. |
| `input_tokens`, `cached_input`, `output_tokens`, `reasoning_tokens` | Per-call sums. Reasoning is `NULL` where unreported. |
| `model_seconds` | Summed model-call time. |
| `cost_inherited` | Σ `min(inp, agent_first_input) × input_price`: re-reading context the agent already held. |
| `cost_growth` | Σ `(inp − min(inp, agent_first_input)) × input_price`: context added during the task. |
| `cost_visible_output` | Σ `(outp − rsn) × price_output`. |
| `cost_reasoning` | Σ `rsn × price_output`. |
| `cost_units` | The sum of the four cost parts. |
| `inherited_context`, `growth_per_step` | The main agent's first-call input, and its input growth per extra call. |
| `delegations` | OMP subagent runs, excluding advisors. 0 elsewhere. |
| `interrupted`, `errored` | 0, 1, or `NULL` when the producer has no such signal. |
| `quick_follow_up` | Another task in the same harness session starts no later than 10 minutes after this one ends. `NULL` for `codex_exec`. |
| `clean_completion` | Not interrupted, not errored, and no quick follow-up. `NULL` for `codex_exec`. |

All queries apply the dashboard time range and `$service_name`.

## 2. Producer contracts

### 2.1 Codex (`codex-app-server`, `codex_exec`, `codex_cli_rs`)

- **Task:** span `session_task.turn` with non-empty `turn.id`, `thread.id` and `model`,
  excluding `model = 'codex-auto-review'`. Effort is `codex.turn.reasoning_effort`.
- **Calls:** `codex.sse_event` rows that carry `input_token_count`, in the same
  `conversation.id`, between the turn's start and end.
  - Fields: `input_token_count`, `cached_token_count`, `output_token_count` and
    `reasoning_token_count`, each read from `attributes_number` or else
    `toFloat64OrZero(attributes_string[...])`.
- **Model time:** Σ `run_sampling_request` span durations sharing the turn's `turn_id`. These
  include retry handling and in-flight tool draining. The token-bearing `response.completed`
  events carry no `duration_ms`.
- **Interrupted:** the turn's `turn.id` appears on a `turn/interrupt` span.
- **Errored:** `NULL`; turns emit no terminal error signal.
- **`codex_exec`:** headless one-shot runs with no user to follow up. Quick follow-up and clean
  completion are `NULL`.

### 2.2 Oh My Pi (`oh-my-pi`)

- **Task:** a root `invoke_agent` span (`parentSpanID = ''`). The task is its trace, including
  subagents.
- **Effort:** the effort of the chats directly under the root span: one value; `unset` when
  there are none; `mixed` when there are several.
- **Calls:** `chat` spans in the trace.
  - Input is `gen_ai.usage.input_tokens`; cached is `gen_ai.usage.cache_read.input_tokens`;
    output is `gen_ai.usage.output_tokens`.
  - Reasoning is `gen_ai.usage.reasoning.output_tokens`, or 0 when absent. It is `NULL` for
    Anthropic-provider chats, which report none.
  - Duration is the span duration.
  - Agent is the chat's `parentSpanID`, so inherited context is measured per agent.
- **Interrupted:** `pi.gen_ai.agent.chats.stop_reason.aborted.count > 0`.
- **Errored:** `pi.gen_ai.agent.chats.stop_reason.error.count > 0`.
- **Delegations:** count of spans named `invoke_agent <agent>`, excluding
  `invoke_agent Advisor:%`.

### 2.3 Claude Code (`claude-code`)

- **Task:** `api_request` log events sharing `prompt.id` (one user prompt), with non-empty
  `session.id`.
  - Start is the earliest call time minus its duration; end is the latest call time.
  - Model is the model with the most output tokens.
  - Effort is the most common `effort`.
- **Calls:** each `api_request`.
  - Input is `input_tokens + cache_creation_tokens + cache_read_tokens`; cached is
    `cache_read_tokens`; output is `output_tokens`.
  - Duration is `duration_ms / 1000`.
- **Reasoning:** `NULL`. Thinking is reported only inside output tokens.
- **Errored:** an `api_error` or `api_retries_exhausted` event shares the `prompt.id`.
- **Interrupted:** `NULL`; Claude Code emits no interrupt event. Clean completion is therefore
  an upper bound.

## 3. Dashboard variables

| Variable | Type | Default | Meaning |
|---|---|---|---|
| `service_name` | Dynamic, multi-select | All five services | Producer filter. |
| `price_input` | Text box | 1 | Weight per uncached input token. |
| `price_cached_input` | Text box | 0.1 | Weight per cached input token. |
| `price_output` | Text box | 8 | Weight per output token, including reasoning. |
| `min_tasks` | Text box | 30 | Below this, a row is flagged Low sample and excluded from baselines and the ladder. |
| `success_tolerance_pts` | Text box | 5 | Effort ladder tolerance, in percentage points. |

The price defaults are relative weights, not money. Enter real per-token or per-million
prices to read cost in money. SigNoz substitutes text-box values as quoted strings, so queries
read them with `toFloat64OrZero` or `toUInt64OrZero`.

## 4. Signal catalog

| # | Signal | Calculation | Notes |
|---:|---|---|---|
| 1 | Median Cost per Task | `quantileExact(0.5)(cost_units)` | |
| 2 | Reasoning Share of Cost | `100 × Σ cost_reasoning / Σ cost_units` | Tasks that report reasoning only. Replaces the earlier "share of output". |
| 3 | Tasks | `count()` | All three producers. |
| 4 | Heavy-Effort Task Share | `100 × countIf(effort IN ('high','xhigh')) / count()` | |
| 5 | Interrupted Tasks | `100 × avg(interrupted)` | Claude Code excluded (`NULL`). |
| 6 | Clean Completion Rate | `100 × avg(clean_completion)` | |
| 7 | Daily Reasoning Tokens by Effort | Per day and effort: Σ reasoning (usage rows) | Codex and OMP. |
| 8 | Daily Task Mix by Effort | Per day and effort: `count()` | |
| 9 | Task Outcomes by Reasoning Effort | Per harness and effort | Adds low-sample flag, P50 model time, model time share and median cost. |
| 10 | Effort Cost Multiplier | Per harness, model and effort: median cost and steps ÷ the baseline's | Baseline is the lowest ranked effort with `n ≥ min_tasks`. |
| 11 | Cost Composition by Effort | `100 × Σ` each cost part ÷ `Σ cost_units` | Plus median inherited context and growth per step. |
| 12 | Effort, Steps and Cost Correlation | Spearman correlation within harness and model | Ranks use tie-averaging and `corr`. `NULL` when below `min_tasks`, a single effort level, or constant values. |
| 13 | Success Chance by Effort | Clean-completion rate with a 95% Wilson interval | |
| 14 | Effort Ladder | Best effort, and the lowest effort within `success_tolerance_pts` of it | Both need `n ≥ min_tasks`; adds cost saving. |
| 15 | Goal Chains by First-Task Effort | Chains of tasks in the same session, each no more than 10 minutes after the previous ends | Chain cost multiplier = Σ chain cost ÷ Σ first-task cost. `codex_exec` excluded. |
| 16 | Daily Clean Completion Rate by Effort | Per day and effort | The legend shows `effort (n=…)` over the selected range. |
| 17 | Daily Median Task Duration by Effort | Per day and effort: P50 duration | |
| 18 | Daily Model Steps per Task by Effort | Per day and effort: average steps | |
| 19 | Reasoning Spend by Effort | Usage rows per harness and effort | |
| 20 | Model × Effort Comparison | Per harness, model and effort | Adds low-sample flag and median cost. |
| 21 | Heaviest Reasoning Tasks | Top 50 tasks by `cost_units` | Links to session traces. |

Daily panels use `$end_timestamp_nano-$start_timestamp_nano >= 86400000000000`.

## 5. Interpretation constraints

1. **Reasoning's cost is mostly indirect.**
   - Input is about 98–99.8% of tokens, and reasoning tokens are at most 0.5%.
   - Price-weighted, reasoning is about 3–11% of task cost. Its main lever is more model
     steps, each of which re-reads the context; read the step multiplier beside the cost
     multiplier.
2. **Inherited context often outweighs effort.** A long session makes every step expensive
   whatever the effort.
3. **Effort is user-chosen**, so differences are correlational. Compare within one harness and
   model, and check the confidence intervals.
4. **Clean completion is a friction proxy.** A quick follow-up may be the next planned step
   rather than a correction. Claude Code clean completion is an upper bound, because it has no
   interrupt signal.
5. **`unset` is not reasoning disabled**; it is the provider default. There is no
   reasoning-off population.
6. **Model time can exceed wall-clock time** when OMP or Claude Code subagents call models in
   parallel.
7. **OMP inherited context is per agent.** Subagents start with their own context.
8. **Plan adherence, over-engineering, sentiment and verification quality** are not measurable
   from vanilla SigNoz data for all three producers. They need capabilities beyond OTEL
   data, such as plan capture, prompt judgement and verification parsing.
