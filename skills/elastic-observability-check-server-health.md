---
name: elastic-observability-check-server-health
description: Verify an Elastic APM Server is reachable, authenticated and ready before shipping telemetry to it.
api: openapi/elastic-observability-server-info-api-openapi.yml
operations:
  - getServerHealth
  - postServerHealth
generated: '2026-08-29'
method: generated
source: openapi/_original/elastic-observability-observability-intake-openapi.yml
---

# Check APM Server health

`getServerHealth` (`GET /`) is the cheapest safe call in this API — it is a read, it writes
nothing, and it is the right first request from any new integration.

## Steps

1. `GET /` against your APM Server base URL with your `Authorization` header.
2. A `200` confirms three things at once: the host is right, the credential is accepted, and the
   server is up. Use it as a preflight before the first `postEventIntake`.
3. `postServerHealth` (`POST /`) returns the same information for clients that cannot issue a GET.

## Why this is the right probe

Every other operation in this contract is a write into an append-only telemetry stream with no
reversal path. Do not "test the connection" by sending a throwaway event — you cannot take it back,
and it will show up in the customer's data. Probe with `getServerHealth` instead.

## Related

- `security/elastic-observability-domain-security.yml` — TLS posture of the Elastic hosts.
- `lifecycle/elastic-observability-lifecycle.yml` — the Elastic Cloud status page and its
  machine-readable Statuspage API, for distinguishing "my deployment" from "Elastic".
