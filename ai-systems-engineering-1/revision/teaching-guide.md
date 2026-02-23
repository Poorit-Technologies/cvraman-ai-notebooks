# Teaching Guide: Student Questions & Verbal Talking Points

**AI Systems Engineering 1 — Revision Session**
CV Raman Global University, Bhubaneswar

---

## How to Use This Guide

This document lists every foundational question students asked during the original classes. It's organized by **teaching order** — explain Layer 1 before opening the notebook, Layer 2 at the start of Act 1, and Layer 3 as the relevant code appears.

Each question is marked:
- **[VERBAL]** — Explain this aloud; the notebook doesn't cover it
- **[NOTEBOOK]** — The notebook demonstrates this; point at the code and walk through it
- **[NOTEBOOK + VERBAL]** — The notebook shows it, but add verbal context for the "why"

---

## Layer 1: Python & Environment Foundations

> **When:** Explain BEFORE opening the notebook (first 10 minutes)
> **Why:** These are prerequisites — if students don't get these, the code won't make sense.

---

### Q: What is `pip install`?
**[VERBAL]**

It's like downloading an app on your phone. `pip install openai` downloads the OpenAI library so your Python code can use it. The notebook runs `!pip install` at the top — that's just pip inside Jupyter.

---

### Q: What are libraries/packages?
**[VERBAL]**

Pre-written code that someone else made so you don't have to. Instead of writing 500 lines to talk to OpenAI's servers, you `import openai` and use their code.

---

### Q: What does each library do? (`openai`, `requests`, `beautifulsoup4`, `tiktoken`, `gradio`)
**[VERBAL]**

| Library | One-line explanation |
|---------|---------------------|
| `openai` | Talks to OpenAI's API (sends prompts, gets responses) |
| `requests` | Sends HTTP requests (fetches web pages, calls APIs) |
| `beautifulsoup4` | Parses HTML — extracts text from web pages |
| `tiktoken` | Counts tokens — shows how the model sees your text |
| `gradio` | Builds web UIs from Python functions (textboxes, buttons, chat) |

---

### Q: What is `getpass` vs `input`?
**[VERBAL]**

Both take input from the user. `input()` shows what you type. `getpass()` hides the text — use it for secrets like API keys so they don't appear on screen. The notebook uses `getpass` in the setup cell.

---

### Q: What is `os.environ`?
**[VERBAL]**

Think of it as a shared noticeboard for your program. You pin a note: `os.environ['OPENAI_API_KEY'] = "sk-..."`. Later, any code (including the OpenAI library) can read that note. It's how we store values so other parts of the code can find them.

---

### Q: What is `f"..."`? (f-strings)
**[VERBAL]**

A way to put variables inside strings. Instead of `"Hello " + name`, you write `f"Hello {name}"`. The `f` before the quotes means "format this" — Python replaces `{name}` with the actual value. You'll see this throughout the notebook.

---

### Q: What is a `class`, `__init__`, `self`?
**[VERBAL]**

A class is a blueprint for creating objects. Think of it like a recipe card:
- `class Conversation:` — the recipe name
- `__init__` — the setup step (what ingredients you start with)
- `self` — refers to "this particular dish I'm making right now"

When you write `client = OpenAI()`, you're creating an object from the `OpenAI` class. The `client` variable holds that object, and you call methods on it like `client.chat.completions.create()`.

---

### Q: What are Python generators / `yield` vs `return`?
**[NOTEBOOK + VERBAL]**

**Say first:** `return` gives you everything at once, like getting match highlights after the game. `yield` gives you results one at a time, like live ball-by-ball commentary.

**Then point to:** The cricket commentary demo in Act 4 (cells with `get_commentary_return` and `get_commentary_yield`). Run both and compare.

**Key point:** A function with `yield` becomes a generator — you loop over it with `for`, and each iteration gets the next yielded value.

---

### Q: What are callbacks?
**[VERBAL]**

Passing a function as an argument to another function. Instead of calling the function yourself, you hand it to someone else and say "call this when you're ready."

In Gradio: `gr.Interface(fn=shout, ...)` — you pass the `shout` function (not `shout()`). Gradio calls it when the user clicks Submit. You'll see this clearly in Act 5.

---

## Layer 2: "How the Internet Works" Basics

> **When:** Explain at the start of Act 1 (before the first API call)
> **Why:** These bridge the gap between Python → API calls.

---

### Q: How does the internet work (for our purposes)?
**[VERBAL]**

Very simply: your code sends a **request** to a server → the server processes it → the server sends back a **response**. That's it. Every API call follows this pattern.

---

### Q: What is HTTP? What are POST and GET?
**[VERBAL]**

HTTP is the protocol for sending requests over the internet. Two types matter:
- **GET** = "give me data" (fetching a web page)
- **POST** = "here's some data, process it" (sending a prompt to OpenAI)

When we call `requests.get(url)` for web scraping → GET. When we call the OpenAI API → POST (sending our prompt).

---

### Q: What is an API?
**[VERBAL]**

A way for code to talk to another service. Instead of going to chat.openai.com and typing in a browser, your Python code sends the same request directly to OpenAI's servers and gets back the same answer — programmatically.

---

### Q: What is `requests.post` vs `OpenAI()` client?
**[NOTEBOOK + VERBAL]**

**Say first:** They do the same thing. `requests.post` is raw HTTP — you build the request yourself. `OpenAI()` is a convenience wrapper that builds the request for you.

**Then point to:** The "Client Library vs Raw HTTP" demo in Act 1. Show both side by side. Highlight the syntax difference:
- Raw: `response.json()["choices"][0]["message"]["content"]` (dictionary access)
- Client: `response.choices[0].message.content` (attribute access)

Same data, different access syntax.

---

### Q: What is `localhost`?
**[VERBAL]**

Your own computer acting as a server. When Gradio launches and shows `http://localhost:7860`, it means your computer is running a web server that you can visit in your browser. Only you can see it (unless you share the link).

---

### Q: What is a rate limit error?
**[VERBAL]**

Too many requests too fast. The API says "slow down!" It's like a restaurant saying "we can only take 10 orders per minute." If students hit this during exercises, just wait a few seconds and try again.

---

## Layer 3: Design Decision Questions

> **When:** Explain AS the relevant code appears in the notebook
> **Why:** These are "why this way?" questions — answer them at the moment students see the code.

---

## During Act 1: API Calls

---

### Q: What is `client = OpenAI()` doing?
**[NOTEBOOK + VERBAL]**

**Point to:** Setup cell (cell 4).

It creates an object that knows how to talk to OpenAI's servers. It's NOT a live connection — it's a configured helper. Think of it as hiring a waiter who knows the restaurant's menu and address. The actual "trip to the restaurant" happens when you call `.chat.completions.create()`.

---

### Q: What does `client.chat.completions.create` accept?
**[NOTEBOOK]**

**Point to:** The first API call in Act 1 (cell 8).

Two required things:
- `model` — which model to use (e.g., `"gpt-4o-mini"`)
- `messages` — the conversation as a list of dicts

---

### Q: What should the messages list look like?
**[NOTEBOOK]**

**Point to:** Cell 8.

A list of dictionaries, each with `"role"` and `"content"`:
```python
[
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "What is AI?"}
]
```

---

### Q: What is a prompt?
**[VERBAL]**

The text you send to the model. In our code, it's the `"content"` value in the messages. The system prompt tells the model HOW to behave. The user prompt tells it WHAT to do right now.

---

### Q: What are the three roles (system, user, assistant)?
**[NOTEBOOK + VERBAL]**

**Point to:** Cell 8 for system/user, then cell 26 for all three.

- **system** — standing instructions (like "always respond in Hindi")
- **user** — the human's input
- **assistant** — the model's previous reply (used when building conversation history)

---

### Q: Why is `response.choices[0].message.content` so long?
**[NOTEBOOK + VERBAL]**

**Point to:** Cell 10 where the raw response is printed.

The API returns a complex object with metadata (model used, token counts, finish reason, etc.). We drill down to just the text:
- `response` → the full object
- `.choices[0]` → first response option (the API can return multiple, but we usually get one)
- `.message.content` → the actual text

---

### Q: Why `base_url` for Gemini?
**[NOTEBOOK]**

**Point to:** Cell 4 (setup) and cell 15 (provider comparison).

Same waiter (OpenAI client library), different kitchen (Gemini's servers). We just change the `base_url` to point to Google's API instead of OpenAI's. The code stays identical — same `messages`, same `.choices[0].message.content`.

---

## During Act 1: Web Scraping

---

### Q: What is BeautifulSoup?
**[NOTEBOOK + VERBAL]**

**Point to:** Cell 5 (`fetch_website_contents`).

A library that parses HTML so you can extract text from it. Websites are written in HTML (tags, styles, scripts) — BeautifulSoup strips away all that code and gives you just the text.

---

### Q: Why do we need to parse HTML?
**[VERBAL]**

Because websites aren't plain text. If you fetch a web page, you get `<div class="content"><p>Hello</p></div>`. The model doesn't need HTML tags — it needs "Hello". BeautifulSoup converts HTML → clean text.

---

### Q: What is `headers` in requests?
**[NOTEBOOK + VERBAL]**

**Point to:** The `HEADERS` dict in cell 5.

Pretending to be a browser so websites don't block us. Some websites reject requests that don't look like they're coming from a real browser. The `User-Agent` header says "I'm Chrome on Windows" — it's like wearing a uniform to get past security.

---

### Q: What is `fetch_website_contents`?
**[NOTEBOOK]**

**Point to:** Cell 5.

A function that: fetches a web page → strips away HTML junk (scripts, styles, images) → returns clean text. It also truncates to `max_chars` so the text fits in the model's context window.

---

### Q: What is `fetch_website_links`?
**[VERBAL]**

A function from notebook 4 that gets all URLs from a page. It's used in chaining — the LLM picks which links are relevant, then code fetches those pages. Not shown in the revision notebook, but mentioned in Act 3's pipeline explanation.

---

### Q: What is `max_chars` truncation?
**[NOTEBOOK + VERBAL]**

**Point to:** The `[:max_chars]` slice in cell 5.

Limiting text size so it fits in the context window. Web pages can be huge — we only take the first N characters. Otherwise the API call would fail or cost too much.

---

### Q: Why does website content go in the USER prompt, not SYSTEM?
**[NOTEBOOK]**

**Point to:** The explanation cell in Act 1 (cell 11).

System = standing instructions (HOW to behave). User = the specific thing to process THIS time. The website content is the "order" — it changes every time. The system prompt stays the same ("summarize this clearly").

---

### Q: What is `create_messages` doing?
**[VERBAL]**

Building the messages list from the system prompt + website content. It takes the system instruction and the fetched website text and assembles them into the format the API expects. This is from notebook 1 — the revision notebook shows the messages list built inline instead.

---

### Q: What is `summarize_website` doing?
**[VERBAL]**

Yes, it calls the LLM. The function flow: calls `fetch_website_contents` → then `create_messages` → then the API. It's a wrapper that combines fetching + prompting + calling into one step. The revision notebook builds these steps inline so you can see each one.

---

### Q: What is `display_summary`?
**[VERBAL]**

A wrapper that calls `summarize_website` and renders the result as formatted markdown using `Markdown()` + `display()`. It's the outermost layer in notebook 1.

---

### Q: Why `Markdown` + `display`?
**[VERBAL]**

Renders formatted text in Jupyter instead of raw markdown syntax. Without it, you'd see `**bold**` as literal text. With `display(Markdown(text))`, you see **bold**. It's a Jupyter display feature.

---

### Q: What is the function flow in notebook 1?
**[VERBAL]**

`display_summary` → `summarize_website` → `fetch_website_contents` + `create_messages` → API call.

Each function calls the next one down. The revision notebook flattens this into visible steps so students see each layer.

---

## During Act 2: Tokens & Memory

---

### Q: Why tokenization?
**[NOTEBOOK + VERBAL]**

**Point to:** Cell 20 (tokenization demo).

Models don't read text — they read numbers (token IDs). Every word gets converted to one or more numbers before the model processes it. That's why we have token counts and token costs.

---

### Q: Why not character-level tokenization?
**[VERBAL]**

Too granular, too expensive, loses word meaning. "Hello" as characters = 5 tokens. As a word token = 1 token. Character-level would make everything 5x more expensive and the model would struggle to understand words.

---

### Q: Do different models tokenize differently?
**[NOTEBOOK + VERBAL]**

**Point to:** Cell 20 — show how the same text produces different token counts.

Yes. Every model family has its own tokenizer (encoding). That's why `tiktoken.encoding_for_model("gpt-4o-mini")` specifies which model — the token IDs would be different for a different model.

---

### Q: What are context windows?
**[NOTEBOOK + VERBAL]**

**Point to:** The context window explanation cell (cell 27).

The maximum "memory" of a single API call. Everything — system prompt + conversation history + current message + the response — must fit within this limit (e.g., 128K tokens for GPT-4o-mini). Go over, and the call fails.

---

### Q: What is statelessness?
**[NOTEBOOK]**

**Point to:** Cells 23-26 (the memory illusion demo).

Each API call is completely independent. The model has zero memory between calls. When ChatGPT seems to "remember," it's sending the entire conversation history with every message. The demo proves this — Call 2 forgets the name, Call 3 (with history) remembers it.

---

## During Act 3: JSON & Chaining

---

### Q: What are JSON objects?
**[NOTEBOOK + VERBAL]**

**Point to:** Cell 34 (JSON mode demo).

Structured key-value data that code can parse. Like a form with labeled fields:
```json
{"topic": "AI", "difficulty": "beginner", "summary": "..."}
```
Code can read `result["topic"]` to get "AI". Free text can't be parsed this reliably.

---

### Q: What is `response_format`?
**[NOTEBOOK]**

**Point to:** Cell 34.

Tells the model to return JSON instead of free text. `response_format={"type": "json_object"}` forces the output to be valid JSON. Without it, the model might return JSON-ish text with extra words around it.

---

### Q: What is one-shot prompting?
**[NOTEBOOK + VERBAL]**

**Point to:** The system prompt in cell 34 that includes an example.

Giving one example in the prompt so the model follows the format. The system prompt says: `'Example: {"topic": "AI", "summary": "...", "difficulty": "beginner"}'`. This shows the model exactly what shape the output should have.

---

### Q: What is the full function flow in notebook 4 (pamphlet generator)?
**[VERBAL]**

`create_pamphlet` → `create_pamphlet_prompt` → `fetch_page_and_relevant_links` → `select_relevant_links` (LLM call 1: picks which links matter) + `fetch_website_contents` (fetches those pages) → LLM call 2 (writes the pamphlet).

The revision notebook simplifies this into Act 3's pipeline diagram. Point to the pipeline explanation in cell 37.

---

### Q: What is chaining?
**[NOTEBOOK + VERBAL]**

**Point to:** Cell 37 (pipeline explanation).

Output of one LLM call feeds into the next. LLM Call 1 returns JSON saying "these links are relevant" → code fetches those links → LLM Call 2 uses that content to write the pamphlet. The first LLM's decision drives what happens next — that's why it's called "agentic."

---

## During Act 4: Streaming

---

### Q: What are chunks?
**[NOTEBOOK]**

**Point to:** Cell 44 (streaming demo).

Small pieces of the response arriving one at a time. Instead of waiting for the full answer, you get it word by word (or a few tokens at a time). Each piece is called a "chunk."

---

### Q: Why `.delta.content` instead of `.message.content`?
**[NOTEBOOK + VERBAL]**

**Point to:** Cell 43 (non-streaming) vs cell 44 (streaming).

`delta` = the small new piece that just arrived. `message` = the complete message. In streaming, each chunk only contains the NEW part (delta), not the full message so far. That's why we accumulate: `result += chunk.choices[0].delta.content or ""`.

---

### Q: What is `update_display`?
**[VERBAL]**

Jupyter's way of updating a cell's output in place — like refreshing a scoreboard. It was used in notebook 4 for streaming in plain Jupyter. The revision notebook skips it because Gradio handles display updates automatically via `yield`.

---

## During Act 5: Gradio

---

### Q: What does each Gradio component parameter do?
**[NOTEBOOK]**

**Point to:** Cell 49 (simplest Gradio app).

- `fn` — your Python function (the "cook")
- `inputs` — what the user provides (textbox, dropdown, etc.)
- `outputs` — where the result appears (textbox, markdown, etc.)

---

### Q: How do Python functions work as variable names?
**[NOTEBOOK + VERBAL]**

**Point to:** `fn=shout` in cell 49.

`fn=shout` passes the function itself (like handing someone a recipe card). `fn=shout()` would CALL the function immediately and pass the RESULT. The parentheses matter:
- `shout` = the function object
- `shout()` = calling the function right now

---

### Q: What are callbacks (in Gradio context)?
**[NOTEBOOK + VERBAL]**

**Point to:** Cell 49.

Gradio calls YOUR function when the user clicks Submit. You don't call `shout()` yourself — you hand it to Gradio with `fn=shout`, and Gradio calls it at the right time with whatever the user typed. That's a callback: "call this function back when something happens."

---

### Q: `gr.Interface` vs `gr.ChatInterface` — what's the difference?
**[NOTEBOOK]**

**Point to:** Cell 49 (Interface) vs cell 57 (ChatInterface).

- `gr.Interface` — one-shot: input → output. No memory between uses.
- `gr.ChatInterface` — conversation with history. Gradio automatically manages the `history` parameter and displays the chat thread.

Use Interface for single tasks (summarize this, explain that). Use ChatInterface for back-and-forth conversations.

---

## Quick Reference: Teaching Flow

```
BEFORE NOTEBOOK (10 min)
  Layer 1: pip install, libraries, getpass, os.environ, f-strings,
           classes, generators, callbacks
     ↓
START OF ACT 1 (5 min)
  Layer 2: HTTP basics, APIs, requests vs client, localhost, rate limits
     ↓
DURING ACTS 1-5 (as code appears)
  Layer 3: Design decisions — answer "why?" at the moment students see the code
```

---

*This guide accompanies `revision-unit1-and-unit2.ipynb`. The notebook has the code; this guide has the verbal explanations.*
