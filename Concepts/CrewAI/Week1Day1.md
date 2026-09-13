# Week 3 Day 1 — CrewAI: Key Points & Deep Dive

## 1. What Is CrewAI?

**CrewAI** is a Python framework for orchestrating multiple AI agents.

The course positions it in the progression:

1. **No framework**
2. **OpenAI Agents SDK** — lightweight and less opinionated
3. **CrewAI** — more opinionated, simple, and batteries-included
4. **LangGraph** — more sophisticated and flexible, covered later

The key philosophy of CrewAI is to make multi-agent systems relatively easy to build by providing higher-level abstractions.

---

## 2. CrewAI's Product Landscape

The transcript distinguishes several CrewAI offerings:

### CrewAI Open Source

This is the framework being used in the course.

It provides the building blocks for creating and orchestrating AI agents.

### CrewAI AMP / Enterprise

A platform for deploying, running, and monitoring agents in production.

### CrewAI Studio

A low-code/no-code environment for designing agent workflows.

### Important distinction

For this course, the important component is:

> **CrewAI Open Source**

The other products are surrounding commercial offerings.

---

# 3. Crews vs. Flows

CrewAI has two major constructs.

## Crews

A **Crew** is a team of agents working together to accomplish a goal.

The course describes a Crew as:

> A collection of agents and tasks.

Crews emphasize:

* Agent collaboration
* Role specialization
* Autonomous behavior
* Task execution
* Multi-agent problem solving

### Mental model

```text
Crew
 ├── Agent
 ├── Agent
 ├── Agent
 └── Tasks
```

---

## Flows

**Flows** provide a more structured orchestration layer.

They define:

* Steps
* Logic
* State
* Data movement
* Higher-level workflow orchestration

A useful mental model is:

```text
Flow
 ├── Crew A
 ├── Crew B
 ├── Crew C
 └── Workflow logic
```

The transcript emphasizes that **Crews are currently the primary focus**, while Flows provide a higher-level backbone for combining workflows.

---

# 4. The Four Core CrewAI Concepts

The most important concepts introduced are:

1. **Agent**
2. **Task**
3. **Crew**
4. **Process**

---

## 5. Agent

An Agent consists conceptually of:

* An LLM
* Instructions
* Tools
* Memory/context

However, CrewAI is more opinionated about how an agent is described.

Instead of directly writing one large system prompt, CrewAI emphasizes three fields:

### Role

What is the agent's purpose?

Example:

```text
You are a compelling debater.
```

### Goal

What should the agent accomplish?

Example:

```text
Present a clear and convincing argument.
```

### Backstory

What background or context should the agent have?

Example:

```text
You are an experienced debater with a reputation
for concise and persuasive arguments.
```

### Mental model

```text
Agent
 ├── Role
 ├── Goal
 ├── Backstory
 ├── LLM
 ├── Tools
 └── Memory
```

CrewAI uses these pieces to construct the instructions given to the LLM.

### Advantage

You get a structured, opinionated approach to agent design.

### Trade-off

You have less direct control over the exact system prompt than when manually writing the prompt yourself.

---

# 6. Task

A **Task** represents something that needs to be done.

A task typically contains:

* Description
* Expected output
* Assigned agent
* Optional output file
* Dependencies/context

For example:

```text
Task:
Research the most popular AI agent frameworks.

Agent:
Researcher

Expected output:
A detailed research report.
```

### Important lesson

The instructor emphasizes that **tasks are extremely important**.

Good agents with poorly designed tasks can still produce poor results.

Therefore:

> Spend significant effort designing clear tasks.

---

# 7. Crew

A Crew combines:

```text
Agents + Tasks
```

For example:

```text
Crew
│
├── Researcher
│
├── Writer
│
└── Tasks
    ├── Research
    └── Write report
```

The Crew determines how those agents and tasks work together.

---

# 8. Sequential vs. Hierarchical Processes

CrewAI supports two important process styles introduced in the lesson.

## Sequential

Tasks execute in an explicitly defined sequence.

```text
Task 1
   ↓
Task 2
   ↓
Task 3
```

This closely resembles **code-based orchestration**.

The developer determines the workflow.

---

## Hierarchical

A manager LLM decides how tasks should be coordinated.

```text
              Manager LLM
             /     |      \
            ↓      ↓       ↓
         Agent A Agent B Agent C
```

This is conceptually similar to **LLM-based orchestration**.

### Connection to Week 2

This is one of the most important conceptual links to remember:

| Approach          | Orchestration                |
| ----------------- | ---------------------------- |
| Sequential Crew   | Code-defined workflow        |
| Hierarchical Crew | LLM/manager-defined workflow |

This connects directly to the previous week's discussion of **orchestration by code vs. orchestration by LLM**.

---

# 9. Why CrewAI Is "Opinionated"

Compared with a lower-level framework, CrewAI makes more decisions for you.

Examples include:

* How agents are described
* How tasks are structured
* How projects are organized
* How configuration is separated
* How agents collaborate
* How tasks are executed

### Benefit

You can get a multi-agent system running quickly.

### Cost

You give up some low-level control.

This is the classic framework trade-off:

```text
More abstraction
       ↓
Easier development
       ↓
Less direct control
```

---

# 10. YAML Configuration

One of CrewAI's major conveniences is separating configuration from Python code.

The course demonstrates:

```text
agents.yaml
tasks.yaml
```

The YAML files contain agent/task definitions while Python contains the application logic.

This gives a clean separation:

```text
Configuration
     ↓
YAML
     ↓
Agents + Tasks

Application logic
     ↓
Python
     ↓
Crew execution
```

---

# 11. Typical CrewAI Project Structure

The course demonstrates the classic project structure:

```text
project/
│
├── src/
│   └── project/
│       ├── config/
│       │   ├── agents.yaml
│       │   └── tasks.yaml
│       │
│       ├── crew.py
│       └── main.py
│
└── ...
```

The four files to remember are:

| File          | Purpose                                    |
| ------------- | ------------------------------------------ |
| `agents.yaml` | Agent definitions                          |
| `tasks.yaml`  | Task definitions                           |
| `crew.py`     | Connects agents/tasks to the Crew          |
| `main.py`     | Starts the application and supplies inputs |

### Important current-version note

The transcript teaches the **classic YAML-based project structure**. Current CrewAI documentation also supports a newer JSON-first scaffold, while explicitly retaining the classic YAML scaffold as an option. So don't be surprised if a current installation looks different from the course project.

---

# 12. `agents.yaml`

This file defines agents.

Conceptually:

```yaml
researcher:
  role: >
    Researcher

  goal: >
    Find useful information about the topic.

  backstory: >
    You are an experienced researcher.

  llm: ...
```

The important fields introduced in the lesson are:

* `role`
* `goal`
* `backstory`
* `llm`

### YAML indentation matters

YAML is whitespace-sensitive.

A misplaced indentation can cause confusing errors.

For example, fields such as:

```yaml
role:
goal:
backstory:
llm:
```

must belong to the same indentation level.

---

# 13. `tasks.yaml`

This file defines the work that agents perform.

A task contains things such as:

```yaml
research_task:
  description: >
    Research the topic thoroughly.

  expected_output: >
    A detailed research report.

  agent: researcher
```

Tasks can also specify output files.

For example:

```yaml
output_file: report.md
```

This allows a task to automatically write its result to a file.

---

# 14. `crew.py`

`crew.py` connects the configuration to actual CrewAI objects.

The generated Python functions associate Python methods with the corresponding YAML definitions.

Conceptually:

```text
agents.yaml
     ↓
crew.py
     ↓
Agent objects

tasks.yaml
     ↓
crew.py
     ↓
Task objects
```

The names used in Python must correspond correctly to the names in the YAML configuration.

---

# 15. `main.py`

`main.py` is where the application is started.

It can also collect user input.

For example:

```python
motion = input("Enter the motion: ")
```

That value can then be passed into the Crew.

This enables dynamic prompts using placeholders such as:

```text
{motion}
```

The value is supplied through the input dictionary.

---

# 16. Dynamic Inputs

A powerful pattern demonstrated in the debate example is:

```text
User input
    ↓
main.py
    ↓
inputs dictionary
    ↓
Crew kickoff
    ↓
YAML placeholders
    ↓
Agent/task prompts
```

For example:

```python
inputs = {
    "motion": motion
}
```

Then YAML can reference:

```text
{motion}
```

This makes the same Crew reusable for different inputs.

---

# 17. LiteLLM Integration

CrewAI uses **LiteLLM** to make it easier to work with many different model providers.

The advantage is that you can switch models without redesigning the entire Crew.

LiteLLM provides a common interface across many LLM providers and also supports features such as routing, fallbacks, and cost tracking.

### Mental model

```text
                 CrewAI
                    |
                 LiteLLM
          /    /    |    \    \
       OpenAI Claude Gemini  ...
```

This is particularly useful when experimenting with different models.

---

# 18. Environment Variables

API keys are stored in environment variables, commonly through:

```text
.env
```

The course highlights several configuration problems that can occur.

### Important lesson

Make sure you know **which `.env` file is actually being loaded**.

The generated project may create an additional `.env` inside the project directory.

According to the course, this can override or interfere with the parent `.env`.

---

# 19. Environment Variable Gotcha

The transcript highlights a specific Google/Gemini naming issue:

```text
GEMINI_API_KEY
```

rather than assuming a generic Google API key variable.

When using a framework, always check the framework's expected environment-variable names.

---

# 20. Another `.env` Gotcha

The transcript also discusses global environment variables.

If a machine already has something like:

```text
OPENAI_API_KEY
```

defined globally, it can potentially take precedence over the value expected from the project's `.env` configuration.

This can create confusing authentication errors.

### Debugging principle

When API-key errors don't make sense:

1. Check the project's `.env`
2. Check for nested `.env` files
3. Check shell environment variables
4. Check global/system environment variables
5. Confirm which value the application is actually reading

---

# 21. Installing CrewAI

The course uses **UV** for installation.

The demonstrated approach is:

```bash
uv tool install crewai
```

The instructor also demonstrates pinning a specific version for reproducibility.

### Why version pinning matters

Frameworks evolve quickly and can introduce breaking changes.

For a course or production environment, pinning versions can make the environment reproducible.

However, the specific version used in the lesson should be treated as **course-specific**, not as the current recommended version.

The current official repository documents installation through the CrewAI CLI and current project creation workflow.

---

# 22. Creating a CrewAI Project

The course uses:

```bash
crewai create crew <project_name>
```

This creates the project scaffolding.

The current CrewAI documentation still documents this command, although the generated structure has evolved. The current scaffold can use JSONC configuration, while the older/classic structure can be generated with the classic option.

---

# 23. Running a Crew

The course demonstrates:

```bash
crewai run
```

This starts the Crew.

The current documentation also uses `crewai run` after project setup.

---

# 24. First Example — Researcher

The first project is a simple research Crew.

Conceptually:

```text
Researcher Agent
       ↓
Research Task
       ↓
Reporting Analyst
       ↓
Report
```

The important lesson is that the generated CrewAI project already gives you a working starting point.

You then customize:

* Agents
* Tasks
* Model
* Input
* Output

---

# 25. Important Limitation of the First Example

The course makes an important observation:

> The basic researcher example was not actually performing web search.

It was using the LLM's existing knowledge.

This distinction is extremely important.

```text
LLM knowledge
      ≠
Live web research
```

If you want current information, you need to provide appropriate tools or external data sources.

---

# 26. Second Example — Debate Crew

The second project demonstrates multi-agent collaboration through a debate.

There are two agents:

```text
Debater
Judge
```

And three tasks:

```text
Propose
Oppose
Decide
```

The overall architecture is:

```text
             Motion
               |
        +------+------+
        |             |
        ↓             ↓
     Propose        Oppose
        |             |
        +------+------+
               |
               ↓
             Judge
               |
               ↓
            Decision
```

---

# 27. The Debate Agent

The debater is given:

### Role

Compelling debater

### Goal

Present a clear argument for or against the motion.

### Backstory

An experienced debater capable of concise and convincing arguments.

This demonstrates how CrewAI's `role`, `goal`, and `backstory` structure an agent.

---

# 28. The Judge Agent

The judge's purpose is to evaluate the arguments.

Its goal is essentially:

```text
Given the arguments for and against,
determine which side is more convincing.
```

The judge therefore acts as a separate evaluation agent.

---

# 29. Task Dependencies and Context

One of the most important demonstrations is how the judge receives the results of the earlier tasks.

The sequence is:

```text
Propose
   ↓
Oppose
   ↓
Decide
```

The judge receives the previous task outputs as context.

The instructor verifies this using tracing rather than simply assuming it happened.

---

# 30. Tracing

Tracing lets you inspect what happened inside the Crew.

You can examine:

* LLM calls
* Tasks
* Prompts
* Inputs
* Outputs
* Context passed between tasks

This is extremely valuable for debugging agentic systems.

### Key lesson

Don't just ask:

> "Did the agent do what I expected?"

Inspect the actual execution trace.

---

# 31. Output Files

Tasks can specify output files.

For example:

```text
propose.md
oppose.md
decide.md
```

The resulting structure might look like:

```text
output/
├── propose.md
├── oppose.md
└── decide.md
```

This makes CrewAI useful for applications where agent outputs need to become persistent artifacts.

---

# 32. Why YAML Separation Is Valuable

One of the strongest design ideas demonstrated is:

```text
Prompts / Configuration
        ↓
      YAML

Execution Logic
        ↓
      Python
```

This makes it easier to:

* Modify prompts
* Experiment with agent roles
* Review configurations
* Keep Python code cleaner
* Collaborate with others
* Version-control configuration separately

---

# 33. CrewAI vs. OpenAI Agents SDK

The lesson frames the difference approximately as:

| OpenAI Agents SDK           | CrewAI                            |
| --------------------------- | --------------------------------- |
| Lightweight                 | More opinionated                  |
| Direct instructions         | Role + Goal + Backstory           |
| Less prescribed structure   | Strong project structure          |
| Flexible                    | Batteries included                |
| Agents/tools/handoffs       | Agents/tasks/crews                |
| Code-oriented orchestration | Sequential/hierarchical processes |

Neither approach is inherently "better."

The trade-off is:

```text
OpenAI Agents SDK
      ↓
More direct control

CrewAI
      ↓
More built-in structure
```

---

# 34. Key Mental Model

A useful way to remember CrewAI is:

```text
AGENT
= Who am I?

TASK
= What do I need to do?

CREW
= Which agents work together?

PROCESS
= How are the tasks coordinated?

FLOW
= How do larger workflows connect?

LLM
= Which model powers the agents?
```

---

# 35. Most Important Takeaways

## 1. CrewAI is opinionated

It deliberately provides abstractions and conventions to make multi-agent development easier.

## 2. Agents have structured identities

Think:

```text
Role + Goal + Backstory
```

## 3. Tasks are critical

Good task design can matter more than excessively detailed agent descriptions.

## 4. Crews combine agents and tasks

```text
Crew = Agents + Tasks
```

## 5. Sequential and hierarchical processes map to two orchestration styles

```text
Sequential → explicit workflow
Hierarchical → manager-driven workflow
```

## 6. YAML separates configuration from implementation

```text
agents.yaml
tasks.yaml
      ↓
   crew.py
      ↓
   main.py
```

## 7. LiteLLM makes model switching easier

You can work with many model providers through a common interface.

## 8. Tracing is essential

Use traces to understand what agents actually received and produced.

## 9. Framework versions change quickly

The exact commands/project structure shown in a course can become outdated. Always check the current official documentation when starting a new project.

---

# 36. Quick Revision Cheat Sheet

| Concept       | Remember                            |
| ------------- | ----------------------------------- |
| CrewAI        | Multi-agent orchestration framework |
| Agent         | Role + Goal + Backstory + LLM/tools |
| Task          | Specific piece of work              |
| Crew          | Agents + Tasks                      |
| Sequential    | Predetermined execution order       |
| Hierarchical  | Manager LLM coordinates execution   |
| Flow          | Higher-level structured workflow    |
| `agents.yaml` | Agent configuration                 |
| `tasks.yaml`  | Task configuration                  |
| `crew.py`     | Connects configuration to Crew      |
| `main.py`     | Starts execution / supplies inputs  |
| LiteLLM       | Multi-provider LLM abstraction      |
| `.env`        | Environment/API-key configuration   |
| Tracing       | Inspect agent execution             |
| `output_file` | Persist task results                |

---

# 37. Suggested Hands-On Exercises

### Exercise 1 — Modify the Researcher

Change the research topic from AI agent frameworks to:

```text
The future of autonomous software development
```

Observe how the output changes.

### Exercise 2 — Change the Agent

Modify:

```text
role
goal
backstory
```

and compare the resulting behavior.

### Exercise 3 — Experiment with Models

Run the same Crew using different supported LLMs.

Compare:

* Quality
* Speed
* Cost
* Reasoning
* Consistency

### Exercise 4 — Build the Debate Crew

Create:

```text
Debater
Judge
```

with:

```text
Propose
Oppose
Decide
```

### Exercise 5 — Inspect Traces

Run the debate and inspect:

1. Propose LLM call
2. Oppose LLM call
3. Decide LLM call
4. Context passed to the judge

### Exercise 6 — Add Another Agent

Extend the debate with a:

```text
Fact Checker
```

Possible workflow:

```text
Propose
   ↓
Oppose
   ↓
Fact Check
   ↓
Judge
```

### Exercise 7 — Build a Different Crew

Try:

```text
Researcher
Analyst
Writer
Reviewer
```

with:

```text
Research
   ↓
Analyze
   ↓
Write
   ↓
Review
```

This is a good bridge from the Deep Research project in Week 2 to CrewAI.

---

# 38. Recommended Deep-Dive Resources

## Official CrewAI Documentation

[CrewAI official documentation](https://docs.crewai.com/?utm_source=chatgpt.com)

Start here for the current API, agents, tasks, crews, flows, tools, memory, and project structure.

## CrewAI GitHub Repository

[CrewAI GitHub repository](https://github.com/crewAIInc/crewAI?utm_source=chatgpt.com)

Useful for examining the framework itself, examples, releases, issues, and current project structure.

## CrewAI GitHub Organization

[CrewAI GitHub organization](https://github.com/crewAIInc?utm_source=chatgpt.com)

Useful for exploring the official examples and related CrewAI repositories.

## LiteLLM Documentation

[LiteLLM official documentation](https://docs.litellm.ai/?utm_source=chatgpt.com)

Particularly useful for understanding how CrewAI can work across different LLM providers and how routing/fallbacks work.

---

# 39. Important Current-vs-Course Note

The uploaded lesson demonstrates a **classic CrewAI project structure** involving:

```text
agents.yaml
tasks.yaml
crew.py
main.py
```

CrewAI's current documentation has evolved and now documents a JSON-first project scaffold as well, while still providing a classic option. Therefore, if you install CrewAI today and see a somewhat different directory structure, that does **not necessarily mean the course is wrong**—the framework has changed since the lesson was recorded.

When learning from this course, focus first on the **concepts**:

```text
Agent
Task
Crew
Process
Flow
Orchestration
Context
Tracing
```

Then map those concepts onto the current API.

---

# 40. Final Mental Model

The entire lesson can be compressed into:

```text
                   CREWAI
                      |
          +-----------+-----------+
          |                       |
        CREWS                   FLOWS
          |                       |
   Agent collaboration      Structured workflow
          |
     +----+----+
     |         |
   Agents    Tasks
     |         |
 Role/Goal   Description
 Backstory   Output
 LLM         Context
     |
     +----------------+
                      |
                PROCESS
               /       \
        Sequential    Hierarchical
            |             |
        Code-like       Manager LLM
        workflow        orchestration
```

The most important conceptual transition from Week 2 is:

> **CrewAI packages multi-agent orchestration into a more opinionated, structured framework, making it easier to build collaborative agents while giving up some of the low-level control of a lighter framework.**

And the most important practical skill from Day 1 is:

> **Learn to design good agents and tasks, separate configuration from code, run the Crew, and inspect traces to understand what actually happened.**

---

## References

1. [CrewAI official documentation](https://docs.crewai.com/?utm_source=chatgpt.com)
2. [CrewAI GitHub repository](https://github.com/crewAIInc/crewAI?utm_source=chatgpt.com)
3. [CrewAI GitHub organization](https://github.com/crewAIInc?utm_source=chatgpt.com)
4. [LiteLLM documentation](https://docs.litellm.ai/?utm_source=chatgpt.com)
5. Uploaded course transcript — Week 3, Day 1: CrewAI.
