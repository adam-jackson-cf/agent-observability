# setup-signoz-telemetry

## Overview

Set up and operate a vanilla, version-pinned local SigNoz instance, and connect explicitly requested OTEL producers using documented, privacy-safe configuration.

## When to use it

- Bootstrapping SigNoz after cloning the repository.
- Setting or checking telemetry retention.
- Checking collector and OTLP ingestion health.
- Configuring a supported producer to export telemetry.
- Stopping, restarting, or relocating the stack without losing recent data.
- Diagnosing missing producer telemetry.

## Example prompts

- Set up a local SigNoz instance for this repository.
- Set telemetry retention to 90 days and verify existing data keeps it.
- Configure Claude Code to send OTEL telemetry to this SigNoz instance.
- Verify logs, metrics, traces, and the requested producer end to end.

## Inputs

- SigNoz release version, and the deployment directory holding the official deployment files.
- Container runtime and its data location.
- UI URL, OTLP gRPC and HTTP endpoints, and the backend database container or host.
- The authentication method for the SigNoz API.
- The requested producers and signals.

Supply these when the skill runs. None are stored in the repository. Producer signal semantics live in the dashboards' `*-signal-reference.md` files.
