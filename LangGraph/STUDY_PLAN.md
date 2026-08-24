# LangChain and LangGraph study plan

Read this file instead of running the website. It is the same syllabus as the app: official docs first, then a sample in `samples/python`, then an exercise.

LangChain 1.x entry point is `create_agent`. LangGraph is the graph runtime (`StateGraph`, checkpointers, interrupts). Skip older tutorials that still center on `AgentExecutor`, `ConversationBufferMemory`, or `LLMChain`.

**How to study each lesson**

1. Open the official page (add `.md` to the docs URL if you want a clean source).
2. Run the listed sample, if there is one.
3. Do the exercise in your own file.
4. Check the box only after the exercise works.

Most graph samples need no API key. `04_create_agent.py` uses official `create_agent` when `OPENAI_API_KEY` or `ANTHROPIC_API_KEY` is set; otherwise it rehearses the same loop on LangGraph.

```bash
cd samples/python
python -m venv .venv
# Windows: .venv\Scripts\Activate.ps1
source .venv/bin/activate
pip install -r requirements.txt
python 00_check_install.py
```

---

## Contents

- [Stack](#stack)
- [Keep these open](#keep-these-open)
- [Phase 0 — Orientation](#phase-0--orientation)
- [Phase 1 — LangChain primitives](#phase-1--langchain-primitives)
- [Phase 2 — Agent capabilities](#phase-2--agent-capabilities)
- [Phase 3 — LangGraph core](#phase-3--langgraph-core)
- [Phase 4 — Official tutorials](#phase-4--official-tutorials)
- [Phase 5 — Observe, test, deploy](#phase-5--observe-test-deploy)

---

## Stack

| Product | Role |
| --- | --- |
| [LangChain](https://docs.langchain.com/oss/python/langchain/overview) | Agent framework: models, tools, messages, `create_agent` |
| [LangGraph](https://docs.langchain.com/oss/python/langgraph/overview) | Orchestration: graphs, persistence, interrupts, streaming |
| [Deep Agents](https://docs.langchain.com/oss/python/deepagents/overview) | Batteries-included harness on LangChain agents (later, optional) |
| [LangSmith](https://docs.langchain.com/langsmith/observability) | Trace, evaluate, and debug runs |

## Keep these open

- [LangChain Python docs (overview)](https://docs.langchain.com/oss/python/langchain/overview)
- [LangGraph Python docs (overview)](https://docs.langchain.com/oss/python/langgraph/overview)
- [Learn: tutorials index](https://docs.langchain.com/oss/python/learn)
- [langchain-ai/langgraph-101](https://github.com/langchain-ai/langgraph-101)
- [langchain-ai/langchain-academy](https://github.com/langchain-ai/langchain-academy)
- [LangChain Academy: Intro to LangGraph](https://academy.langchain.com/courses/intro-to-langgraph)
- [Use these docs (MCP)](https://docs.langchain.com/use-these-docs)

---

## Phase 0 — Orientation

Map the stack before you install anything extra.

LangChain 1.x is an agent harness. LangGraph is the low-level graph runtime those agents sit on. Mixing old chain tutorials with new agent docs is the fastest way to get lost.

### LangChain vs LangGraph vs Deep Agents

- **Time:** 45–60 min · **Track:** both
- **Summary:** Learn which product to reach for: `create_agent` for a customizable loop, LangGraph for explicit control flow, Deep Agents for a full harness.

**You should be able to**

- Explain Agent = Model + Harness in your own words.
- Know when to stay on `create_agent` vs drop down to `StateGraph`.

**Study steps**

1. Read the LangChain overview, including the comparison callout.
2. Read the LangGraph overview, especially the note to learn models and tools first.
3. Skim frameworks, runtimes, and harnesses if you want the product map.

**Official docs**

- [LangChain overview](https://docs.langchain.com/oss/python/langchain/overview)
- [LangGraph overview](https://docs.langchain.com/oss/python/langgraph/overview)
- [LangChain philosophy](https://docs.langchain.com/oss/python/langchain/philosophy)
- [Component architecture](https://docs.langchain.com/oss/python/langchain/component-architecture)

**Exercise:** Write a 6-sentence note: one sentence each for LangChain, LangGraph, Deep Agents, LangSmith, when you would use `create_agent`, and when you would write a custom graph.

- [ ] Done

### Install and environment

- **Time:** 30–45 min · **Track:** both
- **Sample:** `samples/python/00_check_install.py`

**You should be able to**

- Have a dedicated virtualenv with `langchain` and `langgraph`.
- Know which env vars each provider expects.

**Study steps**

1. Follow Install LangChain, then Install LangGraph.
2. Create `samples/python/.venv` and `pip install -r requirements.txt`.
3. Copy `.env.example` to `.env` only if you will call a live model.

**Official docs**

- [Install LangChain](https://docs.langchain.com/oss/python/langchain/install)
- [Install LangGraph](https://docs.langchain.com/oss/python/langgraph/install)
- [Chat model integrations](https://docs.langchain.com/oss/python/integrations/chat)

**Exercise:** Run `00_check_install.py` and confirm both packages import.

- [ ] Done

### How to study the official docs

- **Time:** 20 min · **Track:** both

**You should be able to**

- Bookmark the Learn page and both Python package indexes.
- Know that `create_agent` is the current agent entry point.

**Study steps**

1. Open the Learn page and note which tutorials are labeled LangChain vs LangGraph vs multi-agent.
2. Optionally install the LangChain Docs MCP server from Use these docs.

**Official docs**

- [Learn](https://docs.langchain.com/oss/python/learn)
- [LangChain docs index (llms.txt)](https://docs.langchain.com/oss/python/langchain/llms.txt)
- [LangGraph docs index (llms.txt)](https://docs.langchain.com/oss/python/langgraph/llms.txt)
- [Use these docs](https://docs.langchain.com/use-these-docs)

**Exercise:** List the four official LangChain tutorials on the Learn page and the two LangGraph counterparts. You will complete them in Phase 4.

- [ ] Done

---

## Phase 1 — LangChain primitives

Models, messages, tools, then your first agent.

LangGraph docs tell you to learn models and tools first. `create_agent` is a thin harness around those primitives.

### Models

- **Time:** 1–1.5 hr · **Sample:** `01_models_fake.py`

**Official docs:** [Models](https://docs.langchain.com/oss/python/langchain/models)

**Study steps**

1. Read Models: identifiers, `init_chat_model`, streaming mention.
2. Run `01_models_fake.py`, then retry with a live identifier if you have a key.

**Exercise:** Write a 10-line script that asks a model (fake or live) two questions in one conversation and prints both replies.

- [ ] Done

### Messages

- **Time:** 45–60 min · **Sample:** `02_messages.py`

**Official docs:** [Messages](https://docs.langchain.com/oss/python/langchain/messages)

**Study steps**

1. Read Messages. Pay attention to role strings vs class constructors.
2. Run `02_messages.py` and inspect the printed types.

**Exercise:** Build a four-message transcript (system, human, AI-with-tool-call, tool-result) using the official message classes.

- [ ] Done

### Tools

- **Time:** 1 hr · **Sample:** `03_tools.py`

**Official docs:** [Tools](https://docs.langchain.com/oss/python/langchain/tools)

**Study steps**

1. Read Tools: function tools, schema, and context.
2. Run `03_tools.py`.

**Exercise:** Add a second tool (for example `get_time`) and print both schemas. Do not wire an agent yet.

- [ ] Done

### Quickstart: create_agent

- **Time:** 1–1.5 hr · **Sample:** `04_create_agent.py`

**Official docs**

- [LangChain quickstart](https://docs.langchain.com/oss/python/langchain/quickstart)
- [Agents](https://docs.langchain.com/oss/python/langchain/agents)

**Study steps**

1. Read the LangChain quickstart in full.
2. Skim Agents for the same `create_agent` knobs.
3. Run `04_create_agent.py` (offline path). If you have a key, set `MODEL` and rerun.

**Exercise:** Replace `get_weather` with a tool that looks up a hardcoded inventory of three SKUs. Ask the agent for one SKU.

- [ ] Done

---

## Phase 2 — Agent capabilities

Streaming, memory, human-in-the-loop, middleware, retrieval.

Study these in LangChain first. LangGraph reuses the same ideas at a lower level.

### Streaming

- **Time:** 1 hr · **Sample:** `05_streaming.py`

**Official docs**

- [Streaming](https://docs.langchain.com/oss/python/langchain/streaming)
- [Event streaming](https://docs.langchain.com/oss/python/langchain/event-streaming)

**Exercise:** Print only tool start/end events from a streamed run of the weather agent (live or the offline stand-in).

- [ ] Done

### Structured output

- **Time:** 45 min · **Sample:** `06_structured_output.py`

**Official docs:** [Structured output](https://docs.langchain.com/oss/python/langchain/structured-output)

**Exercise:** Define a Ticket schema (priority, owner, summary) and extract it from a fake support email string.

- [ ] Done

### Short-term and long-term memory

- **Time:** 1.5–2 hr · **Sample:** `07_short_term_memory.py`

**Official docs**

- [Short-term memory](https://docs.langchain.com/oss/python/langchain/short-term-memory)
- [Long-term memory](https://docs.langchain.com/oss/python/langchain/long-term-memory)

**Exercise:** Two invokes on the same thread: tell the agent your name, then ask what it is. Confirm the second reply uses the first turn.

- [ ] Done

### Human-in-the-loop, middleware, guardrails

- **Time:** 2 hr · **Sample:** `08_middleware_notes.py`

**Official docs**

- [Human-in-the-loop](https://docs.langchain.com/oss/python/langchain/human-in-the-loop)
- [Middleware overview](https://docs.langchain.com/oss/python/langchain/middleware/overview)
- [Prebuilt middleware](https://docs.langchain.com/oss/python/langchain/middleware/built-in)
- [Custom middleware](https://docs.langchain.com/oss/python/langchain/middleware/custom)
- [Guardrails](https://docs.langchain.com/oss/python/langchain/guardrails)

**Exercise:** From the prebuilt list, pick one middleware and write a 5-line usage snippet in a new file. Do not skip the official page.

- [ ] Done

### Retrieval, RAG, and MCP

- **Time:** 2 hr · **Sample:** `09_retrieval_toy.py`

**Official docs**

- [Retrieval](https://docs.langchain.com/oss/python/langchain/retrieval)
- [Model Context Protocol (MCP)](https://docs.langchain.com/oss/python/langchain/mcp)
- [Build a semantic search engine](https://docs.langchain.com/oss/python/langchain/knowledge-base)

**Exercise:** Index three local paragraphs with a dict lookup (no vector DB) and return the best match for a query. Replace this with the official tutorial in Phase 4.

- [ ] Done

---

## Phase 3 — LangGraph core

`StateGraph`, the Functional API, persistence, interrupts.

When you need loops, branches, durable state, or mixed deterministic/agentic steps, you write the graph yourself.

### Thinking in LangGraph

- **Time:** 1 hr

**Official docs**

- [Thinking in LangGraph](https://docs.langchain.com/oss/python/langgraph/thinking-in-langgraph)
- [Workflows and agents](https://docs.langchain.com/oss/python/langgraph/workflows-agents)
- [Choosing between Graph and Functional APIs](https://docs.langchain.com/oss/python/langgraph/choosing-apis)
- [Graph API overview](https://docs.langchain.com/oss/python/langgraph/graph-api)

**Exercise:** Sketch two designs for “research then write”: one linear workflow, one agent loop with a tools node.

- [ ] Done

### Hello world graph

- **Time:** 45 min · **Sample:** `10_hello_world.py`

**Official docs**

- [LangGraph overview (hello world)](https://docs.langchain.com/oss/python/langgraph/overview)
- [LangGraph quickstart](https://docs.langchain.com/oss/python/langgraph/quickstart)

**Exercise:** Add a second node that uppercases the AI text and wire START → mock_llm → shout → END.

- [ ] Done

### Use the Graph API

- **Time:** 3–4 hr · **Samples:** `11_sequence.py`, `12_branch.py`

**Official docs:** [Use the graph API](https://docs.langchain.com/oss/python/langgraph/use-graph-api)

**Exercise:** Build a loop: a node increments `n` until `n >= 3`, then END. Use a conditional edge, not a Python `for` around `invoke`.

- [ ] Done

### Functional API

- **Time:** 1.5 hr · **Sample:** `13_functional_api.py`

**Official docs**

- [Functional API overview](https://docs.langchain.com/oss/python/langgraph/functional-api)
- [Use the functional API](https://docs.langchain.com/oss/python/langgraph/use-functional-api)

**Exercise:** Rewrite your sequence sample as `@task` functions inside one `@entrypoint`. Invoke both and compare outputs.

- [ ] Done

### Persistence, checkpointers, stores

- **Time:** 2 hr · **Sample:** `14_checkpoint.py`

**Official docs**

- [Persistence](https://docs.langchain.com/oss/python/langgraph/persistence)
- [Checkpointers](https://docs.langchain.com/oss/python/langgraph/checkpointers)
- [Stores](https://docs.langchain.com/oss/python/langgraph/stores)
- [Fault tolerance](https://docs.langchain.com/oss/python/langgraph/fault-tolerance)

**Exercise:** Invoke twice with the same `thread_id` and once with a new id. Prove the new thread does not see the old messages.

- [ ] Done

### Interrupts, subgraphs, time travel

- **Time:** 2–3 hr · **Sample:** `15_interrupt.py`

**Official docs**

- [Interrupts](https://docs.langchain.com/oss/python/langgraph/interrupts)
- [Subgraphs](https://docs.langchain.com/oss/python/langgraph/use-subgraphs)
- [Use time-travel](https://docs.langchain.com/oss/python/langgraph/use-time-travel)

**Exercise:** Add an interrupt before a “send email” node. Resume with `Command(resume=True)` only after you print the draft.

- [ ] Done

### LangGraph streaming

- **Time:** 1 hr · **Sample:** `16_graph_streaming.py`

**Official docs**

- [Streaming](https://docs.langchain.com/oss/python/langgraph/streaming)
- [Event streaming](https://docs.langchain.com/oss/python/langgraph/event-streaming)

**Exercise:** Stream your sequence graph and print the node name for every update.

- [ ] Done

---

## Phase 4 — Official tutorials

Build the sample programs LangChain publishes, not lookalikes. Do the LangChain version, then the LangGraph version.

### Semantic search engine

- **Time:** 2–3 hr · **Sample (warmup):** `09_retrieval_toy.py`

**Official docs:** [Build a semantic search engine with LangChain](https://docs.langchain.com/oss/python/langchain/knowledge-base)

**Exercise:** Finish the official tutorial, then write three queries that should hit different chunks. Record misses.

- [ ] Done

### SQL agent (LangChain)

- **Time:** 2–3 hr

**Official docs:** [Build a SQL agent](https://docs.langchain.com/oss/python/langchain/sql-agent)

**Exercise:** Add one read-only question the tutorial does not ask. Refuse any destructive SQL in the tool.

- [ ] Done

### Agentic RAG (LangGraph)

- **Time:** 2–3 hr

**Official docs:** [Build a custom RAG agent with LangGraph](https://docs.langchain.com/oss/python/langgraph/agentic-rag)

**Exercise:** Add a conditional edge that skips retrieval when the question is a greeting. Prove it with two invokes.

- [ ] Done

### SQL agent (LangGraph)

- **Time:** 2–3 hr

**Official docs:** [Build a custom SQL agent](https://docs.langchain.com/oss/python/langgraph/sql-agent)

**Exercise:** List three nodes you got for free in `create_agent` that you had to write yourself here.

- [ ] Done

### Multi-agent patterns

- **Time:** 4–6 hr · **Sample:** `17_supervisor_toy.py`

**Official docs**

- [Multi-agent](https://docs.langchain.com/oss/python/langchain/multi-agent/index)
- [Subagents](https://docs.langchain.com/oss/python/langchain/multi-agent/subagents)
- [Build a personal assistant with subagents](https://docs.langchain.com/oss/python/langchain/multi-agent/subagents-personal-assistant)
- [Handoffs](https://docs.langchain.com/oss/python/langchain/multi-agent/handoffs)
- [Build customer support with handoffs](https://docs.langchain.com/oss/python/langchain/multi-agent/handoffs-customer-support)
- [Router](https://docs.langchain.com/oss/python/langchain/multi-agent/router)
- [Build a multi-source knowledge base with routing](https://docs.langchain.com/oss/python/langchain/multi-agent/router-knowledge-base)
- [Skills](https://docs.langchain.com/oss/python/langchain/multi-agent/skills)

**Exercise:** Using `17_supervisor_toy.py` as a shape, replace the fake specialists with two real `create_agent` instances once you have a model key.

- [ ] Done

---

## Phase 5 — Observe, test, deploy

A graph you cannot trace or test is a demo.

### LangSmith observability

- **Time:** 1 hr

**Official docs**

- [LangChain observability](https://docs.langchain.com/oss/python/langchain/observability)
- [LangGraph observability](https://docs.langchain.com/oss/python/langgraph/observability)
- [Tracing quickstart](https://docs.langchain.com/langsmith/trace-with-langchain)

**Exercise:** Trace `04_create_agent.py` or `10_hello_world.py`. Note the run URL in your study log.

- [ ] Done

### Test and evaluate

- **Time:** 2 hr · **Sample:** `test_graphs.py`

**Official docs**

- [Test (LangChain)](https://docs.langchain.com/oss/python/langchain/test/index)
- [Unit testing](https://docs.langchain.com/oss/python/langchain/test/unit-testing)
- [Agent Evals](https://docs.langchain.com/oss/python/langchain/test/evals)
- [Test (LangGraph)](https://docs.langchain.com/oss/python/langgraph/test)

**Exercise:** Add a pytest for your branch graph: given `topic='billing'`, the route field equals the specialist you expect.

- [ ] Done

### Studio, local server, deploy

- **Time:** 2–3 hr

**Official docs**

- [LangSmith Studio (LangGraph)](https://docs.langchain.com/oss/python/langgraph/studio)
- [Run a local server](https://docs.langchain.com/oss/python/langgraph/local-server)
- [Application structure](https://docs.langchain.com/oss/python/langgraph/application-structure)
- [LangGraph deployment](https://docs.langchain.com/oss/python/langgraph/deploy)
- [LangChain deployment](https://docs.langchain.com/oss/python/langchain/deploy)

**Exercise:** Point `langgraph.json` at `10_hello_world.py` (or a small graph module) and open Studio. Invoke `hi`.

- [ ] Done

### Official sample programs

- **Time:** 8–12 hr

**Official docs / repos**

- [LangChain Academy course](https://academy.langchain.com/courses/intro-to-langgraph)
- [langchain-ai/langchain-academy](https://github.com/langchain-ai/langchain-academy)
- [langchain-ai/langgraph-101](https://github.com/langchain-ai/langgraph-101)
- [LangChain Academy page in docs](https://docs.langchain.com/oss/python/langchain/academy)

**Study steps**

1. Clone langchain-academy and follow its README (modules 0–5).
2. Clone langgraph-101. Do 101 before 201.
3. Use the notebooks as labs *after* you have read the matching docs.

**Exercise:** After each Academy module, log: module name, one new concept, and the official doc URL that covers it.

- [ ] Done
