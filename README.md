<div align="center">

<h1>Agent Observability</h1>

**SigNoz dashboards that show how coding agents (Codex, Claude Code, and Oh My Pi) use models, caches, and reasoning effort, built from each agent's own OpenTelemetry data.**

![SigNoz](https://img.shields.io/badge/signoz-v0.128.0-blue.svg?style=flat-square)
![Runtime](https://img.shields.io/badge/runtime-docker%20compose-lightgrey.svg?style=flat-square)
![Producers](https://img.shields.io/badge/producers-codex%20%7C%20claude--code%20%7C%20oh--my--pi-orange.svg?style=flat-square)

</div>

## Quick Install

Prerequisites:

- Docker with `docker compose`
- `git`
- Values for the SigNoz root user (`SIGNOZ_USER_ROOT_ENABLED`, `SIGNOZ_USER_ROOT_EMAIL`, `SIGNOZ_USER_ROOT_PASSWORD`, `SIGNOZ_USER_ROOT_ORG_NAME`), supplied from a secret store or the shell

Run a vanilla SigNoz at the pinned release. `<signoz-dir>` is any directory you choose.

```bash
set -a; source ops/signoz/versions.env; set +a
git clone --branch "$SIGNOZ_DEPLOY_REF" https://github.com/SigNoz/signoz <signoz-dir>
cp ops/signoz/docker-compose.override.example.yaml <signoz-dir>/deploy/docker/docker-compose.override.yaml
# Apply the three local adjustments listed in ops/signoz/README.md, then:
cd <signoz-dir>/deploy/docker && docker compose up -d
```

The override binds the UI (`127.0.0.1:8080`) and OTLP listeners (`127.0.0.1:4317` gRPC, `127.0.0.1:4318` HTTP) to loopback only. The full steps and the reason for each adjustment are in [`ops/signoz/README.md`](ops/signoz/README.md).

## Quick Start

The repository has no build or CLI. Its setup and dashboard workflows are agent skills under `.agents/skills/`. Ask your coding agent, from the repository root:

```text
Set up a local SigNoz instance for this repository.
Configure Claude Code to send OTEL telemetry to this SigNoz instance.
Validate and apply the repository dashboard to SigNoz.
```

The first prompt runs `setup-signoz-telemetry`. It sets retention, sends synthetic logs, metrics, and traces to prove ingestion works, then configures the producers you ask for, with prompt and response export left off. The last prompt runs `develop-signoz-dashboard`. It checks every panel query against the backend, then creates the dashboard in SigNoz or updates it in place, matched by the document's `uuid`.

## What Agent Observability Does

Each coding agent reports its model calls differently, so the same field names do not always mean the same thing. This repository documents what each agent actually emits, normalizes where the numbers are compatible, and puts the result on two SigNoz dashboards. The dashboards are for spotting operational problems within one agent over time, not for ranking agents against each other.

- `caching-all-model-usage.json`: **All Model Usage**. Token use, input-cache utilization, sessions, native success and failure, and P95 duration (16 panels).
- `reasoning-effort-effectiveness.json`: **Reasoning Effort Effectiveness**. Whether higher reasoning effort pays for its extra cost, steps, and time (21 panels).
- `*-signal-reference.md`: the field-level contract and formula behind every panel of the matching dashboard.
- `ops/signoz/`: the SigNoz version pin and the deployment adjustments the dashboards were validated against.
- `audit.md`: dashboard defects and data incidents, with evidence, fix, and status.

## Core Concepts

- **Producer / harness**: an agent that sends telemetry, identified by its OTel `service.name`: `codex-app-server`, `codex_exec`, `codex_cli_rs`, `claude-code`, `oh-my-pi`.
- **Signal reference**: the authoritative contract for one dashboard. Its first line, `Source of truth: <dashboard>.json`, links it to that dashboard. Change semantics here before changing panels.
- **Usage event**: one accepted, token-bearing model call, normalized to harness, session, provider, model, input, cached input, and output.
- **Native operation boundary**: the agent's own unit for success, failure, and duration (a Codex `/responses` attempt, a Claude Code `claude_code.llm_request` span, an Oh My Pi `chat` span). These units differ, so absolute values are not comparable across agents.
- **Task**: one user-started agent run (a Codex turn, an Oh My Pi root `invoke_agent` trace, a Claude Code `prompt.id`). The reasoning-effort dashboard uses it as its unit of cost and outcome.
- **Clean completion**: a task that was not interrupted, did not error, and had no follow-up task within 10 minutes. It measures friction, not success.

## Go Deeper

- [`docs/references/dashboard-catalog.md`](docs/references/dashboard-catalog.md): which dashboard answers which question, and which signals each producer supplies.
- [`docs/references/dashboard-change-workflow.md`](docs/references/dashboard-change-workflow.md): how to change a dashboard without breaking its contract, its identity, or audited fixes.
- [`docs/references/signoz-operations-workflow.md`](docs/references/signoz-operations-workflow.md): how to set retention, stop and restart without losing data, and upgrade the pinned SigNoz version.
- [`model-usage-signal-reference.md`](model-usage-signal-reference.md) and [`reasoning-effort-signal-reference.md`](reasoning-effort-signal-reference.md): exact fields, formulas, and interpretation limits for each panel.
