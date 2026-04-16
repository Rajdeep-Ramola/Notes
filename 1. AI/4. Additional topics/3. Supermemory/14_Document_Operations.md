# 📋 Document Operations

> A reference for listing, retrieving, updating, and monitoring documents in Supermemory.

---

## 📑 Table of Contents

1. [List Documents](#1-list-documents)
2. [Get Document](#2-get-document)
3. [Update Document](#3-update-document)
4. [Processing Queue](#4-processing-queue)

---

## 1. List Documents

```python
documents = client.documents.list(
    limit=10,
    container_tags=["user_123"]
)

for doc in documents.memories:
    print(doc.id, doc.title, doc.status)
```

**Parameters:**

| Parameter      | Type       | Default     | Description                        |
|----------------|------------|-------------|------------------------------------|
| limit          | number     | 50          | Items per page (max 200)           |
| page           | number     | 1           | Page number                        |
| containerTags  | string[]   | —           | Filter by tags                     |
| sort           | string     | createdAt   | Sort by `createdAt` or `updatedAt` |
| order          | string     | desc        | `desc` (newest) or `asc` (oldest)  |

---

## 2. Get Document

Get a specific document:

```python
doc = client.documents.get("doc_abc123")
print(doc.status)
print(doc.content)
```

---

## 3. Update Document

Update a document's content or metadata:

```python
client.documents.update(
    "doc_abc123",
    content="Updated content here",
    metadata={"version": 2, "reviewed": True}
)
```

---

## 4. Processing Queue

Check documents currently being processed:

```python
response = client.documents.list_processing()
print(f"{len(response.documents)} documents processing")
```
