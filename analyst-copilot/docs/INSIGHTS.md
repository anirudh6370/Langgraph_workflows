# Insights & design rationale

This document records *why*, not *what* — the tradeoffs behind each architectural
choice, and the specific lessons carried over from reviewing the earlier
`database_discovery_fleet.ipynb` ("DataScribe") notebook.

## Lessons carried over from DataScribe

| DataScribe issue | Fix here |
|---|---|
| `Config` referenced `self.openai_api_key`, which was never set — crashed on init. | Config validation is covered by a test in Phase 7; fields are validated against what's actually assigned. |
| Schema "discovery" asked an LLM to guess tables/columns/FKs as JSON, parsed with a brittle string slice (`output_[output_.find('\n')+1:...]`). Hallucination-prone and fragile. | Schema comes from SQLAlchemy reflection — exact, instant, free. The LLM never has to reconstruct structural facts it can just be given. |
| `SupervisorAgent.discovery_agent` was built but never used; `discover_database()` built a second, separate `DiscoveryAgent` instead. | Agents are constructed once (via `st.cache_resource` at the UI layer, or a single composition root elsewhere) and passed in, not silently duplicated. |
| A fresh `ChatGroq` client was constructed inside `classify_user_input` on every single graph invocation. | LLM clients are constructed once at startup and reused across nodes/turns. |
| Discovery ran once per process (via a manual `if state.get('db_graph') is None` check inside graph state) — lost on every restart. | Long-term `Store` persists the schema graph across restarts, not just across turns within one process. |
| No enforcement of the read-only disclaimer beyond a warning comment. | Read-only DB connection + query-checker step, as an actual constraint. |
| `create_tool_calling_agent` + `AgentExecutor` (via `langchain_classic`) — the legacy agent pattern. | `langgraph.prebuilt.create_react_agent`, the current idiomatic replacement. |

## Why four branches instead of always going through the deep agent

It would be simpler to route everything through one deep agent and let it decide how
much planning a question needs. The reason not to: a deep agent's planning + virtual
file system overhead is pure latency and token cost on a question like "how many
tracks are there" — and demonstrating *when a ReAct agent is the right tool* is as
important a lesson as demonstrating the deep agent itself. The routing/classification
step is doing real work, not just decoration.

## Why `Send` + reducer instead of a loop inside one node

A single node could loop over sub-questions sequentially and concatenate results
itself. That avoids needing a reducer at all — but it also serializes work that has no
dependency between steps, and it hides the fan-out from LangGraph's own execution
model (no per-branch tracing, no independent retries, no natural place to parallelize
later). Using `Send` makes the parallelism a first-class part of the graph rather than
an implementation detail buried in a node function.

## Why two memory primitives instead of one

Early instinct: just persist all state to one database and call it "memory." The
problem is that short-term (per-conversation) and long-term (per-user, cross-session)
memory have different scoping, different lifetimes, and different write patterns —
short-term is written every turn and can be pruned; long-term is written rarely and
must never silently get overwritten by an unrelated thread. LangGraph's checkpointer
and `Store` map directly onto that distinction, so using both (rather than one system
serving double duty) mirrors a real semantic difference, not just API availability.

## Why SQLite for both memory backends initially

Matches the existing project's zero-infra setup (no external DB server to run for a
learning project) while keeping the interface (`BaseCheckpointSaver`, `BaseStore`)
identical to what a Postgres-backed swap would use later — the migration path is a
constructor change, not a rewrite.

## Open questions to revisit once Phase 1–2 are built

- Whether the "simple vs compound question" split in the classifier is reliable enough
  from a single LLM call, or whether it needs a few-shot prompt / examples from real
  Chinook questions.
- Whether `InMemoryStore` is sufficient through Phase 6 or whether a persistent Store
  backend is needed earlier to actually test cross-restart recall.
