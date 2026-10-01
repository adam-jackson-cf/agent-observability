# Panel design workflow

## Objective

Specify each panel from approved signal semantics before JSON generation.

## Required actions

1. For each panel, define:
   - signal and formula
   - grouping, filters, and time behavior
   - visualization, unit, and legend
   - empty state
2. Aggregate numerators and denominators before dividing. Use weighted ratios only where the contract defines weighted aggregation.
3. Keep incompatible producer-native boundaries in separate series or rows.
4. Apply the dashboard time range and producer filter variables in every query.
5. Specify every dashboard variable's name, type, and default, and how queries read it:
   - Dynamic multi-select variables substitute as a value list, used with `IN $name`.
   - Text-box variables substitute as quoted strings, read with `toFloat64OrZero($name)` or `toUInt64OrZero($name)`.
6. Make the data shape match the panel type:
   - Time series return `timestamp`, `series`, and `value`.
   - Value panels return one row.
   - Tables return named columns.
   - Graphs whose buckets need a minimum range, such as daily charts, return no rows below it rather than misleading partial points.
7. For rates and comparisons, show sample size beside the result, and flag or suppress groups below a declared minimum.

## Done when

- Every panel has a complete, deterministic specification.
- No panel invents an undocumented signal.
