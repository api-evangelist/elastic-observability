---
name: elastic-observability-export-otlp-telemetry
description: Export OpenTelemetry traces, metrics and logs into Elastic Observability over OTLP/HTTP or OTLP/gRPC.
api: openapi/elastic-observability-opentelemetry-intake-api-openapi.yml
operations:
  - postOtlpHttpTraces
  - postOtlpHttpMetrics
  - postOtlpHttpLogs
  - postOtlpGrpcTraces
  - postOtlpGrpcMetrics
  - postOtlpGrpcLogs
generated: '2026-08-29'
method: generated
source: openapi/_original/elastic-observability-observability-intake-openapi.yml
---

# Export OTLP telemetry to Elastic Observability

Elastic accepts native OpenTelemetry Protocol on the same port as the APM intake API, in two
encodings. Pick one — do not send the same signal twice.

## OTLP/HTTP

- `postOtlpHttpTraces` — `POST /v1/traces`
- `postOtlpHttpMetrics` — `POST /v1/metrics`
- `postOtlpHttpLogs` — `POST /v1/logs`

## OTLP/gRPC

- `postOtlpGrpcTraces` — `POST /opentelemetry.proto.collector.trace.v1.TraceService/Export`
- `postOtlpGrpcMetrics` — `POST /opentelemetry.proto.collector.metrics.v1.MetricsService/Export`
- `postOtlpGrpcLogs` — `POST /opentelemetry.proto.collector.logs.v1.LogsService/Export`

## Steps

1. Point any conformant OTLP exporter at the Elastic endpoint. Nothing Elastic-specific is required
   in the payload — this is the standard wire format, which is the whole point of using it.
2. Set the body encoding to `application/x-protobuf` (or `application/json`; both are declared).
3. Authenticate with an Elastic API key in the OTLP headers:
   `OTEL_EXPORTER_OTLP_HEADERS="Authorization=ApiKey <key>"`.
4. Set `OTEL_EXPORTER_OTLP_ENDPOINT` to your Elastic endpoint.
5. Success is `200`.

## Choosing an SDK

Elastic ships Elastic Distributions of OpenTelemetry (EDOT) for Node.js, Python, Java, .NET and PHP
— upstream OTel SDKs with Elastic defaults already set. See
`packages/elastic-observability-packages.yml`. Using EDOT is not required; a stock OTel SDK works,
because the endpoint is standards-conformant.

## Handling failure

Elastic documents `429 Too Many Requests` on managed OTLP ingest, surfaced over gRPC as
`rpc error: code = ResourceExhausted desc = request exceeded available capacity`. **The published
OpenAPI does not declare it** and no `Retry-After` or `X-RateLimit-*` header is documented, so back
off on your own schedule. See `rate-limits/elastic-observability-rate-limits.yml`.

Ingest is append-only and irreversible — the same warning as the v2 intake path applies.
