# Unit 3: HuggingFace Platform and Pipelines

**AI Systems Engineering 1**
CV Raman Global University, Bhubaneswar — AI Center of Excellence

---

## What We'll Cover Today

- What are model weights and why they matter
- PyTorch and GPU basics
- The Transformer architecture (high level)
- HuggingFace ecosystem — Hub, Transformers, Pipelines
- Three pipeline tasks: generation, sentiment, zero-shot

> **Speaker notes:** Frame the session — "Today we go from concepts to code. We'll understand what's inside a model, then use HuggingFace Pipelines to run three AI tasks with just a few lines of Python."

---

## What Is a "Model"?

- A model is a **mathematical function** with millions (or billions) of learned numbers called **parameters**
- Two parts:
  - **Architecture** — the structure (what the model *can* do)
  - **Weights** — the learned numbers (how *well* it does it)

**Analogy:** A recipe (architecture) + specific measurements (weights)

- Same recipe, different measurements = different dish
- Same architecture, different weights = different model

> **Speaker notes:** Make this concrete — "Think of GPT-2 and GPT-4. They share similar architectural ideas, but GPT-4's weights were trained on far more data with far more parameters. The architecture is the blueprint; the weights are what make each model unique."

---

## What Are Weights?

- Weights = the numbers a model **learned during training**
- Training = adjusting millions of numbers until the model gets good at a task
- Example: GPT-2 has **124 million** numbers, each one fine-tuned
- Stored in files (`.bin` or `.safetensors`)

**Analogy:** If the model is a brain, weights are the synaptic connections that encode knowledge

> **Speaker notes:** "When you download a model from HuggingFace, you're downloading these weight files. Someone already spent the compute to train them — you just reuse the result."

---

## Why Weights Matter

- **Different weights = different capabilities**
  - Same architecture, different training data = different model
- **Pre-trained weights** — someone already spent weeks/months and thousands of dollars training
- We download and reuse them — this is **transfer learning**
- Without pre-trained weights, you'd need massive data + massive compute to train from scratch

> **Speaker notes:** "Transfer learning is one of the most important ideas in modern AI. Instead of starting from zero, we stand on the shoulders of giants — or at least on their GPU bills."

> **Visual:** Diagram showing the training pipeline: `Large Dataset + GPU Cluster → Training (weeks) → Weights File → You download in seconds → Ready to use`

---

## Model Sizes

| Model | Parameters | File Size | Can Run Locally? |
|-------|-----------|-----------|-----------------|
| DistilBERT | 66M | ~250 MB | Yes |
| GPT-2 | 124M | ~500 MB | Yes |
| BART-large | 406M | ~1.6 GB | Yes |
| LLaMA-7B | 7B | ~13 GB | Needs good GPU |
| GPT-4 | ~1.8T (est.) | Not public | No |

**The models we use today are small enough to run on a laptop.**

> **Speaker notes:** "Notice the jump — from millions to billions to trillions. Today we stay in the millions range, which means everything runs locally in our notebook. No API keys needed."

---

## PyTorch and CUDA

- **PyTorch** (`torch`) — the framework that runs model computations
- **CUDA** — lets models run on NVIDIA GPUs for faster inference
- **CPU works fine** for small models (like today's) — just slower

In the notebook:
- `device = 0` → use GPU
- `device = -1` → use CPU

> **Speaker notes:** "You'll see this pattern in the notebook. We check for a GPU, and if there isn't one, we fall back to CPU. For the models we're using today, CPU is perfectly fine — inference takes seconds, not minutes."

---

## What Are Transformers?

- The architecture behind **GPT, BERT, BART**, and virtually every modern AI model
- Introduced in 2017 — "Attention Is All You Need"
- **Key idea:** the model looks at **all words in a sentence at once** (not one by one) to understand context

> **Visual:** Contrast diagram — *Previous approach:* words processed left-to-right, one at a time (sequential). *Transformer approach:* all words attend to each other simultaneously (parallel).

> **Speaker notes:** "Before Transformers, models read text like we read — left to right. Transformers read everything at once and figure out which words relate to which. That's the 'attention' mechanism. This is why they're so good at understanding context."

---

## The `transformers` Library

- HuggingFace's Python library to **load and use** transformer models
- **Same code pattern** whether you use BERT, GPT-2, or BART
- Handles:
  - Downloading weights from the Hub
  - Loading the architecture
  - Tokenization
- This is what we `pip install` in the notebook

```
pip install transformers torch
```

> **Speaker notes:** "The beauty of this library is the consistent API. You learn the pattern once, and it works across hundreds of models. That's a huge deal — you don't need to learn a new framework for each model."

---

## HuggingFace = "GitHub for AI"

- **Hub** — 500k+ models, anyone can upload/download
- **Datasets** — pre-processed datasets ready to use
- **Spaces** — host ML demos (remember Gradio from Unit 2?)

Think of it as:
- Hub **stores** the weights
- `transformers` library **loads** them into code

> **Speaker notes:** "Just like GitHub is where developers share code, HuggingFace is where ML practitioners share models. You can browse models by task, sort by downloads, and try them out in your browser before writing any code."

---

## How It All Connects

```
HuggingFace Hub        transformers library       Pipeline API
 (stores weights)  →   (loads model + tokenizer)  →  (easy inference)
```

When you write:
```python
pipeline("sentiment-analysis")
```

It:
1. **Downloads** weights from the Hub
2. **Loads** them using the transformers library
3. **Returns** a ready-to-use function

> **Visual:** Flow diagram with three boxes connected by arrows: `Hub (cloud)` → `transformers (local library)` → `pipeline (your code)`. Annotate each arrow: "downloads", "wraps into simple API".

> **Speaker notes:** "This is the mental model I want you to have. The Hub is the warehouse, the library is the delivery truck, and the pipeline is the product on your desk ready to use."

---

## What Is a Pipeline?

A pipeline wraps **3 steps** into one function call:

1. **Tokenizer** — converts text → numbers (tokens)
2. **Model** — processes tokens using weights → raw output
3. **Post-processor** — converts raw output → human-readable result

> **Visual:** Step-by-step diagram:
>
> `"I love AI"` → Tokenizer → `[101, 1045, 2293, 9932, 102]` → Model → `[0.02, 0.98]` → Post-process → `POSITIVE (98%)`

You'll learn tokenization in detail in **Notebook 2**!

> **Speaker notes:** "Each of these three steps is something you could do manually — and sometimes you will. But the pipeline gives you a fast lane. Think of it as the drive-through window of AI inference."

---

## Pipeline = Abstraction

**Without pipeline** (~15 lines):
```
import tokenizer, import model
tokenize input, run model.forward()
apply softmax, map to labels...
```

**With pipeline** (2 lines):
```python
classifier = pipeline("sentiment-analysis")
classifier("I love AI")
```

**Same result.** Pipelines let you use state-of-the-art models without understanding every internal step.

> **Speaker notes:** "We'll see both approaches eventually in this course. But today, we use the easy button. The point is: you can be productive with AI models from day one, and learn the internals over time."

---

## Pipeline Tasks We'll Use Today

| Task | Pipeline Name | Model |
|------|--------------|-------|
| Text Generation | `text-generation` | GPT-2 |
| Sentiment Analysis | `sentiment-analysis` | DistilBERT |
| Zero-Shot Classification | `zero-shot-classification` | BART |

> **Speaker notes:** "Three tasks, three models, three use cases. Each one demonstrates a different capability of modern AI — creating text, understanding sentiment, and classifying without training data."

---

## Text Generation: How It Works

- **Autoregressive** — predicts one token at a time, feeds it back as input

```
"AI is" → "AI is transforming" → "AI is transforming the" → ...
```

- **Temperature** controls randomness:
  - Low (0.1) = predictable, safe
  - High (1.0) = creative, surprising
- `max_new_tokens` limits output length

> **Speaker notes:** "Temperature is like a creativity dial. Turn it down for factual content, turn it up for brainstorming. In the notebook, we use 0.7 — a nice middle ground."

> **Visual:** A slider labeled "Temperature" with 0.1 on the left ("The capital of France is Paris.") and 1.0 on the right ("The capital of France is a shimmering beacon of croissant diplomacy...").

---

## Sentiment Analysis: Classification

- Input text → model → **probability for each class**
- Confidence score tells you how sure the model is
- Default model: **DistilBERT** fine-tuned on sentiment data (SST-2)

**Works well for:**
- Clear positive/negative statements

**Struggles with:**
- Sarcasm, irony, nuance
- Neutral or mixed sentiment

> **Speaker notes:** "In the notebook, we'll see the model handle clear cases well. But try feeding it something sarcastic like 'Oh great, another meeting' — you'll see the model's confidence drop or get it wrong. That's a good discussion point."

---

## Zero-Shot: The Breakthrough

**Traditional classification:**
- Train on specific labels ("shipping" / "payment" / "quality")
- Can **only** predict those labels

**Zero-shot classification:**
- Give **any labels** at inference time
- Model figures it out — no training needed!

**How?** Based on **Natural Language Inference (NLI)**
- Model asks: "Does this text *imply* this label?"
- Tests each candidate label against the input

> **Speaker notes:** "This is genuinely remarkable. You can reclassify your data into completely new categories without retraining. In the notebook, we use a customer support example — the model identifies 'shipping issue' from labels it was never trained on."

> **Visual:** Two-column comparison. Left: "Traditional" — training data with labeled examples → fixed categories. Right: "Zero-shot" — no training data → any categories you want, defined at runtime.

---

## Key Takeaways

1. **Models = architecture + weights**
   - Pre-trained weights are reusable — that's transfer learning

2. **Pipelines abstract the complexity**
   - Tokenize → Model → Post-process, all in one call

3. **Zero-shot = classify without task-specific training**
   - Provide any labels at inference time

> **Speaker notes:** "These three ideas are the foundation for everything else in this unit. Models are reusable, pipelines make them easy, and zero-shot means you can solve new problems without new training data."

---

## Let's Code!

Time to open the notebook and run all three pipelines.

**What you'll do:**
- Run text generation with GPT-2
- Analyze sentiment with DistilBERT
- Classify text with zero-shot (BART)
- Try your own inputs in the exercise

> **Speaker notes:** "Switch to the notebook now. Walk through each cell, pausing to discuss output. Encourage students to modify the inputs and see how results change."

---

## Common Pipeline Tasks Reference

| Task | Pipeline Name | Example Use Case |
|------|--------------|------------------|
| Text Generation | `text-generation` | Chatbots, content creation |
| Sentiment Analysis | `sentiment-analysis` | Product reviews, social media |
| Zero-Shot Classification | `zero-shot-classification` | Customer support routing |
| Summarization | `summarization` | News articles, documents |
| Question Answering | `question-answering` | FAQ systems, search |
| Translation | `translation` | Multi-language support |
| Named Entity Recognition | `ner` | Information extraction |

> **Speaker notes:** "This is a reference slide — we won't cover all of these today, but know that the same `pipeline()` pattern works for all of them. If you can use one, you can use any."

---

## Resources

- [HuggingFace Hub](https://huggingface.co/models) — Browse 500k+ models
- [Transformers Documentation](https://huggingface.co/docs/transformers) — Full library reference
- [Pipeline Tutorial](https://huggingface.co/docs/transformers/pipeline_tutorial) — Official guide

---
