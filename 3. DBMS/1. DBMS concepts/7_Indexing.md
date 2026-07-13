# Indexing

---

## Table of Contents

1. [What is Indexing](#1-what-is-indexing)
2. [Why Indexing](#2-why-indexing)
3. [Structure of an Index](#3-structure-of-an-index)
   - 3.1 [Search Key](#31-search-key)
   - 3.2 [Data Reference](#32-data-reference)
4. [Indexing Methods](#4-indexing-methods)
   - 4.1 [Primary Index (Clustering Index)](#41-primary-index-clustering-index)
   - 4.2 [Secondary Index (Non-Clustering Index)](#42-secondary-index-non-clustering-index)
5. [Dense and Sparse Index](#5-dense-and-sparse-index)
   - 5.1 [Dense Index](#51-dense-index)
   - 5.2 [Sparse Index](#52-sparse-index)
6. [Primary Indexing – Key vs Non-Key Attribute](#6-primary-indexing--key-vs-non-key-attribute)
   - 6.1 [Based on Key Attribute](#61-based-on-key-attribute)
   - 6.2 [Based on Non-Key Attribute (Clustering Index)](#62-based-on-non-key-attribute-clustering-index)
7. [Multi-level Index](#7-multi-level-index)
8. [Advantages of Indexing](#8-advantages-of-indexing)
9. [Limitations of Indexing](#9-limitations-of-indexing)

---

## 1. What is Indexing

Indexing is a **data structure technique** used to locate and access the data in a database table quickly, without having to search every row of the table.

- Indexing is **optional** — a table can function without an index — but it increases access speed massively.
- It is **not the primary means** to access a tuple (row); it is a **secondary means**. The primary means is always the actual data file itself.
- An **index file is always sorted**, even if the underlying data file is not.

**Example:** Think of a book. You could find a topic by flipping through every page (like scanning a table row by row), or you could jump straight to the page number using the **index at the back of the book**. A database index works the same way — it points you directly to the disk block where the data lives.

---

## 2. Why Indexing

The core purpose of indexing is to **reduce the search space**. Instead of scanning the entire table, the DBMS uses the index to jump straight to the relevant disk block and then performs the search there.

- It **optimises performance** by **minimising the number of disk accesses** required when a query is processed.
- It **speeds up read operations** — `SELECT` queries, `WHERE` clauses, joins, etc.

**Example:** Consider a `Students` table with 1 million rows.

| roll_no | name | dept | marks |
|---------|------|------|-------|
| 1001 | Aman | CSE | 88 |
| 1002 | Riya | ECE | 76 |
| ... | ... | ... | ... |
| 500000 | Zoya | ME | 91 |

A query like `SELECT * FROM Students WHERE roll_no = 500000;` without an index would force the DBMS to scan all 1 million rows (a full table scan). With an index on `roll_no`, the DBMS directly jumps to the disk block containing that row.

---

## 3. Structure of an Index

An index record (also called an **index entry**) is made up of two parts:

### 3.1 Search Key

- Contains a **copy of the primary key or candidate key** of the table — or, in some cases, some other attribute entirely.
- This is the value the DBMS actually searches through when looking up data.

### 3.2 Data Reference

- A **pointer** holding the **address of the disk block** where the value of the corresponding search key is actually stored.
- This is what lets the DBMS jump directly to the data instead of scanning for it.

**Example:** An index on `roll_no` for the `Students` table above might look like:

| Search Key (roll_no) | Data Reference (Block Address) |
|-----------------------|--------------------------------|
| 1001 | Block 101 |
| 1002 | Block 101 |
| 1003 | Block 102 |
| ... | ... |

---

## 4. Indexing Methods

A file may have **several indices**, built on different search keys, on top of the same data file.

### 4.1 Primary Index (Clustering Index)

- If the data file containing the records is stored in **sequential (sorted) order**, a **Primary Index** is an index whose search key **also defines the sequential order** of the file.
- All files here are ordered sequentially on **some** search key — it could be the **primary key**, or it could be a **non-primary key** attribute.

> **Note:** The term "primary index" is sometimes loosely used to mean "an index built on the primary key." This usage is **non-standard and should be avoided** — a primary index is really about the file being *ordered* on the search key, not about the search key being the *primary key* specifically.

**Example:** A `Students` table physically sorted on disk by `roll_no`:

| roll_no | name | dept |
|---------|------|------|
| 1001 | Aman | CSE |
| 1002 | Riya | ECE |
| 1003 | Karan | ME |

Since the data file itself is physically sorted by `roll_no`, an index built on `roll_no` here is a **primary (clustering) index**.

### 4.2 Secondary Index (Non-Clustering Index)

- Data file is **unsorted** with respect to this attribute — hence a primary index is **not possible** on it.
- Called "secondary" indexing because normally **one indexing (the primary index) is already applied** to the file, and this is an *additional* index on top.
- Can be built on a **key or a non-key** attribute.
- **Number of entries in the index file = number of records in the data file** (i.e., it is always a **dense** index — see Section 5.1).

**Example:** The same `Students` table, physically sorted by `roll_no`, but now we also want fast lookups by `name`:

| Search Key (name) | Data Reference |
|--------------------|-----------------|
| Aman | Block 101 |
| Karan | Block 102 |
| Riya | Block 101 |

Here `name` does **not** match the physical sort order of the file (which is sorted by `roll_no`), so this is a **secondary index** — and it must have one entry per record.

---

## 5. Dense and Sparse Index

Indices can further be classified based on **how many entries** they contain relative to the data file.

### 5.1 Dense Index

- Contains an **index record for every single search-key value** in the data file.
- Each index record holds the search-key value and a **pointer to the first data record** with that search-key value; the remaining records with the same key are stored **sequentially right after** it.
- Needs **more space** to store the index itself, since there's an entry for every record (or every unique key, if duplicates are handled via chaining).

**Example:** Dense index on `dept` (non-key, but every unique value indexed):

| Search Key (dept) | Pointer |
|--------------------|---------|
| CSE | → Block 101 |
| ECE | → Block 101 |
| ME | → Block 102 |

If duplicates exist, only the *first* occurrence is pointed to, and the rest follow sequentially in the data file.

### 5.2 Sparse Index

- An index record appears for **only some** of the search-key values — not all of them.
- It helps resolve the space problem of dense indexing.
- A **range of records maps to the same data block address**; when a specific record needs to be retrieved, the DBMS fetches the block and then **scans within that block** to find the exact record.
- Requires **less space** than a dense index, but lookups need one extra step (scan inside the block).

**Example:** Sparse index on `roll_no`, where only the *first* roll number of each disk block is indexed:

| Search Key (roll_no) | Data Reference |
|------------------------|-----------------|
| 1001 | Block 101 |
| 1050 | Block 102 |
| 1099 | Block 103 |

To find `roll_no = 1075`, the DBMS jumps to Block 102 (since 1050 ≤ 1075 < 1099) and then scans within that block.

---

## 6. Primary Indexing – Key vs Non-Key Attribute

Primary indexing can be based on whether the data file is sorted with respect to a **key attribute** or a **non-key attribute**.

### 6.1 Based on Key Attribute

- Data file is sorted with respect to the **primary key attribute**.
- The primary key is used as the search key in the index.
- Since the primary key is unique, we don't need an entry for every record — a **sparse index** is formed, i.e., **number of entries in the index file = number of blocks in the data file**.

**Example:** `Students` table sorted by `roll_no` (primary key), one index entry per block:

| Search Key (roll_no) | Block |
|------------------------|-------|
| 1001 | Block 101 |
| 1010 | Block 102 |
| 1020 | Block 103 |

### 6.2 Based on Non-Key Attribute (Clustering Index)

- Data file is sorted with respect to a **non-key attribute** (values can repeat).
- **Number of entries in the index = number of unique non-key attribute values** in the data file.
- This ends up being a **dense index**, since all unique values get an entry.
- This is also called a **Clustering Index** — records with the same value of the non-key attribute are grouped ("clustered") together.

**Example:** A company recruits employees across departments. If the `Employees` table is physically sorted by `dept` (a non-key, repeating attribute), clustering indexing should be created for all employees belonging to the same department:

| Search Key (dept) | Pointer |
|--------------------|---------|
| Finance | → Block 201 |
| HR | → Block 203 |
| Sales | → Block 205 |

All `Finance` employees are stored contiguously starting at Block 201, all `HR` employees starting at Block 203, and so on.

---

## 7. Multi-level Index

- An index built with **two or more levels**.
- If a single-level index itself becomes **too large**, doing a binary search *within the index* would take too much time — so we **break the index down into multiple levels**, creating an "index of the index."
- We can build indexing with 2 or more such levels, forming a tree-like structure.

**Example:** A huge index on `roll_no` split into an outer (top-level) index and an inner (lower-level) index:

**Outer Index (index of the index):**

| Search Key | Points to Inner Index Block |
|------------|------------------------------|
| 1001 | Inner Block A |
| 5001 | Inner Block B |

**Inner Index Block A:**

| Search Key (roll_no) | Data Reference |
|------------------------|-----------------|
| 1001 | Block 101 |
| 1050 | Block 102 |

The DBMS first does a quick lookup on the small outer index, then jumps to the relevant inner index block, and only then to the actual data block — cutting down search time drastically.

---

## 8. Advantages of Indexing

- **Faster access and retrieval** of data.
- **Less I/O** — fewer disk accesses are required to locate a record.

---

## 9. Limitations of Indexing

- Requires **additional space** to store the index table itself, separate from the data file.
- **Decreases performance** on `INSERT`, `DELETE`, and `UPDATE` queries, since the index must be updated every time the underlying data changes.
