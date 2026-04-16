# 📥 Ingesting Context to Supermemory

> Guide to updating, replacing, formatting, uploading, and deleting content in Supermemory.

---

## 📑 Table of Contents

1. [Updating Content](#1-updating-content)
2. [Replace Entire Document](#2-replace-entire-document)
3. [Formatting Conversations](#3-formatting-conversations)
4. [Upload Files](#4-upload-files)
5. [Delete Content](#5-delete-content)

---

## 1. Updating Content

Use `customId` to update existing docs or conversations. When you send content with the same `customId`, Supermemory intelligently processes only what's new.

```javascript
// Supermemory detects the diff and only processes new parts
await client.add({
  content: "user: Hi, I'm Sarah.\nassistant: Nice to meet you!\nuser: What's the weather?\nassistant: It's sunny today.",
  customId: "conv_123",
  containerTag: "user_sarah"
});
```

---

## 2. Replace Entire Document

```javascript
// Replace the entire document content
await client.documents.update("doc_id_123", {
  content: "Completely new content replacing everything",
  metadata: { version: 2 }
});
```

---

## 3. Formatting Conversations

We can format our conversations however we want:

```javascript
// Simple string
content: "user: Hello\nassistant: Hi there!"

// JSON stringify
content: JSON.stringify(messages)

// Template literal
content: messages.map(m => `${m.role}: ${m.content}`).join('\n')

// Any format --- just make it a string
content: formatConversation(messages)
```

---

## 4. Upload Files

```python
with open('document.pdf', 'rb') as file:
    client.documents.upload_file(
        file=file,
        container_tags='user_123'
    )
```

---

## 5. Delete Content

**Single delete:**

```javascript
await client.documents.delete("doc_id_123");
```

**Bulk delete by IDs:**

```javascript
await client.documents.deleteBulk({
  ids: ["doc_1", "doc_2", "doc_3"]
});
```
