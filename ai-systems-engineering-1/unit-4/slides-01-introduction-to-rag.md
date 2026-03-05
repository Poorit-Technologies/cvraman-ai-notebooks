# Unit 4: Introduction to RAG

**AI Systems Engineering 1**
CV Raman Global University, Bhubaneswar — AI Center of Excellence

---

## What We'll Cover Today

- Why LLMs need help
- What RAG is (with analogies)
- How the RAG pipeline works
- Temperature and LLM parameters
- What we'll build in the notebook

> **Speaker notes:** "Today we learn why even powerful LLMs aren't enough on their own, and how a technique called RAG fixes their biggest weakness. By the end, you'll understand the concept well enough to build one yourself in the notebook."

---

## LLMs Are Impressive, But...

LLMs can write essays, translate languages, generate code — they're incredibly capable.

**But they have blind spots:**

| What LLMs Don't Know | Example |
|---|---|
| **Your company's internal data** | Employee details, policies, pricing |
| **Recent events** | News after the training cutoff |
| **Private documents** | Internal reports, meeting notes |
| **Domain-specific knowledge** | Niche industry data, local context |

> **Speaker notes:** "Ask students: If you asked ChatGPT about our university's exam schedule, would it know? What about a company's leave policy? These are real limitations."

---

## The Hallucination Problem

When LLMs don't know the answer, they don't say "I don't know."

They **guess — confidently.**

This is called **hallucination**:

```
Q: "Who is the CEO of TechSolutions India?"

A: "The CEO of TechSolutions India is Rajesh Kumar.
    He has over 20 years of experience in the IT industry
    and holds an MBA from IIM Ahmedabad."
```

*Every detail above is fabricated — but it sounds completely believable.*

> **Speaker notes:** "We'll see this live in the notebook — GPT will either make up a fake CEO with a convincing backstory or admit it doesn't know. Either way, it can't give the right answer."

---

## The Fix — Give It the Right Information

What if we could give the LLM a **cheat sheet** before it answers?

**Without context:**
```
Q: "Who is the CEO of TechSolutions India?"
A: ❌ Makes up an answer (hallucination)
```

**With context:**
```
Context: "TechSolutions India... CEO: Priya Sharma..."
Q: "Who is the CEO of TechSolutions India?"
A: ✅ "The CEO is Priya Sharma" (correct!)
```

Provide the right documents → get the right answer.

> **Speaker notes:** "This is the 'aha' moment. The notebook demo shows the same question with and without context, side by side. The difference is dramatic."

---

## Analogy 1 — Open-Book vs Closed-Book Exam

| | Closed-Book (LLM Alone) | Open-Book (LLM + RAG) |
|---|---|---|
| **Knowledge source** | Only what's memorized (training data) | Can look up reference materials |
| **Accuracy** | May guess or hallucinate | Answers grounded in real documents |
| **Current info** | Frozen at training cutoff | Can access up-to-date information |
| **Domain knowledge** | General knowledge only | Can use specialized documents |

> **Speaker notes:** "Ask students which exam they'd prefer for a subject they haven't studied. That's exactly the choice we're making for the LLM — and we're choosing open-book."

---

## Analogy 2 — The New Employee

Imagine a **smart new employee** on their first day at work.

Someone asks: *"What's our company's leave policy?"*

The employee doesn't know yet, but they're smart. So they:

1. **Search** the company wiki for "leave policy"
2. **Read** the relevant page
3. **Answer** the question using what they found

**That's exactly what RAG does — but with an LLM instead of an employee.**

> **Speaker notes:** "This is the most relatable analogy. Walk through each step slowly. The employee is the LLM. The company wiki is the knowledge base. The search is the retrieval step."

---

## Analogy 3 — The Librarian

You walk into a library and ask a question.

The librarian doesn't have every book memorized. Instead:

1. **Finds** the right books on the shelf
2. **Reads** the relevant sections
3. **Gives** you the answer based on what she read

The librarian is **smart** (like an LLM) but still needs to **look things up** (like RAG).

> **Speaker notes:** "Three analogies give different students different entry points into the concept. Some relate to exams, some to work, some to libraries. All describe the same pattern: search, read, answer."

---

## So What is RAG?

**R**etrieval — find relevant documents

**A**ugmented — add them to the prompt

**G**eneration — LLM generates answer using context

### One-Line Definition:

> A technique that gives LLMs access to external knowledge by finding relevant information and injecting it into the prompt.

> **Speaker notes:** "Have students repeat the acronym. R-A-G. Retrieval, Augmentation, Generation. Each word maps to a concrete step we'll build in the notebook."

---

## The RAG Pipeline

```
┌──────────────┐     ┌──────────────┐     ┌──────────────────┐     ┌──────────────┐
│   QUESTION   │────>│  RETRIEVAL   │────>│   AUGMENTATION   │────>│  GENERATION  │
│              │     │              │     │                  │     │              │
│ User asks a  │     │ Search the   │     │ Add retrieved    │     │ LLM generates│
│ question     │     │ knowledge    │     │ docs to the      │     │ answer using │
│              │     │ base         │     │ prompt as context│     │ the context  │
└──────────────┘     └──────────────┘     └──────────────────┘     └──────────────┘
```

Each step is distinct. Each step can be improved independently.

> **Speaker notes:** "Draw this on the board. Have students identify which step each analogy maps to. The new employee searching the wiki = Retrieval. Reading the page = Augmentation. Answering = Generation."

---

## RAG Pipeline — Step by Step

**Step 1:** User asks *"Who is the CEO?"*

**Step 2:** System searches the knowledge base → finds the company document

**Step 3:** Company document is injected into the prompt as context

**Step 4:** LLM reads the context + question → generates an accurate answer

```
User: "Who is the CEO?"
          │
          ▼
   ┌─────────────┐
   │  RETRIEVAL   │  → Searches knowledge base
   │  Found:      │     "TechSolutions India...
   │  company doc │      CEO: Priya Sharma..."
   └──────┬──────┘
          ▼
   ┌─────────────┐
   │ AUGMENTATION │  → Adds doc to system prompt
   └──────┬──────┘
          ▼
   ┌─────────────┐
   │ GENERATION   │  → "The CEO is Priya Sharma."
   └─────────────┘
```

> **Speaker notes:** "Walk through with the TechSolutions example from the notebook. This exact flow is what `answer_question_verbose()` makes visible in the code."

---

## Why RAG?

| Problem | RAG Solution |
|---------|-------------|
| **Knowledge cutoff** | Provide up-to-date information |
| **Hallucinations** | Ground answers in real documents |
| **Domain expertise** | Add company-specific knowledge |
| **Cost** | Cheaper than fine-tuning the model |
| **Transparency** | Can cite sources for answers |

> **Speaker notes:** "Emphasize cost — fine-tuning is expensive and requires ML expertise. RAG just needs good documents. Any company can build a RAG system, but few can fine-tune their own model."

---

## Understanding Temperature

Temperature controls how **random** or **deterministic** the LLM's output is.

```
Temperature Scale:

 0.0          0.7          1.0          1.5          2.0
  |------------|------------|------------|------------|
  Deterministic  Balanced     Creative     Wild      Chaotic
  (factual)    (default)   (storytelling)           (nonsensical)
```

| Temperature | Behavior | Best For |
|---|---|---|
| **0.0** | Always picks the most likely word. Same input = same output. | Factual Q&A, RAG, data extraction |
| **0.3 - 0.5** | Mostly deterministic with slight variation | Summarization, translation |
| **0.7** | Balanced creativity (OpenAI default) | General conversation, writing |
| **1.0** | More diverse and creative outputs | Brainstorming, creative writing |
| **> 1.0** | Increasingly random and unpredictable | Rarely useful in practice |

> **Speaker notes:** "We've used temperature in every unit but never fully explained it. This fills that gap. Think of it as a creativity dial."

---

## Temperature in Action

Same prompt, different temperatures:

**Prompt:** *"Suggest a tagline for a tech company in Bhubaneswar."*

**temp = 0 (deterministic):**
```
Attempt 1: "Innovating Tomorrow from the Temple City"
Attempt 2: "Innovating Tomorrow from the Temple City"   ← same!
```

**temp = 0.7 (balanced):**
```
Attempt 1: "Innovating Tomorrow from the Temple City"
Attempt 2: "Where Heritage Meets Innovation"            ← slight variation
```

**temp = 1.0 (creative):**
```
Attempt 1: "Code Meets Culture in the Heart of Odisha"
Attempt 2: "Silicon Shores of the Eastern Coast"        ← very different
```

> **Speaker notes:** "The notebook has a live demo — students will run this and see the difference themselves. Have them notice: temp=0 is identical every time."

---

## Why RAG Uses Temperature = 0

RAG wants **factual, consistent** answers — not creative improvisation.

When you've retrieved the right document, you want the LLM to:
- **Faithfully report** what's in the document
- **Not add** creative embellishments
- Give the **same answer** every time for the same question

> temp = 0 means: "Just read the context and answer accurately."

> **Speaker notes:** "This connects the temperature concept directly to RAG. Creative writing needs high temperature. Factual Q&A needs zero. Students will see this in every RAG call in the notebook."

---

## What We'll Build Today

In the notebook, you'll build a complete RAG system:

1. **Knowledge base** — documents about TechSolutions India
2. **Keyword retrieval** — search function to find relevant docs
3. **RAG pipeline** — retrieval + context injection + LLM generation
4. **Gradio chat interface** — interactive chatbot
5. **Pipeline visibility tool** — see each RAG step as it runs

> **Speaker notes:** "Preview the notebook sections. If you've run it beforehand, show the Gradio demo quickly. The students will build this themselves step by step."

---

## Limitations Preview

Our retrieval today is **intentionally simple** — keyword matching.

This means:

| Question | What Happens |
|----------|-------------|
| *"Who is Priya?"* | Matches "priya" document |
| *"Who is the CEO?"* | Matches "company" doc (has "CEO" keyword) |
| *"Who founded the company?"* | May **miss** the "priya" doc entirely |
| *"What's the leave policy?"* | Matches "policies" doc |

**"CEO" won't match the "priya" document**, even though Priya is the CEO.

**Next notebook:** Vector embeddings and semantic search solve this — finding documents by **meaning**, not just keywords.

> **Speaker notes:** "Set expectations — today's retrieval is intentionally simple so we can focus on the RAG concept. The 'aha' of seeing its limitations motivates the next notebook on embeddings."

---

## Visual Demos & Resources

Explore these to deepen your understanding:

| Resource | Link |
|---|---|
| **IBM: What is RAG? (video)** | [ibm.com/think/videos/rag](https://www.ibm.com/think/videos/rag) |
| **RAG Playground (interactive)** | [ragplay.vercel.app](https://ragplay.vercel.app/) |
| **RAG Pipeline Diagrams** | [designveloper.com/.../rag-pipeline-diagram](https://www.designveloper.com/blog/rag-pipeline-diagram/) |
| **DeepLearning.AI RAG Course** | [learn.deeplearning.ai/.../rag](https://learn.deeplearning.ai/courses/retrieval-augmented-generation/) |
| **RAG Architecture Guide** | [clickittech.com/.../rag-architecture-diagram](https://www.clickittech.com/ai/rag-architecture-diagram/) |

> **Speaker notes:** "If time permits, do a live demo of the RAG Playground in class — students can see chunking, embeddings, and retrieval happening in real time. The IBM video is also great as a pre-class assignment."

---

## Key Takeaways

1. **LLMs hallucinate without context** — RAG fixes this by providing real documents
2. **RAG = Retrieval + Augmentation + Generation** — find, inject, generate
3. **Temperature = 0 for factual answers** — deterministic, consistent output
4. **Simple keyword retrieval works** — but has clear limitations
5. **Next: embeddings and semantic search** — finding documents by meaning

> **Speaker notes:** "Recap each point. Ask students to explain RAG in their own words using one of the three analogies. If they can explain it to a friend, they've got it."

---

## Let's Code!

Time to open the notebook and build your own RAG system.

**What you'll do:**
- See an LLM hallucinate (Section 2)
- Compare temperature settings live (Section 4)
- Build a knowledge base and retrieval function (Sections 5-6)
- Create a RAG-powered chatbot with Gradio (Sections 7-8)
- Watch the pipeline run step by step (Section 9)

> **Speaker notes:** "Switch to the notebook now. Walk through Section 2 demo first — seeing the LLM fail, then succeed with context, is the 'aha' moment that motivates everything else."

---
