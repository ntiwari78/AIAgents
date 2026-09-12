# Week 2 Day 1 — OpenAI Agents SDK: Key Points

## 1. OpenAI Agents SDK

* The **OpenAI Agents SDK** is an open-source framework for building agentic AI applications.
* It is designed to be **lightweight and code-first**, with relatively few abstractions.
* It supports **Python and TypeScript**.
* The framework is intended to work with different model providers rather than being limited conceptually to OpenAI models.
* It evolved from the earlier experimental **Swarm** project.
* The main goal is to make common agent patterns easier without hiding too much of the underlying implementation.

**Core idea:**

> An agent is essentially an LLM configured with instructions, tools, and related runtime behavior.

[Official OpenAI Agents SDK documentation](https://openai.github.io/openai-agents-python/)

[Official Python SDK repository](https://github.com/openai/openai-agents-python)

---

## 2. Async Python (`asyncio`)

Async Python is important because agent applications frequently wait for external services such as LLM APIs.

### The two basic rules

```python
async def my_function():
    ...
```

and:

```python
result = await my_function()
```

* `async def` defines a **coroutine**.
* Calling a coroutine does not immediately execute it; it produces a coroutine object.
* `await` allows the coroutine to execute through Python's asynchronous event loop.

### Event loop

* The **event loop** manages asynchronous tasks.
* A coroutine runs until it reaches an `await`.
* At that point, control can be given to another coroutine.
* This is known as **cooperative multitasking**.
* Unlike traditional multiprocessing or multithreading, asyncio does not mean that multiple Python coroutines are literally executing simultaneously on different CPU threads.

### `asyncio.gather`

When several independent asynchronous operations should progress together:

```python
await asyncio.gather(
    task_one(),
    task_two(),
    task_three()
)
```

This is particularly useful when several operations are **I/O-bound**, such as multiple API or LLM calls.

### When asyncio is useful

* Excellent for **I/O-bound workloads**.
* Useful when applications spend significant time waiting for network responses.
* Lightweight compared with managing large numbers of threads or processes.
* Less suitable for heavily **CPU-bound** computation.

[Python `asyncio` documentation](https://docs.python.org/3/library/asyncio.html)

[Python Coroutines and Tasks](https://docs.python.org/3/library/asyncio-task.html)

---

## 3. Core OpenAI Agents SDK Concepts

The lesson introduces three important concepts:

### Agent

An `Agent` combines:

* An LLM/model
* Instructions/system prompt
* Tools
* Optional capabilities such as guardrails and handoffs

Conceptually:

```text
Agent = Model + Instructions + Tools
```

The recommendation is to **start with one agent** and introduce multiple agents only when doing so clearly improves the solution.

[Agents documentation](https://openai.github.io/openai-agents-python/agents/)

---

### Agents as Tools

One agent can expose another agent as a callable tool.

This provides a lightweight mechanism for agents to collaborate without requiring a complicated multi-agent architecture.

---

### Handoffs

**Handoffs** allow an agent to transfer responsibility to another specialized agent.

The lesson emphasizes that these mechanisms are ultimately built around relatively simple LLM interactions rather than mysterious autonomous behavior.

---

### Guardrails

Guardrails provide validation and control around agent inputs and outputs.

They are intended to address one of the major concerns with agent systems:

> **How do we make agent behavior more reliable and controllable?**

[OpenAI Agents SDK guardrails and orchestration documentation](https://openai.github.io/openai-agents-python/)

---

## 4. The Basic Agent Execution Model

The lesson presents three fundamental steps:

### Step 1 — Create an Agent

```python
agent = Agent(
    name="Jokester",
    instructions="Tell funny jokes.",
    model="..."
)
```

The `instructions` parameter effectively serves as the agent's system prompt.

### Step 2 — Add Tracing

Tracing provides visibility into what the agent is doing.

```python
with trace("My workflow"):
    ...
```

### Step 3 — Run the Agent

```python
result = await Runner.run(
    agent,
    "Tell me a joke about AI agents."
)
```

The runner manages the agent loop and returns the result.

[OpenAI Agents SDK Quickstart](https://openai.github.io/openai-agents-python/quickstart/)

---

## 5. `Runner.run()` and the Agent Loop

`Runner.run()` is responsible for executing an agent task.

An application-level turn can involve:

1. Sending a prompt to the model
2. Receiving a response
3. Calling tools if necessary
4. Feeding tool results back
5. Continuing the loop
6. Stopping when the task is complete

This hides much of the repetitive orchestration code that would otherwise have to be written manually.

---

## 6. Getting Agent Results

Two useful result properties/methods are highlighted.

### `result.final_output`

Provides the final answer produced by the agent.

```python
print(result.final_output)
```

### `result.to_input_list()`

Provides the conversation messages in a format that can be reused as input.

This becomes particularly useful when implementing conversation history or memory manually.

---

## 7. Observability and Tracing

**Observability** is a major theme of the lesson.

It means being able to inspect what is happening inside an agent workflow rather than treating the agent as a black box.

Tracing can expose:

* User input
* System instructions
* Model calls
* Tool calls
* Tool results
* Agent execution
* Timing information
* Workflow structure

The SDK has built-in tracing capabilities.

This is especially important when debugging agent behavior.

### Key lesson

> Don't anthropomorphize the agent. Inspect the actual prompts, outputs, and tool calls.

If the agent behaves incorrectly:

1. Inspect the trace.
2. Examine the prompts.
3. Examine the tool calls.
4. Identify what went wrong.
5. Refine the instructions.
6. Test again.

This is an **iterative and empirical process**.

[OpenAI Agents SDK Tracing](https://openai.github.io/openai-agents-python/tracing/)

---

## 8. Streaming

Instead of waiting for the complete response, the SDK can stream results progressively.

The lesson introduces:

```python
Runner.run_stream(...)
```

Streaming is useful when you want users to see output as it is generated rather than waiting for the entire response.

The lesson relates this to **SSE (Server-Sent Events)**.

---

## 9. Tools Without Manual JSON

One of the major benefits demonstrated is the ability to convert an ordinary Python function into an agent tool.

Instead of manually creating a large JSON tool schema, the SDK can derive tool information from the function.

Example:

```python
@function_tool
def push_tool(message: str) -> str:
    """Send a message to the user as a push notification."""
    ...
```

The important ingredients are:

* A Python function
* Type hints
* A descriptive docstring
* The tool decorator

The SDK can then construct the tool metadata automatically.

The official SDK describes `FunctionTool` as a mechanism for wrapping Python functions as tools.

[OpenAI Agents SDK — Tools](https://openai.github.io/openai-agents-python/tools/)

---

## 10. Why Type Hints and Docstrings Matter

The lesson emphasizes that type hints and documentation are not merely stylistic.

For example:

```python
def push_tool(message: str) -> str:
    """Send a message to the user as a push notification."""
```

The framework can use this information to understand:

* The function's purpose
* The parameter names
* Parameter types
* Expected return information

This allows the SDK to generate the required tool schema automatically.

---

## 11. Adding Tools to an Agent

Tools can be supplied when creating an agent:

```python
agent = Agent(
    name="Notifier",
    instructions="Notify the user when requested.",
    tools=[push_tool],
)
```

The agent can then decide when the tool should be called.

The basic workflow becomes:

```text
User request
     ↓
Agent
     ↓
LLM decides to use tool
     ↓
Tool executes
     ↓
Tool result returned
     ↓
Agent continues
     ↓
Final response
```

The SDK's official tools documentation covers function tools and other tool categories.

---

## 12. Memory / Sessions

An important lesson is that an `Agent` does **not automatically remember previous independent `Runner.run()` calls** simply because the same agent object is reused.

For example:

```text
Run 1: "My name is Ed."
Run 2: "What's my name?"
```

The second run does not automatically receive the first conversation's history.

### Manual conversation history

One approach is to maintain the conversation yourself using a list of messages.

For example:

```python
next_input = response.to_input_list()

next_input.append({
    "role": "user",
    "content": "What's my name?"
})
```

You can then pass the complete conversation history into the next run.

This means memory can be stored wherever the application needs it:

* Files
* Databases
* S3
* MongoDB
* Application state
* Other storage systems

---

## 13. Sessions

The SDK also provides session-based conversation history management.

The lesson demonstrates a SQL-like session concept:

```python
session = SQLiteSession("conversation_id")
```

Then the session can be supplied when running the agent.

This allows the SDK to maintain conversation history between separate agent runs.

The current SDK documentation describes sessions as automatic conversation-history management across agent runs.

[OpenAI Agents SDK — Sessions / Quickstart](https://openai.github.io/openai-agents-python/quickstart/)

---

## 14. Coding Agents: Stay in Control

The lesson also provides guidance for using coding agents such as Cursor Agent or Claude Code.

### Recommended approach

* **Be precise** when writing instructions.
* Ask for current APIs and explicitly specify the relevant date/version when necessary.
* Ask for concise answers.
* Avoid unnecessary documentation and excessive code.
* Start with a **small task**.
* Work incrementally.
* Test each change.
* Validate results against clear success criteria.
* Ask the coding agent to explain or defend its implementation.
* Demand evidence and test results.
* Refine prompts when the agent gets stuck.
* Don't blindly accept generated code.

### Most important principle

> **The coding agent is your assistant; you remain responsible for the work.**

The purpose of the course is to learn how to build agents, so allowing a coding agent to build everything defeats the learning objective.

---

## 15. Start Simple

A recurring principle throughout the lesson is:

> **Start with the simplest possible architecture.**

For agent systems:

```text
Start with one agent
        ↓
Evaluate
        ↓
Identify limitations
        ↓
Add tools / memory / additional agents only when needed
```

Don't introduce multiple agents simply because the framework supports them.

---

## 16. Key Mental Model

The lesson repeatedly demystifies agent frameworks.

A useful mental model is:

```text
Agent
  ↓
LLM + instructions + tools
  ↓
Runner
  ↓
Agent loop
  ↓
Tool calls when necessary
  ↓
Final output
```

More advanced capabilities such as:

* Memory
* Sessions
* Tracing
* Handoffs
* Guardrails
* Streaming

are additional mechanisms around this basic loop.

The underlying system remains largely composed of **LLM calls, prompts, tool calls, and application logic**.

---

## 17. Most Important Takeaways

1. **OpenAI Agents SDK is intentionally lightweight.**
2. **Start with one agent before introducing multi-agent architectures.**
3. **Async Python is essential for modern agent applications.**
4. `async def` defines coroutines and `await` executes/schedules them.
5. `asyncio` is particularly valuable for I/O-bound LLM workloads.
6. `Agent` combines a model, instructions, and tools.
7. `Runner.run()` executes the agent loop.
8. **Tracing is essential for understanding and debugging agent behavior.**
9. Function decorators can turn ordinary Python functions into agent tools.
10. **Type hints and docstrings help the SDK construct tool schemas.**
11. Separate `Runner.run()` calls do not automatically share conversation history.
12. Sessions provide a convenient mechanism for maintaining conversational state.
13. Streaming allows incremental delivery of model output.
14. Agent behavior should be evaluated by inspecting actual prompts, outputs, and tool calls.
15. Coding agents should be treated as assistants—not autonomous replacements for the developer.
16. **Stay in the driver's seat and remain accountable for generated code.**

---

## References

* [OpenAI Agents SDK — Official Documentation](https://openai.github.io/openai-agents-python/)
* [OpenAI Agents SDK — GitHub Repository](https://github.com/openai/openai-agents-python)
* [OpenAI Agents SDK — Quickstart](https://openai.github.io/openai-agents-python/quickstart/)
* [OpenAI Agents SDK — Agents](https://openai.github.io/openai-agents-python/agents/)
* [OpenAI Agents SDK — Tools](https://openai.github.io/openai-agents-python/tools/)
* [OpenAI Agents SDK — Tracing](https://openai.github.io/openai-agents-python/tracing/)
* [Python `asyncio` Documentation](https://docs.python.org/3/library/asyncio.html)
* [Python Coroutines and Tasks](https://docs.python.org/3/library/asyncio-task.html)

---
---

# Week 2 Day 2 — Agent Orchestration and Automated SDR

## 1. Main Project: Automated SDR

The day's practical project is an **automated sales development representative (SDR)** system.

The system uses multiple agents to:

1. Generate different sales emails.
2. Evaluate/select the best email.
3. Send the selected email.
4. Explore different ways for agents to collaborate.

The project demonstrates **agent orchestration** using the OpenAI Agents SDK.

The important underlying idea is:

> Multi-agent systems are still fundamentally LLM calls, prompts, messages, conversation history, and tool calls.

There is no "magic" happening behind the framework.

[OpenAI Agents SDK](https://openai.github.io/openai-agents-python/)

---

# 2. Agent Orchestration

**Orchestration** means controlling:

* Which agents run
* In what order they run
* How they communicate
* How decisions are made
* Which agent is responsible for the next step

The lesson covers two major approaches:

```text
                    Agent Orchestration
                           │
             ┌─────────────┴─────────────┐
             │                           │
       Orchestrating                Orchestrating
         by Code                       by LLM
             │                           │
      Deterministic              Autonomous
      Predictable                Flexible
             │                           │
                         ┌───────────────┴──────────────┐
                         │                              │
                   Agents as Tools                 Handoffs
```

OpenAI's documentation describes the same two broad approaches: **orchestration via code** and **allowing the LLM to make orchestration decisions**.

[OpenAI Agents SDK — Agent Orchestration](https://openai.github.io/openai-agents-python/multi_agent/)

---

# 3. Orchestration by Code

This is the simplest and most predictable approach.

The application explicitly determines which agent runs next.

For example:

```text
Agent A
   ↓
Output
   ↓
Python code
   ↓
Agent B
   ↓
Output
   ↓
Agent C
```

In Python, this can simply mean making multiple `Runner.run()` calls.

### Advantages

* Predictable
* Deterministic
* Easy to understand
* Easy to debug
* Easy to log
* Easier to test
* Appropriate for business-critical workflows

### Key principle

> If you already know the workflow, there is often little reason to ask an LLM to decide the workflow.

---

# 4. Orchestration by LLM

With LLM-based orchestration, the LLM itself decides which agent/tool to use next.

Instead of writing:

```text
Run Agent A
→ Run Agent B
→ Run Agent C
```

you give an agent several capabilities and allow the model to determine what to do.

This provides:

* More autonomy
* More flexibility
* More adaptive workflows
* Greater ability to handle open-ended tasks

But it also introduces:

* Less predictability
* Less deterministic behavior
* Greater variability
* More difficulty in testing

OpenAI's documentation describes LLM orchestration as allowing an LLM to plan, reason, and decide which steps to take.

---

# 5. When to Use Each Approach

A useful decision rule from the lesson is:

| Situation                           | Preferred approach     |
| ----------------------------------- | ---------------------- |
| Fixed business workflow             | **Code orchestration** |
| Business-critical decisions         | **Code orchestration** |
| Predictability is important         | **Code orchestration** |
| Clearly defined sequence            | **Code orchestration** |
| Open-ended task                     | **LLM orchestration**  |
| High autonomy required              | **LLM orchestration**  |
| Agent needs to choose its own path  | **LLM orchestration**  |
| Coding agents / autonomous products | **LLM orchestration**  |

The lesson uses coding agents such as Claude Code and Codex as examples where LLM-driven orchestration is valuable because autonomy is central to the product.

---

# 6. Parallel Agent Execution with `asyncio.gather`

The SDR example creates three different sales agents:

1. **Professional Sales Agent**
2. **Humorous Sales Agent**
3. **Executive Sales Agent**

Each generates the same sales email in a different style.

Instead of running them sequentially, the lesson uses:

```python
await asyncio.gather(
    Runner.run(agent1, prompt),
    Runner.run(agent2, prompt),
    Runner.run(agent3, prompt),
)
```

This allows the three I/O-bound LLM requests to make progress concurrently.

### Important connection to Day 1

This builds directly on the previous lesson's `asyncio` concepts.

The framework is not magically making the agents parallel.

**Python's asynchronous execution is doing the work.**

---

# 7. The Three Sales Agents

The example uses the same underlying business context but different writing styles.

### Professional Agent

Produces an email that is:

* Professional
* Serious
* Credible
* Formal

### Humorous Agent

Produces an email that is:

* Witty
* Engaging
* Humorous

### Executive Agent

Produces an email that is:

* Concise
* Direct
* Appropriate for a busy executive

This is a useful multi-agent pattern:

> **Same task + different specialized prompts → multiple candidate outputs**

---

# 8. The Sales Picker

After generating three emails, another agent—the **Sales Picker**—evaluates them.

Its job is essentially:

```text
Three candidate emails
        ↓
Sales Picker
        ↓
Best email
```

The picker is instructed to think like a potential customer and select the email it would be most likely to respond to.

This demonstrates a common agent pattern:

> **Generate → Evaluate → Select**

---

# 9. Agent Collaboration Is Often Just Message Passing

One of the most important conceptual points is that agent collaboration can look much more sophisticated than it actually is.

Underneath:

```text
Agent A
  ↓
Output
  ↓
Prompt construction
  ↓
Agent B
  ↓
Output
```

There is no fundamentally new intelligence created merely by adding multiple agents.

The quality still depends heavily on:

* Prompts
* Context
* Model capabilities
* Tool definitions
* Evaluation
* Workflow design

---

# 10. Structured Outputs

The lesson also introduces **structured outputs** as a way of making orchestration more controlled.

Instead of asking an LLM to return free-form text, you can require a structured object such as JSON.

Conceptually:

```text
LLM
 ↓
Structured output
 ↓
Python code
 ↓
Decision
 ↓
Next agent
```

This provides more control than interpreting arbitrary natural-language output.

### Why structured outputs matter

They can make agent workflows:

* More predictable
* Easier to validate
* Easier to parse
* Easier to integrate with application logic

The lesson emphasizes that structured outputs sit somewhat between pure code orchestration and LLM-driven orchestration.

---

# 11. Production Systems Favor Predictability

A major takeaway is that production agentic systems often favor **code-driven orchestration**.

Why?

Because business systems frequently need:

* Predictable behavior
* Deterministic workflows
* Clear failure modes
* Auditable execution
* Easier testing
* Controlled business logic

Giving an LLM complete control over the workflow can make behavior harder to predict.

Therefore:

> **Use LLM autonomy when autonomy provides real value—not simply because the framework makes it possible.**

---

# 12. Function Tools

The project also demonstrates turning a normal Python function into an agent tool.

For example:

```python
@function_tool
def send_email_tool(
    subject: str,
    text_body: str,
    html_body: str
):
    ...
```

The SDK can derive the tool schema from:

* Function name
* Type hints
* Docstring
* Parameter descriptions

This avoids manually writing the complete JSON schema.

[OpenAI Agents SDK — Tools](https://openai.github.io/openai-agents-python/tools/)

---

# 13. Good Tool Documentation Matters

The lesson demonstrates that a tool's docstring is important because the description is exposed to the LLM.

For example:

```python
def send_email_tool(
    subject: str,
    text_body: str,
    html_body: str
):
    """
    Send an email.

    subject: Subject of the email.
    text_body: Plain-text body.
    html_body: HTML body.
    """
```

The resulting tool schema can communicate:

* What the tool does
* What parameters it accepts
* What each parameter means
* What data types are expected

### Key lesson

> **Tool descriptions are part of the prompt/context provided to the model.**

Poor descriptions can therefore produce poor tool usage.

---

# 14. Forcing Tool Usage

The lesson encounters a practical problem: the model sometimes doesn't call the email tool even when instructed to do so.

A framework setting can require tool usage.

This illustrates a broader lesson:

> Prompting alone is not always sufficient when a particular behavior is mandatory.

When a workflow has a hard requirement, application-level controls can be preferable to hoping the LLM follows instructions every time.

---

# 15. Agents as Tools

The first LLM-based orchestration technique is **agents as tools**.

An existing agent can be converted into a tool using:

```python
sales_agent.as_tool(...)
```

Conceptually:

```text
Manager Agent
     │
     ├── Sales Writer 1
     ├── Sales Writer 2
     ├── Sales Writer 3
     └── Send Email
```

The manager remains in control.

It decides:

* Which specialist to call
* What to ask it
* When to call it
* What to do with its output

OpenAI's current documentation describes this as the **manager pattern**: the central agent retains control and invokes specialist agents as tools.

[OpenAI Agents SDK — Agents as Tools](https://openai.github.io/openai-agents-python/tools/)

---

# 16. Why Agents-as-Tools Can Be Useful

This pattern works particularly well when:

* One agent should remain responsible for the final result.
* Specialist agents perform bounded subtasks.
* Their results need to be combined.
* A manager needs to evaluate several specialists.
* Shared controls/guardrails should remain with the manager.

Example:

```text
                Manager
                   │
       ┌───────────┼───────────┐
       ↓           ↓           ↓
   Researcher   Writer      Analyst
       │           │           │
       └───────────┼───────────┘
                   ↓
             Final Answer
```

---

# 17. Handoffs

The second LLM-driven orchestration mechanism is **handoffs**.

A handoff transfers control from one agent to another.

Conceptually:

```text
Agent A
   │
   │ handoff
   ↓
Agent B
   │
   ↓
Agent C
```

The key difference is:

### Agents as tools

```text
Manager → Specialist → Manager
```

The manager retains control.

### Handoff

```text
Manager → Specialist
```

The specialist takes over.

OpenAI's documentation explicitly distinguishes these patterns: with agents-as-tools, the manager remains responsible for the conversation; with handoffs, the receiving specialist becomes the active agent.

[OpenAI Agents SDK — Handoffs](https://openai.github.io/openai-agents-python/handoffs/)

---

# 18. Agents-as-Tools vs Handoffs

| Feature                   | Agents as Tools | Handoffs        |
| ------------------------- | --------------- | --------------- |
| Manager retains control   | Yes             | No              |
| Specialist takes over     | No              | Yes             |
| Specialist returns result | Yes             | Not necessarily |
| Good for bounded subtasks | **Yes**         | Sometimes       |
| Good for routing          | Sometimes       | **Yes**         |
| Final conversation owner  | Manager         | Specialist      |
| Workflow style            | Centralized     | Delegated       |

A useful mental model:

> **Tools = "Help me do this."**

> **Handoff = "You take over from here."**

---

# 19. Handoffs Are Still Tools Under the Hood

An important technical insight from the lesson is that handoffs are exposed to the LLM through tool-like mechanisms.

The current OpenAI documentation confirms that handoffs are represented to the LLM as tools such as `transfer_to_<agent>`.

This reinforces the overall theme:

> Framework abstractions often reduce to familiar primitives such as prompts, tool calls, and messages.

---

# 20. Observability Is Essential

The lesson repeatedly returns to **tracing**.

For a multi-agent system, tracing allows you to see:

```text
Sales Manager
    │
    ├── Professional Agent
    ├── Humorous Agent
    ├── Executive Agent
    │
    └── Sales Picker
           │
           ↓
       Send Email
```

You can inspect:

* Which agents ran
* The order of execution
* Parallel execution
* Prompts
* Outputs
* Tool calls
* Tool parameters
* Final results

This makes it possible to determine whether the system actually behaved as intended.

[OpenAI Agents SDK — Tracing](https://openai.github.io/openai-agents-python/tracing/)

---

# 21. Don't Anthropomorphize Agents

A particularly useful engineering principle is:

> **Don't assume the agent understands what you intended. Inspect what actually happened.**

Instead of thinking:

> "The sales manager decided that the humorous email was best."

Look at:

* The actual system prompt
* The user prompt
* The candidate emails
* The tool calls
* The model output
* The trace

This turns debugging into an empirical process rather than guesswork.

---

# 22. Email Infrastructure

The lesson deliberately keeps email infrastructure lightweight because the primary objective is **agent orchestration**, not email delivery.

Three approaches are discussed:

### SMTP

Use an existing email provider's SMTP server.

Examples mentioned include:

```text
Gmail       → smtp.gmail.com
Outlook     → smtp-mail.outlook.com
Microsoft 365 → smtp.office365.com
iCloud      → smtp.mail.me.com
```

### Pushover

Instead of sending an email, the system can send a push notification.

This is useful for development/testing.

### Professional email APIs

For a real production system, services such as **SendGrid** or **Resend** can be considered.

[SendGrid API Documentation](https://www.twilio.com/docs/sendgrid/api-reference)

---

# 23. Why the Course Avoids Bulk Email Setup

Services such as SendGrid are designed for email at scale and therefore require more configuration around:

* Domain ownership
* DNS
* Authentication
* Email reputation
* Sending infrastructure

The course intentionally avoids spending most of the lab on those details.

The point is:

> **Demonstrate agent collaboration rather than build a production email-delivery platform.**

---

# 24. The Biggest Mistake: No Quantitative Evaluation

The instructor explicitly identifies a weakness in the lab.

The system generates and selects emails, but it does **not** quantitatively prove that the selected emails are effective.

This is extremely important.

An LLM is very good at generating **plausible content**.

That does not mean the content is commercially effective.

For sales automation, useful metrics might include:

* Open rate
* Reply rate
* Positive reply rate
* Meeting-booking rate
* Conversion rate
* Revenue generated
* Deals won

### Critical principle

```text
Generate content
       ↓
Measure real-world outcome
       ↓
Evaluate
       ↓
Improve prompts/workflow
       ↓
Measure again
```

---

# 25. The Difference Between "Looks Good" and "Works"

An email may sound excellent while producing zero sales.

Therefore:

> **Human/LLM judgment of content quality is not the same as business performance.**

For production systems, evaluation should ultimately connect agent behavior to actual business KPIs.

This is one of the most important lessons of the entire session.

---

# 26. Iterative Prompt Engineering

When the agent behaves unpredictably:

1. Inspect the trace.
2. Identify the undesirable behavior.
3. Modify the prompt.
4. Test again.
5. Compare results.
6. Repeat.

The instructor describes this as a highly:

* Empirical
* Experimental
* Iterative

process.

Agent engineering is therefore not simply:

```text
Write prompt → Done
```

It is closer to:

```text
Prompt
 ↓
Run
 ↓
Observe
 ↓
Evaluate
 ↓
Modify
 ↓
Run again
```

---

# 27. Harder Extension: Full Automated Sales Agent

The optional challenge is to move beyond an automated SDR.

Instead of:

```text
Generate email
      ↓
Send email
```

build:

```text
Generate email
      ↓
Send email
      ↓
Receive reply
      ↓
Continue conversation
      ↓
Handle objections
      ↓
Qualify prospect
      ↓
Move toward closing
```

This would create a much more complete **automated sales agent**.

---

# 28. Other Possible Extensions

The lesson suggests several extensions:

* Add an agent that **refines/improves generated emails**.
* Build the extension using **all three orchestration patterns**.
* Replace basic SMTP with a professional email provider.
* Allow the agent to continue conversations after replies.
* Integrate with **Telegram**.
* Extend the system toward automated deal closing.

[Telegram Bot API](https://core.telegram.org/bots/api)

---

# 29. Generalization Beyond Sales

The same architecture can apply to many business processes involving:

* Conversations
* User interactions
* Decisions
* Content generation
* External tools
* Multiple specialized agents

Examples could include:

```text
Customer support
Research
Recruiting
Marketing
Operations
Lead qualification
Content workflows
Internal knowledge systems
```

The important question is not:

> "Can I use multiple agents?"

It is:

> **"Does multiple-agent orchestration produce a measurable improvement in the business process?"**

---

# 30. Most Important Takeaways

1. **Agent orchestration controls how multiple agents collaborate.**
2. There are two broad approaches: **orchestration by code** and **orchestration by LLM**.
3. **Code orchestration is more predictable and deterministic.**
4. **LLM orchestration provides greater autonomy but less predictability.**
5. `asyncio.gather()` can run multiple independent LLM calls concurrently.
6. A common pattern is **generate multiple candidates → evaluate/select one**.
7. `Agent.as_tool()` allows one agent to use another agent as a tool.
8. **Agents-as-tools keep the manager in control.**
9. **Handoffs transfer control to another agent.**
10. Tools and handoffs are both mechanisms for agent collaboration.
11. Tool descriptions, type hints, and docstrings are important because they become part of the model's context.
12. **Tracing is essential for debugging multi-agent workflows.**
13. Don't anthropomorphize agents—inspect their actual prompts, tool calls, and outputs.
14. Start with the simplest architecture that solves the problem.
15. Don't introduce multi-agent complexity merely because the framework supports it.
16. **LLM-generated content being plausible does not mean it is effective.**
17. Production systems should be evaluated against **real business KPIs**.
18. Prompt and workflow development is an **iterative empirical process**.
19. Use LLM orchestration when autonomy is genuinely valuable.
20. Use code orchestration when predictability and control matter more.

---

# Quick Revision Cheat Sheet

| Concept                      | Remember                                        |
| ---------------------------- | ----------------------------------------------- |
| **Orchestration**            | Managing how agents collaborate                 |
| **Code orchestration**       | Python controls the workflow                    |
| **LLM orchestration**        | LLM controls the workflow                       |
| **`asyncio.gather()`**       | Run independent async tasks concurrently        |
| **Agents as tools**          | Manager calls specialists but retains control   |
| **Handoff**                  | Specialist takes over                           |
| **Function tool**            | Python function exposed to an agent             |
| **Tracing**                  | Inspect what actually happened                  |
| **Structured output**        | Predictable machine-readable model output       |
| **SDR example**              | Generate → evaluate → select → send             |
| **Production priority**      | Predictability + measurable outcomes            |
| **Key evaluation principle** | Measure business results, not just text quality |

---

# References

* [OpenAI Agents SDK — Python](https://openai.github.io/openai-agents-python/)
* [OpenAI Agents SDK — Agent Orchestration](https://openai.github.io/openai-agents-python/multi_agent/)
* [OpenAI Agents SDK — Agents](https://openai.github.io/openai-agents-python/agents/)
* [OpenAI Agents SDK — Tools](https://openai.github.io/openai-agents-python/tools/)
* [OpenAI Agents SDK — Handoffs](https://openai.github.io/openai-agents-python/handoffs/)
* [OpenAI Agents SDK — Tracing](https://openai.github.io/openai-agents-python/tracing/)
* [SendGrid API Documentation](https://www.twilio.com/docs/sendgrid/api-reference)
* [Telegram Bot API](https://core.telegram.org/bots/api)
