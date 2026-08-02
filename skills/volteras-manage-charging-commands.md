---
name: Manage EV charging with Volteras commands
description: >-
  Safely send asynchronous charging and vehicle commands (start/stop charging,
  charge limit, locks, climate) and track their execution and rate limits.
api: openapi/volteras-connect-openapi-original.json
generated: '2026-07-21'
method: generated
operations:
  - action_start_charging_v1_vehicles__vehicle_id__command_execution_start_charging_post
  - action_stop_charging_v1_vehicles__vehicle_id__command_execution_stop_charging_post
  - action_set_charge_limit_v1_vehicles__vehicle_id__command_execution_set_charge_limit_post
  - action_lock_doors_v1_vehicles__vehicle_id__command_execution_lock_doors_post
  - action_unlock_doors_v1_vehicles__vehicle_id__command_execution_unlock_doors_post
  - get_vehicle_command_execution_v1_vehicles__vehicle_id__command_executions__vehicle_command_execution_id__get
  - list_vehicle_command_executions_v1_vehicles__vehicle_id__command_executions_get
  - get_rate_limits_v1_rate_limits_get
---

# Manage EV charging with Volteras commands

## Preconditions
- Bearer token from `POST /v1/oauth2/token`; the connected account must have granted the `vehicle:commands` consent scope (401 otherwise).
- Commands are **physical actions on real vehicles** — in agent contexts require explicit human confirmation before lock/unlock or charging changes (see `agentic-access/volteras-agentic-access.yml`).

## Send a command (all async)
1. Create the execution, e.g. `action_start_charging_v1_vehicles__vehicle_id__command_execution_start_charging_post`, `action_stop_charging_v1_vehicles__vehicle_id__command_execution_stop_charging_post`, or `action_set_charge_limit_v1_vehicles__vehicle_id__command_execution_set_charge_limit_post` (payload `SetChargeLimitPayload`). Door control: `action_lock_doors_...` / `action_unlock_doors_...`.
2. Only **one pending command of each type per vehicle** is allowed — creating a duplicate while one is pending fails. Commands time out after 5 minutes.
3. Track the result: poll `get_vehicle_command_execution_v1_vehicles__vehicle_id__command_executions__vehicle_command_execution_id__get` (status `PENDING` → `EXECUTED`/`FAILED`) or subscribe to the `command.updated` webhook. Audit history via `list_vehicle_command_executions_v1_vehicles__vehicle_id__command_executions_get`.

## Handle failures and limits
- On `FAILED`, read `failedReason` and match it in `errors/volteras-error-codes.yml` — OEM-side conditions like `OEM_VEHICLE_OFFLINE`, `OEM_COMMAND_REFUSED_VEHICLE_AWAKE_IN_DRIVING_OR_READY_TO_DRIVE`, or `OEM_STOP_CHARGE_RATE_LIMITED` are not retryable immediately.
- Check quota before bursts: `get_rate_limits_v1_rate_limits_get` (`objectId` = vehicle id, `domain=VEHICLE`) returns `remainingRequests`/`maxRequests`/`windowSizeSeconds`; the `rate_limit.updated` webhook pushes changes.
