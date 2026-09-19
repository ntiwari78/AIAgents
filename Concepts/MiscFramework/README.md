
# Week Five Agent Framework Survey: Strands, Pydantic AI, Microsoft Agent Framework, Agno, and Mastra

## Overview: Agent-framework training session

* Technical lecture and hands-on comparison of five-step agent implementations across Python and TypeScript frameworks
* One instructor-led speaker; no other participants are clearly identified
* Critical outcomes:

  * Strands Agents and Pydantic AI completed the standard five-step agent workflow
  * Microsoft Agent Framework and Agno demonstrated the same workflow with framework-specific syntax
  * Mastra showed the TypeScript-native version of the workflow, including its developer studio
  * The frameworks share a common pattern: model, agent, tools, MCP, and an agent loop

## Strands Agents: Lightweight AWS-Originated Framework

[Strands Agents](https://strandsagents.com/docs/user-guide/quickstart/overview/), available through `strandsagents.com`, was presented as an open-source framework from Amazon AWS that intentionally keeps AWS branding relatively understated. It is closely connected to Bedrock but is designed to support the broader Python and TypeScript communities.

* **Framework profile**:

  * Lightweight, new, open source, and currently relatively immature despite being at version 1.0
  * Bedrock is the natural default, but other providers can be swapped in easily
  * Demonstrated with `GPT-4o-mini`
  * Includes tools, MCP server support, an agent loop, provider flexibility, and observability integrations
  * Observability can use AWS services or [OpenTelemetry](https://opentelemetry.io/docs/)-based platforms such as [Langfuse](https://langfuse.com/docs)
* **Implementation pattern**:

  * Model configured as an OpenAI model with an API key and model ID
  * Agent created with a model and a `system_prompt`
  * Agent execution uses `invoke_async`
  * A Spanish greeting produced “Hola”
* **Tool integration**:

  * Tools use the `@tool` decorator rather than OpenAI’s `@function_tool`
  * Docstrings, type hints, and argument descriptions provide the information needed to generate tool schemas
  * To-do-board tools included showing to-dos, planning steps, and completing tasks
* **MCP integration**:

  * [MCP (Model Context Protocol)](https://modelcontextprotocol.io/) uses `STDIO` server parameters and an MCP client
  * The file-system MCP server successfully read `notes.txt` and summarized it as a small language to-do project
* **Agent loop**:

  * A worker agent read the pending goal from a shared SQLite to-do board
  * It planned and completed the steps to read `notes.txt`, translate it into Spanish, and write `Spanish.txt`
  * The same workflow ran successfully from `strands_worker.py` using `UV run`
* **Trade-offs**:

  * Bedrock integration is especially convenient for existing AWS users
  * Version changes happen quickly, so production users should pin versions and monitor updates
  * Strands was characterized as an early-stage framework with relatively little abstraction

---

## Pydantic AI: Similar Workflow with Strong Typing and Structured Outputs

[Pydantic AI](https://pydantic.dev/docs/ai/overview/) was introduced as another lightweight framework whose overall workflow closely resembles Strands Agents, ADK, the OpenAI Agents SDK, and LangGraph’s agent creation utilities.

* **Framework profile**:

  * Uses Pydantic’s typing model throughout the framework
  * Supports flexible tool definitions, MCP, model swapping, and structured outputs
  * Offers Logfire as its observability platform
  * A V2 release was described as being in beta and potentially capable of introducing breaking changes
* **Agent and model setup**:

  * Model specified using provider/model syntax such as `OpenAI-chat: GPT-4o-mini`
  * Agent created with the model and instructions
  * Tools can be passed as ordinary functions; decorators are optional
* **MCP integration**:

  * Uses an STDIO transport with an MCP tool set
  * Configuration is similar to other frameworks, with small naming differences such as `init_timeout`
  * The file-system MCP tool set successfully read and summarized `notes.txt`
* **Agent loop**:

  * Worker agent received the goal of reading the notes, translating them into Spanish, and writing the result
  * To-do tools and the file-system MCP tool set were supplied together
  * The agent planned multiple steps, completed the goal, and created `Spanish.txt`
  * `pydantic_worker.py` reproduced the workflow successfully as a Python module
* **Observability note**:

  * Logfire was not used in the course because it introduced additional abstraction and package cross-dependencies
  * It was compared conceptually with Langfuse, which is covered in the production course

---

## Microsoft Agent Framework: Consolidation of AutoGen and Semantic Kernel

[Microsoft Agent Framework](https://learn.microsoft.com/en-us/agent-framework/) was presented as Microsoft’s new official open-source Agent SDK and the intended successor to its earlier agent-framework products.

* **Product history**:

  * Microsoft originally created AutoGen with the open-source community
  * A separate group later formed AG2, based more closely on the original AutoGen
  * Microsoft continued with a substantially revised AutoGen before moving toward Microsoft Agent Framework
  * The new framework combines ideas from AutoGen and Semantic Kernel
  * AutoGen remains available but is being sunset rather than further developed
* **Framework characteristics**:

  * Supports Python and .NET
  * Uses plain functions as tools
  * Provides MCP support, agent loops, telemetry, and more sophisticated workflow functionality
  * Its internal workflow capabilities were compared to a graph-based system and linked to Semantic Kernel’s more robust orchestration features
  * The framework remains young and has experienced class renames and other churn, including changes in 2026
* **Five-step implementation**:

  * An OpenAI chat client was created for `GPT-4o-mini`
  * Agents use `instructions` rather than `system_prompt`
  * Execution uses `await agent.run`
  * Plain functions were supplied as to-do-board tools
  * MCP tools were added through an MCP tool object and STDIO parameters
* **Demonstrated result**:

  * The file-system MCP server read `notes.txt`
  * The worker agent read the board, planned the translation task, completed it, and generated the Spanish file
  * `MAF_worker.py` successfully ran the same process as a Python module
* **Environment caveat**:

  * Notebook execution required extra Windows-specific handling around the MCP STDIO tool
  * The production implementation was described as cleaner and did not require the notebook workaround

---

## Agno: Fast, Lightweight Runtime-Oriented Framework

[Agno](https://docs.agno.com/index), formerly known as PyData, was presented as a mature, lightweight open-source framework paired with the AgentOS runtime.

* **Framework profile**:

  * Very low overhead; creating an agent was described as taking roughly three microseconds
  * Agent objects are lightweight data objects
  * Supports model swapping, function-based tools, MCP tools, and asynchronous execution
  * Uses OpenAI Chat with `GPT-4o-mini` in the examples
* **Agent execution**:

  * Agent creation resembles the Microsoft Agent Framework
  * Execution uses `await agent.arun`
  * A Spanish greeting returned “Hola”
* **Tools and MCP**:

  * Tools were defined directly as functions without decorators
  * Docstrings are converted into the JSON tool descriptions used by the framework
  * MCP tools were connected through STDIO server parameters and an MCP context manager
  * A notebook-specific Windows workaround was again required
* **Agent loop**:

  * The worker combined to-do-board tools with file-system MCP tools
  * It autonomously planned and completed the translation task
  * The run created `Spanish.txt` and crossed the goal and sub-tasks off the board
  * `agno_worker.py` reproduced the process as a Python module
* **AgentOS trade-off**:

  * AgentOS provides deployment, execution, scaling, and runtime capabilities for Agno agents
  * It was described as primarily a paid product with a possible free tier
  * Agno itself is mature, while the AgentOS product had recently undergone a revamp
  * V2-related changes and the commercial deployment emphasis are considerations for adopters

---

## Mastra: TypeScript-Native Agent Development

[Mastra](https://mastra.ai/) was introduced as the first genuinely TypeScript-native framework in the sequence. It was built by the team behind Gatsby and emphasizes developer experience, local tooling, and integration with the Vercel AI SDK.

* **Framework characteristics**:

  * Created primarily for TypeScript rather than ported from a Python-first framework
  * Model-agnostic through the Vercel AI SDK
  * Supports tools, MCP servers, agent loops, and a local developer studio
  * Uses Zod for schemas, analogous to Pydantic in Python
  * Agent execution uses `agent.generate`
* **Environment setup**:

  * Work was performed in five TypeScript programs rather than Python notebooks
  * Node version 24 or later was recommended
  * Dependencies were installed with `npm install`
  * The SQLite to-do board was implemented in `board.ts`
* **Tool definitions**:

  * Tools were created with Mastra’s `createTool`
  * Each tool includes an ID, description, Zod input schema, and an `execute` callback
  * `tools.ts` contained the three board tools and the file-system MCP client
* **Five-step implementation**:

  * Step one created an agent with an ID, name, instructions, and model
  * Step two used `agent.generate` to produce “Hola”
  * Step three supplied board tools and retrieved the pending item
  * Step four exposed the file-system MCP server’s tools through `listTools`
  * Step five combined board tools and MCP tools, then called `worker.generate`
* **Agent loop result**:

  * The worker read the notes, translated the contents into Spanish, wrote `spanish.txt`, and completed the board goal
  * The process was run through commands such as `npm run step five`
* **Developer studio**:

  * `npm run dev` launched Mastra’s local studio
  * The studio was available at `localhost:4111`
  * The worker agent could be selected and interacted with through a browser-based interface
  * A prompt to work the board caused the agent to use its tools and return the Spanish result

---

## Core Lesson: Framework Syntax Changes, Agent Architecture Does Not

The repeated demonstrations were intentionally similar to establish that most agent frameworks implement the same core architecture.

* **Shared five-step pattern**:

  * Create an agent and configure a model
  * Run the agent
  * Add ordinary function tools
  * Add tools from an MCP server
  * Put the agent in a loop that works toward a goal
* **Framework-specific differences**:

  * Strands Agents uses `invoke_async` and an `@tool` decorator
  * Pydantic AI supports typed functions and structured outputs
  * Microsoft Agent Framework uses `agent.run`, plain functions, and .NET support
  * Agno uses `agent.arun` and emphasizes low overhead and AgentOS
  * Mastra uses TypeScript, Zod, `createTool`, and `agent.generate`
* **Practical implication**:

  * Developers do not need to memorize every framework’s syntax
  * Documentation, coding agents, and the course labs can provide the small implementation differences
  * The labs function as reusable cookbook examples for switching frameworks
* **Course progression**:

  * The session covered Strands Agents and Pydantic AI on week five, day two
  * Microsoft Agent Framework and Agno were covered on day three
  * Mastra completed the framework survey on day four
  * The course reached 80% completion, with the upcoming project focused on combining the frameworks’ shared agent-loop concepts

## References

* [Strands Agents documentation](https://strandsagents.com/docs/user-guide/quickstart/overview/) ([Strands Agents][1])
* [Pydantic AI documentation](https://pydantic.dev/docs/ai/overview/) ([Pydantic Docs][2])
* [Microsoft Agent Framework documentation](https://learn.microsoft.com/en-us/agent-framework/) ([Microsoft Learn][3])
* [Agno documentation](https://docs.agno.com/index) ([Agno Documentation][4])
* [Mastra](https://mastra.ai/) ([Mastra][5])
* [Model Context Protocol](https://modelcontextprotocol.io/) ([Model Context Protocol Blog][6])
* [OpenTelemetry documentation](https://opentelemetry.io/docs/) ([OpenTelemetry][7])
* [Langfuse documentation](https://langfuse.com/docs) ([Langfuse][8])

The original material is preserved; the additions are Markdown structure and verified reference links.

[1]: https://strandsagents.com/docs/user-guide/quickstart/overview/?utm_source=chatgpt.com "Get started | Strands Agents"
[2]: https://ai.pydantic.dev/ "Pydantic AI | Pydantic Docs"
[3]: https://learn.microsoft.com/en-us/agent-framework/?utm_source=chatgpt.com "Agent Framework documentation | Microsoft Learn"
[4]: https://docs.agno.com/index?utm_source=chatgpt.com "Welcome to Agno | Agno"
[5]: https://mastra.ai/?trk=article-ssr-frontend-pulse_little-text-block&utm_source=chatgpt.com "TypeScript AI Framework for Agents and Apps | Mastra"
[6]: https://blog.modelcontextprotocol.io/posts/welcome-to-mcp-blog/?utm_source=chatgpt.com "The Model Context Protocol Blog | Model Context Protocol Blog"
[7]: https://opentelemetry.io/docs/?utm_source=chatgpt.com "Documentation | OpenTelemetry"
[8]: https://langfuse.com/docs?utm_source=chatgpt.com "Open Source AI Engineering Platform - Langfuse"

---
---

I treated the transcript as a request for the same **key-points + external-links study guide** format as your previous Week 5 notes. The summary below stays grounded in the uploaded transcript. 

# Week 5 Day 5 — Agent Loop Project

## 1. Main Theme

The final project of Week 5 brings together the agent frameworks studied throughout the week to demonstrate a central idea:

> **Different agent frameworks have different syntax, but the underlying agent architecture is remarkably similar.**

The project demonstrates:

* Multiple agent frameworks working together
* Agents collaborating through a shared to-do list
* An **outer agent loop** controlling multiple **inner agent loops**
* Google ADK acting as the overall orchestrator
* Strands, Pydantic AI, Microsoft Agent Framework, Agno, and Mastra acting as worker frameworks
* MCP being used to give agents browser capabilities
* Automated QA using Playwright
* Measurable outcomes and feedback loops as a path toward more reliable agents

---

## 2. The Six Frameworks

The course has now covered six major agent frameworks:

1. **Google ADK**
2. **Strands Agents**
3. **Pydantic AI**
4. **Microsoft Agent Framework**
5. **Agno**
6. **Mastra**

### Core observation

Although the syntax differs, the fundamental workflow remains:

```text
Create agent
    ↓
Give it a model
    ↓
Give it tools
    ↓
Run the agent
    ↓
Let it work toward a goal
```

The instructor emphasizes that a framework is essentially **utility and abstraction code** that makes common agent-development activities easier.

You can still build agents using direct LLM API calls without a framework.

---

# 3. Framework Differences Are Mostly Implementation Details

The instructor highlights several differences:

| Framework                 | Notable characteristic                                        |
| ------------------------- | ------------------------------------------------------------- |
| Google ADK                | Defaults naturally toward Gemini; sophisticated orchestration |
| Strands                   | Uses a `@tool` decorator                                      |
| Pydantic AI               | Strong typing and structured outputs                          |
| Microsoft Agent Framework | Sophisticated orchestration and Python/.NET                   |
| Agno                      | Lightweight runtime and AgentOS                               |
| Mastra                    | TypeScript-native development and developer tooling           |

The important lesson is not to memorize every API.

Instead:

* Understand the **agent architecture**
* Understand how tools work
* Understand how agents loop
* Learn the framework-specific syntax when you need it

---

# 4. Frameworks Are Optional

One of the most important conceptual points is that **agent frameworks are not mandatory**.

You can implement an agent directly with:

* An LLM API
* Tool definitions
* A prompt
* A loop
* Some state

Frameworks primarily provide abstractions around these recurring patterns.

Therefore:

```text
LLM API
   +
Tools
   +
Loop
   =
Agent-like system
```

A framework makes this easier to build and maintain, but it does not fundamentally change what an agent is doing.

---

# 5. Project #7 — The Agent Loop

The main Week 5 Day 5 project is called:

**Agent Loop**

The goal is to create a team of agents that collaborate to build a working website.

The user supplies a language, such as:

```text
Spanish
```

The agent team then creates a website containing multiple games and puzzles designed to teach that language.

The website runs locally rather than being deployed.

---

# 6. Multi-Agent Architecture

The project contains:

* **One orchestrator**
* **Five worker agents**
* **One shared to-do board**
* **One QA agent**
* **One CSS agent**

The five worker frameworks are:

```text
Strands
Pydantic AI
Microsoft Agent Framework
Agno
Mastra
```

The orchestrator is built using:

```text
Google ADK
```

---

# 7. The Shared To-Do Board

The central coordination mechanism is a shared SQLite to-do list.

The board contains:

* High-level goals
* Individual tasks
* Progress
* Completed items

Each worker agent can:

1. Look at the board
2. Find an available task
3. Work on the task
4. Update the board
5. Mark the task as complete
6. Pick up another task

This simple mechanism creates the appearance of autonomous collaboration.

---

# 8. Why the To-Do List Creates Agentic Behavior

A key conceptual lesson is that the LLM does not literally "know" that it is inside an agent loop.

Instead, the LLM is repeatedly given:

* A goal
* Tools
* Instructions
* Access to the current state

The to-do-list tools allow it to:

* Inspect pending work
* Select tasks
* Perform actions
* Mark tasks complete
* Continue to the next task

This produces behavior that looks autonomous.

```text
LLM
 ↓
Check task
 ↓
Use tools
 ↓
Complete task
 ↓
Mark task complete
 ↓
Check next task
 ↓
Repeat
```

This is essentially the same agent-loop concept introduced earlier in the course without using an agent framework.

---

# 9. Nested Agent Loops

This project demonstrates something more advanced:

## Loop within a loop

### Outer loop

The **ADK orchestrator**:

* Determines the overall work
* Creates goals
* Assigns tasks
* Launches workers
* Judges progress
* Performs final repairs

### Inner loops

Each worker agent:

* Checks the shared board
* Takes a task
* Performs the work
* Updates the board
* Continues until its work is complete

Conceptually:

```text
                 ADK Orchestrator
                        │
          ┌─────────────┼─────────────┐
          │             │             │
       Worker         Worker        Worker
        Loop           Loop          Loop
          │             │             │
      To-do Board   To-do Board   To-do Board
```

All workers ultimately share the same overall board.

---

# 10. Parallel Agent Execution

The orchestrator can discover worker agents in subdirectories.

If a framework directory exists, the orchestrator can launch that worker.

If you do not want to use a particular framework, its directory can simply be removed.

The workers can then run **in parallel**.

This means the project demonstrates:

```text
One overall goal
      ↓
Multiple independent tasks
      ↓
Multiple agents
      ↓
Parallel execution
      ↓
Shared progress
```

---

# 11. Why This Architecture Was Chosen

The instructor emphasizes that the architecture was not created merely because multi-agent diagrams look impressive.

The goal is to create a system that solves the problem effectively.

A useful principle is:

> Organize agents and LLM calls according to what produces the best measurable solution.

The boxes in the architecture should have a purpose.

Do not create:

* An orchestrator because "agents need orchestrators"
* Multiple agents because "multi-agent sounds better"
* Layers because architecture diagrams look sophisticated

Instead, each component should exist because it improves the solution.

---

# 12. Why A2A Is Not Needed Here

An important distinction is made between **agent collaboration** and **A2A**.

The project contains multiple agents, but they are all controlled by the same application.

Therefore, there is no need for A2A.

### In this project

```text
Your application
      ↓
Agent A
      ↓
Agent B
      ↓
Agent C
```

The application controls everything.

Agents can communicate through:

* Python code
* Tools
* Shared state
* Prompts

### A2A is useful when

Agents are:

* Independently deployed
* Controlled by different systems or organizations
* Running on different infrastructure
* Required to discover and communicate with each other

The key distinction is **control and discovery**.

---

# 13. Orchestration by Code vs. Orchestration by LLM

Another major lesson is the difference between two approaches.

## Orchestration by code

A deterministic workflow might look like:

```python
agent1()
agent2()
agent3()
agent4()
agent5()
```

Advantages:

* Predictable
* Deterministic
* Easier to reason about
* Easier to test

## Orchestration by LLM

The orchestrator itself decides:

```text
What should I do next?
Which agent should I call?
What should happen now?
Should I repeat something?
```

Advantages:

* More flexible
* Can adapt dynamically
* Can potentially repeat agents
* Can handle less predictable workflows

Disadvantages:

* Less deterministic
* Can choose the wrong order
* Can behave unpredictably
* Requires stronger evaluation and feedback

---

# 14. Why Use LLM Orchestration in This Project?

The instructor acknowledges that a simple Python workflow could be more reliable.

However, the purpose of this project is specifically to demonstrate **agent loops**.

Using an LLM orchestrator makes it possible to explore:

* Dynamic task assignment
* Repeated agent execution
* Flexible workflows
* Nested loops
* Autonomous decisions

For a production system, the simpler deterministic approach may sometimes be preferable.

---

# 15. The ADK Orchestrator

The orchestrator lives in:

```text
orchestrator.py
```

Its core structure follows the standard ADK pattern:

```text
Create LLM agent
        ↓
Configure model
        ↓
Provide instructions
        ↓
Provide tools
        ↓
Create session
        ↓
runner.run_async()
```

The tools allow the orchestrator to:

* Put tasks on the board
* Manage goals
* Start workers
* Coordinate the overall process

The instructor emphasizes focusing on this business logic rather than the platform-specific process-management code.

---

# 16. Separate Prompt File

The project follows a lesson learned from CrewAI:

> Keep prompts separate from application code.

The orchestrator's instructions live in a separate prompt file.

This makes it easier to:

* Modify the overall objective
* Experiment with different instructions
* Reuse the orchestration code
* Change the problem without rewriting the system

The prompt defines the overall task the agent network should accomplish.

---

# 17. Generic Worker Agents

An important design decision is that the five worker agents are **generic**.

They are not hard-coded specifically for Spanish.

Instead, they:

1. Look at the to-do board
2. Find their assigned task
3. Perform it
4. Mark it complete

This means the same worker agents can potentially be reused for a completely different project.

The task comes from the board rather than being permanently embedded in the worker.

---

# 18. QA Agent

The project introduces a separate **QA agent**.

Its purpose is to test the website created by the workers.

The QA agent uses:

* Google ADK
* Playwright
* MCP

The agent receives instructions for testing the games and tools for reporting the results.

---

# 19. Playwright MCP

The QA agent uses the Playwright MCP server to interact with the generated website.

The workflow is approximately:

```text
QA Agent
   ↓
Playwright MCP
   ↓
Browser
   ↓
Open website
   ↓
Play games
   ↓
Check functionality
   ↓
Report results
```

The official Playwright MCP implementation provides browser automation through MCP and exposes web pages to agents through structured accessibility information. ([GitHub][1])

---

# 20. CSS Agent

The project also contains a small dedicated **CSS agent**.

Its job is to generate general CSS that can be applied across the website.

The reason for making this a separate agent is important:

> A narrowly defined task can sometimes produce better results.

It could technically be considered part of the orchestrator's work, but separating it into a dedicated LLM call gives the task a clearer boundary.

---

# 21. Model Configuration

The project centralizes model selection in:

```text
config.py
```

The instructor chooses relatively powerful models for the demonstration but notes that cheaper models can also be used.

The important lesson is:

> Model selection is configurable and should depend on the task, cost, and desired quality.

---

# 22. Running the Project

The project is launched with a command along the lines of:

```bash
uv run agents_loop.py
```

The language can be changed from the default Spanish.

It is also possible to skip particular frameworks.

This allows you to experiment with different combinations of workers.

---

# 23. What Happened During the Demonstration

The orchestrator created assignments for:

* Strands
* Pydantic AI
* Microsoft Agent Framework
* Agno
* Mastra

Each worker then:

1. Received an assignment
2. Created its own to-do steps
3. Worked through those steps
4. Marked tasks complete
5. Reported progress to the shared board

The result was multiple frameworks working simultaneously under the control of one ADK orchestrator.

---

# 24. The QA Phase

After the games were created, the QA process began.

The Playwright-based QA agent:

* Opened a browser
* Visited the generated games
* Played through them
* Tested their behavior
* Reported whether they worked

The instructor observed occasional MCP timeouts during testing.

This demonstrates an important real-world point:

> Autonomous multi-agent systems can work, but they are not automatically reliable.

Smaller models may struggle more, and stronger prompting or repeated runs may sometimes be necessary.

---

# 25. Final Product

The agents created a language-learning arcade.

The overall website contained multiple independent games.

Examples included:

* Color matching
* Number guessing
* Ordering
* Greeting-related activities
* Tapas matching
* Verb/action games

Each framework produced a different game.

The games could become progressively harder and could be generated for different languages.

---

# 26. The Real Point of the Project

The project is **not primarily about generating HTML games**.

The important lessons are:

### Lesson 1 — Framework similarity

Different frameworks can perform essentially the same agent-loop operations.

### Lesson 2 — Agent loops

A simple to-do list can provide the state and structure needed to create loop-based agent behavior.

### Lesson 3 — Nested loops

A larger agent loop can coordinate multiple smaller agent loops.

### Lesson 4 — Multi-framework collaboration

Different frameworks can participate in one system without requiring A2A.

### Lesson 5 — Tool-driven behavior

Tools and prompts can create behavior that appears autonomous.

---

# 27. Measurable Outcomes Are Important

One of the strongest lessons near the end of the project is the importance of **measurable outcomes**.

Instead of asking another LLM:

```text
"Is this website good?"
```

try to define an objective score.

For example:

```text
Quality score = 73
```

Then the outer loop can use that score as feedback.

---

# 28. Feedback Loops

A more advanced architecture becomes:

```text
Build
  ↓
Test
  ↓
Measure
  ↓
Score
  ↓
Improve
  ↓
Build again
  ↓
Measure again
```

The system can continue iterating until it reaches a target.

This creates an **outer feedback loop around the agent loops**.

Conceptually:

```text
             ┌─────────────────────┐
             │   Outer Feedback    │
             │        Loop         │
             └──────────┬──────────┘
                        ↓
                  Agent System
                        ↓
                    Measure
                        ↓
                     Score
                        │
                        └──────→ Improve
```

This is presented as a path toward getting better and more reliable outcomes.

---

# 29. Week 5 Challenge

The challenge is to take the existing architecture and apply it to a **different problem that matters to you**.

The proposed structure is:

```text
Orchestrator
     ↓
Shared To-Do Board
     ↓
Multiple Specialized Workers
     ↓
Parallel Execution
     ↓
Measurable Outcome
     ↓
Feedback Loop
```

You do not have to use all five frameworks.

Choose the frameworks that are useful for your experiment.

---

# 30. Possible Challenge Structure

Choose a problem such as:

* A business problem
* A personal pain point
* A learning project
* An AI research task
* A commercial objective

Then:

1. Define the overall objective
2. Create an orchestrator
3. Define tasks on a shared board
4. Assign specialized agents
5. Use different frameworks where useful
6. Run agents in parallel
7. Measure the output
8. Feed the measurement back into the system
9. Iterate until the score improves

---

# 31. The Most Important Design Principle

The project strongly reinforces this principle:

> **Build agent architecture around the problem, not around the desire to use more agents.**

Ask:

* Does this task actually need multiple agents?
* Should orchestration be done by code or an LLM?
* Can the outcome be measured?
* Can the agents work independently?
* Is a shared state mechanism useful?
* Does a separate agent improve the result?
* Can the system automatically evaluate its own output?

---

# 32. Key Mental Model

The most useful mental model from this project is:

```text
             Overall Goal
                  ↓
            Orchestrator
                  ↓
        ┌─────────┼─────────┐
        ↓         ↓         ↓
     Worker    Worker    Worker
       Loop      Loop      Loop
        ↓         ↓         ↓
        └──── Shared State ──┘
                  ↓
                QA
                  ↓
              Measure
                  ↓
              Feedback
                  ↓
             Iterate
```

This is essentially:

**agent loop → multi-agent loop → feedback loop**

---

# 33. Week 5 Final Takeaways

* Agent frameworks have different syntax but similar underlying architecture.
* Frameworks are abstractions and utilities, not requirements.
* An LLM plus tools plus a loop can already produce agent-like behavior.
* A shared to-do board can provide simple but powerful coordination.
* Multiple agents can run in parallel against shared state.
* An outer agent loop can manage multiple inner agent loops.
* A2A is not necessary when all agents are controlled by the same application.
* Code-based orchestration is generally more deterministic.
* LLM-based orchestration provides more flexibility but introduces variability.
* Specialized agents can be useful when a narrowly defined task produces better results.
* MCP can extend agents with capabilities such as browser automation.
* Autonomous systems are not automatically reliable; evaluation and feedback are important.
* Measurable outcomes enable automatic iteration.
* The strongest agent systems should be designed around a problem and measurable objective rather than around the number of agents or frameworks.

---

# 34. Week 5 → Week 6

Week 5 concludes with:

* Six agent frameworks studied
* A multi-agent collaboration project
* Nested agent loops
* Shared-state coordination
* Browser-based QA
* Framework interoperability

The course then moves into **Week 6: MCP (Model Context Protocol)**.

## Progress

* **83% complete**
* **17% remaining**
* Next major topic: **MCP**

---

## References

* Google Agent Development Kit (ADK)
* Model Context Protocol (MCP)
* Playwright
* Playwright MCP
* Strands Agents
* Pydantic AI
* Microsoft Agent Framework
* Agno
* Mastra

### External references

* [Google ADK documentation](https://google.github.io/adk-docs/?utm_source=chatgpt.com)
* [Model Context Protocol documentation](https://modelcontextprotocol.io/?utm_source=chatgpt.com)
* [Playwright documentation](https://playwright.dev/?utm_source=chatgpt.com)
* [Playwright MCP on GitHub](https://github.com/microsoft/playwright-mcp?utm_source=chatgpt.com)

The source transcript is preserved as the basis for these notes. 

[1]: https://github.com/microsoft/playwright-mcp?utm_source=chatgpt.com "GitHub - microsoft/playwright-mcp: Playwright MCP server · GitHub"
