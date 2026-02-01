# 👻 Phantom Reads - Complete Practical Demo

> **Read Phenomenon** | Concurrent Transactions | MySQL & PostgreSQL Comparison

---

## 📚 What are Read Phenomena?

Database lo concurrent transactions run ayinappudu **3 types of read problems** occur avvachu:

| Phenomenon | Description | Telugu Meaning |
|------------|-------------|----------------|
| **Dirty Read** | Reading uncommitted data from another transaction | Commit avvani data chadavdam |
| **Non-Repeatable Read** | Same row returns different values on re-read | Same row ki different values |
| **Phantom Read** | New rows appear/disappear in range queries | Kotta rows sudden ga appear avvadam |

---

## 👻 Phantom Read - Detailed Explanation

### 🎯 Definition
>
> **Phantom Read** occurs when a transaction re-executes a **range query** and gets **different rows** because another transaction **inserted/deleted** rows in that range.

### 📖 Real-World Example

```
Transaction 1 (Report Generator):
┌─────────────────────────────────────────────────────────┐
│ Step 1: SELECT COUNT(*) FROM sales WHERE date = TODAY  │
│         Result: 5 sales                                 │
│                                                         │
│ Step 2: Doing some processing...                        │
│                                                         │
│ Step 3: SELECT SUM(price) FROM sales WHERE date = TODAY │
│         Result: Includes 6 sales! 😱                    │
│         (Someone inserted a new sale meanwhile)         │
└─────────────────────────────────────────────────────────┘

Transaction 2 (Sale Maker):
┌─────────────────────────────────────────────────────────┐
│ INSERT INTO sales VALUES (6, 100, TODAY);               │
│ COMMIT;                                                 │
└─────────────────────────────────────────────────────────┘
```

> ⚠️ **Problem**: Report shows inconsistent data - count says 5, but sum includes 6 rows!

---

## 🚀 Setup - MySQL Container

### Terminal 1 - Start MySQL Container

```bash
# Run MySQL container
docker container run \
    --name mysql_dummy \
    --publish 3306:3306 \
    --env MYSQL_ROOT_PASSWORD=root \
    --detach \
    mysql:latest

# Verify container is running
docker ps

# Enter MySQL shell
docker exec -it mysql_dummy mysql -u root -proot
```

---

## 🛠️ Database Setup

### Terminal 1 - Create Test Data

```sql
-- Create database
CREATE DATABASE phantom_demo;
USE phantom_demo;

-- Create sales table
CREATE TABLE sales (
    sale_id INT AUTO_INCREMENT PRIMARY KEY,
    pid INT,
    price DECIMAL(10,2),
    sale_date DATE
);

-- Insert sample data
INSERT INTO sales (pid, price, sale_date) VALUES
(1, 10.00, '2021-02-05'),
(1, 10.00, '2021-02-06'),
(1, 10.00, '2021-02-07'),
(1, 5.00, '2021-02-07'),
(1, 5.00, '2021-02-08');

-- Verify
SELECT * FROM sales;
```

**Expected Output:**

```
+---------+------+-------+------------+
| sale_id | pid  | price | sale_date  |
+---------+------+-------+------------+
|       1 |    1 | 10.00 | 2021-02-05 |
|       2 |    1 | 10.00 | 2021-02-06 |
|       3 |    1 | 10.00 | 2021-02-07 |
|       4 |    1 |  5.00 | 2021-02-07 |
|       5 |    1 |  5.00 | 2021-02-08 |
+---------+------+-------+------------+
5 rows
```

---

## 🔴 DEMO 1: Phantom Read Problem (Default Isolation)

### Open Terminal 2

```bash
docker exec -it mysql_dummy mysql -u root -proot
USE phantom_demo;
```

---

### Step-by-Step Execution

| Step | Terminal 1 (Report Generator) | Terminal 2 (Sale Maker) |
|------|-------------------------------|-------------------------|
| 1 | `START TRANSACTION;` | - |
| 2 | `SELECT pid, SUM(price) FROM sales GROUP BY pid;` | - |
| 3 | - | `INSERT INTO sales (pid, price, sale_date) VALUES (1, 15.00, '2021-02-07');` |
| 4 | `SELECT pid, SUM(price) FROM sales GROUP BY pid;` | - |
| 5 | `SELECT * FROM sales;` | - |
| 6 | `COMMIT;` | - |

---

### 📝 Detailed Commands

#### Terminal 1 - Start Transaction & Query

```sql
START TRANSACTION;

-- First query: Sum of sales per product
SELECT pid, SUM(price) FROM sales GROUP BY pid;
```

**Output (Step 2):**

```
+------+------------+
| pid  | SUM(price) |
+------+------------+
|    1 |      40.00 |  ← Total $40
+------+------------+
```

#### Terminal 2 - Insert New Sale (Auto-commits by default)

```sql
INSERT INTO sales (pid, price, sale_date) VALUES (1, 15.00, '2021-02-07');
```

#### Terminal 1 - Query Again (Same Transaction)

```sql
-- Second query: Same sum query
SELECT pid, SUM(price) FROM sales GROUP BY pid;
```

**Output (Step 4) - MySQL REPEATABLE READ:**

```
+------+------------+
| pid  | SUM(price) |
+------+------------+
|    1 |      40.00 |  ← Still $40! MySQL REPEATABLE READ protects us!
+------+------------+
```

> 💡 **MySQL's Default**: `REPEATABLE READ` prevents phantom reads using **gap locking**!

---

## 🔄 MySQL vs PostgreSQL Comparison

```sql
-- Check current isolation level
-- MySQL:
SELECT @@transaction_isolation;

-- PostgreSQL:
SHOW transaction_isolation;
```

| Database | Default Isolation | Phantom Read Protection |
|----------|------------------|------------------------|
| **MySQL** | REPEATABLE READ | ✅ Protected (Gap Locks) |
| **PostgreSQL** | READ COMMITTED | ❌ Not Protected |
| **Oracle** | READ COMMITTED | ❌ Not Protected |
| **SQL Server** | READ COMMITTED | ❌ Not Protected |

---

## 🔴 DEMO 2: Force Phantom Read in MySQL (READ COMMITTED)

### Terminal 1 - Use READ COMMITTED

```sql
-- Rollback any existing transaction
ROLLBACK;

-- Set session isolation level to READ COMMITTED
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- Verify
SELECT @@transaction_isolation;
-- Output: READ-COMMITTED

-- Start transaction
START TRANSACTION;

-- First query
SELECT pid, SUM(price) FROM sales GROUP BY pid;
```

**Output:**

```
+------+------------+
| pid  | SUM(price) |
+------+------------+
|    1 |      55.00 |  ← Current total
+------+------------+
```

### Terminal 2 - Insert Another Sale

```sql
INSERT INTO sales (pid, price, sale_date) VALUES (1, 20.00, '2021-02-07');
```

### Terminal 1 - Query Again

```sql
-- Same query in same transaction
SELECT pid, SUM(price) FROM sales GROUP BY pid;
```

**Output - PHANTOM READ OCCURS!:**

```
+------+------------+
| pid  | SUM(price) |
+------+------------+
|    1 |      75.00 |  ← Changed to $75! 👻 PHANTOM READ!
+------+------------+
```

> ⚠️ **Phantom Read Occurred!** In `READ COMMITTED`, new rows are visible immediately!

---

## 🔐 DEMO 3: SERIALIZABLE Isolation (Maximum Protection)

### Terminal 1 - Use SERIALIZABLE

```sql
ROLLBACK;

-- Set session isolation level to SERIALIZABLE
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Start transaction
START TRANSACTION;

-- Query
SELECT pid, SUM(price) FROM sales GROUP BY pid;
```

**Output:**

```
+------+------------+
| pid  | SUM(price) |
+------+------------+
|    1 |      75.00 |
+------+------------+
```

### Terminal 2 - Try to Insert

```sql
INSERT INTO sales (pid, price, sale_date) VALUES (1, 25.00, '2021-02-07');
-- This will WAIT (block) until Terminal 1 commits!
```

> 💡 **SERIALIZABLE blocks concurrent inserts** that might cause phantom reads!

### Terminal 1 - Query Again

```sql
SELECT pid, SUM(price) FROM sales GROUP BY pid;
-- Still shows 75.00 - No phantom read!

COMMIT;
-- Now Terminal 2's insert will complete
```

---

## 📊 Isolation Levels & Read Phenomena Matrix

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|-----------------|------------|---------------------|--------------|
| **READ UNCOMMITTED** | ✅ Possible | ✅ Possible | ✅ Possible |
| **READ COMMITTED** | ❌ Prevented | ✅ Possible | ✅ Possible |
| **REPEATABLE READ** | ❌ Prevented | ❌ Prevented | ⚠️ DB Dependent |
| **SERIALIZABLE** | ❌ Prevented | ❌ Prevented | ❌ Prevented |

### REPEATABLE READ - Database Specific Behavior

| Database | Phantom Read in REPEATABLE READ |
|----------|--------------------------------|
| **PostgreSQL** | ❌ Prevented (MVCC Snapshots) |
| **MySQL InnoDB** | ❌ Prevented (Gap Locks) |
| **Oracle** | ✅ Possible |
| **SQL Server** | ✅ Possible |

---

## 🎯 When to Use Which Isolation Level?

| Use Case | Recommended Level | Reason |
|----------|------------------|--------|
| **General CRUD** | READ COMMITTED | Good balance of consistency & performance |
| **Reports/Analytics** | REPEATABLE READ | Consistent snapshot for entire report |
| **Financial Transactions** | SERIALIZABLE | Maximum data integrity |
| **High Concurrency** | READ COMMITTED | Minimal locking, better throughput |

---

## 🔑 Quick Reference Commands

### MySQL

```sql
-- Check current isolation level
SELECT @@transaction_isolation;

-- Set for session
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Set globally (all new connections)
SET GLOBAL TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

### PostgreSQL

```sql
-- Check current isolation level
SHOW transaction_isolation;

-- Start transaction with specific level
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Set default for session
SET default_transaction_isolation = 'repeatable read';
```

---

## 🧹 Cleanup

```bash
# Stop and remove container
docker stop mysql_dummy
docker rm mysql_dummy
```

---

## 📌 Key Takeaways

1. **Phantom Read** = New rows appearing/disappearing in range queries within same transaction
2. **MySQL's REPEATABLE READ** prevents phantom reads using **gap locks** (unique feature!)
3. **PostgreSQL's REPEATABLE READ** prevents phantom reads using **MVCC snapshots**
4. **Oracle/SQL Server** need **SERIALIZABLE** to prevent phantom reads
5. **SERIALIZABLE** is safest but **slowest** due to locking overhead
6. Choose isolation level based on your **consistency vs performance** requirements

---

> 💡 **Pro Tip**: Production applications lo mostly `READ COMMITTED` use chestaru. `SERIALIZABLE` only for critical financial operations where data integrity is paramount.
