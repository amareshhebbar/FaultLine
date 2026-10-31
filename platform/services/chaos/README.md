# chaos

Breaks the shop on purpose and writes down the answer.

## Responsibilities

* `GET /catalogue`: the fault kinds and which services each one can target
* `POST /faults`: start a fault with target, kind, params and duration
* `GET /faults/{id}`: read the ground truth for one fault
* `POST /faults/{id}/stop`: end a fault early
* Writes every fault to the `faults` table: target, kind, params, started_at, ends_at

## Rules

* Sends **no** telemetry to the observability stack
* Services apply faults silently, so the only visible signal is the failure itself
* Only the eval scorer reads the `faults` table

## Phase 1 fault kinds

`latency`, `error`. Then `crash`, `db_slow`, `duplicate`. Full list in the main README.
