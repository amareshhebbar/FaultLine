# dataset

Every eval run produces a labelled example for free, because we caused the incident ourselves.

## One row

* Telemetry window and extracted signals per service
* Label: target service, fault kind, params
* Prediction from the detective

## Uses

* Training the small signal compression model
* A public HuggingFace dataset of labelled distributed systems incidents
* Regression tests for future versions of the detective
