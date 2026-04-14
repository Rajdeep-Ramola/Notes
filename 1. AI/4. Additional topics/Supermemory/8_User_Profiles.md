# 👤 User Profiles

> User profiles are automatically maintained collections of facts about your users that Supermemory builds from all their interactions.

---

## 📑 Table of Contents

1. [Why We Use Profiles](#1-why-we-use-profiles)
2. [Profiles Separate 2 Types of Information](#2-profiles-separate-2-types-of-information)
3. [How It Works](#3-how-it-works)
4. [Profiles + Search](#4-profiles--search)
5. [Use Cases](#5-use-cases)

---

## 1. Why We Use Profiles

| Problem             | Search Only                    | With Profiles        |
|---------------------|--------------------------------|----------------------|
| Context retrieval   | 3–5 queries                    | 1 call               |
| Response time       | 200–500ms                      | 50–100ms             |
| Basic user info     | Requires specific queries      | Always available     |

**Search is too narrow:** When you search for "project updates", you miss that the user prefers bullet points, works in PST, and uses specific terminology.

**Profiles provide the foundation:** Instead of searching for basic context, profiles give your LLM a complete picture of who the user is.

---

## 2. Profiles Separate 2 Types of Information

**Static Profile** — Long-term, stable facts.

**Dynamic Profile** — Recent context and temporary states.

---

## 3. How It Works

Profiles are built automatically through ingestion:

1. **Ingest content** — Users add documents, chat, or any content
2. **Extract facts** — AI analyzes content for facts about the user
3. **Update profile** — System adds, updates, or removes facts
4. **Always current** — Profiles reflect the latest information

---

## 4. Profiles + Search

Profiles don't replace search — they complement it:

- **Profile** = broad foundation (who the user is, preferences, background)
- **Search** = specific details (exact memories matching a query)

**Example:**

User asks: **"Can you help me debug this?"**

Without profiles: LLM has no context about expertise, projects, or preferences.

---

## 5. Use Cases

- Personalized AI assistant
- Customer support
- Educational platform
- Development tools
