# 📚 Database Internals - Complete Guide

> Simple ga understand avvali ani Telugu + English mix lo raasanu!

---

## 📖 Learning Path (Follow This Order!)

```
┌─────────────────────────────────────────────────────────┐
│  START HERE                                             │
│     ↓                                                   │
│  1. Fundamentals (Tables, Pages, IO)                   │
│     ↓                                                   │
│  2. Heap & Indexes (Storage + Fast Access)             │
│     ↓                                                   │
│  3. Primary & Secondary Keys (Key Mechanics)           │
│     ↓                                                   │
│  4. Query Execution (How Queries Work!) ⭐              │
│     ↓                                                   │
│  5. Row vs Column Storage (Types Comparison)           │
│     ↓                                                   │
│  6. Pages Deep Dive (Buffer Pool, Internals)           │
│     ↓                                                   │
│  DONE! 🎉                                               │
└─────────────────────────────────────────────────────────┘
```

---

## 1️⃣ [Fundamentals](./01_Fundamentals/)

Database basics - foundation concepts

| File | Topic |
|------|-------|
| [01_Tables_And_Rows](./01_Fundamentals/01_Tables_And_Rows.md) | Table ante enti? Row ante enti? |
| [02_Pages_Concept](./01_Fundamentals/02_Pages_Concept.md) | Page ante enti? Size entha? |
| [03_IO_Database_Currency](./01_Fundamentals/03_IO_Database_Currency.md) | IO ante enti? Enduku important? |

---

## 2️⃣ [Heap & Indexes](./02_Heap_And_Indexes/)

Data storage and fast access methods

| File | Topic |
|------|-------|
| [01_Heap_And_Clustering](./02_Heap_And_Indexes/01_Heap_And_Clustering.md) | Heap ante enti? Clustering enti? |
| [02_Indexes_And_BTrees](./02_Heap_And_Indexes/02_Indexes_And_BTrees.md) | Index ante enti? B-Tree ela work avutundi? |

---

## 3️⃣ [Primary & Secondary Keys](./03_Primary_Secondary_Keys/)

Key types and their behavior

| File | Topic |
|------|-------|
| [01_Primary_Key_Clustered_Index](./03_Primary_Secondary_Keys/01_Primary_Key_Clustered_Index.md) | Primary Key ela work avutundi? |
| [02_Secondary_Index_Mechanics](./03_Primary_Secondary_Keys/02_Secondary_Index_Mechanics.md) | Secondary Index ante enti? |
| [03_Database_Specific_Behavior](./03_Primary_Secondary_Keys/03_Database_Specific_Behavior.md) | MySQL vs PostgreSQL vs Oracle |

---

## 4️⃣ [Query Execution](./04_Query_Execution/) ⭐

**IMPORTANT!** How queries actually work internally

| File | Topic |
|------|-------|
| [01_Query_Execution_Flow](./04_Query_Execution/01_Query_Execution_Flow.md) | 🔥 Query internally ela execute avutundi? Complete flow! |

---

## 5️⃣ [Row vs Column Storage](./05_Row_vs_Column/)

Different storage strategies comparison

| File | Topic |
|------|-------|
| [01_Row_Oriented_Basics](./05_Row_vs_Column/01_Row_Oriented_Basics.md) | Row store ela work avutundi? |
| [02_Row_Store_Query_Analysis](./05_Row_vs_Column/02_Row_Store_Query_Analysis.md) | Row store lo queries |
| [03_Column_Oriented_Basics](./05_Row_vs_Column/03_Column_Oriented_Basics.md) | Column store ela work avutundi? |
| [04_Column_Store_Query_Analysis](./05_Row_vs_Column/04_Column_Store_Query_Analysis.md) | Column store lo queries |
| [05_Comparison_And_When_To_Use](./05_Row_vs_Column/05_Comparison_And_When_To_Use.md) | Ekkada edi use cheyali? |

---

## 6️⃣ [Pages Deep Dive](./06_Pages_Deep_Dive/)

Advanced page concepts

| File | Topic |
|------|-------|
| [01_Page_Basics_And_Buffer_Pool](./06_Pages_Deep_Dive/01_Page_Basics_And_Buffer_Pool.md) | Page basics & Memory cache |
| [02_Read_Write_Operations](./06_Pages_Deep_Dive/02_Read_Write_Operations.md) | Read/Write ela jaragutayi? |
| [03_Page_Content_Types](./06_Pages_Deep_Dive/03_Page_Content_Types.md) | Different databases lo pages |
| [04_PostgreSQL_Page_Layout](./06_Pages_Deep_Dive/04_PostgreSQL_Page_Layout.md) | PostgreSQL page structure |

---

## 🎯 Quick Reference

| Concept | File Location |
|---------|--------------|
| What is a Page? | 01_Fundamentals/02_Pages_Concept |
| What is Heap? | 02_Heap_And_Indexes/01_Heap_And_Clustering |
| What is an Index? | 02_Heap_And_Indexes/02_Indexes_And_BTrees |
| How Query Works? | 04_Query_Execution/01_Query_Execution_Flow |
| Row vs Column? | 05_Row_vs_Column/05_Comparison |
| Buffer Pool? | 06_Pages_Deep_Dive/01_Buffer_Pool |
