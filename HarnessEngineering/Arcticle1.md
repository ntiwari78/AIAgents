
# How to Design an Agent Harness: Six Decisions That Turn a Model into a Worker You Can Leave Alone

> **Core idea:** An AI agent is not just a model. The surrounding system determines what the model can see, what it can do, what it remembers, how it recovers, what boundaries it operates within, and how completion is verified.

An agent can tell you that a task is complete even when its tests were never executed.

It can perform extremely well for twenty minutes and then forget important rules from the beginning of the task.

You may have to repeat the same instructions every time you start a session.

And if you cannot walk away because an approval dialog is constantly waiting for you, the system is not truly autonomous.

Simply switching to a more capable model does not necessarily solve these problems.

Many of them exist in the software surrounding the model.

That surrounding software is commonly called the **agent harness**.

---

# 1. What Is an Agent Harness?

A useful mental model is:

```text
Agent = Model + Harness
```

The model provides the reasoning capability.

The harness determines how that capability is used.

A harness can control six broad areas:

1. **The loop** that keeps the agent working
2. **The tools** available to it
3. **The memory/context** it receives
4. **The state** that survives crashes and session boundaries
5. **The permissions and boundaries** within which it operates
6. **The definition of completion** and how results are verified

Some of this infrastructure is provided by the agent vendor.

The rest belongs to the application developer or user.

That second category includes things such as:

* Instruction files
* Tests
* Permissions
* Repository structure
* Validation rules
* Acceptance criteria
* Persistent task state
* Definition of "done"

Even if you have never explicitly designed a harness, you already have one.

Every rule you repeatedly paste into a chat is effectively part of your harness.

![img1](https://github.com/ntiwari78/AIAgents/blob/main/images/1_six_surfaces.png)


## Analysis

The important conceptual shift is:

> **Prompt engineering controls what you ask the model to do. Harness engineering controls the environment in which the model works.**

This becomes increasingly important as agents move from answering questions to performing multi-step work.

OpenAI now explicitly describes the harness as the control plane around an agent, including the agent loop, tool routing, approvals, tracing, recovery, and run state. ([OpenAI Developers][2])

Anthropic similarly describes harness engineering as critical for long-running agents because agents must maintain progress across multiple context windows and sessions. ([Anthropic][3])

---

# 2. Real-World Harness Patterns

![img2](https://github.com/ntiwari78/AIAgents/blob/main/images/2_three_harnesses.png)

The article highlights three very different approaches.

## DoorDash: Harness as a Platform

DoorDash built a platform around engineering agents.

The platform provides:

* Isolated execution environments
* Repositories
* Development tools
* Credentials
* YAML-based playbooks
* Scoped permissions
* An internal gateway
* Logging and auditing
* Automated validation

DoorDash reported that its Flux platform automated approximately **130,000 engineering tasks in a single month**, including more than **25,000 automated code reviews per week**. ([DoorDash][4])

The important architectural idea is not the raw number.

It is the separation between:

```text
Task Definition
       ↓
Playbook
       ↓
Agent
       ↓
Scoped Tools
       ↓
Sandbox
       ↓
Validation
       ↓
Result
```

The playbook can decide which parts should be performed by an agent and which should be handled deterministically by ordinary software.

### Analysis

This is a strong pattern for enterprise environments.

Instead of giving every agent unrestricted access to every internal system:

```text
Agent → Everything
```

you create:

```text
Agent → Gateway → Approved Capabilities
```

That makes authorization, auditing, and incident investigation much easier.

DoorDash's own description emphasizes sandboxing, an MCP gateway, playbooks, and repeatable invocation surfaces as the primitives that make delegation scalable. ([DoorDash][4])

---

# 3. OpenAI: Harness as a Repository

OpenAI describes a different approach.

Rather than building an entirely separate platform, the repository itself becomes an important part of the harness.

The repository contains:

* Architecture documentation
* Product specifications
* Execution plans
* Quality information
* Security rules
* Reliability rules
* Skills
* Tests
* Linters
* Generated documentation
* Decision records

A particularly important principle is:

> **Don't make one giant instruction file. Give the agent a map to the information it needs.**

OpenAI reports using a relatively small `AGENTS.md` as an entry point, with deeper information organized in a structured `docs/` directory. ([OpenAI][1])

The idea is similar to a table of contents:

```text
AGENTS.md
    ↓
Architecture
    ↓
Product docs
    ↓
Design docs
    ↓
Execution plans
    ↓
Quality / security / reliability docs
```

Instead of:

```text
One giant 1,000-page instruction file
```

### Analysis

This is essentially **progressive disclosure for agents**.

The agent initially receives only the highest-signal information.

When it needs more context, it follows links or reads the relevant files.

This aligns closely with Anthropic's concept of **context engineering**, where the objective is not simply to maximize context size but to provide the smallest useful set of high-signal information for the current task. ([Anthropic][5])

OpenAI's real-world experiment also reported approximately 1,500 pull requests being opened and merged over five months by a small team using Codex, while emphasizing that the repository structure, tools, feedback loops, and constraints were essential to achieving that throughput. ([OpenAI][1])

---

# 4. Anthropic: Harness as Role Separation

Anthropic demonstrated another model:

```text
Planner
   ↓
Generator
   ↓
Evaluator
   ↓
Feedback
   ↓
Generator
```

The three agents have different responsibilities.

### Planner

Transforms a short request into a more detailed product specification.

### Generator

Builds the application.

### Evaluator

Actually tests and evaluates the result.

The evaluator can interact with the running application using browser automation rather than simply reading the generated code.

Anthropic reported an experiment where a solo agent spent approximately 20 minutes and $9 but produced a broken application, while the fuller harness took roughly six hours and $200 and produced a substantially more functional result. ([Anthropic][6])

The lesson is not:

> "Always use three agents."

The lesson is:

> **Separate creation from evaluation when the creator is not reliable at judging its own work.**

Anthropic explicitly describes self-evaluation as a recurring problem and found that separating the generator from the evaluator created a stronger feedback loop. ([Anthropic][6])

### Analysis

This resembles traditional software engineering:

```text
Developer → Code
Reviewer  → Review
QA        → Test
Developer → Fix
```

The difference is that the workers can now be AI agents.

This is one of the most important patterns in modern agent architecture:

> **Do not make the component that produced the result the only component responsible for deciding whether the result is correct.**

---

# 5. The Six Harness Decisions

The article's central framework consists of six decisions.

---

![img3](https://github.com/ntiwari78/AIAgents/blob/main/images/3_the_agent_loop.png)

## Decision 1: The Loop and Where It Stops

An agent normally works through an iterative loop:

```text
User Task
   ↓
Model
   ↓
Tool Call
   ↓
Tool Result
   ↓
Model
   ↓
Tool Call
   ↓
...
   ↓
Completion
```

The model requests an action.

The harness executes it.

The result goes back to the model.

The process continues until the model stops requesting actions or the harness stops it.

The important design question is:

> **What actually constitutes completion?**

Do not use:

```text
Agent says: "Done."
```

as your definition of success.

Instead define something observable.

For example:

```text
Done =
    Application starts
    AND
    Tests pass
    AND
    Required endpoint returns 200
    AND
    No critical lint errors
```

### Recommended practices

#### 1. Define completion before execution

Write down what success means.

Bad:

> Improve error handling.

Better:

> Requests without an `id` must return HTTP 400 with a structured error response, and the corresponding automated test must pass.


#### 2. Decide what happens after failure

Choose explicitly between:

```text
Failure → Retry
```

or:

```text
Failure → Pause for human intervention
```

Otherwise the agent may invent its own recovery strategy.

#### 3. Set hard limits

Use:

* Maximum turns
* Maximum runtime
* Maximum token budget
* Maximum retries
* Maximum attempts on a single file/task

An agent that has repeatedly failed to fix the same problem may simply continue consuming resources.

#### 4. Log unattended runs

A six-hour autonomous run should leave an inspectable trail.

Useful information includes:

* Tool calls
* Errors
* State transitions
* Files changed
* Test results
* Decisions
* Evaluator feedback

### Analysis

This is closely related to reliability engineering.

A production agent should behave more like a controlled workflow than an infinitely patient chatbot.

The harness should answer:

```text
What happened?
Why did it happen?
What did the agent change?
What evidence says it succeeded?
What should happen if it fails?
```

OpenAI's current agent architecture explicitly includes recovery, run state, tool routing, and observability as parts of the harness rather than leaving everything to the model. ([OpenAI Developers][7])

---

# Decision 2: Which Tools Can the Agent See?

The model does not directly see your entire system.

It sees descriptions of the tools made available to it.

Therefore:

> **The tool menu is part of the agent's world model.**

If you expose dozens or hundreds of tools, you increase:

* Context consumption
* Tool-selection ambiguity
* Potential misuse
* Maintenance cost
* Security exposure

### Recommended practices

#### Load tools progressively

Instead of exposing everything:

```text
Agent
 ├── Tool A
 ├── Tool B
 ├── Tool C
 ├── Tool D
 ├── Tool E
 ├── Tool F
 └── ...
```

use:

```text
Agent
   ↓
Capability Directory
   ↓
Load only what is required
```

OpenAI's current Skills mechanism follows a similar progressive-disclosure idea: the harness can expose a directory of capabilities and allow the model to read detailed instructions only when relevant. ([OpenAI Developers][8])

#### Make errors useful

Instead of:

```text
Request failed.
```

return:

```text
Error:
field = customer_id
problem = required field missing

Expected:
customer_id: UUID

Suggested next action:
retrieve the customer record before retrying.
```

This gives the model information that can guide recovery.

#### Remove unused tools

If a capability has not been used for a long period, question whether it should be exposed.

#### Avoid overlapping tool names

For example:

```text
search_customer
find_customer
lookup_customer
get_customer
```

can create unnecessary ambiguity if the tools are functionally similar.

Prefer clear, differentiated capabilities.

### Analysis

Tool design is not merely an API-design problem.

It is also **model-interface design**.

A narrow tool such as:

```text
issue_refund(order_id, amount, reason)
```

is easier to authorize and validate than:

```text
execute_sql(query)
```

because the first exposes a business operation while the second exposes an entire database.

This is especially important for agent security.

---

# Decision 3: What Stays in Memory?

Agent context is finite.

Long-running tasks continually accumulate:

* User instructions
* Tool results
* Code
* Errors
* Plans
* Decisions
* Conversation history
* External information

Eventually something has to be removed, summarized, or compressed.

This creates an important distinction:

```text
Context
    ≠
Persistent knowledge
```

### Recommended practices

#### 1. Do not assume a huge context window solves everything

More context does not automatically mean better reasoning.

Anthropic describes context as a finite resource and argues that good context engineering focuses on high-signal information rather than simply maximizing the amount of text passed to the model. ([Anthropic][5])

#### 2. Separate work into stages

For example:

```text
Research
   ↓
Specification
   ↓
Planning
   ↓
Implementation
   ↓
Testing
```

Each stage can produce a durable artifact.

#### 3. Pin critical rules

Some information must survive context compression.

Examples:

```text
Never access production.
Never expose credentials.
Do not modify the public API contract.
All database migrations require tests.
```

These rules should live in durable, easily discoverable artifacts and, where appropriate, in enforceable system controls.

#### 4. Restart when context becomes contaminated

A fresh context can sometimes be better than endlessly carrying forward an incorrect assumption.

Anthropic's long-running agent research explicitly discusses context resets as a way to give an agent a clean slate while transferring state through structured artifacts. ([Anthropic][6])

### Analysis

This leads to an important principle:

> **Memory should preserve state, not preserve noise.**

A good harness distinguishes:

```text
Facts
Rules
Decisions
Progress
Temporary reasoning
Raw tool output
```

These should not all have equal persistence.

---

# Decision 4: What Survives a Crash?

Long-running agents will fail.

The process may:

* Crash
* Time out
* Exhaust context
* Lose network connectivity
* Hit a tool failure
* Be interrupted by a human
* Reach a resource limit

If all useful state lives only inside the conversation, recovery becomes difficult.

A durable harness should therefore externalize important state.

## Recommended persistent artifacts

### `SPEC.md`

Defines what is being built.

Ideally written by the human or derived from an approved specification.

### `PLAN.md`

Describes the implementation plan and acceptance criteria.

### `PROGRESS.md`

Records:

* What has been completed
* What is currently being worked on
* What failed
* What should happen next

### `DECISIONS.md`

Records important architectural and product decisions.

Prefer append-only behavior.

### Optional: `features.json`

A machine-readable feature checklist:

```json
{
  "authentication": "passed",
  "search": "passed",
  "payments": "failed",
  "notifications": "pending"
}
```

### Git history

Commit meaningful increments.

Then recovery becomes:

```text
Inspect state
   ↓
Inspect Git history
   ↓
Read progress
   ↓
Read next task
   ↓
Continue
```

rather than:

```text
"What were we doing six hours ago?"
```

### Analysis

Anthropic's long-running-agent work strongly reinforces this idea.

Their approach uses structured artifacts to hand state from one context/session to another rather than depending entirely on conversational history. ([Anthropic][3])

OpenAI's harness engineering work similarly treats repository-local documents, execution plans, decision logs, and other versioned artifacts as a system of record. ([OpenAI][1])

A useful mental model is:

> **If the next agent cannot find it in the workspace, it probably does not exist operationally.**

---

# Decision 5: What Is the Agent Allowed to Touch?

This is one of the most important decisions.

An agent running on a developer's machine may potentially have access to:

* Source code
* SSH keys
* Cloud credentials
* Environment variables
* Local files
* Databases
* Network services
* CLI sessions
* Browser sessions

The problem is not necessarily malicious behavior.

A prompt injection or malicious piece of external content may cause an otherwise well-intentioned agent to perform an unsafe action.

## Use multiple boundaries

A safer architecture looks like:

```text
                Agent
                  |
             Harness
                  |
        +---------+---------+
        |                   |
   Tool policy          Sandbox
        |                   |
        +---------+---------+
                  |
          Approved systems
```

### Filesystem isolation

Limit what the agent can read and write.

### Network isolation

Allow connections only to approved destinations where possible.

### Credential isolation

Avoid giving long-lived credentials to autonomous processes.

Prefer:

* Short-lived credentials
* Narrow scopes
* Task-specific access
* Revocation
* External authorization

### Gateway-based access

For enterprise systems:

```text
Agent
  ↓
Policy Gateway
  ↓
Approved API
```

This creates an auditable control point.

---

## Do Not Rely Entirely on Approval Dialogs

A human approval button sounds safe:

```text
Agent → "Can I execute this?"
Human → "Approve"
```

But repeated prompts create **approval fatigue**.

Anthropic reports that Claude Code users approved roughly 93% of permission prompts in its telemetry, and argues that repeatedly asking users to approve actions can lead to reduced attention. ([Anthropic][9])

Anthropic also reports that sandboxing reduced permission prompts by 84% in its internal Claude Code usage by moving more of the security boundary into the environment itself. ([Anthropic][10])

### Better model

Instead of:

```text
Ask for permission every time
```

use:

```text
Define safe operating boundaries
        ↓
Allow routine actions inside them
        ↓
Block actions outside them
        ↓
Escalate genuinely consequential actions
```

### Analysis

This is a major architectural lesson.

A permission dialog is **a human-interface control**.

A sandbox is **an enforcement control**.

The latter is generally stronger because the security boundary does not depend entirely on a person making a good decision hundreds of times.

OpenAI's current sandbox guidance similarly recommends isolated workloads, restricted network access, and separated credentials. ([OpenAI Developers][11])

---

# Decision 6: Who Decides That the Work Is Finished?

One of the most dangerous patterns is:

```text
Agent writes code
       ↓
Agent reviews code
       ↓
Agent says "Looks good"
```

The agent is effectively grading its own homework.

A stronger architecture is:

```text
Generator
    ↓
Artifact
    ↓
Independent Evaluator
    ↓
Tests / Browser / Static Analysis
    ↓
Pass or Fail
    ↓
Generator
```

## Recommended practices

### 1. Use an independent evaluation step

A fresh agent or deterministic test system should evaluate the result.

### 2. Run the application

For a web application:

```text
Start server
   ↓
Open browser
   ↓
Perform user actions
   ↓
Verify result
```

For an API:

```text
Start service
   ↓
Call endpoint
   ↓
Validate response
   ↓
Inspect side effects
```

For a CLI:

```text
Execute command
   ↓
Check exit code
   ↓
Validate output
```

### 3. Build an evaluation dataset

Collect real failures.

For example:

```text
evals/
├── login-invalid-password
├── missing-customer
├── duplicate-order
├── expired-token
├── concurrent-update
└── malformed-request
```

The evaluation set should evolve as the system encounters new failure modes.

### 4. Test multiple times

Agent behavior is probabilistic.

One successful run is weak evidence.

Repeated evaluation gives you a better picture of reliability.

### 5. Look for confident failure

The most dangerous output is not necessarily:

```text
ERROR
```

It can be:

```text
Everything is working correctly.
```

when it is not.

### Analysis

This is one of the strongest ideas in the article.

A production agent needs **observable evidence of completion**, not a textual claim of completion.

Anthropic's evaluator architecture demonstrates this principle directly: the evaluator interacts with the running application, tests functionality, and grades it against explicit criteria. ([Anthropic][6])

OpenAI similarly describes using tests, review agents, browser interaction, logs, metrics, and traces to make agent-generated work verifiable. ([OpenAI][1])

---

# 6. The Practical Weekend Version

![img4](https://github.com/ntiwari78/AIAgents/blob/main/images/4_the_weekend_version.png)

If you do not want to build a sophisticated multi-agent platform, start small.

The article's practical recommendation can be distilled into five components.

## 1. Create a short instruction file

For example:

```text
AGENTS.md
```

Keep it concise.

Its job is to tell the agent:

* What the repository is
* What rules matter
* Where important documentation lives
* How to run tests
* How to validate work
* What it must never do

Do not turn it into an encyclopedia.

---

## 2. Convert repeated mistakes into automation

If you repeatedly tell the agent:

> "Don't do this."

ask whether it should instead become:

```text
Lint rule
Test
Type check
CI check
Policy
Validation script
```

The progression is:

```text
Repeated human instruction
          ↓
Documented rule
          ↓
Automated check
          ↓
Machine-enforced invariant
```

This is much more durable.

OpenAI's harness engineering experience strongly emphasizes converting architectural and quality expectations into linters, structural tests, and other mechanically enforced constraints. ([OpenAI][1])

---

## 3. Maintain persistent state

Start with:

```text
SPEC.md
PLAN.md
PROGRESS.md
DECISIONS.md
```

Add structured machine-readable state if needed.

---

## 4. Pin critical safety rules

Examples:

```text
Never deploy directly to production.

Never commit secrets.

Never modify database production data.

Always run tests before declaring completion.

Never bypass the approval boundary for financial actions.
```

Where possible, enforce these outside the model as well.

---

## 5. Build a small evaluation set

Start with perhaps:

```text
20 real tasks
×
3 runs each
```

Track:

* Success
* Failure
* Failure type
* Cost
* Runtime
* Human intervention
* Regression

The goal is not to create a perfect benchmark.

The goal is to create a feedback loop.

---

# 7. A Useful Architecture

Putting everything together:

```text
                       HUMAN
                         |
                    Task / Goal
                         |
                         v
                 +----------------+
                 |     HARNESS    |
                 +----------------+
                    |     |     |
          +---------+     |     +---------+
          |               |               |
          v               v               v
      Context           Tools          Policies
      Manager          / MCP           / Limits
          |               |               |
          +---------------+---------------+
                          |
                          v
                       MODEL
                          |
                          v
                    Agent Loop
                          |
             +------------+------------+
             |                         |
             v                         v
         Workspace                 External
         / Sandbox                 Systems
             |                         |
             +------------+------------+
                          |
                          v
                    Verification
                          |
          +---------------+---------------+
          |               |               |
        Tests           Lint          Browser
          |               |               |
          +---------------+---------------+
                          |
                          v
                     Evaluator
                          |
                 +--------+--------+
                 |                 |
                FAIL              PASS
                 |                 |
                 v                 v
              Retry            Complete
                 |
                 +------> Agent
```

This is the deeper idea behind the article.

The model is only one component.

The surrounding system determines whether the model can operate reliably.

---

# 8. The Most Important Design Principle

The article's six decisions can be reduced to one question:

> **What must the system provide so that the model can safely continue working without constant human supervision?**

That leads to six corresponding answers:

| Problem                         | Harness capability         |
| ------------------------------- | -------------------------- |
| Agent stops too early           | Completion criteria        |
| Agent gets confused             | Curated context            |
| Agent lacks capability          | Well-designed tools        |
| Agent loses progress            | Persistent state           |
| Agent can cause damage          | Sandboxing and permissions |
| Agent cannot judge its own work | Independent evaluation     |

---

# 9. Harness Complexity Should Not Become Permanent

One of the most important nuances in the article is that a harness itself can become technical debt.

Every harness component encodes an assumption:

```text
"The model cannot reliably do X."
```

But models improve.

Eventually:

```text
Model capability improves
        ↓
Old workaround becomes unnecessary
        ↓
Harness complexity becomes overhead
```

Anthropic explicitly describes this phenomenon in its harness research. Its earlier harness used context resets and sprint decomposition for models that needed them, while later model improvements allowed parts of that structure to be removed. ([Anthropic][6])

Therefore:

> **A good harness should evolve with the model.**

Do not permanently preserve scaffolding merely because it once solved a problem.

---

# 10. Cost Versus Reliability

A sophisticated harness can cost considerably more than a simple agent.

For example:

```text
Simple agent
    ↓
Fast
Cheap
Less verification
```

versus:

```text
Planner
    ↓
Generator
    ↓
Evaluator
    ↓
Retry
    ↓
Regression tests
    ↓
Final verification
```

The second architecture can consume substantially more:

* Tokens
* Compute
* Time
* Infrastructure
* Engineering effort

The question therefore should not be:

> "Is the harness expensive?"

It should be:

> **"Is the additional reliability worth the cost for this task?"**

For a five-minute disposable task, probably not.

For:

* Production software
* Security-sensitive automation
* Financial operations
* Infrastructure changes
* Long-running research
* Customer-facing workflows

the answer may be very different.

---

# 11. A Practical Maturity Model

A useful way to implement these ideas incrementally is:

## Level 0: Raw Model

```text
Prompt → Model → Answer
```

Suitable for simple informational tasks.

---

## Level 1: Tool-Using Agent

```text
Prompt
  ↓
Model
  ↓
Tools
  ↓
Model
```

The agent can perform actions.

---

## Level 2: Persistent Workspace

Add:

```text
SPEC.md
PLAN.md
PROGRESS.md
```

Now the agent can recover from interruptions.

---

## Level 3: Verification

Add:

```text
Tests
Lint
Type checking
Smoke tests
```

The agent can no longer rely entirely on its own claim of success.

---

## Level 4: Sandboxing

Add:

```text
Filesystem isolation
Network restrictions
Credential boundaries
Scoped tools
```

The blast radius becomes controlled.

---

## Level 5: Independent Evaluation

Add:

```text
Generator
   ↓
Evaluator
   ↓
Feedback
   ↓
Generator
```

Now the system can actively improve its own output.

---

## Level 6: Self-Improving Harness

Capture failures and turn them into:

```text
New test
New lint
New rule
New tool
New documentation
New evaluator criterion
```

This creates:

```text
Agent failure
     ↓
Human analysis
     ↓
Harness improvement
     ↓
Future failures decrease
```

That is where harness engineering becomes a compounding advantage.

---

# 12. Final Takeaways

The central message of the article is not that you need a complicated multi-agent system.

It is that **autonomy is an engineering problem, not simply a model-capability problem.**

The most important lessons are:

1. **Define "done" objectively.**
2. **Keep the tool surface small and intentional.**
3. **Treat context as a scarce resource.**
4. **Persist important state outside the conversation.**
5. **Use OS-level or infrastructure-level boundaries for security.**
6. **Do not depend entirely on approval dialogs.**
7. **Do not let the generator be the only evaluator.**
8. **Turn repeated failures into automated checks.**
9. **Use real-world evaluation tasks rather than only synthetic benchmarks.**
10. **Review and simplify the harness as models improve.**

The deepest mental model is:

```text
              MODEL
                |
        +-------+-------+
        |               |
      Context          Tools
        |               |
        +-------+-------+
                |
             Harness
        +-------+-------+
        |       |       |
      State   Safety   Eval
        |       |       |
        +-------+-------+
                |
             Outcome
```

The model supplies intelligence.

The harness supplies **structure, memory, capability, boundaries, and feedback**.

That is what turns an LLM from something that can answer a question into something that can perform a job.

---

# External References for Deep Dive

The references below are grouped by topic so you can study the concepts behind the article rather than relying only on the article itself.

## Agent Harness Engineering

**[1] OpenAI — Harness engineering: leveraging Codex in an agent-first world**
Primary source for repository-as-harness, `AGENTS.md`, documentation structure, linters, feedback loops, agent legibility, and autonomous coding.

**[2] Anthropic — Harness design for long-running application development**
Primary source for planner/generator/evaluator architecture, context resets, evaluator agents, Playwright-based verification, and the cost/reliability tradeoff.

**[3] Martin Fowler — Harness engineering for coding agent users**
Useful independent technical perspective on the emerging concept of harness engineering and the relationship between models, coding agents, guides, and feedback sensors.

## Context Engineering

**[4] Anthropic — Effective context engineering for AI agents**
Deep dive into context as a finite resource, progressive disclosure, context curation, tool descriptions, and long-running agent context management.

## Sandboxing and Security

**[5] OpenAI — Sandbox Agents**
Explains the separation between the agent harness/control plane and sandbox execution environment.

**[6] OpenAI — Sandbox Security**
Covers workload isolation, network restrictions, credentials, and security boundaries.

**[7] Anthropic — Beyond permission prompts: making Claude Code more secure and autonomous**
Useful for understanding filesystem/network sandboxing and approval fatigue.

**[8] Anthropic — How we contain Claude across products**
Deeper discussion of human approval, sandboxing, virtual machines, and limiting the blast radius of autonomous agents.

## Enterprise Agent Platforms

**[9] DoorDash — Delegating Engineering Work to Cloud-Based Agents**
Primary source for Flux, sandboxed engineering agents, playbooks, MCP gateway, automated workflows, and large-scale agent execution.

## Agent Architecture

**[10] OpenAI — Agents API Architecture**
Useful for understanding the separation between harness, execution environment, application server, sessions, tools, and recovery.

**[11] OpenAI — Agents API**
Practical documentation for durable sessions, orchestration, context compaction, tools, environments, and long-running agents.

## Repository-Based Agent Instructions

**[12] OpenAI — Skills**
Shows how reusable agent capabilities can be discovered and loaded progressively rather than putting every instruction into one enormous prompt.

## GitHub / Software Engineering Controls

For production coding agents, combine harness techniques with normal software-engineering controls such as:

* Protected branches
* Required reviews
* CODEOWNERS
* CI status checks
* Automated tests
* Static analysis
* Audit logs

GitHub documents protected branches and required reviews here, while CODEOWNERS can require appropriate owners to approve changes to specific parts of a repository.

### Verified external references

* [OpenAI — Harness engineering: leveraging Codex in an agent-first world](https://openai.com/index/harness-engineering/?utm_source=chatgpt.com)
* [Anthropic — Harness design for long-running application development](https://www.anthropic.com/engineering/harness-design-long-running-apps?utm_source=chatgpt.com)
* [Anthropic — Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents?utm_source=chatgpt.com)
* [Anthropic — Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents?utm_source=chatgpt.com)
* [OpenAI — Sandbox Agents](https://developers.openai.com/api/docs/guides/agents/sandboxes?utm_source=chatgpt.com)
* [OpenAI — Sandbox Security](https://developers.openai.com/api/docs/guides/agents-api/environments/security?utm_source=chatgpt.com)
* [Anthropic — Beyond permission prompts: making Claude Code more secure and autonomous](https://www.anthropic.com/engineering/claude-code-sandboxing?utm_source=chatgpt.com)
* [Anthropic — How we contain Claude across products](https://www.anthropic.com/engineering/how-we-contain-claude?utm_source=chatgpt.com)
* [DoorDash — Delegating Engineering Work to Cloud-Based Agents](https://careersatdoordash.com/blog/delegating-engineering-work-to-cloud-based-agents/?utm_source=chatgpt.com)
* [Martin Fowler — Harness engineering for coding agent users](https://martinfowler.com/articles/harness-engineering.html?utm_source=chatgpt.com)
* [OpenAI — Agents API Architecture](https://developers.openai.com/api/docs/guides/agents-api/architecture?utm_source=chatgpt.com)
* [OpenAI — Skills](https://developers.openai.com/api/docs/guides/tools-skills?utm_source=chatgpt.com)
* [GitHub — About protected branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches?utm_source=chatgpt.com)
* [GitHub — About CODEOWNERS](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners?utm_source=chatgpt.com)

**One particularly useful takeaway for your agent-engineering studies:** think of a harness as the **engineering layer around the LLM**. Your previous work on agents, MCP, tools, memory, RAG, structured outputs, evaluation, and observability all fit naturally into this model. The next step is to stop thinking of these as isolated features and start designing them as one **control system around the agent**.

[1]: https://openai.com/index/harness-engineering/?utm_source=chatgpt.com "Harness engineering: leveraging Codex in an agent-first world | OpenAI"
[2]: https://developers.openai.com/api/docs/guides/agents/sandboxes?utm_source=chatgpt.com "Sandbox Agents | OpenAI API"
[3]: https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents?utm_source=chatgpt.com "Effective harnesses for long-running agents \ Anthropic"
[4]: https://careersatdoordash.com/blog/delegating-engineering-work-to-cloud-based-agents/?utm_source=chatgpt.com "Delegating Engineering Work To Cloud-Based Agents - DoorDash"
[5]: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents?utm_source=chatgpt.com "Effective context engineering for AI agents \ Anthropic"
[6]: https://www.anthropic.com/engineering/harness-design-long-running-apps?utm_source=chatgpt.com "Harness design for long-running application development \ Anthropic"
[7]: https://developers.openai.com/api/docs/guides/agents-api/architecture?utm_source=chatgpt.com "Architecture | OpenAI API"
[8]: https://developers.openai.com/api/docs/guides/tools-skills?utm_source=chatgpt.com "Skills | OpenAI API"
[9]: https://www.anthropic.com/engineering/how-we-contain-claude?utm_source=chatgpt.com "How we contain Claude across products \ Anthropic"
[10]: https://www.anthropic.com/engineering/claude-code-sandboxing?utm_source=chatgpt.com "Making Claude Code more secure and autonomous with sandboxing \ Anthropic"
[11]: https://developers.openai.com/api/docs/guides/agents-api/environments/security?utm_source=chatgpt.com "Sandbox security | OpenAI API"
