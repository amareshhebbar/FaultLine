# Platform

The shop that gets broken on purpose. Owned by the **platform** side.

## What lives here

* `services/`: one folder per microservice
* `deploy/`: Docker Compose for local runs, Kubernetes manifests later

## Stack

Fill this in before writing code.

* Language:
* HTTP framework:
* Database: Postgres
* Cache: Redis
* Queue: NATS JetStream or Kafka
* Telemetry: OpenTelemetry SDK, exported over OTLP

## Contract with the AI engine

1. Every service sets `service.name` to its folder name.
2. Every log line carries `trace_id` and `span_id`.
3. Faults look like real failures. No log, metric or span may mention the chaos controller.
4. The chaos API and the `faults` table shape stay stable once Phase 1 ends.

## Done means

One command starts everything, and a single checkout shows up in Grafana as one trace across gateway, orders and payments.
