# 🏢 Database Specific Behavior

> Different databases lo Primary Key ela work avutundi? MySQL vs PostgreSQL vs Oracle

---

## 📊 Quick Comparison

| Database | Primary Key Behavior | Secondary Index |
|----------|---------------------|-----------------|
| **MySQL InnoDB** | Always Clustered | Points to Primary Key |
| **PostgreSQL** | NOT Clustered (like secondary) | Points to Row |
| **Oracle** | Option for IOT | Points to Row |
| **SQL Server** | Default Clustered | Points to Clustered Key |

---

## 🐬 MySQL InnoDB

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   MySQL InnoDB = ALWAYS CLUSTERED                       │
│                                                         │
│   ┌──────────────────────────────────────┐             │
│   │  PRIMARY KEY (Clustered Index)        │             │
│   │                                       │             │
│   │  [1] → Full Row Data                  │             │
│   │  [2] → Full Row Data                  │             │
│   │  [3] → Full Row Data                  │             │
│   │                                       │             │
│   │  Data IS the index!                   │             │
│   └──────────────────────────────────────┘             │
│                                                         │
│   Secondary Index in MySQL:                             │
│   ┌──────────────────────────────────────┐             │
│   │  [Name] → Primary Key Value           │             │
│   │  [Ravi] → 1                           │             │
│   │  [Priya] → 2                          │             │
│   │                                       │             │
│   │  Points to PRIMARY KEY, not Row ID!  │             │
│   └──────────────────────────────────────┘             │
│                                                         │
│   SELECT * via secondary index:                         │
│   Step 1: Secondary Index → Get Primary Key             │
│   Step 2: Primary Key Index → Get Row                   │
│   = Two index traversals! 🤔                            │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 🐘 PostgreSQL

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   PostgreSQL = ALL INDEXES ARE SECONDARY!               │
│                                                         │
│   ┌──────────────────────────────────────┐             │
│   │  HEAP (Table)                         │             │
│   │                                       │             │
│   │  Page 0, Item 1 → [1, Ravi, 50000]   │             │
│   │  Page 0, Item 2 → [2, Priya, 60000]  │             │
│   │  Page 1, Item 1 → [3, Kiran, 55000]  │             │
│   │                                       │             │
│   │  Data is in HEAP, unordered!         │             │
│   └──────────────────────────────────────┘             │
│                                                         │
│   Primary Key Index (also secondary!):                  │
│   ┌──────────────────────────────────────┐             │
│   │  [1] → (Page 0, Item 1)               │             │
│   │  [2] → (Page 0, Item 2)               │             │
│   │  [3] → (Page 1, Item 1)               │             │
│   │                                       │             │
│   │  Points to Tuple ID (TID)!           │             │
│   └──────────────────────────────────────┘             │
│                                                         │
│   🤔 Even Primary Key = Secondary behavior!             │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 🏛️ Oracle

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   Oracle = YOU CHOOSE!                                  │
│                                                         │
│   Option 1: Regular Table (HOT - Heap Organized Table) │
│   ─────────                                             │
│   - Data in heap                                        │
│   - Primary key = secondary index                       │
│   - Like PostgreSQL                                     │
│                                                         │
│   Option 2: IOT (Index Organized Table)                │
│   ─────────                                             │
│   - Data in primary key index                           │
│   - Clustered!                                         │
│   - Like MySQL                                         │
│                                                         │
│   CREATE TABLE employees (                              │
│       id INT PRIMARY KEY,                               │
│       name VARCHAR(50)                                  │
│   ) ORGANIZATION INDEX;   ← Makes it IOT!              │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 🏢 SQL Server

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   SQL Server = DEFAULT CLUSTERED, but optional         │
│                                                         │
│   Default:                                              │
│   CREATE TABLE employees (                              │
│       id INT PRIMARY KEY    ← Clustered by default!    │
│   );                                                    │
│                                                         │
│   Non-Clustered:                                        │
│   CREATE TABLE employees (                              │
│       id INT PRIMARY KEY NONCLUSTERED  ← Heap style   │
│   );                                                    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 📊 Impact Summary

| Scenario | MySQL | PostgreSQL | Winner |
|----------|-------|-----------|--------|
| `SELECT * WHERE id=5` | 1 lookup | 2 lookups | MySQL ⚡ |
| Range query on PK | Very fast | Slower | MySQL ⚡ |
| UPDATE row (no PK change) | Possible reorg | Just update | PostgreSQL ⚡ |
| Secondary index lookup | 2 index traversals | 1 index + 1 heap | Depends 🤔 |

---

## 💡 Key Takeaways

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  MySQL InnoDB:                                          │
│  ✅ Great for range queries on primary key              │
│  ⚠️ Avoid random UUIDs as primary key                   │
│  ⚠️ Secondary indexes are "heavier" (store PK)          │
│                                                         │
│  PostgreSQL:                                            │
│  ✅ More flexible (all indexes equal)                   │
│  ✅ HOT optimization for updates                        │
│  ⚠️ No automatic clustering benefit                     │
│                                                         │
│  Oracle:                                                │
│  ✅ Choose based on workload                            │
│  ✅ IOT for read-heavy, range query workloads          │
│                                                         │
│  SQL Server:                                            │
│  ✅ Similar to MySQL by default                         │
│  ✅ Option to go non-clustered                          │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 🌟 Key Points (Remember!)

```
✅ MySQL = Always clustered primary key
✅ PostgreSQL = All indexes are secondary
✅ Oracle = IOT option for clustering
✅ SQL Server = Clustered by default
✅ Choose database based on your workload!
```

---

## ⬅️ Back to: [README](../README.md)
