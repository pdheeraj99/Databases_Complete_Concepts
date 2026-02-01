# ACID Interview Questions 🎯

Quick reference for job interviews.

## Atomicity Questions

**Q1: What is Atomicity?**
> All operations in a transaction succeed or all fail. No partial execution.

**Q2: How does DB achieve atomicity after crash?**
> WAL (Write Ahead Log) stores undo information. On restart, incomplete transactions are rolled back.

---

## Consistency Questions

**Q3: Who is responsible for consistency - DB or Developer?**
> Developer defines rules (constraints, triggers). DB enforces them.

**Q4: What's the relationship between Atomicity, Isolation, and Consistency?**
> Consistency is the **result**. Atomicity and Isolation are mechanisms that achieve it.

---

## Durability Questions

**Q5: What happens when we COMMIT?**
> Only WAL is written to disk (fsync). Main tables are updated later during checkpoint.

**Q6: Why use WAL instead of writing directly to tables?**
> Sequential I/O (WAL) is faster than Random I/O (tables). Plus centralized recovery.

**Q7: What is fsync?**
> System call that forces OS to write data to physical disk, bypassing OS cache.

**Q8: Explain REDO vs UNDO in crash recovery.**
>
> - REDO: COMMIT found, but table not updated → Apply changes
> - UNDO: No COMMIT found → Rollback changes

---

## Isolation Questions

**Q9: Name the 4 read phenomena.**
>
> 1. Dirty Read - Reading uncommitted data
> 2. Non-Repeatable Read - Same query, different values
> 3. Phantom Read - New rows appearing
> 4. Lost Update - Concurrent overwrites

**Q10: What isolation level prevents all problems?**
> Serializable - but it's slowest. Most apps use Read Committed.

**Q11: What is MVCC?**
> Multi-Version Concurrency Control. Keeps multiple versions of data so readers never block writers. Used by PostgreSQL, MySQL.

**Q12: Explain Deadlock.**
> Two transactions waiting for locks held by each other. Solutions: timeout, wait-die, wound-wait.

**Q13: What is Two-Phase Locking (2PL)?**
> Growing phase: acquire locks. Shrinking phase: release locks. Once you release, can't acquire more.

---

## Quick Comparison Table

| ACID Property | One-Line Definition | Key Mechanism |
|:---|:---|:---|
| **Atomicity** | All or Nothing | WAL + Rollback |
| **Consistency** | Valid state to valid state | Constraints + Triggers |
| **Isolation** | Transactions don't interfere | Locks / MVCC |
| **Durability** | Committed = Permanent | WAL + fsync |

---

## Practical Demo Questions 🔬

**Q14: How would you test Atomicity in PostgreSQL?**
> Start transaction, make changes, kill the container before COMMIT. Restart and verify data rolled back.

**Q15: What's the default isolation level in PostgreSQL? MySQL?**
> PostgreSQL: Read Committed | MySQL (InnoDB): Repeatable Read

**Q16: How to change isolation level in SQL?**
> `BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;`

**Q17: Why might a report show inconsistent counts?**
> Using Read Committed level - concurrent INSERTs become visible mid-transaction. Use Repeatable Read for consistent snapshots.

**Q18: How to prove Durability works?**
> Insert row, COMMIT, immediately kill container. Restart and verify row exists. If COMMIT returned, data is on disk.

---

## Docker Quick Test Commands

```bash
# Setup
docker run --name pg_acid -e POSTGRES_PASSWORD=postgres -d postgres:13
docker exec -it pg_acid psql -U postgres

# Kill for testing
docker stop pg_acid && docker start pg_acid
```
