---
name: Inventory a FranklinWH fleet
description: Authenticate to the FranklinWH partner API and page through every site and device an installer owns.
api: openapi/franklin-whole-home-openapi.yml
operations:
  - tokenizer
  - querySiteList
  - querySiteInfo
  - queryDeviceList
  - queryDeviceParameters
  - queryDeviceRunningStatus
generated: '2026-08-16'
method: generated
source: openapi/franklin-whole-home-openapi.yml
---

# Inventory a FranklinWH fleet

Base URL: `https://test-api.franklinwh.com` (the only base URL FranklinWH publishes; the production
base URL is issued during partner onboarding). All operations are `GET` unless noted.

## 1. Get a token

`POST /api-common/tokenizer` (`tokenizer`) with your `cp` and `ck` credential pair in the JSON body.
The response is the standard envelope — `{"code": ..., "msg": ..., "data": ...}`.

Send the returned token as the **raw value** of the `Authorization` header on every subsequent
request. There is no `Bearer ` prefix.

## 2. Page through the sites

`GET /api-common/querySiteList` (`querySiteList`). Optional filters: `installerId`, `siteName`,
`userAccount`, `deviceId`.

Pagination is offset-based:

- `current` — page number, starts at **1**, defaults to page 1 if omitted
- `pageSize` — defaults to **20**, **maximum 50**

Loop `current` upward until a page comes back short. FranklinWH publishes no total-count field, so
short-page is the only stop condition available.

Use `GET /api-common/querySiteInfo` (`querySiteInfo`) with a required `siteId` when you need one
site on its own.

## 3. Walk the devices

`GET /api-common/queryDeviceList` (`queryDeviceList`) returns the aGate / aPower units. For each
device id:

- `GET /api-common/queryDeviceParameters` (`queryDeviceParameters`) — configured parameters
- `GET /api-common/queryDeviceRunningStatus` (`queryDeviceRunningStatus`) — live running status

Device ids are strings; the one example FranklinWH publishes is a 20-character alphanumeric serial.

## Rules this API imposes

- **Check `code`, not the HTTP status.** Failures come back with **HTTP 200** and a non-zero `code`
  in the body: `401` = `wrong token`, `403` = `missing token or token param`. Treating HTTP 200 as
  success will silently swallow an auth failure.
- **On `code: 401`, re-run `tokenizer`** and retry once. Do not loop.
- **No rate-limit headers.** FranklinWH says rate limiting and anomaly detection are enforced but
  publishes no limits, no window and no response headers, and documents no 429. Pace yourself
  conservatively and back off on any repeated non-zero `code`.
- **No response schemas are published.** Do not assume field names inside `data`; read what the API
  returns and fail loudly on shape changes.

See `conventions/franklin-whole-home-conventions.yml` and
`errors/franklin-whole-home-problem-types.yml`.
