# Transactions

---

## Table of Contents

1. [Transaction](#1-transaction)
2. [ACID Properties](#2-acid-properties)
   - 2.1 [Atomicity](#21-atomicity)
   - 2.2 [Consistency](#22-consistency)
   - 2.3 [Isolation](#23-isolation)
   - 2.4 [Durability](#24-durability)
3. [Transaction States](#3-transaction-states)
4. [Implementing Atomicity and Durability](#4-implementing-atomicity-and-durability)
   - 4.1 [Shadow-Copy Scheme](#41-shadow-copy-scheme)
   - 4.2 [Log-Based Recovery](#42-log-based-recovery)
     - 4.2.1 [Deferred Database Modification](#421-deferred-database-modification)
     - 4.2.2 [Immediate Database Modification](#422-immediate-database-modification)
     - 4.2.3 [Checkpoint Recovery](#423-checkpoint-recovery)

---

## 1. Transaction

A **transaction** is a unit of work done against the DB in a logical sequence. It consists of **one or more SQL statements**, and the sequence in which they run matters.

In a transaction, either **all** the statements get executed successfully and the changes are made permanent, or if a failure happens at any point, the transaction is **rolled back** to the state it was in before it started.

**Example — transferring money between two accounts:**

```sql
BEGIN TRANSACTION;

UPDATE Accounts SET balance = balance - 5000 WHERE acc_id = 'A101';
UPDATE Accounts SET balance = balance + 5000 WHERE acc_id = 'A102';

COMMIT;
```

If the system crashes after the first `UPDATE` but before the second, the transaction is rolled back — ₹5000 is put back into `A101` — instead of leaving the bank ₹5000 short.

---

## 2. ACID Properties

To ensure the integrity of data, the DB system must maintain the following four properties for every transaction:

| Property | One-line meaning |
|---|---|
| Atomicity | All or nothing |
| Consistency | Valid state → valid state |
| Isolation | Concurrent transactions don't interfere |
| Durability | Committed changes survive failures |

---

### 2.1 Atomicity

Either **all** the steps of a transaction are completed and reflected in the DB, or **none** of them are.

**Example:** Suppose there are two bank accounts. While transferring money, the amount is deducted from the first account, but before it is added to the second account, the system crashes. This creates inconsistency — to prevent it, the DB **rolls back** to the old state.

---

### 2.2 Consistency

The value of data in the DB must remain consistent before and after a transaction.

**Example:** If the total money in a bank is `x`, it must still be `x` after any number of transactions have taken place within the bank.

---

### 2.3 Isolation

Even though multiple transactions may appear to execute concurrently on the surface, the system ensures that — for every pair of transactions `Ti` and `Tj` — it appears to `Ti` that either `Tj` finished before `Ti` started, or `Tj` started only after `Ti` finished.

Multiple transactions can happen in the system **in isolation**, without interfering with each other.

---

### 2.4 Durability

Once a transaction has completed successfully, the changes it made to the DB must remain permanent even if there is a system failure right after.

This responsibility lies with the **recovery management component** — it maintains logs, and on failure, those logs are used to identify and re-apply any operations that hadn't been persisted yet.

---

## 3. Transaction States

| State | What happens |
|---|---|
| **Active** | All read/write operations are being performed. |
| **Partially Committed** | Transaction has finished executing; changes sit in a memory buffer, not yet permanent on disk. |
| **Committed** | Changes are made permanent on the DB. Rollback is no longer possible from here. |
| **Failed** | An error occurs during execution (or even after partial commit), so the transaction can't proceed. |
| **Aborted** | All buffered changes are reversed and the transaction fully rolls back; the DB returns to its pre-transaction state. |
| **Terminated** | The transaction has either committed or aborted. |

```
                 ┌───────────┐
        ┌───────►│  Active   │
        │        └─────┬─────┘
        │              │ executes without error
        │              ▼
        │     ┌──────────────────┐
        │     │ Partially        │
        │     │ Committed        │
        │     └───┬──────────┬───┘
        │  success │          │ failure
        │          ▼          ▼
        │   ┌────────────┐ ┌────────┐
        │   │ Committed  │ │ Failed │
        │   └─────┬──────┘ └───┬────┘
        │         │            │ rollback
        │         │            ▼
        │         │      ┌──────────┐
        │         └─────►│ Aborted  │
        │                └────┬─────┘
        │                     │
        └─────────────────────┘
                    ▼
              ┌────────────┐
              │ Terminated │
              └────────────┘
```

- **Active** — read/write operations are being performed. If they execute without error, the transaction moves to **Partially Committed**; if an error occurs, it moves to **Failed**.
- **Partially Committed** — after execution, changes are saved in a buffer in main memory. If they're made permanent on disk, the state becomes **Committed**; if a failure occurs, it becomes **Failed**.
- **Committed** — updates are permanent on the DB. Rollback cannot be done from this state.
- **Failed** — an error occurs during execution (this can also happen after partial commit).
- **Aborted** — once in the failed state, all buffered changes are reversed and the transaction rolls back completely, returning the DB to its pre-transaction state.
- **Terminated** — the transaction has either committed or aborted.

---

## 4. Implementing Atomicity and Durability

### 4.1 Shadow-Copy Scheme

This approach is based on making copies of the DB.

- Only **one transaction is active at a time** (assumption).
- A **db-pointer** is maintained on disk, always pointing to the current copy of the DB.
- A transaction that wants to update the DB first creates a **complete copy of the DB on disk**.
- All further updates are made on this **new copy**, leaving the original (shadow copy) untouched.
- If the transaction has to be aborted, the system simply **deletes the new copy** — the original is unaffected.
- If the transaction succeeds:
  1. The OS ensures all pages of the new copy are written to disk.
  2. The db-pointer is updated to point to the new copy.
  3. The new copy is now the current copy of the DB.
  4. The old copy is deleted.
- The transaction is said to be **committed** at the exact point the updated db-pointer is written to disk.

**How atomicity is attained:**
- If the transaction fails at any point before the db-pointer is updated, the old DB content is completely unaffected.
- So either all updates are reflected (pointer flips to the new copy) or none are (pointer never moves).

**How durability is attained:**
- If the system fails before the db-pointer is updated, restart reads the (unchanged) db-pointer and sees only the original DB — none of the transaction's effects are visible.
- If the system fails *after* the db-pointer has been updated, restart reads the new copy, and all committed effects are visible.
- This depends on the write to the db-pointer being **atomic**. Disks guarantee atomic writes to a single sector, so the db-pointer is stored entirely within one sector (typically at the start of a block) to exploit this.

**Disadvantage:** Very inefficient, since the entire DB is copied for every transaction.

> ### ✅ Correctness check — Shadow-Copy Scheme
> Your write-up is accurate on how the scheme works and on atomicity/durability — that part matches the standard textbook explanation closely and is correct.
>
> One small but important fix: you wrote that the transaction "first creates a complete copy of db **in the memory**" and that "if T is successful, the new db is **written to the disk**" — implying the shadow copy is built in RAM and only pushed to disk at commit time. That's not quite right. In the shadow-copy scheme, the **new copy is created directly on disk** from the very start (that's the whole point — it's a *page-level, disk-based* technique, not a memory-buffering technique). At commit, the OS just needs to make sure any pages still sitting in OS/DB buffers get **flushed to disk**, and then the db-pointer is atomically updated — the new copy isn't "written to disk" at that point, it's already there; only the pointer switch happens then. So replace "in the memory" → "on disk", and think of the commit step as "flush + pointer switch" rather than "write the new copy to disk."

---

### 4.2 Log-Based Recovery

A **log** is a sequence of records. The log of every transaction is maintained in **stable storage**, so that if a failure occurs, the DB can be recovered from it.

- If any operation is performed on the database, it is first recorded in the log.
- The log record for an operation must be written **before** the actual update is applied to the database (this is the Write-Ahead Logging principle).
- **Stable storage** is a storage technology that guarantees atomicity for any single write and is robust against hardware/power failures.

**Typical log record:**

| Field | Example |
|---|---|
| Transaction ID | T1 |
| Data item | A |
| Old value | 5000 |
| New value | 4000 |

---

#### 4.2.1 Deferred Database Modification

Atomicity is ensured by recording all DB modifications in the log, but **deferring** the actual write operations until the transaction's final action (commit) has been logged.

```
Read data into memory
        ↓
Perform computations in memory
        ↓
Before recording an intended DB update, write the corresponding log record
        ↓
Keep the actual DB pages unchanged
        ↓
Write the COMMIT record
        ↓
Apply the updates to the database
```

- The log information is used to actually execute the writes only **after** the transaction has completed.
- If the system crashes before the transaction completes, or if the transaction is aborted, the log entries for it are simply **ignored**.
- If the transaction completes, its log records are used to carry out the (deferred) writes.
- If a failure occurs while these deferred writes are being applied, the log is checked on recovery to see which updates were already reflected in the DB — any that weren't are **redone**.

**Example log timeline for T1 (transfer ₹1000 from A to B):**

```
<T1 start>
<T1, A, 4000>        -- new value of A after debit
<T1, B, 6000>         -- new value of B after credit
<T1 commit>
```
Only after `<T1 commit>` is written does the system actually apply `A = 4000` and `B = 6000` to the database.

---

#### 4.2.2 Immediate Database Modification

Here, DB modifications are output to the database **while the transaction is still active** — we don't wait for the whole transaction to finish before applying updates.

- Updates written by an active transaction are called **uncommitted modifications**.
- A DB update is applied only **after** its log record has been written to stable storage.
- On transaction **failure**: the system uses the **old value** field in the log to undo the changes (rollback).
- On transaction **completion followed by a system crash**: the system uses the **new value** field to redo the transaction, provided its commit record is present in the log.

**Example log timeline for T1:**

```
<T1 start>
<T1, A, 5000, 4000>   -- old value 5000, new value 4000
<T1, B, 5000, 6000>   -- old value 5000, new value 6000
<T1 commit>
```
If the crash happens *before* `<T1 commit>`, the system uses the old-value fields (5000, 5000) to undo. If it happens *after*, the system uses the new-value fields (4000, 6000) to redo.

---

#### 4.2.3 Checkpoint Recovery

A **checkpoint** marks a point in time where the DBMS guarantees the database is in a consistent state and all prior transactions have been committed. Checkpoints are taken **periodically**.

- Once a checkpoint is reached, all transaction log records **before** that point can be safely discarded.
- A new log file is started to record transactions from that point onward.
- This keeps the log file small and speeds up recovery, since the recovery process no longer needs to scan the **entire** log history — only the log since the last checkpoint.

> ### ✅ Correctness check — Log-Based Recovery
> All three sub-sections — **Deferred Modification**, **Immediate Modification**, and **Checkpoint Recovery** — are logically correct as written in your notes. In particular:
> - The WAL rule ("logs should be stored before the actual transaction is applied") is stated correctly and is the foundation the rest of the section depends on.
> - Deferred modification correctly ignores logs of incomplete/aborted transactions and only performs (redo) writes once a commit record exists — that's right.
> - Immediate modification correctly separates the **undo** case (use old value, for failed/uncommitted transactions) from the **redo** case (use new value, for committed transactions where the crash happened after commit but before all pages were flushed) — that's the correct rule.
> - Checkpointing is described correctly: it bounds how far back recovery needs to scan, and logs before a checkpoint can be discarded.
>
> No corrections needed here — this section is solid.
