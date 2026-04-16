# 🗂️ Organizing and Filtering Memories

> Supermemory provides 2 ways to organize your memories: Container Tags and Metadata.

---

## 📑 Table of Contents

1. [Container Tags](#1-container-tags)
2. [Metadata](#2-metadata)
   - [2.1 Filter Types](#21-filter-types)
   - [2.2 Excluding Results](#22-excluding-results)
   - [2.3 Metadata Key Rules](#23-metadata-key-rules)

---

## 1. Container Tags

Container tags create isolated memory spaces. Use them to separate memories by user, project, or any logical boundary.

```javascript
await client.add({
  content: "Meeting notes from Q1 planning",
  containerTags: ["user_123"]
});
```

**Recommended Patterns:**

| Pattern          | Example                        | Use Case                    |
|------------------|--------------------------------|-----------------------------|
| User isolation   | `user_{userId}`                | Per-user memories           |
| Project grouping | `project_{projectId}`          | Project-specific content    |
| Hierarchical     | `org_{orgId}_team_{teamId}`    | Multi-level organization    |

---

## 2. Metadata

Metadata lets you attach custom properties to memories and filter by them later.

**Adding memories with metadata:**

```javascript
await client.add({
  content: "Technical design document for auth system",
  containerTags: ["user_123"],
  metadata: {
    category: "engineering",
    priority: "high",
    year: 2024
  }
});
```

**Searching with metadata filters:**

```javascript
await client.add({
  content: "Technical design document for auth system",
  containerTags: ["user_123"],
  metadata: {
    category: "engineering",
    priority: "high",
    year: 2024
  }
});
```

---

### 2.1 Filter Types

| Type             | Example                                                                             | Description        |
|------------------|-------------------------------------------------------------------------------------|--------------------|
| String equality  | `{ key: "status", value: "published" }`                                             | Exact match        |
| String contains  | `{ filterType: "string_contains", key: "title", value: "react" }`                  | Substring match    |
| Numeric          | `{ filterType: "numeric", key: "priority", value: "5", numericOperator: ">=" }`    | Number comparison  |
| Array contains   | `{ filterType: "array_contains", key: "tags", value: "important" }`                | Check array membership |

---

### 2.2 Excluding Results

Use `negate: true` to exclude matches:

```javascript
const results = await client.search.documents({
  q: "documentation",
  filters: {
    AND: [
      { key: "status", value: "draft", negate: true }
    ]
  }
});
```

**Real-world pattern — User's work documents from 2024:**

```javascript
const results = await client.search.documents({
  q: "quarterly report",
  containerTags: ["user_123"],
  filters: {
    AND: [
      { key: "category", value: "work" },
      { key: "type", value: "report" },
      { filterType: "numeric", key: "year", value: "2024", numericOperator: "=" }
    ]
  }
});
```

---

### 2.3 Metadata Key Rules

- Allowed characters: `a-z`, `A-Z`, `0-9`, `_`, `-`, `.`
- Max length: 64 characters
- No spaces or special characters
