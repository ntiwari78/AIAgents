# Week 1, Day 2 — AI Agents & Agentic Design Patterns

## What Is an AI Agent?

There is no single universally accepted definition of an AI agent. The lecture presents three useful definitions:

1. **An AI system that can do work independently for you.**
2. **A system where an LLM controls the workflow.**
3. **An LLM with tools in a loop to achieve a goal.**

The third definition is presented as the most solidified:

> **An AI agent is an LLM with tools in a loop to achieve a goal.**

### LLM vs. AI Agent

An **LLM** itself only:

* Takes input tokens.
* Predicts likely tokens that follow those inputs.
* Generates text.

An LLM does **not inherently take actions**.

An **agent** is the surrounding code/system that:

1. Calls the LLM.
2. Interprets its output.
3. Uses that output to decide what to do.
4. Calls tools when necessary.
5. Feeds results back to the LLM.
6. Repeats until the goal is achieved.

The important mental model is:

> **LLM = token generation engine**
> **Agent = code that interprets LLM outputs and uses them to accomplish a goal.**

---

## Workflows vs. Agents

According to the terminology discussed from [Anthropic's *Building Effective Agents*](https://www.anthropic.com/research/building-effective-agents), both workflows and agents are types of **agentic systems**.

### Workflows

A workflow has:

* A series of LLM calls.
* Predefined paths through the system.
* Known scenarios that can be tested in advance.
* Potentially an LLM deciding which predefined path to take.

The key characteristic is that the overall structure is **relatively well-defined**.

**Example:** A deep-research system that:

1. Asks clarifying questions.
2. Performs web searches.
3. Builds a report.
4. Synthesizes the findings.
5. Produces the final output.

### Agents

Agents are more open-ended:

* The LLM continuously decides what happens next.
* There is no fixed path through the system.
* The system can interact with an environment.
* Feedback is incorporated into subsequent decisions.
* The agent continues until it determines that the goal has been achieved.

A useful distinction is:

| Workflow                           | Agent                              |
| ---------------------------------- | ---------------------------------- |
| Predefined paths                   | Open-ended paths                   |
| Known sequence/structure           | Dynamic decisions                  |
| LLM may choose among defined paths | LLM determines what to do next     |
| Easier to predict and test         | More flexible but less predictable |
| Example: deep research workflow    | Example: coding/operator agent     |

There can also be **gray areas** between the two.

---

# Five Workflow Design Patterns

The lecture discusses five patterns associated with Anthropic's framework. These should be treated as **ideas rather than rigid rules**.

## 1. Prompt Chaining

A large task is divided into smaller subtasks.

```text
LLM Call 1
    ↓
LLM Call 2
    ↓
LLM Call 3
    ↓
Output
```

The output from one LLM call becomes the input to the next.

**Use when:** A complex task can naturally be broken into sequential steps.

---

## 2. Routing

An initial LLM determines which specialized LLM/prompt should handle the request.

```text
                 ┌→ Technical LLM
Input → Router ──┼→ Business LLM
                 └→ Other LLM
```

For example, a customer-support system might classify a request as:

* Technical support
* Business support
* Other

Each route can then use a prompt containing information specifically relevant to that category.

### Key idea

Routing provides **separation of concerns**, rather than putting all possible information into one enormous prompt.

---

## 3. Parallelization

Independent tasks are performed simultaneously.

```text
              ┌→ LLM 1 ─┐
Input → Code ─┼→ LLM 2 ─┼→ Aggregator → Output
              └→ LLM 3 ─┘
```

Here, the **code** determines the tasks and distributes them to multiple LLM calls.

**Use when:** Several pieces of work can be performed independently.

---

## 4. Orchestrator-Worker

This looks similar to parallelization but has an important difference: **an LLM dynamically decides how to break up the task and how to synthesize the results.**

```text
                 ┌→ LLM Worker 1 ─┐
Input → LLM      ├→ LLM Worker 2 ─┼→ LLM Synthesizer → Output
       Orchestrator└→ LLM Worker 3 ─┘
```

The orchestrator:

* Receives the larger problem.
* Determines how to divide it.
* Determines what should be sent to the workers.

The synthesizer then uses an LLM to combine the results.

### Difference from parallelization

**Parallelization:** Code decides the tasks.

**Orchestrator-worker:** An LLM decides how to divide and combine the tasks.

---

## 5. Evaluator-Optimizer

One LLM generates an output, while another evaluates it.

```text
        ┌──────────────────────┐
        ↓                      │
Generator → Evaluator ── Reject┘
                  │
                Accept
                  ↓
                Output
```

The evaluator can:

* Reject the result and send it back for improvement.
* Accept the result and allow it to proceed.

This is often described as:

> **LLM as a judge**

---

# Agent Pattern

Unlike the five workflow patterns, an agent has an open-ended feedback loop.

```text
           ┌───────────────┐
           ↓               │
Input → LLM → Tool/Action → Environment
           ↑               │
           └── Feedback ───┘
                   │
                   ↓
             Goal achieved?
                   │
                  Yes
                   ↓
                 Stop
```

The agent continues deciding what to do until it believes the goal has been achieved.

This closely matches the lecture's central definition:

> **LLM + tools + loop + goal = AI agent**

---

# Risks of Agentic AI

The flexibility of agentic AI is also one of its biggest risks.

## 1. Unpredictability

Agentic systems can have:

* Unpredictable paths.
* Non-deterministic outputs.
* Variable execution times.
* Variable costs.

The same system may behave differently between runs.

### Mitigation: Monitoring

Two important monitoring approaches are:

#### Observability

Capture traces showing what is happening during LLM calls and agent execution.

#### Evals

Measure whether the system is actually performing well.

The lecture emphasizes that the strongest evaluation is often tied to the **real-world business outcome**.

For example:

* Number of leads generated.
* Number of customers acquired.
* Revenue generated.

An evaluation can also directly assess the agent's output, such as using an LLM as a judge.

---

# Guardrails

**Guardrails** are code and checks placed around an agent system to keep its behavior within intended boundaries.

They can check:

* Inputs.
* Outputs.
* Individual agent steps.
* The overall agent system.

The basic idea is:

```text
Input
  ↓
Guardrail
  ↓
Agent
  ↓
Guardrail
  ↓
Output
```

Monitoring tells you **what is happening**.

Guardrails help ensure **what is allowed to happen**.

---

# Agentic Trap #1: Starting With the Solution

A common mistake is to start with:

> "I need an AI agent."

Instead, start with:

> **"What business problem are we trying to solve?"**

An organization might request a "strategy agent" or "culture agent" without clearly identifying the underlying problem.

The lecture's example is a proposed **culture agent**. After probing further, the actual problem turned out to involve:

* Morale.
* Employee attrition.
* People leaving the company.

Once the actual problem is identified, an AI agent becomes only **one possible solution**.

Other solutions might include something as simple as an employee survey.

### Correct approach

```text
Business problem
       ↓
Measurable outcome
       ↓
Possible solutions
       ↓
AI agent, if appropriate
       ↓
Measure results
       ↓
Iterate
```

---

# The Importance of Measurable Outcomes

LLMs are good at generating **believable content**.

However:

> **Believable does not necessarily mean accurate or useful.**

The way to determine whether an agent is actually useful is to measure it against a meaningful outcome.

For example:

```text
Agent generates sales emails
          ↓
Measure response rate
          ↓
Measure leads generated
          ↓
Measure revenue
          ↓
Improve the system
```

The business metric provides a feedback mechanism for improving the AI system.

---

# Agentic Trap #2: Anthropomorphizing Agents

Another common mistake is treating LLMs as if they are human team members.

For example, someone might immediately design an architecture containing:

* Trade Manager Agent
* Market Research Agent
* Trading Agent
* Risk Manager Agent

This may sound intuitively correct because it resembles a human organization.

But the lecture warns against starting this way.

### Why?

LLMs are not people.

They are **token-generation engines** that predict plausible tokens based on their input.

Therefore, agent architectures should not automatically mirror human organizational structures.

---

# Start Simple

The recommended approach is:

1. Start with **one LLM**.
2. Give it **one prompt**.
3. Give it **one objective**.
4. Define a measurable business outcome.
5. Measure performance.
6. Experiment with more complex architectures.
7. Keep complexity only when it improves the outcome.

```text
One LLM
   ↓
Measure
   ↓
Experiment
   ↓
More LLMs / routing / agents / patterns
   ↓
Measure again
   ↓
Keep what improves performance
```

A multi-agent architecture might ultimately be excellent—but it should be reached **intentionally through experimentation**, not because it looks good or resembles how humans organize teams.

---

# Hallucinations: An AI Engineer's Responsibility

For an ordinary AI user, it is reasonable to complain when an LLM hallucinates.

For an **AI engineer building a system**, the lecture takes a different position.

The LLM's job is to predict plausible tokens.

The engineer's job is to build the surrounding system so that those outputs can be used reliably.

That includes:

* Guardrails.
* Evaluation.
* Monitoring.
* Accuracy checks.
* Feedback loops.
* Business-outcome measurement.

The key principle is:

> **Don't blame hallucination; engineer the system to account for it.**

---

# "Agentic Engineer" Is Ambiguous

The term **agentic engineer** can mean two different things:

### Meaning 1 — Using agents to engineer

For example, using coding agents to write software.

### Meaning 2 — Engineering agents

Building AI agent systems themselves.

Because both meanings exist, the recommendation is to be **explicit about which meaning is intended**.

---

# Major Takeaways

1. **An AI agent can be understood as an LLM with tools in a loop to achieve a goal.**

2. **Keep the LLM and the agent conceptually separate.**
   The LLM generates tokens; the surrounding code interprets those tokens and enables action.

3. **Workflows have relatively predefined paths.**

4. **Agents are open-ended and continuously decide what to do next.**

5. **The five workflow patterns are:**

   * Prompt chaining
   * Routing
   * Parallelization
   * Orchestrator-worker
   * Evaluator-optimizer

6. **These patterns are ideas, not rigid rules.**

7. **Agentic AI introduces unpredictability in behavior, outputs, paths, and cost.**

8. **Monitoring, observability, evals, and guardrails help mitigate those risks.**

9. **Always start with the business problem, not the desire to build an agent.**

10. **Define a measurable outcome before deciding on the architecture.**

11. **Start simple—one LLM, one prompt, one objective—and add complexity only when measurement shows that it improves performance.**

12. **LLMs generate believable outputs; the AI engineer's job is to make those outputs useful, reliable, and aligned with the business objective.**

13. **Don't anthropomorphize LLMs or assume that agent architectures should mirror human organizations.**

14. **Don't blame hallucinations—design the system to handle them.**

---

## References

* [Anthropic — Building Effective Agents](https://www.anthropic.com/research/building-effective-agents)
* [OpenAI — Agents](https://platform.openai.com/docs/guides/agents)

**Source:** Uploaded lecture transcript. 
