# deploy

How the platform runs.

## Phase 1

Docker Compose with:

* Postgres with the schema and the `faults` table
* `grafana/otel-lgtm`: OpenTelemetry collector, Prometheus, Loki, Tempo and Grafana in one container
* gateway, orders, payments, chaos, loadgen

Ports to expose for the engine: Prometheus `9090`, Loki `3100`, Tempo `3200`, Grafana `3000`, chaos API.

## Phase 4

Helm charts, a kind cluster, and GitHub Actions that deploy and run a smoke eval.
