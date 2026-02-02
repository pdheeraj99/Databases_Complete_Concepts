# ⚠️ Lesson 6: LIKE Operator & Index Pitfalls

## 📌 The LIKE Operator Problem

Hussein's video lo chala important point - **Index undi kuda use avvadu sometimes!**

---

## 🎯 Normal Query (Index Works)

### Exact Match - Perfect

```sql
EXPLAIN ANALYZE 
SELECT id FROM employees WHERE name = 'xZs';
```

**Output:**

```
Bitmap Index Scan on employees_name
  (actual time=0.045..0.047 rows=0 loops=1)
  
  Index Cond: (name = 'xZs'::text)
  
Execution Time: 47 ms ⚡
```

**Analysis:**

- ✅ Index used
- ✅ Fast (47 ms)
- ✅ Bitmap scan

---

## 🔥 Problem Query - LIKE With Wildcards

### Query With Leading Wildcard

```sql
EXPLAIN ANALYZE 
SELECT id FROM employees WHERE name LIKE '%Zs';
```

**Output:**

```
Gather (cost=1000.00..128750.11 rows=1100 width=4)
  (actual time=1456.234..1124.567 rows=1234 loops=1)
  
  Workers Planned: 2
  Workers Launched: 2
  
  -> Parallel Seq Scan on employees
       Filter: (name ~~ '%Zs'::text)    ← LIKE converted to ~~
       Rows Removed by Filter: 3666667
       
Execution Time: 1124 ms  ← SLOW! 😱
```

**Shock Analysis:**

| Aspect | Details |
|--------|---------|
| **Scan Type** | Parallel Sequential Scan |
| **Index Used?** | ❌ NO! Despite index existing |
| **Time** | 1124 ms (very slow) |
| **Rows Scanned** | 11 million (entire table!) |

---

## 🧩 Why Index Not Used?

### LIKE Pattern Variations

```sql
-- Pattern 1: Leading wildcard
WHERE name LIKE '%Zs'
-- Matches: 'abcZs', 'xyzZs', '123Zs', 'Zs'

-- Pattern 2: Trailing wildcard  
WHERE name LIKE 'Zs%'
-- Matches: 'Zs', 'Zs123', 'Zsabc'

-- Pattern 3: Both sides wildcard
WHERE name LIKE '%Zs%'  
-- Matches: 'aZsb', 'xZsy', '1Zs2', anything with Zs
```

### B-Tree Index Structure

```
B-Tree Index organized alphabetically:

        Root
         / \
      'A-M' 'N-Z'
       /      \
    'A-F'    'N-T' 'U-Z'
     /         |      \
 'aaa'     'nnn'    'zzz'
 'abc'     'pqr'    
 'aZs'     'xyz'    
```

**For LIKE '%Zs':**

- Index doesn't know WHERE to start!
- Could be 'aZs', 'bZs', 'cZs'... anywhere
- **No choice but full table scan**

**For LIKE 'Zs%':**

- Index knows: Start at 'Zs'
- ✅ **Index CAN be used!**

---

## 📊 LIKE Patterns - Index Usage

| Pattern | Index Used? | Reason | Performance |
|---------|-------------|--------|-------------|
| `LIKE 'abc'` | ✅ Yes | Same as `= 'abc'` | Fast ⚡ |
| `LIKE 'abc%'` | ✅ Yes | Starts with 'abc' - index knows where | Fast ⚡ |
| `LIKE '%abc'` | ❌ No | Ends with 'abc' - could be anywhere | Slow 🐢 |
| `LIKE '%abc%'` | ❌ No | Contains 'abc' - could be anywhere | Slow 🐢 |
| `LIKE '_abc'` | ❌ No (usually) | One char + 'abc' - inefficient | Slow 🐢 |

---

## 🎨 Visual Comparison

### Query 1: Trailing Wildcard (Index Works)

```sql
WHERE name LIKE 'xZ%'
```

```
┌──────────────────────────────────────────┐
│        B-Tree Index Lookup               │
├──────────────────────────────────────────┤
│                                          │
│  Start at 'xZ'                           │
│    ↓                                     │
│  'xZa', 'xZb', 'xZs', 'xZz'             │
│    ↑                                     │
│  Stop when prefix changes                │
│                                          │
│  Time: ~50 ms ⚡                         │
└──────────────────────────────────────────┘
```

### Query 2: Leading Wildcard (Index Fails)

```sql
WHERE name LIKE '%Zs'
```

```
┌──────────────────────────────────────────┐
│        Full Table Scan                   │
├──────────────────────────────────────────┤
│                                          │
│  Check every row:                        │
│    Row 1: 'abc' → No                     │
│    Row 2: 'xZs' → YES!                   │
│    Row 3: 'def' → No                     │
│    ...                                   │
│    Row 11M: 'pqr' → No                   │
│                                          │
│  Time: ~1100 ms 🐢                       │
└──────────────────────────────────────────┘
```

---

## 🔍 Real Example Analysis

Hussein's video lo demonstration:

### Before Index

```sql
EXPLAIN ANALYZE 
SELECT id FROM employees WHERE name LIKE '%Zs';
```

**Time:** 1100 ms

### After Creating Index

```sql
CREATE INDEX employees_name ON employees(name);

EXPLAIN ANALYZE 
SELECT id FROM employees WHERE name LIKE '%Zs';
```

**Time:** Still 1100 ms! 😱

**Hussein's Reaction:**
> "Back to the slow query. Back to the exact same slow query. Guys, why?"

---

## 🧠 The Core Problem

### Regular Index (B-Tree) Structure

```sql
-- Index stores data like:
'aaa'
'abc'
'aZs'  ← Contains 'Zs' but sorted by first char
'bbb'
'bZs'  ← Also contains 'Zs'
'ccc'
...
'xZs'  ← Also contains 'Zs'
```

**For `LIKE '%Zs'`:**

- Database can't use index effectively
- Has to check ALL entries
- **Planner decides:** "Sequential scan easier than index scan!"

---

## 💡 Solutions for LIKE Pattern Matching

### Solution 1: Change Query Pattern (If Possible)

```sql
-- Instead of
WHERE name LIKE '%Zs'

-- Use (if business logic allows)
WHERE name LIKE 'Zs%'  -- ✅ Index works!
```

### Solution 2: Full-Text Search Indexes (PostgreSQL)

For serious text search, use specialized indexes:

```sql
-- Create GIN index for pattern matching
CREATE INDEX employees_name_gin ON employees 
USING gin(name gin_trgm_ops);

-- Requires extension
CREATE EXTENSION pg_trgm;
```

**Now this works:**

```sql
EXPLAIN ANALYZE 
SELECT id FROM employees WHERE name LIKE '%Zs%';
```

**Output:**

```
Bitmap Heap Scan on employees
  -> Bitmap Index Scan on employees_name_gin
  
Execution Time: 125 ms  -- Much better than 1100 ms!
```

### Solution 3: Calculated Columns

```sql
-- Add reversed column for suffix searches
ALTER TABLE employees ADD COLUMN name_reversed TEXT;
UPDATE employees SET name_reversed = REVERSE(name);
CREATE INDEX idx_name_reversed ON employees(name_reversed);

-- Now for '%Zs' (ends with 'Zs')
SELECT id FROM employees 
WHERE name_reversed LIKE REVERSE('Zs') || '%';  -- 'sZ%'
```

### Solution 4: Use Full-Text Search (Best for Complex)

```sql
-- Add tsvector column
ALTER TABLE employees ADD COLUMN name_tsv tsvector;
UPDATE employees SET name_tsv = to_tsvector('english', name);
CREATE INDEX idx_name_tsv ON employees USING gin(name_tsv);

-- Search
SELECT id FROM employees WHERE name_tsv @@ to_tsquery('Zs');
```

---

## 📊 Performance Comparison

| Approach | Query | Time | Complexity |
|----------|-------|------|------------|
| **No index** | `LIKE '%Zs'` | 1100 ms | Low |
| **B-Tree index (fails)** | `LIKE '%Zs'` | 1100 ms | Low |
| **GIN trigram** | `LIKE '%Zs'` | 125 ms | Medium |
| **Reversed column** | `LIKE 'sZ%'` on reversed | 45 ms | Medium |
| **Full-text search** | `@@` operator | 30 ms | High |

---

## 🎯 Hussein's Key Quote

> [!WARNING]
> "Despite us having an index guys, despite having an index... We couldn't do inlining. We couldn't do any optimizations. This is an **expression**, obviously. So that's why we had to do all that stuff."

**Lesson:** Index exists ≠ Index used!

---

## 🛠️ Best Practices

### DO ✅

1. **Use trailing wildcards when possible**

   ```sql
   WHERE name LIKE 'prefix%'  -- Good!
   ```

2. **Check EXPLAIN before deploying**

   ```sql
   EXPLAIN ANALYZE your_query;  -- Always!
   ```

3. **Use appropriate index types**
   - B-Tree: Exact matches, prefix searches
   - GIN: Full-text, pattern matching
   - GiST: Geometric data, full-text

4. **Avoid wildcards for WHERE clauses**

   ```sql
   -- Instead of LIKE, use full-text search for complex patterns
   ```

### DON'T ❌

1. **Assume index will always be used**

   ```sql
   -- Just because index exists doesn't mean it's used!
   ```

2. **Use leading wildcards carelessly**

   ```sql
   WHERE name LIKE '%pattern'  -- Slow!
   ```

3. **Create indexes blindly**

   ```sql
   -- Always test performance before/after
   ```

---

## 🧪 Testing Your Queries

### Step-by-Step Checklist

```sql
-- 1. Check if index exists
\d tablename

-- 2. Test query performance
EXPLAIN ANALYZE SELECT ... WHERE name LIKE '...';

-- 3. Look for "Seq Scan" in output
-- If present, index NOT used

-- 4. Try alternative query patterns
-- trailing wildcard, GIN index, etc.

-- 5. Re-test and compare
```

---

## 🧠 Key Takeaways

1. **LIKE '%pattern'** (leading wildcard) **cannot use B-Tree index**
2. **LIKE 'pattern%'** (trailing wildcard) **can use B-Tree index**
3. Index existence ≠ Index usage (planner decides)
4. For complex pattern matching, use **GIN trigram indexes**
5. Always **EXPLAIN ANALYZE** to verify index usage
6. Database planner is smart - trusts its decisions

---

## ❓ Common Questions

**Q: Leading wildcard kabatti index pointless aa?**
> A: B-Tree index ki pointless. Kaani GIN trigram index use cheste work avtundi!

**Q: Case-insensitive search ela?**
>
> ```sql
> WHERE LOWER(name) LIKE LOWER('%pattern%')
> -- Kaani index on LOWER(name) create cheyyali!
> CREATE INDEX idx_name_lower ON employees(LOWER(name));
> ```

**Q: Multiple wildcards unte?**
>
> ```sql
> LIKE '%a%b%c%'  -- Chala slow, even with GIN
> -- Better: Full-text search with multiple terms
> ```

**Q: _underscore wildcard (single char) index use chesthunda?**
>
> ```sql
> LIKE '_abc'  -- Usually NO for B-Tree
> LIKE 'abc_'  -- Maybe YES (prefix match)
> ```

---

## ➡️ Next Lesson

**Coming up:**

- **Multi-column indexes** for complex queries
- **Partial indexes** for specific conditions
- **Index maintenance** - VACUUM, REINDEX
