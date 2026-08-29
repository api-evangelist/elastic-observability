---
name: elastic-observability-fetch-agent-config
description: Read central APM agent configuration from Elastic Observability for a given service and environment.
api: openapi/elastic-observability-agent-config-api-openapi.yml
operations:
  - getAgentConfig
  - postAgentConfig
  - getRumAgentConfig
generated: '2026-08-29'
method: generated
source: openapi/_original/elastic-observability-observability-intake-openapi.yml
---

# Fetch central APM agent configuration

Central configuration lets you change agent settings — sample rate, capture body, transaction
max spans — from Kibana without redeploying the instrumented service. Agents poll for it.

## Operations

- `getAgentConfig` — `GET /config/v1/agents`
- `postAgentConfig` — `POST /config/v1/agents` (same lookup, form-encoded body)
- `getRumAgentConfig` — `GET /config/v1/rum/agents` (browser RUM agents)

## Steps

1. Authenticate with an API key or the APM secret token — one `Authorization` header.
2. Identify the service you are asking about. The lookup is keyed on the service name, and
   optionally the service environment, so that a config can be scoped to staging separately from
   production.
3. Read the returned JSON object of configuration values.
4. Poll on an interval. There is no push, no webhook and no ETag documented for this endpoint.

## Handling failure

These are the **only two error responses declared anywhere in this API**, and both are on
`postAgentConfig`:

- `403` — "APM Server is configured to fetch agent configuration from Elasticsearch but the
  operation is not permitted." Fix the APM Server's Elasticsearch privileges, or turn central
  configuration off. Do not retry; it will not clear on its own.
- `503` — "APM Server is starting up or Elasticsearch is unreachable." This one **is** transient.
  Retry with exponential backoff and keep running on the last known configuration meanwhile.

Never fail your application because central config could not be read. It is an optimisation, not a
dependency — carry on with the settings you already have.
