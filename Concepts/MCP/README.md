
# Week 6 — MCP Systems, Context Engineering, and Autonomous Traders

## Overview

This week progresses from **MCP fundamentals** to building MCP servers, applying MCP to context engineering, and combining multiple MCP capabilities in the **Autonomous Traders** capstone.

### Core themes

* MCP architecture and transports
* Building custom MCP servers with FastMCP
* MCP server discovery and marketplaces
* Context engineering
* Long-term memory
* Web search with MCP
* Agentic RAG
* Progressive disclosure of tools
* Multi-agent orchestration
* Observability and tracing
* Autonomous trading systems
* Production-oriented agent engineering

> **Important:** The Autonomous Traders project is presented as an educational project and should not be used for actual financial decisions. 

---

# 1. MCP Architecture

MCP has three core components:

```text
Host
  │
  ├── MCP Client ─── MCP Server
  ├── MCP Client ─── MCP Server
  └── MCP Client ─── MCP Server
```

## Host

The **host** is the application containing the LLM or agent.

Examples:

* Claude Desktop
* Claude Code
* Custom agent platforms

The host creates an MCP client for each MCP server connection.

## MCP Client

The client runs inside the host and connects to an MCP server.

Generally:

> One MCP client connects to one MCP server.

## MCP Server

The server provides:

* Tools
* Resources
* Prompts

The two most important operations are:

```text
list tools
call tool
```

Tools are by far the most common MCP use case.

---

# 2. MCP Servers as API Wrappers

An MCP server frequently acts as a wrapper around another API.

```text
LLM
 ↓
MCP Tool Description
 ↓
MCP Server
 ↓
API
 ↓
Result
```

The server translates between:

* LLM-friendly tool descriptions
* Structured tool calls
* Actual APIs or local operations

This lets an LLM understand and invoke capabilities without knowing the underlying implementation.

---

# 3. MCP Transport Mechanisms

## STDIO

**STDIO** is the common transport for local MCP servers.

The MCP client:

1. Spawns the server process.
2. Sends requests through standard input.
3. Receives responses through standard output.

```text
MCP Client
    │
    │ STDIO
    ▼
Local MCP Server
```

## Streamable HTTP

**Streamable HTTP** is used for remote MCP servers.

```text
MCP Client
    │
    │ HTTP
    ▼
Remote MCP Server
```

It replaces the older **SSE** mechanism, although legacy SSE servers can still exist.

### Rule of thumb

| Server | Typical Transport |
| ------ | ----------------- |
| Local  | STDIO             |
| Remote | Streamable HTTP   |

Streamable HTTP can also be used locally.

---

# 4. Running Local MCP Servers

Local MCP server configuration specifies how the server should be launched.

## Python

```text
uvx
```

can run packaged Python programs.

## Node / JavaScript

```text
npx
```

can run packaged Node programs.

## Docker

Docker can also be used to run a local MCP server.

Conceptually:

```text
Local MCP Server
├── uvx → Python
├── npx → Node/JavaScript
└── Docker → Container
```

---

# 5. Why Build an MCP Server?

The lecture identifies several legitimate reasons.

## Share Reusable Tools

If you've created useful tools and want other people or applications to consume them, MCP provides a standardized interface.

## Enterprise Tool Libraries

MCP can provide a consistent integration mechanism when:

* Multiple teams build tools.
* Different programming languages are involved.
* Different technologies are used.
* Many tools need to be shared across an organization.

## Learn the Plumbing

Building an MCP server is also useful educationally because it exposes how:

```text
Host
 ↓
Client
 ↓
Server
 ↓
Tool
```

actually works.

---

# 6. When NOT to Use MCP

This is one of the strongest practical lessons from the lecture.

If a tool is written **only for your own agent**, you generally do not need MCP.

For example, with the OpenAI Agents SDK:

```text
Python function
      ↓
@function_tool
      ↓
Agent
```

is sufficient.

Introducing MCP would add:

* A separate process
* A process boundary
* STDIO communication
* Additional configuration
* More moving parts

### Practical rule

> If you're building a tool for yourself, use the tool directly.

MCP is especially valuable when you are **sharing or consuming tools across boundaries**. 

---

# 7. MCP Marketplaces

MCP directories allow developers to discover existing MCP servers.

Examples mentioned in the lecture include:

* Glamour
* Smithery
* Context Summon
* Tabular
* Google Sheets
* Firecrawl
* Fetch

A marketplace listing can provide:

* Server description
* Popularity/download information
* Configuration parameters
* Installation instructions
* Transport information

This makes MCP servers easier to discover and integrate.

---

# 8. Building the Accounts MCP Server

The course reuses:

```text
backend/accounts.py
```

from the Week 3 engineering project.

The module manages:

* Account holders
* Balances
* Holdings
* Transactions
* Trading strategies

The course then wraps this existing functionality in an MCP server.

```text
accounts.py
      ↓
Accounts MCP Server
      ↓
LLM-accessible tools
```

---

# 9. FastMCP

The server uses:

```python
from mcp.server.fastmcp import FastMCP
```

A server is created with:

```python
mcp = FastMCP("accounts server")
```

and runs with STDIO:

```python
mcp.run(transport="stdio")
```

This creates the basic MCP server.

The server listens for:

```text
list tools
call tool
```

requests.

---

# 10. Defining MCP Tools

The `get_balance` example wraps the existing account functionality.

Conceptually:

```text
get_balance(name)
        ↓
account.get(name).balance
```

The important part is the tool description.

An MCP tool provides:

* Tool name
* Natural-language description
* Arguments
* Argument descriptions
* Return information

The LLM can then understand the capability.

```text
Tool
+
Description
+
Parameters
 ↓
MCP Tool Schema
```

---

# 11. Accounts MCP Tools

The server exposes account operations including:

* `get balance`
* `get holdings`
* `buy shares`
* `sell shares`
* `change strategy`

Resources were also added for demonstration purposes.

---

# 12. Connecting the Custom Server

Because the server is local, it can be launched directly with:

```text
uv run
```

and the module name.

The MCP client receives the command as its STDIO configuration.

The agent can then:

1. Start the server.
2. Discover its tools.
3. Use those tools.
4. Receive results.
5. Continue reasoning.

---

# 13. Custom MCP Server in an Agent

The example asks the agent:

```text
My name is Ed.
My account is under the name Ed.
What's my balance and my holdings?
```

The agent discovers the appropriate tools and calls:

```text
get balance
get holdings
```

It then uses the results to formulate the response.

Tracing confirms the complete sequence.

---

# 14. Tracing MCP Calls

Traces provide visibility into:

```text
Agent
 ↓
Tool discovery
 ↓
Tool selection
 ↓
MCP call
 ↓
Tool result
 ↓
Agent response
```

This is particularly useful when debugging agent behavior.

Instead of simply seeing the final answer, you can determine:

* Which tools were available.
* Which tools were selected.
* What arguments were used.
* What the server returned.
* How the result affected the final answer.

---

# 15. MCP Deployment Patterns

The lecture highlights three main practical configurations.

## 1. Local MCP Server

```text
Agent
 ↓
Local MCP Server
 ↓
Local capability
```

Example:

* Filesystem

## 2. Local MCP Server + External API

```text
Agent
 ↓
Local MCP Server
 ↓
Remote API
```

Examples:

* Web search
* Stock data
* Other external services

## 3. Remote MCP Server

```text
Agent
 ↓
Network
 ↓
Remote MCP Server
 ↓
Service
```

Example:

* Atlassian Jira

---

# 16. Hosted / Managed MCP

The lecture also discusses **hosted MCP**, where infrastructure for MCP can be run by the provider.

The instructor recommends treating this as a separate option rather than the default.

One concern is ecosystem coupling.

For example, a provider-hosted MCP integration may tie the implementation to that provider's API or model ecosystem.

The lecture's practical preference is:

1. **STDIO**
2. **Streamable HTTP**
3. Treat hosted MCP as a specialized option.

---

# 17. Context Engineering

The next major topic is **context engineering**.

Context engineering is the process of selecting and organizing the information, tools, resources, and instructions provided to an LLM.

The goal is:

> Give the model the best possible context for the task.

---

# 18. Components of Context Engineering

Context can include:

* System instructions
* Conversation history
* Short-term memory
* Long-term memory
* Retrieved information
* RAG results
* Tools
* Tool results
* Structured outputs

These ideas overlap.

For example:

```text
RAG
 ↓
Retrieved information
 ↓
Context
```

and:

```text
Memory
 ↓
Retrieved information
 ↓
Context
```

Tools can also perform retrieval, producing **agentic RAG**.

---

# 19. Short-Term Memory

Short-term memory is essentially the conversation history.

If a conversation becomes very long, the application can summarize older messages while retaining recent messages.

Conceptually:

```text
Old conversation
      ↓
Summary
      +
Recent messages
      ↓
LLM Context
```

This reduces unnecessary context while preserving important information.

---

# 20. Long-Term Memory

Long-term memory stores information outside the current conversation.

Examples:

* Databases
* Files
* Knowledge graphs
* Vector databases

The agent can retrieve this information later.

---

# 21. Long-Term Memory MCP Server

The course uses an Anthropic reference MCP server for persistent memory.

The server stores information in:

```text
memory.json
```

The stored information includes entities such as:

* Ed
* AI Agents course
* MCP protocol

and relationships such as:

```text
Ed
 ↓ teaches
AI Agents Course
 ↓ covers
MCP Protocol
```

The traces show operations such as:

* `create entities`
* `create relations`
* Node searches

This demonstrates persistent memory across conversations.

---

# 22. Tavily Web Search

The course uses **Tavily** as an MCP-based web-search provider.

The API key is stored in:

```text
TBLY_API_KEY
```

The Tavily MCP server exposes capabilities such as:

* Search
* Extract
* Crawl
* Map
* Research

---

# 23. Tool Filtering

The lecture deliberately restricts the agent to only the tool it needs.

For example:

```text
Tavily MCP
├── search       ← allowed
├── extract
├── crawl
├── map
└── research
```

The agent receives only:

```text
search
```

This is achieved with a static tool filter.

### Why?

Because fewer tools can mean:

* Less context
* Fewer choices
* Less ambiguity
* More focused agent behavior

This is an example of context engineering.

---

# 24. Agentic RAG with Qdrant

The course introduces **Qdrant** as a vector store.

The MCP server provides:

```text
qdrant_find
qdrant_store
```

These allow the agent to:

* Store information
* Retrieve semantically relevant information

The vector database is created locally.

---

# 25. Research + Vector Memory

The first agent receives:

```text
Tavily
+
Qdrant
```

Its task is to:

1. Research NVIDIA.
2. Identify important facts.
3. Store the information in Qdrant.

A second agent receives:

```text
Qdrant
```

but does **not** receive Tavily.

It can therefore answer questions using its stored knowledge without accessing the internet.

---

# 26. Traditional RAG vs. Agentic RAG

## Traditional RAG

```text
Question
 ↓
Retriever
 ↓
Relevant documents
 ↓
LLM
```

## Agentic RAG

```text
Question
 ↓
Agent
 ↓
Retrieval tool
 ↓
Relevant information
 ↓
Agent
 ↓
Answer
```

The key difference is that the agent can decide when to use the retrieval capability.

---

# 27. Why MCP Makes RAG Easier

A complete RAG implementation may require:

* Vector database
* Embedding model
* Indexing
* Retrieval logic
* Storage
* Query logic

With an existing Qdrant MCP server, much of that infrastructure is packaged behind the MCP interface.

The agent developer mainly needs to know:

```text
Which server?
Which parameters?
Which tools?
```

---

# 28. Market-Data MCP

The course uses **Massive**, formerly Polygon.io, for market data.

An optional:

```text
MASSIVE_API_KEY
```

provides access to real market data.

Without the key, the project falls back to a simulated market.

This means the capstone can still run without a paid market-data account.

---

# 29. Progressive Disclosure

Massive demonstrates an important MCP design pattern.

Instead of exposing every possible API endpoint as a separate tool, the MCP server can provide higher-level capabilities that let the agent:

1. Search available APIs.
2. Discover relevant endpoints.
3. Read their descriptions.
4. Select an appropriate endpoint.
5. Query the data.

Conceptually:

```text
Agent
 ↓
Discover
 ↓
Explore
 ↓
Select
 ↓
Call
```

---

# 30. Why Progressive Disclosure?

A huge number of tools can pollute the model's context.

For example:

```text
50 tools
 ↓
50 descriptions
 ↓
Many possible choices
```

Progressive disclosure can instead provide:

```text
Small discovery interface
 ↓
Relevant capability
 ↓
Specific API
```

Potential benefits:

* Smaller context
* Fewer choices
* Less ambiguity
* More coherent behavior

However, there is no universal solution.

The appropriate design should be established experimentally.

---

# 31. Experimentation Is Essential

The lecture emphasizes that tool architecture is an empirical problem.

There is no universal answer to:

* How many tools should an agent receive?
* Should tools be direct?
* Should tools be discovered dynamically?
* How many agents should exist?
* Which capabilities belong together?

The recommended approach is:

```text
Hypothesis
 ↓
Experiment
 ↓
Measure
 ↓
Compare
 ↓
Iterate
```

---

# 32. Autonomous Traders Capstone

The capstone project is:

# Autonomous Traders

It builds a virtual trading floor of autonomous AI agents.

The project is specifically applied to:

> **Financial services**

It combines many of the concepts covered throughout the course.

---

# 33. Capstone MCP Stack

The system uses approximately:

* **6 MCP servers**
* **17 tools**

Capabilities include:

* Accounts
* Pushover
* Market data
* Fetch
* Tavily
* Persistent memory
* MCP resources

---

# 34. Four Trader Agents

The trading floor contains four trader agents.

Each trader can:

* Manage its account.
* Read its holdings.
* Modify its portfolio.
* Follow its strategy.
* Access market data.
* Send notifications.
* Call a researcher.

Conceptually:

```text
Trading Floor
│
├── Trader 1
├── Trader 2
├── Trader 3
└── Trader 4
```

---

# 35. Researcher Agents

Each trader has its own researcher agent.

A researcher can use:

* Fetch
* Tavily
* Persistent memory

Its purpose is to investigate stocks and maintain research knowledge.

```text
Trader
  │
  │ agent-as-tool
  ▼
Researcher
  ├── Fetch
  ├── Tavily
  └── Memory
```

---

# 36. Separate Memory Per Researcher

Each researcher receives its own memory database.

This prevents:

* Research contamination
* Knowledge leakage between traders
* Unwanted convergence of trading strategies

Conceptually:

```text
Trader A
 └── Researcher A
      └── Memory A

Trader B
 └── Researcher B
      └── Memory B
```

Each researcher can therefore develop its own research base.

---

# 37. Agent-as-a-Tool

The OpenAI Agents SDK allows the researcher to be exposed as a tool.

Conceptually:

```text
Trader Agent
      │
      │ calls
      ▼
Researcher Agent
```

The trader's LLM decides:

* Whether research is needed.
* What to research.
* When to invoke the researcher.

This is **LLM orchestration**.

---

# 38. LLM Orchestration vs. Code Orchestration

The project deliberately combines both.

## LLM Orchestration

Used when flexibility and autonomous decisions are valuable.

Example:

```text
Trader
 ↓
Should I research?
 ↓
What should I research?
 ↓
Call researcher
```

## Code Orchestration

Used when the process is predetermined.

Example:

```text
Python
 ↓
Trader 1
 ↓
Trader 2
 ↓
Trader 3
 ↓
Trader 4
```

The code controls when and how often the trading floor runs.

---

# 39. Why Use Code for the Trading Loop?

The four traders need to run:

* Repeatedly
* On a schedule
* With predictable timing

There is no need to ask an LLM to decide something that is already fixed.

Therefore:

> Use code for deterministic workflow; use LLMs where flexible decision-making is valuable.

---

# 40. Trader and Researcher Separation

The separation into trader and researcher agents is not simply based on human organizational roles.

The separation exists because they require:

* Different prompts
* Different missions
* Different tools
* Different context

This is an example of **context engineering**.

---

# 41. Parallel Trading

The trading floor uses:

```python
asyncio.gather(...)
```

to run the traders in parallel.

Conceptually:

```text
             Trading Floor
                  │
       ┌──────────┼──────────┐
       ↓          ↓          ↓
    Trader 1   Trader 2   Trader 3   Trader 4
       │          │          │          │
       └──────────┴──────────┴──────────┘
                  │
              Results
```

This avoids unnecessarily waiting for each trader to finish before starting the next one.

---

# 42. Trader Configuration

The project uses environment variables such as:

```text
RUN_EVERY_N_MINUTES
RUN_EVEN_WHEN_MARKET_CLOSED
USE_MANY_MODELS
```

The default interval for the trading loop is:

```text
60 minutes
```

and the market-closed behavior can be configured.

---

# 43. Initial Dashboard

The initial Gradio dashboard displayed information such as:

* Traders
* Portfolio performance
* Traces
* Holdings
* Transactions

The dashboard provides a way to inspect the system's behavior.

---

# 44. Performance Evaluation

Historical results were shown for the simulated trading system.

The lecture notes that one trader, Soros, increased from approximately:

```text
$10,000
```

to nearly:

```text
$30,000
```

However, the lecture also notes that broad market growth and semiconductor holdings likely contributed significantly to the performance.

Therefore:

> Portfolio growth alone does not establish that the agent's decisions caused the performance.

This illustrates why agent evaluation needs careful measurement and appropriate baselines.

---

# 45. Custom Tracing

The project implements custom tracing using:

```text
TracingProcessor
```

A custom:

```text
LogTracer
```

implements methods such as:

* `on_trace_start`
* `on_trace_end`
* `on_span_start`
* `on_span_end`

The events are persisted in a SQL-like database.

The trading floor registers the processor through:

```text
add_trace_processor
```

---

# 46. Adaptive Trading Strategies

The project contains prompts for:

* Researchers
* Traders
* Rebalancing

The rebalance prompt allows traders to modify or replace their strategies based on portfolio performance.

Conceptually:

```text
Strategy
 ↓
Trade
 ↓
Performance
 ↓
Evaluation
 ↓
Rebalance strategy
 ↓
Next cycle
```

This introduces an explicit feedback loop.

---

# 47. FastAPI Backend

The trading system exposes an HTTP API using **FastAPI**.

The API provides information such as:

* Holdings
* Trader details
* Market data
* Log messages

This separates the backend agent system from the front-end interface.

---

# 48. TypeScript Front End

A Vite/TypeScript front end is used for the dashboard.

The dashboard shows:

* Four traders
* Model assignments
* Returns
* Activity
* Recent trades
* Holdings
* Heat-map visualization

The front end communicates with the backend API.

---

# 49. Live Trading Decisions

The demonstration showed traders making autonomous portfolio decisions.

Examples included:

* Ray selling part of a VOO position and buying Microsoft and Apple.
* George selling AMD and Qualcomm and buying UAL and VRT.
* Warren buying Adobe and making changes involving Berkshire Hathaway.
* Kathy making no portfolio changes.

Pushover notifications and dashboard activity provide additional observability.

---

# 50. Tracing Autonomous Behavior

The traces reveal:

* MCP tool calls
* Researcher invocations
* Web searches
* Fetch operations
* Account operations
* Agent decisions

One example showed George making **58 tool calls** and invoking the researcher twice.

This illustrates why traces are essential when evaluating autonomous systems.

---

# 51. Start With the Business Problem

One of the strongest engineering recommendations at the end of the course is:

> Start with the business problem, not with the idea of building an AI agent.

The process should be:

```text
Business problem
 ↓
Desired measurable outcome
 ↓
Baseline workflow
 ↓
Experiment
 ↓
Add autonomy where useful
 ↓
Measure improvement
```

---

# 52. Establish a Code-Orchestrated Baseline

Before introducing significant LLM autonomy:

1. Build a deterministic workflow.
2. Establish a baseline.
3. Measure its performance.
4. Introduce agents.
5. Compare results.

This makes it possible to determine whether the added autonomy actually improves the system.

---

# 53. Bottom-Up Agent Development

The recommended progression is:

```text
1. One LLM call
        ↓
2. Improve prompt
        ↓
3. Add a tool
        ↓
4. Add another capability
        ↓
5. Add an agent if needed
        ↓
6. Add more complexity only when metrics justify it
```

Avoid starting with an elaborate multi-agent architecture before establishing that the simpler system is insufficient.

---

# 54. Model Selection

The course recommends:

1. Start with a strong model.
2. Improve prompts.
3. Improve context.
4. Improve reliability.
5. Measure performance.
6. Only then test smaller models for cost reduction.

This avoids optimizing cost before establishing whether the system actually works.

---

# 55. Observability Is Core Engineering

For agentic systems, observability is not an optional feature.

Useful observability includes:

* Traces
* Tool calls
* Tool arguments
* Tool results
* Agent decisions
* Latency
* Errors
* Business outcomes

A useful mental model is:

```text
Agent
 ↓
Observe
 ↓
Measure
 ↓
Evaluate
 ↓
Improve
```

---

# 56. Final Course Architecture

The entire course can be viewed as a progression:

```text
Raw LLM Calls
      ↓
Tools
      ↓
Agent Loop
      ↓
Agent Frameworks
      ↓
Multi-Agent Systems
      ↓
MCP
      ↓
Context Engineering
      ↓
Memory / RAG
      ↓
Orchestration
      ↓
Observability
      ↓
Measured Autonomous Systems
```

---

# 57. Most Important Lessons

## MCP

* MCP is a protocol, not an agent framework.
* Its primary practical use is tool integration.
* The architecture is:

  * Host
  * Client
  * Server
* Local servers commonly use STDIO.
* Remote servers use Streamable HTTP.

## MCP Server Development

* FastMCP makes server construction straightforward.
* Existing application logic can be wrapped in MCP tools.
* Tool descriptions are critical for LLM usability.
* Don't use MCP unnecessarily for tools that only your own agent needs.

## Context Engineering

* Context is much broader than a system prompt.
* It includes memory, RAG, tools, tool results, instructions, and structured outputs.
* Tool filtering can reduce unnecessary context.
* Progressive disclosure can reduce tool-selection complexity.

## RAG

* Traditional RAG retrieves information through an explicit retrieval pipeline.
* Agentic RAG gives the agent retrieval tools.
* MCP can make sophisticated retrieval systems easier to integrate.

## Multi-Agent Systems

* Separate agents when different tasks require genuinely different:

  * Prompts
  * Missions
  * Tools
  * Context
* Do not create agents simply because human organizations have similar roles.

## Orchestration

* Use LLM orchestration where autonomous decision-making is valuable.
* Use code orchestration where the workflow is deterministic.

## Engineering

* Start with the business problem.
* Establish measurable outcomes.
* Build a simple baseline.
* Add complexity only when metrics justify it.
* Use strong models first.
* Optimize cost after reliability is established.
* Treat traces and observability as core development tools.

---

# 58. Interview / Exam Cheat Sheet

### What are the three MCP components?

```text
Host → Client → Server
```

### What does an MCP server primarily do?

```text
List tools
Call tools
```

### Common local transport?

```text
STDIO
```

### Remote transport?

```text
Streamable HTTP
```

### What is MCP?

> A protocol for standardizing how applications discover and interact with external tools and other context providers.

### When shouldn't you use MCP?

> When you're simply adding your own function directly to your own agent and there is no sharing or integration boundary that MCP solves.

### What is context engineering?

> Designing and organizing the information, tools, memory, retrieval results, instructions, and outputs supplied to an LLM so it has the context needed to perform effectively.

### What is agentic RAG?

> RAG where an agent has retrieval capabilities as tools and can decide when and how to retrieve information.

### What is progressive disclosure?

> Allowing an agent to discover and retrieve the specific capabilities or information it needs rather than loading every possible tool and detail into its context at once.

### When should code orchestration be preferred?

> When the workflow is deterministic and predictable.

### When is LLM orchestration useful?

> When flexible, autonomous decision-making is valuable.

### What should you do before building a complex agent architecture?

> Establish the business problem, measurable outcome, and a simpler baseline.

---

# References

* [Model Context Protocol](https://modelcontextprotocol.io/)
* [OpenAI Agents SDK](https://openai.github.io/openai-agents-python/)
* [FastMCP](https://github.com/modelcontextprotocol/python-sdk)
* [Qdrant](https://qdrant.tech/)
* [Tavily](https://tavily.com/)
* [FastAPI](https://fastapi.tiangolo.com/)
* [Playwright](https://playwright.dev/)
* [Context7](https://context7.com/)
* [Anthropic](https://www.anthropic.com/)
* [OpenAI](https://openai.com/)

**Source:** uploaded Week 6 MCP / Autonomous Traders lecture material. 

# Week 6 Days 2–4 — MCP: Building Servers, Context Engineering, and Autonomous Traders

## Overview

This part of Week 6 moves from **using MCP servers** to:

1. Building your own MCP server.
2. Understanding MCP server marketplaces.
3. Applying MCP to context engineering.
4. Using MCP for long-term memory, web search, and RAG.
5. Exploring progressive disclosure of tools.
6. Building the **Autonomous Traders** capstone.

The central lesson is that **MCP is primarily a standardized way to share and integrate tools**, while the actual capabilities come from the tools and services behind those MCP servers. 

---

# 1. MCP Recap

MCP has three core components:

```text
Host
  │
  ├── MCP Client ─── MCP Server
  ├── MCP Client ─── MCP Server
  └── MCP Client ─── MCP Server
```

## Host

The **host** is the application containing the LLM or agent.

Examples:

* Claude Desktop
* Claude Code
* ChatGPT
* Your own agent platform

The host creates an MCP client for each MCP server it wants to use.

## MCP Client

The client runs inside the host and connects to an MCP server.

Generally:

> One MCP client connects to one MCP server.

## MCP Server

The MCP server provides:

* Tools
* Resources
* Prompts

The two most important operations are:

```text
list tools
call tool
```

Resources and prompts exist, but the lecture emphasizes that **tools are by far the most common use case**. 

---

# 2. MCP Servers Often Wrap APIs

An important mental model is that an MCP server can act as a wrapper around an existing API.

```text
LLM
 ↓
MCP Tool Description
 ↓
MCP Server
 ↓
API
 ↓
Result
```

The MCP server provides an LLM-friendly description of the capability.

For example:

* The LLM understands a `fetch` tool.
* The MCP server receives the request.
* The server performs the underlying web request.
* The result is returned to the agent.

This makes an existing API easier for an LLM to discover and use. 

---

# 3. Local vs. Remote MCP Servers

There are several ways MCP servers can be deployed.

## Local MCP Server

The MCP server runs directly on your computer.

Typical transport:

```text
STDIO
```

The MCP client launches the server as a separate process.

## Local Server + Remote API

The MCP server runs locally but calls an external API.

For example:

```text
Agent
 ↓
Local MCP Server
 ↓
Internet API
 ↓
Result
```

## Remote MCP Server

The MCP server itself runs remotely.

Typical transport:

```text
Streamable HTTP
```

The client connects to the remote endpoint rather than spawning a local process.

---

# 4. STDIO, `uvx`, `npx`, and Docker

For local MCP servers, the client needs parameters describing how to start the server.

### Python

```text
uvx
```

is commonly used for Python-based MCP servers.

### Node / JavaScript

```text
npx
```

is commonly used for Node-based MCP servers.

### Docker

A Docker container can also contain and run an MCP server.

The basic idea is:

```text
MCP Client
    │
    ├── uvx → Python MCP server
    ├── npx → Node MCP server
    └── Docker → Containerized MCP server
```

The important point is that a local MCP server is often simply another program running locally and communicating with the MCP client over STDIO. 

---

# 5. Why Build Your Own MCP Server?

The lecture gives three legitimate reasons.

## 1. Share Your Tools

If you've built useful tools and want other people to use them easily, you can package them as an MCP server.

Other agents can then connect to them without having to understand your internal implementation.

## 2. Enterprise Tool Libraries

An enterprise may have:

* Many tools
* Multiple teams
* Multiple programming languages
* Multiple technologies

Publishing these capabilities as MCP servers can provide a consistent integration mechanism.

## 3. Educational Purposes

Building an MCP server is useful for understanding the underlying plumbing.

That is the primary reason for the course exercise. 

---

# 6. When NOT to Build an MCP Server

This is one of the most important lessons from the lecture.

If you're building a tool **for your own agent**, MCP is usually unnecessary.

For example, with the OpenAI Agents SDK, you can simply define a Python function, document it, and turn it into a tool.

Conceptually:

```text
Your Python function
        ↓
Tool decorator
        ↓
Agent
```

You do not need:

```text
Your function
   ↓
Separate process
   ↓
MCP server
   ↓
MCP client
   ↓
Agent
```

The latter introduces additional:

* Processes
* Boundaries
* Configuration
* Communication
* Failure modes

### Key Rule

> **If you're building a tool for yourself, use the tool directly.**

MCP becomes particularly valuable when you want to **share or consume tools across boundaries**. 

---

# 7. MCP Server Marketplaces

The lecture introduces MCP marketplaces/directories where developers can discover existing servers.

Examples mentioned include:

* Glamour
* Smithery

These directories allow developers to:

1. Browse MCP servers.
2. Search by capability.
3. Examine popularity/downloads.
4. Inspect configuration parameters.
5. Connect the server to an agent.

For example, a marketplace entry can tell you that an MCP server requires:

```text
npx
some-package
```

and therefore show you how to configure the MCP client.



---

# 8. Building an MCP Server with FastMCP

The course builds an **Accounts MCP Server**.

The existing account-management logic comes from the engineering project developed earlier in Week 3.

The idea is:

```text
Existing accounts.py
        ↓
Accounts MCP Server
        ↓
LLM-accessible tools
```

This demonstrates an important pattern:

> Existing application functionality can be wrapped in an MCP interface.

---

# 9. FastMCP

The lecture uses:

```python
from mcp.server.fastmcp import FastMCP
```

A server is created conceptually as:

```python
mcp = FastMCP("accounts server")
```

and run using STDIO.

The server then listens for MCP requests such as:

```text
list tools
call tool
```

The lecture compares the convenience of FastMCP to the convenience provided by frameworks such as FastAPI, while emphasizing that FastMCP is specifically for MCP servers. 

---

# 10. Creating an MCP Tool

The Accounts server wraps functions from the existing `accounts` module.

For example, a `get_balance` tool:

```text
get_balance(name)
```

can call:

```text
account.get(name).balance
```

The critical part is not the delegation itself.

The important part is the **natural-language description** attached to the function.

The MCP server uses this information to produce a tool description that an LLM can understand.

Conceptually:

```text
Tool name
   +
Natural-language description
   +
Arguments
   ↓
MCP tool schema
```

The MCP specification then provides the mechanism for exposing that schema to an MCP client. 

---

# 11. Accounts MCP Server Tools

The example exposes several account-related capabilities:

* Get balance
* Get holdings
* Buy shares
* Sell shares
* Change strategy

The agent can therefore interact with the account through MCP rather than directly importing and calling the account functions.

---

# 12. Calling the Custom MCP Server

The locally developed server can be launched directly with a command such as:

```text
uv run accountserver
```

The command becomes part of the MCP client configuration.

The OpenAI Agents SDK can then:

1. Launch the server.
2. Connect through STDIO.
3. Ask for available tools.
4. Provide those tools to the agent.
5. Allow the agent to call them.

The tool listing confirms that the custom server exposes the expected account functionality. 

---

# 13. Custom MCP Server in Action

The agent is given instructions to manage a client's account.

Example request:

```text
My name is Ed.
My account is under the name Ed.
What's my balance and my holdings?
```

The agent discovers the appropriate MCP tools and calls:

```text
get_balance
get_holdings
```

It then uses the returned information to construct its response.

---

# 14. Why Tracing Matters

The lecture repeatedly emphasizes inspecting traces.

A trace shows:

1. Which tools were available.
2. What the agent received as its task.
3. Which tools the agent selected.
4. What arguments it passed.
5. What the tools returned.
6. How those results contributed to the final response.

For an MCP system, tracing makes the architecture much less mysterious.

You can literally see:

```text
Agent
 ↓
MCP tools discovered
 ↓
Tool selected
 ↓
MCP server called
 ↓
Result returned
 ↓
Agent response
```

---

# 15. MCP Server Architecture — Four Configurations

The lecture describes several configurations.

## Configuration 1 — Local MCP Server

```text
Agent
 ↓
Local MCP Server
 ↓
Local capability
```

Example:

* Filesystem

## Configuration 2 — Local MCP Server + Remote API

```text
Agent
 ↓
Local MCP Server
 ↓
Remote API
```

Examples:

* Web search
* Stock data
* Other APIs

## Configuration 3 — Remote MCP Server

```text
Agent
 ↓
Internet
 ↓
Remote MCP Server
 ↓
Service
```

An example is a hosted Jira/Atlassian MCP server.

## Configuration 4 — Hosted / Managed MCP

The lecture also discusses hosted MCP tools where the provider runs MCP-related infrastructure as part of its own API infrastructure.

The instructor recommends treating this as a separate option and being conscious of potential ecosystem/vendor coupling.

---

# 16. MCP Transport Options

The lecture revisits the major transport choices:

| Transport       | Typical Use                             |
| --------------- | --------------------------------------- |
| STDIO           | Local MCP servers                       |
| Streamable HTTP | Local or remote HTTP-accessible servers |
| SSE             | Legacy transport                        |

The lecture's practical emphasis is on:

```text
STDIO
```

and:

```text
Streamable HTTP
```

---

# 17. Context Engineering

The next major topic is **context engineering**.

Context engineering is the set of decisions made to ensure that an LLM receives the best possible information, tools, and resources for the task.

It expands beyond traditional prompt engineering.

---

# 18. Components of Context Engineering

The lecture identifies several overlapping components.

## Instructions

The system prompt/instructions need to provide useful guidance.

## Short-Term Memory

Short-term memory is essentially the conversation history.

Instead of always sending the entire conversation, older content can potentially be summarized while keeping recent messages in detail.

Example:

```text
Old conversation
      ↓
Summary
      +
Recent messages
      ↓
LLM context
```

## Long-Term Memory

Long-term memory allows an agent to access information outside the current conversation.

This may involve:

* Databases
* Files
* Knowledge stores

## RAG

Retrieval-Augmented Generation uses retrieval to bring relevant information into the model's context.

Traditional RAG can use semantic/vector search.

## Tools

Tool descriptions are themselves part of the agent's context.

Tool results also become part of the context after the tools are called.

## Structured Outputs

Structured outputs provide information in a predictable format that can be used for further processing.

---

# 19. Agentic RAG

The lecture connects tools and RAG through the concept of **agentic RAG**.

Traditional RAG:

```text
Question
 ↓
Retriever
 ↓
Relevant information
 ↓
LLM
```

Agentic RAG:

```text
Question
 ↓
Agent
 ↓
Retrieval tool
 ↓
Relevant information
 ↓
Agent
 ↓
Answer
```

The agent itself can decide when and how to retrieve information.

---

# 20. Long-Term Memory with MCP

The lecture demonstrates long-term memory using an MCP server.

The memory server stores information in:

```text
memory.json
```

The agent can:

1. Store information.
2. End the conversation.
3. Start another conversation.
4. Query the same memory.
5. Retrieve previously stored information.

This demonstrates the distinction between:

```text
Short-term memory
= conversation history
```

and:

```text
Long-term memory
= persistent external information
```



---

# 21. Knowledge Graph Memory

The memory example stores information as entities and relationships.

Conceptually:

```text
        Ed
        │
      teaches
        ↓
 AI Agents Course
        │
      covers
        ↓
 MCP Protocol
```

The traces show operations such as:

* Creating entities
* Creating relationships
* Searching nodes

This provides a graph-like representation of persistent information.

---

# 22. Web Search with Tavily MCP

The next context-engineering example uses **Tavily**.

The MCP server provides web-search capabilities to the agent.

The lecture configures an API key through:

```text
TBLY_API_KEY
```

The MCP server exposes multiple capabilities, including:

* Search
* Extract
* Crawl
* Map
* Research

---

# 23. Tool Filtering

The lecture deliberately gives the agent only the tool it needs.

For example:

```text
Tavily Search
```

rather than exposing every Tavily capability.

The OpenAI Agents SDK supports a static tool filter.

Conceptually:

```text
Tavily MCP Server
       │
       ├── search      ← allowed
       ├── extract
       ├── crawl
       ├── map
       └── research
```

The agent receives only:

```text
search
```

---

# 24. Why Tool Filtering Matters

Giving an agent fewer tools can reduce unnecessary choices and context.

Instead of:

```text
Many tools
 ↓
Many possible decisions
```

you provide:

```text
Only relevant tools
 ↓
Smaller decision space
```

This is another example of **context engineering**.

The goal is not to give the agent every capability imaginable.

The goal is to give it the capabilities needed for the task.

---

# 25. Qdrant and Agentic RAG

The course next introduces **Qdrant** as a local vector database.

The MCP server provides two important tools:

```text
qdrant_find
qdrant_store
```

These allow an agent to:

* Store information.
* Retrieve semantically relevant information later.

The server can run locally and create its database under the project's memory directory.

---

# 26. Researcher + Knowledge Base

The lecture combines Tavily and Qdrant.

The first agent has access to:

```text
Tavily
+
Qdrant
```

Its task is to:

1. Research a topic on the web.
2. Extract important facts.
3. Store them in the vector database.

The demonstration researches NVIDIA and stores relevant information.

---

# 27. Retrieval Without Internet Access

A second agent is given only Qdrant.

It has:

```text
Qdrant
```

but:

```text
No Tavily
No Internet
```

It is then asked about the previously researched topic.

The agent retrieves relevant information from the vector store.

This demonstrates the separation between:

```text
Research
```

and:

```text
Knowledge retrieval
```

---

# 28. Traditional RAG vs. Agentic RAG

### Traditional RAG

The application explicitly controls retrieval.

```text
Question
 ↓
Vector search
 ↓
Retrieved documents
 ↓
LLM
```

### Agentic RAG

The agent has retrieval tools and can decide when to use them.

```text
Question
 ↓
Agent
 ↓
Qdrant tool
 ↓
Relevant information
 ↓
Agent
```

The MCP server makes the underlying retrieval capability easy to attach to an agent.

---

# 29. The MCP Advantage for RAG

Building a complete RAG system manually can require:

* Vector database
* Embedding/encoder model
* Indexing
* Retrieval logic
* Storage
* Query logic

With an existing Qdrant MCP server, the agent can be equipped with the capability using server configuration parameters.

The lecture's broader point is:

> MCP can make sophisticated external capabilities accessible without requiring the agent developer to implement all of the underlying infrastructure.



---

# 30. Progressive Disclosure of Tools

The Massive market-data MCP server demonstrates an important technique.

Instead of exposing every possible API endpoint directly, the server can give the agent higher-level tools for:

1. Exploring available APIs.
2. Finding relevant endpoint documentation.
3. Selecting the appropriate API.
4. Calling it.

Conceptually:

```text
Agent
 ↓
Explore available capabilities
 ↓
Find relevant API
 ↓
Understand endpoint
 ↓
Call endpoint
 ↓
Return result
```

---

# 31. Why Progressive Disclosure Helps

Giving an agent too many tools can pollute its context and increase the number of choices it has to make.

Instead of:

```text
50 tools
 ↓
Many possible choices
```

progressive disclosure can provide:

```text
Small set of discovery tools
 ↓
Explore only when necessary
 ↓
Relevant capability
```

This can reduce context clutter and potentially improve coherence.

---

# 32. There Is No Universal Tool Architecture

The lecture emphasizes that tool organization is an **empirical problem**.

There is no universal rule saying:

> Always expose tools individually.

or:

> Always use progressive disclosure.

The appropriate structure depends on:

* Task
* Model
* Number of tools
* Tool descriptions
* Complexity
* Desired behavior

The recommended approach is experimentation with representative evaluation data.

---

# 33. Autonomous Traders — Capstone

The final project is called:

# Autonomous Traders

The project builds a trading floor containing autonomous AI agents that operate in a virtual market.

The project combines many concepts from the course.

---

# 34. Six MCP Servers / 17 Tools

The project uses approximately:

* **6 MCP servers**
* **17 tools**

Some of the capabilities include:

* Accounts
* Push notifications
* Market data
* Fetch
* Tavily
* Memory / knowledge graph
* MCP resources

This brings together many of the MCP concepts introduced during the week.

---

# 35. Market Data

Market data can come from:

### Real Market Data

The project can use Massive when an API key is available.

### Simulated Market

If the API key is unavailable, the project can use a simulated market.

This makes the project accessible without requiring real market-data credentials.

---

# 36. Trading Floor Architecture

The trading floor contains four trader agents.

```text
                 Trading Floor
                      │
        ┌─────────────┼─────────────┐
        │             │             │
     Trader 1      Trader 2      Trader 3 ... Trader 4
        │             │             │
        └──────── Researcher ───────┘
```

Each trader:

* Manages an account.
* Reads its portfolio.
* Makes buy/sell decisions.
* Has a trading strategy.
* Can send notifications.
* Has access to market data.
* Can collaborate with a researcher agent.

---

# 37. Trader Agent

Each trader has access to the Accounts MCP server.

This allows it to:

* Read account information.
* Read holdings.
* Buy shares.
* Sell shares.
* Understand its strategy.
* Track performance.

The trader can also use the Pushover MCP server for notifications.

---

# 38. Researcher Agent

Each trader has its own researcher.

The researcher can use:

* Fetch
* Tavily
* Memory

The researcher can therefore:

1. Search the web.
2. Investigate a stock.
3. Build a persistent knowledge base.
4. Provide information back to the trader.

---

# 39. Agent-as-a-Tool

The project uses the OpenAI Agents SDK's **agent-as-tool** functionality.

The trader can call the researcher as a tool.

Conceptually:

```text
Trader Agent
     │
     │ calls
     ▼
Researcher Agent
     │
     ├── Fetch
     ├── Tavily
     └── Memory
```

This creates collaboration between agents while allowing the trader's LLM to decide when research is needed.

---

# 40. Two Types of Orchestration

The capstone combines:

## LLM Orchestration

The trader's LLM decides:

* Whether to call the researcher.
* What research request to make.
* How to use the research.

This provides autonomy.

## Code Orchestration

Python controls:

* When traders run.
* Which traders run.
* The sequence of execution.
* Repeated trading cycles.

This provides more deterministic behavior.

---

# 41. Why Combine Both?

The lecture revisits the earlier distinction:

| Approach           | Strength                        |
| ------------------ | ------------------------------- |
| LLM orchestration  | Flexible, autonomous decisions  |
| Code orchestration | Predictable, reliable workflows |

The project combines them intentionally.

For example:

```text
Python
 ↓
Run Trader 1
 ↓
Run Trader 2
 ↓
Run Trader 3
 ↓
Run Trader 4
```

But inside each trader:

```text
Trader LLM
 ↓
Decide whether research is needed
 ↓
Call Researcher
 ↓
Use research
 ↓
Make decision
```

---

# 42. Agent Roles Are Based on Context Engineering

The lecture makes an important distinction about the names **trader** and **researcher**.

The agents were not separated simply because:

> "Humans have traders and researchers."

They were separated because the tasks require:

* Different instructions
* Different missions
* Different tools
* Different context

This makes the division an example of **context engineering** rather than merely copying a human organizational structure.

---

# 43. Experimentation Is Essential

The lecturer recommends experimenting with:

### One large agent

```text
One agent
+
Many tools
```

### Multiple specialized agents

```text
Trader
+
Researcher
+
Specialized tools
```

The correct architecture should be determined through experimentation.

Potential evaluation dimensions include:

* Stability
* Process adherence
* Decision quality
* Performance
* Measurable outcomes

---

# 44. Evaluation and Feedback

The capstone revisits a major theme from earlier weeks:

> Believable LLM output does not necessarily mean correct or effective output.

An agent may produce a convincing explanation for a trade without that trade actually being good.

Therefore, the system needs:

```text
Decision
 ↓
Outcome
 ↓
Measurement
 ↓
Feedback
 ↓
Improvement
```

The project provides measurable outcomes that can be used to evaluate the agentic system.

---

# 45. Important Financial-Safety Note from the Lecture

The instructor explicitly states that the Autonomous Traders project is for **educational purposes**.

The agents and project should **not be treated as a source of actual trading decisions**.

The project is intended to demonstrate agent architecture, MCP, orchestration, evaluation, and feedback in a financial-services scenario.

---

# 46. The Big Picture

The Week 6 progression can be summarized as:

```text
Use MCP servers
      ↓
Understand MCP architecture
      ↓
Build an MCP server
      ↓
Discover existing MCP servers
      ↓
Use MCP for context engineering
      ↓
Add persistent memory
      ↓
Add web search
      ↓
Add vector retrieval
      ↓
Use progressive disclosure
      ↓
Combine multiple MCP servers
      ↓
Build Autonomous Traders
```

---

# 47. Key Takeaways

## MCP

* MCP is a protocol, not an agent framework.
* It standardizes access to external capabilities.
* The core architecture is:

  * Host
  * Client
  * Server
* Tools are the dominant MCP use case.

## Building MCP Servers

* Use MCP when sharing/reusing tools makes sense.
* Don't introduce MCP unnecessarily for tools you directly control.
* FastMCP makes server creation relatively simple.
* Natural-language tool descriptions are critical.

## Context Engineering

* Context includes more than prompts.
* It includes:

  * Instructions
  * Conversation history
  * Long-term memory
  * RAG
  * Tools
  * Tool results
  * Structured outputs
* Tool selection itself is part of context engineering.

## Agentic RAG

* Agents can use retrieval tools directly.
* MCP makes vector stores and other retrieval systems easier to connect.
* Qdrant demonstrates this pattern.

## Progressive Disclosure

* Too many tools can clutter context.
* Discovery-oriented tools can allow agents to explore capabilities progressively.
* Tool architecture should be evaluated empirically.

## Autonomous Traders

* Combines multiple MCP servers.
* Combines LLM and code orchestration.
* Uses agent-as-tool collaboration.
* Separates trader and researcher based on context/tool requirements.
* Uses measurable outcomes for evaluation.
* Is intended as an educational project, not a real trading system.

---

# 48. Exam / Interview Mental Model

If asked **"What is MCP?"**:

> MCP is a protocol that standardizes how LLM applications discover and invoke external tools and other context providers.

If asked **"Why build an MCP server?"**:

> To share tools, standardize tool integration across teams or applications, or understand the MCP plumbing.

If asked **"When should I not use MCP?"**:

> When you're simply adding your own function directly to your own agent and there is no sharing or integration boundary that MCP solves.

If asked **"What are the three MCP components?"**:

```text
Host → Client → Server
```

If asked **"What is the common local transport?"**:

```text
STDIO
```

If asked **"What is the common remote transport?"**:

```text
Streamable HTTP
```

If asked **"What is agentic RAG?"**:

> RAG where an agent has retrieval capabilities as tools and can decide when and how to retrieve information.

If asked **"What is progressive disclosure?"**:

> Giving an agent a manageable set of discovery capabilities so it can progressively retrieve the specific tools or API information it needs instead of loading every capability into its context at once.

---

# References

* Model Context Protocol
* OpenAI Agents SDK
* MCP Python SDK / FastMCP
* Qdrant
* Tavily
* Massive
* Playwright
* Context7
* Anthropic MCP documentation

