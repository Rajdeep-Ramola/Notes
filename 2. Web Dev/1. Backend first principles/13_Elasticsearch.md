# Full-Text Search & Elasticsearch

---

# The Problem: Why Traditional Database Search Breaks Down

## A Story from 2005

Imagine it is the year 2005. You are a software engineer at a rapidly growing e-commerce company — the company is booming because of the dot-com bubble. You are given a task: write a database query to build an API that searches through thousands of products and returns relevant results based on a user's input.

Your company has around **5,000 products** at this point. This is how you write the search query in a typical relational database setup:

```sql
SELECT * FROM products
WHERE name ILIKE '%laptop%'
OR description ILIKE '%laptop%';
```

The `%` symbols are for matching any characters that come before and after the keyword. So if the word `laptop` appears anywhere in the `name` or `description` column, that row is returned.

A customer searches for *laptop*, they get a few results, and life is very simple.

---

## What Happens When the Company Grows

Since it is 2005 and many companies were growing rapidly, your company also grew — and it grew fast. Suddenly you had **millions of products**.

That very straightforward `ILIKE` and percentage-symbol-based search query that once returned results in around **50 milliseconds** is now taking **30 seconds**.

Customers are frustrated. Your manager is frustrated. Everyone is asking you to optimize it, and you are clueless.

---

## The New Requirements

Beyond just making it faster, there are more requirements on the table:

* **Relevance** — When someone searches for *laptop*, we want to show the most relevant results first. For example, show a *MacBook Pro* before a *laptop bag*.
* **Typo tolerance** — Customers in a hurry during sales events frequently type things like *labtop* instead of *laptop*. Even with typos, we still want to return relevant results for *laptop*.
* **Speed** — We need results in milliseconds, not seconds.

With all these requirements — faster, more relevant, and typo-tolerant — this is the story of why **search engines like Elasticsearch** came into existence.

---

# Why Relational Databases Struggle with Text Search

## The Librarian Analogy

Think of your Postgres database as a librarian. You go to the librarian and ask for the location of books on a particular topic. The librarian knows exactly where every book is located — **but it has one fatal flaw**.

If you ask for a book on *machine learning*, here is what the librarian does:

1. Picks up the first book: *Harry Potter and the Philosopher's Stone* — no match, moving on.
2. Picks up the next book: *Game of Thrones* — no match, moving on.
3. Picks up the next book: *Introduction to Machine Learning* — match found, returns it.
4. Continues this process through every single book in the entire library.

Depending on the size of the library, this might take a few minutes, a few hours, or even a few days if you have a library with 10 million or 1 billion books.

**Two core problems emerge:**

* **Speed** — The librarian has to go through every book one by one. It is painfully slow at scale.
* **No sense of relevance** — If the librarian finds two books — one titled *Introduction to Machine Learning* and another where the phrase "machine learning" is mentioned only on the last page — it has no way of knowing which is more relevant. It might return the less relevant one first.

---

## How This Translates to a Database Query

A query like this:

```sql
SELECT * FROM products
WHERE name ILIKE '%laptop%'
OR description ILIKE '%laptop%';
```

This is **exactly** how the librarian searches through the library. The database has to:

* Scan every single row
* Examine every single text field
* Perform pattern matching character by character

It is thorough — it will return all matching results — but it is **painfully slow** at scale, and it has **no sense of relevance**. Results are returned in whatever order the database encounters them internally.

---

# The Information Explosion and the Birth of Search Technology

Around the same time as this scaling problem emerged, the **information explosion** happened. Companies like:

* **Google** — processing and indexing billions of web pages
* **Amazon** — cataloging millions of products
* **LinkedIn** — indexing millions of professional profiles

...simply could not afford to wait 30 seconds for search results. At their scale, even a **2-second delay** is considered unacceptable. Search results need to return in **milliseconds**, because every second of latency costs millions of dollars in lost conversions.

To solve this, the answer came from **decades of research** in the field of text-based search and information retrieval. Computer scientists had been studying this problem since the **1960s**.

---

# The Key Invention: Inverted Index

## The Revolutionary Idea

The breakthrough thought was this:

> Instead of searching through every document to find the relevant terms, what if we **flipped the problem**? What if we looked at it the other way around?

This key idea is called the **inverted index**, and it changed text-based search forever.

---

## How an Inverted Index Works

Going back to the librarian analogy — instead of the librarian going to every book and searching through its title and content for the term "machine learning", here is what we do instead:

**When books first arrive on the shelves**, we take all the words in every book and build an index. For every word, we record:

* Which books contain that word
* Which pages that word appears on

For example, imagine we build this index for the word *machine*:

| Book | Pages |
|---|---|
| *Introduction to Machine Learning* | Pages 1, 15, 23 |
| *The Machine Age* | Pages 5, 89 |
| *Coffee Machine Manual* | Page 1 |

And for the word *learning*:

| Book | Pages |
|---|---|
| *Introduction to Machine Learning* | Pages 1, 16, 24 |
| *Learning to Cook* | Pages 3, 7 |
| *Deep Learning Fundamentals* | Pages 2, 45, 91 |

Now when someone searches for *machine learning*, the librarian does **not** go through every book. Instead, it goes directly to the index, looks up *machine* and *learning*, and immediately knows which books contain those terms and how frequently they appear.

> Instead of going through the content and finding the terms, we have the terms and through the terms we find the content. **We inverted the search** — that is why it is called an inverted index.

---

## What Powers Elasticsearch

This technique — the inverted index — is the technology that powers **Elasticsearch** and similar full-text search tools.

Elasticsearch itself is not a completely new invention. It is built on top of a foundational technology called **Apache Lucene**, which is the core inverted-index-based text search engine. Many other tools also use Apache Lucene under the hood.

> Elasticsearch is not the only full-text search tool. Even modern relational databases like **Postgres** now have built-in support for full-text search, using similar underlying concepts.

---

# Relevance Scoring

## The Librarian is Now Faster — and Smarter

With an inverted index, the librarian can quickly look up which books contain the term *machine*. But there is another huge advantage: **relevance**.

From the index, we can see that:

* *Introduction to Machine Learning* has the word *machine* on pages 1, 15, and 23 (in practice, it would appear hundreds of times throughout the book)
* *The Machine Age* has it on pages 5 and 89
* *Coffee Machine Manual* has it on page 1

Elasticsearch uses this information to **score results by relevance** — so the most meaningful results appear at the top.

---

## The BM25 Algorithm

Elasticsearch uses an algorithm called **BM25** to rank and sort results. It considers several factors:

### 1. Term Frequency
How often a particular term appears in a single document. If the word *machine* appears 100 times in a book, that book is likely very relevant to a search for *machine*.

### 2. Document Frequency
How common the term is **across all documents**. If a word appears in almost every document, it is less of a distinguishing signal.

### 3. Document Length
How long the document is. A term appearing 10 times in a short document is more significant than the same term appearing 10 times in a very long document.

### 4. Field Boosting
This is a feature used heavily in practice. The **location** of the term within a document affects its relevance score:

* Term appears in the **title** → highest relevance boost
* Term appears in the **description** → medium relevance boost
* Term appears in the **body/content** → lower relevance boost

You can configure field boosting yourself using Elasticsearch's query language. If you want content matches to matter more for your particular use case, you can override the defaults. It is fully configurable.

---

## Relevance Scoring in Practice

Consider a search for *machine learning*. Here is how the results would be ranked:

1. **Introduction to Machine Learning** — the terms appear in the title (highest boost) and appear many times throughout the book (high term frequency) → **ranked first**
2. **The Machine Age** — the term *machine* appears in the title but is not as frequently used → **ranked second**
3. **Coffee Machine Manual** — *machine* appears in the title but very infrequently → **ranked third**

---

# Typo Tolerance and Type-Ahead Search

## Handling User Typos

One of the major advantages of full-text search tools like Elasticsearch is their ability to handle **typos** in user queries.

For example, a user types: `what is treading today` (mistyped `trending` as `treading`)

A full-text search system can **derive from context** that the user likely intended to search for `what is trending today` and returns those results. This is a major advantage for building search experiences — especially during high-traffic periods like sales events when users are typing in a hurry.

---

## Type-Ahead Interfaces

You can use Elasticsearch to build a **type-ahead interface** — the kind you see on websites like Amazon, where suggested results appear as you type. Each keystroke fires a search, and relevant results are returned in milliseconds.

This kind of experience is powered by the same full-text search capabilities underneath — fast index lookups, relevance scoring, and typo tolerance working together.

---

# Elasticsearch vs. Postgres Full-Text Search

When building a search feature in your backend application, you have two main options:

## Option 1: Postgres Full-Text Search

Modern Postgres offers built-in full-text search capabilities. It uses similar concepts (an inverted index under the hood) and can be a great choice if:

* Your search requirements are not extremely complex
* You are already using Postgres and want to avoid adding another service to your infrastructure

## Option 2: Elasticsearch

Elasticsearch is a standalone search engine. It is a better choice if:

* Your company already uses Elasticsearch as part of the **ELK stack** (Elasticsearch, Logstash/Kibana) for log management — so the infrastructure is already there
* You need advanced search features, large-scale performance, or richer relevance tuning

---

## The ELK Stack

**ELK** stands for:

* **E**lasticsearch
* **L**ogstash (or Filebeat)
* **K**ibana

This is a very famous stack for managing logs — searching through them, deriving statistics, and visualizing data. Since Elasticsearch is extremely fast at searching through large volumes of text, it became a natural fit for log management. If your company already has ELK deployed, using Elasticsearch for full-text search product features as well makes a lot of sense.

---

# Live Demo: Postgres vs. Elasticsearch

## Setup

A demo project was built using **Next.js** (chosen because LLMs are very good at generating prototype applications in Next.js). The goal was to compare search performance between:

* A **Neon serverless Postgres** instance (cloud-based)
* An **Elastic Cloud** instance

Both instances were deployed in the **US-West** region to keep network latency consistent between them.

### The Dataset

A table called `reviews` was created in Postgres with three fields:

* `id`
* `review` (text field)
* `sentiment` (either `positive` or `negative`)

A CSV file with **50,000 review entries** was used to populate both the Postgres table and the Elasticsearch index.

---

## The Postgres Search Query

```sql
SELECT id, review, sentiment
FROM reviews
WHERE review ILIKE '%laptop%';
```

`ILIKE` is used for case-insensitive matching. The `%` symbols ensure we match the keyword regardless of what characters come before or after it.

---

## The Elasticsearch Search Query

In Elasticsearch, the index was created with the following field mappings:

* `review` → type `text` (full-text analyzed — treats words as individual tokens)
* `sentiment` → type `keyword` (exact match only)

The search was run using Elasticsearch's query string API, converting the search term to lowercase to maximize breadth.

---

## Results

| Query | Elasticsearch | Postgres (`ILIKE`) |
|---|---|---|
| `laptop` | ~1 second | ~3–4 seconds |
| `only` | ~500 milliseconds | ~7.5 seconds |

Both returned the **same number of results** (since both were matching with equivalent case-insensitive patterns), but the **time difference is significant**:

* Elasticsearch returned results up to **15x faster** than Postgres `ILIKE` in some queries
* As the dataset grows, this gap only widens

---

# When Should You Use Full-Text Search?

## The Rule of Thumb

> If you have a **search or type-ahead feature** in your backend application, go with full-text search — either Postgres full-text search or Elasticsearch.

You do not need to master the internals of Elasticsearch the way you need to master databases. Databases are foundational — almost every part of your backend codebase as an engineer involves databases, so you need to master indexing, query optimization, and everything in between.

Elasticsearch, on the other hand, is something you can get started with quickly by:

* Copying relevant snippets from the official docs or an LLM
* Following the provided examples for your use case
* Reading a little more into the docs if you need to optimize

Most common search-based use cases are covered well by the examples and default configurations in the Elasticsearch documentation.

---

# Key Concepts Summary

| Concept | Definition |
|---|---|
| **ILIKE** | Case-insensitive pattern matching in SQL using `%` wildcards — scans every row, slow at scale |
| **Inverted Index** | An index that maps terms to the documents containing them — the core data structure behind fast full-text search |
| **Apache Lucene** | The foundational open-source library that implements the inverted index; powers Elasticsearch and many other tools |
| **Elasticsearch** | A distributed full-text search engine built on Apache Lucene; supports fast search, relevance scoring, and typo tolerance |
| **BM25** | The relevance ranking algorithm used by Elasticsearch; considers term frequency, document frequency, document length, and field boosting |
| **Term Frequency** | How often a search term appears within a single document |
| **Document Frequency** | How commonly a search term appears across all documents in the index |
| **Field Boosting** | Assigning higher relevance weight to a term when it appears in more important fields (e.g., title > description > body) |
| **Typo Tolerance** | The ability to return relevant results even when the user's query contains spelling mistakes |
| **Type-Ahead** | A search interface that shows suggested results as the user types, powered by fast full-text search |
| **ELK Stack** | Elasticsearch + Logstash/Filebeat + Kibana — a popular stack for log management and observability |
| **Postgres Full-Text Search** | Built-in full-text search capabilities in modern Postgres — a lighter alternative to Elasticsearch for simpler use cases |