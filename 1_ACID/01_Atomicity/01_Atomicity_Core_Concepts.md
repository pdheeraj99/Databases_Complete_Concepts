# Atomicity: The "All or Nothing" Rule ⚛️

## 1. What is Atomicity?

Atomicity (ACID lo First Letter 'A') ante simple ga: **"Anni jaragali, leda emi jaragakudadu."**

* **Definition:** A transaction is treated as a **single unit of work**.
* **The Rule:** Transaction lo 100 queries unna sare, andulo **Okka Query Fail** ayina, motham Transaction ni **Rollback** (Undo) cheseyali.
* **Analogy:** Science lo Atom ni ela aithe split cheyyalemo, Database lo Transaction ni kuda split cheyyakudadu.

---

## 2. Why Do We Need It?

Manam real-world lo multiple tasks ni okate pani ga chestam.

* **Example:** Bank Transfer (Debit Sender + Credit Receiver).
* Ivi rendu kalisi **Okkate Transaction**.
* Madhyalo edaina problem vachi, sagam pani jarigi, migatha sagam aagipothe... **Data Inconsistent** ga maripothundi (Ex: Money gayab avvadam).

---

## 3. Two Types of Failures

Database Atomicity ni guarantee cheyyalsina situations rendu unnayi:

| Failure Type | Description | Recovery |
|:---|:---|:---|
| **Program Failure** | Syntax error, Constraint fail (Ex: Balance negative) | Immediate Rollback |
| **System Crash** | Power failure, OS crash mid-transaction | Auto Rollback on Restart |

---

## 4. SQL Transaction Example 💻

```sql
BEGIN TRANSACTION;
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;  -- Debit
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;  -- Credit
COMMIT;  -- Only if BOTH succeed

-- If any fails:
ROLLBACK;  -- Undo everything
```

---

## 🎯 Key Takeaways

1. **All or Nothing** - Partial execution is never allowed
2. **Automatic Cleanup** - DB handles rollback on crash
3. **Commit = Guarantee** - Changes are permanent only after COMMIT
4. **WAL Enables This** - Write Ahead Log stores undo information
