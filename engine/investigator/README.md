# investigator

Finds the suspect.

## Phase 1 baseline (no LLM)

Per service, compare the incident window to the minutes before it:

* Error ratio and request rate
* p95 latency of HTTP and of database calls
* Error log count
* From traces: the deepest failing span, the span that owns the most self time, calls that never reached a server span

Combine these into a score for each `(service, fault kind)` pair.

## Phase 2 agent

An LLM picks which tool to call next: `query_metrics`, `search_logs`, `search_traces`, `get_trace`. Every tool result becomes evidence before the model sees it.

## Phase 3 RAG

Extra tools: `search_runbooks`, `search_code`, `similar_incidents`.
