# 💰 IO - Database Currency

> IO ante enti? Enduku chala important? Simple ga artham cheskondaam!

---

## 🎯 IO Ante Enti?

**IO = Input/Output** = Disk tho communication

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   IO = Disk nundi data techukovadam (Read)              │
│        OR                                               │
│   IO = Disk lo data rayadam (Write)                     │
│                                                         │
│   Simple ga: Disk operation = IO                        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## ⚡ Why IO is EXPENSIVE (Slow)?

```
Speed Comparison:
┌────────────────────────────────────────────────────────┐
│                                                        │
│   RAM (Memory) read  →  ~100 nanoseconds               │
│   SSD (Disk) read    →  ~100,000 nanoseconds           │
│   HDD (Disk) read    →  ~10,000,000 nanoseconds        │
│                                                        │
│   RAM is 1000x to 100,000x FASTER than disk! 🚀         │
│                                                        │
└────────────────────────────────────────────────────────┘
```

**Real life analogy:**

```
RAM = Mee table meeda book (instant access)
Disk = Library lo book (walk chesi techukovaali)
```

---

## 💵 IO is Currency - What Does This Mean?

```
┌─────────────────────────────────────────────────────────┐
│  "IO is the currency of database"                       │
│                                                         │
│  Meaning: IO takkuva unte = Query fast ✅               │
│           IO ekkuva unte = Query slow ❌                │
│                                                         │
│  Money laga! Less spend = Better                        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 📖 One IO = One Page Read

**Important concept:**

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   Database reads data as PAGES, not ROWS!               │
│                                                         │
│   Example:                                              │
│   - Neeku only 1 row kavali                             │
│   - But database reads FULL PAGE (8KB)                  │
│   - Ee page lo 40 rows untayi                           │
│   - 40 lo 1 row matrame use chestav                     │
│   - Remaining 39 rows = waste 😅                         │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Visual:**

```
Your Query: SELECT * FROM employees WHERE id = 5

What you need:    [Row 5]
What DB reads:    [Row 1][Row 2][Row 3][Row 4][Row 5]...[Row 40]
                           ↑
                     Full page = 1 IO
```

---

## 🔄 IO Count Examples

| Query Type | IO Count | Speed |
|-----------|----------|-------|
| Get 1 row by ID (with index) | 1-2 IOs | ⚡ Fast |
| Scan all employees | 100+ IOs | 🐢 Slow |
| Sum of all salaries | Many IOs | 🐢 Slow |

---

## 💡 How to Reduce IO?

```
1. USE INDEXES  → Jump directly to row
2. CACHE       → Keep pages in memory (no disk read)
3. GOOD DESIGN → Store related data together
```

---

## 🌟 Key Points (Remember!)

```
✅ IO = Disk read/write operation
✅ IO is SLOW (disk is slow compared to RAM)
✅ Less IO = Faster query
✅ 1 IO = 1 page read (not 1 row)
✅ Indexes help reduce IO
```

---

## ➡️ Next: [Heap Data Structure](./04_Heap_Data_Structure.md)
