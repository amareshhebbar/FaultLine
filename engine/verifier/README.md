# verifier

The part that keeps the detective honest.

## What it checks

* **Metric evidence**: run the same query at the same time again and compare the value
* **Span evidence**: fetch the trace again and confirm the span ID exists in the claimed service
* **Log evidence**: search the same service around the same timestamp for the same line

## What it rejects

* A claim with no citations
* A claim that cites an ID that does not exist
* A claim where any cited evidence fails its check
* Any report sentence without a valid `[E12]` style citation

## Done means

Unit tests prove that uncited and unverifiable claims never reach the final report.
