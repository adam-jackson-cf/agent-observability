# Runtime verification workflow

## Objective

Verify changed panels on the persisted dashboard through SigNoz itself.

## Required actions

1. Execute every changed panel's persisted query through the SigNoz query API.
   - Supply explicit values for every dashboard variable.
   - Use the intended ranges, including one below any minimum-range guard.
2. Confirm each panel returns correct values or an honest empty state.
3. Change at least one variable and confirm the affected results change accordingly.
4. When a browser is available, check:
   - table readability and graph fields
   - units and legends
   - variable controls
   Report these checks separately. If they could not be done, say so.
5. Record unresolved telemetry gaps without inventing values.

## Done when

- Every changed panel has observable verification evidence through SigNoz.
- Any unavailable signal, or any skipped visual check, is reported explicitly.
