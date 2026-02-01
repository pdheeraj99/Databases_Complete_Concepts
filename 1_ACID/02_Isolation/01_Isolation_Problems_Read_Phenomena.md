# Isolation: The Shield 🛡️

## 1. What is Isolation?

Isolation ante **"Separation"**.

* **The Question:** "Nenu oka Transaction run chestunnappudu, bayata vere vallu (Concurrent Users) chese changes naku kanipinchala vaddu?"
* **The Goal:** Ideal ga, prathi transaction ontari ga (Isolated ga) run ayinattu undali.

---

## 2. The 4 Read Phenomena (Problems) ⚠️

Isolation sarigga lekapothe ee "Deyyalu" (Problems) vastayi:

### A. Dirty Read (The "Kachra" Read) 🗑️

**Definition:** Vere vadu inka **COMMIT kottani** data ni nuvvu chudatam.

```sql
-- Transaction A (Not committed yet)
UPDATE accounts SET balance = 2000 WHERE id = 1;
-- No COMMIT!

-- Transaction B (Reads uncommitted data)
SELECT balance FROM accounts WHERE id = 1;  -- Sees 2000 ❌

-- Transaction A
ROLLBACK;  -- Changed their mind!

-- Result: Transaction B saw FAKE data that never existed!
```

---

### B. Non-Repeatable Read (Changed Value) 🔄

**Definition:** Same Transaction lo, same query rendu sarlu run chesthe veru values ravadam.

```sql
-- Transaction A
SELECT salary FROM employees WHERE id = 5;  -- Returns 50000

-- Transaction B (commits in between)
UPDATE employees SET salary = 60000 WHERE id = 5;
COMMIT;

-- Transaction A (same query again)
SELECT salary FROM employees WHERE id = 5;  -- Returns 60000 ❌

-- Problem: Same query, different answers within ONE transaction!
```

---

### C. Phantom Read (Ghost Row) 👻

**Definition:** Row count suddenly maaradam.

```sql
-- Transaction A
SELECT COUNT(*) FROM orders WHERE amount > 100;  -- Returns 5

-- Transaction B (adds new row)
INSERT INTO orders (amount) VALUES (150);
COMMIT;

-- Transaction A (same query)
SELECT COUNT(*) FROM orders WHERE amount > 100;  -- Returns 6 👻

-- A new row appeared like a ghost!
```

---

### D. Lost Update (Overwrite Disaster) ❌

**Definition:** Iddaru okesari edit chesthe, okari change lost avvadam.

```sql
-- Initial: Balance = 100

-- Transaction A reads
SELECT balance FROM accounts;  -- Gets 100

-- Transaction B reads
SELECT balance FROM accounts;  -- Gets 100

-- Transaction A: Adds 50
UPDATE accounts SET balance = 150;  -- 100 + 50
COMMIT;

-- Transaction B: Adds 30 (still thinks it's 100!)
UPDATE accounts SET balance = 130;  -- 100 + 30
COMMIT;

-- Final: 130 (A's +50 is LOST!)
-- Should be: 100 + 50 + 30 = 180
```

---

## 🎯 Key Takeaways

| Problem | What Goes Wrong | Impact |
|:---|:---|:---|
| **Dirty Read** | Read uncommitted data | See fake/rolled-back data |
| **Non-Repeatable** | Value changes mid-transaction | Inconsistent calculations |
| **Phantom** | Row count changes | Wrong aggregations |
| **Lost Update** | Concurrent overwrites | Data loss |
