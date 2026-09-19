
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
