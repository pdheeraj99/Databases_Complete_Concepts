# 📄 Pages Concept - Data Storage

> Database data ela store chestundi disk lo? PAGES lo!

---

## 🎯 Page Ante Enti?

**Page** = Fixed size data block. Database anni data ee blocks lo store chestundi.

```
┌─────────────────────────────────────────────────────────┐
│                     SIMPLE ANALOGY                       │
│                                                         │
│   Book lo pages untayi right?                           │
│   100 pages book = 100 fixed size papers                │
│                                                         │
│   Database lo kuda same!                                │
│   1000 pages database = 1000 fixed size blocks          │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 📐 Page Size - Different Databases

| Database | Default Page Size |
|----------|------------------|
| PostgreSQL | 8 KB (8192 bytes) |
| MySQL InnoDB | 16 KB |
| SQL Server | 8 KB |
| Oracle | 8 KB |

> 💡 **Simple**: Page size = oka block ekkada data padutundi

---

## 📦 Rows in Pages

Oka page lo multiple rows padtayi!

```
┌─────────────────────────────────────────────────────────┐
│                     PAGE 1 (8 KB)                        │
├─────────────────────────────────────────────────────────┤
│  Row 1: ID=1, Name="Ravi", Salary=50000                 │
│  Row 2: ID=2, Name="Priya", Salary=60000                │
│  Row 3: ID=3, Name="Kiran", Salary=55000                │
│  Row 4: ID=4, Name="Kumar", Salary=45000                │
│  ... more rows until page is full ...                   │
│                                                         │
│  [---------- FREE SPACE -----------]                    │
└─────────────────────────────────────────────────────────┘
```

---

## 🔢 How Many Rows in One Page?

**Calculation:**

```
Page Size = 8 KB = 8192 bytes
Row Size = 100 bytes (example)

Rows per page = 8192 / 100 = ~80 rows per page
```

**Real example:**

```
┌──────────────────────────────────────────┐
│ 1 employee row = ~200 bytes              │
│ 1 page = 8 KB = 8192 bytes               │
│                                          │
│ So: 8192 / 200 = ~40 employees per page  │
└──────────────────────────────────────────┘
```

---

## 🎯 Why Fixed Size Pages?

```
┌─────────────────────────────────────────────────────────┐
│  BENEFITS OF FIXED SIZE:                                 │
│                                                         │
│  1. Easy to read - always 8KB read chestundi            │
│  2. Easy to write - always 8KB write chestundi          │
│  3. Easy to cache - memory lo easy ga fit avutundi      │
│  4. Easy to manage - simple calculations                │
│                                                         │
│  Variable size unte chaos avutundi! 💥                   │
└─────────────────────────────────────────────────────────┘
```

---

## 🌟 Key Points (Remember!)

```
✅ Page = Fixed size block (usually 8KB or 16KB)
✅ Multiple rows fit in one page
✅ All databases use pages for storage
✅ Fixed size = easier to manage
✅ Page number + position = row address
```

---

## ➡️ Next: [IO - Why it's Expensive](./03_IO_Database_Currency.md)
