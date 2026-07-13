# Normalisation

---

## Table of Contents

1. [Normalisation is a Step Towards DB Optimisation](#1-normalisation-is-a-step-towards-db-optimisation)
2. [Functional Dependency (FD)](#2-functional-dependency-fd)
   - 2.1 [Types of FD](#21-types-of-fd)
   - 2.2 [Rules of FD (Armstrong's Axioms)](#22-rules-of-fd-armstrongs-axioms)
3. [Why Normalisation?](#3-why-normalisation)
4. [Anomalies](#4-anomalies)
   - 4.1 [Insertion Anomaly](#41-insertion-anomaly)
   - 4.2 [Deletion Anomaly](#42-deletion-anomaly)
   - 4.3 [Updation Anomaly](#43-updation-anomaly-or-modification-anomaly)
5. [What is Normalisation?](#5-what-is-normalisation)
6. [Normal Forms](#6-normal-forms)
   - 6.1 [1NF – First Normal Form](#61-1nf--first-normal-form)
   - 6.2 [2NF – Second Normal Form](#62-2nf--second-normal-form)
   - 6.3 [3NF – Third Normal Form](#63-3nf--third-normal-form)
   - 6.4 [BCNF – Boyce-Codd Normal Form](#64-bcnf--boyce-codd-normal-form)
7. [Advantages of Normalisation](#7-advantages-of-normalisation)

---

## 1. Normalisation is a Step Towards DB Optimisation

Normalisation is a **database optimisation technique**. It is used to organise the data in a database efficiently so that redundancy is minimised and the overall performance and consistency of the database improves.

---

## 2. Functional Dependency (FD)

**Functional dependency** is when an attribute of a table can be determined by another attribute or attributes of the table.

- It is a relationship between the primary key (or candidate key) of a relation and the other attributes of that same relation — usually the primary key attribute determines the other attributes.
- **Notation:** X → Y, which means if I have the value of X, I can determine the value of Y.
- The left side of the FD (X) is known as the **Determinant**.
- The right side of the FD (Y) is known as the **Dependent**.

**Example:**

| roll_no | name  | branch |
|---------|-------|--------|
| 1       | Ravi  | CSE    |
| 2       | Aman  | ECE    |

Here, `roll_no → name` and `roll_no → branch`, since knowing the `roll_no` lets us uniquely determine the `name` and the `branch`.

---

### 2.1 Types of FD

**Trivial FD**

- A → B has a trivial functional dependency if B is a subset of A.
- A → A and B → B are also trivial FDs.

**Non-trivial FD**

- A → B has a non-trivial functional dependency if B is **not** a subset of A (i.e., A intersection B is NULL).

---

### 2.2 Rules of FD (Armstrong's Axioms)

**Reflexive**

- If A is a set of attributes and B is a subset of A, then A → B holds.
- In other words, if A ⊇ B, then A → B.

**Augmentation**

- If B can be determined from A, then adding an attribute to this functional dependency won't change anything — the FD will still remain true.
- If A → B holds, then AX → BX holds too, where X is a set of attributes.

**Transitivity**

- If A determines B, and B determines C, then we can say A determines C.
- If A → B and B → C, then A → C.

---

## 3. Why Normalisation?

To avoid redundancy in the database — i.e., to avoid storing redundant (duplicate) data.

Redundant data causes **insertion, deletion, and updation anomalies**, which in turn increase the size of the database and slow down its performance. To rectify these anomalies and their effects, we use a database optimisation technique called **Normalisation**.

---

## 4. Anomalies

Anomalies mean abnormalities. There are three types of anomalies introduced by data redundancy.

**Example (unnormalised table):**

| student_id | student_name | course_id | course_name | instructor    |
|------------|---------------|-----------|--------------|---------------|
| 1          | Aman          | C101      | DBMS         | Dr. Sharma    |
| 1          | Aman          | C102      | OS           | Dr. Verma     |
| 2          | Riya          | C101      | DBMS         | Dr. Sharma    |

This single table mixes student, course, and instructor information together — which is where the anomalies below come from.

### 4.1 Insertion Anomaly

When certain data (an attribute) cannot be inserted into the DB without the presence of other data.

- In the table above, we cannot add a new course (say `C103 – Networks`) unless at least one student has enrolled in it, because `course_id` and `course_name` only exist as part of a student's row.

### 4.2 Deletion Anomaly

The delete anomaly refers to the situation where the deletion of data results in the unintended loss of some other important data.

- If Riya (the only student enrolled in `C101`... in this example, if she were the only one) is deleted, we could unintentionally lose all record of the `DBMS` course and its instructor `Dr. Sharma` as well.

### 4.3 Updation Anomaly (or Modification Anomaly)

The update anomaly is when an update of a single data value requires multiple rows of data to be updated.

- If `Dr. Sharma` is renamed or reassigned, every row containing `C101` must be updated. Due to updates being required in many places, data inconsistency can arise if one forgets to update the data at all the intended places.

Due to these anomalies, DB size increases and DB performance becomes very slow.

---

## 5. What is Normalisation?

- Normalisation is used to minimise the redundancy from a relation. It is also used to eliminate undesirable characteristics like Insertion, Update, and Deletion anomalies.
- Normalisation divides composite/larger tables into smaller tables and links them using relationships (foreign keys), decomposing a table until each table follows the **Single Responsibility Principle (SRP)** — i.e., one table represents one single idea/entity.
- The different stages of this decomposition process are called **Normal Forms**, and they are used to reduce redundancy from the database tables.

---

## 6. Normal Forms

### 6.1 1NF – First Normal Form

- Every relation cell must have an **atomic value** (a value that cannot be divided further).
- The relation must **not** have multi-valued attributes — one column shouldn't contain more than one value.

**Example — table that violates 1NF:**

| student_id | student_name | subjects           |
|------------|---------------|---------------------|
| 1          | Aman          | DBMS, OS, Networks  |
| 2          | Riya          | DBMS, DSA           |

The `subjects` column holds multiple values, so this is **not** in 1NF.

**Converted to 1NF** (each value made atomic):

| student_id | student_name | subject   |
|------------|---------------|-----------|
| 1          | Aman          | DBMS      |
| 1          | Aman          | OS        |
| 1          | Aman          | Networks  |
| 2          | Riya          | DBMS      |
| 2          | Riya          | DSA       |

---

### 6.2 2NF – Second Normal Form

- Relation must be in 1NF.
- There should not be any **partial dependency**:
  - All non-prime attributes must be fully dependent on the primary key.
  - A non-prime attribute cannot depend on just a part of the primary key.

**Example — table with partial dependency:**

Suppose the primary key is composite: `(student_id, course_id)`.

| student_id | course_id | student_name | course_name |
|------------|-----------|---------------|--------------|
| 1          | C101      | Aman          | DBMS         |
| 1          | C102      | Aman          | OS           |
| 2          | C101      | Riya          | DBMS         |

Here, `student_name` only depends on `student_id` (part of the key), and `course_name` only depends on `course_id` (part of the key) — this is partial dependency, so the table violates 2NF.

**Converted to 2NF** (split so each non-prime attribute depends on the full key it actually belongs to):

`Students` table:

| student_id | student_name |
|------------|---------------|
| 1          | Aman          |
| 2          | Riya          |

`Courses` table:

| course_id | course_name |
|-----------|--------------|
| C101      | DBMS         |
| C102      | OS           |

`Enrollment` table:

| student_id | course_id |
|------------|-----------|
| 1          | C101      |
| 1          | C102      |
| 2          | C101      |

---

### 6.3 3NF – Third Normal Form

- Relation must be in 2NF.
- No **transitive dependency** should exist — a non-prime attribute should not determine another non-prime attribute.

**Example:** Suppose we have a table with 3 columns — `A` (primary key), `B`, `C` — where `A → B` and `B → C`. Here, `B → C` is a transitive dependency (since `C` depends on `B`, a non-prime attribute, rather than directly on the primary key `A`).

| emp_id (A) | dept_id (B) | dept_name (C) |
|------------|--------------|-----------------|
| 1          | D01          | Sales           |
| 2          | D02          | HR              |
| 3          | D01          | Sales           |

`emp_id → dept_id` and `dept_id → dept_name`, so `emp_id` determines `dept_name` only transitively through `dept_id`.

**Converted to 3NF** (split into two tables so `A` is the PK of one and `B` is the PK of the other, removing the transitive dependency):

`Employee` table:

| emp_id | dept_id |
|--------|---------|
| 1      | D01     |
| 2      | D02     |
| 3      | D01     |

`Department` table:

| dept_id | dept_name |
|---------|-----------|
| D01     | Sales     |
| D02     | HR        |

---

### 6.4 BCNF – Boyce-Codd Normal Form

- Relation must be in 3NF.
- For every functional dependency A → B, **A must be a super key**.
- We must not be able to derive a prime attribute from any prime or non-prime attribute.

**Example:** Consider a table with 3 columns — `student_id`, `subject`, `professor`.

- One student can enroll in multiple subjects, and for each subject a professor is assigned to that student.
- Multiple professors can teach a single subject, but one professor can only teach one subject.

| student_id | subject | professor  |
|------------|---------|------------|
| 1          | DBMS    | Dr. Sharma |
| 1          | OS      | Dr. Verma  |
| 2          | DBMS    | Dr. Sharma |

- The primary key here is the combination `(student_id, subject)`.
- From the primary key, we can find the `professor` — that's expected.
- But we can also go the other way: from `professor`, we can find the `subject` (since one professor teaches only one subject). This means we are deriving a prime attribute (`subject`) from a non-prime attribute (`professor`) — which violates BCNF, since `professor` alone is not a super key but still determines `subject`.

**Converted to BCNF** (split into two tables):

`Student_Professor` table:

| student_id | prof_id |
|------------|---------|
| 1          | P01     |
| 1          | P02     |
| 2          | P01     |

`Professor_Subject` table:

| prof_id | professor  | subject |
|---------|------------|---------|
| P01     | Dr. Sharma | DBMS    |
| P02     | Dr. Verma  | OS      |

---

## 7. Advantages of Normalisation

- Normalisation helps to minimise data redundancy.
- Greater overall database organisation.
- Data consistency is maintained in the DB.
