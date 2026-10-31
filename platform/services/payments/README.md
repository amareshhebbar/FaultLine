# payments

Charges each order exactly once.

## Responsibilities

* `POST /payments` with a required idempotency key
* Unique constraint on the key; a repeat returns the original payment instead of charging again
* Marks repeats on the span (`payment.deduplicated = true`) and counts them in a metric
* Later: consumes payment commands from the queue with idempotent handling

## Done means

10x redelivery of every payment produces zero duplicate charges, checked with SQL.
