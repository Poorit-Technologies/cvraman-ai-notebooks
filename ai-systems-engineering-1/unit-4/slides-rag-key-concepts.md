# Unit 4: RAG — Key Concepts

**AI Systems Engineering 1**
CV Raman Global University, Bhubaneswar — AI Center of Excellence

---

## What is RAG?

**RAG = Retrieval-Augmented Generation** — give an LLM real documents so it answers with facts, not guesses.

```
Documents → Embed → Store → Query → Retrieve → LLM → Answer
```

### Why do we need it?

| Problem | Without RAG | With RAG |
|---------|-------------|----------|
| **Hallucination** | LLM invents plausible-sounding facts | LLM quotes real documents |
| **Knowledge cutoff** | LLM doesn't know anything after its training date | LLM reads up-to-date docs at query time |
| **Domain data** | LLM has no access to your private/company data | LLM answers from your own documents |

> **Speaker notes:** "Start with the problem. Ask students: 'If you ask ChatGPT about your company's leave policy, what happens?' It makes something up. RAG fixes this by giving the LLM your actual documents as context before it answers."

---

## Embeddings — Meaning as Numbers

An **embedding model** converts text into a vector of numbers that captures its meaning.

```
"machine learning"  →  [0.12, -0.45, 0.78, ...]   (384 numbers)
"deep learning"     →  [0.15, -0.42, 0.81, ...]   (similar — similar meaning!)
"cooking recipes"   →  [-0.67, 0.23, -0.11, ...]   (different — different meaning)
```

**Key idea:** similar meaning → similar vectors.

| Embedding Model | Dimensions | Use Case |
|-----------------|-----------|----------|
| all-MiniLM-L6-v2 | 384 | Lightweight, runs locally |
| OpenAI text-embedding-3-small | 1536 | High quality, API-based |

> **Speaker notes:** "Think of embeddings as coordinates in a 'meaning space'. The word 'king' is near 'queen' and far from 'bicycle'. This is what lets us search by meaning instead of keywords."

---

## Similarity Search

**Cosine similarity** measures the angle between two vectors — how close in meaning they are.

| Scale | Meaning |
|-------|---------|
| **1.0** | Identical meaning |
| **0.5+** | Semantically related |
| **~0** | Unrelated |

| Text Pair | Score | Why |
|-----------|-------|-----|
| "ML" ↔ "Deep Learning" | **0.65** | Very related concepts |
| "CEO" ↔ "founded TechSolutions" | **0.48** | Semantically connected — no shared keywords! |
| "ML" ↔ "cooking recipes" | **0.12** | Unrelated topics |

### Keyword Search vs Semantic Search

| Query | Keyword Search | Semantic Search |
|-------|---------------|-----------------|
| *"Who is the CEO?"* | Finds doc with "CEO" | Finds doc with "CEO" |
| *"Who founded the company?"* | **Misses** — no "founded" keyword | **Finds** "CEO and co-founder" doc |

> **Speaker notes:** "Row 2 in the similarity table is the key insight: 'CEO' and 'founded TechSolutions' share zero keywords, but embeddings know they're connected. This is what makes RAG retrieval work."

---

## Chunking — Splitting Documents for Precision

**Problem:** one long document produces a vague embedding that doesn't match specific queries well.

**Solution:** split into smaller **chunks**, each with a focused embedding.

```
Full company handbook (1 embedding — vague)
        ↓ chunk
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ Company Info  │  │   Products   │  │ Leave Policy  │
│ (1 embedding) │  │ (1 embedding) │  │ (1 embedding) │
└──────────────┘  └──────────────┘  └──────────────┘
        ↑ precise match for "Who founded the company?"
```

**Two parameters:**

| Parameter | What it controls |
|-----------|-----------------|
| `chunk_size` | Number of characters per chunk |
| `overlap` | Shared text between consecutive chunks |

**Overlap** ensures no context is lost at chunk boundaries — the end of one chunk repeats at the start of the next.

> **Speaker notes:** "Analogy: searching a whole textbook vs searching by chapter. Chunking gives each topic its own embedding, so the retriever finds the right section — not the whole book."

---

## Vector Database

A **vector database** stores embeddings and finds the most similar ones efficiently.

### SQL vs Vector DB

| Regular Database | Vector Database |
|-----------------|-----------------|
| Exact match: `WHERE name = 'Priya'` | Similarity: "find closest vectors" |
| Structured queries (SQL) | Semantic queries (natural language) |

### Two Phases

```
SETUP (once):
  Documents → Chunk → Embed → Store in Chroma

SEARCH (every query):
  Query → Embed → Compare with stored vectors → Return top matches
```

**Chroma** — lightweight, Python-native vector database. Handles the hard part: efficiently comparing a query vector against thousands of stored vectors without scanning one by one.

> **Speaker notes:** "Without a vector database, you'd compare against every document manually — too slow at scale. Chroma indexes the vectors so similarity search is fast, even with thousands of documents."

---

## The Complete RAG Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│                       RAG Pipeline                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  [Documents] → [Chunk] → [Embed] → [Vector DB]             │
│                                        ↓                    │
│  [User Query] → [Embed Query] → [Similarity Search]        │
│                                        ↓                    │
│  [Retrieved Context] + [Query] → [LLM] → [Answer]          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Three Components, Three Roles

| Component | Role | Tool |
|-----------|------|------|
| **Retriever** | Embed query → search Chroma → return top docs | Chroma + SentenceTransformer |
| **Augmenter** | Inject retrieved docs into the system prompt | System prompt with `{context}` |
| **Generator** | LLM reads context + query → produces grounded answer | GPT-4o-mini (temperature = 0) |

**Why temperature = 0?** We want factual, consistent answers grounded in documents — not creative guessing.

**Why "say I don't know" in the prompt?** Prevents hallucination when the retrieved context doesn't cover the question.

> **Speaker notes:** "Each component maps to a function in the code: the retriever is `retrieve()`, the augmenter is the system prompt template, the generator is `generate_answer()`. The pipeline is modular — you can swap any component independently."

---

## Key Takeaways

| Concept | One-liner |
|---------|-----------|
| **Embeddings** | Text → vectors that capture meaning |
| **Cosine similarity** | Measures how close two vectors are (0–1) |
| **Chunking** | Split long docs into focused pieces for precise retrieval |
| **Vector DB (Chroma)** | Store and search embeddings efficiently |
| **Retriever** | Embed query → find matching docs |
| **Augmenter** | Inject docs into the LLM prompt |
| **Generator** | LLM produces answer grounded in context |

### What Can Go Wrong?

| Problem | Cause | Fix |
|---------|-------|-----|
| Wrong answer | Retriever found wrong docs | Improve chunking, embeddings, or add metadata filters |
| Vague answer | Too many or too few docs retrieved | Tune `n_results` parameter |
| Hallucination | Context doesn't cover the question | Strengthen "I don't know" prompt instructions |

> **Speaker notes:** "If students remember one thing: RAG lets an LLM answer with facts from your documents instead of guessing. Everything we covered — embeddings, chunking, vector databases — exists to make that retrieval accurate."

---
