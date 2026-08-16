---
name: Change FranklinWH battery and circuit settings safely
description: Apply time-of-use profiles, aPower start/stop, smart-circuit and grid-event settings to a FranklinWH device or group, with the replay and audit precautions this API requires.
api: openapi/franklin-whole-home-openapi.yml
operations:
  - tokenizer
  - queryTouProfile
  - setTouProfile
  - queryApowerSwitchStatus
  - setSwitchParam
  - querySmartCircuitParameters
  - setSmartCircuits
  - queryGridEvents
  - setGridEvents
  - setDevicesByGroup
  - fetchOperationLog
generated: '2026-08-16'
method: generated
source: openapi/franklin-whole-home-openapi.yml
---

# Change FranklinWH battery and circuit settings safely

These operations act on physical hardware in someone's home: they start and stop a battery, shed
household circuits, and schedule grid interactions. Read the safety rules at the bottom before
writing anything.

Base URL: `https://test-api.franklinwh.com`. Authenticate with `POST /api-common/tokenizer`
(`tokenizer`) and send the token as the raw `Authorization` header value.

## Always read before you write

Every setting has a matching query operation. Call it first and keep the result — this API has no
idempotency key and no rollback, so the prior state you captured is your only undo.

| Read | Write |
|---|---|
| `GET /api-common/queryTouProfile` (`queryTouProfile`) | `POST /api-common/setTouProfile` (`setTouProfile`) |
| `GET /api-common/queryApowerSwitchStatus` (`queryApowerSwitchStatus`) | `POST /api-common/setSwitchParam` (`setSwitchParam`) |
| `GET /api-common/querySmartCircuitParameters` (`querySmartCircuitParameters`) | `POST /api-common/setSmartCircuits` (`setSmartCircuits`) |
| `GET /api-common/queryGridEvents` (`queryGridEvents`) | `POST /api-common/setGridEvents` (`setGridEvents`) |

## Time-of-use profiles

`setTouProfile` takes `deviceId`, `touId` and a `season` array. Each season carries `month`
(comma-separated month numbers), `dayType`, and a `time` array of `{startTime, endTime, waveType,
schedule}` using `"HH:MM"` clock strings. FranklinWH publishes a complete request example, carried
verbatim in the OpenAPI `requestBody` example — start from it rather than inventing a shape.

## aPower start / stop

`setSwitchParam` takes `deviceId` and `cmd`: **1 = start up, 2 = shut down**. This is a physical
action. Confirm with `queryApowerSwitchStatus` afterwards; do not infer success from the envelope
alone.

## Smart circuits

`setSmartCircuits` controls up to three circuits (`sw1`–`sw3`). Per circuit: `Name`, `MsgType`
(1 manual switch, 2 parameter settings), `SocLowSet` (0–100, the state-of-charge threshold at which
the circuit sheds), `Mode` (0 manually off, 1 manually on, 2 timing plan), `ProLoad` (0 off, 1 on),
`Freq` (0 single, 1 daily, 2 weekly, 3 monthly), `TimeEn` and `Time` (**maximum 2 time periods per
circuit**). Setting the merge flag joins circuits 1 and 2 — every `sw2` value is then ignored.

## Grid events

`setGridEvents` takes `deviceId` and an `eventList` of `{id, cmd, start, end, power, resoc, feedEn}`
where `start` / `end` are **Unix epoch seconds**. A published example ships in the OpenAPI.

## Bulk changes

`POST /api-common/setDevicesByGroup` (`setDevicesByGroup`) applies any of the above to a whole
group, keyed by `groupId` **or** `groupName`, with `setType` selecting which setting (1 device info,
2 energy management, 3 TOU, 4 aPower switch, 5 smart circuits, 6 grid events, 7 grid package,
8 grid interconnection compliance) and `setData` carrying the same fields as the single-device call
minus the device id. One call can change every battery in a fleet. Stage it against one device
first.

## Verify what happened

`GET /api-common/fetchOperationLog` (`fetchOperationLog`) is the audit trail. Filter with
`optRecordsType` (0 all, 1 site, 2 device info, 3 energy management, 4 TOU, 5 aPower switch,
6 smart circuits, 7 grid events, 8 upgrade records, 9 grid package, 10 grid interconnection
compliance, 11 commissioning), plus `deviceId`, `siteId`, `queryDate`, and `current` / `pageSize`
(default 20, max 50). Read it back after every write.

## Safety rules

- **No idempotency.** FranklinWH publishes no idempotency key or request-dedup header anywhere in
  its catalogue. **Never blind-retry a write.** On a timeout or ambiguous response, call the
  matching query operation to find out what actually happened, then decide.
- **Failures arrive as HTTP 200.** Check the envelope `code` — `401` wrong token, `403` missing
  token or token param. An HTTP 200 does not mean the battery moved.
- **No rate-limit signal.** No published limits, no `RateLimit-*` headers, no documented 429. Serialise
  writes; do not fan out.
- **Human in the loop.** `setSwitchParam`, `setSmartCircuits`, `setGridEvents` and any
  `setDevicesByGroup` call carrying them change household power delivery. Confirm with a person
  before executing them autonomously, and never as part of an unattended loop.
- Exercise everything against `https://test-api.franklinwh.com` first — FranklinWH provides it free
  and states its data formats match production.
