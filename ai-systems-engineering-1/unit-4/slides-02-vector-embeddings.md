# Unit 4: Vector Embeddings & Semantic Search

**AI Systems Engineering 1**
CV Raman Global University, Bhubaneswar — AI Center of Excellence

---

## What We'll Cover Today

- Recap: where keyword search fails
- What are embeddings?
- Measuring similarity (cosine similarity)
- Embedding models and tools
- Semantic search vs keyword search
- Document chunking
- Vector databases (Chroma)

> **Speaker notes:** "Last class we built a RAG system with keyword search and saw its limitations. Today we learn how embeddings and semantic search fix those limitations. By the end, you'll understand the technology well enough to build a semantic search engine in the notebook."

---

## Recap: Where Keyword Search Fails

Remember the limitation from our RAG notebook?

| Query | Keyword Search Result | Why? |
|---|---|---|
| *"Who is the CEO?"* | Finds company doc | "CEO" appears in the text |
| *"Who is Priya?"* | Finds Priya's doc | "Priya" appears in the text |
| *"Who founded the company?"* | **Misses Priya's doc** | "founded" ≠ "CEO" or "Priya" |

The Priya document says she's the founder and CEO — but keyword search can't connect "founded" to that document.

**We need search that understands *meaning*, not just matches keywords.**

> **Speaker notes:** "Pull up the RAG notebook if possible and show the 'Who founded the company?' failure live. This is the exact problem embeddings solve. The students saw this limitation — now we show them the fix."

---

## What Are Embeddings?

**Core idea:** convert text into a list of numbers that capture its *meaning*.

```
"machine learning"  →  [0.12, -0.45, 0.78, 0.33, -0.91, ...]
                        (384 numbers)

"deep learning"     →  [0.15, -0.42, 0.81, 0.29, -0.88, ...]
                        (similar numbers — similar meaning!)

"cricket match"     →  [-0.67, 0.23, -0.11, 0.54, 0.08, ...]
                        (very different numbers — different meaning)
```

Similar texts get **similar numbers**. Different texts get **different numbers**.

These lists of numbers are called **vectors** — hence "vector embeddings."

> **Speaker notes:** "Don't let the math intimidate students. The key insight is simple: similar meaning → similar numbers. Show the first two vectors and point out how close the individual numbers are. Then contrast with the third — completely different numbers because the meaning is completely different."

---

## Analogy: GPS Coordinates for Words

Just like GPS puts **nearby cities** close in coordinates, embeddings put **similar meanings** close in number-space.

| City Pair | GPS Distance | Meaning Pair | Embedding Distance |
|---|---|---|---|
| Bhubaneswar ↔ Cuttack | ~25 km (close) | "ML" ↔ "Deep Learning" | Small (similar) |
| Bhubaneswar ↔ Puri | ~60 km (medium) | "ML" ↔ "Data Science" | Medium (related) |
| Bhubaneswar ↔ Mumbai | ~1,600 km (far) | "ML" ↔ "Cooking recipes" | Large (unrelated) |

GPS gives every location a **unique coordinate**. Embeddings give every text a **unique vector**.

Nearby coordinates = nearby locations. Nearby vectors = nearby meanings.

> **Speaker notes:** "Use the local geography — students know exactly how close Cuttack is to Bhubaneswar versus how far Mumbai is. That intuition maps directly to how embeddings work. Two texts about AI topics are 'close' like neighboring cities. An AI text and a cooking text are 'far apart' like cities in different states."

---

## What Do Dimensions Mean?

An embedding model like all-MiniLM-L6-v2 produces **384 numbers** per text.

Each number captures a different **aspect of meaning** — but no single number means "this is about sports" or "this is about technology."

**Analogy — Rating a movie:**

| Dimension | Dangal | 3 Idiots | Inception |
|---|---|---|---|
| Story | 9 | 10 | 8 |
| Acting | 9 | 9 | 7 |
| Music | 7 | 8 | 6 |
| Visuals | 6 | 5 | 10 |

With just **4 numbers**, you can tell Dangal and 3 Idiots are more similar to each other than either is to Inception.

Embeddings do the same thing — but with **384 dimensions** capturing every nuance of meaning.

> **Speaker notes:** "The movie analogy makes dimensions concrete. Students can see how 4 numbers let you compare movies. Now imagine 384 numbers — you can capture incredibly fine differences in meaning. No single number is interpretable on its own, but together they form a 'fingerprint' of meaning."

---

## Cosine Similarity — Measuring Closeness

How do we measure how "close" two embeddings are?

**Cosine similarity** measures the **angle** between two vectors:

```
        Doc B (similar to A)
       ↗
      / ) small angle = high similarity
     /  )
    /───)──────────→  Doc A
    \
     \
      \
       ↘
        Doc C (different from A)
         large angle = low similarity
```

| Score | Meaning |
|---|---|
| **1.0** | Identical meaning (same direction) |
| **0.7 - 0.9** | Very similar (closely related topics) |
| **0.3 - 0.6** | Somewhat related |
| **0.0** | Completely unrelated (perpendicular) |

> **Speaker notes:** "Don't get into the math formula — the intuition is what matters. Two arrows pointing in the same direction = similar. Two arrows pointing in different directions = different. The score is between 0 and 1, where 1 means identical. Draw two arrows on the board at different angles to make this visual."

---

## Similarity in Action — TechSolutions Examples

Using our TechSolutions India knowledge base:

| Text A | Text B | Similarity | Why? |
|---|---|---|---|
| "machine learning" | "deep learning" | **0.65** | Both AI/ML topics |
| "machine learning" | "data analysis" | **0.52** | Related tech fields |
| "CEO of the company" | "Priya Sharma founded TechSolutions" | **0.48** | Both about company leadership |
| "machine learning" | "cooking recipes" | **0.12** | Completely different topics |
| "machine learning" | "cricket tournament" | **0.10** | Completely different topics |

**Notice:** "CEO" and "founded TechSolutions" have a meaningful similarity score — embeddings understand the *relationship* even without shared keywords!

> **Speaker notes:** "This is the key insight. Row 3 is the magic — keyword search fails here because there are no common words, but embeddings capture the semantic connection between 'CEO' and 'founded.' This is exactly what will fix our RAG retrieval."

---

## Embedding Models

Different models produce different quality embeddings:

| Model | Dimensions | Speed | Quality | Cost |
|---|---|---|---|---|
| **all-MiniLM-L6-v2** | 384 | Fast | Good | Free (local) |
| **all-mpnet-base-v2** | 768 | Medium | Better | Free (local) |
| **text-embedding-3-small** | 1536 | Fast (API) | High | Paid (OpenAI) |

**Key tradeoffs:**
- More dimensions = more nuance captured, but more storage and computation
- Local models = free and private, but slightly lower quality
- API models = higher quality, but cost money and need internet

We'll use **all-MiniLM-L6-v2** in the notebook — it's free, fast, and good enough for learning.

> **Speaker notes:** "For a student project or prototype, the free local models are perfect. In production, companies often use API models for better quality. The notebook will let students try both — local with SentenceTransformer and API with LiteLLM."

---

## Local vs API Embeddings

| | Local (SentenceTransformer) | API (OpenAI via LiteLLM) |
|---|---|---|
| **Cost** | Free | ~$0.02 per 1M tokens |
| **Privacy** | Data stays on your machine | Data sent to cloud |
| **Internet** | Not required | Required |
| **Speed** | Depends on your hardware | Consistently fast |
| **Model size** | Smaller (80-400MB) | Large (hosted remotely) |
| **Quality** | Good for most tasks | Slightly better |
| **Best for** | Learning, prototypes, private data | Production, high-quality needs |

**In the notebook:** we start with local embeddings (free, no API key needed), then optionally try OpenAI embeddings via LiteLLM to compare.

> **Speaker notes:** "Students should start with local — no API key needed, no cost. Those who have an OpenAI key can try the API version too. In the real world, the choice depends on budget, privacy requirements, and quality needs."

---

## LiteLLM: One API for All Providers

LiteLLM gives you a **single function call** that works across all embedding providers:

```python
from litellm import embedding

# OpenAI
response = embedding(model="text-embedding-3-small",
                     input=["Hello world"])

# Cohere
response = embedding(model="embed-english-v3.0",
                     input=["Hello world"])

# Same function, same format — just change the model name!
```

**Why this matters:** switch providers without rewriting your code.

> **Speaker notes:** "We used LiteLLM for chat completions in the RAG notebook. Same idea here — one API for everything. Students who already have an API key from the previous notebook can use it here too. The notebook shows both the local SentenceTransformer approach and the LiteLLM approach."

---

## Semantic Search vs Keyword Search

This is where everything comes together.

**Query:** *"Who founded the company?"*

```
┌─────────────────────────────────────────────────────────────────────┐
│                      KEYWORD SEARCH                                 │
│                                                                     │
│  Query: "Who founded the company?"                                  │
│           │                                                         │
│           ▼                                                         │
│  Look for words: "founded", "company"                               │
│           │                                                         │
│           ▼                                                         │
│  company_doc: contains "company" ───── ✅ Match (has "company")     │
│  priya_doc: no "founded" keyword ───── ❌ Miss!                     │
│  products_doc: no match ───────────── ❌ Miss                       │
│                                                                     │
│  Result: Returns company doc only — misses the answer!              │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                     SEMANTIC SEARCH                                  │
│                                                                     │
│  Query: "Who founded the company?"                                  │
│           │                                                         │
│           ▼                                                         │
│  Convert to embedding: [0.23, -0.41, 0.67, ...]                    │
│           │                                                         │
│           ▼                                                         │
│  Compare meaning with all document embeddings:                      │
│  priya_doc: "CEO and founder..." ──── ✅ 0.72 (high similarity!)   │
│  company_doc: "TechSolutions..." ──── ✅ 0.58 (related)            │
│  products_doc: "AI platform..." ───── 0.31 (less relevant)         │
│                                                                     │
│  Result: Finds Priya's doc — the correct answer!                    │
└─────────────────────────────────────────────────────────────────────┘
```

> **Speaker notes:** "This is THE 'aha' slide. Walk through both paths slowly. Keyword search looks for exact words and misses. Semantic search understands that 'founded' is related to 'CEO and founder' even without shared keywords. This is the moment students should feel why embeddings matter."

---

## How Semantic Search Works

```
SETUP (one time):
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  Documents   │────>│  Embedding   │────>│    Vector     │
│              │     │  Model       │     │   Database    │
│ doc1, doc2,  │     │              │     │               │
│ doc3, ...    │     │ Convert each │     │ Store all     │
│              │     │ doc to vector│     │ vectors       │
└──────────────┘     └──────────────┘     └──────────────┘

SEARCH (every query):
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│    Query     │────>│  Embedding   │────>│   Cosine     │────>│  Top Match   │
│              │     │  Model       │     │  Similarity   │     │  Documents   │
│ "Who founded │     │              │     │               │     │              │
│  the company │     │ Convert query│     │ Compare with  │     │ Return most  │
│  ?"          │     │ to vector    │     │ all doc vecs  │     │ similar docs │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
```

**Two phases:**
1. **Setup:** embed all documents once and store them
2. **Search:** embed the query, find the closest stored documents

> **Speaker notes:** "Emphasize the two phases. The setup phase is slow but runs only once. The search phase is fast and runs every time a user asks a question. This is why vector databases exist — to make that search phase efficient even with millions of documents."

---

## Document Chunking — Why and How

**Problem:** long documents dilute meaning. A 10-page document about many topics produces one generic embedding that's not great for any specific query.

**Analogy:** Searching a whole textbook vs searching by chapter.

| Approach | Query: "leave policy" |
|---|---|
| **Whole document** | The full company handbook embedding is vaguely about "company stuff" — weak match |
| **Chunked by section** | The "Leave Policy" section embedding is specifically about leave — strong match! |

**Two key parameters:**

| Parameter | What It Controls | Example |
|---|---|---|
| **chunk_size** | How many characters per chunk | 500 characters |
| **overlap** | Shared text between consecutive chunks | 50 characters |

> **Speaker notes:** "Ask students: if you're studying for an exam, do you read the whole textbook at once or focus on specific chapters? Chunking is like breaking a textbook into chapters so you can find the right one quickly. Overlap ensures you don't lose context at the boundaries."

---

## Chunking Visualized

```
Original document (1200 characters):
┌─────────────────────────────────────────────────────────────────┐
│ TechSolutions India was founded in 2019 by Priya Sharma in     │
│ Bhubaneswar. The company specializes in AI-powered business     │
│ solutions. Their flagship product is an intelligent document    │
│ processing platform. The company has grown to 500 employees     │
│ across three offices. They recently launched a new ML toolkit   │
│ for developers. The leave policy allows 24 days annual leave... │
└─────────────────────────────────────────────────────────────────┘

After chunking (chunk_size=500, overlap=50):
┌─────────────────────────────────┐
│         Chunk 1 (500 chars)     │
│ TechSolutions India was founded │
│ in 2019 by Priya Sharma...      │
│ ...intelligent document         │
│                  ┌──────────────┼──────────────────┐
│              overlap (50 chars) │                   │
└──────────────────┼──────────────┘                   │
                   │       Chunk 2 (500 chars)        │
                   │ ...intelligent document          │
                   │ processing platform. The company │
                   │ has grown to 500 employees...    │
                   │                  ┌───────────────┼──────────────┐
                   │              overlap (50 chars)  │              │
                   └──────────────────┼──────────────┘              │
                                      │      Chunk 3 (200 chars)   │
                                      │ ...new ML toolkit for      │
                                      │ developers. The leave      │
                                      │ policy allows 24 days...   │
                                      └────────────────────────────┘
```

The **overlap** ensures no information is lost at chunk boundaries.

> **Speaker notes:** "Draw this on the board step by step. Show how the overlap region appears in both chunks — so if a key sentence spans the boundary, it's captured in at least one chunk. Without overlap, you could split a sentence in half and lose the meaning."

---

## Vector Databases

**What they are:** databases built specifically for storing and searching embeddings.

**Why not a regular database?**

| | Regular Database (SQL) | Vector Database (Chroma) |
|---|---|---|
| **Query type** | Exact match: `WHERE name = 'Priya'` | Similarity: "find docs closest to this vector" |
| **Data stored** | Text, numbers, dates | Vectors (+ metadata) |
| **Search method** | Index lookup | Approximate nearest neighbor |
| **Best for** | Structured data, exact queries | Semantic search, recommendations |

**We'll use Chroma** — lightweight, Python-native, perfect for learning.

> **Speaker notes:** "Students may wonder why we need a special database. The answer is: regular databases search for exact matches. We need to find the *most similar* vectors — that's a fundamentally different operation. Chroma handles this efficiently."

---

## Vector Database: Store & Retrieve

The full pipeline from documents to answers:

```
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌───────────┐
│          │   │          │   │          │   │           │
│ Documents│──>│  Chunk   │──>│  Embed   │──>│  Store in │
│          │   │          │   │          │   │  Chroma   │
└──────────┘   └──────────┘   └──────────┘   └─────┬─────┘
                                                    │
                              (stored once)         │
                ────────────────────────────────────┘
                              (searched many times)
                                                    │
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌─────▼─────┐
│          │   │  Top     │   │  Cosine  │   │           │
│  Query   │──>│  Embed   │──>│ Similarity──>│  Chroma   │
│          │   │  query   │   │  Search  │   │  Search   │
└──────────┘   └──────────┘   └──────────┘   └───────────┘
       │
       ▼
┌──────────┐
│ Return   │
│ top docs │
└──────────┘
```

**In the notebook,** you'll do this with just a few lines of Python using Chroma's built-in embedding support.

> **Speaker notes:** "Walk through the pipeline: documents get chunked, embedded, and stored once. Then every query gets embedded and compared against the stored vectors. The top matches are returned. This is the same RAG pipeline from the last lecture, but with semantic retrieval replacing keyword retrieval."

## Key Takeaways

1. **Embeddings capture meaning as numbers** — similar text → similar vectors
2. **Cosine similarity measures closeness** — score from 0 (unrelated) to 1 (identical)
3. **Semantic search finds by meaning, not keywords** — "founded" matches "CEO and founder"
4. **Chunking splits documents** for more precise retrieval
5. **Vector databases (Chroma)** store and search embeddings efficiently
6. **LiteLLM** unifies API-based embedding calls across providers

> **Speaker notes:** "Go through each point and ask students to explain it in their own words. The most important takeaway is #3 — semantic search is the whole reason we learned embeddings. If students understand that 'embeddings let us search by meaning,' they've got the core concept."

---

## What's Next: The Notebook

In the notebook, you'll build everything we discussed today:

| Section | What You'll Do |
|---|---|
| **Create embeddings** | Use SentenceTransformer to embed TechSolutions documents |
| **Compare similarities** | Calculate cosine similarity scores between texts |
| **Try API embeddings** | Use OpenAI embeddings via LiteLLM (optional) |
| **Build a Chroma knowledge base** | Store document embeddings in a vector database |
| **Semantic search** | Query by meaning and see the right documents returned |

**The big test:** ask *"Who founded the company?"* and watch semantic search find Priya's document — the query that keyword search couldn't handle.

> **Speaker notes:** "Build excitement for the notebook. The moment they see 'Who founded the company?' correctly return Priya's document — after seeing it fail with keyword search — that's the payoff for this entire lecture. Walk through the notebook sections in order."

---

## Resources

| Resource | Link |
|---|---|
| **Sentence Transformers docs** | [sbert.net](https://www.sbert.net/) |
| **Chroma docs** | [docs.trychroma.com](https://docs.trychroma.com/) |
| **Embedding Projector (interactive)** | [projector.tensorflow.org](https://projector.tensorflow.org/) |
| **LiteLLM embedding docs** | [docs.litellm.ai/docs/embedding](https://docs.litellm.ai/docs/embedding/supported_embedding) |
| **3Blue1Brown: Vectors (video)** | [youtube.com/3blue1brown](https://www.youtube.com/watch?v=fNk_zzaMoSs) |

> **Speaker notes:** "The Embedding Projector is amazing for a live demo — students can type words and see them clustered in 3D space. The 3Blue1Brown video is helpful for students who want to understand vectors more intuitively. Assign the Sentence Transformers docs as reading for students who want to go deeper."

---

## Let's Code!

Time to open the notebook and build your own semantic search engine.

**What you'll do:**
- Embed documents with SentenceTransformer (Section 1-2)
- Calculate and interpret similarity scores (Section 3)
- Build a Chroma vector database (Section 4)
- Run semantic search queries (Section 5)
- Compare with the keyword search from the RAG notebook (Section 6)

> **Speaker notes:** "Switch to the notebook now. Start with the embedding creation — students need to see that calling the model is just 2-3 lines of code. Then the similarity comparison makes embeddings tangible. The Chroma section ties it all together into a working search engine."

---
