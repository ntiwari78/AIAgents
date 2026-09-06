# Week 1, Day 3 — Orchestrating LLMs

## Overview

Week 1, Day 3 focuses on **orchestrating multiple LLM calls** and putting the agentic design patterns from Day 2 into practice.

The main objectives are:

1. Make many LLM API calls and become comfortable with the API.
2. Experiment with different LLM providers and models.
3. Use Python to orchestrate multiple LLM calls.
4. Apply agentic workflow patterns.
5. Use an **LLM as a judge** to evaluate outputs.

> The key idea is that Python code can glue multiple LLM calls together, passing the output from one call into another and creating increasingly sophisticated agentic systems.



---

# 1. You Don't Need Paid Models

The course can be completed using **free models**.

Paid models generally provide stronger results, but free models can still be useful for learning.

With weaker or free models, you may need to:

* Iterate more.
* Refine prompts.
* Take smaller steps.
* Ask for less complex outputs.
* Experiment more.

The lecture emphasizes that this experimentation is actually an important part of learning AI engineering.

---

# 2. Major LLM Providers

The lecture introduces several major providers and model families.

## OpenAI

The lecture describes three sizes:

| Size   | Example  |
| ------ | -------- |
| Small  | GPT Nano |
| Medium | GPT Mini |
| Large  | GPT      |

The smaller models tend to be **cheaper and faster**, while larger models tend to provide stronger capabilities at higher cost.

---

## Anthropic

Anthropic's models are in the **Claude** family:

| Size   | Model         |
| ------ | ------------- |
| Small  | Claude Haiku  |
| Medium | Claude Sonnet |
| Large  | Claude Opus   |

---

## Google

Google's **Gemini** family also has different sizes:

* Gemini Flash Light
* Gemini Flash
* Gemini Pro

---

# 3. Other Providers and Platforms

## DeepSeek

DeepSeek is discussed as a popular Chinese AI startup known particularly for its open-source models.

Models can be:

* Run locally in smaller versions.
* Accessed through cloud providers for larger versions.

---

## Groq

**Groq** and **Grok** are different things.

### Groq

* Spelled with a **Q**.
* An inference provider.
* Runs open-source models in the cloud.
* Known for very fast inference.

### Grok

* Spelled with a **K**.
* An LLM provider associated with xAI.

The lecture repeatedly emphasizes this distinction because the names are easy to confuse.

---

## Ollama

**Ollama** allows open-source models to be run locally on your computer.

The lecture uses Ollama for experimenting with local models.

One advantage is that the same general API style can be used to communicate with local models as with cloud-hosted models.

---

## OpenRouter

**OpenRouter** acts as a layer between you and multiple model providers.

Instead of maintaining separate integrations with many providers, you can use one account/API key and route requests to different models.

Conceptually:

```text
                 ┌→ OpenAI
                 ├→ Anthropic
Your application → OpenRouter
                 ├→ Gemini
                 ├→ Open-source models
                 └→ Other providers
```

This makes experimentation across different models much more convenient.

---

# 4. Comparing Models

The lecture recommends **Artificial Analysis** as a useful resource for comparing LLMs.

It provides information about things such as:

* Intelligence/capability.
* Speed.
* Cost.
* Benchmark performance.

However, benchmarks should be treated with a **healthy pinch of salt**. They are useful for getting a sense of model performance but aren't the complete picture.

---

# 5. OpenAI-Compatible APIs

One particularly useful concept is that many providers offer an **OpenAI-compatible API**.

This means that instead of learning a completely different API for every provider, you can often use the same general client structure and change the:

* Base URL.
* API key.
* Model name.

The lecture notes that even Ollama provides an OpenAI-compatible endpoint.

Conceptually:

```text
OpenAI-compatible client
        │
        ├── OpenAI cloud
        ├── Other providers
        ├── OpenRouter
        ├── Groq
        └── Ollama / local machine
```

This makes it much easier to experiment with different models.

---

# 6. Today's Experiment

The lab creates a mini **LLM competition**.

The process is:

```text
LLM #1
   ↓
Generate a challenging question
   ↓
Send question to multiple LLMs
   ↓
Collect answers
   ↓
Anonymize competitors
   ↓
LLM judge evaluates answers
   ↓
Rank the models
```

This is a practical demonstration of multiple agentic patterns.

---

# 7. Step 1 — Have an LLM Generate the Question

Instead of manually creating a benchmark question, an LLM is first asked to generate one.

The requested question should be:

* Challenging.
* Nuanced.
* Thought-provoking.
* Not a mathematical puzzle.
* Answerable succinctly.

The lecture gives an example:

> "What is one human virtue that can be optimized too aggressively and why?"

The exact question can differ between runs, which is intentional.

The emphasis is on **experimentation**, rather than everyone obtaining exactly the same result.

---

# 8. Step 2 — Send the Question to Multiple LLMs

The generated question is then sent to multiple models.

The lab experiments with models/providers including:

* OpenAI.
* Anthropic.
* Gemini.
* DeepSeek.
* Groq.
* OpenRouter.
* Ollama/local models.

The goal isn't necessarily to find the objectively "best model."

Instead, the exercise demonstrates how Python can **orchestrate multiple LLM calls**.

---

# 9. Using `zip()` in Python

The lecture introduces a useful Python technique: `zip()`.

Suppose you have:

```python
competitors = [...]
answers = [...]
```

You can iterate through both lists together:

```python
for competitor, answer in zip(competitors, answers):
    ...
```

This lets you process corresponding elements from multiple lists at the same time.

---

# 10. Using `enumerate()`

The lecture also introduces `enumerate()`.

It provides both:

* The index.
* The item.

Conceptually:

```python
for index, answer in enumerate(answers):
    ...
```

This becomes useful when associating each answer with a competitor number.

---

# 11. Anonymizing the Results

The results are anonymized before asking another LLM to judge them.

Instead of showing:

```text
Claude → Answer
Gemini → Answer
DeepSeek → Answer
...
```

the judge sees:

```text
Competitor 1 → Answer
Competitor 2 → Answer
Competitor 3 → Answer
...
```

### Why?

To reduce potential bias.

The judging LLM shouldn't know which provider or model produced each answer.

---

# 12. Step 3 — LLM as a Judge

This is one of the most important concepts in the lab.

Instead of manually deciding which answer is best, another LLM evaluates the responses.

```text
Question
   │
   ├──→ Model A → Answer A ─┐
   ├──→ Model B → Answer B ─┤
   ├──→ Model C → Answer C ─┤
   └──→ Model D → Answer D ─┤
                            ↓
                       LLM Judge
                            ↓
                         Ranking
```

The judge is deliberately a model that wasn't used to generate the answers.

This is an example of the **evaluator-optimizer / LLM-as-a-judge** pattern discussed on Day 2.

---

# 13. Results From the Lecture's Experiment

The lecture's particular run produced approximately this ranking:

1. **Kimi K2.6**
2. **Claude Sonnet 4.6**
3. **Gemini Flash Light**
4. **GPT OSS 20B**
5. **GPT 3.5 Turbo**
6. Other tested models
7. **Llama 3.2 1B**

The exact ranking isn't the important takeaway.

Different models, model versions, prompts, and judges can produce different results.

The lecture specifically notes that the experiment should be treated as an opportunity to **try different options and see what happens**.

---

# 14. Why the Result Is Interesting

The winning Kimi answer argued that:

> Honesty can become harmful when pursued too aggressively.

The lecture considers the answer particularly interesting because it was:

* Unique.
* Nuanced.
* Well explained.

It also raises an important point about evaluation:

**Does the judge actually identify the best answer, or does it simply prefer a particular style or idea?**

This is one reason LLM-based evaluation should itself be treated carefully.

---

# 15. What Agentic Patterns Are Being Used?

The lab combines multiple patterns from Day 2.

### Pattern 1 — Prompt Chaining

An LLM generates the question.

That question then becomes the input to subsequent LLM calls.

```text
LLM
 ↓
Question
 ↓
Other LLMs
```

### Pattern 2 — Parallelization

Multiple models independently answer the same question.

```text
             ┌→ LLM A
Question ────┼→ LLM B
             ├→ LLM C
             └→ LLM D
```

These calls could be executed in parallel even though the demonstration does not necessarily run them concurrently.

### Pattern 3 — Evaluator / LLM-as-a-Judge

A final LLM evaluates the collected answers.

```text
Answers
   ↓
LLM Judge
   ↓
Ranking
```

So the overall workflow is effectively a **hybrid of multiple patterns**.

---

# 16. Commercial Application

This architecture has broad practical applications.

A business system could:

1. Send a request to several LLMs.
2. Obtain multiple candidate answers.
3. Evaluate those answers.
4. Select the strongest response.
5. Return that response to the user.

This can potentially improve:

* Resilience.
* Output quality.
* Diversity of solutions.
* Performance on difficult tasks.

The **LLM-as-a-judge** pattern is particularly common in agentic workflows.

---

# 17. The Bigger Lesson: Orchestration

The most important technical concept of the day is **orchestration**.

An individual LLM call is relatively simple:

```text
Input → LLM → Output
```

Agentic systems begin to emerge when you use code to connect multiple calls:

```text
Input
  ↓
LLM
  ↓
Python logic
  ↓
LLM
  ↓
Python logic
  ↓
Multiple LLMs
  ↓
Python aggregation
  ↓
LLM judge
  ↓
Output
```

The Python code provides the structure around the LLMs.

> **The power comes not only from individual LLMs, but from how we orchestrate them.**

---

# 18. Lab Philosophy

The instructor encourages students **not to simply watch code being typed**.

Instead:

1. Execute the cells.
2. Change them.
3. Experiment.
4. Refine prompts.
5. Try different models.
6. Add new patterns.
7. Make the notebooks your own.

This reflects an important principle of AI engineering:

> **AI engineering is, in large part, experimentation.**

The goal isn't merely to reproduce the instructor's exact output.

---

# 19. Key Takeaways

### LLM Providers

* OpenAI, Anthropic, and Google offer major model families.
* Models generally come in different sizes/cost tiers.
* DeepSeek provides open-source models.
* Groq is an inference provider.
* Grok is an LLM provider.
* Ollama enables local model execution.
* OpenRouter provides a convenient way to access many models.

### APIs

* Many providers offer OpenAI-compatible APIs.
* Changing the endpoint and model can allow the same general client structure to interact with different providers.
* This makes model experimentation much easier.

### Orchestration

* Python can connect multiple LLM calls.
* Outputs from one call can become inputs to another.
* Multiple LLMs can work independently.
* Results can be aggregated and evaluated.

### Agentic Patterns

Today's experiment combines:

* **Prompt chaining**
* **Parallelization**
* **LLM as a judge / evaluator**

### Engineering Mindset

* Experimentation is essential.
* Different models can behave differently.
* Free models can still be excellent learning tools.
* Don't assume benchmarks tell the whole story.
* Try different prompts, models, and architectures.
* Measure and evaluate rather than relying purely on intuition.

---

## References

* [OpenAI](https://openai.com/)
* [Anthropic](https://www.anthropic.com/)
* [Google Gemini](https://deepmind.google/technologies/gemini/)
* [DeepSeek](https://www.deepseek.com/)
* [Groq](https://groq.com/)
* [xAI / Grok](https://x.ai/)
* [Ollama](https://ollama.com/)
* [OpenRouter](https://openrouter.ai/)
* [Artificial Analysis](https://artificialanalysis.ai/)

**Source:** Uploaded Week 1, Day 3 lecture transcript. 
