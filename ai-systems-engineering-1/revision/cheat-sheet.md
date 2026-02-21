# AI Systems Engineering 1 - Cheat Sheet

**CV Raman Global University** | AI Center of Excellence

---

## 1. The API Call Pattern

```python
from openai import OpenAI

client = OpenAI(api_key="your-key")

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "system", "content": "You are a helpful assistant"},
        {"role": "user", "content": "Your question here"}
    ]
)

print(response.choices[0].message.content)
```

---

## 2. Messages Format

```python
messages = [
    {"role": "system", "content": "Instructions for the model"},    # Chef's standing orders
    {"role": "user", "content": "First user message"},              # Customer's first order
    {"role": "assistant", "content": "Model's first response"},     # What the chef served
    {"role": "user", "content": "Follow-up message"}                # Next order
]
```

**Roles**: `system` (instructions) | `user` (your input) | `assistant` (model's response)

---

## 3. Switching Providers (Same Code, Different Kitchen)

```python
# OpenAI (default)
client = OpenAI(api_key="sk-...")

# Google Gemini
client = OpenAI(
    base_url="https://generativelanguage.googleapis.com/v1beta/openai/",
    api_key="your-google-key"
)

# Ollama (local, free)
client = OpenAI(
    base_url="http://localhost:11434/v1",
    api_key="ollama"       # dummy key - Ollama doesn't need one
)
```

---

## 4. Web Scraping

```python
import requests
from bs4 import BeautifulSoup

def fetch_website_contents(url, max_chars=2000):
    response = requests.get(url, headers={"User-Agent": "Mozilla/5.0"})
    soup = BeautifulSoup(response.content, "html.parser")
    title = soup.title.string if soup.title else "No title"
    if soup.body:
        for tag in soup.body(["script", "style", "img", "input"]):
            tag.decompose()                              # Remove unwanted elements
        text = soup.body.get_text(separator="\n", strip=True)
    else:
        text = ""
    return (title + "\n\n" + text)[:max_chars]           # Truncate to fit context
```

---

## 5. Streaming Responses

```python
# Enable streaming
stream = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=messages,
    stream=True                          # This enables streaming
)

# Read chunks as they arrive
result = ""
for chunk in stream:
    content = chunk.choices[0].delta.content or ""
    result += content
    print(content, end="")               # Print each piece as it arrives
```

**Key difference**: `response.choices[0].message.content` (normal) vs `chunk.choices[0].delta.content` (streaming)

---

## 6. JSON Structured Output

```python
import json

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "system", "content": "Respond in JSON with keys: topic, summary"},
        {"role": "user", "content": "Explain web scraping"}
    ],
    response_format={"type": "json_object"}    # Force JSON output
)

result = json.loads(response.choices[0].message.content)   # String -> dict
print(json.dumps(result, indent=2))                        # dict -> pretty string
```

---

## 7. Tokenization & Cost

```python
import tiktoken

encoding = tiktoken.encoding_for_model("gpt-4o-mini")
tokens = encoding.encode("Hello world")
print(f"Token count: {len(tokens)}")

# Cost = (input_tokens / 1M) * input_price + (output_tokens / 1M) * output_price
# GPT-4o-mini: $0.15/1M input, $0.60/1M output
```

---

## 8. Gradio Interfaces

```python
import gradio as gr

# Simple interface
gr.Interface(fn=my_function, inputs="textbox", outputs="markdown").launch()

# Streaming interface (use yield instead of return)
def stream_response(prompt):
    result = ""
    for chunk in stream:
        result += chunk.choices[0].delta.content or ""
        yield result                    # yield sends partial results to UI

gr.Interface(fn=stream_response, inputs="textbox", outputs="markdown").launch()

# Chat interface (with history)
def chat(message, history):
    history = [{"role": h["role"], "content": h["content"]} for h in history]
    messages = [{"role": "system", "content": "You are helpful"}] + history + [{"role": "user", "content": message}]
    stream = client.chat.completions.create(model="gpt-4o-mini", messages=messages, stream=True)
    response = ""
    for chunk in stream:
        response += chunk.choices[0].delta.content or ""
        yield response

gr.ChatInterface(fn=chat, type="messages").launch()
```

---

## Quick Reference Table

| What | Code |
|---|---|
| Create client | `client = OpenAI(api_key=key)` |
| Make API call | `client.chat.completions.create(model=MODEL, messages=msgs)` |
| Get response text | `response.choices[0].message.content` |
| Enable streaming | Add `stream=True` to create() |
| Get stream chunk | `chunk.choices[0].delta.content` |
| Force JSON output | Add `response_format={"type": "json_object"}` |
| Parse JSON string | `json.loads(string)` |
| Count tokens | `len(tiktoken.encoding_for_model("gpt-4o-mini").encode(text))` |
| Simple Gradio UI | `gr.Interface(fn, inputs, outputs).launch()` |
| Chat Gradio UI | `gr.ChatInterface(fn, type="messages").launch()` |
| Switch provider | Change `base_url` in `OpenAI()` constructor |

---

*AI Systems Engineering 1 | Developed by [Poorit Technologies](https://poorit.in)*
