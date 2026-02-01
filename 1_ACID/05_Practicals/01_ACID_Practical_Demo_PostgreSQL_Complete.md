# 🔥 ACID Practical Demonstration - PostgreSQL

> **Complete Hands-on Guide** | Two Terminal Setup | Step-by-Step Commands

---

## 📋 Prerequisites

```bash
# Docker installed undali
docker --version
```

---

## 🚀 STEP 1: PostgreSQL Container Setup

### Terminal 1 - Container Start Cheyydam

```bash
# PostgreSQL container start
docker run --name pg-acid -e POSTGRES_PASSWORD=postgres -d postgres:13

# Container running check
docker ps

# Container lo enter avvadam
docker exec -it pg-acid psql -U postgres
```

> 💡 **Note**: `pg-acid` is our container name. `-d` means detached mode (background lo run avtundi)

---

## 🛠️ STEP 2: Tables Create Cheyydam

### Terminal 1 - Database Setup

```sql
-- Products table create
CREATE TABLE products (
    pid SERIAL PRIMARY KEY,
    name TEXT,
    price FLOAT,
    inventory INTEGER
);

-- Sales table create
CREATE TABLE sales (
    sale_id SERIAL PRIMARY KEY,
    pid INTEGER,
    price FLOAT,
    quantity INTEGER
);

-- Initial product insert
INSERT INTO products (name, price, inventory) VALUES ('Phone', 999.99, 100);

-- Verify
SELECT * FROM products;
```

**Expected Output:**

```
 pid | name  | price  | inventory 
-----+-------+--------+-----------
   1 | Phone | 999.99 |       100
```

---

## ⚛️ STEP 3: ATOMICITY Demo

### 🎯 Concept: Atomicity ante enti?
>
> **All or Nothing** - Transaction lo unna anni operations successful avvali, lekapothe anni rollback avvali

### Terminal 1 - Crash Simulation

```sql
-- Transaction start
BEGIN;

-- Inventory update (10 phones sold)
UPDATE products SET inventory = inventory - 10 WHERE pid = 1;

-- Check inside transaction
SELECT * FROM products;
-- Output: inventory = 90 (transaction lo)
```

**⚠️ NOW SIMULATE CRASH - EXIT WITHOUT COMMIT!**

```sql
-- Type this or just close terminal
\q
```

### Terminal 1 - Re-connect and Verify

```bash
# Re-enter container
docker exec -it pg-acid psql -U postgres
```

```sql
-- Check if data rolled back
SELECT * FROM products;
```

**Expected Output:**

```
 pid | name  | price  | inventory 
-----+-------+--------+-----------
   1 | Phone | 999.99 |       100  ← ROLLBACK AYYINDI! 🎉
```

> 💡 **Atomicity Proved!** Commit cheyakunda exit chesthe, changes anni rollback ayyayi!

---

## ✅ STEP 4: ATOMICITY + CONSISTENCY - Proper Transaction

### Terminal 1 - Complete Transaction

```sql
-- Begin transaction
BEGIN;

-- Step 1: Decrease inventory
UPDATE products SET inventory = inventory - 10 WHERE pid = 1;

-- Step 2: Record the sale
INSERT INTO sales (pid, price, quantity) VALUES (1, 999.99, 10);

-- Verify both tables INSIDE transaction
SELECT * FROM products;
SELECT * FROM sales;

-- COMMIT the transaction
COMMIT;
```

**Expected Output After Commit:**

```sql
SELECT * FROM products;
--  pid | name  | price  | inventory 
-- -----+-------+--------+-----------
--    1 | Phone | 999.99 |        90

SELECT * FROM sales;
--  sale_id | pid | price  | quantity 
-- ---------+-----+--------+----------
--        1 |   1 | 999.99 |       10
```

> 💡 **Consistency Check**: 90 (inventory) + 10 (sold) = 100 (original) ✅

---

## 🔒 STEP 5: ISOLATION Demo - Part 1 (Read Committed - Default)

### 🎯 Concept: Isolation ante enti?
>
> Concurrent transactions okadini okadu disturb cheyyakudadu

### Setup: More Sales Data Add Cheyydam

```sql
-- Terminal 1: Add more test data
INSERT INTO products (name, price, inventory) VALUES ('Earbuds', 199.99, 50);

INSERT INTO sales (pid, price, quantity) VALUES 
(1, 999.99, 1),
(1, 999.99, 2),
(2, 199.99, 3);

-- Verify
SELECT * FROM sales;
```

---

### 🔴 Inconsistent Report Demo (READ COMMITTED - Default)

**Open TWO terminals simultaneously!**

### Terminal 2 - Open Second Connection

```bash
docker exec -it pg-acid psql -U postgres
```

---

### Step-by-Step Execution (Follow this order exactly!)

| Step | Terminal 1 (Report Generator) | Terminal 2 (Sale Maker) |
|------|-------------------------------|-------------------------|
| 1 | `BEGIN;` | - |
| 2 | `SELECT pid, COUNT(pid) FROM sales GROUP BY pid;` | - |
| 3 | - | `BEGIN;` |
| 4 | - | `INSERT INTO sales (pid, price, quantity) VALUES (1, 999.99, 10);` |
| 5 | - | `UPDATE products SET inventory = inventory - 10 WHERE pid = 1;` |
| 6 | - | `COMMIT;` |
| 7 | `SELECT pid, price, quantity FROM sales;` | - |
| 8 | `COMMIT;` | - |

---

### 📝 Detailed Commands

#### Terminal 1 - Start Report

```sql
BEGIN;

-- First query: Count sales per product
SELECT pid, COUNT(pid) FROM sales GROUP BY pid;
```

**Output (Step 2):**

```
 pid | count 
-----+-------
   2 |     3
   1 |     4  ← 4 sales for Phone
```

#### Terminal 2 - Meanwhile, Make a Sale

```sql
BEGIN;

-- Insert new sale
INSERT INTO sales (pid, price, quantity) VALUES (1, 999.99, 10);

-- Update inventory
UPDATE products SET inventory = inventory - 10 WHERE pid = 1;

COMMIT;
```

#### Terminal 1 - Continue Report

```sql
-- Second query: Get detailed sales
SELECT pid, price, quantity FROM sales;
```

**Output (Step 7):**

```
 pid | price  | quantity 
-----+--------+----------
   1 | 999.99 |       10
   1 | 999.99 |        1
   1 | 999.99 |        2
   2 | 199.99 |        3
   1 | 999.99 |       10  ← NEW ROW VISIBLE! 😱
```

> ⚠️ **PROBLEM!** First query said 4 sales, but second query shows 5! **Inconsistent Report!**

---

## 🔐 STEP 6: ISOLATION Demo - Part 2 (REPEATABLE READ - Fix!)

### Terminal 1 - Repeatable Read Transaction

```sql
ROLLBACK;  -- First, rollback any active transaction

-- Start with REPEATABLE READ isolation
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- First query
SELECT pid, COUNT(pid) FROM sales GROUP BY pid;
```

**Output:**

```
 pid | count 
-----+-------
   2 |     3
   1 |     5  ← Current count
```

### Terminal 2 - Try to Insert During Repeatable Read

```sql
BEGIN;

INSERT INTO sales (pid, price, quantity) VALUES (1, 999.99, 10);
UPDATE products SET inventory = inventory - 10 WHERE pid = 1;

COMMIT;
```

### Terminal 1 - Query Again (Still in REPEATABLE READ)

```sql
-- Second query - SAME transaction
SELECT pid, price, quantity FROM sales;

-- Count again
SELECT pid, COUNT(pid) FROM sales GROUP BY pid;
```

**Output:**

```
 pid | count 
-----+-------
   2 |     3
   1 |     5  ← SAME AS BEFORE! 🎉 (Not 6)
```

> 💡 **Repeatable Read Works!** Terminal 2 committed a new sale, but Terminal 1 still sees the SAME snapshot!

```sql
-- Now commit and check
COMMIT;

-- After transaction ends, new data is visible
SELECT pid, COUNT(pid) FROM sales GROUP BY pid;
```

**Output After Commit:**

```
 pid | count 
-----+-------
   2 |     3
   1 |     6  ← NOW we see 6!
```

---

## 💾 STEP 7: DURABILITY Demo

### 🎯 Concept: Durability ante enti?
>
> Once committed, data is **permanently saved** - even if server crashes!

### Terminal 1 - Insert and Immediately Kill Container

```sql
BEGIN;

-- Insert new product
INSERT INTO products (name, price, inventory) VALUES ('TV', 3000.00, 10);

-- Commit
COMMIT;
```

### Terminal 2 - Kill Container IMMEDIATELY After Commit

```bash
docker stop pg-acid
```

### Verify Durability - Restart and Check

```bash
# Start container again
docker start pg-acid

# Enter container
docker exec -it pg-acid psql -U postgres
```

```sql
-- Check if TV product exists
SELECT * FROM products;
```

**Expected Output:**

```
 pid | name    |  price  | inventory 
-----+---------+---------+-----------
   1 | Phone   |  999.99 |        70
   2 | Earbuds |  199.99 |        50
   3 | TV      | 3000.00 |        10  ← DATA SURVIVED! 🎉
```

> 💡 **Durability Proved!** Server crash ayyina, committed data safe ga undi!

---

## 📊 ACID Summary Table

| Property | Telugu Meaning | What It Guarantees |
|----------|----------------|-------------------|
| **A**tomicity | పూర్తిగా లేదా ఏమీ లేదు | All operations succeed or all rollback |
| **C**onsistency | సమంజసత్వం | Database always in valid state |
| **I**solation | వేరుచేయడం | Concurrent transactions don't interfere |
| **D**urability | శాశ్వతత్వం | Committed data survives crashes |

---

## 🧹 Cleanup Commands

```bash
# Stop container
docker stop pg-acid

# Remove container
docker rm pg-acid

# (Optional) Remove image
docker rmi postgres:13
```

---

## 🔑 Key Isolation Levels Reference

| Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|-------|------------|---------------------|--------------|
| READ UNCOMMITTED | ✅ Possible | ✅ Possible | ✅ Possible |
| READ COMMITTED | ❌ Prevented | ✅ Possible | ✅ Possible |
| REPEATABLE READ | ❌ Prevented | ❌ Prevented | ✅ Possible* |
| SERIALIZABLE | ❌ Prevented | ❌ Prevented | ❌ Prevented |

> *PostgreSQL's REPEATABLE READ also prevents phantom reads due to MVCC implementation!

---

## ✨ Quick Commands Reference

```sql
-- Transaction Commands
BEGIN;                                          -- Start transaction
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;  -- Start with specific isolation
COMMIT;                                         -- Save changes
ROLLBACK;                                       -- Discard changes

-- View current isolation level
SHOW transaction_isolation;

-- Change default isolation level
SET default_transaction_isolation = 'repeatable read';
```

---

> 📌 **Pro Tip**: Production lo mostly `READ COMMITTED` use chestaru (default). `REPEATABLE READ` only when you need consistent snapshots (reports, analytics).
