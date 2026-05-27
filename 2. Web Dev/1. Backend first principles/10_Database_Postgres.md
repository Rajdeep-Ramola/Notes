# Databases and Backend Systems

# Introduction

When working with backend systems, interacting with and handling databases is one of the most important and frequent operations you’ll perform.

Because of that, understanding the concepts around databases is essential if you want to be efficient as a backend engineer.

This section covers:

* Why databases are needed
* What persistence means
* What a database is
* Disk-based storage vs RAM
* DBMS (Database Management Systems)
* Why text files are not enough
* Types of databases
* Relational vs Non-relational databases
* Why PostgreSQL is often the best choice
* Common PostgreSQL data types

---

# Why Do We Need Databases?

At its core, a database is simply a way to **persist information across different sessions**.

# What Is Persistence?

Persistence means:

> Storing data in a way that it survives even after the program that created it has stopped.

## Example: To-Do List App

Think about a to-do list application.

You:

* Add tasks
* Mark tasks as completed
* Close the app

Then later you reopen it.

You expect:

* Your tasks to still be there
* Completed tasks to remain checked
* The app to restore exactly the same state you left it in

That is persistence.

## Without Persistence

If persistence didn’t exist:

Every time you opened the app:

* Your tasks would disappear
* You would need to recreate everything
* You would lose all progress

That would make the app unusable.

---

# What Is a Database?

The term **database** is broader than many people think.

In the simplest sense:

> Any structured storage system can be considered a database.

## Examples of Databases

### 1. Contact List on Your Phone

Your phone stores:

* Names
* Numbers
* Emails

This is structured data.

That contact list is a database.

### 2. Browser Storage

Browsers provide storage mechanisms like:

* Local Storage
* Session Storage
* Cookies

These are also databases.

### 3. Text Files

Even a simple text file where you write notes and read them later can be considered a very basic database.

---

# CRUD Operations

If we derive the common pattern, a database is:

> A persistent system that provides ways to Create, Read, Update, and Delete data.

CRUD stands for:

* **Create**
* **Read**
* **Update**
* **Delete**

---

# Databases in Backend Systems

In backend development, when developers say “database,” they usually mean:

> Disk-based databases running in servers.

---

# RAM vs Disk Storage

## RAM (Primary Memory)

RAM is:

* Fast
* Expensive
* Limited in size

Typical sizes:

* 8 GB
* 16 GB
* 32 GB
* 64 GB

## Disk Storage (Secondary Memory)

Disk storage includes:

* HDD
* SSD

Disk storage is:

* Slower than RAM
* Much cheaper
* Available in much larger capacity

Typical sizes:

* 512 GB
* 1 TB
* 2 TB+

## Why Databases Use Disk

Traditional databases use disk because:

> They need large storage capacity at lower cost.

## Why Caches Use RAM

Caching systems like Redis often use RAM because:

> RAM is extremely fast for reads and writes.

---

# What Is DBMS?

DBMS stands for:

> **Database Management System**

A DBMS is software whose responsibility is to:

* Store data efficiently
* Provide CRUD operations
* Maintain correctness and consistency of data

---

# Responsibilities of a DBMS

## 1. Data Organization

Data must be organized so reads and writes remain efficient.

## 2. Data Access

It provides CRUD operations to users and applications.

## 3. Data Integrity

Integrity means:

> Data remains valid, accurate, and consistent.

### Example

If `payment_amount` should store numbers, then:

Valid:

```text
499
```

Invalid:

```text
hello
```

A DBMS should reject invalid values.

## 4. Security

DBMS systems also provide:

* Users
* Roles
* Permissions
* Access control

---

# Why Not Store Everything in Text Files?

Before modern databases, storing data in text files was common.

But it creates major problems.

## 1. Parsing Is Slow

To search data inside a text file, applications must:

* Read the file
* Parse the contents
* Split lines
* Compare values manually

This becomes slow as data grows.

## 2. No Structure

Text files do not enforce:

* Data types
* Required fields
* Validation rules

This makes consistency difficult.

## 3. Concurrency Problems

Concurrency means:

> Multiple users modifying the same data at the same time.

### Example

Suppose:

```text
amount = 40
```

Two users read it together.

One updates it to:

```text
60
```

Another updates it to:

```text
20
```

Whichever saves last wins.

This causes inconsistent results.

---

# Types of Databases

There are two major categories:

1. Relational Databases
2. Non-Relational Databases

---

# Relational Databases

Relational databases organize data in:

* Tables
* Rows
* Columns

Relationships between tables are defined using concepts like:

* Primary Key
* Foreign Key

## Key Characteristics

* Structured schema
* Strong constraints
* High data integrity
* SQL-based querying

## Examples

* PostgreSQL
* MySQL
* SQL Server
* SQLite

---

# Non-Relational Databases (NoSQL)

Non-relational databases are more flexible.

They do not require a strict predefined schema.

Different entries can have different structures.

## Example

MongoDB

A collection can store documents with varying shapes.

This makes it useful for dynamic or unpredictable data.

---

# Relational vs Non-Relational: Use Cases

## CRM → Relational Database

CRM systems store:

* Customer details
* Contacts
* Payments
* Sales opportunities

This data needs:

* Accuracy
* Relationships
* Consistency

Relational databases are a strong fit.

## CMS → Non-Relational Database

CMS systems may store:

* Articles
* Images
* Embeds
* Rich text blocks

Since content structure can vary, non-relational databases can work well here.

---

# Why PostgreSQL?

PostgreSQL is often the default recommendation for backend applications.

## Reasons

### 1. Open Source

Free and open source.

### 2. SQL Standard Compliant

Follows SQL standards closely.

Makes migration easier later.

### 3. Extensible

Supports many extensions and advanced features.

### 4. Reliable and Scalable

Widely trusted in startups and large companies.

### 5. Excellent JSON Support

PostgreSQL supports:

* `JSON`
* `JSONB`

This makes it flexible enough for structured and semi-structured data.

---

# PostgreSQL Data Types

## Numeric Types

### SERIAL

Auto-incrementing integer.

Often used for IDs.

### SMALLINT / INTEGER / BIGINT

Used for storing numbers of different size ranges.

### DECIMAL / NUMERIC

Used where precision matters.

Best for:

* Prices
* Money
* Financial values

Example:

```sql
DECIMAL(10,2)
```

### FLOAT / REAL / DOUBLE PRECISION

Used where approximate values are acceptable.

Best for:

* Measurements
* Scientific calculations

Rule:

* Use **DECIMAL** when accuracy matters
* Use **FLOAT** when performance matters more than exact precision

---

# String Types

## CHAR

Fixed-length text.

## VARCHAR

Variable-length text with maximum limit.

## TEXT

Stores text of any length.

Most commonly preferred in PostgreSQL.

---

# Boolean

Stores:

* `TRUE`
* `FALSE`

---

# Date and Time Types

Includes:

* `DATE`
* `TIME`
* `TIMESTAMP`
* `TIMESTAMPTZ`

`TIMESTAMPTZ` stores date, time, and timezone.

---

# UUID

UUID stands for:

> Universally Unique Identifier

Commonly used as a primary key.

Example:

```text
550e8400-e29b-41d4-a716-446655440000
```

---

# JSON and JSONB

PostgreSQL provides native JSON support.

## JSON

Stored as plain JSON text.

## JSONB

Stored in optimized binary format.

Advantages:

* Faster queries
* Better indexing
* Better performance

In most backend applications:

> Prefer `JSONB`

---

# Array Types

PostgreSQL can store arrays of values.

Examples:

* Arrays of integers
* Arrays of strings
* Arrays of JSON

---

# Summary

In this part we covered:

* Why databases are needed
* Persistence
* What databases are
* CRUD operations
* Disk vs RAM storage
* What DBMS is
* Why text files are not enough
* Relational vs non-relational databases
* Why PostgreSQL is widely preferred
* Important PostgreSQL data types

These concepts form the foundation for working with databases in backend development.

# Database Design, Migrations, and Modeling a Project Management Platform

## Introduction

Next, we are going to design the database for a **Project Management Platform**.

In the previous section, we designed the APIs. Now the goal is to understand practical database concepts—especially concepts used in **PostgreSQL**—by modeling the database schema and writing queries that power those APIs.

This covers both foundational and advanced relational database concepts used in real backend systems.

---

# Database Migrations

Before modeling tables, it’s important to understand **migrations**.

In production systems, developers do not directly open database GUIs (such as TablePlus) and manually run SQL queries against production databases.

Why?

Because then:

* Changes become hard to track over time.
* There’s no clear version history of schema updates.
* It becomes difficult to know **who changed what** and **when**.
* Rolling back changes becomes risky.

## What are migrations?

Migrations are version-controlled SQL files used to evolve a database schema over time.

Typical structure:

```bash
/db
  /migrations
    1.sql
    2.sql
    3.sql
```

Each file contains SQL statements such as:

* `CREATE TABLE`
* `ALTER TABLE`
* `CREATE INDEX`
* `DROP TABLE`
* `CREATE TYPE`

Migration tools execute these files **in sequence**.

Common naming strategies:

* Incremental numbers → `1.sql`, `2.sql`, `3.sql`
* Timestamps → `202605251210_create_users.sql`

---

# Migration Tools

Common migration tools:

* **dbmate**
* **golang-migrate**

These tools:

* Read migration files sequentially
* Apply schema changes to the database
* Track current schema version
* Support rollback when needed

---

# Up Migrations vs Down Migrations

Most migration systems have two sections:

## Up Migration

Defines what should be applied.

Example:

```sql
CREATE TABLE users (...);
```

## Down Migration

Defines how to undo that change.

Example:

```sql
DROP TABLE users;
```

This makes rollback possible if deployment fails.

---

# Why migrations are useful

## 1. Schema version tracking

Every database change is preserved in source control.

## 2. Rollbacks

If a deployment breaks production, migrations can be reversed.

## 3. Team collaboration

Everyone shares the same schema history.

## 4. Safe schema evolution

Changes are repeatable across:

* local development
* staging
* production

---

# Database Modeling: Project Management Platform

The platform will include:

* Users
* User Profiles
* Projects
* Tasks
* Project Members

---

# Enum Types

Before creating tables, custom enums are created.

## Project Status

```sql
active
completed
archived
```

## Task Status

```sql
pending
in_progress
completed
cancelled
```

## Member Role

```sql
owner
admin
member
```

---

# Why use Enums?

## Data Integrity

Only predefined values are allowed.

Example:

If `status` only allows:

```sql
active | completed | archived
```

then inserting:

```sql
"something-random"
```

will fail at database level.

---

## Better Documentation

Enums document allowed values directly in schema.

Anyone reading migrations can immediately understand allowed states without reading application code.

---

# Users Table

Stores account-level user data.

Typical fields:

* `id`
* `email`
* `full_name`
* `password_hash`
* `created_at`
* `updated_at`

---

# Primary Keys

`id` is the **primary key**.

Primary key implies:

* `NOT NULL`
* `UNIQUE`

This ensures every row can be uniquely identified.

---

# UUID IDs

Instead of incremental integers, UUIDs are used.

Benefits:

* globally unique
* hard to guess
* safer for public APIs

Example:

```sql
DEFAULT gen_random_uuid()
```

---

# Constraints

Important PostgreSQL constraints:

## NOT NULL

Prevents null values.

## UNIQUE

Prevents duplicate values.

Example:

```sql
email UNIQUE
```

No two users can register with the same email.

## CHECK

Custom validation logic.

Example:

```sql
CHECK (priority >= 1 AND priority <= 5)
```

---

# Naming Conventions

Common conventions followed:

## Table names → plural

Examples:

* `users`
* `projects`
* `tasks`

## Column names → snake_case

Examples:

* `full_name`
* `password_hash`
* `created_at`

Avoid camelCase because PostgreSQL is case-insensitive unless quoted.

---

# User Profiles Table

Stores optional profile-related information separately.

Examples:

* avatar URL
* bio
* phone number
* preferences
* metadata

---

# One-to-One Relationship

`users` ↔ `user_profiles`

Each user has one profile.

Implemented using:

```sql
user_id PRIMARY KEY REFERENCES users(id)
```

This makes `user_id` both:

* primary key
* foreign key

---

# Projects Table

Stores project data.

Fields:

* `id`
* `name`
* `description`
* `status`
* `owner_id`
* `created_at`
* `updated_at`

---

# Foreign Keys & Referential Integrity

Example:

```sql
owner_id REFERENCES users(id)
```

This guarantees the project owner must exist in `users` table.

---

# Referential Actions

## ON DELETE RESTRICT

Prevents deleting a parent row if dependent rows exist.

Example:

A user cannot be deleted if they still own projects.

---

## ON DELETE CASCADE

Deleting parent deletes related child rows.

Example:

Deleting a project also deletes all its tasks.

---

## ON DELETE SET NULL

Deletes parent, but sets child reference to null.

Used for optional references.

---

# Tasks Table

Represents tasks inside projects.

Fields include:

* `id`
* `project_id`
* `title`
* `description`
* `priority`
* `status`
* `due_date`
* `assigned_to`
* `created_at`
* `updated_at`

---

# One-to-Many Relationship

`projects` → `tasks`

One project can have many tasks.

Implemented using:

```sql
project_id REFERENCES projects(id)
```

---

# Many-to-Many Relationship

`users` ↔ `projects`

A user can belong to multiple projects.
A project can contain multiple users.

This is modeled through a **linking table**.

---

# Project Members Table

Acts as join table between users and projects.

Fields:

* `project_id`
* `user_id`
* `role`
* `created_at`
* `updated_at`

---

# Composite Primary Key

Instead of separate `id`, use:

```sql
PRIMARY KEY (project_id, user_id)
```

This ensures:

* a user cannot join the same project twice
* relationship remains unique

---

# Running Migrations

Using dbmate:

```bash
dbmate up
```

This applies migrations to database.

Migration tools also create a version tracking table like:

```sql
schema_migrations
```

This stores current migration version.

---

# Database Seeding

After schema creation, test data is inserted.

This process is called **seeding**.

Purpose:

* local testing
* development
* API testing before real users exist

---

# Seed Data Examples

Insert sample:

* users
* profiles
* projects
* tasks
* project members

Often done using:

```sql
INSERT INTO ...
```

or with **CTEs**.

---

# CTE (Common Table Expression)

Used to simplify multi-step insert operations.

Example:

```sql
WITH inserted_users AS (...)
```

Helps chain inserts cleanly.

---

# Writing Queries for APIs

Once schema and data exist, backend APIs query the database.

---

# Fetch All Users API

Example endpoint:

```http
GET /v1/users
```

Basic query:

```sql
SELECT * FROM users;
```

---

# Joining User Profiles

To return profile info along with user data:

```sql
FROM users u
LEFT JOIN user_profiles up
  ON u.id = up.user_id
```

---

# Why LEFT JOIN?

Because a user may exist without profile data.

LEFT JOIN ensures user is still returned even if profile row is missing.

---

# Converting Joined Row to JSON

PostgreSQL supports:

```sql
to_jsonb(...)
```

Used to embed profile object into response.

Example result:

```json
{
  "id": "...",
  "email": "...",
  "profile": {
    "bio": "...",
    "avatar_url": "..."
  }
}
```

---

# Sorting Query Results

Database results should usually be ordered explicitly.

Common pattern:

```sql
ORDER BY created_at DESC
```

This returns newest rows first.

---

# Parameterized Queries

Used when values come dynamically from user input.

Example:

```http
GET /v1/users/:userId
```

Here `userId` is dynamic.

---

# Why Parameterized Queries Matter

Main reason:

## Security

They prevent **SQL Injection**.

User input is treated as data, not executable SQL.

Even malicious input is escaped safely.

---

# Summary

This section covered:

* Database migrations
* Up/down migrations
* Schema versioning
* Enum types
* Primary keys
* UUIDs
* Constraints
* One-to-one relationships
* One-to-many relationships
* Many-to-many relationships
* Linking tables
* Composite primary keys
* Foreign keys
* Referential integrity
* Seeding data
* CTEs
* JOINs
* LEFT JOIN
* JSON aggregation
* Ordering query results
* Parameterized queries

These are core concepts used daily while building backend systems with PostgreSQL.

# 3. SQL Queries, Filtering, Pagination, Indexes, and Triggers

## Query Parameters in Backend APIs

Database tools often provide a UI to pass parameters into SQL queries, but in real backend development this is handled inside application code using the database driver or ORM.

A request generally flows like this:

```text
Frontend API call
→ Router
→ Handler / Controller
→ Service
→ Repository
→ Database query
→ Response serialized as JSON
```

---

# Fetching a Single User

To fetch one user by ID:

```sql
SELECT
  u.*,
  row_to_json(p.*) AS profile
FROM users u
LEFT JOIN user_profiles p ON p.user_id = u.id
WHERE u.id = :userId;
```

### What happens:

* `SELECT u.*` → fetches all user columns
* `LEFT JOIN` → fetches related profile data
* `WHERE u.id = :userId` → filters for one specific user
* `:userId` is a parameterized query variable

Example API:

```http
GET /v1/users/:userId
```

---

# Dynamic Filtering, Sorting & Pagination

List APIs usually support:

* Filtering
* Sorting
* Pagination

Examples:

```http
GET /users?page=1&limit=10&letter=j&sortBy=email&sortOrder=asc
```

---

## Default Query Parameters

If the client does not provide parameters:

```text
page = 1
limit = 10
sortBy = created_at
sortOrder = desc
```

---

# Filtering by First Letter of Name

Example query:

```sql
WHERE u.full_name ILIKE :letter || '%'
```

Example:

```text
letter = J
```

Matches:

```text
John Doe
Jane Smith
```

### Why `%`?

`%` means “anything after this”.

So:

```sql
J%
```

means:

```text
Starts with J
```

---

# Dynamic Sorting

Example:

```sql
ORDER BY :sortBy :sortOrder
```

Possible values:

```text
sortBy = email | full_name | created_at
sortOrder = asc | desc
```

Example:

```sql
ORDER BY u.email DESC
```

---

# Pagination

Pagination prevents returning massive datasets in a single API call.

Typical SQL:

```sql
LIMIT :limit
OFFSET :offset
```

Example:

```text
page = 1
limit = 10
offset = 0
```

For page 2:

```text
offset = 10
```

Formula:

```text
offset = (page - 1) * limit
```

---

# Insert Query – Create User

Example API:

```http
POST /users
```

SQL:

```sql
INSERT INTO users (
  email,
  full_name,
  password_hash
)
VALUES (
  :email,
  :fullName,
  :passwordHash
)
RETURNING *;
```

### Purpose

* Inserts a new user into `users`
* Returns the created row immediately

---

# Update Query – Update Profile

Example API:

```http
PATCH /users/:id
```

Example SQL:

```sql
UPDATE user_profiles
SET
  bio = :bio,
  phone = :phone
WHERE user_id = :userId
RETURNING *;
```

### Notes

Payload can be partial.

User may send:

```json
{
  "bio": "Updated bio"
}
```

or:

```json
{
  "phone": "9999999999"
}
```

Only provided fields are updated.

---

# `updated_at` Problem

When updating rows:

```text
created_at → stays the same
updated_at → should change every update
```

Two ways to handle it:

## 1. Manual update

```sql
updated_at = NOW()
```

inside every update query.

## 2. Database Trigger (preferred)

Automatically updates timestamp whenever a row changes.

---

# Database Indexes

Indexes are critical for query performance.

---

## Book Index Analogy

A book index tells you:

```text
Chapter 4 → Page 54
```

You jump directly to page 54 instead of scanning every page.

Database index works similarly.

Instead of scanning every row:

```text
Row 1 → check
Row 2 → check
Row 3 → check
...
```

Database can use index lookup and directly jump to the matching row.

---

# Why Indexes Improve Performance

Without index:

```text
Sequential scan across rows
```

With index:

```text
Lookup value → find disk location → fetch row
```

Much faster for large datasets.

Especially useful when table size becomes:

```text
100,000+
1,000,000+
```

---

# When to Create Indexes

Usually when a field is used in:

## 1. WHERE clause

Example:

```sql
WHERE status = 'pending'
```

---

## 2. JOIN condition

Example:

```sql
ON tasks.project_id = projects.id
```

---

## 3. ORDER BY sorting

Example:

```sql
ORDER BY created_at DESC
```

---

# Common Index Examples

## Users Table

### Email index

```sql
CREATE INDEX idx_users_email ON users(email);
```

Used for:

* lookup by email
* joins involving email

---

## Created At index

```sql
CREATE INDEX idx_users_created_at_desc
ON users(created_at DESC);
```

Used for:

```sql
ORDER BY created_at DESC
```

---

## Task Table – Project ID

```sql
CREATE INDEX idx_tasks_project_id
ON tasks(project_id);
```

Used when fetching tasks belonging to a project.

---

## Task Table – Assigned To

```sql
CREATE INDEX idx_tasks_assigned_to
ON tasks(assigned_to);
```

Used when fetching tasks assigned to a user.

---

## Task Table – Status

```sql
CREATE INDEX idx_tasks_status
ON tasks(status);
```

Used for filtering:

```sql
WHERE status = 'pending'
```

---

# Index Trade-Offs

Indexes improve read performance.

But they also add write overhead.

Because whenever you:

```text
INSERT
UPDATE
DELETE
```

Database must also maintain the index.

So every index has a cost.

---

## Rule of Thumb

Create indexes when:

* query runs frequently
* field appears in JOIN / WHERE / ORDER BY
* performance benefit is worth maintenance cost

Avoid unnecessary indexes.

---

# Database Triggers

Triggers run automatically when a database event happens.

Example event:

```text
UPDATE on table
```

Example action:

```text
Set updated_at = current timestamp
```

---

# Trigger Function Example

```sql
CREATE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

---

# Trigger Example

```sql
CREATE TRIGGER set_timestamp
BEFORE UPDATE ON users
FOR EACH ROW
EXECUTE FUNCTION update_updated_at_column();
```

Now every time a row updates:

```text
updated_at automatically changes
```

No manual query logic required.

---

# Migration Purpose

A migration can contain:

## Up Migration

* create indexes
* create trigger functions
* create triggers

## Down Migration

* drop indexes
* drop triggers
* drop functions

This keeps database schema version-controlled.

---

# Backend Engineering Workflow with Databases

Typical workflow:

```text
Design API
→ Understand request payload
→ Build SQL query
→ Add parameterized values safely
→ Execute query
→ Return JSON response
→ Optimize with indexes
→ Automate repeated DB behavior with triggers
```

---

# Key Takeaways

## Querying

* Use parameterized queries
* Avoid string concatenation for user input

## List APIs

Support:

* filtering
* sorting
* pagination

## Performance

Use indexes for:

* JOIN
* WHERE
* ORDER BY

## Automation

Use triggers for repetitive database behavior like:

```text
updated_at timestamps
```

## Migrations

Use migrations to:

* evolve schema safely
* add indexes
* create triggers
* rollback changes when needed

---

# Summary

This section covers practical backend database work:

* fetching a single user
* building dynamic list queries
* filtering
* sorting
* pagination
* insert queries
* update queries
* index optimization
* triggers
* migrations

These are the most common database operations backend engineers work with daily.
