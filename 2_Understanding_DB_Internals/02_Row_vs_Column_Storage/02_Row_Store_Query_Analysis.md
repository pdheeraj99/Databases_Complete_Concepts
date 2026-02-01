# 🔍 Row Store - Query Analysis

> Row store lo different queries ela work avutayi? IO count enti?

---

## 📊 Sample Table

```
employees table (1000 rows, 4 pages lo fit avutundi):

┌────┬────────┬────────┬───────────┬─────────────────┐
│ ID │ Name   │ Salary │ SSN       │ Email           │
├────┼────────┼────────┼───────────┼─────────────────┤
│ 1  │ Ravi   │ 50000  │ 111-11-11 │ ravi@mail.com   │
│ 2  │ Priya  │ 60000  │ 222-22-22 │ priya@mail.com  │
│ 3  │ Kiran  │ 55000  │ 333-33-33 │ kiran@mail.com  │
│... │ ...    │ ...    │ ...       │ ...             │
└────┴────────┴────────┴───────────┴─────────────────┘

Pages:
- Page 1: Rows 1-250
- Page 2: Rows 251-500
- Page 3: Rows 501-750
- Page 4: Rows 751-1000
```

---

## 🔎 Query 1: Find by Column (with Index)

```sql
SELECT name FROM employees WHERE ssn = '666-66-66'
```

**Process:**

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  Step 1: Index lo SSN find chey                        │
│          → SSN '666-66-66' is in Row 666               │
│          → Row 666 is in Page 3                        │
│          IO: 1-2 (Index pages)                         │
│                                                         │
│  Step 2: Page 3 read chey, Row 666 get chey            │
│          IO: 1 (Data page)                             │
│                                                         │
│  Step 3: From row, name extract chey                   │
│          → Name = "Someone"                            │
│                                                         │
│  TOTAL IO: ~3 IOs ✅                                    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 🔎 Query 2: Get Full Row (SELECT *)

```sql
SELECT * FROM employees WHERE id = 1
```

**Process:**

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  Step 1: ID is Primary Key (probably has index)        │
│          → ID 1 is in Page 1                           │
│          IO: 1-2 (Index lookup)                        │
│                                                         │
│  Step 2: Page 1 read chey                              │
│          → Full row available! ✅                       │
│                                                         │
│  ┌───────────────────────────────────────┐             │
│  │ Page 1:                               │             │
│  │ Row 1: [1|Ravi|50000|111..|ravi@..]  ← Got it!      │
│  │ Row 2: [2|Priya|60000|222..|priya@..]│             │
│  │ ...                                   │             │
│  └───────────────────────────────────────┘             │
│                                                         │
│  TOTAL IO: ~1 IO ⚡ (Super fast!)                       │
│                                                         │
│  👍 Row store excel in SELECT *                        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 🔎 Query 3: SUM of Column (Aggregate)

```sql
SELECT SUM(salary) FROM employees
```

**Process:**

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  ❌ PROBLEM: I need ONLY salary column                  │
│              But Row Store has FULL rows!              │
│                                                         │
│  Page 1: Read full page (8KB)                          │
│          → Got: [id|name|SALARY|ssn|email]...          │
│          → Use: Only SALARY                            │
│          → Waste: id, name, ssn, email 🗑️               │
│          IO: 1                                         │
│                                                         │
│  Page 2: Read full page (8KB)                          │
│          → Same waste 🗑️                                │
│          IO: 1                                         │
│                                                         │
│  Page 3: Same...IO: 1                                  │
│  Page 4: Same...IO: 1                                  │
│                                                         │
│  TOTAL IO: 4 IOs 🐢                                    │
│                                                         │
│  ⚠️ All 4 pages read chesam, but only salary column    │
│     use chesam. Rest all WASTE!                        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 📊 Summary - Row Store IO Count

| Query | What We Need | What We Read | IO Count | Efficiency |
|-------|--------------|--------------|----------|------------|
| `SELECT name WHERE ssn=X` | 1 column | Full row | ~3 | ⚡ OK |
| `SELECT * WHERE id=1` | Full row | Full row | ~1 | ⚡ GREAT |
| `SELECT SUM(salary)` | 1 column | All rows | ~4 | 🐢 POOR |

---

## 💡 Key Insight

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  Row Store:                                             │
│  ✅ SELECT * queries → FAST (all data together)         │
│  ❌ Aggregate queries → SLOW (lot of waste data)        │
│                                                         │
│  "Mottam row kavali ante Row Store best!               │
│   Only one column kavali ante wasteful!" 🤔             │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 🌟 Key Points (Remember!)

```
✅ SELECT * = Very efficient in Row Store (1 IO)
✅ SUM/AVG = Inefficient (reads unnecessary columns)
✅ Row Store = Best for "get full record" queries
✅ Waste happens when only few columns needed
```

---

## ➡️ Next: [Column Oriented Basics](./03_Column_Oriented_Basics.md)
