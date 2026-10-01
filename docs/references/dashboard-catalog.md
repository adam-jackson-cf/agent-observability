# Dashboard Catalog

Use this catalog to pick the dashboard that answers your question, and to see which telemetry each producer supplies to it. For field paths and formulas, go to the signal reference named in each row.

## Dashboards

| Dashboard | File | Question it answers | Panels | Signal reference |
| --- | --- | --- | --- | --- |
| All Model Usage | `caching-all-model-usage.json` | How many tokens are agents using, how much input comes from cache, how many sessions are active, and are native operations failing or slowing down? | 16 | [`model-usage-signal-reference.md`](../../model-usage-signal-reference.md) |
| Reasoning Effort Effectiveness | `reasoning-effort-effectiveness.json` | Does higher reasoning effort pay for its extra cost, model steps, and time with cleaner task completion? | 21 | [`reasoning-effort-signal-reference.md`](../../reasoning-effort-signal-reference.md) |

Both dashboards filter by `$service_name` and the selected time range. Their daily panels stay blank when the selected range is shorter than 24 hours. That blank is a query guard, not missing data.

## Producers

| Producer (`service.name`) | Usage and cost source | Native outcome and duration source | Reasoning tokens | Interrupt signal | Evidence source |
| --- | --- | --- | --- | --- | --- |
| `codex-app-server`, `codex_exec`, `codex_cli_rs` | Logs: `codex.sse_event` | Outcome: logs, `codex.api_request` on `/responses`. Duration: `run_sampling_request` spans | Yes | Yes (`turn/interrupt` span) | `model-usage-signal-reference.md` §2.1, `reasoning-effort-signal-reference.md` §2.1 |
| `claude-code` | Logs: `api_request` | Traces: `claude_code.llm_request` spans | No (included in output tokens) | No | `model-usage-signal-reference.md` §2.2, `reasoning-effort-signal-reference.md` §2.3 |
| `oh-my-pi` | Traces: `chat` spans | Traces: `chat` spans | Yes, except Anthropic-provider chats | Yes (`stop_reason.aborted`) | `model-usage-signal-reference.md` §2.3, `reasoning-effort-signal-reference.md` §2.2 |

## Reasoning Effort Variables

The Reasoning Effort Effectiveness dashboard has text-box variables that change its results:

| Variable | Default | What it controls |
| --- | --- | --- |
| `price_input` | 1 | Weight per uncached input token |
| `price_cached_input` | 0.1 | Weight per cached input token |
| `price_output` | 8 | Weight per output token, including reasoning |
| `min_tasks` | 30 | Rows with fewer tasks are flagged Low sample and excluded from baselines and the effort ladder |
| `success_tolerance_pts` | 5 | How many percentage points of clean completion the effort ladder may give up for a lower effort |

The price defaults are relative weights. Enter real per-token prices to read cost in money. Source: `reasoning-effort-signal-reference.md` §3.

## Notes

- Use these dashboards to compare each producer with its own history. Producers define operations, sessions, and tasks at different points in their lifecycle, so cross-producer totals and rates are not like-for-like.
- `codex_exec` runs are headless, so quick follow-up and clean completion are `NULL` for that service.
- Claude Code has no interrupt event, so its clean-completion rate is an upper bound.
- Effort is chosen by the user, so differences between effort levels show correlation, not cause. Compare within one harness and model.
