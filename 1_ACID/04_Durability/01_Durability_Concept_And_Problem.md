# Durability: The "Save Game" Guarantee 💾

## 1. What is Durability?

Simple ga cheppalante, Durability (ACID lo 'D') ante **Persistence**.

| Rule | Description |
|:---|:---|
| **Rule 1** | COMMIT chesina changes permanent ga Non-Volatile Storage (Disk/SSD) meeda save avvali |
| **Rule 2** | Power poyina, OS crash ayina, DB restart ayina data safe ga undali |
| **Critical** | Ee guarantee only **Committed Transactions** ki matrame |

---

## 2. The Hardware Problem: Speed vs Safety 🐢 vs 🚀

Database design lo pedda thalanoppi idhe. Manaki rendu contradicting requirements untayi.

### RAM vs Disk Comparison

| Feature | RAM (Memory) 🚀 | Disk (Storage) 🐢 |
|:---|:---|:---|
| **Speed** | Nanoseconds | Milliseconds (1000x slower!) |
| **Nature** | **Volatile** (Power pothe data pothundi) | **Non-Volatile** (Safe) |
| **Use Case** | Fast calculations | Permanent storage |

---

## 3. Why Not Write Directly to Data Files?

```text
❌ Problem with Direct Writes:
┌──────────────────────────────────────────┐
│  UPDATE row in Table X                   │
│         ↓                                │
│  Find row location on disk (Random I/O)  │  ← SLOW!
│         ↓                                │
│  Write to that specific location         │  ← SLOW!
└──────────────────────────────────────────┘

✅ Solution: WAL (Write Ahead Log)
┌──────────────────────────────────────────┐
│  UPDATE row in Table X                   │
│         ↓                                │
│  Append to Log file (Sequential I/O)     │  ← FAST!
│         ↓                                │
│  Update actual table later (Checkpoint)  │
└──────────────────────────────────────────┘
```

---

## 4. The Solution: WAL (Write Ahead Log) 💡

> **Simple Logic:** "Main Data ni tarvatha mellaga update cheddam. Mundu em change chesamo oka **Simple Log File** lo raseddam."

### Why WAL Works

| Aspect | Random I/O (Data Files) | Sequential I/O (WAL) |
|:---|:---|:---|
| **Disk Head Movement** | Everywhere | Continuous append |
| **Speed** | Slow | Fast |
| **Reliability** | Complex | Simple |

---

## 🎯 Key Takeaways

1. **Persistence = Disk** - RAM is not durable
2. **Speed vs Safety Trade-off** - WAL solves this elegantly
3. **Sequential > Random** - Appending to log is faster than updating tables
4. **Commit ≠ Table Update** - Log is saved, table update is lazy
