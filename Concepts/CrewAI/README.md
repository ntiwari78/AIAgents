# CrewAI Week 3 — Days 2–4: Key Points

## 1. The Five-Step CrewAI Project Recipe

The course repeatedly uses the same five-step workflow:

### Step 1 — Create the project

```bash
crewai create crew my_project
```

Then configure the generated project/environment.

### Step 2 — Configure agents and tasks

In the course's classic project structure:

```text
config/
├── agents.yaml
└── tasks.yaml
```

These files contain the agent/task definitions and prompt information.

### Step 3 — Configure `crew.py`

`crew.py` connects:

* Agents
* Tasks
* Tools
* Structured outputs
* The Crew
* Process type

### Step 4 — Configure `main.py`

`main.py` supplies the runtime inputs that replace placeholders such as:

```text
{company}
{current_date}
{sector}
```

### Step 5 — Run

```bash
crewai run
```

Tracing can also be enabled to inspect what actually happened.

> **Core lesson:** Once this recipe becomes familiar, creating new CrewAI projects becomes much faster.

---

# 2. Agents, Tasks and Crews — Refresher

## Agent

An agent contains:

* LLM
* Role
* Goal
* Backstory
* Tools
* Memory

CrewAI uses the combination of **role + goal + backstory** to construct the agent's instructions.

## Task

A task represents a specific piece of work.

It normally defines:

* Description
* Expected output
* Assigned agent
* Context/dependencies
* Optional output file
* Optional structured output

## Crew

A Crew combines agents and tasks to solve a larger problem.

```text
Agents
   +
Tasks
   ↓
Crew
```

The course emphasizes that **task design is particularly important**. A sophisticated agent cannot compensate for poorly defined tasks.

---

# 3. Sequential vs. Hierarchical Processes

## Sequential

Tasks execute according to the defined workflow and dependencies.

```text
Task A
   ↓
Task B
   ↓
Task C
```

This resembles **code-based orchestration**.

It is useful when you want:

* Predictability
* Explicit dependencies
* Repeatable execution
* Easier debugging

## Hierarchical

A manager agent/LLM decides how the work should be delegated and coordinated.

```text
              Manager
             /   |   \
            ↓    ↓    ↓
         Agent Agent Agent
```

This resembles **LLM-based orchestration**.

It provides more autonomy but also introduces more variability.

The current CrewAI documentation continues to support sequential and hierarchical processes.

---

# 4. Context and Task Dependencies

One of the major new concepts is **task context**.

Suppose:

```text
Research Task
      ↓
Analysis Task
```

The analysis task can explicitly specify that it needs the output of the research task as context.

Conceptually:

```text
Research
   |
   | output
   ↓
Analysis
```

This is useful because the second agent can work directly from the first task's results.

---

# 5. Implicit vs. Explicit Context

An important subtlety from the lesson:

If you don't explicitly specify context, CrewAI can make prior task outputs available according to its process behavior.

If you explicitly specify context, you can narrow the context to particular task outputs.

For example:

```text
Research Task
       ↓
       └── context
              ↓
         Analysis Task
```

This is an important **context-engineering** lesson.

More context isn't automatically better.

You want:

> **The right context, at the right time, for the right task.**

The transcript specifically demonstrates inspecting traces to understand what context CrewAI actually sends to the model.

---

# 6. CrewAI Tools

Tools give agents capabilities beyond simply generating text.

Examples include:

* Web search
* APIs
* File operations
* Notifications
* Database access
* Custom application functions

This is one reason CrewAI is described as **"batteries included."**

The agent can reason about a task and then invoke a tool when necessary.

```text
User goal
    ↓
Agent
    ↓
Reasoning
    ↓
Tool call
    ↓
External system
    ↓
Result
    ↓
Agent
```

CrewAI's current documentation describes tools as a core capability for connecting agents to external services and data.

---

# 7. SerperDevTool — Web Search

The financial-research project introduces **SerperDevTool**.

The purpose is to give an agent access to current web-search results.

This solves a major limitation of an LLM-only research agent:

```text
LLM knowledge
      ≠
Current information
```

With web search:

```text
Agent
  ↓
Search tool
  ↓
Current web results
  ↓
Agent analysis
```

The course uses Serper because it provides a Google-search-style API.

Serper currently advertises search, news, images, maps, places, shopping, scholar and other search capabilities.

---

# 8. Current Information Must Be Explicitly Requested

A particularly useful prompt-engineering lesson is to tell the research agent that information must be:

* Current
* Up to date
* Relevant to the current date

For example:

```text
Make sure your information is up to date
and relevant to the current date.
```

This is important because simply giving an agent a research task does not guarantee that it will retrieve fresh information.

The tool provides access to current information; the prompt tells the agent that freshness matters.

---

# 9. Financial Researcher Project

The first major project in the attached content is a **financial researcher**.

Architecture:

```text
                 Company
                    ↓
        ┌───────────┴───────────┐
        ↓                       ↓
   Researcher                Analyst
        ↓                       ↑
        └──── Research ─────────┘
                    ↓
              Final Report
```

There are two agents:

### Senior Financial Researcher

Responsible for:

* Finding current company information
* Searching news
* Identifying relevant developments
* Researching the company's position

### Financial Analyst

Responsible for:

* Reviewing research
* Analyzing the company
* Producing a comprehensive report

---

# 10. Output Files

CrewAI tasks can write their results to files.

For example:

```yaml
output_file: report.md
```

This gives you a persistent artifact rather than only a response printed to the terminal.

Typical workflow:

```text
Agent
 ↓
Task
 ↓
Output
 ↓
Markdown / JSON file
```

---

# 11. CrewAI Tracing

Tracing becomes increasingly important as the projects become more complex.

A trace can reveal:

* Which agent ran
* Which task ran
* Which LLM calls occurred
* Which tools were called
* What prompts were sent
* What context was supplied
* What outputs were generated

This is much more useful than simply looking at the final answer.

### Debugging principle

Don't only inspect:

```text
"What did my agent produce?"
```

Also inspect:

```text
"What did I actually send to the model?"
```

---

# 12. Reading the Trace to Understand CrewAI

The transcript makes an important point:

CrewAI generates additional prompt structure behind the scenes.

Your YAML might contain:

```yaml
role:
goal:
backstory:
```

But the actual model receives a larger system prompt assembled by CrewAI.

Tracing lets you compare:

```text
Your YAML
   ↓
CrewAI processing
   ↓
Generated system prompt
   ↓
LLM
```

This is one of the best ways to understand an abstraction-heavy framework.

---

# 13. Structured Outputs

The Stock Picker project introduces **structured outputs**.

Instead of asking the LLM to return arbitrary text, you define a schema.

The course uses **Pydantic**.

Conceptually:

```python
class TrendingCompany(BaseModel):
    name: str
    ticker: str
    reason: str
```

The model is expected to produce data conforming to that structure.

---

# 14. Why Structured Outputs Matter

Without structured output:

```text
LLM
 ↓
Free-form text
```

With structured output:

```text
LLM
 ↓
Defined schema
 ↓
Validated object
```

Benefits include:

* Predictable data
* Easier downstream processing
* Validation
* JSON serialization
* Easier integration with other software

The course uses Pydantic `BaseModel` classes to define the desired schema.

---

# 15. Pydantic Models Used in the Stock Picker

The project defines structures conceptually similar to:

```text
TrendingCompany
 ├── name
 ├── ticker
 └── reason
```

and:

```text
TrendingCompanyResearch
 ├── name
 ├── market_position
 ├── future_outlook
 └── investment_potential
```

Then lists of these objects are produced.

```text
TrendingCompanyList
        ↓
[Company, Company, Company]

TrendingCompanyResearchList
        ↓
[Research, Research, Research]
```

---

# 16. `output_pydantic`

The key CrewAI mechanism demonstrated is specifying a Pydantic output at the task level.

Conceptually:

```python
output_pydantic=TrendingCompanyList
```

This tells CrewAI what structured result the task should produce.

This is one of the simplest and most useful patterns introduced in the project.

---

# 17. Custom Tools

The lesson then demonstrates creating your own CrewAI tool.

The modern approach shown uses a decorator:

```python
@tool("send_push_notification")
```

The function:

* Has type hints
* Has a descriptive docstring
* Performs the desired action
* Returns a result

Conceptually:

```text
@tool
   ↓
Python function
   ↓
CrewAI tool
   ↓
Agent can invoke it
```

This is very similar conceptually to function tools in other agent frameworks.

---

# 18. Push Notification Tool

The Stock Picker's final agent can send a push notification.

The workflow becomes:

```text
Research
   ↓
Analyze
   ↓
Select company
   ↓
Send notification
   ↓
Generate report
```

This demonstrates that agent tools don't have to be information-retrieval tools.

They can also **perform actions**.

---

# 19. Stock Picker Architecture

The project contains three main agents:

### 1. Trending Company Finder

Finds companies currently attracting attention.

### 2. Financial Researcher

Performs detailed research on those companies.

### 3. Stock Picker

Selects the company it considers the best candidate based on the research.

Overall:

```text
                Sector
                  ↓
       Trending Company Finder
                  ↓
          Trending Companies
                  ↓
        Financial Researcher
                  ↓
          Research Results
                  ↓
             Stock Picker
              /       \
             ↓         ↓
        Push Alert   Report
```

---

# 20. Memory

The Stock Picker then introduces CrewAI memory.

The course presents a simplified approach:

```text
memory = True
```

The idea is that agents can retain/retrieve relevant information from previous interactions.

Conceptually:

```text
Past interactions
       ↓
     Memory
       ↓
     Retrieval
       ↓
     Agent context
```

---

# 21. Memory Is Essentially Retrieved Context

A particularly important conceptual insight from the transcript is that memory becomes useful because information from previous interactions can be retrieved and supplied to the agent.

The trace showed CrewAI adding a section containing memories from previous interactions.

This makes memory closely related to:

* Retrieval
* RAG
* Context engineering
* Tool use

The course describes this as effectively giving the agent retrieval capabilities.

---

# 22. Memory: Benefit vs. Cost

### Benefit

Very easy to activate.

```python
memory=True
```

### Cost

The framework hides much of the implementation.

You may not immediately know:

* What was retrieved
* Why it was retrieved
* How it was ranked
* Exactly how it entered the prompt

Therefore:

> **Use tracing to understand memory behavior.**

This is a recurring theme throughout the course.

---

# 23. Hierarchical Stock Picker

The project is then switched from:

```text
process = sequential
```

to:

```text
process = hierarchical
```

A manager agent is introduced.

Architecture:

```text
                 Manager
                    |
       +------------+------------+
       |            |            |
       ↓            ↓            ↓
    Finder      Researcher   Stock Picker
```

The manager decides how the tasks should be coordinated.

---

# 24. Manager Agent

The manager is given its own:

* Role
* Goal
* LLM
* Delegation capability

The goal is to coordinate the other agents toward the overall investment-selection objective.

This demonstrates **LLM-based orchestration** in practice.

---

# 25. `allow_delegation`

The manager agent is configured to delegate work.

Conceptually:

```text
Manager
  ↓
Decides what needs doing
  ↓
Delegates
  ↓
Specialist agents
```

This is a major difference from a purely sequential workflow.

---

# 26. Sequential vs. Hierarchical — Practical Difference

| Sequential             | Hierarchical              |
| ---------------------- | ------------------------- |
| Explicit order         | Manager chooses           |
| More deterministic     | More autonomous           |
| Easier to reason about | More adaptive             |
| Easier debugging       | More complex debugging    |
| Code-driven            | LLM-driven                |
| Predictable            | Potentially more flexible |

A good engineering decision is not:

> "Hierarchical is better."

Instead:

> Choose the simplest orchestration strategy that reliably solves the problem.

---

# 27. MCP

The attached material also discusses **Model Context Protocol (MCP)** in the broader agent/tool context.

MCP provides a standardized way for AI applications to connect models with external tools and data sources. The protocol uses a client/server architecture and supports capabilities such as tools and resources.

The course demonstrates the idea of connecting agents to external documentation through an MCP server such as **Context7**.

Conceptually:

```text
Agent
  ↓
MCP client
  ↓
MCP server
  ↓
Documentation / external data
  ↓
Agent
```

The major lesson is:

> Instead of implementing every integration yourself, an agent can consume capabilities exposed through a standardized protocol.

---

# 28. Developer / Engineering Agent

The later part of the attached content moves toward a more ambitious **developer/engineering team**.

The goal is to have agents work on software requirements.

The architecture introduces specialized roles such as:

* Engineering lead
* Front-end engineer
* Other implementation/review roles

The agents are given tools for:

* Reading files
* Writing files
* Executing code
* Working in a sandbox
* Consulting documentation

---

# 29. Sandboxed Code Execution

A major concept is giving coding agents a controlled workspace.

Instead of letting an agent execute arbitrary code on the host machine, the course builds tools around a dedicated sandbox directory.

Conceptually:

```text
Engineering Agent
       ↓
Sandbox tools
       ↓
Sandbox directory
       ↓
Code / files
       ↓
Execution
```

This allows the agent to:

* Create files
* Modify files
* Inspect files
* Execute code
* Build software iteratively

The transcript explicitly treats sandboxing as a way to let coding agents work on a project while keeping the work isolated.

---

# 30. Why Sandboxing Matters

Code-writing agents are fundamentally different from agents that only return text.

A coding agent may need to:

```text
Plan
 ↓
Write code
 ↓
Run code
 ↓
Observe error
 ↓
Modify code
 ↓
Run again
 ↓
Test
 ↓
Fix
```

This creates an iterative software-engineering loop.

---

# 31. UV Projects Inside the Sandbox

The developer project extends the sandbox so it can contain its own UV project.

Conceptually:

```text
sandbox/
├── pyproject.toml
├── source files
├── tests
└── application files
```

The project can then install dependencies such as Gradio into the isolated environment.

This is a useful pattern for autonomous coding agents.

---

# 32. Context7 / Documentation Retrieval

The developer-agent project also demonstrates giving coding agents access to current documentation.

Instead of relying solely on model training data:

```text
Agent
 ↓
Documentation tool
 ↓
Current API information
 ↓
Agent
```

This is especially valuable for rapidly changing libraries.

The attached lesson uses MCP to make current documentation available to the agent.

---

# 33. Context Engineering

A major theme running through these projects is **context engineering**.

The agent's performance depends on more than the user prompt.

Relevant context can include:

```text
System instructions
+
Task
+
Tools
+
Tool results
+
Structured-output schema
+
RAG
+
Short-term memory
+
Long-term memory
+
Previous task outputs
```

Therefore:

> **Building an effective agent is largely about engineering the context that reaches the model.**

---

# 34. Agentic RAG

The lesson connects tools and RAG.

Traditional RAG:

```text
Retrieve information
       ↓
Put information into prompt
       ↓
LLM
```

Agentic RAG:

```text
Agent
  ↓
Decides what information it needs
  ↓
Calls retrieval/search tool
  ↓
Receives result
  ↓
Continues reasoning
```

The second approach gives the agent more control over information retrieval.

---

# 35. Tracing Is Also a Learning Tool

Tracing isn't merely for production monitoring.

During development it helps you understand:

* How prompts are assembled
* How tools are selected
* How context is passed
* How memory is retrieved
* How agents delegate
* How outputs are generated

The course repeatedly recommends comparing:

```text
What I configured
        vs.
What the LLM actually received
```

This is arguably one of the most valuable learning techniques in the entire lesson.

---

# 36. Critical Warning — Don't Trust the Stock Picker

The instructor explicitly warns against using an LLM-generated stock recommendation as an actual investment decision.

The project is a **demonstration of CrewAI capabilities**, not a validated investment system.

Why?

Because:

```text
LLM-generated analysis
        ≠
Reliable financial prediction
```

The project needs:

* Evaluation
* Historical testing
* Feedback loops
* Performance measurement
* Robust data validation

before it could responsibly support real decisions.

---

# 37. Start Simple

Another important engineering lesson:

> **Don't create multiple agents simply because multiple agents sound impressive.**

Start with:

```text
1 Agent
+
1 Task
```

Then ask:

> Does splitting the problem into multiple agents measurably improve the result?

Only add additional agents when there is a real benefit.

This prevents unnecessary complexity.

---

# 38. Overall Architecture Learned in These Lessons

The progression can be visualized as:

```text
                CrewAI
                   |
        +----------+----------+
        |                     |
      Agents                Tasks
        |                     |
   Role/Goal/            Description/
   Backstory             Expected Output
        |                     |
        +----------+----------+
                   |
                  Crew
                   |
          +--------+--------+
          |                 |
      Sequential        Hierarchical
          |                 |
       Explicit          Manager
       workflow            LLM
          |
          +-----------------------+
                                  |
                              Capabilities
                                  |
             +--------------------+-------------------+
             |          |          |        |         |
           Tools      Memory     RAG     MCP      Sandbox
             |
       +-----+-----+
       |           |
   Built-in      Custom
     tools        tools
```

---

# 39. Most Important Takeaways

### 1. Learn the five-step CrewAI workflow

```text
Create
 ↓
Configure YAML
 ↓
Configure crew.py
 ↓
Configure main.py
 ↓
Run
```

### 2. Tools give agents capabilities

Without tools:

```text
Agent → Text
```

With tools:

```text
Agent → Action → External system → Result
```

### 3. Context controls agent behavior

The quality of the context is often more important than simply making prompts longer.

### 4. Structured outputs make agents usable by software

Use Pydantic schemas when downstream code needs reliable structure.

### 5. Memory is retrieved context

Memory is powerful, but inspect what the framework actually retrieves.

### 6. Hierarchical orchestration introduces autonomy

A manager LLM can decide how specialist agents should work.

### 7. MCP standardizes external integrations

It allows agents to consume tools/data exposed by MCP servers.

### 8. Coding agents need execution environments

A sandbox gives an agent somewhere to write, run and test code.

### 9. Tracing is essential

Always inspect what actually happened.

### 10. Evaluate before trusting

A working demo is not necessarily a reliable production system.

---

# 40. Quick Revision Cheat Sheet

| Concept             | Meaning                                       |
| ------------------- | --------------------------------------------- |
| Agent               | LLM-powered specialist                        |
| Task                | Specific assignment                           |
| Crew                | Collection of agents + tasks                  |
| Sequential          | Explicit workflow                             |
| Hierarchical        | Manager-driven workflow                       |
| Context             | Information supplied to a task                |
| Tool                | Capability an agent can invoke                |
| Custom Tool         | Your own Python-based capability              |
| Serper              | Web-search API used in the project            |
| Structured Output   | Schema-constrained model output               |
| Pydantic            | Schema/model definition                       |
| Memory              | Retrieved information from prior interactions |
| MCP                 | Standard protocol for tools/context           |
| Sandbox             | Controlled coding/execution environment       |
| Tracing             | Visibility into execution                     |
| Agentic RAG         | Agent-controlled retrieval                    |
| Context Engineering | Designing the information supplied to the LLM |

---

# 41. Suggested Hands-On Exercises

## Exercise 1 — Add a Tool

Create a custom tool that:

```text
Input → company name
Output → company information
```

Give it to a research agent.

## Exercise 2 — Experiment with Context

Build:

```text
Research → Analysis
```

Run it once with implicit context and once with explicit context.

Inspect the traces.

## Exercise 3 — Structured Output

Create:

```python
class Company(BaseModel):
    name: str
    ticker: str
    reason: str
```

Use it as a task output.

## Exercise 4 — Build a Notification Tool

Create:

```text
@tool("notify_user")
```

and connect it to your final agent.

## Exercise 5 — Sequential vs. Hierarchical

Run the same workflow in both modes.

Compare:

* Number of LLM calls
* Execution order
* Results
* Cost
* Predictability

## Exercise 6 — Memory Experiment

Run the same Crew multiple times with memory enabled.

Inspect the trace to determine:

* What was remembered?
* What was retrieved?
* How was it inserted into the prompt?

## Exercise 7 — MCP Experiment

Connect an agent to an MCP documentation server and ask it about a library whose API changes frequently.

Compare the result with an agent that has no documentation access.

## Exercise 8 — Coding Agent

Build a simple engineering Crew:

```text
Planner
   ↓
Coder
   ↓
Tester
   ↓
Reviewer
```

Give the coder and tester access to a sandbox.

---

# 42. Final Mental Model

The most important conceptual progression is:

```text
Basic Agent
     ↓
Agent + Task
     ↓
Multiple Agents + Tasks
     ↓
Crew
     ↓
Crew + Context
     ↓
Crew + Tools
     ↓
Crew + Structured Outputs
     ↓
Crew + Memory
     ↓
Crew + Hierarchical Manager
     ↓
Crew + MCP
     ↓
Crew + Sandbox
     ↓
Production Agentic System
```

The overarching lesson is that **CrewAI abstracts away a large amount of agent infrastructure**.

That is its strength:

> You can build sophisticated multi-agent systems relatively quickly.

But it is also its weakness:

> The more the framework handles automatically, the more important tracing and observability become.

For practical development, remember this loop:

```text
Design
  ↓
Run
  ↓
Trace
  ↓
Inspect
  ↓
Evaluate
  ↓
Improve
```

That is the core engineering mindset behind the projects in this section.
