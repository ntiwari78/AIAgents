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
