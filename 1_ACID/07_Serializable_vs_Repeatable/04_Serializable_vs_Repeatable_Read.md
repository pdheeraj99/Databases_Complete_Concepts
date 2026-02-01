# 🔄 Serializable vs Repeatable Read - Deep Dive

> **Isolation Level Comparison** | When to Use Which | Practical Demo

---

## 🎯 The Core Question

> **Why would you pick SERIALIZABLE over REPEATABLE READ?**

Both prevent dirty reads and non-repeatable reads, but they handle **concurrent write dependencies** very differently!

---

## 📊 Visual Example - The Swap Problem

### Initial State

```
┌─────────────┐
│   Table     │
├─────────────┤
│     A       │
│     A       │
│     B       │
│     B       │
└─────────────┘
```

### Two Concurrent Transactions

```
Transaction 1 (T1):              Transaction 2 (T2):
UPDATE table                     UPDATE table
SET field = 'B'                  SET field = 'A'
WHERE field = 'A';               WHERE field = 'B';

(Changes A → B)                  (Changes B → A)
```

---

## 🔴 REPEATABLE READ Behavior

```
┌──────────────────────────────────────────────────────────────────┐
│                     REPEATABLE READ                               │
├──────────────────────────────────────────────────────────────────┤
│  Initial: [A, A, B, B]                                           │
│                                                                   │
│  T1 starts → Sees: [A, A, B, B]                                  │
│  T2 starts → Sees: [A, A, B, B]                                  │
│                                                                   │
│  T1 executes: Changes A→B on rows 1,2                            │
│  T2 executes: Changes B→A on rows 3,4                            │
│                                                                   │
│  T1 commits ✅                                                    │
│  T2 commits ✅                                                    │
│                                                                   │
│  Final Result: [B, B, A, A]  ← SWAPPED! 😱                       │
└──────────────────────────────────────────────────────────────────┘
```

> ⚠️ **Problem**: Both transactions succeeded because they touched **different rows**. But the result is a complete swap - maybe NOT what you wanted!

---

## 🔐 SERIALIZABLE Behavior

```
┌──────────────────────────────────────────────────────────────────┐
│                     SERIALIZABLE                                  │
├──────────────────────────────────────────────────────────────────┤
│  Initial: [A, A, B, B]                                           │
│                                                                   │
│  T1 starts → Sees: [A, A, B, B]                                  │
│  T2 starts → Sees: [A, A, B, B]                                  │
│                                                                   │
│  T1 executes: Changes A→B on rows 1,2                            │
│  T2 executes: Changes B→A on rows 3,4                            │
│                                                                   │
│  T1 commits ✅                                                    │
│  T2 commits ❌ ERROR!                                             │
│                                                                   │
│  Error: "Could not serialize access due to                       │
│          read/write dependencies among transactions"             │
│                                                                   │
│  Final Result: [B, B, B, B]  ← Only T1's changes! ✅             │
└──────────────────────────────────────────────────────────────────┘
```

> ✅ **Solution**: Database detects the read/write dependency and **rejects** one transaction. You must **retry** the failed transaction.

---

## 🚀 Practical Demo - PostgreSQL

### Setup

```bash
# Start PostgreSQL container
docker run --name pg-serial -e POSTGRES_PASSWORD=postgres -d postgres:13

# Enter container
docker exec -it pg-serial psql -U postgres
```

```sql
-- Create test table
CREATE TABLE test (t CHAR(1));

-- Insert initial data
INSERT INTO test VALUES ('A'), ('A'), ('B'), ('B');

-- Verify
SELECT * FROM test;
```

**Output:**

```
 t 
---
 A
 A
 B
 B
```

---

## 🔴 Demo 1: REPEATABLE READ - The Swap Problem

### Open Two Terminals

```bash
# Terminal 2
docker exec -it pg-serial psql -U postgres
```

### Step-by-Step Execution

| Step | Terminal 1 | Terminal 2 |
|------|------------|------------|
| 1 | `BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;` | - |
| 2 | - | `BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;` |
| 3 | `UPDATE test SET t = 'B' WHERE t = 'A';` | - |
| 4 | - | `UPDATE test SET t = 'A' WHERE t = 'B';` |
| 5 | `SELECT * FROM test;` | `SELECT * FROM test;` |
| 6 | `COMMIT;` | - |
| 7 | - | `COMMIT;` |
| 8 | `SELECT * FROM test;` | - |

---

### 📝 Detailed Commands

#### Terminal 1

```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Change all A's to B's
UPDATE test SET t = 'B' WHERE t = 'A';

-- Check (inside transaction)
SELECT * FROM test;
-- Output: B, B, B, B (in my view)

COMMIT;
```

#### Terminal 2

```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Change all B's to A's
UPDATE test SET t = 'A' WHERE t = 'B';

-- Check (inside transaction)
SELECT * FROM test;
-- Output: A, A, A, A (in my view)

COMMIT;
```

#### Final Check (Any Terminal)

```sql
SELECT * FROM test;
```

**Output - SWAPPED!:**

```
 t 
---
 A
 B
 B
 A
```

> ⚠️ **Result**: Data got swapped! T1 changed rows 1,2. T2 changed rows 3,4. No conflict detected!

---

## 🔐 Demo 2: SERIALIZABLE - Dependency Detection

### Reset Data First

```sql
-- Reset table
DELETE FROM test;
INSERT INTO test VALUES ('A'), ('A'), ('B'), ('B');
SELECT * FROM test;
```

### Step-by-Step Execution

| Step | Terminal 1 | Terminal 2 |
|------|------------|------------|
| 1 | `BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;` | - |
| 2 | - | `BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;` |
| 3 | `UPDATE test SET t = 'B' WHERE t = 'A';` | - |
| 4 | - | `UPDATE test SET t = 'A' WHERE t = 'B';` |
| 5 | `COMMIT;` | - |
| 6 | - | `COMMIT;` ← **ERROR!** |

---

### 📝 Detailed Commands

#### Terminal 1

```sql
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Change all A's to B's
UPDATE test SET t = 'B' WHERE t = 'A';

-- Commit successfully
COMMIT;
```

**Output:**

```
COMMIT
```

#### Terminal 2

```sql
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Change all B's to A's
UPDATE test SET t = 'A' WHERE t = 'B';

-- Try to commit
COMMIT;
```

**Output - ERROR!:**

```
ERROR:  could not serialize access due to read/write dependencies among transactions
DETAIL:  Reason code: Canceled on identification as a pivot, during commit attempt.
HINT:  The transaction might succeed if retried.
```

> ✅ **Serializable detected the dependency and rejected T2!**

#### Final Check

```sql
SELECT * FROM test;
```

**Output:**

```
 t 
---
 B
 B
 B
 B
```

> ✅ **Only T1's changes applied!** Data is consistent.

---

## 🔄 Retry Pattern for SERIALIZABLE

When using SERIALIZABLE, you **MUST** handle failures with retry logic:

### Application Code Pattern (Pseudo-code)

```python
max_retries = 3
retry_count = 0

while retry_count < max_retries:
    try:
        begin_transaction(isolation_level='SERIALIZABLE')
        
        # Your business logic here
        update_data()
        
        commit()
        break  # Success, exit loop
        
    except SerializationError:
        rollback()
        retry_count += 1
        time.sleep(random_backoff())  # Wait before retry
        
if retry_count == max_retries:
    raise Exception("Transaction failed after max retries")
```

### Java/Spring Example

```java
@Transactional(isolation = Isolation.SERIALIZABLE)
@Retryable(value = SerializationException.class, maxAttempts = 3)
public void updateData() {
    // Your business logic
}
```

---

## 📊 Comparison Table

| Aspect | REPEATABLE READ | SERIALIZABLE |
|--------|-----------------|--------------|
| **Dirty Read** | ❌ Prevented | ❌ Prevented |
| **Non-Repeatable Read** | ❌ Prevented | ❌ Prevented |
| **Phantom Read** | ⚠️ DB Dependent | ❌ Prevented |
| **Write Skew** | ✅ Possible | ❌ Prevented |
| **Performance** | 🟢 Good | 🟡 Slower |
| **Failure Rate** | 🟢 Low | 🟡 Higher |
| **Retry Required** | Usually No | Yes, Always |

---

## 🎯 Write Skew Anomaly Explained

**Write Skew** = When two transactions read overlapping data, then make decisions based on what they read, then write non-overlapping data.

### Example: Doctor On-Call System

```
Rule: At least 1 doctor must be on call
Current State: [Dr. Alice: ON, Dr. Bob: ON]

T1 (Alice wants to leave):            T2 (Bob wants to leave):
1. SELECT COUNT(*) WHERE on_call      1. SELECT COUNT(*) WHERE on_call
   → Result: 2 doctors                    → Result: 2 doctors
2. "2 doctors on call, safe to leave" 2. "2 doctors on call, safe to leave"
3. UPDATE SET on_call=false           3. UPDATE SET on_call=false
   WHERE name='Alice'                    WHERE name='Bob'
4. COMMIT ✅                           4. COMMIT ✅

Final State: [Dr. Alice: OFF, Dr. Bob: OFF]
⚠️ NO DOCTORS ON CALL! Rule violated!
```

> **REPEATABLE READ** allows this! **SERIALIZABLE** would reject one transaction.

---

## 🔑 When to Use Which?

| Use Case | Recommended Level | Why |
|----------|------------------|-----|
| **General CRUD** | READ COMMITTED | Good performance, sufficient for most cases |
| **Reports/Analytics** | REPEATABLE READ | Consistent snapshot, no read anomalies |
| **Financial Transfers** | SERIALIZABLE | Must prevent all anomalies |
| **Inventory Management** | SERIALIZABLE | Prevent overselling |
| **Constraint Enforcement** | SERIALIZABLE | Prevent write skew |
| **High-Concurrency** | READ COMMITTED + Explicit Locks | Better control |

---

## 🔧 Alternative: Pessimistic Locking

Instead of SERIALIZABLE, you can use explicit locks:

```sql
-- Lock rows explicitly (READ COMMITTED level)
BEGIN;

SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
-- ↑ This locks the row until commit

UPDATE accounts SET balance = balance - 100 WHERE id = 1;

COMMIT;
```

### FOR UPDATE vs SERIALIZABLE

| Aspect | FOR UPDATE | SERIALIZABLE |
|--------|------------|--------------|
| **Lock Type** | Explicit, row-level | Implicit, predicate |
| **Blocking** | Other transactions wait | Fail fast, retry |
| **Performance** | Predictable | Variable |
| **Code Complexity** | Higher (must add locks) | Lower (automatic) |

---

## 📌 Key Takeaways

1. **REPEATABLE READ** prevents re-reading different values but allows **write skew**
2. **SERIALIZABLE** detects **read/write dependencies** and rejects conflicting transactions
3. **With SERIALIZABLE, you MUST implement retry logic** in your application
4. **PostgreSQL's SERIALIZABLE** uses **SSI (Serializable Snapshot Isolation)** - very efficient!
5. **Choose based on your consistency requirements** vs performance trade-offs

---

## 🧹 Cleanup

```bash
docker stop pg-serial
docker rm pg-serial
```

---

> 💡 **Pro Tip**: Use SERIALIZABLE for **critical business logic** where data integrity is paramount. For everything else, REPEATABLE READ or READ COMMITTED with explicit locking gives you better control and performance.
