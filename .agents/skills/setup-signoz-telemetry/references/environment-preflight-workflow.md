# Environment preflight workflow

## Objective

Resolve portable deployment inputs and requested producer scope before changing anything.

## Required actions

1. Identify the container runtime, its version, the host architecture, and where the runtime stores data. Note whether that storage is removable or external.
2. Resolve each input from the user, the environment, or the running instance:
   - SigNoz release version and deployment directory
   - UI URL
   - OTLP gRPC and HTTP endpoints
   - backend database location
   - API authentication method
3. Detect any existing SigNoz instance before starting a new one. Report it instead of duplicating or replacing it.
4. Record requested producers, signals, protocols, and ports.
5. Reject undocumented machine-specific dependencies, such as hard-coded home paths, volume names, or host-specific helper scripts. Record any such dependency as an input to supply.

## Done when

- Deployment inputs, prerequisites, and producer scope are explicit.
- No required input depends on one user's filesystem or one instance's identifiers.
