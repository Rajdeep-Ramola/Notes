# 🚀 Super RAG

> Supermemory doesn't just store your content — it transforms it into optimized, searchable knowledge. Every upload goes through an intelligent pipeline that extracts, chunks, and indexes content in the ideal way for its type.

---

## 📑 Table of Contents

1. [Automatic Content Intelligence](#1-automatic-content-intelligence)
2. [Smart Chunking by Content Type](#2-smart-chunking-by-content-type)
3. [Hybrid Memory + RAG](#3-hybrid-memory--rag)
4. [Search Optimization](#4-search-optimization)
5. [Why It's "Super"](#5-why-its-super)

---

## 1. Automatic Content Intelligence

> Detects the content type
>
> Extracts content optimally
>
> Chunks intelligently
>
> Generates embeddings
>
> Builds relationships

---

## 2. Smart Chunking by Content Type

- **Documents** — chunked by semantic sections
- **Code** — chunked using code-chunk
- **Web pages** — chunked by article structure: headings, paragraphs, lists
- **Markdown** — chunked by heading hierarchy

---

## 3. Hybrid Memory + RAG

Supermemory combines the best of both approaches in every search.

With `searchMode: "hybrid"` (the default), you get both:

```javascript
const results = await client.search({
  q: "how do I deploy the app?",
  containerTag: "user_123",
  searchMode: "hybrid"
});

// Returns:
// - Deployment docs from your knowledge base (RAG)
// - User's previous deployment preferences (Memory)
// - Their specific environment configs (Memory)
```

---

## 4. Search Optimization

**Reranking** — Re-scores results using a cross-encoder model for better relevance:

```javascript
const results = await client.search({
  q: "complex technical question",
  rerank: true // +~100ms, significantly better ranking
});
```

Use when precision matters more than speed.

**Query rewriting** — Expands your query to capture more relevant results:

```javascript
const results = await client.search({
  q: "how to auth",
  rewriteQuery: true // Expands to "authentication login oauth jwt..."
});
```

---

## 5. Why It's "Super"

| Traditional RAG               | SUPER RAG                        |
|-------------------------------|----------------------------------|
| Manual chunking config        | Automatic per content type       |
| One-size-fits-all splits      | AST-aware code chunking          |
| Just document retrieval       | Hybrid memory + documents        |
| Static embeddings             | Relationship-aware graph         |
| Generic search                | Rerank + query rewriting         |
