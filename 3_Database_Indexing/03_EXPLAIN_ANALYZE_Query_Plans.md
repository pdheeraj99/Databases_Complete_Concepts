# 📊 Lesson 3: EXPLAIN ANALYZE - Query Plans Artham Chesukovadam

## 📌 EXPLAIN ANALYZE Ante Em?

PostgreSQL lo **query ela execute ayyindo** choodaniki powerful tool. Ee command database ni question chestundi:

> "Nuvvu ee query ela process chesav? Entha time pattindi? Index vaadava leda table scan chesava?"

---

## 🎯 Basic Syntax

```sql
EXPLAIN ANALYZE <your_query>;
```

**Two Keywords:**

| Keyword | Purpose |
|---------|---------|
| `EXPLAIN` | Query execution **plan** chupistundi (actually run cheyyadu) |
| `ANALYZE` | Query **actually run chesi** real timings istundi |

> [!TIP]
> Always `EXPLAIN ANALYZE` together use cheyandi - plan + actual performance both kavali!

---

## 🔍 First Example - Index Scan

### Query

```sql
EXPLAIN ANALYZE 
SELECT id FROM employees WHERE id = 2000;
```

### Output Breakdown

```
Index Scan using employees_pkey on employees  
  (cost=0.43..8.45 rows=1 width=4) 
  (actual time=0.045..0.047 rows=1 loops=1)
  
  Index Cond: (id = 2000)
  
Planning Time: 0.123 ms
Execution Time: 0.068 ms
```

**Let's decode each line:**

#### Line 1: Scan Type & Cost

```
Index Scan using employees_pkey on employees
```

| Part | Meaning |
|------|---------|
| `Index Scan` | Index use chesi data search chesindi |
| `employees_pkey` | PRIMARY KEY index name |
| `on employees` | employees table meeda operation |

#### Line 2: Cost & Row Info

```
(cost=0.43..8.45 rows=1 width=4)
```

| Metric | Value | Explanation |
|--------|-------|-------------|
| **Startup cost** | 0.43 | Query start cheyadaniki cost |
| **Total cost** | 8.45 | Pura query complete avvadam cost |
| **rows** | 1 | Expected rows count |
| **width** | 4 | Each row size (bytes) - INTEGER = 4 bytes |

> [!NOTE]
> Cost ante "abstract units" - actual seconds kaadu. Comparison ki use chesthaaru.

#### Line 3: Actual Timings

```
(actual time=0.045..0.047 rows=1 loops=1)
```

| Metric | Value | Explanation |
|--------|-------|-------------|
| **actual time** | 0.045..0.047 ms | Real execution time (milliseconds) |
| **rows** | 1 | Actual rows returned |
| **loops** | 1 | Query enni saarllu loop chesindi |

#### Line 4: Filter Condition

```
Index Cond: (id = 2000)
```

- Index meeda ee condition apply chesindi

#### Final Lines: Summary

```
Planning Time: 0.123 ms   ← Query plan create cheyadaniki time
Execution Time: 0.068 ms  ← Actual execution time
```

**Total Time = Planning + Execution = 0.191 ms** ⚡

---

## 🔥 Performance Visualization

```mermaid
gantt
    title Query Execution Timeline
    dateFormat X
    axisFormat %L ms
    
    section Planning
    Analyze query    :0, 0.123
    
    section Execution
    Index scan       :0.123, 0.191
```

---

## 🐌 Second Example - Sequential Scan (Slow!)

### Query (No Index on 'name')

```sql
EXPLAIN ANALYZE 
SELECT id FROM employees WHERE name = 'xZs';
```

### Output

```
Gather  (cost=1000.00..128750.11 rows=1 width=4) 
  (actual time=1456.234..3024.567 rows=0 loops=1)
  
  Workers Planned: 2
  Workers Launched: 2
  
  -> Parallel Seq Scan on employees  
       (cost=0.00..127750.11 rows=1 width=4) 
       (actual time=1234.567..2987.345 rows=0 loops=3)
       
       Filter: (name = 'xZs'::text)
       Rows Removed by Filter: 3666667
       
Planning Time: 0.834 ms
Execution Time: 3024.789 ms
```

**Analysis:**

| Aspect | Details |
|--------|---------|
| **Scan Type** | `Parallel Seq Scan` - Full table scan! |
| **Parallel Workers** | 2 workers help chestunnaru |
| **Execution Time** | **3024 ms = 3 seconds!** 😱 |
| **Rows Removed** | 3.6 million rows filter chesindi (waste!) |

---

## 📊 Cost Comparison Table

| Query Type | Scan Method | Time | Rows Scanned |
|------------|-------------|------|--------------|
| **With Index (id)** | Index Scan | 0.068 ms | 1 |
| **Without Index (name)** | Sequential Scan | 3024 ms | 11,000,000 |
| **Difference** | — | **44,470x slower!** | 11M times more work |

---

## 🧩 Understanding Scan Types

### Index Scan

```
        Index (Small, Organized)
             ↓
    Smart search → Found!
             ↓
    Jump to table row
```

**Characteristics:**

- ✅ Very fast
- ✅ Minimal rows touched
- ✅ Used when exact match or range query
- ⚠️ Requires index to exist

### Sequential Scan (Seq Scan)

```
Table (Huge, Unorganized)
    ↓
Row 1 → Check → No
Row 2 → Check → No
...
Row 11M → Check → No
```

**Characteristics:**

- ❌ Very slow
- ❌ Reads entire table
- ✅ No index needed
- ⚠️ Used when index unavailable or inefficient

### Parallel Sequential Scan

```
Worker 1: Rows 1 - 3.6M
Worker 2: Rows 3.6M - 7.3M  
Worker 3: Rows 7.3M - 11M

↓ All work simultaneously ↓

Faster than single-threaded,
but still slow overall
```

---

## 🎯 How to Read EXPLAIN Output

### Step-by-Step Process

1. **Look at scan type** (first line)
   - `Index Scan` = Good! ✅
   - `Seq Scan` = Bad (unless table small) ❌
   - `Bitmap Scan` = Medium (next lesson)

2. **Check execution time** (last line)
   - < 1 ms = Excellent ⚡
   - 1-100 ms = Good ✅
   - > 100 ms = Investigate 🔍
   - > 1000 ms = Problem! 🚨

3. **Verify rows returned**
   - Expected vs actual should match
   - `rows=0` but took 3 seconds? Index missing!

4. **Look for filters**
   - `Rows Removed by Filter: 3.6M` = Waste of work

---

## 🛠️ Practical Tips

### Tip 1: Always Check First Query

```sql
-- First query cached avtundi
EXPLAIN ANALYZE SELECT * FROM employees WHERE id = 1000;

-- Second query clean test
EXPLAIN ANALYZE SELECT * FROM employees WHERE id = 1001;
```

### Tip 2: Compare Before/After Index

**Before Index:**

```sql
EXPLAIN ANALYZE SELECT * FROM employees WHERE name = 'abc';
-- Output: Seq Scan, 3000 ms
```

**After Index:**

```sql
CREATE INDEX idx_employees_name ON employees(name);

EXPLAIN ANALYZE SELECT * FROM employees WHERE name = 'abc';
-- Output: Index Scan, 45 ms
```

**Improvement = 67x faster!**

---

## 📐 Cost Estimation Formula

PostgreSQL estimates cost based on:

```
Total Cost = 
    (Disk Pages × seq_page_cost) +
    (Index Pages × random_page_cost) +
    (CPU Processing × cpu_tuple_cost)
```

**Default values:**

- `seq_page_cost = 1.0`
- `random_page_cost = 4.0` (index access more expensive)
- `cpu_tuple_cost = 0.01`

> [!NOTE]
> Cost units are **abstract** - only useful for comparison, not actual time!

---

## 🧠 Key Takeaways

1. **EXPLAIN ANALYZE** shows query execution plan + actual timings
2. **Index Scan** = Fast, efficient
3. **Sequential Scan** = Slow, reads entire table
4. **Execution Time** is what matters (not cost)
5. **Parallel workers** help but don't fix missing indexes
6. Always compare **before/after** index creation

---

## ❓ Common Questions

**Q: Cost high but time low - em chesali?**
> A: **Time e important!** Cost just estimate. Real-world lo time chusi decide cheyyali.

**Q: Parallel Seq Scan when happens?**
> A:
>
> - Index lenapudu
> - Table chala pedda ayinapudu (>1M rows typically)
> - PostgreSQL config lo `max_parallel_workers` set chesthe

**Q: Index undi kaani Seq Scan chestundi, why?**
> A: Many reasons:
>
> - Query returns large % of table (index inefficient)
> - Statistics outdated (`ANALYZE` command run cheyandi)
> - Index not suitable for query type (e.g., LIKE '%abc%')

---

## ➡️ Next Lesson

**Coming up:**

- **Heap Fetches** ante em?
- **Bitmap Scan** vs Index Scan
- **Inline queries** - sweetest optimization!
