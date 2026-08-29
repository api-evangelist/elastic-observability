---
name: elastic-observability-send-apm-events
description: Ship APM transactions, spans and errors into Elastic Observability over the v2 intake API using newline-delimited JSON.
api: openapi/elastic-observability-event-intake-api-openapi.yml
operations:
  - postEventIntake
generated: '2026-08-29'
method: generated
source: openapi/_original/elastic-observability-observability-intake-openapi.yml
---

# Send APM events to Elastic Observability

Use `postEventIntake` (`POST /intake/v2/events`) to deliver backend telemetry.

## Before you start

In almost every case you should not call this API by hand — install the first-party agent for your
runtime (see `packages/elastic-observability-packages.yml`) and let it manage batching, backoff and
the correlation ids. Call the endpoint directly only when you are building an agent, a bridge, or a
test harness.

## Authentication

Send exactly one `Authorization` header. The two schemes are alternatives, never both:

- `Authorization: ApiKey <base64(id:api_key)>` — preferred; scoped and individually revocable.
- `Authorization: Bearer <secret-token>` — the deployment-wide APM secret token.

## Base URL

There is no global host. Elastic Cloud Hosted serves this at
`https://{deployment}.apm.{region}.cloud.es.io:443`; a self-managed APM Server defaults to
`http://localhost:8200`. Read the address from your Elastic Cloud console.

## Steps

1. Set `Content-Type: application/x-ndjson`.
2. Write the **metadata line first**. Every batch begins with one `MetadataEvent`:
   `{"metadata":{"service":{...}}}`. `metadata.service` is required. Everything after it inherits
   this context, so a batch with no metadata line is rejected.
3. Append one JSON object per line, one event per line, no array wrapper:
   - `{"transaction":{...}}` — requires `trace_id`, `id`, `type`, `span_count`, `duration`.
   - `{"span":{...}}` — requires `id`, `trace_id`, `name`, `parent_id`, `type`, `duration`.
   - `{"error":{...}}` — requires `id`.
   - `{"metricset":{...}}` — requires `samples`.
4. Generate the correlation ids yourself: a 128-bit hex `trace_id`, 64-bit hex `id`, and set
   `parent_id` / `transaction_id` to stitch spans and errors to their transaction. The server
   assigns nothing.
5. POST the body. A success is **`202 Accepted`** — not 200, and not a body you can read ids out of.

## Handling failure

- The published contract declares only `202`. It does **not** declare a 400, a 429 or a 5xx, so do
  not build control flow on documented error shapes that are not there. See
  `errors/elastic-observability-problem-types.yml`.
- A malformed NDJSON line makes the server reject events; validate locally before sending.

## The rule that matters most

**This surface is append-only, has no idempotency key, and has no reversal operation.** Re-sending
a batch after an ambiguous timeout creates duplicate telemetry — it does not replay safely. Treat a
POST whose response you did not see as *possibly delivered*, and prefer dropping a batch over
double-sending one. There is no cancel, void or delete to call afterwards; see
`conventions/elastic-observability-conventions.yml`.
