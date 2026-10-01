# Signal contract workflow

## Objective

Resolve exact producer fields and calculations, and record them in the signal reference, before panel design.

## Required actions

1. For each producer, record:
   - source table and signal type (logs, traces, or metrics)
   - acceptance filters and identifiers
   - field paths, and fallbacks such as numeric-or-string attribute pairs
   - calculations
2. Record aggregation order, grouping keys, time bucketing, null-versus-zero behavior, and the unit of analysis, such as a model call, task, or session.
3. Keep producer-native boundaries distinct from normalized rows. Never treat similarly named fields from different producers as interchangeable.
4. Confirm each field's presence and coverage in live data with read-only, time-bounded queries before relying on it. Record fields a producer does not emit as not reported, never as zero.
5. Update the signal reference in the same change as the dashboard:
   - logical rows
   - producer contracts
   - variables
   - signal catalog
   - interpretation constraints

## Done when

- Every requested signal is traceable to exact source fields confirmed in live data.
- Every derived calculation is deterministic and documented in the signal reference.
