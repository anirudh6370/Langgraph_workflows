# Analyst Copilot

A conversational AI data-analyst built on **LangGraph**, sitting on top of a relational
database (starting with the existing Chinook-style `ani.db`). It answers direct
questions, fans out compound questions to parallel sub-agents, and hands open-ended
"analyze and report" requests to a planning **deep agent** — all while remembering
context within a session and durable facts across sessions.

This project is the successor to `../database_discovery_fleet.ipynb` ("DataScribe"),
carrying forward the idea but fixing its architectural issues (see
[docs/INSIGHTS.md](docs/INSIGHTS.md)) and deliberately exercising the core LangGraph
primitives an AI engineer should know cold.

## Problem statement

Give a non-expert user a chat interface over a database that can:

1. **Answer direct factual questions** ("how many tracks does each genre have?") via a
   single ReAct SQL agent.
2. **Answer compound questions** ("compare revenue by genre AND by country") by
   splitting into sub-questions, running them **in parallel**, and merging the results
   with a **reducer**.
3. **Handle open-ended analysis/report requests** ("write me a Q3 vs Q4 regional
   performance report") via a **deep agent** — planning, a scratch file system, and
   sub-agent delegation — because a single ReAct loop doesn't have the right shape for
   a multi-step, artifact-producing task.
4. **Remember the current conversation** (short-term memory, checkpointed per thread)
   so follow-up questions work without resending history.
5. **Remember durable facts across sessions** (long-term memory): the schema graph
   (computed once, not re-guessed by an LLM), a learned business glossary, and past
   report artifacts.
6. Optionally **pause for human approval** before an expensive multi-table plan runs.

## Where things live

| Path | Purpose |
|---|---|
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | Graph topology, state schema, memory design, tech stack |
| [docs/ROADMAP.md](docs/ROADMAP.md) | Phased build plan |
| [docs/INSIGHTS.md](docs/INSIGHTS.md) | Design rationale, lessons carried over from DataScribe |
| `src/analyst_copilot/agents/` | ReAct agent(s) and deep-agent wiring |
| `src/analyst_copilot/graph/` | The LangGraph `StateGraph` definition, routing, `Send` fan-out |
| `src/analyst_copilot/memory/` | Checkpointer (short-term) and `Store` (long-term) setup |
| `src/analyst_copilot/tools/` | SQL tools, schema-reflection tool, custom tools |
| `data/` | Reuses `../data/ani.db` — no duplicate copy is kept |

## Status

Documentation and scaffolding only (Phase 0 — see the roadmap). No agent code has been
written yet.
