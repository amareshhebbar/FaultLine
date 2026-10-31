# loadgen

Robot shoppers. Sends a steady stream of checkouts so there is always traffic to observe.

## Responsibilities

* Configurable requests per second
* Random customers, items and amounts
* Prints a short success and failure summary every few seconds
* Sends no telemetry of its own, so traces start at the gateway

## Done means

Grafana shows a flat, healthy baseline when no fault is active.
