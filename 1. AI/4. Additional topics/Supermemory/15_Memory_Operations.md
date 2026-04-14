# 🧠 Memory Operations

> A reference for creating, forgetting, and updating memories directly in Supermemory.

---

## 📑 Table of Contents

1. [Create Memories](#1-create-memories)
2. [Forget Memory](#2-forget-memory)
3. [Update Memory](#3-update-memory)

---

## 1. Create Memories

Create memories directly. Useful for storing user preferences when we already know the exact memory content.

```javascript
const response = await fetch("https://api.supermemory.ai/v4/memories", {
  method: "POST",
  headers: {
    "Authorization": `Bearer ${API_KEY}`,
    "Content-Type": "application/json"
  },
  body: JSON.stringify({
    memories: [
      {
        content: "John prefers dark mode",
        isStatic: false,
        metadata: { source: "user_preference" }
      },
      {
        content: "John is from Seattle",
        isStatic: true
      }
    ],
    containerTag: "user_123"
  })
});

const data = await response.json();
// {
//   documentId: "abc123",
//   memories: [
//     { id: "mem_1", memory: "John prefers dark mode", isStatic: false, createdAt: "2025-..." },
//     { id: "mem_2", memory: "John is from Seattle", isStatic: true, createdAt: "2025-..." }
//   ]
// }
```

**Parameters:**

| Parameter              | Type    | Required | Description                                                                         |
|------------------------|---------|----------|-------------------------------------------------------------------------------------|
| memories               | array   | yes      | Array of memory objects (1–100 items)                                               |
| memories[].content     | string  | yes      | The memory text (max 10,000 chars). Should be entity-centric, e.g. "John prefers dark mode" |
| memories[].isStatic    | boolean | no       | `true` for permanent identity traits (name, hometown). Defaults to `false`          |
| memories[].metadata    | object  | no       | Key-value metadata (strings, numbers, booleans)                                     |
| containerTag           | string  | yes      | Space / container tag these memories belong to                                      |

---

## 2. Forget Memory

Soft deleting a memory:

```javascript
await fetch("https://api.supermemory.ai/v4/memories/mem_abc123/forget", {
  method: "POST",
  headers: {
    "Authorization": `Bearer ${API_KEY}`
  }
});
```

---

## 3. Update Memory

Update a memory by creating a new version:

```javascript
await fetch("https://api.supermemory.ai/v4/memories", {
  method: "PATCH",
  headers: {
    "Authorization": `Bearer ${API_KEY}`,
    "Content-Type": "application/json"
  },
  body: JSON.stringify({
    // Identify by ID or content
    id: "mem_abc123",
    // content: "Original content to match",
    newContent: "Updated content goes here",
    metadata: {
      tags: ["updated"]
    }
  })
});
```

**Parameters:**

| Parameter  | Type   | Required | Description                                   |
|------------|--------|----------|-----------------------------------------------|
| id         | string | *        | Memory ID to update                           |
| content    | string | *        | Original content to match (alternative to ID) |
| newContent | string | yes      | New content for the memory                    |
| metadata   | object | no       | Updated metadata                              |
