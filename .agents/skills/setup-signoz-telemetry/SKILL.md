---
name: "setup-signoz-telemetry"
description: "Use when setting up, operating, or diagnosing a vanilla local SigNoz instance, or configuring requested producers to send OTLP telemetry to it."
---

# Workflow

### Step 1: Discover environment and scope

- **Purpose**: Resolve deployment inputs, host capabilities, and explicitly requested producers without machine assumptions.
- Resolve every deployment input from the user, the environment, or the running instance. Never assume a default path, host, or container name.
- Record requested producers, telemetry signals, endpoints, and security constraints.
- Workflow: [Environment preflight workflow](references/environment-preflight-workflow.md)

### Step 2: Bootstrap SigNoz safely

- **Purpose**: Start or confirm a version-pinned SigNoz deployment with safe local defaults.
- Use the official SigNoz deployment at the pinned release; never upgrade implicitly.
- Bind UI and OTLP listeners to loopback by default, and check secret presence without displaying values.
- Workflow: [Stack bootstrap workflow](references/stack-bootstrap-workflow.md)

### Step 3: Set retention deliberately

- **Purpose**: Make each signal's retention an explicit, verified choice rather than an inherited default.
- Set logs, traces, and metrics retention through SigNoz's supported settings.
- Verify that existing data, not only new data, carries the new expiry.
- Workflow: [Retention workflow](references/retention-workflow.md)

### Step 4: Prove independent ingestion

- **Purpose**: Verify SigNoz and its collector before changing any producer.
- Check UI health, container health, collector pipelines, and OTLP listeners.
- Send bounded synthetic logs, metrics, and traces, and verify newly ingested records.
- Workflow: [Synthetic ingestion workflow](references/synthetic-ingestion-workflow.md)

### Step 5: Discover producer capabilities

- **Purpose**: Identify documented, producer-native OTEL configuration surfaces and emitted schemas.
- Inspect only requested producers and their documented configuration surfaces.
- Leave unsupported capabilities unresolved rather than inferring them.
- Workflow: [Producer discovery workflow](references/producer-discovery-workflow.md)

### Step 6: Configure requested producers

- **Purpose**: Route producer telemetry through documented OTLP settings without exporting unintended sensitive content.
- Change only the requested producers' configuration, and preserve unrelated settings.
- Get explicit approval before enabling export of prompts, responses, tool arguments or output, or API bodies.
- Workflow: [Producer configuration workflow](references/producer-configuration-workflow.md)

### Step 7: Verify end-to-end telemetry

- **Purpose**: Prove one fresh producer event reaches SigNoz with the expected service and schema.
- Start a fresh producer process, generate one bounded operation, and compare exact expected fields.
- Workflow: [End-to-end validation workflow](references/end-to-end-validation-workflow.md)

### Step 8: Operate and stop safely

- **Purpose**: Keep recently written telemetry durable across stops, restarts, and host sleep.
- Quiesce the database before stopping the stack or its container runtime.
- Keep the host awake during long ingestion or maintenance work.
- Workflow: [Safe operation workflow](references/safe-operation-workflow.md)

### Step 9: Diagnose failed boundaries

- **Purpose**: Locate the first failed boundary without bypassing security or inventing support.
- **When**: When bootstrap, retention, ingestion, producer configuration, or verification fails.
- Diagnose in dependency order, and report the first failed boundary.
- Do not disable authentication or broaden network exposure as a shortcut.
- Workflow: [Troubleshooting workflow](references/troubleshooting-workflow.md)

## Output

### Result Format

- Report the selected setup or producer operation, and the resolved deployment inputs. Mask secrets.
- Report SigNoz, collector, listener, retention, and synthetic-ingestion status.
- List producer capability classifications and the configuration files changed.
- Provide fresh end-to-end telemetry evidence and unresolved risks.
