# Roadmap

Each phase should be independently runnable/demoable — no phase depends on a later
phase's code existing.

## Phase 0 — Documentation & scaffolding (this)
- [x] Problem statement, architecture, folder skeleton.

## Phase 1 — Schema reflection & long-term memory store
- [ ] SQLAlchemy-based schema reflection tool (tables, columns, types, FKs) — replaces
      DataScribe's LLM-guessed JSON schema discovery entirely.
- [ ] NetworkX graph builder over the reflected schema (reused idea from DataScribe,
      now fed real data instead of LLM output).
- [ ] Long-term `Store` wired up; schema graph cached in it, keyed by a schema hash so
      it's invalidated automatically if `ani.db`'s structure changes.

## Phase 2 — Single ReAct SQL agent + short-term memory + minimal Streamlit shell
- [ ] `create_react_agent` over `SQLDatabaseToolkit` tools, read-only connection.
- [ ] `SqliteSaver` checkpointer wired up, keyed by `thread_id`.
- [ ] Streamlit chat UI: single input box, message history, `st.cache_resource` around
      agent/DB/LLM construction (critical — without this, every rerun rebuilds
      everything from scratch).
- [ ] Demoable: ask direct factual questions, follow up in the same session.

## Phase 3 — Classification + planning + parallel fan-out
- [ ] `classify_input` node (reuses DataScribe's category set: DATABASE_QUERY /
      GREETING / CHITCHAT / FAREWELL, split DATABASE_QUERY further into simple vs
      compound).
- [ ] Planner node that splits a compound question into independent sub-questions.
- [ ] `Send`-based fan-out to N ReAct sub-agents; `sub_results` reducer; `synthesize`
      node to merge.
- [ ] Demoable: ask a compound question, observe parallel sub-agent traces in the
      Streamlit sidebar/log panel.

## Phase 4 — Deep agent for open-ended analysis/report requests
- [ ] Wire `deepagents` (planning tool, virtual file system, sub-agent spawner) as a
      subgraph node.
- [ ] Report output rendered in Streamlit (markdown, plus any generated chart
      references).
- [ ] Demoable: ask for a multi-step report; watch the plan, drafts, and final report.

## Phase 5 — Human-in-the-loop
- [ ] `interrupt()` before executing a plan that touches more than N tables or is
      flagged as expensive; resume via `Command(resume=...)` from a Streamlit
      confirm button.

## Phase 6 — Long-term memory: glossary + report recall
- [ ] Mid-conversation fact capture ("active user" style definitions) written to the
      Store, retrieved and injected into future prompts.
- [ ] Past report summaries stored and retrievable ("what did we find about Q3 last
      time").

## Phase 7 — Polish
- [ ] Streaming responses (`.stream()`) into the Streamlit UI.
- [ ] Defense-in-depth query checker rejecting non-`SELECT` statements.
- [ ] Basic tests: schema reflection correctness, reducer merge behavior, routing
      logic for the classifier.
- [ ] `.gitignore` covering `.env`, local SQLite memory DBs, `__pycache__`.
