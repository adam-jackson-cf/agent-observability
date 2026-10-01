# Runtime application workflow

## Objective

Apply a validated dashboard to a running SigNoz instance without losing persisted identity or unrelated content.

## Required actions

1. Resolve, at apply time, the target instance URL and the authentication it requires. Never disable or bypass authentication, and never commit these values.
2. Confirm instance health and that its dashboard API supports listing, reading, creating, and updating dashboards (`/api/v1/dashboards` and `/api/v1/dashboards/{id}`).
3. List dashboards and find the target by its nested document `uuid`. The route ID is instance-specific; resolve it at apply time.
4. If no dashboard has that `uuid`, create it once.
5. If one does:
   - Back up the full persisted response outside the repository.
   - Update that route in place with the raw document, not the response wrapper.
   - Never create or import a copy instead of updating.
6. Re-read the dashboard and confirm:
   - The persisted document equals the file.
   - The route ID and `uuid` are unchanged.
   - The total dashboard count is unchanged.
7. If the dashboard was locked, unlock it only for the update and lock it again afterwards.

## Done when

- The intended dashboard exists exactly once, with the expected identity.
- Unrelated dashboards and content are unchanged.
