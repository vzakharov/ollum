# O(LLM) — aka **ollum**

> *We wielded. We ruled. We caused.*
> — Old Norse, probably about recursive prompt evolution.

O(LLM) is a Python framework for evolving ideas, code, and content using large language models as both creators *and* critics.

It's built on a simple idea:
LLMs can generate, mutate, compare, and recombine outputs — so why not let them *evolve* better ones over time?

## ✨ What It Does

O(LLM) orchestrates a feedback loop where LLMs:

* **Seed** content (ideas, pitches, poems, code, you name it)
* **Evaluate** it pairwise using **Elo-style battles**
* **Mutate** top candidates slightly
* **Breed** strong candidates into hybrid outputs

All wrapped in an async-aware engine with budget control, model selection, and plug-and-play evolution parameters.

## 🧬 Example Use Cases

* Auto-generate marketing copy that *iteratively* improves
* Evolve satirical tweets until they're actually funny
* Refine AI-generated poetry or short stories
* Mix-and-match code snippets for optimization or obfuscation
* Prompt engineering: evolve the prompts *themselves*

## 🛠️ Installation

```bash
pip install ollum
```

## 🧪 Quickstart

```python
import asyncio
from ollum import Ollum
from ollum.generators import openai_generator, claude_generator
from transformers import pipeline

# Define a custom async generator using transformers
async def local_generator(prompt, **kwargs):
    # Use asyncio.to_thread to make a synchronous operation non-blocking
    return await asyncio.to_thread(
        lambda: pipeline(
            "text-generation",
            model=kwargs.get("model", "mistralai/Mistral-7B-Instruct-v0.2"),
            device=kwargs.get("device", "cpu")
        )(
            prompt, 
            max_length=kwargs.get("max_length", 500)
        )[0]["generated_text"]
    )

# Initialize ollum with these generators
ollum = Ollum(
    openai_generator,   # base generator if none specified elsewhere
    model="gpt-4o",     # default generation args for all steps
    temperature=0.7,
    seeding={
        'generators': [
            (openai_generator, {'model': 'gpt-4o', 'temperature': 1.2}),
            (claude_generator, {'model': 'claude-3-opus', 'temperature': 1.0})
        ]
    },
    evaluation={
        'prompt': "Choose the text that best balances absurdity and clarity."
    },
    mutation={
        'generator': (local_generator, {'temperature': 0.7})
    },
    crossover={
        'temperature': 0.9,
        'top_p': 0.85
    },
    budget_usd_per_hour=0.10
)

prompt = ollum.generate("Write a sarcastic product pitch about toothpaste for vampires")
# Or use async, where supported:
# prompt = await ollum.agenerate("Write a sarcastic product pitch about toothpaste for vampires")

print(prompt)
```

## 🤖 Customizable Components

### Shared Base Args

Any `**kwargs` passed to `Ollum(...)` are used across all steps (seeding, mutation, crossover, evaluation) unless specifically overridden by that step's config.

This provides a convenient way to set a common model, API key, decoding parameters, etc., without duplicating.

### Step Config

Each step (`seeding`, `evaluation`, `mutation`, `crossover`) can be configured with either:

* `'generator'`: a tuple of `(async_function, kwargs)`
* `'generators'`: a list of such tuples to be picked at random
* Plain `kwargs`: interpreted as parameters to the evaluation logic for that step or, if none match, kwargs to the default generator

### Async-First Design

O(LLM) is designed to be async-first. All built-in generators (like `openai_generator` and `claude_generator`) are async functions, and any custom generators you define should also be async or be wrapped with `asyncio.to_thread` as shown in the example above.

While the package provides both `generate()` and `agenerate()` methods, named this way as per convention, the synchronous version is just a convenience wrapper that runs the async function in an event loop. For best performance, especially when running multiple generations, use the async interface.

## ⚙️ Key Features

### 🔁 Async Evolution

Every seeding, evaluation, and mutation is scheduled in a non-blocking loop with configurable throttling and budget awareness.

### 🎯 Elo-Based Selection

Outputs don't need to be scored absolutely. LLMs just pick the better of two in head-to-head matchups, and winners rise.

### 🧪 Mutation + 🧬 Crossover

Let the system:

* Rewrite candidates with slight variation
* Merge promising ones into hybrids using prompt-based crossover

### 💸 Budget-Aware

Give it a max dollars/hour rate and it'll throttle requests to keep your wallet intact.

### 🤖 Model-Selective

Use different models for different tasks. Fast cheap ones for mutation, slow smart ones for seeding. Entirely up to you.

## 🧠 Philosophy

O(LLM) isn't about infinite generations or massive populations. LLMs are already good — we're here to **refine**, not reinvent.

Think of it as:

> *Evolution meets editorial workflow, but the editor is also the author and the intern.*

## 🛣 Roadmap

*

## 📜 License

MIT. Go evolve responsibly.