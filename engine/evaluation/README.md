# evaluation

Turns the detective into a number.

## One run

1. Pick a random target and fault kind from the catalogue (seeded)
2. Start the fault, wait for it to end, wait a little more for telemetry to arrive
3. Investigate the window, verify, write the report
4. Read the ground truth and score

## Scores

* Top 1 accuracy: first guess has the right service
* Top 3 accuracy: right service is in the first three guesses
* Kind accuracy: first guess also has the right fault kind
* Time to diagnosis and cost per investigation
* Claims rejected by the verifier, and uncited claims in final reports (must be 0)

## Output

One folder per eval run with a summary, every outcome, every report, and the dataset rows.
