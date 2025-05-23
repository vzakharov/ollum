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
from ollum import Ollum
from ollum.generators import openai

ollum = Ollum(
    openai,
    model="gpt-4o",
    temperature=0.7
)

# Or even just:
# ollum = Ollum() # uses `openai, model="gpt-4o", temperature=0.7` by default

prompt = ollum.generate("Write a sarcastic product pitch about toothpaste for vampires")
print(prompt)

# Async version is also available:
# prompt = await ollum.agenerate("Write a sarcastic product pitch about toothpaste for vampires")
```

## 🤖 Customizable Components

### Step Config

Each step (`seeding`, `evaluation`, `mutation`, `crossover`) can be configured with either:

* `'generator'`: a tuple of `(async_function, kwargs)` or just `async_function` by itself
* `'generators'`: a list of such tuples/functions to be picked at random
* Plain `kwargs`: interpreted as parameters to the evaluation logic for that step or, if none match, kwargs to the default generator

```python
from ollum import Ollum
from ollum.generators import openai, anthropic

# Configure specific steps with different generators or parameters
ollum = Ollum(
    openai,
    model="gpt-4o",
    seeding={
        'generators': [
            openai,
            (openai, {'model': 'gpt-4o', 'temperature': 1.2}),
            (anthropic, {'model': 'claude-3-opus', 'temperature': 1.0})
        ]
    },
    evaluation={
        'prompt': "Choose the text that best balances absurdity and clarity."
        # Customizes the selection logic via a different prompt
    },
    mutation={
        'temperature': 0.9  # Uses default generator (openai) with increased temperature
    },
    budget_usd_per_hour=0.10
)
```

Any parameters not customized for a step will be taken from the default generator defined above step configs.

### Custom Generators

You can create your own generators for any step. All generators must be async or wrapped with a decorator to make them async:

```python
from ollum import Ollum
from ollum.generators import openai
from ollum.utils import run_in_thread
from transformers import pipeline

# Use the decorator to create an async generator
@run_in_thread
def local_generator(prompt: str):
    return pipeline(
        "text-generation",
        model="mistralai/Mistral-7B-Instruct-v0.2",
        device="cpu"
    )(
        prompt, 
        max_length=500
    )[0]["generated_text"]

# Use your custom generator in ollum
ollum = Ollum(
    openai,
    mutation={
        'generator': local_generator
    }
)
```

### Async-First Design

O(LLM) is designed to be async-first. All built-in generators (e.g. `openai` and `anthropic`, as imported from `ollum.generators`) are async functions, and any custom generators you define should also be async or be wrapped with the provided `run_in_thread` decorator as shown in the example above.

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

* **More Built-in Generators**: Support for additional LLM providers (Gemini, Cohere, etc.)
* **Colab Notebook**: Interactive examples and templates for quick experimentation
* **MCP Server**: Managed computation environment for distributed evolution
* **Configurable Evaluators**: 
  * Code evaluation through sandbox execution and test running
  * Image evaluation with dedicated vision models
  * User/crowd-facing interfaces for human-in-the-loop comparisons
* **Population Analytics**: Track and visualize evolution progress and population diversity
* **Sub-problem Decomposition**: Break complex problems into parts that evolve separately
* Have an idea? [Open an issue](https://github.com/vzakharov/ollum/issues) or [submit a PR](https://github.com/vzakharov/ollum/pulls).

## 📜 License

MIT. Go evolve responsibly.

## The Name

If you’ve made it this far, congrats — either you're *really* curious or just catastrophically distracted. Either way, let’s finally address the name-shaped elephant in the repo.

* **O(LLM)** is a cheeky nod to [Big O notation](https://en.wikipedia.org/wiki/Big_O_notation). Think of it like saying, "This algorithm? Oh, it's wrapped in LLMs all the way down — complexity optional, recursion mandatory."
* **ollum** (say it with your whole chest: *aw-luhm*) is how we imagined you'd pronounce and import it, because "oh-ell-ell-em" sounds like someone slowly falling down a flight of stairs.
* Coincidentally (or fatefully?), *ollum* is [Old Norse for "we wielded, we ruled, we caused."](https://en.wiktionary.org/wiki/ollum) We found that out after naming it, but we're absolutely claiming it like it was foretold in the sagas.
* And no, it has absolutely *nothing* to do with Tolkien's [emotionally unstable jewelry enthusiast](https://en.wikipedia.org/wiki/Gollum). Unless you want it to. We're not your dad.

So there you have it — half algorithm, half accidental Viking prophecy, and just coherent enough to pip install without shame. Just another day at [Glitchporn](https://glitchporn.substack.com).

Go forth and mutate, rebels. 🧬🔥

~ Vova & Finn