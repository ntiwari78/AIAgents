The new attachment covers **Week 5, Days 2–4**: Strands Agents, Pydantic AI, Microsoft Agent Framework, Agno, and Mastra. The strongest theme is that the frameworks look remarkably similar once you understand the underlying agent architecture. 

# Week 5 Days 2–4 — Agent Frameworks

## 1. The Main Lesson

The most important message across these days is:

> **Once you understand one agent framework, you understand the core concepts of most of the others.**

Although the APIs differ, the same fundamental pattern keeps appearing:

```text
1. Create an agent
        ↓
2. Run the agent
        ↓
3. Add tools
        ↓
4. Add MCP
        ↓
5. Put the agent in a loop with a goal
```

The differences are mostly in:

* API syntax
* Tool definitions
* Model integration
* Observability
* Workflow/orchestration
* Developer experience
* Deployment/runtime options

The course deliberately repeats the same exercise to make this similarity obvious. 

---

# 2. The Universal Agent Pattern

Across Strands, Pydantic AI, Microsoft Agent Framework, Agno, and Mastra, the architecture is essentially:

```text
                 LLM
                  │
                  ↓
              Agent Loop
                  │
          ┌───────┴───────┐
          ↓               ↓
        Tools            MCP
          │               │
          └───────┬───────┘
                  ↓
                 Goal
                  ↓
             Goal achieved
```

The framework mainly provides convenient abstractions around this loop.

---

# 3. Strands Agents

**Strands Agents** is the framework introduced first on Day 2.

The material presents it as:

* Lightweight
* Open source
* Python and TypeScript
* Easy to switch models
* MCP-enabled
* Tool-oriented
* Equipped with observability options
* Closely connected with AWS/Bedrock

The course specifically demonstrates Strands using an OpenAI model rather than requiring Bedrock. 

Current Strands documentation likewise describes it as a library that runs inside your own Python or Node process, with Bedrock as the default model provider but support for other providers. ([Strands Agents][1])

---

# 4. Strands — Creating an Agent

The basic pattern is very lightweight:

```python
agent = Agent(
    model=model,
    system_prompt="You are a concise friendly assistant."
)
```

The course emphasizes that Strands uses a straightforward `system_prompt` rather than introducing different naming conventions such as `instruction` or `instructions`.

---

# 5. Strands — Running an Agent

The execution method demonstrated is:

```python
await agent.invoke_async(...)
```

For example:

```text
Say hello in Spanish.
```

produces:

```text
Hola
```

Again, at this point this is essentially an LLM call wrapped in an agent abstraction.

---

# 6. Strands — Tools

Strands uses a `@tool` decorator.

Conceptually:

```python
@tool
def show_todos():
    """Show the current to-do list."""
    ...
```

The tool definition uses:

* Function
* Type hints
* Docstring
* Optional argument descriptions

The framework uses this information to construct the tool interface for the model. 

---

# 7. Strands — MCP

The course then connects a filesystem MCP server.

The resulting architecture is:

```text
Strands Agent
      ↓
MCP Client
      ↓
Filesystem MCP Server
      ↓
Workspace
```

The agent can then read files such as:

```text
notes.txt
```

without the developer manually implementing file-reading tools.

---

# 8. Strands — Agent Loop

The final Strands example combines:

* Agent
* Tools
* MCP
* To-do board
* Goal

The agent is asked to:

1. Read `notes.txt`.
2. Translate it into Spanish.
3. Write the translation.
4. Complete the to-do items.

The agent creates steps, executes them, and marks them complete.

This demonstrates the full:

```text
LLM + tools + loop + goal
```

pattern. 

---

# 9. Strands — Key Takeaway

Strands demonstrates that an agent framework can remain extremely small while still supporting:

* Tools
* MCP
* Agent loops
* Multiple model providers
* Observability

The course characterizes it as a relatively early/lightweight framework, so version pinning is highlighted as useful because the framework is changing quickly. 

---

# 10. Pydantic AI

The next framework is **Pydantic AI**.

Its defining characteristic in the course is the strong relationship with:

* Python typing
* Pydantic models
* Structured outputs
* Tool schemas
* Validation
* Logfire observability

The material presents it as another framework with essentially the same agent architecture but a particularly strong typing orientation. 

The current Pydantic AI documentation similarly emphasizes typed agents, typed tools, structured outputs, multiple model providers, MCP, and observability. ([Pydantic Docs][2])

---

# 11. Pydantic AI — Model Definition

The course uses a model specification resembling:

```text
openai-chat: GPT-4o-mini
```

The model is then supplied to:

```python
Agent(...)
```

This gives the framework a provider/model combination.

---

# 12. Pydantic AI — Tools

Pydantic AI can use ordinary functions as tools.

The course deliberately points out that decorators are not always necessary.

This gives a flexible model:

```text
Python function
      ↓
Agent tool
```

The tool's:

* Function signature
* Type hints
* Docstring

provide the information needed to expose the tool to the model.

Current Pydantic AI documentation similarly describes typed function tools whose signatures and docstrings become part of the tool schema, with arguments validated before execution. ([Pydantic Docs][2])

---

# 13. Pydantic AI — MCP

The MCP integration follows the same general pattern seen in the previous frameworks.

Conceptually:

```text
Pydantic Agent
      ↓
MCP Tool Set
      ↓
Filesystem MCP
      ↓
Workspace
```

The agent can therefore access tools supplied by an external MCP server.

---

# 14. Pydantic AI — Agent Loop

The final example again uses:

```text
To-do board
     +
Tools
     +
Filesystem MCP
     +
Goal
     ↓
Agent loop
```

The goal is to:

* Read notes
* Translate them
* Write the translation
* Complete the board

The resulting behavior is essentially identical to the Strands and ADK examples.

That repetition is intentional.

---

# 15. Pydantic AI — Observability

The material highlights **Logfire**.

Logfire is the observability platform associated with the Pydantic ecosystem.

The instructor describes it as a potentially powerful option for observing agent execution.

The course does not use it in the lab because of additional dependencies, but recommends exploring it separately. 

The current Pydantic documentation also describes Logfire as its observability platform and emphasizes OpenTelemetry-based instrumentation. ([Pydantic Docs][2])

---

# 16. Pydantic AI — Structured Outputs

This is an especially natural fit for Pydantic.

Instead of:

```text
LLM → arbitrary text
```

you can define:

```python
class Result(BaseModel):
    name: str
    score: float
    explanation: str
```

and ask the agent to return that structured type.

The advantage is that application code can work with validated Python objects rather than parsing arbitrary text.

---

# 17. Microsoft Agent Framework

Day 3 begins with the **Microsoft Agent Framework**.

The course provides some historical context:

```text
AutoGen
   +
Semantic Kernel
   ↓
Microsoft Agent Framework
```

The material presents Microsoft Agent Framework as Microsoft's newer direction for agent development, while AutoGen is being transitioned toward it. 

Current Microsoft documentation describes Agent Framework as supporting agents, workflows, middleware, tools, memory, human-in-the-loop, checkpoints, orchestration, and hosting. ([Microsoft Learn][3])

---

# 18. Microsoft Agent Framework — Languages

A major distinction is that Microsoft provides:

* Python
* .NET / C#

This is particularly relevant for organizations already invested in the Microsoft/.NET ecosystem.

---

# 19. Microsoft Agent Framework — Agent

The basic pattern again looks familiar:

```python
client = OpenAIChatClient(...)
agent = Agent(
    client=client,
    instructions="..."
)
```

The agent is then run using:

```python
await agent.run(...)
```

The course emphasizes how similar this is to the other frameworks.

---

# 20. Microsoft Agent Framework — Tools

Tools can again be ordinary functions.

The agent receives something like:

```text
tools = [
    show_todos,
    plan_steps,
    complete_task
]
```

The framework then makes these functions available to the model.

---

# 21. Microsoft Agent Framework — MCP

The course demonstrates MCP integration through an MCP tool object.

The overall pattern remains:

```text
Agent
  ↓
MCP tool
  ↓
Filesystem server
```

The agent can then read `notes.txt` and summarize it.

---

# 22. Microsoft Agent Framework — Workflows

One important difference is that Microsoft Agent Framework has more sophisticated workflow functionality underneath the basic agent abstraction.

The course describes it as having capabilities reminiscent of graph-based orchestration.

The lesson doesn't go deeply into these workflows, but this is an area worth studying after understanding the basic agent loop.

Current Microsoft documentation explicitly separates **agents** and **workflows**, and includes orchestration, checkpoints/resuming, and human-in-the-loop capabilities. ([Microsoft Learn][3])

---

# 23. Microsoft Agent Framework — Key Takeaway

The framework again follows:

```text
Create
  ↓
Run
  ↓
Tools
  ↓
MCP
  ↓
Goal loop
```

The primary additional consideration is its Microsoft ecosystem integration and richer workflow capabilities.

---

# 24. Agno

The next Python framework is **Agno**.

The course describes Agno as:

* Lightweight
* Fast
* Open source
* Relatively mature
* Simple to use
* Focused on low overhead

The material also introduces **AgentOS**, the runtime/deployment component associated with Agno. 

Current Agno documentation describes the stack as:

```text
Agno SDK
    ↓
AgentOS
    ↓
Control Plane
```

where the SDK builds agents, teams, and workflows; AgentOS runs them; and the Control Plane manages them. ([Agno Documentation][4])

---

# 25. Agno — Agent Creation

The pattern is again familiar:

```python
model = OpenAIChat(...)
agent = Agent(
    model=model,
    instructions="..."
)
```

---

# 26. Agno — Running an Agent

The course uses:

```python
await agent.arun(...)
```

This is another example of a small syntactic difference between frameworks.

Compare:

```text
Strands       → invoke_async
Pydantic AI   → run
Microsoft     → agent.run
Agno          → arun
Mastra        → generate
```

The underlying operation remains the same:

```text
Send goal/message
        ↓
Agent loop
        ↓
Response
```

---

# 27. Agno — Tools

Agno can accept ordinary Python functions as tools.

Again:

```text
Python function
      ↓
Tool
      ↓
Agent
```

The function's documentation helps the model understand how to use it.

---

# 28. Agno — MCP

The course again connects the filesystem MCP server.

The agent receives:

* To-do tools
* MCP filesystem tools

and uses them together.

This allows it to:

1. Read the goal.
2. Plan tasks.
3. Read the file.
4. Translate it.
5. Write the result.
6. Complete the goal.

---

# 29. AgentOS

AgentOS is the operational/runtime side of Agno.

The course describes it as a way to:

* Deploy agents
* Run agents
* Scale agents

Current documentation goes further, describing AgentOS as a FastAPI-based runtime providing APIs, persistent state, authorization, tracing, evaluations, scheduling, MCP, and interfaces. ([Agno Documentation][5])

---

# 30. Agno — Key Takeaway

Agno emphasizes:

```text
Lightweight Agent
      +
Simple Tools
      +
MCP
      +
AgentOS Runtime
```

The course therefore distinguishes the **agent-building SDK** from the **runtime/deployment layer**.

---

# 31. Mastra

Day 4 introduces **Mastra**.

Unlike the preceding examples, Mastra is:

> **TypeScript-native.**

The course emphasizes that it was designed from the beginning as a TypeScript framework rather than being a Python framework that later acquired TypeScript support. 

Current Mastra documentation similarly describes it as a TypeScript framework for AI applications and agents. ([GitHub][6])

---

# 32. Mastra — Developer Experience

One of Mastra's major themes is developer experience.

The course highlights:

* TypeScript
* Local development
* Developer Studio
* Model flexibility
* Zod schemas
* MCP support
* Agent loops

---

# 33. Mastra — Zod

Mastra uses **Zod** for schemas.

This is roughly analogous to using Pydantic for typed schemas in Python.

Conceptually:

```text
Python
Pydantic
   ↓
Typed schema

TypeScript
Zod
   ↓
Typed schema
```

This is particularly useful when defining tool inputs.

---

# 34. Mastra — Tools

The course demonstrates:

```typescript
createTool(...)
```

A tool contains things such as:

* ID
* Description
* Input schema
* Execute function

The current Mastra documentation describes tools as typed functions with an input schema and executor, and uses Zod for schema definition. ([Mastra][7])

---

# 35. Mastra — MCP

Mastra provides an MCP client that can connect to MCP servers.

The course's filesystem example follows:

```text
Mastra Agent
      ↓
MCP Client
      ↓
Filesystem MCP
      ↓
Workspace
```

The current documentation confirms that Mastra agents can consume tools from MCP servers and can also expose Mastra agents as MCP tools. ([Mastra][7])

---

# 36. Mastra — Agent Loop

The execution method used in the course is:

```typescript
await worker.generate(...)
```

Again, this is simply another API spelling for the same fundamental operation.

The workflow remains:

```text
Goal
 ↓
Agent
 ↓
Tools
 ↓
MCP
 ↓
Work
 ↓
Goal completed
```

---

# 37. Mastra Studio

Mastra provides a local development environment called **Studio**.

The course demonstrates running:

```bash
npm run dev
```

and opening the local Studio interface.

The Studio allows you to:

* Select an agent
* Chat with the agent
* Observe its operation
* Test tools
* Iterate during development

The current Mastra Studio documentation similarly describes Studio as a local workspace for building, testing, tracing, and improving agents. ([Mastra][8])

---

# 38. Python vs TypeScript

The transition to Mastra is useful because it shows that the same agent concepts survive a language change.

### Python frameworks

* Strands
* Pydantic AI
* Microsoft Agent Framework
* Agno

### TypeScript

* Mastra

Yet the same architecture remains:

```text
Agent
+
Model
+
Tools
+
MCP
+
Loop
+
Goal
```

This reinforces the course's central argument.

---

# 39. Framework Comparison

| Framework                 | Language emphasized | Tool style                 | Agent execution | MCP | Notable emphasis                |
| ------------------------- | ------------------- | -------------------------- | --------------- | --- | ------------------------------- |
| Strands                   | Python / TypeScript | `@tool`                    | `invoke_async`  | Yes | Lightweight                     |
| Pydantic AI               | Python              | Typed functions/decorators | `run`           | Yes | Typing / structured output      |
| Microsoft Agent Framework | Python / .NET       | Functions                  | `agent.run`     | Yes | Microsoft ecosystem / workflows |
| Agno                      | Python              | Functions                  | `arun`          | Yes | Lightweight / AgentOS           |
| Mastra                    | TypeScript          | `createTool` + Zod         | `generate`      | Yes | TypeScript developer experience |

The precise APIs will evolve, so the table is most useful as a **conceptual comparison**, not as a permanent API reference. 

---

# 40. The Five-Step Cheat Sheet

## Strands

```text
Agent
 ↓
invoke_async()
 ↓
@tool
 ↓
MCP
 ↓
Agent loop
```

## Pydantic AI

```text
Agent
 ↓
run()
 ↓
Functions / tools
 ↓
MCP
 ↓
Agent loop
```

## Microsoft Agent Framework

```text
Agent
 ↓
agent.run()
 ↓
Functions
 ↓
MCP
 ↓
Agent loop
```

## Agno

```text
Agent
 ↓
arun()
 ↓
Functions
 ↓
MCP
 ↓
Agent loop
```

## Mastra

```text
Agent
 ↓
generate()
 ↓
createTool()
 ↓
MCP
 ↓
Agent loop
```

---

# 41. The Real Skill Being Taught

The course is **not** primarily asking you to memorize:

```text
invoke_async
run
arun
generate
```

Instead, understand:

```text
How do I build an agent?
How do I give it tools?
How do I connect external tools through MCP?
How do I make it iterate toward a goal?
```

Once those ideas are understood, learning another framework becomes mostly an API/documentation exercise.

---

# 42. What Actually Changes Between Frameworks?

The repeated labs reveal a useful classification.

### Layer 1 — Agent API

Different syntax:

```text
Agent(...)
LlmAgent(...)
```

### Layer 2 — Execution API

Different names:

```text
invoke_async()
run()
arun()
generate()
```

### Layer 3 — Tool API

Different mechanisms:

```text
@tool
plain function
createTool()
```

### Layer 4 — Schema system

Examples:

```text
Pydantic
Zod
type hints
```

### Layer 5 — Runtime/observability

Examples:

```text
Logfire
ADK Web
AgentOS
Mastra Studio
```

The core agent loop remains largely the same.

---

# 43. The Importance of MCP

MCP is the common denominator appearing throughout these frameworks.

The pattern is:

```text
Framework
    ↓
MCP Client
    ↓
MCP Server
    ↓
Reusable tools/resources
```

This means the same external capability can potentially be reused across different agent frameworks.

That is an important architectural benefit.

---

# 44. Why This Matters

Imagine you build a filesystem MCP server.

You can potentially connect it to:

```text
Google ADK
Strands
Pydantic AI
Microsoft Agent Framework
Agno
Mastra
```

Instead of rebuilding filesystem functionality separately for every framework.

This is one reason the course repeatedly introduces MCP.

---

# 45. Development Workflow

Another practical lesson is the development pattern used throughout the labs:

```text
Experiment
   ↓
Notebook / small program
   ↓
Verify behavior
   ↓
Package as Python/TypeScript module
   ↓
Run from terminal
   ↓
Use framework's developer tooling
```

This is a useful workflow for learning and prototyping agent systems.

---

# 46. Framework Selection

The material argues that framework choice often comes down to:

* Team expertise
* Language preference
* Developer experience
* Model ecosystem
* Tooling
* Observability
* Deployment requirements
* Existing infrastructure

The course's main message is that the underlying concepts are transferable.

Therefore, learning one framework deeply gives you a strong foundation for learning the others.

---

# 47. Important Caveat: Frameworks Change Quickly

The source repeatedly mentions:

* Renames
* Version changes
* New versions
* Breaking changes
* Early-stage APIs

This is particularly relevant to the frameworks covered here.

Therefore:

> **Treat framework syntax as version-specific knowledge.**

Your durable knowledge should be the architecture:

```text
Model
+
Tools
+
MCP
+
Agent loop
+
Goal
```

rather than memorizing every method name.

---

# 48. Final Mental Model

The entire four-day section can be summarized as:

```text
                         AGENT
                           │
                           ↓
                         MODEL
                           │
                           ↓
                     Agent Loop
                           │
             ┌─────────────┴─────────────┐
             ↓                           ↓
           Tools                        MCP
             ↓                           ↓
      Local functions             External capabilities
             │                           │
             └─────────────┬─────────────┘
                           ↓
                          GOAL
                           │
                           ↓
                      Goal achieved
```

The frameworks mostly differ in **how they package this architecture**.

---

# 49. One-Sentence Summary of Each Framework

### Strands

> Lightweight agent framework with simple tool integration, MCP, model flexibility, and observability.

### Pydantic AI

> Python-focused agent framework emphasizing typing, validation, structured outputs, and Pydantic ecosystem integration.

### Microsoft Agent Framework

> Microsoft's newer agent framework combining ideas from AutoGen and Semantic Kernel with agent and workflow capabilities.

### Agno

> Lightweight Python agent framework paired with AgentOS for running and serving agents.

### Mastra

> TypeScript-native framework emphasizing developer experience, typed tools with Zod, MCP, agents, workflows, and local Studio.

---

# 50. Final Takeaways

1. **Agent frameworks are more alike than different.**
2. Learn the **agent architecture**, not just framework syntax.
3. The recurring five steps are:

   * Create agent
   * Run agent
   * Add tools
   * Add MCP
   * Add an agent loop with a goal
4. **Tools** give agents the ability to act.
5. **MCP** provides reusable external capabilities.
6. **Pydantic AI** emphasizes typed Python development.
7. **Strands** emphasizes lightweight agent construction.
8. **Microsoft Agent Framework** emphasizes Microsoft's broader agent/workflow ecosystem.
9. **Agno** emphasizes lightweight agents plus AgentOS runtime capabilities.
10. **Mastra** provides a TypeScript-native agent development experience.
11. API method names are less important than understanding the underlying loop.
12. Framework versions change quickly, so always check current documentation.
13. The same agent concepts can transfer between Python and TypeScript.
14. Developer tooling and observability become increasingly important as agents become more complex.

## External links for further deep dive

### Strands Agents

* [Strands Agents — official documentation](https://strandsagents.com/docs/user-guide/quickstart/overview/?utm_source=chatgpt.com) — Start here for the current Python/TypeScript APIs, model providers, tools, and MCP.
* [Strands Agents — official site](https://strandsagents.com/?utm_source=chatgpt.com)

The current documentation describes Strands as a library rather than a hosted platform and documents provider switching beyond Amazon Bedrock. ([Strands Agents][1])

### Pydantic AI

* [Pydantic AI — official documentation](https://pydantic.dev/docs/ai/overview/?utm_source=chatgpt.com) — Recommended deep dive for agents, tools, structured output, MCP, models, and typed development.
* [Pydantic AI GitHub](https://github.com/pydantic/pydantic-ai?utm_source=chatgpt.com)
* [Pydantic Logfire](https://logfire.pydantic.dev/?utm_source=chatgpt.com) — For the observability portion of the lesson.

The current documentation shows that Pydantic AI has expanded significantly beyond the basic agent API covered in this lesson, including typed tools, structured outputs, MCP, multi-agent patterns, evaluations, and durable execution. ([Pydantic Docs][2])

### Microsoft Agent Framework

* [Microsoft Agent Framework documentation](https://learn.microsoft.com/en-us/agent-framework/?utm_source=chatgpt.com) — Official current documentation.
* [Microsoft Agent Framework — GitHub](https://github.com/microsoft/agent-framework?utm_source=chatgpt.com)
* [Microsoft migration guidance from AutoGen and Semantic Kernel](https://learn.microsoft.com/en-us/agent-framework/?utm_source=chatgpt.com)

The current documentation has dedicated sections for agents, workflows, agent harnesses, middleware, memory, tools, human-in-the-loop, checkpoints, and hosting. ([Microsoft Learn][3])

### Agno

* [Agno documentation](https://docs.agno.com/?utm_source=chatgpt.com) — Main documentation.
* [Agno Quickstart](https://docs.agno.com/examples/basics/overview?utm_source=chatgpt.com) — Good hands-on starting point.
* [Agno AgentOS](https://docs.agno.com/agent-os/introduction?utm_source=chatgpt.com) — Runtime/deployment concepts.
* [Agno GitHub](https://github.com/agno-agi/agno?utm_source=chatgpt.com)

The current Agno docs cover agents, structured output, memory, knowledge, guardrails, human-in-the-loop, multi-agent teams, workflows, and AgentOS. ([Agno Documentation][9])

### Mastra

* [Mastra official documentation](https://mastra.ai/docs?utm_source=chatgpt.com)
* [Mastra GitHub](https://github.com/mastra-ai/mastra?utm_source=chatgpt.com)
* [Mastra MCP tools documentation](https://mastra.ai/docs/agents/mcp-guide?utm_source=chatgpt.com)
* [Mastra Studio](https://mastra.ai/studio?utm_source=chatgpt.com)

The current Mastra project describes itself as a TypeScript framework for AI applications and agents, with model routing, agents, MCP servers, evaluations, and observability. ([GitHub][6])

## Recommended deep-dive order

```text
1. Strands
      ↓
2. Pydantic AI
      ↓
3. Microsoft Agent Framework
      ↓
4. Agno + AgentOS
      ↓
5. Mastra
      ↓
6. MCP across all of them
      ↓
7. Build the same agent in 2–3 frameworks
```

The most valuable exercise is **not** building five unrelated agents. Build the *same* small agent—using the same tools, MCP server, and goal—in two or three frameworks and compare the code. That directly reinforces the central lesson of the attached material. 

[1]: https://strandsagents.com/docs/user-guide/quickstart/overview/?utm_source=chatgpt.com "Get started | Strands Agents"
[2]: https://ai.pydantic.dev/ "Pydantic AI | Pydantic Docs"
[3]: https://learn.microsoft.com/en-us/agent-framework/?utm_source=chatgpt.com "Agent Framework documentation | Microsoft Learn"
[4]: https://docs.agno.com/index?utm_source=chatgpt.com "Welcome to Agno | Agno"
[5]: https://docs.agno.com/agent-os/introduction?utm_source=chatgpt.com "What is AgentOS? | Agno"
[6]: https://github.com/mastra-ai/mastra/blob/main/README.md?utm_source=chatgpt.com "mastra/README.md at main · mastra-ai/mastra · GitHub"
[7]: https://mastra.ai/docs/agents/mcp-guide?utm_source=chatgpt.com "Using Tools with Agents | Agents | Mastra Docs"
[8]: https://mastra.ai/studio?utm_source=chatgpt.com "Observe, Evaluate and Improve Your AI Agents | Mastra"
[9]: https://docs.agno.com/examples/basics/overview?utm_source=chatgpt.com "Quickstart | Agno"
