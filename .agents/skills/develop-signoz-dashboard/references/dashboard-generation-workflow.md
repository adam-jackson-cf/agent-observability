# Dashboard generation workflow

## Objective

Generate portable SigNoz dashboard JSON that preserves approved identity and semantics.

## Required actions

1. Write the dashboard document to its root-level JSON file.
2. On update:
   - Keep the document `uuid`.
   - Keep the IDs, layout entries, and query IDs of unchanged panels.
   - Give new or replaced panels new IDs.
   - List every retired panel in the change summary.
3. Keep every layout entry inside the 12-column grid and free of overlaps, with one layout entry per widget.
4. Define every variable that a query references, and reference every defined variable.
5. Exclude instance-specific values: SigNoz URLs, dashboard route IDs, container names, absolute local paths, and credentials. Context links use relative SigNoz routes.
6. Keep generation tooling outside the repository unless the repository already adopts it. The committed JSON is the maintained artifact.

## Done when

- The JSON represents the approved panel specification.
- Identity rules for create and update are satisfied.
