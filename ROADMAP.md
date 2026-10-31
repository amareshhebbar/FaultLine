# Faultline Roadmap

A simple, phase by phase plan from an empty repo to a published result. Two owners work in parallel:

* **Platform**: the shop, the telemetry, the chaos controller, Kubernetes, CI/CD
* **AI**: the investigator, the verifier, RAG, the fine tuned model, the eval harness

Each phase ends with a **demo you can show** and **exit criteria you can check**. Do not start the next phase until the current exit criteria pass.

<br>

## 1. Project definition

**What we are building**

A distributed order and payment backend that is deliberately broken by a chaos controller, plus an AI engine that diagnoses each break from logs, metrics and traces, cites evidence for every claim, and is scored against the recorded ground truth.

**In scope**

* Microservices: gateway, orders, payments first; inventory, notifications, ledger later
* Postgres, Redis, a message queue (NATS JetStream or Kafka)
* OpenTelemetry traces, structured logs, Prometheus metrics, Grafana LGTM stack
* A chaos controller with a fault catalogue and a sealed ground truth table
* An AI investigator with tool calling, RAG, a verifier and an eval harness
* A small fine tuned signal compression model and a public dataset on HuggingFace
* Kubernetes deployment, CI/CD, load tests, written trade off documents
* Mintlify docs and a results page with measured numbers

**Out of scope**

* A frontend shop for real users
* Real payment providers or real money
* Automatic fixing of incidents (diagnosis only)
* Multi cloud or multi region deployment

**Hard rules**

1. The engine never reads the `faults` table during an investigation. Only the scorer does.
2. The chaos controller never sends telemetry into the observability stack.
3. A fault must look like a real failure. No log line may say "fault injected".
4. Every claim in a final report cites at least one evidence ID that the verifier reran successfully.
5. No number goes into the README until it is measured.

<br>

## 2. Phase overview

```
 Phase 0   Scaffold                 repo, docs, roadmap, banner              done
    │
 Phase 1   Walking skeleton         3 services, telemetry, 2 faults,         week 1 to 2
    │                               baseline detective, first score
    │
 Phase 2   Reliable platform        queue, outbox, saga, idempotency,        week 3 to 5
    │      + agent detective        retries, breakers, cache;
    │                               LLM tool calling, verifier hardened
    │
 Phase 3   Fault catalogue          10 fault kinds, 500 incident dataset,    week 6 to 7
    │      + RAG                    RAG over runbooks, incidents, code
    │
 Phase 4   Kubernetes + CI/CD       kind cluster, Helm, GitHub Actions,      week 8 to 9
    │      + fine tuned model       signal compressor on HuggingFace
    │
 Phase 5   Results + launch         500 run eval, load tests, results        week 10
                                    page, blog post, demo video
```

<br>

## 3. Phases and tasks

### Phase 0: Scaffold (done)

**Goal:** everyone agrees on what we build and where things live.

* [x] Repository tree with a README in every folder
* [x] Main README written for non technical readers
* [x] Animated banner
* [x] Mintlify docs site
* [x] This roadmap
* [ ] Both owners read the hard rules and agree on the stack

**Exit criteria:** `mint dev` runs in `docs/`, both owners have signed off on section 1.

<br>

### Phase 1: Walking skeleton

**Goal:** the smallest end to end loop. Break one thing, detect it, score it.

**Platform tasks**

* [ ] Choose language and libraries, write them in `platform/README.md`
* [ ] Compose file with Postgres and `grafana/otel-lgtm`
* [ ] `gateway`: `POST /checkout`, calls orders
* [ ] `orders`: creates order, reserves stock with a conditional update, calls payments
* [ ] `payments`: inserts a payment with a unique idempotency key
* [ ] OpenTelemetry traces, metrics and logs from all three services, with `trace_id` in every log line
* [ ] `faults` table: id, target, kind, params, started_at, ends_at, stopped_at
* [ ] `chaos` service: `GET /catalogue`, `POST /faults`, `GET /faults/{id}`, `POST /faults/{id}/stop`
* [ ] Fault kinds: `latency`, `error`
* [ ] `loadgen`: steady checkouts at a configurable rate

**AI tasks**

* [ ] Clients for Prometheus (`/api/v1/query`), Loki (`/loki/api/v1/query_range`), Tempo (`/api/search`, `/api/traces/{id}`)
* [ ] Evidence format: `{id, kind, service, summary, ref, value}` with IDs `E1`, `E2`, ...
* [ ] Verifier: rerun every metric query, refetch every span, find every log line again; reject claims with any failed citation
* [ ] Baseline investigator without an LLM: error ratio, p95, deepest failing span, span with most self time
* [ ] CLI: `faultline inject`, `faultline investigate`, `faultline eval`
* [ ] First eval: 20 runs over `latency` and `error`

**Demo:** inject `latency` into payments, watch Grafana, read the report, see MATCH.

**Exit criteria**

* One command brings the whole stack up
* A trace in Grafana shows gateway, orders and payments in one request
* 20 run eval finishes and prints top 1 and top 3 accuracy
* Verifier unit tests pass, including a test that an uncited claim is rejected

<br>

### Phase 2: Reliable platform and agent detective

**Goal:** the platform behaves like production software, and the detective becomes an agent.

**Platform tasks**

* [ ] Message queue (NATS JetStream or Kafka), decision written down
* [ ] Transactional outbox in orders, relay process publishes events
* [ ] Saga for checkout: reserve stock, charge, confirm; compensation on failure
* [ ] Idempotent consumers, deduplication by key
* [ ] Retries with exponential backoff and full jitter, per call timeouts
* [ ] Circuit breaker on service to service calls
* [ ] Dead letter queue with a replay tool
* [ ] Redis cache for product data, invalidation strategy written down
* [ ] Concurrency doc: row locks versus optimistic versioning, with the chosen trade off
* [ ] New fault kinds: `crash`, `db_slow`, `duplicate`
* [ ] Test: 10x redelivery of every payment message produces zero duplicate charges

**AI tasks**

* [ ] Agent loop: the LLM chooses which query to run next from a fixed tool list
* [ ] Tool schemas: `query_metrics`, `search_logs`, `search_traces`, `get_trace`, `list_services`
* [ ] Every tool result is stored as evidence with an ID before the LLM sees it
* [ ] Report writer must cite IDs; sentence filter drops anything uncited
* [ ] Budget limits: max tool calls, max tokens, max seconds per investigation
* [ ] Works with a local model (Ollama) and a hosted model through one OpenAI compatible interface
* [ ] Eval compares baseline versus agent on the same seeds

**Demo:** side by side report from baseline and agent for the same incident.

**Exit criteria**

* Zero duplicate charges under 10x redelivery, proven by a SQL check
* Agent beats or matches baseline top 1 accuracy on 50 runs
* Zero uncited claims in all final reports

<br>

### Phase 3: Fault catalogue and RAG

**Goal:** many realistic failures, and a detective that knows the system.

**Platform tasks**

* [ ] Add `inventory`, `notifications`, `ledger` services
* [ ] Fault kinds: `poison_message`, `network_partition` (Toxiproxy), `clock_skew` (libfaketime), `bad_deploy` (version flag), `cache_stampede`, `pool_exhaustion`
* [ ] Fault scheduler: random target, kind, intensity and duration with a seed
* [ ] Runbooks in `docs/` for each service

**AI tasks**

* [ ] RAG index over runbooks, past incident reports and the platform source code
* [ ] Retrieval tool for the agent: `search_runbooks`, `search_code`, `similar_incidents`
* [ ] Retrieved passages also become evidence with IDs
* [ ] Dataset builder: every eval run writes a labelled row (signals, label, prediction)
* [ ] 500 incident dataset collected across all fault kinds

**Demo:** the agent cites a runbook paragraph and a line of source code next to the trace evidence.

**Exit criteria**

* 10 fault kinds pass a manual check that each looks real from the outside
* 500 labelled incidents saved with balanced kinds
* RAG improves top 1 accuracy over Phase 2 on the same seeds, or the reason it does not is written down

<br>

### Phase 4: Kubernetes, CI/CD and the fine tuned model

**Goal:** production style deployment, and a small model that makes investigations cheaper.

**Platform tasks**

* [ ] Helm chart for every service
* [ ] Local cluster with kind, one command to install
* [ ] GitHub Actions: lint, test, build images, deploy to kind, run a 5 run smoke eval
* [ ] Load tests with k6, p95 checkout latency reported at fixed rates
* [ ] Optional: Chaos Mesh as a second fault injector inside Kubernetes

**AI tasks**

* [ ] Training pairs: raw telemetry window as input, structured signals as output
* [ ] Fine tune a small open model (1.5B to 3B) with LoRA
* [ ] Publish model and dataset on HuggingFace with a model card
* [ ] Agent uses the small model to compress windows before the large model reasons
* [ ] Measure cost and time per investigation with and without compression

**Demo:** CI pipeline goes green and posts a smoke eval score on the pull request.

**Exit criteria**

* Fresh clone to running cluster in one command
* Model and dataset public on HuggingFace
* Cost per investigation measured both ways

<br>

### Phase 5: Results and launch

**Goal:** real numbers, written up clearly.

* [ ] 500 run eval on the final build, fixed seeds, results committed
* [ ] Results page in docs: accuracy by fault kind, confusion table, time, cost
* [ ] Platform numbers: duplicate charges under redelivery, p95 at a fixed rate
* [ ] README updated with measured numbers only
* [ ] Blog post: how we built a root cause AI with a real accuracy number
* [ ] Two minute demo video
* [ ] Resume lines filled in from the results page

**Exit criteria:** every number on the README links to a result file in the repo.

<br>

## 4. Path to completion

**How to start:** Phase 1 only. Three services, two faults, one baseline detective, one score. Resist adding anything else until the loop closes.

**How to test**

* Platform: unit tests per service, an integration test that runs a checkout end to end, a SQL check for duplicate charges, k6 for load
* AI: unit tests for the verifier and scoring with fake data, then the eval harness against the live stack
* Together: the eval harness is the shared integration test. If it runs, the contract between both sides holds.

**How to bridge milestones**

The contract between platform and AI is small and fixed early:

1. Service names in telemetry: `service.name` equals the folder name
2. The chaos API: `GET /catalogue`, `POST /faults`, `GET /faults/{id}`
3. The ground truth row: target, kind, params, started_at, ends_at

As long as these three do not change, both owners can move at their own speed.

**How to finish**

The project is done when a stranger can clone the repo, run one command, watch a fault get injected and diagnosed, and find the accuracy number on the results page with the raw files behind it.
