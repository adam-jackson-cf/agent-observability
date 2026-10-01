# Producer configuration workflow

## Objective

Configure requested producers through documented OTEL settings while preserving unrelated state.

## Required actions

1. Back up the affected configuration outside any automatic import path.
2. Set the documented endpoint, protocol, service name, headers, and export controls. Use the resolved OTLP endpoint, never a hard-coded one.
3. Keep content export off by default: prompts, responses, tool arguments or output, and API bodies. Enable a content control only with explicit approval for that producer, and record which control was enabled.
4. Validate syntax without printing secrets.

## Done when

- The requested producer configuration is syntactically valid.
- Unrelated configuration and sensitive defaults are preserved.
