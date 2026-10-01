# Producer discovery workflow

## Objective

Classify requested producers by documented OTEL capability and emitted schema.

## Required actions

1. Inspect installed versions and documented configuration surfaces.
2. Record supported signal types, endpoint protocols, service identity, schema fields, and restart behavior.
3. Classify unsupported or ambiguous capabilities as unresolved.

## Done when

- Every requested producer is supported or unresolved with evidence.
- No capability is inferred from unrelated producers.
