# Synthetic ingestion workflow

## Objective

Prove logs, metrics, and traces can pass through the collector independently of producers.

## Required actions

1. Record a verification start timestamp.
2. Send bounded synthetic OTLP logs, metrics, and traces, using a version-pinned OTLP generator.
   - Address the collector as it is reachable from the generator's network: the host loopback, or the runtime's host gateway name from inside a container.
3. Query backend records newer than the start timestamp for each signal.
4. Stop before producer debugging if any required signal fails.

## Done when

- Every required synthetic signal is newly observed.
- The first failed ingestion boundary, if any, is known.
