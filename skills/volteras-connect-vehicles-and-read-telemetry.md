---
name: Connect vehicles and read telemetry with Volteras
description: >-
  Onboard vehicles onto the Volteras Connect platform (consent flow or bulk
  VIN) and read live telemetry, range, and battery state of health.
api: openapi/volteras-connect-openapi-original.json
generated: '2026-07-21'
method: generated
operations:
  - auth_token_v1_oauth2_token_post
  - test_token_v1_oauth2_test_token_post
  - action_connect_v1_accounts_connect_post
  - Bulk_VIN_connect_v1_vehicles_connect_post
  - bulk_vehicle_connect_group_status_v1_vehicles_connect__id__get
  - bulk_vehicle_connect_vin_results_v1_vehicles_connect__id__results_get
  - list_vehicles_v1_vehicles_get
  - get_vehicle_telemetry_v1_vehicles__vehicle_id__telemetry_get
  - get_vehicle_range_v1_vehicles__vehicle_id__range_get
  - get_vehicle_state_of_health_v1_vehicles__vehicle_id__state_of_health_get
---

# Connect vehicles and read telemetry

## Auth
1. Obtain a token with `auth_token_v1_oauth2_token_post` (`POST /v1/oauth2/token`) using the `client_id`/`client_secret` from a Volteras Portal application. Use a **sandbox** application first (base `https://api.sandbox.volteras.com`); production is `https://api.volteras.com`.
2. Validate it with `test_token_v1_oauth2_test_token_post` — it returns your `clientId` and `organizationId`. Send `Authorization: Bearer <access-token>` on every call.

## Connect vehicles (choose one)
- **Consent flow**: `action_connect_v1_accounts_connect_post` (`POST /v1/accounts:connect`) generates a connect URL (valid one hour). Pass `scopes` (e.g. `vehicle:telemetry`, `vehicle:information`) and, for BMW/MINI/Hyundai/Kia, the `region`. In sandbox any credentials complete the flow.
- **Bulk VIN**: `Bulk_VIN_connect_v1_vehicles_connect_post` (max 100 VINs; returns **202** with a group id — the connection is async). Poll `bulk_vehicle_connect_group_status_v1_vehicles_connect__id__get`, then read per-VIN outcomes with `bulk_vehicle_connect_vin_results_v1_vehicles_connect__id__results_get`. Per-VIN errors use codes from `errors/volteras-error-codes.yml` (e.g. `INVALID_VIN`, `OEM_VEHICLE_NOT_COMPATIBLE`). Or subscribe to the `device.connected` webhook.

## Read data
- `list_vehicles_v1_vehicles_get` — cursor pagination (`maxPageSize`, `pageToken` → `nextPageToken`) plus the JSON filter object (field/operator/value). Without `vehicle:information` consent, `vin` and `registrationPlate` are omitted.
- `get_vehicle_telemetry_v1_vehicles__vehicle_id__telemetry_get` — requires `vehicle:telemetry` scope (401 otherwise). All values are SI units.
- `get_vehicle_range_v1_vehicles__vehicle_id__range_get` and `get_vehicle_state_of_health_v1_vehicles__vehicle_id__state_of_health_get` for range analytics and battery health.

## Rules
- Errors arrive in the envelope `{requestId, timestamp, error, message, data}` — match `error` against the registry in `errors/volteras-error-codes.yml`.
- No idempotency keys exist; do not retry non-GET calls blindly.
