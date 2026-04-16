# 🔍 Search Memories and Docs

> A reference for search modes, filtering, and query optimization in Supermemory.

---

## 📑 Table of Contents

1. [Search Modes](#1-search-modes)
2. [Filtering](#2-filtering)
3. [Query Optimization](#3-query-optimization)

---

## 1. Search Modes

**Hybrid** — searches both memories and doc chunks, returns the most relevant.

**Memories** — only searches extracted memories.

```javascript
// Hybrid: memories + document chunks (recommended)
await client.search.memories({
  q: "quarterly goals",
  containerTag: "user_123",
  searchMode: "hybrid"
});

// Memories only: just extracted facts
await client.search.memories({
  q: "user preferences",
  containerTag: "user_123",
  searchMode: "memories"
});
```

---

## 2. Filtering

For metadata-based filtering:

```javascript
const results = await client.search.memories({
  q: "meeting notes",
  containerTag: "user_123",
  filters: {
    AND: [
      { key: "type", value: "meeting" },
      { key: "year", value: "2024" }
    ]
  }
});
```

**Filter Types:**

- **String equality:** `{ key: "status", value: "active" }`
- **String contains:** `{ filterType: "string_contains", key: "title", value: "react" }`
- **Numeric:** `{ filterType: "numeric", key: "priority", value: "5", numericOperator: ">=" }`
- **Array contains:** `{ filterType: "array_contains", key: "tags", value: "important" }`
- **Negate:** `{ key: "status", value: "draft", negate: true }`

---

## 3. Query Optimization

**Reranking** — re-scores results for better relevance. Adds ~100ms latency.

**Threshold** — control result quality vs quantity.

```javascript
const results = await client.search.memories({
  q: "complex technical question",
  containerTag: "user_123",
  rerank: true,
  threshold: 0.6
});
```
