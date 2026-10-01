---
name: "develop-signoz-dashboard"
description: "Use when designing, generating, updating, validating, or applying a repository-owned SigNoz dashboard from its documented signal contract."
---

# Workflow

### Step 1: Validate dashboard scope

- **Purpose**: Bind the request to one repository-owned dashboard and its signal contract.
- **When**: Before query or panel design.
- Decide whether the request creates or updates a dashboard.
- Locate the dashboard JSON and the signal reference whose `Source of truth` line names it.
- Stop when a requested signal has no authoritative contract.
- Workflow: [Scope workflow](references/scope-workflow.md)

### Step 2: Resolve signal semantics

- **Purpose**: Fix the producer fields, boundaries, formulas, dimensions, time behavior, and empty-state meaning.
- Map every requested signal to exact producer source fields, and confirm they exist in live data.
- Record new or changed semantics in the signal reference before designing panels.
- Workflow: [Signal contract workflow](references/signal-contract-workflow.md)

### Step 3: Design panels and queries

- **Purpose**: Turn approved signal definitions into complete panel and query specifications.
- Specify visualization, formula, grouping, filters, variables, unit, legend, and no-data behavior.
- Reject cross-producer comparisons whose boundaries are incompatible.
- Workflow: [Panel design workflow](references/panel-design-workflow.md)

### Step 4: Generate dashboard JSON

- **Purpose**: Create or update the repository-owned dashboard document without changing approved semantics.
- Keep unchanged panels' IDs stable, and give replaced panels new ones.
- Exclude instance-specific identifiers, endpoints, paths, and credentials.
- Workflow: [Dashboard generation workflow](references/dashboard-generation-workflow.md)

### Step 5: Validate the artifact

- **Purpose**: Prove structural validity, agreement with the contract, and working queries before any change to a running SigNoz instance.
- Check structure, identities, layout, variables, and portability.
- Run every panel query against the backend through bounded, read-only execution.
- Workflow: [Static validation workflow](references/static-validation-workflow.md)
- Workflow: [Query validation workflow](references/query-validation-workflow.md)

### Step 6: Apply safely to SigNoz

- **Purpose**: Create the dashboard, or update it while preserving its identity, through the instance's supported dashboard API.
- **When**: Only when applying it to a running instance is explicitly requested.
- Resolve the target instance and its authentication at apply time; never store them in the repository.
- Back up the persisted dashboard before an update.
- Workflow: [Runtime application workflow](references/runtime-application-workflow.md)

### Step 7: Verify dashboard behavior

- **Purpose**: Confirm the persisted dashboard returns the intended signals and honest empty states.
- Execute every changed panel through SigNoz with explicit values for every variable.
- Report visual checks separately from query checks.
- Workflow: [Runtime verification workflow](references/runtime-verification-workflow.md)

## Output

### Result Format

- Report the target dashboard and its signal reference.
- List generated or updated repository files, including the signal reference.
- Report static validation, query validation, runtime application, and runtime verification status separately.
- Provide panel-level verification evidence and unresolved telemetry gaps.
