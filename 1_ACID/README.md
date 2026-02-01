# ACID Properties - Complete Notes Index 📚

## Quick Navigation

### 🎯 Start Here

| File | Description |
|:---|:---|
| [Interview Questions](./Interviews/00_Interview_Questions.md) | Quick Q&A for job prep |
| [Practical Demo PostgreSQL](./05_Practicals/01_ACID_Practical_Demo_PostgreSQL_Complete.md) | **Hands-on Lab** with Docker |

---

### ⚛️ Atomicity (All or Nothing)

| # | File | Topics |
|:---:|:---|:---|
| 1 | [Core Concepts](./01_Atomicity/01_Atomicity_Core_Concepts.md) | Definition, Why needed, Failure types |
| 2 | [Examples & Recovery](./01_Atomicity/02_Atomicity_Examples_And_Recovery.md) | Bank example, Savepoints, Recovery flow |

---

### 🛡️ Isolation (Separation)

| # | File | Topics |
|:---:|:---|:---|
| 1 | [Read Phenomena](./02_Isolation/01_Isolation_Problems_Read_Phenomena.md) | Dirty, Non-Repeatable, Phantom, Lost Update |
| 2 | [Isolation Levels](./02_Isolation/02_Isolation_Levels.md) | 4 Levels, SQL commands, DB defaults |
| 3 | [Implementation](./02_Isolation/03_Isolation_Implementation_Deep_Dive.md) | Locking, MVCC, 2PL, Deadlocks |

---

### 🏗️ Consistency (Valid State)

| # | File | Topics |
|:---:|:---|:---|
| 1 | [Data Integrity](./03_Consistency/01_Consistency_Data_Integrity.md) | FK, Constraints, A+I=C |
| 2 | [Reads & Eventual](./03_Consistency/02_Consistency_Reads_And_Eventual.md) | Strong vs Eventual, CAP theorem |

---

### 💾 Durability (Persistence)

| # | File | Topics |
|:---:|:---|:---|
| 1 | [Concept & Problem](./04_Durability/01_Durability_Concept_And_Problem.md) | RAM vs Disk, Why WAL |
| 2 | [WAL Architecture](./04_Durability/02_WAL_Architecture.md) | Log mechanism, fsync, OS Cache |
| 3 | [Transaction Lifecycle](./04_Durability/03_Transaction_Lifecycle_Deep_Dive.md) | Phases, Checkpoint |
| 4 | [Crash Recovery](./04_Durability/04_Crash_Recovery_Logic.md) | REDO/UNDO, ARIES |
| 5 | [Advanced Q&A](./04_Durability/05_Durability_Advanced_Clarifications.md) | Edge cases, Dirty writes |

---

### 🔬 Advanced Topics

| # | File | Topics |
|:---:|:---|:---|
| 1 | [Phantom Reads](./06_Phantom_Reads/03_Phantom_Reads_Practical_Demo.md) | Read phenomena, MySQL vs PostgreSQL behavior |
| 2 | [Serializable vs Repeatable](./07_Serializable_vs_Repeatable/04_Serializable_vs_Repeatable_Read.md) | Write skew, When to use which |
| 3 | [Eventual Consistency](./08_Eventual_Consistency/05_Eventual_Consistency_Deep_Dive.md) | Data vs Read consistency, Replication |

---

## One-Line Summary

| Property | Rule | Mechanism |
|:---|:---|:---|
| **A**tomicity | All or Nothing | WAL + Rollback |
| **C**onsistency | Valid → Valid | Constraints + A + I |
| **I**solation | No interference | Locks / MVCC |
| **D**urability | Committed = Permanent | WAL + fsync |

---

## Suggested Learning Path

```text
1. 📖 Read theory files (Atomicity → Isolation → Consistency → Durability)
2. 🔬 Do Practical Demo
3. 📚 Study Advanced Topics (Phantom Reads, Serializable, Eventual Consistency)
4. 🎯 Review Interview Questions
5. 🔄 Revisit any weak areas
```
