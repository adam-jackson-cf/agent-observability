# Stack bootstrap workflow

## Objective

Start or confirm a version-pinned SigNoz deployment with safe defaults.

## Required actions

1. Use the official SigNoz deployment files for the resolved release. Never upgrade the version or schema as a side effect of setup.
2. Check that the deployment files and required environment variables are present. Check presence only; never print values.
3. Use configurable storage and endpoint values from the resolved inputs.
4. Bind UI and OTLP ports to loopback unless the user explicitly authorizes broader exposure.
5. Verify the effective runtime configuration after startup:
   - health endpoint
   - service and container health
   - published ports
   - authentication mode

## Done when

- SigNoz services are healthy at the pinned version.
- Effective listeners and security posture match the configuration.
