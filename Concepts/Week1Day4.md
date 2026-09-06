# Week 1, Day 4 — Tools, Agent Loops & Autonomy

## 1. Orchestrating LLMs Is Simple

Orchestrating multiple LLMs essentially means:

1. Make an LLM call.
2. Take its output.
3. Use that output as input to another LLM.
4. Repeat as needed.

> **LLM orchestration = connecting multiple LLM calls together to achieve a task.**

Day 4 applies the same idea to **tools, agent loops, and autonomy**. 

---

# 2. The AI Agent Landscape

The lecture divides the agent ecosystem into four broad categories.

## AI Builders

Tools that allow technical and non-technical users to construct agent solutions, often through visual or drag-and-drop interfaces.

Examples mentioned include:

* Flowise
* Other visual agent-building environments

## Agent Development Products

Products focused on using agents to build software or perform engineering tasks.

The distinction from builders can sometimes be blurry.

## Agent Runtimes

A **runtime** is infrastructure capable of executing agents in production.

Examples discussed include:

* AWS Lambda and other general cloud infrastructure
* AWS Bedrock AgentCore
* Vertex AI Agent Engine
* Other specialized agent runtimes

The basic idea is:

```text
Agent code
    ↓
Runtime
    ↓
Agent executes in production
    ↓
Interacts with tools / other agents / environment
```

## Agent Frameworks

Frameworks are libraries that make it easier for developers to build agents.

Examples mentioned:

* OpenAI Agents SDK
* CrewAI
* LangGraph
* LangChain
* Google ADK
* AWS Strands Agents

---

# 3. What Is an Agent Framework?

The lecture deliberately demystifies agent frameworks.

They are essentially:

> **Libraries containing utilities and helper code that make it easier to build agents based on multiple LLM calls.**

They are **abstraction layers** over functionality that would otherwise need to be implemented repeatedly.

Three important things they commonly handle are:

### 1. Orchestration

Connecting multiple LLM calls together.

### 2. Tool Calling

Connecting an LLM to functions/tools that allow it to perform actions.

### 3. Structured Outputs

Getting an LLM to produce structured data that can be interpreted by normal code.

Together, these capabilities make it easier to build an **agent loop**.

---

# 4. Agent Frameworks Differ in How Opinionated They Are

Some frameworks are relatively lightweight.

Examples:

* OpenAI Agents SDK
* Google ADK
* AWS Strands Agents

Other frameworks are more **opinionated**.

An opinionated framework provides a prescribed way of building systems.

### Advantages

* Faster development.
* Powerful functionality out of the box.
* Easier to build complex systems if you follow the framework's conventions.

### Disadvantages

* You may struggle when you want to do something differently.
* You become more dependent on the framework's design philosophy.

The lecture describes **CrewAI** as both opinionated and "batteries included."

---

# 5. Why Build an Agent Loop From Scratch?

During Week 1, the course intentionally avoids agent frameworks.

Instead, the goal is to **hand-build an agent loop**.

Why?

Because once you understand the underlying mechanism, you can better understand what frameworks are actually doing for you.

The progression is:

```text
Build it yourself
      ↓
Understand the mechanics
      ↓
Understand what frameworks abstract
      ↓
Use frameworks effectively
```

The important lesson is not that frameworks are bad.

It's that you should understand what they're hiding.

---

# 6. What Are Tools?

Tools are fundamental to agentic AI.

An LLM naturally generates text.

Tools give the LLM the ability to perform actions such as:

* Querying a database.
* Running SQL.
* Looking up a stock price.
* Searching the internet.
* Editing a Google Sheet.
* Performing other operations exposed through functions.

Conceptually:

```text
LLM
 ↓
Tool
 ↓
External system / data
 ↓
Tool result
 ↓
LLM
```

---

# 7. Tool Calling / Function Calling

Tool calling is also called **function calling**.

The basic idea is:

1. Write a function.
2. Tell the LLM that the function is available.
3. Give the LLM a user's request.
4. The LLM determines whether the tool should be used.
5. Your software executes the function.
6. The result is returned to the LLM.
7. The LLM continues processing.

For example:

```text
User:
"How much does it cost to go to Paris?"

        ↓

LLM:
"I need the ticket-price tool."

        ↓

Your code:
get_ticket_price("Paris")

        ↓

Tool result

        ↓

LLM:
Generates the final response
```

---

# 8. The "Magic" Behind Tool Calling

Tool calling can look magical:

> "The LLM decided to run my Python function!"

But the lecture emphasizes that the reality is much simpler.

The LLM is **still only generating tokens**.

It can generate output that effectively says:

```text
Use the SQL tool.
```

or:

```text
Call get_ticket_price with destination = Paris.
```

Then **your software interprets that output and actually calls the function**.

So the real architecture is:

```text
User
 ↓
Your software
 ↓
LLM
 ↓
LLM generates tool-call instructions
 ↓
Your software interprets them
 ↓
Your software executes the tool
 ↓
Tool result
 ↓
Your software
 ↓
LLM
 ↓
Final response
```

> **The LLM doesn't directly reach into your computer and execute the function. Your code does.**

---

# 9. Why Do We Need an LLM?

An important question arises:

> If we already have `get_ticket_price("Paris")`, why not simply call it directly?

Because the user might not explicitly say:

```text
get ticket price Paris
```

They might say:

> "How much does it cost to go to Paris?"

The LLM understands the **intent and meaning** of natural language and translates it into a specific action.

This is the important capability:

> **Natural language → structured action**

That ability is a major reason LLMs are useful as the decision-making layer around tools.

---

# 10. Autonomy Is Also Less Magical Than It Sounds

The lecture demonstrates a simple example where an LLM is told:

* Move North.
* Move East.
* Move South.
* Move West.

The model chooses one.

For example:

```text
LLM → "Move North"
```

Some external code could interpret that output and physically move a robot north.

We might describe this as:

> "The LLM autonomously decided to move north."

But mechanically, it is still:

```text
Input tokens
     ↓
LLM
     ↓
Output tokens
     ↓
Code interprets output
     ↓
Action
```

So the lecture's provocative takeaway is:

> **What we call autonomy can ultimately be implemented through prompting, interpreting the LLM's output, and taking actions based on that output.**

---

# 11. The Agent Loop

The same principle applies to a full agent.

It can look conceptually like:

```text
              ┌───────────────┐
              │               ↓
User → LLM → Tool → Result → LLM
              │               │
              └───────────────┘
                       ↓
                 Goal achieved?
                       ↓
                      Yes
                       ↓
                     Stop
```

But the actual implementation is:

```text
while not goal_achieved:

    response = call_llm(context)

    if response_requests_tool:
        result = execute_tool(...)
        context += result
    else:
        return response
```

The **software owns the loop**.

The **software owns the tool execution**.

The **LLM generates the tokens that guide the next step**.

---

# 12. Structured Outputs

Another important capability provided by agent frameworks is **structured outputs**.

Normally:

```text
LLM → Text
```

With structured outputs:

```text
LLM → JSON → Python object
```

The process is roughly:

1. Define a JSON schema.
2. Include the desired structure in the request.
3. Ask the LLM to produce output matching that structure.
4. Parse the JSON using normal code.
5. Turn it into a usable Python object.

Conceptually:

```text
LLM
 ↓
JSON matching schema
 ↓
Parser
 ↓
Python object
 ↓
Program logic
```

The lecture emphasizes that this isn't magic either.

It is largely:

> **Prompting + JSON + normal program logic.**

Agent frameworks simply package this functionality so developers don't have to repeatedly implement it themselves.

---

# 13. System Prompts

A **system prompt** establishes the overall context for an LLM.

It can specify:

* The role the LLM should play.
* The desired style.
* General instructions.
* Background information.
* Other contextual information.

For example:

```python
messages = [
    {
        "role": "system",
        "content": "You are a snarky GPT assistant."
    }
]
```

The system prompt establishes the overall framing for the interaction.

---

# 14. Conversation History

An LLM API call can contain a sequence of messages.

For example:

```text
system
user
assistant
user
assistant
user
```

This sequence represents the **conversation history**.

The final user message is the one the LLM is expected to respond to.

---

# 15. The Illusion of Memory

One of the most important foundational concepts in the lecture is that an LLM is fundamentally **stateless**.

Suppose you make this call:

```text
User: My name is Ed.
```

Then make a completely separate call:

```text
User: What's my name?
```

The LLM does not inherently remember the first call.

It only sees the input provided to the second call.

---

## How Does ChatGPT Appear to Remember?

The application sends the conversation history again.

Instead of:

```text
User: What's my name?
```

the model receives something like:

```text
System: You are a helpful assistant.

User: My name is Ed.

Assistant: Hi Ed, nice to meet you.

User: What's my name?
```

The LLM can now answer:

> "Your name is Ed."

It appears to remember, but the application has actually provided the relevant previous context again.

### Mental model

```text
Conversation history
        ↓
Input sequence
        ↓
LLM
        ↓
Next-token prediction
```

> **LLM memory is largely an illusion created by repeatedly supplying context.**

---

# 16. System Prompt + Conversation History + Memory

The overall input can be thought of as:

```text
┌──────────────────────────┐
│ System prompt             │
│ Overall instructions      │
│ Background context        │
│ Memory                    │
├──────────────────────────┤
│ Conversation history      │
│ User                      │
│ Assistant                 │
│ User                      │
│ Assistant                 │
├──────────────────────────┤
│ Current user message      │
└──────────────────────────┘
              ↓
             LLM
```

External "memory" features can also work by dynamically inserting relevant information into this input context.

---

# 17. Notebook Execution Matters

The lecture highlights an important practical issue when working in Jupyter notebooks.

A variable can be redefined in multiple cells.

For example:

```python
messages = [...]
```

and later:

```python
messages = [...]
```

The value of `messages` depends on **which cell was executed most recently**.

Therefore, changing a cell isn't enough—you must actually execute it.

This is a useful habit to develop when working interactively with notebooks.

---

# 18. Day 4's Big Picture

The lecture connects all of these concepts together:

```text
                    ┌──────────────┐
                    │ System       │
                    │ Prompt       │
                    └──────┬───────┘
                           ↓
User → Conversation History → LLM
                           ↓
                    Tool decision
                           ↓
                     Your code
                           ↓
                       Tool call
                           ↓
                      Tool result
                           ↓
                          LLM
                           ↓
                  More tool calls?
                     ↙          ↘
                   Yes           No
                    ↓             ↓
                  Loop          Answer
```

The important realization is that the architecture is fundamentally ordinary software wrapped around an LLM.

---

# 19. Key Takeaways

1. **Agent frameworks are abstraction layers.**
   They simplify orchestration, tool calling, structured outputs, and agent loops.

2. **Tools give LLMs new capabilities.**
   Examples include databases, SQL, web search, stock prices, and spreadsheets.

3. **Tool calling isn't magic.**
   The LLM generates instructions; your software interprets those instructions and executes the actual function.

4. **Autonomy isn't magic either.**
   Prompt → token output → program interprets output → action.

5. **The agent loop belongs to your software.**
   Your code repeatedly calls the LLM and executes tools based on the LLM's outputs.

6. **Structured outputs turn LLM-generated JSON into useful program objects.**

7. **System prompts establish the overall role and context.**

8. **Conversation history provides context for subsequent LLM calls.**

9. **LLMs are stateless by themselves.**
   What looks like memory is largely the result of repeatedly providing conversation history and relevant context.

10. **Build an agent loop yourself at least once.**
    Doing so makes the abstractions provided by frameworks much easier to understand.

11. **Don't over-focus on framework selection.**
    Different frameworks often provide different implementations of broadly similar underlying capabilities.

12. **The fundamental mechanism remains simple:**

```text
Tokens in
   ↓
LLM
   ↓
Tokens out
   ↓
Code interprets output
   ↓
Action / tool
   ↓
Result fed back to LLM
   ↓
Repeat
```

**Source:** Uploaded Week 1, Day 4 lecture transcript. 
