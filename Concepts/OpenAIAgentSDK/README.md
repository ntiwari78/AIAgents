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

---
---

# OpenAI Agents SDK — Key Points & Deep Dive

## 1. Big Picture

The central theme is moving from simple LLM calls toward **reliable agentic workflows**.

The transcript emphasizes two broad ways to orchestrate agents:

1. **Orchestration with code**

   * Explicitly control the sequence of `runner.run()` calls.
   * More predictable, deterministic, understandable, and resilient.
   * Particularly suitable when the workflow is already known.

2. **Orchestration with LLMs**

   * Give an agent tools representing other agents.
   * Let the LLM decide which agents/tools to invoke and in what order.
   * More autonomous and flexible, but less predictable.

The OpenAI Agents SDK officially supports agents, tools, handoffs, guardrails, structured outputs, MCP integration, tracing, and sandbox agents.

### Practical rule

> **If you know the workflow, orchestrate with code. If you need the system to decide the workflow dynamically, consider LLM-based orchestration.**

---

# 2. Using Models Other Than OpenAI

One important lesson is that an agent framework does not necessarily have to be tied to a single model provider.

The transcript demonstrates using:

* Google Gemini
* OpenRouter models
* Groq-hosted models
* OpenAI models

The basic pattern is:

```text
Provider's OpenAI-compatible endpoint
        ↓
Python client
        ↓
Model object
        ↓
Agent
```

Instead of passing a model name as a simple string, you can construct a model object using the appropriate client and provider endpoint.

The transcript demonstrates this by creating multiple sales agents with **identical instructions but different underlying models**.

### Why this matters

It allows you to experiment with:

* Cost
* Latency
* Reasoning capability
* Model quality
* Provider availability
* Open-source models

without completely redesigning your agent architecture.

---

# 3. Agents as Tools

A particularly useful pattern is:

```text
Manager Agent
     |
     +---- Sales Agent A
     |
     +---- Sales Agent B
     |
     +---- Sales Agent C
     |
     +---- Email Tool
```

The specialist agents can be converted into tools.

The manager can then ask several specialists to perform subtasks and use their results.

The transcript demonstrates this pattern with three different models generating sales emails and a manager selecting the best result.

The official SDK documentation describes `Agent.as_tool()` as a way to expose one agent as a callable tool while keeping the manager in control of the workflow.

### Mental model

**Agents-as-tools:**

```text
Manager
   ↓
Specialist
   ↓
Result
   ↓
Manager continues
```

This is useful when the manager needs the specialist's result before deciding what to do next.

---

# 4. Handoffs

A handoff is different.

With a handoff:

```text
Agent A
   ↓
Agent B takes control
```

The original agent does not simply receive a result from the specialist.

The specialist becomes the active agent.

The transcript compares:

| Pattern         | Control                 |
| --------------- | ----------------------- |
| Agents as tools | Manager retains control |
| Handoff         | Control is transferred  |

The transcript is skeptical about handoffs because of perceived framework coupling and reliability issues. That is the instructor's opinion; the current OpenAI SDK documentation treats handoffs as a supported orchestration primitive.

### Key distinction

```text
Agents as tools:

A → B → A


Handoff:

A → B
```

---

# 5. Structured Outputs

This is one of the most important concepts in the transcript.

Normally an LLM produces:

```text
plain natural language
```

With structured output, you can ask it to produce data conforming to a defined schema.

For example:

```python
class EmailReview(BaseModel):
    is_professional: bool
    number_of_sentences: int
    contains_placeholders: bool
```

Instead of receiving:

```text
"This email is not professional..."
```

your application receives something conceptually like:

```python
EmailReview(
    is_professional=False,
    number_of_sentences=3,
    contains_placeholders=True
)
```

The transcript explains that the framework uses JSON/schema representation and converts the resulting structured data into a Python object such as a Pydantic model.

The current Agents SDK supports `output_type` for structured output and can use Python types such as Pydantic models.

---

# 6. Why Structured Outputs Are Powerful

Structured output creates a bridge between:

```text
LLM reasoning
        ↓
structured data
        ↓
normal Python code
        ↓
business logic
```

For example:

```python
if not review.is_professional:
    reject_email()
```

This is much more useful than asking the LLM to return a sentence such as:

> "I don't think this email is appropriate."

The application can directly make a deterministic decision.

### Major benefit

**LLMs generate judgments; code executes consequences.**

That separation is extremely important for reliable agentic systems.

---

# 7. Pydantic

The transcript uses **Pydantic** to define the structured objects.

Pydantic provides a convenient way to define Python data models and generate/validate JSON-compatible schemas.

For example:

```python
class WebSearchItem(BaseModel):
    reason: str
    query: str
```

Then:

```python
class WebSearchPlan(BaseModel):
    searches: list[WebSearchItem]
```

This creates a hierarchy such as:

```text
WebSearchPlan
 ├── WebSearchItem
 │    ├── reason
 │    └── query
 ├── WebSearchItem
 │    ├── reason
 │    └── query
 └── ...
```

The transcript specifically recommends putting the **reason before the query**, because the generated fields appear in that order and the rationale can influence the quality of the subsequent query.

---

# 8. Structured Outputs + Constrained Decoding

The transcript goes one level deeper into why structured output can be reliable.

The simplified idea is:

```text
LLM generates probability distribution
              ↓
Invalid tokens are constrained
              ↓
Only schema-compatible output is allowed
              ↓
Valid structured data
```

This is related to **constrained decoding**.

The important conceptual takeaway is:

> Structured output is not simply "please output valid JSON."

The system can impose a formal output structure so that the generated result conforms to the required schema.

---

# 9. Guardrails

Guardrails are controls that prevent an agent from producing or accepting undesirable results.

Typical examples:

* Reject inappropriate user input.
* Prevent unsafe tool calls.
* Validate generated content.
* Stop an email from being sent if it fails quality checks.
* Prevent invalid data from entering a workflow.

The transcript demonstrates an email checker that verifies:

```text
is_professional
contains_placeholders
```

If the email fails the checks, execution is stopped.

The current SDK documents three main guardrail categories:

1. **Input guardrails**
2. **Output guardrails**
3. **Tool guardrails**

---

# 10. Guardrail Tripwires

A guardrail can trigger a **tripwire**.

Conceptually:

```text
Agent generates output
        ↓
Guardrail checks output
        ↓
Problem?
   ↙       ↘
 Yes        No
 ↓           ↓
Tripwire    Continue
 ↓
Stop
```

The transcript demonstrates an intentionally bad "cowboy" sales email being rejected by the guardrail.

The SDK raises an exception when the tripwire is triggered.

---

# 11. Input vs Output vs Tool Guardrails

| Guardrail | Purpose                              |
| --------- | ------------------------------------ |
| Input     | Validate incoming user input         |
| Output    | Validate final agent output          |
| Tool      | Validate function-tool calls/results |

An important implementation detail is that agent-level input/output guardrails have workflow boundaries.

The current SDK documentation confirms:

* Input guardrails run on the first agent.
* Output guardrails run on the final agent.
* Tool guardrails can run around individual function-tool calls.

---

# 12. Simple Code-Based Guardrails

The instructor's preferred approach is worth understanding separately from the SDK feature itself.

Instead of relying entirely on framework-specific guardrail abstractions:

```text
Agent
 ↓
Guardrail
 ↓
Another framework mechanism
```

you can explicitly write:

```text
Agent
 ↓
checker agent
 ↓
Python validation
 ↓
if valid:
    continue
else:
    stop
```

The transcript argues that this approach can be:

* Easier to understand
* Easier to debug
* More portable
* Less coupled to one framework

This is an architectural preference expressed by the instructor, rather than a statement that the SDK guardrail system is incorrect.

---

# 13. Tracing and Observability

**Tracing is essential when building agent systems.**

Instead of guessing what an agent did, inspect the actual execution trace.

You can see:

```text
Agent
 ├── LLM call
 ├── Tool call
 ├── Specialist agent
 ├── Handoff
 ├── Guardrail
 └── Final output
```

The transcript repeatedly uses traces to determine:

* Which model ran
* Which agents ran
* Whether agents ran in parallel
* Which tool was selected
* Which email was generated
* Why a guardrail triggered
* Which model produced the winning output

The Agents SDK provides built-in tracing for LLM generations, tools, handoffs, guardrails, and other workflow events.

### Important mindset

Don't ask:

> "Why did the agent decide that?"

Instead inspect:

> **What prompt did it receive? What tool did it call? What output did it generate? What happened next?**

That is a much better debugging methodology.

---

# 14. Sandbox Agents

The transcript introduces **SandboxAgent** as an optional newer capability.

The idea is to give an agent an isolated execution environment.

Instead of allowing an agent to freely modify your real filesystem:

```text
Your computer
     ↓
Sandbox
     ↓
Agent works here
```

The sandbox can contain:

* Files
* Directories
* Shell access
* Code
* Other workspace resources

The transcript demonstrates an agent reviewing a Python file, discovering a bug, and writing the corrected version into an output directory.

The current SDK documentation describes a sandbox as an isolated workspace with concepts such as:

* `Manifest`
* Capabilities
* Sandbox session
* `SandboxRunConfig`

---

# 15. Why Sandboxing Matters

Agents increasingly need to:

* Read files
* Edit files
* Run programs
* Execute shell commands
* Analyze repositories
* Modify code
* Test their changes

Giving an autonomous agent direct access to your real machine is risky.

A sandbox provides an execution boundary.

### Mental model

```text
Agent
  ↓
Sandbox
  ├── Read files
  ├── Edit files
  ├── Run commands
  └── Test code
```

The transcript's example illustrates the value by allowing the agent to repair a bug without modifying the original source directly.

---

# 16. The Sandbox Bug Example

The example contains a classic Python mutable-object bug.

An empty list was reused for multiple dictionary keys, meaning several customer IDs effectively referenced the same list.

The result was that orders for different customers became mixed together.

The agent identified the problem and produced a corrected implementation using `setdefault`.

### Lesson

Sandbox agents are not merely chatbots.

They can potentially operate as:

```text
Inspect
 → Diagnose
 → Modify
 → Test
 → Produce artifact
```

This is an important step toward autonomous coding agents.

---

# 17. Model Context Protocol — MCP

The transcript gives a preview of **Model Context Protocol (MCP)**.

The core idea:

> MCP provides a standardized way for AI applications to connect to external tools and data sources.

Instead of manually implementing every integration:

```text
Agent
 ↓
Custom Python wrapper
 ↓
API
 ↓
External system
```

you can connect an MCP server:

```text
Agent
 ↓
MCP
 ↓
External tool/service
```

MCP defines standardized interactions between AI applications and servers that expose capabilities such as tools, resources, and prompts.

---

# 18. Why MCP Is Important

The transcript gives a particularly good use case:

### Problem

An older model doesn't know about a newly released API or SDK feature.

### Solution

Connect the agent to an MCP server that can retrieve current documentation.

```text
User question
      ↓
Agent
      ↓
MCP server
      ↓
Current documentation
      ↓
Agent
      ↓
Accurate answer
```

The transcript demonstrates this using a documentation-oriented MCP service.

### Key insight

MCP allows capabilities written by someone else to become available to your agent without you having to implement every integration yourself.

---

# 19. MCP vs Function Tools

A useful distinction:

### Function tool

You define the function yourself:

```python
@function_tool
def get_customer():
    ...
```

### MCP

Someone else can expose the functionality through an MCP server.

```text
Agent
 ↓
MCP client
 ↓
MCP server
 ↓
External capability
```

This makes MCP especially interesting for **interoperability and reusable integrations**.

The official Agents SDK supports MCP-backed tools alongside normal function tools.

---

# 20. Hosted Tools

The transcript also discusses hosted tools such as:

* Web search
* File search
* Code interpreter
* Hosted MCP

The advantage is speed of development.

You can get a working prototype without building every infrastructure component yourself.

The trade-offs discussed include:

* Cost
* Provider dependence
* Reduced infrastructure control
* Potential ecosystem lock-in

The transcript specifically notes that OpenAI-hosted functionality is more tightly connected to OpenAI infrastructure than the model-agnostic parts of the Agents SDK.

---

# 21. Deep Research Agent

The major project introduced in the transcript is a **Deep Research Agent**.

The important insight is that a deep-research system does not need to be magical.

It can be decomposed into a small number of understandable components.

The proposed architecture contains four agents:

```text
                User Question
                      ↓
                 Planner Agent
                      ↓
             Search Plan / Queries
                      ↓
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
    Search Agent  Search Agent  Search Agent
        └─────────────┼─────────────┘
                      ↓
                 Writer Agent
                      ↓
                 Research Report
                      ↓
                 Email Agent
                      ↓
                    Email
```

The transcript explicitly describes four agents:

1. **Search Agent**
2. **Planner Agent**
3. **Writer Agent**
4. **Email Agent**

---

# 22. Search Agent

The Search Agent:

* Receives a search query.
* Uses web search.
* Produces a concise summary of results.

The transcript demonstrates this with a hosted web-search tool.

The important point is that the search agent is intentionally simple.

```text
Search term
    ↓
Web search
    ↓
Summary
```

---

# 23. Planner Agent

The Planner Agent receives the user's original question and generates multiple searches.

For example:

```text
User question
      ↓
Planner
      ↓
1. Query A
2. Query B
3. Query C
4. Query D
5. Query E
```

The transcript uses structured output for this.

Conceptually:

```python
class WebSearchItem(BaseModel):
    reason: str
    query: str


class WebSearchPlan(BaseModel):
    searches: list[WebSearchItem]
```

This is a strong example of structured output being used to control an agentic workflow.

---

# 24. Why Multiple Searches?

One search is unlikely to provide sufficient coverage.

Multiple searches allow the research system to investigate different dimensions of the question.

For example:

```text
Question:
"What are the leading AI agent frameworks?"

Search 1 → popularity
Search 2 → GitHub activity
Search 3 → enterprise adoption
Search 4 → technical capabilities
Search 5 → community activity
```

The planner therefore converts:

```text
one ambiguous question
```

into:

```text
multiple concrete research tasks
```

---

# 25. Parallel Search Execution

The transcript then uses Python orchestration to run the searches in parallel.

Conceptually:

```python
results = await asyncio.gather(
    search(query_1),
    search(query_2),
    search(query_3),
    search(query_4),
    search(query_5),
)
```

Instead of:

```text
Search 1
 ↓
Search 2
 ↓
Search 3
 ↓
Search 4
 ↓
Search 5
```

you get:

```text
Search 1 ─┐
Search 2 ─┤
Search 3 ─┼──→ Writer
Search 4 ─┤
Search 5 ─┘
```

This is one of the biggest practical benefits of code-based orchestration.

---

# 26. Writer Agent

The Writer Agent receives:

* Original research question
* Search results

and produces a structured research report.

The transcript defines a report object containing:

* Short summary
* Markdown report
* Follow-up research questions

Conceptually:

```text
Search results
      ↓
Writer Agent
      ↓
ReportData
 ├── summary
 ├── report
 └── follow-up questions
```

---

# 27. Email Agent

The Email Agent takes the completed research report and turns it into an email.

It uses a normal function tool such as:

```text
send_email(subject, body)
```

The transcript uses the same email infrastructure introduced in the earlier sales-agent project.

This demonstrates an important pattern:

> **LLMs decide what content should be produced; ordinary code handles real-world side effects.**

---

# 28. Complete Deep Research Workflow

The entire system becomes:

```text
                 User Question
                       │
                       ▼
                ┌─────────────┐
                │   Planner   │
                └──────┬──────┘
                       │
                Search Plan
                       │
         ┌─────────────┼─────────────┐
         ▼             ▼             ▼
      Search         Search        Search
      Agent          Agent         Agent
         │             │             │
         └─────────────┼─────────────┘
                       ▼
                 Search Results
                       │
                       ▼
                ┌─────────────┐
                │    Writer   │
                └──────┬──────┘
                       │
                       ▼
                 Research Report
                       │
                       ▼
                ┌─────────────┐
                │    Email    │
                └──────┬──────┘
                       │
                       ▼
                     User
```

The transcript emphasizes that the apparent complexity of "deep research" can be reduced to these relatively simple components.

---

# 29. Why Code Orchestration Works Well Here

The workflow is already known:

```text
Plan
→ Search
→ Aggregate
→ Write
→ Send
```

Therefore there is little reason to make an LLM decide the overall workflow.

Code can guarantee:

```text
Step 1 happens
↓
Step 2 happens
↓
Step 3 happens
↓
Step 4 happens
```

This makes the system easier to reason about and debug.

---

# 30. Where LLM Orchestration Would Help

The transcript contrasts this with a more autonomous architecture.

An LLM could decide:

```text
Question
 ↓
What should I research?
 ↓
Search
 ↓
What should I investigate next?
 ↓
Search again
 ↓
Do I need another source?
 ↓
Write report
```

This gives more flexibility but introduces more uncertainty.

### Trade-off

| Code orchestration | LLM orchestration         |
| ------------------ | ------------------------- |
| Predictable        | Autonomous                |
| Deterministic      | Flexible                  |
| Easier to debug    | More difficult to debug   |
| Known workflow     | Dynamic workflow          |
| More reliable      | Potentially more creative |
| Less autonomous    | More autonomous           |

---

# 31. The Most Important Architectural Principle

A powerful way to think about agentic systems is:

```text
LLM = judgment / generation
Code = control / guarantees
Tools = capabilities
Structured outputs = interface
Guardrails = constraints
Tracing = observability
Sandbox = execution boundary
MCP = interoperability
```

This is probably the most useful mental model to retain from the entire lesson.

---

# 32. Production Mindset

The transcript repeatedly emphasizes that **a demo working is not the same as a production system working**.

You should evaluate:

* Accuracy
* Latency
* Cost
* Reliability
* Tool failures
* Safety
* User satisfaction
* Business outcomes

For a research agent, for example:

```text
Did it produce a report?
```

is not enough.

You should ask:

```text
Was the report factually accurate?
Were sources relevant?
Was important information missed?
How much did it cost?
How long did it take?
Would users trust it?
```

---

# 33. Key Lessons to Remember

## The 10 most important takeaways

1. **Prefer code orchestration when the workflow is known.**
2. **Use LLM orchestration when you genuinely need dynamic decision-making.**
3. **Agents can be exposed as tools to other agents.**
4. **Handoffs transfer control rather than simply returning a result.**
5. **Structured outputs turn LLM responses into usable application data.**
6. **Pydantic is an excellent way to define structured agent outputs in Python.**
7. **Guardrails are essential for controlling unreliable agent behavior.**
8. **Tracing is critical for debugging agentic workflows.**
9. **Sandbox agents provide an isolated workspace for code/file operations.**
10. **MCP provides a standardized way to connect agents with external tools and data.**

---

# 34. Quick Revision Cheat Sheet

| Concept            | Remember                               |
| ------------------ | -------------------------------------- |
| Agent              | Model + instructions + tools           |
| Runner             | Executes agent workflows               |
| Tool               | Capability an agent can invoke         |
| Agents-as-tools    | Specialist returns result to manager   |
| Handoff            | Specialist takes control               |
| Structured output  | Typed/schema-constrained result        |
| Pydantic           | Python data/schema model               |
| Guardrail          | Validation/control mechanism           |
| Tripwire           | Stops execution when a guardrail fails |
| Tracing            | Observe/debug agent execution          |
| Sandbox            | Isolated execution workspace           |
| MCP                | Standardized tool/context integration  |
| Planner            | Converts question → searches           |
| Search agent       | Performs research                      |
| Writer             | Converts research → report             |
| Email agent        | Performs delivery                      |
| `asyncio.gather`   | Run independent tasks concurrently     |
| Hosted tool        | Provider-managed capability            |
| Code orchestration | Predictable workflow                   |
| LLM orchestration  | Autonomous workflow                    |

---

# 35. Recommended Learning Path

If you want to go deeper, study these in roughly this order:

### Step 1 — Agents SDK fundamentals

[OpenAI Agents SDK documentation](https://openai.github.io/openai-agents-python/?utm_source=chatgpt.com)

Learn:

* Agents
* Runner
* Tools
* Models
* Sessions

### Step 2 — Multi-agent orchestration

[Agents SDK — Agent orchestration](https://openai.github.io/openai-agents-python/multi_agent/?utm_source=chatgpt.com)

Focus on:

* Agents as tools
* Handoffs
* Manager/specialist patterns

### Step 3 — Structured outputs

[Agents SDK — Agents and structured output types](https://openai.github.io/openai-agents-python/agents/?utm_source=chatgpt.com)

Then learn:

* JSON Schema
* Pydantic
* Schema validation
* Typed application logic

### Step 4 — Pydantic

[Pydantic documentation](https://docs.pydantic.dev/?utm_source=chatgpt.com)

Focus on:

* `BaseModel`
* Validation
* JSON Schema
* Nested models

### Step 5 — Guardrails

[OpenAI Agents SDK — Guardrails](https://openai.github.io/openai-agents-python/guardrails/?utm_source=chatgpt.com)

Study:

* Input guardrails
* Output guardrails
* Tool guardrails
* Tripwires
* Parallel vs blocking execution

### Step 6 — Tracing

[OpenAI Agents SDK — Tracing](https://openai.github.io/openai-agents-python/tracing/?utm_source=chatgpt.com)

Learn to inspect:

```text
LLM calls
Tools
Handoffs
Guardrails
Token usage
Latency
```

### Step 7 — Sandbox agents

[OpenAI Agents SDK — Sandbox concepts](https://openai.github.io/openai-agents-python/sandbox/guide/?utm_source=chatgpt.com)

Focus on:

* `SandboxAgent`
* Manifest
* Capabilities
* Sandbox sessions
* `SandboxRunConfig`

### Step 8 — MCP

[OpenAI Agents SDK — MCP integration](https://openai.github.io/openai-agents-python/mcp/?utm_source=chatgpt.com)

And study the protocol itself:

[Model Context Protocol specification](https://modelcontextprotocol.io/specification/2025-03-26/basic?utm_source=chatgpt.com)

MCP is especially worth studying deeply because its specification continues to evolve; the July 2026 release introduced significant protocol changes, including a stateless protocol core.

### Step 9 — Web search

[OpenAI API developer quickstart — tools and web search](https://platform.openai.com/docs/quickstart/make-your-first-api-request?utm_source=chatgpt.com)

Understand how web search can be attached to model calls and used as part of research workflows.

---

# 36. Final Mental Model

The entire lesson can be compressed into this:

```text
                    AGENTIC APPLICATION
                           │
          ┌────────────────┼────────────────┐
          │                │                │
       Agents            Tools           MCP
          │                │                │
          └──────────┬─────┴────────────────┘
                     │
               Orchestration
                /          \
             Code           LLM
              │              │
        Predictable       Autonomous
              │              │
              └──────┬───────┘
                     │
             Structured Output
                     │
                Guardrails
                     │
                 Tracing
                     │
                  Sandbox
                     │
               Production App
```

**The deepest lesson:** don't think of an agent as a mysterious autonomous entity. Think of it as an LLM embedded inside a software system. **Code controls the workflow, tools provide capabilities, structured outputs create reliable interfaces, guardrails enforce constraints, tracing provides visibility, sandboxes provide safe execution, and MCP connects the system to external capabilities.**

The uploaded transcript itself demonstrates this progression from simple multi-agent orchestration to a complete deep-research workflow.


---
---

# Week 2 Day 5 — Deep Research Agent: Python Modules, Gradio UI & Deployment

## 1. Main Goal

The goal of Day 5 is to take the Deep Research Agent built in Jupyter notebooks and turn it into a **deployable AI application**.

The progression is:

```text
Jupyter Notebook
      ↓
Python Modules
      ↓
Research Manager
      ↓
Gradio UI
      ↓
Hugging Face Spaces
      ↓
Live Deep Research Application
```

The project is the classic **agentic AI deep-research use case**.

---

# 2. The Four-Agent Architecture

The previous day's research system consisted of four agents:

1. **Search Agent**
2. **Planner Agent**
3. **Writer Agent**
4. **Email Agent**

The Day 5 task is to move each of these from notebook cells into reusable Python modules.

```text
                    User Question
                         │
                         ▼
                  Planner Agent
                         │
                  Search Plan
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       Search          Search         Search
       Agent           Agent          Agent
          └──────────────┼──────────────┘
                         ▼
                   Writer Agent
                         │
                         ▼
                   Research Report
                         │
                         ▼
                   Email Agent
```

This modular structure makes the application easier to maintain, test, reuse, and deploy.

The OpenAI Agents SDK similarly treats agents as reusable components defined with instructions, tools, models, and optional structured outputs. ([OpenAI Agents SDK](https://openai.github.io/openai-agents-python/))

---

# 3. Search Agent Module

The Search Agent is responsible for searching the web.

Its configuration includes:

* Instructions
* Web search tool
* Model
* Model settings

The transcript also introduces a useful configuration pattern:

```text
.env
  ↓
DEFAULT_MODEL_NAME
  ↓
Search Agent
```

If a model name is provided through `.env`, the application uses it.

Otherwise, it falls back to a default model.

### Why this is useful

You don't have to modify Python source code every time you want to change the model.

For example:

```text
Development
→ cheaper/faster model

Production
→ more capable model
```

This is a good example of separating **configuration from application code**.

---

# 4. Planner Agent

The Planner Agent converts a user question into multiple searches.

The transcript continues using structured output with Pydantic models.

Conceptually:

```python
class WebSearchItem(BaseModel):
    reason: str
    query: str


class WebSearchPlan(BaseModel):
    searches: list[WebSearchItem]
```

The planner then uses:

```text
output_type = WebSearchPlan
```

so that the result can be handled programmatically.

The current Agents SDK supports Pydantic models and other Pydantic-compatible types as structured output types.

### Key idea

Instead of asking:

> "Give me some searches."

the application receives something structurally like:

```text
WebSearchPlan
 ├── Search 1
 ├── Search 2
 ├── Search 3
 ├── Search 4
 └── Search 5
```

This makes downstream processing much more reliable.

---

# 5. Writer Agent

The Writer Agent takes the results of the searches and creates the final research report.

It also uses structured output.

Conceptually:

```text
Search Results
      ↓
Writer Agent
      ↓
ReportData
 ├── Summary
 ├── Detailed Report
 └── Follow-up Questions
```

One important lesson from the transcript is that **the notebook phase is where you refine prompts**.

The recommended workflow is:

```text
Experiment in notebook
        ↓
Iterate on instructions
        ↓
Evaluate results
        ↓
Refine prompts
        ↓
Move stable version into Python module
```

This is a very practical development strategy for agentic applications.

---

# 6. Email Agent

The fourth agent is the Email Agent.

Its responsibilities are simple:

```text
Research Report
      ↓
Email Agent
      ↓
Send Email
```

The email agent has:

* Instructions
* Model
* Email-sending tool

An important improvement in this version is requiring the agent to actually use the tool.

Conceptually:

```text
Agent
  ↓
Must use send_email tool
  ↓
Email sent
```

This prevents the agent from merely describing what it would send instead of actually invoking the tool.

---

# 7. Email vs Push Notifications

The application supports two output mechanisms:

```text
USE_EMAIL=true
      ↓
Email


USE_EMAIL=false
      ↓
Push notification
```

This is another example of configuration through `.env`.

The important architectural lesson is:

> Keep the agent logic independent from the delivery mechanism.

The research system should generate the report; another component determines how that report reaches the user.

---

# 8. `researchmanager.py`

After defining the four agents, the next layer is the **Research Manager**.

This module contains the Python functions that actually execute the agents.

The transcript defines functions conceptually equivalent to:

```text
plan_searches()
perform_searches()
search()
write_report()
send_email()
```

Each function is responsible for one step of the workflow.

---

# 9. Separating Planning from Searching

An important design decision is separating:

```text
Planning
```

from:

```text
Executing searches
```

The workflow becomes:

```text
User Question
     ↓
plan_searches()
     ↓
Search Plan
     ↓
perform_searches()
     ↓
Search Results
```

This separation is useful because it makes each stage:

* Easier to test
* Easier to debug
* Easier to modify
* Easier to evaluate

---

# 10. Parallel Search Execution

The individual searches can run concurrently.

The transcript uses:

```python
asyncio.gather(...)
```

Conceptually:

```text
Search 1 ─┐
Search 2 ─┤
Search 3 ─┼──→ Search Results
Search 4 ─┤
Search 5 ─┘
```

Instead of:

```text
Search 1
   ↓
Search 2
   ↓
Search 3
   ↓
Search 4
   ↓
Search 5
```

### Why this matters

Parallel execution can significantly reduce total latency when the searches are independent.

This is one of the major advantages of using asynchronous Python for agent workflows.

---

# 11. `ResearchManager`

The transcript wraps the entire workflow inside a class:

```python
ResearchManager
```

Its main `run()` method performs:

```text
1. Plan searches
2. Perform searches
3. Write report
4. Send email
```

Conceptually:

```python
async def run(query):
    plan = await plan_searches(query)
    results = await perform_searches(plan)
    report = await write_report(results)
    await send_email(report)
```

The major benefit is that the entire workflow is visible in one place.

This makes the orchestration logic much easier to understand.

---

# 12. Python Generators and `yield`

One of the more important Python concepts introduced is `yield`.

Instead of the `run()` function returning only once at the end, it can produce intermediate results:

```text
run()
 ↓
yield "Planning searches..."
 ↓
yield "Searching..."
 ↓
yield "Writing report..."
 ↓
yield "Sending email..."
 ↓
yield "Complete"
```

This turns the function into a **generator**.

### Why use it here?

Because Gradio can use those intermediate values to update the UI while the research process is still running.

The architecture becomes:

```text
Research Manager
      ↓
yield status
      ↓
Gradio UI
      ↓
Update screen
```

This makes a long-running agent workflow feel interactive rather than frozen.

---

# 13. Gradio

The next major step is adding a user interface using **Gradio**.

The basic UI contains:

* A text box for the research question
* A button to start research
* A field to display the result/status

The application connects:

```text
Gradio Button
      ↓
Callback Function
      ↓
ResearchManager.run()
      ↓
yield status updates
      ↓
Gradio displays updates
```

Gradio is designed for quickly creating web interfaces around Python applications and models.

[Gradio Documentation](https://www.gradio.app/docs/)

---

# 14. Simple UI First

The transcript initially creates a very simple `simple.py`.

This is a valuable development principle:

> **Get the functionality working before worrying about visual polish.**

The first version focuses on:

```text
Does the application work?
```

rather than:

```text
Does the application look beautiful?
```

Only after the underlying workflow works does the transcript improve the UI.

---

# 15. UI Refinement

The transcript then uses a coding agent to improve the UI.

The coding agent:

* Refactors the interface
* Adds styling
* Adds CSS
* Adds JavaScript
* Adds better layout
* Adds colors
* Adds example questions
* Improves presentation
* Adds light/dark mode

The important lesson isn't the specific CSS.

It is this:

> Coding agents can be used as development assistants to transform a functional prototype into a polished interface.

The architecture remains essentially the same:

```text
ResearchManager
      ↓
Gradio
      ↓
Styled UI
```

Only the presentation layer changes.

---

# 16. Live Progress in the UI

One particularly useful feature is showing progress while the research is happening.

For example:

```text
🔎 Planning research...
🔎 Running search 1/5
🔎 Running search 2/5
🔎 Running search 3/5
✍️ Writing report...
📧 Sending report...
✅ Complete
```

This is enabled by the combination of:

* Python generators
* `yield`
* Gradio callbacks

This pattern is useful for any long-running AI workflow.

---

# 17. Why Streaming Status Matters

Deep research may take significant time because it involves:

* Multiple LLM calls
* Multiple web searches
* Parallel processing
* Report generation
* Email delivery

A blank screen can make the application appear broken.

Progress updates communicate:

```text
The system is still working.
```

This improves perceived responsiveness and user experience.

---

# 18. Hugging Face Spaces

Once the application works locally, the transcript deploys it to **Hugging Face Spaces**.

Hugging Face Spaces provides infrastructure for hosting interactive ML/AI applications, including Gradio apps. Spaces are backed by Git repositories, so pushing changes can trigger a rebuild/restart.

### Deployment architecture

```text
Local Python Application
        ↓
Git / Hugging Face Space
        ↓
Hugging Face infrastructure
        ↓
Public Web Application
```

---

# 19. Hugging Face Space Configuration

A Gradio Space generally contains:

```text
app.py
requirements.txt
README.md
other Python modules
```

The Space configuration identifies the SDK as Gradio.

Hugging Face documents configuration through the YAML block in `README.md`, including fields such as `sdk`, `python_version`, and `sdk_version`.

---

# 20. Dependencies

If the application requires packages beyond the default environment, they can be specified in:

```text
requirements.txt
```

Hugging Face Spaces installs these dependencies when building the environment.

This is particularly important when your application depends on packages such as:

```text
openai-agents
gradio
pydantic
other libraries
```

---

# 21. Secrets and Environment Variables

The transcript emphasizes that secrets such as API tokens should **not be hard-coded into the application**.

Instead:

```text
Local development
→ .env


Cloud deployment
→ Space Secrets
```

For example:

```text
OPENAI_API_KEY
PUSHOVER_USER
PUSHOVER_TOKEN
```

should be stored as secrets.

The transcript demonstrates configuring Pushover credentials as Space secrets.

Hugging Face's deployment documentation similarly recommends storing tokens as Space secrets rather than embedding them in source code.

---

# 22. Hugging Face Deployment Flow

The overall deployment process is:

```text
1. Build application locally
          ↓
2. Test Gradio UI
          ↓
3. Create Hugging Face Space
          ↓
4. Select Gradio
          ↓
5. Upload / push application
          ↓
6. Configure dependencies
          ↓
7. Add secrets
          ↓
8. Restart / rebuild
          ↓
9. Open public application
```

Hugging Face's current Spaces documentation covers creating, configuring, and deploying Gradio Spaces.

---

# 23. Push Notifications

The transcript demonstrates using **Pushover** as an alternative to email after deployment.

The flow becomes:

```text
Deep Research
     ↓
Report
     ↓
Pushover
     ↓
Phone notification
```

This is useful because it demonstrates that the research agent isn't restricted to a browser UI.

The same research result can be delivered through different channels.

---

# 24. A More General Architecture

The project can therefore be viewed as four layers:

```text
┌──────────────────────────────┐
│          UI Layer            │
│       Gradio Application     │
└──────────────┬───────────────┘
               │
┌──────────────▼───────────────┐
│      Orchestration Layer     │
│       ResearchManager        │
└──────────────┬───────────────┘
               │
┌──────────────▼───────────────┐
│         Agent Layer          │
│ Planner / Search / Writer    │
│          / Email             │
└──────────────┬───────────────┘
               │
┌──────────────▼───────────────┐
│     External Capabilities    │
│ Web Search / Email / Push    │
└──────────────────────────────┘
```

This separation is a very useful production architecture.

---

# 25. The Clarifying Questions Challenge

The first major extension proposed in the transcript is adding a **clarifying-question stage**.

Instead of:

```text
Question
 ↓
Plan
 ↓
Search
```

the improved flow becomes:

```text
Question
 ↓
Clarifying Questions
 ↓
User Answers
 ↓
Research Plan
 ↓
Search
 ↓
Report
```

This is a powerful improvement because the research agent can better understand the user's actual intent before spending resources on searches.

---

# 26. Clarifications Must Influence the Whole Workflow

A subtle but important point is that clarification should not simply be collected and then ignored.

The answers should influence:

* Search queries
* Search priorities
* Report content
* Final recommendations

For example:

```text
User:
"Research AI agent frameworks."

Clarification:
"Are you interested in enterprise or open-source frameworks?"

User:
"Open-source."

↓
Planner

Searches should now emphasize:
- GitHub activity
- Licensing
- Community
- Open-source adoption
```

The transcript explicitly recommends weaving the clarifications throughout the research process.

---

# 27. Evaluation / Evals

Another major lesson is the importance of **evaluation**.

A research agent generating a polished report doesn't necessarily mean it is producing the correct answer.

You should evaluate:

```text
Did it find the right frameworks?
Did it miss important ones?
Were the sources relevant?
Was the ranking defensible?
Were claims supported?
```

The transcript recommends creating evaluations that measure whether the system is actually achieving the intended outcome.

### Key principle

> **LLMs are responsible for generating content; you are responsible for determining whether that content meets the goal.**

---

# 28. Iterative Agent Development

The transcript recommends a practical development loop:

```text
Build
 ↓
Run
 ↓
Inspect output
 ↓
Inspect traces
 ↓
Add print/debug information
 ↓
Modify prompts
 ↓
Run again
 ↓
Evaluate
 ↓
Repeat
```

This is essentially **empirical prompt engineering**.

Don't assume your prompt is correct.

Test it.

Measure the result.

Improve it.

---

# 29. The Big Challenge: Code vs LLM Orchestration

The final major exercise is to rewrite the Deep Research Agent using **LLM-based orchestration**.

Current architecture:

```text
Python
 ↓
Planner
 ↓
Search
 ↓
Writer
 ↓
Email
```

Alternative architecture:

```text
                Manager Agent
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
    Planner        Search        Writer
     Agent         Agent         Agent
       │             │             │
       └─────────────┼─────────────┘
                     ▼
                  Email
```

The manager agent gets the other agents as tools and decides what to do.

---

# 30. Why LLM Orchestration Could Be Better

The fixed workflow is:

```text
Plan
→ Search
→ Write
→ Email
```

But real research isn't always linear.

Suppose the agent discovers something confusing.

It might want to:

```text
Search
 ↓
Discover ambiguity
 ↓
Ask user clarification
 ↓
Search again
 ↓
Discover another issue
 ↓
Search again
 ↓
Write
```

A manager agent could potentially decide this dynamically.

That is the primary advantage of LLM orchestration.

---

# 31. The Trade-Off

The transcript gives a very important engineering trade-off:

| Code Orchestration | LLM Orchestration         |
| ------------------ | ------------------------- |
| Deterministic      | Autonomous                |
| Predictable        | Flexible                  |
| Easier to debug    | Harder to debug           |
| Fixed workflow     | Dynamic workflow          |
| More reliable      | Potentially more adaptive |
| Less autonomous    | More autonomous           |

The goal isn't to declare one universally better.

Instead:

> **Choose the architecture according to the problem.**

---

# 32. How to Decide

### Use code orchestration when:

* Workflow is known
* Steps are predictable
* Reliability is critical
* Compliance matters
* You need deterministic behavior
* Debugging simplicity is important

### Use LLM orchestration when:

* Workflow is open-ended
* The agent needs to decide what to do next
* Multiple paths may be valid
* User clarification may be needed dynamically
* Exploration is valuable
* Autonomy is a major requirement

---

# 33. Production Architecture

The final production-oriented architecture can look like:

```text
                    ┌──────────────┐
                    │   Gradio UI  │
                    └──────┬───────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │ Research Manager│
                  └────────┬────────┘
                           │
             ┌─────────────┼─────────────┐
             │             │             │
             ▼             ▼             ▼
         Planner        Search         Writer
          Agent          Agent          Agent
             │             │             │
             │             ▼             │
             │        Web Search         │
             │                           │
             └─────────────┬─────────────┘
                           ▼
                      Report Data
                           │
                           ▼
                      Email Agent
                           │
                    ┌──────┴──────┐
                    ▼             ▼
                  Email        Pushover
```

Then:

```text
Local Application
       ↓
Hugging Face Spaces
       ↓
Internet
       ↓
Real Users
```

---

# 34. Most Important Takeaways

## ⭐ 1. Move from notebooks to modules

A notebook is excellent for experimentation.

Python modules are better for reusable applications.

```text
Experiment → Stabilize → Modularize
```

---

## ⭐ 2. Separate agents from orchestration

Agents define capabilities.

The Research Manager defines the workflow.

```text
Agents = What can be done

Manager = When it gets done
```

---

## ⭐ 3. Use `asyncio.gather()` for independent work

Parallel searches can reduce latency.

```text
Independent tasks
      ↓
asyncio.gather()
      ↓
Parallel execution
```

---

## ⭐ 4. Use generators for progress updates

`yield` allows the backend to expose intermediate progress.

This works particularly well with Gradio.

---

## ⭐ 5. Keep configuration outside code

Use:

```text
.env
```

for local configuration and:

```text
Hugging Face Space Secrets
```

for deployed credentials.

---

## ⭐ 6. Build the functional UI first

Start with:

```text
Simple UI
```

then add:

```text
Styling
CSS
Dark mode
Examples
Better UX
```

---

## ⭐ 7. Deploy only after local validation

A good workflow is:

```text
Notebook
 ↓
Python modules
 ↓
Local tests
 ↓
Gradio
 ↓
UI refinement
 ↓
Deployment
```

---

## ⭐ 8. Add clarifying questions

This can substantially improve research quality by making the system understand the user's intent before spending resources.

---

## ⭐ 9. Add evaluations

Never judge an agent purely by:

> "The output looks good."

Measure whether the output actually accomplishes the intended goal.

---

## ⭐ 10. Compare orchestration architectures experimentally

Build both:

```text
Code orchestration
```

and:

```text
LLM orchestration
```

Then measure:

* Accuracy
* Reliability
* Latency
* Cost
* Research completeness
* User satisfaction

That comparison will teach you more than simply reading about the two approaches.

---

# 35. Quick Revision Cheat Sheet

| Concept              | Key Idea                                    |
| -------------------- | ------------------------------------------- |
| Python module        | Reusable application component              |
| `.env`               | External configuration                      |
| Research Manager     | Controls research workflow                  |
| `runner.run()`       | Executes an agent                           |
| `asyncio.gather()`   | Parallel execution                          |
| `yield`              | Intermediate results from a generator       |
| Generator            | Produces values incrementally               |
| Gradio               | Python-based web UI                         |
| Hugging Face Spaces  | Deployment/hosting for AI apps              |
| Space Secrets        | Secure deployment credentials               |
| Pushover             | Alternative notification channel            |
| Structured output    | Typed agent result                          |
| Planner              | Question → search plan                      |
| Search Agent         | Search → research information               |
| Writer Agent         | Research → report                           |
| Email Agent          | Report → delivery                           |
| Evals                | Measure whether the agent achieves its goal |
| Code orchestration   | Predictable workflow                        |
| LLM orchestration    | Dynamic workflow                            |
| Clarifying questions | Improve understanding before research       |

---

# 36. Recommended Deep-Dive Resources

## OpenAI Agents SDK

Start here for the overall framework:

[OpenAI Agents SDK](https://openai.github.io/openai-agents-python/)

Then study:

[Agents & structured outputs](https://openai.github.io/openai-agents-python/agents/)

[Agents SDK Quickstart](https://openai.github.io/openai-agents-python/quickstart/)

---

## Gradio

For building the research application's UI:

[Gradio Documentation](https://www.gradio.app/docs/)

Focus especially on:

* Interfaces
* Blocks
* Events
* Streaming/generator functions
* Custom CSS
* Deployment

---

## Hugging Face Spaces

For deploying the application:

[Hugging Face Spaces Overview](https://huggingface.co/docs/hub/en/spaces-overview)

[Gradio Spaces Guide](https://huggingface.co/docs/hub/en/spaces-sdks-gradio)

[Spaces Dependencies](https://huggingface.co/docs/hub/en/spaces-dependencies)

[Spaces Configuration Reference](https://huggingface.co/docs/hub/spaces-config-reference)

---

## Hugging Face + Gradio APIs

An interesting next step is using a deployed Gradio Space as an API:

[Spaces as API endpoints](https://huggingface.co/docs/hub/spaces-api-endpoints)

This means your Deep Research application can eventually become:

```text
Web UI
   +
API
   +
Agent backend
```

rather than only a browser-based application.

---

## Pydantic

Since structured outputs are heavily used:

[Pydantic Documentation](https://docs.pydantic.dev/)

Study:

* `BaseModel`
* Validation
* Nested models
* JSON Schema
* Type adapters

---

# 37. Suggested Hands-On Exercises

### Beginner

1. Convert the four notebook agents into separate `.py` modules.
2. Create a `ResearchManager`.
3. Add a Gradio textbox and button.
4. Display intermediate status using `yield`.

### Intermediate

5. Add 5–10 parallel searches.
6. Add a configurable number of searches through `.env`.
7. Add email and Pushover as interchangeable outputs.
8. Add custom CSS.
9. Deploy to Hugging Face Spaces.

### Advanced

10. Add a clarifying-question agent.
11. Make the answers influence all subsequent searches.
12. Add evaluation criteria for research quality.
13. Compare different models.
14. Compare sequential vs parallel search performance.
15. Rewrite the workflow using agents-as-tools.
16. Give the manager freedom to ask additional clarification questions.
17. Allow the manager to perform additional searches when the initial research is insufficient.
18. Compare **code orchestration vs LLM orchestration quantitatively**.

---

# 38. Final Mental Model

The entire Day 5 lesson can be summarized as:

```text
                DEEP RESEARCH APPLICATION
                         │
                         ▼
                    Gradio UI
                         │
                         ▼
                 Research Manager
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       Planner         Search          Writer
       Agent           Agents          Agent
          │              │              │
          └──────────────┼──────────────┘
                         │
                         ▼
                    Final Report
                         │
                         ▼
                    Email / Push
                         │
                         ▼
               Hugging Face Spaces
                         │
                         ▼
                    Live Web App
```

### The biggest lesson

The transition from **agent experiment → real AI application** requires more than writing prompts.

You need:

```text
Good agents
+
Reliable orchestration
+
Structured data
+
Async execution
+
User interface
+
Configuration
+
Secrets management
+
Evaluation
+
Deployment
```

The transcript's most important architectural lesson is that **the agent is only one component of the application**. The surrounding software—workflow orchestration, UI, configuration, observability, evaluation, and deployment—is what turns an interesting notebook experiment into a usable product.

I reviewed the uploaded **Week 2, Day 5** content. The main focus is taking the deep-research prototype from the previous day and turning it into a **real Python application with a Gradio UI and Hugging Face Spaces deployment**. 

# Week 2 Day 5 — Deep Research Agent: Python Modules, Gradio UI & Deployment

## 1. Main Goal

The goal of Day 5 is to take the Deep Research Agent built in Jupyter notebooks and turn it into a **deployable AI application**.

The progression is:

```text
Jupyter Notebook
      ↓
Python Modules
      ↓
Research Manager
      ↓
Gradio UI
      ↓
Hugging Face Spaces
      ↓
Live Deep Research Application
```

The project is the classic **agentic AI deep-research use case**. 

---

# 2. The Four-Agent Architecture

The previous day's research system consisted of four agents:

1. **Search Agent**
2. **Planner Agent**
3. **Writer Agent**
4. **Email Agent**

The Day 5 task is to move each of these from notebook cells into reusable Python modules.

```text
                    User Question
                         │
                         ▼
                  Planner Agent
                         │
                  Search Plan
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       Search          Search         Search
       Agent           Agent          Agent
          └──────────────┼──────────────┘
                         ▼
                   Writer Agent
                         │
                         ▼
                   Research Report
                         │
                         ▼
                   Email Agent
```

This modular structure makes the application easier to maintain, test, reuse, and deploy.

The OpenAI Agents SDK similarly treats agents as reusable components defined with instructions, tools, models, and optional structured outputs. ([OpenAI Agents SDK](https://openai.github.io/openai-agents-python/)) ([OpenAI GitHub Pages][1])

---

# 3. Search Agent Module

The Search Agent is responsible for searching the web.

Its configuration includes:

* Instructions
* Web search tool
* Model
* Model settings

The transcript also introduces a useful configuration pattern:

```text
.env
  ↓
DEFAULT_MODEL_NAME
  ↓
Search Agent
```

If a model name is provided through `.env`, the application uses it.

Otherwise, it falls back to a default model.

### Why this is useful

You don't have to modify Python source code every time you want to change the model.

For example:

```text
Development
→ cheaper/faster model

Production
→ more capable model
```

This is a good example of separating **configuration from application code**.

---

# 4. Planner Agent

The Planner Agent converts a user question into multiple searches.

The transcript continues using structured output with Pydantic models.

Conceptually:

```python
class WebSearchItem(BaseModel):
    reason: str
    query: str


class WebSearchPlan(BaseModel):
    searches: list[WebSearchItem]
```

The planner then uses:

```text
output_type = WebSearchPlan
```

so that the result can be handled programmatically.

The current Agents SDK supports Pydantic models and other Pydantic-compatible types as structured output types. ([OpenAI GitHub Pages][2])

### Key idea

Instead of asking:

> "Give me some searches."

the application receives something structurally like:

```text
WebSearchPlan
 ├── Search 1
 ├── Search 2
 ├── Search 3
 ├── Search 4
 └── Search 5
```

This makes downstream processing much more reliable.

---

# 5. Writer Agent

The Writer Agent takes the results of the searches and creates the final research report.

It also uses structured output.

Conceptually:

```text
Search Results
      ↓
Writer Agent
      ↓
ReportData
 ├── Summary
 ├── Detailed Report
 └── Follow-up Questions
```

One important lesson from the transcript is that **the notebook phase is where you refine prompts**.

The recommended workflow is:

```text
Experiment in notebook
        ↓
Iterate on instructions
        ↓
Evaluate results
        ↓
Refine prompts
        ↓
Move stable version into Python module
```

This is a very practical development strategy for agentic applications. 

---

# 6. Email Agent

The fourth agent is the Email Agent.

Its responsibilities are simple:

```text
Research Report
      ↓
Email Agent
      ↓
Send Email
```

The email agent has:

* Instructions
* Model
* Email-sending tool

An important improvement in this version is requiring the agent to actually use the tool.

Conceptually:

```text
Agent
  ↓
Must use send_email tool
  ↓
Email sent
```

This prevents the agent from merely describing what it would send instead of actually invoking the tool.

---

# 7. Email vs Push Notifications

The application supports two output mechanisms:

```text
USE_EMAIL=true
      ↓
Email


USE_EMAIL=false
      ↓
Push notification
```

This is another example of configuration through `.env`.

The important architectural lesson is:

> Keep the agent logic independent from the delivery mechanism.

The research system should generate the report; another component determines how that report reaches the user.

---

# 8. `researchmanager.py`

After defining the four agents, the next layer is the **Research Manager**.

This module contains the Python functions that actually execute the agents.

The transcript defines functions conceptually equivalent to:

```text
plan_searches()
perform_searches()
search()
write_report()
send_email()
```

Each function is responsible for one step of the workflow.

---

# 9. Separating Planning from Searching

An important design decision is separating:

```text
Planning
```

from:

```text
Executing searches
```

The workflow becomes:

```text
User Question
     ↓
plan_searches()
     ↓
Search Plan
     ↓
perform_searches()
     ↓
Search Results
```

This separation is useful because it makes each stage:

* Easier to test
* Easier to debug
* Easier to modify
* Easier to evaluate

---

# 10. Parallel Search Execution

The individual searches can run concurrently.

The transcript uses:

```python
asyncio.gather(...)
```

Conceptually:

```text
Search 1 ─┐
Search 2 ─┤
Search 3 ─┼──→ Search Results
Search 4 ─┤
Search 5 ─┘
```

Instead of:

```text
Search 1
   ↓
Search 2
   ↓
Search 3
   ↓
Search 4
   ↓
Search 5
```

### Why this matters

Parallel execution can significantly reduce total latency when the searches are independent.

This is one of the major advantages of using asynchronous Python for agent workflows.

---

# 11. `ResearchManager`

The transcript wraps the entire workflow inside a class:

```python
ResearchManager
```

Its main `run()` method performs:

```text
1. Plan searches
2. Perform searches
3. Write report
4. Send email
```

Conceptually:

```python
async def run(query):
    plan = await plan_searches(query)
    results = await perform_searches(plan)
    report = await write_report(results)
    await send_email(report)
```

The major benefit is that the entire workflow is visible in one place.

This makes the orchestration logic much easier to understand.

---

# 12. Python Generators and `yield`

One of the more important Python concepts introduced is `yield`.

Instead of the `run()` function returning only once at the end, it can produce intermediate results:

```text
run()
 ↓
yield "Planning searches..."
 ↓
yield "Searching..."
 ↓
yield "Writing report..."
 ↓
yield "Sending email..."
 ↓
yield "Complete"
```

This turns the function into a **generator**.

### Why use it here?

Because Gradio can use those intermediate values to update the UI while the research process is still running.

The architecture becomes:

```text
Research Manager
      ↓
yield status
      ↓
Gradio UI
      ↓
Update screen
```

This makes a long-running agent workflow feel interactive rather than frozen.

---

# 13. Gradio

The next major step is adding a user interface using **Gradio**.

The basic UI contains:

* A text box for the research question
* A button to start research
* A field to display the result/status

The application connects:

```text
Gradio Button
      ↓
Callback Function
      ↓
ResearchManager.run()
      ↓
yield status updates
      ↓
Gradio displays updates
```

Gradio is designed for quickly creating web interfaces around Python applications and models.

[Gradio Documentation](https://www.gradio.app/docs/) ([Hugging Face][3])

---

# 14. Simple UI First

The transcript initially creates a very simple `simple.py`.

This is a valuable development principle:

> **Get the functionality working before worrying about visual polish.**

The first version focuses on:

```text
Does the application work?
```

rather than:

```text
Does the application look beautiful?
```

Only after the underlying workflow works does the transcript improve the UI.

---

# 15. UI Refinement

The transcript then uses a coding agent to improve the UI.

The coding agent:

* Refactors the interface
* Adds styling
* Adds CSS
* Adds JavaScript
* Adds better layout
* Adds colors
* Adds example questions
* Improves presentation
* Adds light/dark mode

The important lesson isn't the specific CSS.

It is this:

> Coding agents can be used as development assistants to transform a functional prototype into a polished interface.

The architecture remains essentially the same:

```text
ResearchManager
      ↓
Gradio
      ↓
Styled UI
```

Only the presentation layer changes.

---

# 16. Live Progress in the UI

One particularly useful feature is showing progress while the research is happening.

For example:

```text
🔎 Planning research...
🔎 Running search 1/5
🔎 Running search 2/5
🔎 Running search 3/5
✍️ Writing report...
📧 Sending report...
✅ Complete
```

This is enabled by the combination of:

* Python generators
* `yield`
* Gradio callbacks

This pattern is useful for any long-running AI workflow.

---

# 17. Why Streaming Status Matters

Deep research may take significant time because it involves:

* Multiple LLM calls
* Multiple web searches
* Parallel processing
* Report generation
* Email delivery

A blank screen can make the application appear broken.

Progress updates communicate:

```text
The system is still working.
```

This improves perceived responsiveness and user experience.

---

# 18. Hugging Face Spaces

Once the application works locally, the transcript deploys it to **Hugging Face Spaces**.

Hugging Face Spaces provides infrastructure for hosting interactive ML/AI applications, including Gradio apps. Spaces are backed by Git repositories, so pushing changes can trigger a rebuild/restart. ([Hugging Face][3])

### Deployment architecture

```text
Local Python Application
        ↓
Git / Hugging Face Space
        ↓
Hugging Face infrastructure
        ↓
Public Web Application
```

---

# 19. Hugging Face Space Configuration

A Gradio Space generally contains:

```text
app.py
requirements.txt
README.md
other Python modules
```

The Space configuration identifies the SDK as Gradio.

Hugging Face documents configuration through the YAML block in `README.md`, including fields such as `sdk`, `python_version`, and `sdk_version`. ([Hugging Face][4])

---

# 20. Dependencies

If the application requires packages beyond the default environment, they can be specified in:

```text
requirements.txt
```

Hugging Face Spaces installs these dependencies when building the environment. ([Hugging Face][5])

This is particularly important when your application depends on packages such as:

```text
openai-agents
gradio
pydantic
other libraries
```

---

# 21. Secrets and Environment Variables

The transcript emphasizes that secrets such as API tokens should **not be hard-coded into the application**.

Instead:

```text
Local development
→ .env


Cloud deployment
→ Space Secrets
```

For example:

```text
OPENAI_API_KEY
PUSHOVER_USER
PUSHOVER_TOKEN
```

should be stored as secrets.

The transcript demonstrates configuring Pushover credentials as Space secrets. 

Hugging Face's deployment documentation similarly recommends storing tokens as Space secrets rather than embedding them in source code. ([Hugging Face][6])

---

# 22. Hugging Face Deployment Flow

The overall deployment process is:

```text
1. Build application locally
          ↓
2. Test Gradio UI
          ↓
3. Create Hugging Face Space
          ↓
4. Select Gradio
          ↓
5. Upload / push application
          ↓
6. Configure dependencies
          ↓
7. Add secrets
          ↓
8. Restart / rebuild
          ↓
9. Open public application
```

Hugging Face's current Spaces documentation covers creating, configuring, and deploying Gradio Spaces. ([Hugging Face][7])

---

# 23. Push Notifications

The transcript demonstrates using **Pushover** as an alternative to email after deployment.

The flow becomes:

```text
Deep Research
     ↓
Report
     ↓
Pushover
     ↓
Phone notification
```

This is useful because it demonstrates that the research agent isn't restricted to a browser UI.

The same research result can be delivered through different channels.

---

# 24. A More General Architecture

The project can therefore be viewed as four layers:

```text
┌──────────────────────────────┐
│          UI Layer            │
│       Gradio Application     │
└──────────────┬───────────────┘
               │
┌──────────────▼───────────────┐
│      Orchestration Layer     │
│       ResearchManager        │
└──────────────┬───────────────┘
               │
┌──────────────▼───────────────┐
│         Agent Layer          │
│ Planner / Search / Writer    │
│          / Email             │
└──────────────┬───────────────┘
               │
┌──────────────▼───────────────┐
│     External Capabilities    │
│ Web Search / Email / Push    │
└──────────────────────────────┘
```

This separation is a very useful production architecture.

---

# 25. The Clarifying Questions Challenge

The first major extension proposed in the transcript is adding a **clarifying-question stage**.

Instead of:

```text
Question
 ↓
Plan
 ↓
Search
```

the improved flow becomes:

```text
Question
 ↓
Clarifying Questions
 ↓
User Answers
 ↓
Research Plan
 ↓
Search
 ↓
Report
```

This is a powerful improvement because the research agent can better understand the user's actual intent before spending resources on searches.

---

# 26. Clarifications Must Influence the Whole Workflow

A subtle but important point is that clarification should not simply be collected and then ignored.

The answers should influence:

* Search queries
* Search priorities
* Report content
* Final recommendations

For example:

```text
User:
"Research AI agent frameworks."

Clarification:
"Are you interested in enterprise or open-source frameworks?"

User:
"Open-source."

↓
Planner

Searches should now emphasize:
- GitHub activity
- Licensing
- Community
- Open-source adoption
```

The transcript explicitly recommends weaving the clarifications throughout the research process. 

---

# 27. Evaluation / Evals

Another major lesson is the importance of **evaluation**.

A research agent generating a polished report doesn't necessarily mean it is producing the correct answer.

You should evaluate:

```text
Did it find the right frameworks?
Did it miss important ones?
Were the sources relevant?
Was the ranking defensible?
Were claims supported?
```

The transcript recommends creating evaluations that measure whether the system is actually achieving the intended outcome. 

### Key principle

> **LLMs are responsible for generating content; you are responsible for determining whether that content meets the goal.**

---

# 28. Iterative Agent Development

The transcript recommends a practical development loop:

```text
Build
 ↓
Run
 ↓
Inspect output
 ↓
Inspect traces
 ↓
Add print/debug information
 ↓
Modify prompts
 ↓
Run again
 ↓
Evaluate
 ↓
Repeat
```

This is essentially **empirical prompt engineering**.

Don't assume your prompt is correct.

Test it.

Measure the result.

Improve it.

---

# 29. The Big Challenge: Code vs LLM Orchestration

The final major exercise is to rewrite the Deep Research Agent using **LLM-based orchestration**.

Current architecture:

```text
Python
 ↓
Planner
 ↓
Search
 ↓
Writer
 ↓
Email
```

Alternative architecture:

```text
                Manager Agent
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
    Planner        Search        Writer
     Agent         Agent         Agent
       │             │             │
       └─────────────┼─────────────┘
                     ▼
                  Email
```

The manager agent gets the other agents as tools and decides what to do.

---

# 30. Why LLM Orchestration Could Be Better

The fixed workflow is:

```text
Plan
→ Search
→ Write
→ Email
```

But real research isn't always linear.

Suppose the agent discovers something confusing.

It might want to:

```text
Search
 ↓
Discover ambiguity
 ↓
Ask user clarification
 ↓
Search again
 ↓
Discover another issue
 ↓
Search again
 ↓
Write
```

A manager agent could potentially decide this dynamically.

That is the primary advantage of LLM orchestration.

---

# 31. The Trade-Off

The transcript gives a very important engineering trade-off:

| Code Orchestration | LLM Orchestration         |
| ------------------ | ------------------------- |
| Deterministic      | Autonomous                |
| Predictable        | Flexible                  |
| Easier to debug    | Harder to debug           |
| Fixed workflow     | Dynamic workflow          |
| More reliable      | Potentially more adaptive |
| Less autonomous    | More autonomous           |

The goal isn't to declare one universally better.

Instead:

> **Choose the architecture according to the problem.**

---

# 32. How to Decide

### Use code orchestration when:

* Workflow is known
* Steps are predictable
* Reliability is critical
* Compliance matters
* You need deterministic behavior
* Debugging simplicity is important

### Use LLM orchestration when:

* Workflow is open-ended
* The agent needs to decide what to do next
* Multiple paths may be valid
* User clarification may be needed dynamically
* Exploration is valuable
* Autonomy is a major requirement

---

# 33. Production Architecture

The final production-oriented architecture can look like:

```text
                    ┌──────────────┐
                    │   Gradio UI  │
                    └──────┬───────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │ Research Manager│
                  └────────┬────────┘
                           │
             ┌─────────────┼─────────────┐
             │             │             │
             ▼             ▼             ▼
         Planner        Search         Writer
          Agent          Agent          Agent
             │             │             │
             │             ▼             │
             │        Web Search         │
             │                           │
             └─────────────┬─────────────┘
                           ▼
                      Report Data
                           │
                           ▼
                      Email Agent
                           │
                    ┌──────┴──────┐
                    ▼             ▼
                  Email        Pushover
```

Then:

```text
Local Application
       ↓
Hugging Face Spaces
       ↓
Internet
       ↓
Real Users
```

---

# 34. Most Important Takeaways

## ⭐ 1. Move from notebooks to modules

A notebook is excellent for experimentation.

Python modules are better for reusable applications.

```text
Experiment → Stabilize → Modularize
```

---

## ⭐ 2. Separate agents from orchestration

Agents define capabilities.

The Research Manager defines the workflow.

```text
Agents = What can be done

Manager = When it gets done
```

---

## ⭐ 3. Use `asyncio.gather()` for independent work

Parallel searches can reduce latency.

```text
Independent tasks
      ↓
asyncio.gather()
      ↓
Parallel execution
```

---

## ⭐ 4. Use generators for progress updates

`yield` allows the backend to expose intermediate progress.

This works particularly well with Gradio.

---

## ⭐ 5. Keep configuration outside code

Use:

```text
.env
```

for local configuration and:

```text
Hugging Face Space Secrets
```

for deployed credentials.

---

## ⭐ 6. Build the functional UI first

Start with:

```text
Simple UI
```

then add:

```text
Styling
CSS
Dark mode
Examples
Better UX
```

---

## ⭐ 7. Deploy only after local validation

A good workflow is:

```text
Notebook
 ↓
Python modules
 ↓
Local tests
 ↓
Gradio
 ↓
UI refinement
 ↓
Deployment
```

---

## ⭐ 8. Add clarifying questions

This can substantially improve research quality by making the system understand the user's intent before spending resources.

---

## ⭐ 9. Add evaluations

Never judge an agent purely by:

> "The output looks good."

Measure whether the output actually accomplishes the intended goal.

---

## ⭐ 10. Compare orchestration architectures experimentally

Build both:

```text
Code orchestration
```

and:

```text
LLM orchestration
```

Then measure:

* Accuracy
* Reliability
* Latency
* Cost
* Research completeness
* User satisfaction

That comparison will teach you more than simply reading about the two approaches.

---

# 35. Quick Revision Cheat Sheet

| Concept              | Key Idea                                    |
| -------------------- | ------------------------------------------- |
| Python module        | Reusable application component              |
| `.env`               | External configuration                      |
| Research Manager     | Controls research workflow                  |
| `runner.run()`       | Executes an agent                           |
| `asyncio.gather()`   | Parallel execution                          |
| `yield`              | Intermediate results from a generator       |
| Generator            | Produces values incrementally               |
| Gradio               | Python-based web UI                         |
| Hugging Face Spaces  | Deployment/hosting for AI apps              |
| Space Secrets        | Secure deployment credentials               |
| Pushover             | Alternative notification channel            |
| Structured output    | Typed agent result                          |
| Planner              | Question → search plan                      |
| Search Agent         | Search → research information               |
| Writer Agent         | Research → report                           |
| Email Agent          | Report → delivery                           |
| Evals                | Measure whether the agent achieves its goal |
| Code orchestration   | Predictable workflow                        |
| LLM orchestration    | Dynamic workflow                            |
| Clarifying questions | Improve understanding before research       |

---

# 36. Recommended Deep-Dive Resources

## OpenAI Agents SDK

Start here for the overall framework:

[OpenAI Agents SDK](https://openai.github.io/openai-agents-python/) ([OpenAI GitHub Pages][1])

Then study:

[Agents & structured outputs](https://openai.github.io/openai-agents-python/agents/) ([OpenAI GitHub Pages][2])

[Agents SDK Quickstart](https://openai.github.io/openai-agents-python/quickstart/) ([OpenAI GitHub Pages][8])

---

## Gradio

For building the research application's UI:

[Gradio Documentation](https://www.gradio.app/docs/) ([Hugging Face][3])

Focus especially on:

* Interfaces
* Blocks
* Events
* Streaming/generator functions
* Custom CSS
* Deployment

---

## Hugging Face Spaces

For deploying the application:

[Hugging Face Spaces Overview](https://huggingface.co/docs/hub/en/spaces-overview) ([Hugging Face][7])

[Gradio Spaces Guide](https://huggingface.co/docs/hub/en/spaces-sdks-gradio) ([Hugging Face][3])

[Spaces Dependencies](https://huggingface.co/docs/hub/en/spaces-dependencies) ([Hugging Face][5])

[Spaces Configuration Reference](https://huggingface.co/docs/hub/spaces-config-reference) ([Hugging Face][4])

---

## Hugging Face + Gradio APIs

An interesting next step is using a deployed Gradio Space as an API:

[Spaces as API endpoints](https://huggingface.co/docs/hub/spaces-api-endpoints) ([Hugging Face][9])

This means your Deep Research application can eventually become:

```text
Web UI
   +
API
   +
Agent backend
```

rather than only a browser-based application.

---

## Pydantic

Since structured outputs are heavily used:

[Pydantic Documentation](https://docs.pydantic.dev/)

Study:

* `BaseModel`
* Validation
* Nested models
* JSON Schema
* Type adapters

---

# 37. Suggested Hands-On Exercises

### Beginner

1. Convert the four notebook agents into separate `.py` modules.
2. Create a `ResearchManager`.
3. Add a Gradio textbox and button.
4. Display intermediate status using `yield`.

### Intermediate

5. Add 5–10 parallel searches.
6. Add a configurable number of searches through `.env`.
7. Add email and Pushover as interchangeable outputs.
8. Add custom CSS.
9. Deploy to Hugging Face Spaces.

### Advanced

10. Add a clarifying-question agent.
11. Make the answers influence all subsequent searches.
12. Add evaluation criteria for research quality.
13. Compare different models.
14. Compare sequential vs parallel search performance.
15. Rewrite the workflow using agents-as-tools.
16. Give the manager freedom to ask additional clarification questions.
17. Allow the manager to perform additional searches when the initial research is insufficient.
18. Compare **code orchestration vs LLM orchestration quantitatively**.

---

# 38. Final Mental Model

The entire Day 5 lesson can be summarized as:

```text
                DEEP RESEARCH APPLICATION
                         │
                         ▼
                    Gradio UI
                         │
                         ▼
                 Research Manager
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       Planner         Search          Writer
       Agent           Agents          Agent
          │              │              │
          └──────────────┼──────────────┘
                         │
                         ▼
                    Final Report
                         │
                         ▼
                    Email / Push
                         │
                         ▼
               Hugging Face Spaces
                         │
                         ▼
                    Live Web App
```

### The biggest lesson

The transition from **agent experiment → real AI application** requires more than writing prompts.

You need:

```text
Good agents
+
Reliable orchestration
+
Structured data
+
Async execution
+
User interface
+
Configuration
+
Secrets management
+
Evaluation
+
Deployment
```

The transcript's most important architectural lesson is that **the agent is only one component of the application**. The surrounding software—workflow orchestration, UI, configuration, observability, evaluation, and deployment—is what turns an interesting notebook experiment into a usable product. 

### References

* [OpenAI Agents SDK](https://openai.github.io/openai-agents-python/) — agents, tools, orchestration, tracing, sessions, and production workflows. ([OpenAI GitHub Pages][1])
* [OpenAI Agents — Structured Outputs](https://openai.github.io/openai-agents-python/agents/) — `output_type`, Pydantic models, and typed agent results. ([OpenAI GitHub Pages][2])
* [OpenAI Agents SDK Quickstart](https://openai.github.io/openai-agents-python/quickstart/) — project setup and first agent. ([OpenAI GitHub Pages][8])
* [Gradio Documentation](https://www.gradio.app/docs/) — building Python-based interfaces.
* [Hugging Face Spaces](https://huggingface.co/docs/hub/en/spaces-overview) — deployment and hosting. ([Hugging Face][7])
* [Gradio Spaces](https://huggingface.co/docs/hub/en/spaces-sdks-gradio) — deploying Gradio applications. ([Hugging Face][3])
* [Hugging Face Spaces Dependencies](https://huggingface.co/docs/hub/en/spaces-dependencies) — `requirements.txt` and environment dependencies. ([Hugging Face][5])
* [Hugging Face Spaces Configuration](https://huggingface.co/docs/hub/spaces-config-reference) — Space metadata and runtime configuration. ([Hugging Face][4])
* [Spaces as API Endpoints](https://huggingface.co/docs/hub/spaces-api-endpoints) — turning a deployed Gradio Space into an API. ([Hugging Face][9])
* [Pydantic](https://docs.pydantic.dev/) — Python data validation and structured models.



The **best next step** after this lesson is to actually implement the project yourself: first get the four-agent version working locally, then add the Gradio UI, deploy it, and finally attempt the harder **LLM-orchestrated version** and compare the two architectures.

[1]: https://openai.github.io/openai-agents-python/?utm_source=chatgpt.com "OpenAI Agents SDK"
[2]: https://openai.github.io/openai-agents-python/agents/?utm_source=chatgpt.com "Agents - OpenAI Agents SDK"
[3]: https://huggingface.co/docs/hub/en/spaces-sdks-gradio?utm_source=chatgpt.com "Gradio Spaces · Hugging Face"
[4]: https://huggingface.co/docs/hub/spaces-config-reference?utm_source=chatgpt.com "Spaces Configuration Reference · Hugging Face"
[5]: https://huggingface.co/docs/hub/en/spaces-dependencies?utm_source=chatgpt.com "Handling Spaces Dependencies in Gradio Spaces · Hugging Face"
[6]: https://huggingface.co/docs/inference-providers/guides/building-first-app?utm_source=chatgpt.com "Building Your First AI App with Inference Providers · Hugging Face"
[7]: https://huggingface.co/docs/hub/en/spaces-overview?utm_source=chatgpt.com "Spaces Overview · Hugging Face"
[8]: https://openai.github.io/openai-agents-python/quickstart/?utm_source=chatgpt.com "Quickstart - OpenAI Agents SDK"
[9]: https://huggingface.co/docs/hub/spaces-api-endpoints?utm_source=chatgpt.com "Spaces as API endpoints · Hugging Face"
