# Isolation Levels: Speed vs Safety ⚖️

Ee problems ni apadaniki Databases manaki 4 Levels isthayi.

## Quick Reference Table

| Level | Dirty Read | Non-Repeatable | Phantom | Speed |
|:---|:---:|:---:|:---:|:---:|
| **Read Uncommitted** | ⚠️ Yes | ⚠️ Yes | ⚠️ Yes | 🚀🚀🚀 |
| **Read Committed** | ✅ No | ⚠️ Yes | ⚠️ Yes | 🚀🚀 |
| **Repeatable Read** | ✅ No | ✅ No | ⚠️ Yes* | 🚀 |
| **Serializable** | ✅ No | ✅ No | ✅ No | 🐢 |

*Postgres lo Phantoms kuda prevent avthay!

---

## 1. Read Uncommitted (Lowest Safety) 🚀

```sql
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
```

* **Behavior:** "Evaru em rasina (Commit avvakapoina) naku chupinchu"
* **Use Case:** Approximate analytics (YouTube Views count)
* **Risk:** Dirty Reads possible

---

## 2. Read Committed (Default for PostgreSQL) ✅

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

* **Behavior:** "Only COMMIT ayina data ne chupinchu"
* **Use Case:** Real-time apps (Ticket Booking, ATM)
* **Still Risky:** Value can change between reads

---

## 3. Repeatable Read (Snapshot) 🔒

```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

* **Behavior:** "Nenu start ayyappudu data ela undo, end varaku alage chupinchu"
* **Use Case:** Financial Reports, Audits
* **How:** Creates snapshot at transaction start

---

## 4. Serializable (Maximum Safety) 🛡️

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

* **Behavior:** "One after another. No concurrency."
* **Use Case:** Critical financial operations
* **Trade-off:** Very slow, may cause retries

---

## Database Defaults

| Database | Default Level |
|:---|:---|
| **PostgreSQL** | Read Committed |
| **MySQL (InnoDB)** | Repeatable Read |
| **Oracle** | Read Committed |
| **SQL Server** | Read Committed |

---

## Setting in Code Examples

```java
// Java/JDBC
connection.setTransactionIsolation(
    Connection.TRANSACTION_REPEATABLE_READ);

// Spring Boot
@Transactional(isolation = Isolation.SERIALIZABLE)
public void transferMoney() { ... }
```

---

## 🎯 Key Takeaways

1. **Higher Safety = Lower Speed** - Always a trade-off
2. **Know Your Default** - Check your DB's default level
3. **Use Case Matters** - Social media ≠ Banking
4. **Most Apps** - Read Committed is fine for 90% cases
