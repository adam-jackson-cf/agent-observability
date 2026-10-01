# develop-signoz-dashboard

## Overview

Develop repository-owned SigNoz dashboards from explicit producer signal contracts without losing calculation semantics or persisted identity.

## When to use it

- Creating a dashboard from documented producer telemetry.
- Adding or changing panels in an existing repository-owned dashboard.
- Validating dashboard JSON against its signal contract and the live backend.
- Applying and verifying a dashboard in a running SigNoz instance.

## Example prompts

- Create a SigNoz dashboard from the model-usage signal contract.
- Add a producer breakdown without changing cache-utilization semantics.
- Validate and apply the repository dashboard to SigNoz.

## Repository conventions

- Dashboard documents are SigNoz dashboard JSON files at the repository root, such as `caching-all-model-usage.json`.
- Each dashboard has a signal reference (`*-signal-reference.md`) whose first line, `Source of truth: <dashboard>.json`, links the two.
- A signal reference documents logical rows, producer contracts, variables, the signal catalog, and interpretation constraints.
- Instance-specific values, such as SigNoz URLs, dashboard route IDs, credentials, and container names, are supplied when the skill runs and never committed.
