# Static validation workflow

## Objective

Check the dashboard JSON's structure, identity, and portability, and its agreement with the signal reference, without contacting any backend.

## Required actions

1. Parse the JSON and check:
   - Widget IDs are unique.
   - Every widget has exactly one layout entry.
   - Query IDs are unique.
   - The layout fits the grid with no overlaps.
2. Check that every query applies the time-range and producer-filter variables, and that defined and referenced variables match exactly.
3. Compare query fields, filters, formulas, dimensions, and units with the signal reference.
4. On update, confirm the document `uuid` and unchanged panel IDs match the previous committed version.
5. Scan for credentials, absolute local paths, instance URLs, and route IDs.

## Done when

- Structure, identity, and contract checks pass.
- No portability-sensitive value remains in the document.
