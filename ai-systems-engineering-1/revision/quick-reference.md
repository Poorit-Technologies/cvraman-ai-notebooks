# Quick-Reference Table: AI Systems Engineering 1 — Revision Session

**47 concepts in teaching order. Scan in under 2 minutes.**

---

## Layer 1: Python Foundations (before opening the notebook — 10 min)

| # | Concept | Key Talking Points | Analogy | Code |
|---|---------|-------------------|---------|------|
| 1 | `pip install` | Downloads a library so your Python code can use it. `!pip install` is pip inside Jupyter. | Like downloading an app on your phone | `!pip install openai tiktoken gradio` |
| 2 | Libraries / Packages | Pre-written code so you don't have to write it from scratch. `openai` talks to the API, `requests` fetches web pages, `beautifulsoup4` parses HTML, `tiktoken` counts tokens, `gradio` builds UIs. | — | `import openai` |
| 3 | `getpass` vs `input` | Both take user input; `getpass` hides what you type. Use it for secrets like API keys. | — | `from getpass import getpass`<br>`key = getpass("API Key: ")` |
| 4 | `os.environ` | A shared noticeboard for your program — pin a value, any code can read it later. The OpenAI library reads the key from here automatically. | Shared noticeboard | `os.environ['OPENAI_API_KEY'] = key` |
| 5 | f-strings | Put variables inside strings. The `f` before quotes means "format this" — Python replaces `{name}` with the actual value. | — | `f"Hello {name}"` |
| 6 | `class`, `__init__`, `self` | A class is a blueprint. `__init__` is the setup step. `self` refers to "this particular object." `OpenAI()` creates an object from the OpenAI class. | Recipe card: class = recipe name, `__init__` = ingredients, `self` = this dish | `client = OpenAI()` |
| 7 | Generators / `yield` vs `return` | `return` gives everything at once; `yield` gives results one at a time. A function with `yield` becomes a generator — loop over it with `for`. | `return` = match highlights DVD; `yield` = live ball-by-ball commentary | `def gen():`<br>`    yield "a"`<br>`    yield "b"` |
| 8 | Callbacks | Passing a function as an argument. You hand it to someone else and say "call this when you're ready." In Gradio, `fn=shout` — note no parentheses. | — | `gr.Interface(fn=shout, ...)` |
| 9 | `Markdown` + `display` | Renders formatted text in Jupyter. Without it you see raw `**bold**`; with it you see **bold**. | — | `display(Markdown(text))` |

---

## Layer 2: How the Internet Works (start of Act 1 — 5 min)

| # | Concept | Key Talking Points | Analogy | Code |
|---|---------|-------------------|---------|------|
| 10 | Request → Response cycle | Your code sends a request to a server → server processes it → server sends back a response. Every API call follows this pattern. | — | — |
| 11 | HTTP / GET vs POST | HTTP is the protocol. GET = "give me data" (fetching a page). POST = "here's data, process it" (sending a prompt to OpenAI). | — | `requests.get(url)` / `requests.post(url, json=payload)` |
| 12 | API | A way for code to talk to another service. Instead of typing in a browser, your Python code sends the request directly and gets the answer programmatically. | — | — |
| 13 | Client library vs Raw HTTP | They do the same thing. The client library builds the request for you. Raw HTTP: dictionary access `["key"]`. Client: attribute access `.key`. | Same waiter, same order — one fills out the form for you | Client: `response.choices[0].message.content`<br>Raw: `response.json()["choices"][0]["message"]["content"]` |
| 14 | `localhost` | Your own computer acting as a server. When Gradio shows `localhost:7860`, only you can see it. | — | `http://localhost:7860` |
| 15 | Rate limit error | Too many requests too fast. The API says "slow down!" Wait a few seconds and retry. | Restaurant: "we can only take 10 orders per minute" | — |

---

## Act 1: API Calls

| # | Concept | Key Talking Points | Analogy | Code |
|---|---------|-------------------|---------|------|
| 16 | `client = OpenAI()` | Creates a configured helper that knows how to talk to OpenAI's servers. NOT a live connection — the actual call happens at `.create()`. | Hiring a waiter who knows the restaurant's menu and address | `client = OpenAI(api_key=key)` |
| 17 | `client.chat.completions.create` | The actual API call. Two required params: `model` and `messages`. | Placing the order | `client.chat.completions.create(model=MODEL, messages=messages)` |
| 18 | Messages list format | A list of dicts, each with `"role"` and `"content"`. This is the conversation the model sees. | The order slip | `[{"role": "system", "content": "..."},`<br>` {"role": "user", "content": "..."}]` |
| 19 | Prompt (system vs user) | The text you send to the model. System prompt = HOW to behave (standing instructions). User prompt = WHAT to do right now (the specific order). | System = chef's standing instructions; User = each specific order | — |
| 20 | Three roles: system, user, assistant | `system` = standing instructions. `user` = the human's input. `assistant` = the model's previous reply (used to build conversation history). | — | `{"role": "assistant", "content": prev_reply}` |
| 21 | `response.choices[0].message.content` | The API returns a complex object with metadata. We drill down: `response` → `.choices[0]` (first response option) → `.message.content` (the actual text). | — | `response.choices[0].message.content` |
| 22 | `base_url` for Gemini | Same client library, different server. Change `base_url` to point to Google's API; code stays identical. | Same waiter, different kitchen | `OpenAI(base_url="https://generativelanguage.googleapis.com/v1beta/openai/", api_key=key)` |

---

## Act 1: Web Scraping

| # | Concept | Key Talking Points | Analogy | Code |
|---|---------|-------------------|---------|------|
| 23 | BeautifulSoup | Parses HTML so you can extract text. Websites are HTML tags + styles + scripts — BS4 strips it all and gives you just the text. | — | `soup = BeautifulSoup(response.content, "html.parser")` |
| 24 | Why parse HTML | Raw fetch gives `<div><p>Hello</p></div>`. The model needs "Hello", not HTML tags. BS4 converts HTML → clean text. | — | `soup.body.get_text(separator="\n", strip=True)` |
| 25 | `headers` / User-Agent | Pretending to be a browser so websites don't block us. The `User-Agent` header says "I'm Chrome." | Wearing a uniform to get past security | `HEADERS = {"User-Agent": "Mozilla/5.0 ..."}` |
| 26 | `fetch_website_contents` | Fetches a web page → strips HTML junk (scripts, styles, images) → returns clean text, truncated to `max_chars`. | — | `def fetch_website_contents(url, max_chars=2000):` |
| 27 | `max_chars` truncation | Limiting text size so it fits in the context window. Web pages can be huge — we slice to first N characters. | — | `text[:max_chars]` |
| 28 | Website content in USER prompt | System = standing instructions (HOW). User = specific content to process THIS time. Website text is the "order" — it changes every call. | System = chef's rules; User = each order | `{"role": "user", "content": website_text}` |
| 29 | Function flow (Notebook 1) | `display_summary` → `summarize_website` → `fetch_website_contents` + `create_messages` → API call. Each function calls the next one down. | Nested wrappers — outermost calls innermost | — |
| 30 | `fetch_website_links` | Gets all URLs from a page. Used in chaining — the LLM picks which links are relevant, then code fetches those pages. | — | — |

---

## Act 2: Tokens & Memory

| # | Concept | Key Talking Points | Analogy | Code |
|---|---------|-------------------|---------|------|
| 31 | Tokenization | Models read numbers (token IDs), not text. Every word is converted to one or more numbers before processing. That's why we have token counts and costs. | — | `encoding = tiktoken.encoding_for_model("gpt-4o-mini")`<br>`tokens = encoding.encode("Hello")` |
| 32 | Why not character-level | Too granular, too expensive, loses word meaning. "Hello" as characters = 5 tokens; as a word token = 1. Character-level makes everything ~5× more expensive. | — | — |
| 33 | Different model tokenizers | Every model family has its own tokenizer. Same text → different token counts depending on the model. That's why you specify the model in `encoding_for_model()`. | — | `tiktoken.encoding_for_model("gpt-4o-mini")` |
| 34 | Context window | The maximum "memory" of a single API call. System prompt + history + current message + response must all fit (e.g., 128K tokens for GPT-4o-mini). Go over → call fails. | — | — |
| 35 | Statelessness | Each API call is completely independent — zero memory. ChatGPT "remembers" by sending the entire conversation history every time. Proven by the 3-call demo: Call 2 forgets; Call 3 (with history) remembers. | The Memory Illusion | `messages = [system, user1, assistant1, user2]` |

---

## Act 3: JSON & Chaining

| # | Concept | Key Talking Points | Analogy | Code |
|---|---------|-------------------|---------|------|
| 36 | JSON objects | Structured key-value data that code can parse. Like a form with labeled fields. Code reads `result["topic"]` reliably — free text can't be parsed this way. | A form with labeled fields | `{"topic": "AI", "difficulty": "beginner"}` |
| 37 | `response_format` | Tells the model to return valid JSON instead of free text. Without it, the model might wrap JSON in extra words. | — | `response_format={"type": "json_object"}` |
| 38 | One-shot prompting | Give one example in the prompt so the model follows the format. The system prompt includes a sample JSON to show the expected output shape. | — | `'Example: {"topic": "AI", "summary": "..."}'` |
| 39 | Chaining (agentic) | Output of one LLM call feeds into the next. LLM Call 1 returns JSON ("these links are relevant") → code fetches those links → LLM Call 2 uses that content. The first LLM's decision drives what happens next. | Factory assembly line — Station 1 labels the box, Station 2 fetches parts, Station 3 assembles | — |
| 40 | Pamphlet pipeline (Notebook 4) | `create_pamphlet` → `create_pamphlet_prompt` → `fetch_page_and_relevant_links` → `select_relevant_links` (LLM call 1) + `fetch_website_contents` → LLM call 2. | Assembly line with 3 stations | — |

---

## Act 4: Streaming

| # | Concept | Key Talking Points | Analogy | Code |
|---|---------|-------------------|---------|------|
| 41 | Chunks | Small pieces of the response arriving one at a time instead of waiting for the full answer. Each piece is a "chunk." Enable with `stream=True`. | Ball-by-ball cricket commentary vs full highlights | `stream = client.chat.completions.create(..., stream=True)` |
| 42 | `.delta.content` vs `.message.content` | `delta` = the small NEW piece that just arrived. `message` = the complete message. In streaming, accumulate deltas: `result += delta.content or ""`. | — | `for chunk in stream:`<br>`    result += chunk.choices[0].delta.content or ""` |
| 43 | `update_display` | Jupyter's way of updating a cell's output in place — like refreshing a scoreboard. Used in notebook 4. The revision notebook skips it because Gradio handles updates via `yield`. | Refreshing a live scoreboard | — |

---

## Act 5: Gradio

| # | Concept | Key Talking Points | Analogy | Code |
|---|---------|-------------------|---------|------|
| 44 | `gr.Interface` params | `fn` = your Python function. `inputs` = what the user provides. `outputs` = where the result appears. Any Python function becomes a web app. | The shop counter | `gr.Interface(fn=shout, inputs="textbox", outputs="textbox").launch()` |
| 45 | Functions as variables | `fn=shout` passes the function itself (recipe card). `fn=shout()` would CALL it immediately and pass the result. Parentheses matter. | Handing someone a recipe card vs cooking the dish right now | `fn=shout` ✓<br>`fn=shout()` ✗ |
| 46 | Callbacks in Gradio | Gradio calls YOUR function when the user clicks Submit. You don't call `shout()` yourself — Gradio calls it at the right time with the user's input. That's a callback. | "Call this function back when something happens" | `gr.Interface(fn=shout, ...)` — Gradio calls `shout(user_input)` on Submit |
| 47 | `gr.Interface` vs `gr.ChatInterface` | `Interface` = one-shot (input → output, no memory). `ChatInterface` = conversation with history; Gradio manages the `history` param and displays the chat thread. | Interface = shop counter; ChatInterface = restaurant with order history | `gr.ChatInterface(fn=chat, type="messages").launch()` |

---

*Source: [teaching-guide.md](teaching-guide.md) and [revision-unit1-and-unit2.ipynb](revision-unit1-and-unit2.ipynb)*
