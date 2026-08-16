---
name: Pull FranklinWH energy and backup telemetry
description: Read power, energy, power-source, telemetry-pool and historical load data for a FranklinWH device, plus its warning and backup-event history.
api: openapi/franklin-whole-home-openapi.yml
operations:
  - tokenizer
  - queryPowerData
  - getMaxPowerData
  - queryEnergyData
  - queryPowerSourceDetails
  - queryDeviceDataPool
  - historyDataLoad
  - queryDeviceHistoricalWarning
  - queryBackupEvents
generated: '2026-08-16'
method: generated
source: openapi/franklin-whole-home-openapi.yml
---

# Pull FranklinWH energy and backup telemetry

Base URL: `https://test-api.franklinwh.com`. Authenticate first with
`POST /api-common/tokenizer` (`tokenizer`) and send the token as the raw `Authorization` header
value on every call.

You need a `deviceId` before any of this works — get one from
`skills/franklin-whole-home-fleet-inventory.md`.

## Instantaneous and interval power

- `GET /api-common/queryPowerData` (`queryPowerData`) — power data for a device
- `POST /api-common/getMaxPowerData` (`getMaxPowerData`) — peak power over a window
- `POST /api-common/queryPowerSourceDetails` (`queryPowerSourceDetails`) — the breakdown across
  solar, battery, grid, generator and EV

## Energy totals

- `POST /api-common/queryEnergyData` (`queryEnergyData`)

## Raw telemetry and history

- `GET /api-common/queryDeviceDataPool` (`queryDeviceDataPool`) — telemetry grouping
- `GET /api-common/historyDataLoad` (`historyDataLoad`) — historical load
- `GET /api-common/queryInventory` (`queryInventory`) — inventory readings

## Outages and faults

- `GET /api-common/queryDeviceHistoricalWarning` (`queryDeviceHistoricalWarning`)
- `GET /api-common/queryBackupEvents` (`queryBackupEvents`) — grid-outage / backup events

## Time formats — this API uses three

Read the parameter notes before formatting a timestamp. FranklinWH mixes:

1. **ISO 8601 UTC** on the Sunrun backfill window (published example: `2025-01-09T00:00:00Z`)
2. **Unix epoch seconds** on grid-event `start` / `end`
3. **Clock strings** — `"HH:MM"` and `"YYYY-MM-DD HH:MM/YYYY-MM-DD HH:MM"` — on TOU and
   smart-circuit schedules

Do not carry a format across operations.

## Rules

- Failures return **HTTP 200** with a non-zero `code`. Check `code` on every response.
- FranklinWH publishes **no response payload schemas**. Treat `data` as untyped; validate defensively.
- There are **no rate-limit response headers** and no documented 429. Poll conservatively; this is
  telemetry, not a control surface, so widen your interval rather than retrying hard.
- This is homeowner energy data. Responses are stated to exclude personally identifiable
  information, but site records carry addresses, postcodes and user accounts — handle accordingly.
