# Engine

The AI detective. Owned by the **AI** side.

## What lives here

* `investigator/`: gathers metrics, logs and traces and ranks suspects
* `verifier/`: rechecks every citation and removes unproven claims
* `evaluation/`: injects faults, runs investigations, scores them against ground truth
* `dataset/`: labelled incidents that feed the fine tuned model and HuggingFace

## Flow of one investigation

1. Take a time window
2. Collect signals per service, store each finding as evidence `E1`, `E2`, ...
3. Rank `(service, fault kind)` hypotheses
4. Verify every cited evidence ID against the live sources
5. Write a short report where every sentence cites evidence

## Rules

* Never read the `faults` table while investigating
* A claim without verified evidence never reaches the report
* Build the verifier and the baseline before adding an LLM

## Done means

`faultline eval` runs N incidents and prints top 1 accuracy, top 3 accuracy, time and cost.
