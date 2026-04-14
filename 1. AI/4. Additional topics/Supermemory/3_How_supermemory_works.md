# ⚙️ How Supermemory Works

> A guide to how Supermemory builds, connects, and manages memories using a semantic knowledge graph.

---

## 📑 Table of Contents

1. [How Supermemory Works](#1-how-supermemory-works)
2. [How Graph Memory Works](#2-how-graph-memory-works)
   - [2.1 Memory Types](#21-memory-types)

---

## 1. How Supermemory Works

Supermemory creates memories which are:

- Semantic chunks with meaning
- Embedded for similarity search
- Connected through relationships
- Dynamically updated over time

**The graph connects memories through relationships:**

### Updates: Information Changes

When new information contradicts or updates existing knowledge, Supermemory creates an "update" relationship.

### Extends: Information Enriches

When new information adds to existing knowledge without replacing it, Supermemory creates an "extends" relationship.

### Derives: Information Infers

When Supermemory infers new connections from patterns in your knowledge.

---

## 2. How Graph Memory Works

Supermemory's graph is facts built on top of other facts.

**Automatic memory extraction** — From a single conversation, Supermemory extracts multiple connected memories.

**Automatic forgetting** — Supermemory knows when memories become irrelevant:

**Time-based forgetting:** Temporary facts are automatically forgotten when they expire.

```
"I have an exam tomorrow"
↓
After the exam date passes → automatically forgotten

"Meeting with Alex at 3pm today"
↓
After today → automatically forgotten
```

**Contradiction resolution:** When new facts contradict old ones, the Update relationship ensures searches return current information.

**Noise filtering:** Casual, non-meaningful content doesn't become permanent memories.

---

### 2.1 Memory Types

Supermemory distinguishes memory types automatically:

| Type       | Example                          | Behavior                        |
|------------|----------------------------------|---------------------------------|
| Facts      | "Alex is a PM at Stripe"         | Persists until updated          |
| Preferences| "Alex prefers morning meetings"  | Strengthens with repetition     |
| Episodes   | "Met Alex for coffee Tuesday"    | Decays unless significant       |
