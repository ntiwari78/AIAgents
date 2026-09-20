Below is a cleaned-up **GitHub Markdown study guide** based on the uploaded Week 6 MCP lecture, with the original technical terminology and structure preserved. 

# Week 6 — Model Context Protocol (MCP)

## Overview

This week introduces **Model Context Protocol (MCP)** after reviewing agent frameworks and agent harnesses.

### Critical Outcomes

* MCP is a **standard for integrating primarily third-party tools** into agents, not an agent framework.
* MCP architecture consists of:

  * A **host**
  * **Clients** inside the host
  * Separate **servers** providing tools
* Local MCP servers typically use **STDIO**.
* Remote MCP servers use **Streamable HTTP**.
* The **OpenAI Agents SDK** can create MCP clients and equip an agent with:

  * Fetch
  * Playwright
  * Filesystem tools

---

# 1. Agent Definitions, Harnesses, and Framework Trade-offs

## What Is an AI Agent?

An AI agent can be thought of as:

> An LLM equipped with tools that loops until it achieves a goal.

## What Is an Agent Harness?

An **agent harness** is the collection of:

* Tools
* Control loop
* Goal machinery
* Supporting capabilities

that turn an LLM into an agent.

In Week 1, the course built a harness directly using:

* LLM calls
* Tools
* A `while` loop

Agent frameworks provide varying levels of abstraction and reduce the amount of harness code developers need to write.

---

## Agent Framework Continuum

The lecture positioned frameworks on a subjective continuum.

### Lightweight Frameworks

* Direct LLM calls from Week 1
* LangChain core components such as `ChatOpenAI`
* Custom tools and loops
* LangGraph
* Strands Agent

LangGraph still requires developers to describe:

* Orchestration
* Dependencies
* Workflows
* Execution

Strands Agent was described as one of the lightest frameworks encountered in the course.

### Middle of the Continuum

* OpenAI Agents SDK
* Google SDK
* Agno
* Mastra
* Pydantic
* LangChain `create_agent`

### Higher-Level Frameworks

* CrewAI

  * More opinionated
  * More complex
  * More "batteries included"
* LangChain Deep Agents

  * Tools
  * Memory
  * Agent loop
  * Other capabilities around a high-level goal

### Claude Agent SDK

Claude Agent SDK was positioned as an outlier rather than a conventional agent framework.

It provides programmatic control of **Claude Code** through:

* Python
* TypeScript

It uses the `query` function and iterates over Claude Code responses.

Built-in capabilities include:

* File reading
* Command execution
* Web search
* Code editing

It is tied specifically to Claude and does not allow model substitution.

---

## Framework Trade-off

Higher-level frameworks reduce implementation effort but can make failures harder to debug because developers surrender some control to the framework.

The appropriate level of abstraction depends on:

* Project maturity
* Desired control
* Team skills

---

# 2. MCP as a Universal Tool-Integration Standard

## What Is MCP?

**Model Context Protocol (MCP)** is:

* A protocol
* A specification
* An agreed standard

It is **not**:

* An agent framework
* An agent-coding method
* A fundamental change to the agent loop

MCP standardizes how the following can be exposed to an agent:

* Tools
* Resources
* Prompts

In practice, adoption is overwhelmingly centered on **tools**. Resources are sometimes used, while prompts are rarely encountered in practice.

---

## The USB-C Analogy

Anthropic compares MCP to a **USB-C port for AI applications**.

The idea is to provide a common, plug-and-play connector for tools from different providers.

Before MCP, tool ecosystems were often closely tied to particular frameworks.

For example, LangChain tools were easiest to reuse inside LangChain-based applications.

MCP allows tools written by different organizations to be consumed by compatible hosts.

---

## MCP vs. the Underlying Tool

MCP's value should be separated from the value of the actual tool.

For example:

* **Playwright** provides browser-automation capabilities.
* **MCP** provides a standardized way to describe and expose those capabilities to an LLM.

Custom functions can already be made into tools relatively easily in frameworks such as the OpenAI Agents SDK.

Therefore, MCP is particularly useful for integrating **someone else's tools**.

---

## MCP Ecosystem

MCP's popularity has created a large ecosystem, but also a "wild west" of tools and marketplaces that developers need to evaluate.

The lecture notes that:

* Anthropic created MCP.
* MCP was later transferred to the **Agentic AI Foundation** under the **Linux Foundation**.

---

# 3. MCP Host, Client, and Server Architecture

MCP uses three important terms.

## MCP Host

The **host** is the application containing the LLM or agent that needs additional tools.

Examples include:

* ChatGPT
* Claude AI
* Claude Desktop
* Claude Code
* A Jupyter notebook running an agent
* A custom agent product

## MCP Client

The **client** is code running inside the host that connects the application to an MCP server.

There is generally **one client per connected server**.

## MCP Server

The **server** is a separate process or remote service containing the:

* Tools
* Resources
* Prompts

supplied by a third party.

---

## The Two Core MCP Operations

The client mediates two core operations:

### `list tools`

Returns JSON descriptions containing:

* Available tools
* Tool purposes
* Parameters
* Input schemas

### `call tool`

Invokes a selected tool with supplied inputs and returns its result.

---

## Frameworks and MCP Clients

Agent frameworks usually create and manage MCP clients automatically.

The developer specifies which servers they want, while the framework:

1. Creates or launches the required client.
2. Connects the client to the server.
3. Makes the server's tools available to the agent.

---

# 4. MCP Deployment Patterns

The lecture covered three deployment patterns.

## 1. Local Server Using Local Capabilities

A server runs on the developer's computer and operates on local resources.

Example:

* Filesystem MCP server

## 2. Local Server Wrapping a Remote API

The server runs locally but calls an external web API.

Examples:

* Weather
* Stock prices
* Webpage retrieval

## 3. Remote Server

The MCP server runs on a third-party computer and is accessed over a network.

Example:

* Atlassian's remote server for Jira customers

### Important Point

A local MCP server does **not** necessarily contain the underlying service.

For example, a local weather server might:

1. Describe the weather API to the LLM.
2. Receive a tool call.
3. Make the actual web request.
4. Return the result.

---

# 5. Tool Descriptions and MCP's Main Contribution

MCP servers provide **LLM-readable JSON** describing:

* Available tools
* Tool purpose
* Arguments
* Parameter types
* Natural-language explanations
* When and how parameters should be used

This description layer is a major part of MCP's practical value.

Tool authors can create effective descriptions and schemas so that LLMs are more likely to use tools correctly.

---

## MCP as an Adapter

An MCP server often acts as an adapter between natural-language tool use and an existing API.

The flow is:

```text
LLM
 ↓
MCP Tool Schema
 ↓
Structured Tool Call
 ↓
MCP Server
 ↓
API / Local Operation
 ↓
Result
 ↓
Agent
```

More explicitly:

1. The MCP server exposes a tool schema understandable by an LLM.
2. The agent requests a tool call with structured arguments.
3. The MCP server translates the request into an API call or local operation.
4. The result is returned to the agent.

Examples include:

* Anthropic Fetch server
* Weather servers
* Stock-price servers
* Microsoft Playwright server
* Anthropic filesystem server

---

# 6. MCP Transports

Two transport mechanisms were covered.

## STDIO

**STDIO** means standard input/output.

The MCP client:

1. Launches a local process.
2. Sends requests through standard input.
3. Reads responses from standard output.

STDIO is common for locally executed:

* Python servers
* JavaScript/TypeScript servers
* Docker-based servers

---

## Streamable HTTP

**Streamable HTTP** allows the client to connect to an MCP server over HTTP.

It:

* Supports streaming responses.
* Is used for remote servers.
* Replaces the older SSE approach.

Legacy SSE servers may still exist.

---

## Transport Rule of Thumb

| Server            | Common Transport |
| ----------------- | ---------------- |
| Local MCP server  | STDIO            |
| Remote MCP server | Streamable HTTP  |

A local server can technically use Streamable HTTP, but STDIO is generally the common choice.

A remote server requires a network transport because the client cannot spawn a process on another computer.

---

# 7. MCP Server Configuration

Server configuration parameters describe how to launch or reach an MCP server.

### Python Server

Example configuration concept:

```text
command: uvx
arguments: mcp-server-fetch
```

### JavaScript/TypeScript Server

Example:

```text
command: npx
arguments: playwright-mcp@latest
```

### Remote Server

Example configuration includes:

```text
URL
timeout
```

as demonstrated with Context7.

---

## `uvx`

`uvx` runs a Python package in an isolated environment.

It is a shortcut for:

```text
uv tool run
```

## `npx`

`npx` provides the analogous experience for Node-based packages.

Docker containers are another possible way to run MCP servers.

---

# 8. MCP Timeouts

A **60-second timeout** is recommended for STDIO examples.

Why?

The first invocation of:

* `uvx`
* `npx`

may need to download and install a package.

A default **5-second timeout** can therefore fail during initial setup.

---

# 9. Lab: Fetch, Playwright, and Filesystem MCP Servers

The lab returned to the **OpenAI Agents SDK** and demonstrated MCP in code.

## Windows Support

Windows support for local STDIO MCP servers has stabilized, so WSL is no longer required.

The notebook included a workaround to prevent server output to standard error from crashing Windows notebooks.

---

# 10. Fetch MCP Server

The Fetch MCP server was configured with:

```text
uvx mcp-server-fetch
```

The server exposed one tool:

```text
fetch
```

The tool can:

* Retrieve a URL
* Optionally extract its contents as Markdown

---

## Fetch Tool Description

Fetch included an expert-written tool description explaining that the tool provides internet access.

This is important because it can overcome an LLM's default tendency to claim that it cannot browse the internet.

The server also returned a JSON input schema describing the:

```text
url
```

parameter.

---

# 11. Playwright MCP

Playwright setup required Node and Playwright installation for learners who had skipped earlier course material.

### Node Requirement

The lecture recommended:

```text
Node 22+
```

A Playwright check generated a:

```text
Playwright check PNG
```

containing the Hacker News front page.

The Playwright MCP server was launched using:

```text
npx playwright-mcp@latest
```

---

## Playwright MCP Tools

The server exposed browser-related tools including:

* Close browser
* Resize browser
* Upload files
* Fill forms
* Navigate
* Take snapshots
* Click
* Drag
* Hover

This demonstrates how MCP can expose sophisticated browser automation to an agent without the agent developer implementing each browser operation manually.

---

# 12. Filesystem MCP Server

The filesystem server was configured using `npx -y` and restricted to a:

```text
sandbox
```

directory.

Its tools included:

* Read files
* Read multiple files
* List directories
* Create directories
* Search files

The sandbox restriction limits filesystem access to the intended working area.

---

# 13. Understanding the Lab Architecture

In the notebook example:

| Component                        | Role                    |
| -------------------------------- | ----------------------- |
| Jupyter notebook / Python kernel | MCP host                |
| OpenAI Agents SDK                | MCP client              |
| `uvx` process                    | MCP server              |
| `npx` process                    | MCP server              |
| STDIO                            | Communication transport |

The OpenAI Agents SDK created the MCP clients and connected to the local MCP servers over STDIO.

---

# 14. End-to-End Agent Demonstration

An OpenAI Agents SDK agent was equipped with two MCP servers:

### Filesystem MCP

Used for:

* Reading files
* Writing files
* Working within `sandbox`

### Playwright MCP

Used for:

* Browsing the internet
* Interacting with websites

---

## Agent Instructions

The agent was instructed to:

* Browse persistently.
* Handle cookies and pop-ups.
* Try alternate websites when needed.
* Write files only inside `sandbox`.

---

## Task

The task was:

> Find a good banoffee pie recipe and summarize it in `banoffee.md`.

The agent used browser tools to visit websites, including:

```text
bbcgoodfood.com
```

It then wrote a Markdown recipe to the local sandbox.

### Key Lesson

A relatively small amount of configuration can give an agent:

* Internet browsing
* Browser automation
* Local file reading
* Local file writing

without manually implementing all of those capabilities.

---

# 15. Remote MCP with Context7

A second demonstration used **Context7** through **Streamable HTTP**.

The question concerned the `manifest` object in the OpenAI Agents SDK's sandbox agents feature.

---

## Why Context7 Was Needed

The lecture demonstrated a situation where:

* GPT-4o-mini lacked the relevant information because its training data was older.
* Context7 could retrieve current documentation.

The remote Context7 MCP server was configured with:

* A remote URL
* A timeout

---

## Context7 Workflow

The agent first used:

```text
resolve library ID
```

to identify the relevant OpenAI Agents SDK documentation.

It then used:

```text
query docs
```

to retrieve information about the `manifest` object.

The retrieved documentation enabled the same model to answer accurately rather than guess.

---

# 16. Why MCP Matters

The lecture's central distinction is:

```text
Agent Framework
        ≠
MCP
```

An agent framework provides machinery for building and running agents.

MCP provides a standardized way for agents and applications to interact with external capabilities.

A simplified architecture is:

```text
                 ┌─────────────────────┐
                 │      MCP Host       │
                 │                     │
                 │  LLM / Agent        │
                 │        │            │
                 │   MCP Clients       │
                 └───────┬─┬───────────┘
                         │ │
                  STDIO  │ │  HTTP
                         │ │
                ┌────────┘ └─────────┐
                ▼                    ▼
        ┌──────────────┐     ┌──────────────┐
        │ Local Server │     │Remote Server │
        │              │     │              │
        │ Filesystem   │     │ Context7     │
        │ Playwright   │     │ Jira         │
        │ Fetch        │     │ Other APIs   │
        └──────────────┘     └──────────────┘
```

---

# 17. Key Concepts to Remember

## Agent

An LLM equipped with tools that loops until it achieves a goal.

## Agent Harness

The supporting machinery around the LLM:

* Tools
* Loop
* Goal
* Control mechanisms

## MCP

A protocol/standard for exposing:

* Tools
* Resources
* Prompts

to compatible hosts.

## Host

The application containing the LLM or agent.

## Client

Code inside the host that connects to an MCP server.

## Server

A local process or remote service that exposes MCP capabilities.

## Tool Schema

LLM-readable JSON describing:

* What a tool does
* Its inputs
* Its parameters
* How and when to use them

## STDIO

The common transport for local MCP servers.

## Streamable HTTP

The transport used for remote MCP servers.

## `uvx`

Runs Python packages in isolated environments.

## `npx`

Runs Node-based packages.

---

# 18. The Most Important Mental Model

Think about MCP as a **standardized plug system**:

```text
                  MCP
                   │
        ┌──────────┼──────────┐
        │          │          │
    Playwright   Fetch    Filesystem
        │          │          │
      Browser    Web/API     Files
```

The underlying tools provide the actual functionality.

MCP provides a standardized interface through which compatible hosts can discover and invoke that functionality.

---

# 19. Practical MCP Flow

A typical MCP interaction looks like:

```text
1. Host starts
       ↓
2. MCP client connects to server
       ↓
3. Client requests available tools
       ↓
4. Server returns tool schemas
       ↓
5. LLM sees available tools
       ↓
6. LLM selects a tool
       ↓
7. Client sends call_tool request
       ↓
8. Server performs the operation
       ↓
9. Server returns result
       ↓
10. Agent continues its loop
```

---

# 20. Final Takeaways

1. **MCP is a protocol, not an agent framework.**
2. MCP primarily standardizes how external tools are exposed to LLM applications.
3. The three fundamental MCP components are:

   * Host
   * Client
   * Server
4. Local MCP servers commonly communicate through **STDIO**.
5. Remote MCP servers use **Streamable HTTP**.
6. MCP servers expose **LLM-readable tool schemas**.
7. Tool descriptions are important because they help LLMs understand when and how to use tools.
8. MCP can act as an adapter between LLM-friendly tool calls and existing APIs.
9. The OpenAI Agents SDK can connect agents to multiple MCP servers.
10. Fetch provides web retrieval.
11. Playwright provides browser automation.
12. Filesystem MCP provides controlled file operations.
13. Context7 demonstrates how remote MCP can provide up-to-date documentation to an agent.
14. Higher-level agent frameworks trade implementation control for convenience.
15. Inspecting **agent traces** is important for understanding which MCP tools an agent discovers and invokes.

---

# References

* [Model Context Protocol](https://modelcontextprotocol.io/)
* [MCP Specification](https://modelcontextprotocol.io/specification/latest)
* [OpenAI Agents SDK](https://openai.github.io/openai-agents-python/)
* [Playwright](https://playwright.dev/)
* [Playwright MCP](https://github.com/microsoft/playwright-mcp)
* [LangGraph](https://www.langchain.com/langgraph)
* [LangChain](https://www.langchain.com/)
* [Claude Agent SDK](https://platform.claude.com/docs/en/agent-sdk/overview)
* [Context7](https://context7.com/)
* [Anthropic MCP documentation](https://docs.anthropic.com/en/docs/agents-and-tools/mcp)

**Source:** uploaded Week 6 MCP lecture. 

# Week 6 — Model Context Protocol (MCP)

## 1. Week 6 Overview

Week 6 is dedicated to **MCP — Model Context Protocol**.

The week begins by revisiting:

* What an AI agent is
* What an agent framework provides
* The concept of an **agent harness**
* The spectrum from low-level LLM calls to high-level agent harnesses

Then it introduces:

* MCP as a standard/protocol
* MCP hosts, clients, and servers
* MCP tools
* Local vs. remote MCP servers
* STDIO and Streamable HTTP transports
* MCP server configuration
* Using MCP with the OpenAI Agents SDK
* Fetch, Playwright, File System, and Context7 MCP servers

---

# 2. What Is an AI Agent?

The course's core definition remains:

> **An AI agent is an LLM equipped with tools that loops to achieve a goal.**

Another way to express the same idea is:

```text
LLM + Harness = Agent
```

The **harness** contains the additional machinery around the LLM, including:

* Tools
* Looping
* Goal-oriented execution
* Supporting state and orchestration

---

# 3. What Is an Agent Harness?

An **agent harness** is the collection of code that turns a basic LLM call into an agent.

You can build a harness yourself.

For example:

```text
LLM
 ↓
Give it tools
 ↓
Ask it what to do
 ↓
Execute tool
 ↓
Give result back to LLM
 ↓
Repeat
```

This is essentially what was demonstrated in Week 1 using direct LLM calls.

Agent frameworks provide abstractions that make this easier.

---

# 4. Agent Frameworks Exist on a Spectrum

The instructor describes agent frameworks as existing on a continuum.

```text
Low-level                                      High-level
──────────────────────────────────────────────────────────>

Raw LLM
  ↓
LangChain Core
  ↓
LangGraph
  ↓
Strands
  ↓
OpenAI Agents SDK / Google ADK / Agno / Mastra /
Pydantic AI / LangChain create_agent
  ↓
CrewAI
  ↓
LangChain Deep Agents
  ↓
Claude Agent SDK
```

This positioning is subjective rather than an objective ranking.

---

# 5. Low-Level vs. High-Level Frameworks

## Lower-level frameworks

Characteristics:

* More control
* More code written by the developer
* More responsibility for orchestration
* Easier to understand what is happening internally

Examples discussed:

* Raw LLM calls
* LangChain Core
* LangGraph
* Strands

## Higher-level frameworks

Characteristics:

* More functionality provided automatically
* Less code required
* More batteries included
* Potentially harder to debug when something goes wrong

Examples discussed:

* CrewAI
* LangChain Deep Agents
* Claude Agent SDK

---

# 6. Control vs. Convenience

A key trade-off is:

```text
More abstraction
      ↓
Less developer work
      ↓
Less direct control
      ↓
Potentially harder debugging
```

Whereas:

```text
Less abstraction
      ↓
More developer work
      ↓
More control
      ↓
Potentially easier debugging
```

The instructor's personal preference leans toward the lower-level side, but the appropriate choice depends on:

* Project maturity
* Team skills
* Desired control
* Complexity
* Development speed

---

# 7. Claude Agent SDK

The lecture briefly introduces the **Claude Agent SDK**.

It is described as somewhat different from the other agent frameworks.

The SDK provides programmatic control over Claude Code using:

* Python
* TypeScript

A key function is:

```python
query()
```

The SDK allows Claude Code-style capabilities to be driven programmatically, including:

* Reading files
* Running commands
* Searching the web
* Editing code

The instructor describes it as an "odd one out" because it is tightly coupled to Claude rather than being a general model-agnostic agent abstraction.

---

# 8. MCP Is Not an Agent Framework

This is one of the most important distinctions.

**MCP is not an agent framework.**

It does not:

* Create agents
* Define agent loops
* Replace an agent framework
* Provide an agent architecture

Instead, MCP is a:

> **Protocol / standard / specification**

It defines an agreed way for software to expose and consume capabilities.

---

# 9. What Does MCP Stand For?

**MCP = Model Context Protocol**

The important word is:

**Protocol**

A protocol is an agreed way for different systems to communicate.

The goal is interoperability.

---

# 10. What Does MCP Standardize?

The lecture describes MCP as a standard way to integrate:

* **Tools**
* **Resources**
* **Prompts**

However, the instructor emphasizes that:

> **Tools are by far the most important and widely used part of MCP in practice.**

The lecture therefore focuses primarily on MCP tools.

---

# 11. The USB-C Analogy

Anthropic describes MCP as:

> **A USB-C port for AI applications**

The analogy is:

```text
USB-C
    ↓
Standard connector
    ↓
Many different devices

MCP
    ↓
Standard interface
    ↓
Many different tools
```

The value comes from having a common standard.

A tool provider can build an MCP server, and different AI applications can connect to it without each integration requiring a completely custom interface.

---

# 12. MCP vs. Custom Tools

You can already turn your own function into a tool.

For example, an agent framework might allow:

```python
@function_tool
def get_weather(city):
    ...
```

You do not need MCP to do this.

MCP becomes especially valuable when:

> **Someone else has built the tool and you want to use it.**

For example:

* Microsoft's Playwright tools
* Anthropic's Fetch server
* File-system tools
* Context7 documentation tools
* Third-party SaaS integrations

---

# 13. What MCP Actually Adds

Suppose Microsoft creates sophisticated browser automation functionality using Playwright.

The major functionality is:

```text
Playwright
```

MCP provides the standardized interface that makes that functionality easy for an LLM application to discover and use.

So keep this distinction clear:

```text
Tool functionality
        +
MCP standard interface
        =
Reusable agent capability
```

MCP itself is not the underlying browser automation technology.

---

# 14. The Three MCP Components

The three key terms are:

1. **MCP Host**
2. **MCP Client**
3. **MCP Server**

These terms can be confusing because they do not exactly correspond to the usual meanings of host, client, and server in general computing.

---

# 15. MCP Host

The **MCP host** is the application containing the LLM or agent that wants to use MCP tools.

Examples include:

* ChatGPT
* Claude Desktop
* Claude Code
* Your own agent application
* An agent framework running inside your application
* The Jupyter/Python process used in the lab

Think:

```text
MCP Host
=
The application running the agent/LLM
```

---

# 16. MCP Client

The **MCP client** is the component inside the host that communicates with an MCP server.

Conceptually:

```text
Host
 └── MCP Client
       └── MCP Server
```

There is generally:

> **One MCP client for each MCP server connection.**

When using an agent framework, you often do not implement the MCP client yourself.

The framework creates it for you.

---

# 17. MCP Server

The **MCP server** is the component that exposes capabilities through MCP.

It can provide:

* Tools
* Resources
* Prompts

In practice, the important part is usually:

```text
MCP Server
    ↓
List tools
    +
Call tools
```

Examples include:

* Playwright MCP
* Fetch MCP
* File System MCP
* Context7 MCP

---

# 18. The Basic Architecture

The relationship is:

```text
┌─────────────────────────────┐
│          MCP Host           │
│                             │
│       LLM / Agent           │
│            │                │
│       MCP Client            │
└────────────┼────────────────┘
             │
             │ MCP
             ↓
┌─────────────────────────────┐
│        MCP Server           │
│                             │
│    Tool descriptions        │
│    Tool implementations     │
└─────────────────────────────┘
```

The client is inside the host.

The server is the separately running component providing the capabilities.

---

# 19. MCP Server: Two Fundamental Operations

The lecture emphasizes two important MCP operations.

## 1. List tools

The client asks:

```text
What tools do you provide?
```

The server returns structured descriptions of those tools.

## 2. Call a tool

The client asks:

```text
Call tool X
with these arguments
```

The server executes it and returns the result.

So:

```text
MCP Server
   ├── List Tools
   └── Call Tool
```

---

# 20. Tool Descriptions Are Critical

MCP servers provide descriptions of tools in a format that an LLM can understand.

A tool description contains information such as:

* Tool name
* Purpose
* Parameters
* Parameter descriptions
* Input schema

Conceptually:

```json
{
  "name": "get_city_price",
  "description": "Find the price for a city",
  "parameters": {
    "city": {
      "description": "The city to search for"
    }
  }
}
```

This connects directly back to the structured tool schemas studied in Week 1.

---

# 21. MCP as a Natural-Language Interface to APIs

One of the most important explanations in the lecture is that an MCP server often acts as a bridge between:

```text
LLM-friendly descriptions
          ↕
       MCP Server
          ↕
      API / Service
```

For example:

```text
LLM
 ↓
"Get the weather for London"
 ↓
MCP tool
 ↓
Weather API
 ↓
Weather result
 ↓
LLM
```

The MCP server provides the standardized tool interface and often makes the actual API request underneath.

---

# 22. MCP Does Not Necessarily Contain the Actual Service

This distinction is important.

If you use a weather MCP server, the weather data does not necessarily come from code running locally.

Instead:

```text
Local MCP Server
       ↓
Weather API
       ↓
Internet
       ↓
Weather data
```

The local MCP server may simply provide:

* Tool descriptions
* Input schemas
* Code that calls the external API

---

# 23. Local MCP Servers

A very common architecture is:

```text
Your Computer

┌─────────────────────┐
│ MCP Host            │
│   ↓                 │
│ MCP Client          │
└─────────┬───────────┘
          │
          ↓
    MCP Server
          │
          ↓
    External API
```

The MCP server itself runs locally.

However, the underlying service it accesses may be remote.

---

# 24. Remote MCP Servers

Another architecture is:

```text
Your Computer
┌─────────────────────┐
│ MCP Host            │
│   ↓                 │
│ MCP Client          │
└─────────┬───────────┘
          │
       Internet
          │
          ↓
┌─────────────────────┐
│ Remote MCP Server   │
└─────────────────────┘
```

The MCP server itself runs on another computer.

An example discussed is a third-party service such as Jira/Atlassian providing a remote MCP endpoint.

---

# 25. Three MCP Deployment Patterns

The lecture essentially describes three patterns.

### Pattern 1 — Local MCP + local functionality

```text
Host
 ↓
Client
 ↓
Local MCP Server
 ↓
Local files/system
```

Example:

**File System MCP**

### Pattern 2 — Local MCP + remote API

```text
Host
 ↓
Client
 ↓
Local MCP Server
 ↓
Internet/API
```

Examples:

* Fetch
* Weather
* Stock-price tools

### Pattern 3 — Remote MCP Server

```text
Host
 ↓
Client
 ↓
HTTP
 ↓
Remote MCP Server
 ↓
Third-party service
```

Example:

* SaaS integrations such as Jira

---

# 26. MCP Transport Mechanisms

The transport mechanism determines how the MCP client communicates with the server.

The lecture focuses on two:

1. **STDIO**
2. **Streamable HTTP**

---

# 27. STDIO

**STDIO = Standard Input/Output**

It is the most common approach when running MCP servers locally.

Conceptually:

```text
MCP Client
    ↓
Launch process
    ↓
MCP Server
    ↓
stdin / stdout
```

The client launches another process and communicates with it through:

* Standard input
* Standard output

The MCP server might be:

* Python
* JavaScript/TypeScript
* Dockerized software

---

# 28. Streamable HTTP

The second transport is:

**Streamable HTTP**

It allows an MCP client to connect to an MCP server over HTTP.

Conceptually:

```text
MCP Client
    ↓
HTTP
    ↓
Remote MCP Server
```

This is the normal approach when the MCP server is remote.

---

# 29. STDIO vs. Streamable HTTP

A useful rule of thumb:

| Server            | Typical transport |
| ----------------- | ----------------- |
| Local MCP server  | STDIO             |
| Remote MCP server | Streamable HTTP   |

More precisely:

* Local servers can use STDIO or HTTP
* STDIO is usually simpler for local servers
* Remote servers generally require HTTP-based communication

The lecture also notes that older MCP implementations used **SSE (Server-Sent Events)**, but Streamable HTTP has replaced it for current use.

---

# 30. MCP Server Parameters

When connecting to an MCP server, you need to describe how to launch or reach it.

For a local STDIO server, parameters describe the process.

For example:

```python
{
    "command": "uvx",
    "args": ["mcp-server-fetch"]
}
```

For a remote server, parameters can look like:

```python
{
    "url": "...",
    "timeout": 60
}
```

These parameters are not magical.

They simply tell the MCP client:

> **How do I start or connect to this MCP server?**

---

# 31. `uvx` for Python MCP Servers

For Python-based MCP servers, the lecture uses:

```text
uvx
```

`uvx` provides a convenient way to run Python tools in an isolated environment.

Conceptually:

```text
uvx
 ↓
Install/run Python package
 ↓
Launch MCP server
```

This avoids manually managing all of the package installation and environment details.

---

# 32. `npx` for JavaScript/TypeScript MCP Servers

For JavaScript/TypeScript MCP servers, the equivalent approach is:

```text
npx
```

For example:

```text
npx @playwright/mcp@latest
```

Conceptually:

```text
npx
 ↓
Install/run Node package
 ↓
Launch MCP server
```

---

# 33. Docker as Another Option

The lecture also mentions a third possibility:

```text
Docker
```

An MCP server can be packaged into a Docker container and launched as the MCP server process.

The course does not focus on this approach, but it is another way MCP servers can be distributed.

---

# 34. MCP Lab #1

The first lab returns to the **OpenAI Agents SDK**.

The important point is that the MCP concepts are not tied to a single agent framework.

The same MCP architecture can be used with different frameworks.

---

# 35. Windows Setup

The lecture notes that MCP on Windows previously required more complicated setup involving WSL for stable STDIO operation.

The situation has improved, and direct Windows operation is now supported.

However, the notebook environment still requires a small workaround to prevent problems when MCP servers write to standard error.

---

# 36. Fetch MCP Server

The first practical MCP server is:

**Fetch**

It is used to retrieve web pages.

The server is launched using Python tooling:

```text
uvx
```

The conceptual configuration is:

```python
{
    "command": "uvx",
    "args": ["mcp-server-fetch"]
}
```

---

# 37. Fetch MCP: Listing Tools

The agent framework can ask the MCP server:

```text
List your tools.
```

The Fetch server provides a tool named:

```text
fetch
```

The tool can retrieve a URL and optionally extract its contents as Markdown.

---

# 38. Why Tool Descriptions Matter

The lecture highlights that the Fetch MCP server includes carefully designed descriptions.

For example, the tool description explains:

* What the tool does
* How the LLM should use it
* What inputs it requires

The quality of these descriptions matters because they influence whether an LLM correctly decides to use the tool.

---

# 39. MCP Tool Schema

The Fetch tool also exposes an input schema.

Conceptually:

```json
{
  "url": "URL to fetch"
}
```

The important idea is that the LLM receives both:

```text
What the tool does
+
How to call the tool
```

This allows it to decide when and how to use the capability.

---

# 40. Playwright MCP

The next example is:

**Playwright MCP**

It exposes Microsoft's Playwright browser automation capabilities through MCP.

The server is launched using Node tooling:

```text
npx
```

Conceptually:

```text
npx playwright MCP
```

---

# 41. Playwright MCP Tools

Playwright MCP exposes many browser-related tools, including capabilities for:

* Opening browsers
* Closing browsers
* Navigating
* Clicking
* Filling forms
* Uploading files
* Taking snapshots
* Hovering
* Dragging
* Resizing

The MCP server makes these browser capabilities available to an LLM through standardized tool descriptions.

---

# 42. MCP vs. Playwright

This distinction is important:

```text
Playwright
=
Browser automation technology

MCP
=
Standardized interface for exposing that capability to an LLM
```

The innovation in Playwright is browser automation.

The MCP layer provides the standardized LLM-facing interface.

---

# 43. File System MCP

The third local example is the **File System MCP server**.

It provides tools for interacting with a local sandbox.

Examples include:

* Read file
* Read multiple files
* List directory
* Create directory
* Search files

The server is restricted to the specified sandbox directory.

---

# 44. Identifying Host, Client, and Server in the Lab

This is an important exercise.

In the Jupyter notebook example:

### Host

The host is essentially:

```text
Jupyter notebook / Python kernel
```

More precisely, it is the process running the agent framework.

### Client

The MCP client is created by:

```text
OpenAI Agents SDK
```

### Server

The MCP server is the process launched by:

```text
uvx
```

or:

```text
npx
```

---

# 45. Why Developers Usually Don't Write MCP Clients

You can implement an MCP client yourself.

However, the lecture points out that modern agent frameworks already handle this.

Instead of manually writing code to:

1. Create an MCP client
2. Spawn a process
3. Establish communication
4. List tools
5. Call tools

You can simply configure the framework with the MCP server.

The framework handles the client layer.

---

# 46. Agent + Multiple MCP Servers

The lab combines two MCP servers:

```text
Agent
 ├── File System MCP
 └── Playwright MCP
```

The agent therefore gets:

* Internet/browser access
* Local file-system access

This is a powerful combination.

---

# 47. Banoffee Pie Example

The demonstration gives the agent a task:

> Find a great recipe for banoffee pie and save a Markdown summary as `banoffee.md`.

The agent receives:

### Playwright MCP

For browsing the web.

### File System MCP

For writing the result locally.

The workflow becomes:

```text
Agent
 ↓
Browse web
 ↓
Find recipe
 ↓
Read information
 ↓
Summarize recipe
 ↓
Write banoffee.md
```

---

# 48. Why This Feels Agentic

The agent can:

* Choose websites
* Navigate pages
* Accept cookies
* Read information
* Decide what to inspect
* Write a file
* Complete the task

The capabilities come from the tools.

MCP makes those external capabilities easy to attach to the agent.

---

# 49. Remote MCP: Context7

The final major example introduces a **remote MCP server**:

**Context7**

The use case is retrieving up-to-date library documentation.

The problem:

```text
Older LLM
 ↓
Question about a newer API
 ↓
Model does not know the latest API
```

MCP can provide access to current documentation.

---

# 50. Context7 Architecture

Instead of using a local STDIO server, the example uses:

```text
MCP Client
     ↓
Streamable HTTP
     ↓
Context7 MCP Server
     ↓
Current library documentation
```

The client is configured using:

* URL
* Timeout

---

# 51. Why Context7 Is Useful

Suppose an older model is asked about a recently introduced API.

Its training data may not contain that API.

Instead of relying on memory, the agent can:

1. Connect to Context7
2. Resolve the library
3. Search its documentation
4. Retrieve relevant information
5. Use the retrieved documentation to answer

---

# 52. Context7 Example Workflow

The demonstration asks about the `manifest` object in the OpenAI Agents SDK sandbox agents feature.

The agent:

```text
Question
   ↓
Context7
   ↓
Resolve library ID
   ↓
Find relevant documentation
   ↓
Query documentation
   ↓
Return current information
   ↓
LLM answers
```

This is an example of MCP being used for **current documentation retrieval**.

---

# 53. The Big Picture

The Week 6 mental model is:

```text
                     MCP Host
                        │
                      Agent
                        │
                   MCP Client(s)
                 ┌──────┼──────┐
                 ↓      ↓      ↓
              Server  Server  Server
                 │      │      │
              Tools   Tools   Tools
                 │      │      │
              APIs   Browser  Files
```

MCP provides the standardized connection layer.

---

# 54. MCP Does Not Replace Agent Frameworks

Keep the distinction clear:

| Technology      | Primary purpose                                |
| --------------- | ---------------------------------------------- |
| Agent framework | Build/run agents                               |
| Agent harness   | Provide tools + loop + goal-oriented execution |
| MCP             | Standardize tool/context connectivity          |
| MCP client      | Connect host to an MCP server                  |
| MCP server      | Expose tools/resources/prompts                 |
| Playwright      | Browser automation                             |
| Context7        | Documentation retrieval                        |
| Fetch           | Web-page retrieval                             |

MCP and agent frameworks solve different problems.

---

# 55. The Most Important MCP Mental Model

Remember this chain:

```text
Host
 ↓
MCP Client
 ↓
MCP Server
 ↓
Tools
 ↓
External capability
```

And remember:

> **The MCP server is not necessarily the underlying service.**

It may simply provide the standardized interface that translates an LLM's tool request into an API call or another operation.

---

# 56. Local vs. Remote: Quick Revision

### Local

```text
Host
 ↓
MCP Client
 ↓
Local MCP Server
```

Typical transport:

```text
STDIO
```

Typical examples:

* File System
* Fetch
* Playwright

### Remote

```text
Host
 ↓
MCP Client
 ↓
Internet
 ↓
Remote MCP Server
```

Typical transport:

```text
Streamable HTTP
```

Typical example:

* Context7
* Third-party SaaS integrations

---

# 57. Transport Quick Reference

| Transport       | Typical use                           |
| --------------- | ------------------------------------- |
| STDIO           | Local MCP servers                     |
| Streamable HTTP | Remote MCP servers                    |
| SSE             | Older/legacy HTTP-based MCP transport |

The course emphasizes **STDIO** and **Streamable HTTP**.

---

# 58. MCP Server Configuration Quick Reference

### Python server

```text
uvx
```

Example concept:

```python
{
    "command": "uvx",
    "args": ["mcp-server-fetch"]
}
```

### JavaScript/TypeScript server

```text
npx
```

Example concept:

```python
{
    "command": "npx",
    "args": ["..."]
}
```

### Remote server

```python
{
    "url": "...",
    "timeout": 60
}
```

---

# 59. Key Questions to Test Yourself

### What is an AI agent?

An LLM equipped with tools that loops to achieve a goal.

### What is an agent harness?

The tools, loops, and supporting machinery around an LLM that turn it into an agent.

### Is MCP an agent framework?

No. MCP is a protocol/standard for connecting applications to tools, resources, and prompts.

### What are the three MCP components?

* Host
* Client
* Server

### Where does the MCP client live?

Inside the MCP host/application.

### What does an MCP server provide?

Primarily tools, along with resources and prompts.

### What are the two major MCP operations discussed?

* List tools
* Call tools

### What is the most common transport for local servers?

STDIO.

### What is commonly used for remote servers?

Streamable HTTP.

### Why is MCP valuable?

It standardizes how independently developed capabilities can be exposed to AI applications.

### Do you need MCP to create your own tool?

No.

### When is MCP particularly useful?

When you want to integrate tools or capabilities created by someone else.

---

# 60. Final Week 6 Takeaways

* An **AI agent** can be understood as an LLM + tools + loop + goal.
* An **agent harness** is the machinery surrounding the LLM that enables this behavior.
* Agent frameworks range from lightweight abstractions to full, batteries-included harnesses.
* **MCP is not an agent framework.**
* MCP is a **protocol/standard** for connecting AI applications with capabilities.
* MCP focuses on standardizing access to **tools, resources, and prompts**.
* Tools are by far the most important practical use case.
* An **MCP host** is the application running the LLM/agent.
* An **MCP client** runs inside the host and connects to MCP servers.
* An **MCP server** exposes tools and other capabilities.
* MCP servers can run locally or remotely.
* **STDIO** is commonly used for local MCP servers.
* **Streamable HTTP** is commonly used for remote MCP servers.
* `uvx` is useful for launching Python-based MCP servers.
* `npx` is useful for launching JavaScript/TypeScript MCP servers.
* MCP servers expose tool descriptions and input schemas that LLMs can understand.
* Agent frameworks can handle MCP client creation automatically.
* Playwright MCP exposes browser automation capabilities.
* File System MCP exposes local file operations.
* Fetch MCP provides web-page retrieval.
* Context7 provides access to current library documentation through a remote MCP server.
* MCP does not replace the underlying tool technology; it standardizes the interface.
* The major value of MCP comes from **interoperability and the growing ecosystem of reusable tools**.

---

# 61. Week 6 in One Diagram

```text
                         AI Agent
                            │
                    ┌───────┴────────┐
                    │       LLM      │
                    └───────┬────────┘
                            │
                       Agent Harness
                            │
                       MCP Client
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
          ↓                 ↓                 ↓
    Fetch Server     Playwright Server   File System Server
          │                 │                 │
          ↓                 ↓                 ↓
      Web/API            Browser             Files

                            │
                            │
                    Remote MCP Example
                            ↓
                     Streamable HTTP
                            ↓
                       Context7
                            ↓
                 Current Documentation
```

## One-sentence summary

> **MCP is a standardized connectivity layer that lets AI applications discover and use tools created by other people and systems, while agent frameworks remain responsible for building and running the agents themselves.**

### External references

* [Model Context Protocol — Official Documentation](https://modelcontextprotocol.io/)
* [MCP Specification](https://modelcontextprotocol.io/specification/latest)
* [OpenAI Agents SDK](https://openai.github.io/openai-agents-python/)
* [Anthropic MCP documentation](https://docs.anthropic.com/en/docs/agents-and-tools/mcp)
* [Playwright](https://playwright.dev/)
* [Playwright MCP](https://github.com/microsoft/playwright-mcp)
* [Context7](https://context7.com/)
* [LangGraph](https://docs.langchain.com/oss/python/langgraph/overview)
* [LangChain](https://docs.langchain.com/)
* [Claude Agent SDK](https://docs.claude.com/en/docs/agent-sdk/overview)

Source: Week 6 transcript provided in the uploaded file. 
