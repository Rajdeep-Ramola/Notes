# 📚 Supermemory

> Supermemory is the long-term and short-term memory and context infrastructure for AI agents.

---

## 📑 Table of Contents

1. [How Does It Work?](#1-how-does-it-work)
2. [Ways to Add Context](#2-ways-to-add-context)
3. [Basic Example](#3-basic-example)

---

## 1. How Does It Work?

- You send Supermemory text, files, and chats.
- Supermemory intelligently indexes them and builds a semantic understanding graph on top of an entity (e.g., a user, a document, a project, an organization).
- At query time, we fetch only the most relevant context and pass it to your models.

---

## 2. Ways to Add Context

Supermemory offers 3 ways to add context to our LLMs:

**Memory API** — Continuously learns, updates, and forgets user facts in real time to maintain an evolving, accurate context for the LLM.

**User Profiles** — Maintains a unified profile of long‑term user traits and short‑term conversational history to enable deeply personalized responses.

**RAG (Advanced Semantic Search)** — Combines user memory with smart context retrieval using metadata filtering and chunking to deliver highly relevant, grounded answers.

---

## 3. Basic Example

```python
from supermemory import Supermemory

client = Supermemory()

USER_ID = "dhravya"

conversation = [
    {"role": "assistant", "content": "Hello, how are you doing?"},
    {"role": "user", "content": "Hello! I am Dhravya. I am 20 years old. I love to code!"},
    {"role": "user", "content": "Can I go to the club?"},
]

# Get user profile + relevant memories for context (creates profile if it doesn't exist)
profile = client.profile(container_tag=USER_ID, q=conversation[-1]["content"])

# Split supermemory into 3 layers
static = "\n".join(profile.profile.static)
dynamic = "\n".join(profile.profile.dynamic)
memories = "\n".join(r.get("memory", "") for r in profile.search_results.results)

# Build an overall context
context = f"""Static profile:

{static}

Dynamic profile:

{dynamic}

Relevant memories:

{memories}"""

# Build messages with memory-enriched context
messages = [{"role": "system", "content": f"User context:\n{context}"}, *conversation]

# response = llm.chat(messages=messages)

# Store conversation for future context
client.add(
    content="\n".join(f"{m['role']}: {m['content']}" for m in conversation),
    container_tag=USER_ID,
)
```

> **Note:** Use the `threshold` parameter in `client.profile()` to filter search results by relevance score.
