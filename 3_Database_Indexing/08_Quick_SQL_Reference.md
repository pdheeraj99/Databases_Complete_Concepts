# 📚 Lesson 8: Quick SQL Reference for Hussein's Video

## 🎯 All Commands from the Video - Copy-Paste Ready

### Docker Setup

```bash
# Start PostgreSQL container
docker run --name pg -e POSTGRES_PASSWORD=postgres -d postgres

# If container already exists, start it
docker start pg

# Access PostgreSQL shell
docker exec -it pg psql -U postgres
```

---

## 🗃️ Table Creation

```sql
-- Create employees table
CREATE TABLE employees(
    id SERIAL PRIMARY KEY, 
    name TEXT
);
```

---

## 🔧 Random String Function

```sql
-- Create function to generate random strings
CREATE OR REPLACE FUNCTION random_string(length INTEGER) RETURNS TEXT AS 
$$
DECLARE
  chars TEXT[] := '{0,1,2,3,4,5,6,7,8,9,A,B,C,D,E,F,G,H,I,J,K,L,M,N,O,P,Q,R,S,T,U,V,W,X,Y,Z,a,b,c,d,e,f,g,h,i,j,k,l,m,n,o,p,q,r,s,t,u,v,w,x,y,z}';
  result TEXT := '';
  i INTEGER := 0;
  length2 INTEGER := (SELECT TRUNC(RANDOM() * length + 1));
BEGIN
  IF length2 < 0 THEN
    RAISE EXCEPTION 'Given length cannot be less than 0';
  END IF;
  FOR i IN 1..length2 LOOP
    result := result || chars[1+RANDOM()*(ARRAY_LENGTH(chars, 1)-1)];
  END LOOP;
  RETURN result;
END;
$$ LANGUAGE plpgsql;
```

---

## 📊 Insert Million Rows

```sql
-- Insert 1 million+ rows
INSERT INTO employees(name)
(SELECT random_string(10) FROM generate_series(0, 1000000));

-- Check count
SELECT COUNT(*) FROM employees;
-- Output: 1000001
```

---

## 🔍 Query Examples

### Example 1: Query ID (Indexed Column)

```sql
-- Select ID where ID = 2000 (inline query)
EXPLAIN ANALYZE 
SELECT id FROM employees WHERE id = 2000;
```

**Expected Output:**

```
Index Scan using employees_pkey on employees
  (actual time=0.045..0.047 rows=1 loops=1)
  Index Cond: (id = 2000)
Execution Time: 0.6 ms
```

### Example 2: Query Name (Get Name from ID)

```sql
-- Select name where ID = 5000 (heap fetch required)
EXPLAIN ANALYZE 
SELECT name FROM employees WHERE id = 5000;
```

**Expected Output:**

```
Index Scan using employees_pkey on employees
  (actual time=0.102..2.567 rows=1 loops=1)
  Index Cond: (id = 5000)
  Heap Fetches: 1
Execution Time: 2.5 ms
```

### Example 3: Query by Name (No Index - Slow!)

```sql
-- Before creating index on name
EXPLAIN ANALYZE 
SELECT id FROM employees WHERE name = 'xZs';
```

**Expected Output:**

```
Gather (cost=1000.00..128750.11 rows=1 width=4)
  (actual time=1456.234..3024.567 rows=0 loops=1)
  Workers Planned: 2
  Workers Launched: 2
  -> Parallel Seq Scan on employees
       Filter: (name = 'xZs'::text)
       Rows Removed by Filter: 11000001
Planning Time: 0.834 ms
Execution Time: 3024 ms
```

---

## 🏗️ Create Index

```sql
-- Create index on name column
CREATE INDEX employees_name ON employees(name);

-- Or more descriptive name
CREATE INDEX idx_employees_name ON employees(name);
```

**Note:** This will take several minutes for 11M rows!

---

## ⚡ After Index Creation

```sql
-- Same query, now with index
EXPLAIN ANALYZE 
SELECT id FROM employees WHERE name = 'xZs';
```

**Expected Output:**

```
Bitmap Heap Scan on employees
  (cost=4.44..8.46 rows=1 width=4)
  (actual time=0.045..0.047 rows=0 loops=1)
  Recheck Cond: (name = 'xZs'::text)
  -> Bitmap Index Scan on employees_name
       (cost=0.00..4.44 rows=1 width=0)
       (actual time=0.042..0.042 rows=0 loops=1)
       Index Cond: (name = 'xZs'::text)
Execution Time: 47 ms
```

**Improvement: 3024 ms → 47 ms = 64x faster!**

---

## 🔥 LIKE Operator Examples

### LIKE with Wildcards (Index Fails!)

```sql
-- This will NOT use index (leading wildcard)
EXPLAIN ANALYZE 
SELECT id FROM employees WHERE name LIKE '%Zs';
```

**Output:**

```
Parallel Seq Scan on employees
  Filter: (name ~~ '%Zs'::text)
Execution Time: 1124 ms  -- Still slow despite index!
```

### LIKE with Trailing Wildcard (Index Works!)

```sql
-- This WILL use index
EXPLAIN ANALYZE 
SELECT id FROM employees WHERE name LIKE 'xZ%';
```

**Output:**

```
Bitmap Index Scan on employees_name
  Index Cond: ((name >= 'xZ'::text) AND (name < 'xZ{'::text))
Execution Time: 45 ms  -- Fast!
```

---

## 📊 Useful Utility Commands

### List All Indexes

```sql
-- View indexes on employees table
\d employees

-- Or via query
SELECT 
    indexname, 
    indexdef 
FROM pg_indexes 
WHERE tablename = 'employees';
```

### Check Table Size

```sql
SELECT 
    pg_size_pretty(pg_total_relation_size('employees')) AS total_size,
    pg_size_pretty(pg_relation_size('employees')) AS table_size,
    pg_size_pretty(pg_indexes_size('employees')) AS indexes_size;
```

### Check Index Usage Statistics

```sql
SELECT 
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
WHERE tablename = 'employees';
```

---

## 🧪 Performance Test Script

```sql
-- Complete test script

-- 1. Before index
\timing on

SELECT id FROM employees WHERE name = 'testValue';
-- Note the time

-- 2. Create index
CREATE INDEX idx_employees_name ON employees(name);

-- 3. Update statistics
ANALYZE employees;

-- 4. After index
SELECT id FROM employees WHERE name = 'testValue';
-- Compare time

-- 5. Show execution plan
EXPLAIN ANALYZE SELECT id FROM employees WHERE name = 'testValue';
```

---

## 🎯 Key Queries to Remember

### Check Primary Key Index

```sql
SELECT * FROM pg_indexes 
WHERE tablename = 'employees' 
AND indexname LIKE '%pkey%';
```

### Drop Index (if needed)

```sql
DROP INDEX employees_name;
-- Or
DROP INDEX idx_employees_name;
```

### Recreate Index Concurrently (Production)

```sql
-- Safe for production (no table lock)
CREATE INDEX CONCURRENTLY idx_employees_name ON employees(name);
```

### Update Table Statistics

```sql
-- After bulk inserts or creating indexes
ANALYZE employees;
```

---

## 🧠 Quick Troubleshooting

### Index Not Being Used?

```sql
-- 1. Check if index exists
\d employees

-- 2. Update statistics
ANALYZE employees;

-- 3. Check query plan
EXPLAIN ANALYZE SELECT ... ;

-- 4. Check if query pattern suitable
-- LIKE '%pattern' won't use B-Tree index
-- Use LIKE 'pattern%' instead
```

### Query Still Slow After Index?

```sql
-- Check if returning too many rows
EXPLAIN ANALYZE SELECT ... ;
-- Look at "rows" in output

-- If returning >30% of table, seq scan might be faster
-- Consider:
-- 1. More selective WHERE clause
-- 2. Partial index
-- 3. Different query approach
```

---

## 📋 Complete Setup Checklist

```bash
# Terminal 1: Start Docker
docker run --name pg -e POSTGRES_PASSWORD=postgres -d postgres
docker exec -it pg psql -U postgres
```

```sql
-- Terminal 1: PostgreSQL Shell

-- 1. Create table
CREATE TABLE employees(id SERIAL PRIMARY KEY, name TEXT);

-- 2. Create function
CREATE OR REPLACE FUNCTION random_string(length INTEGER) RETURNS TEXT AS 
$$
DECLARE
  chars TEXT[] := '{0,1,2,3,4,5,6,7,8,9,A,B,C,D,E,F,G,H,I,J,K,L,M,N,O,P,Q,R,S,T,U,V,W,X,Y,Z,a,b,c,d,e,f,g,h,i,j,k,l,m,n,o,p,q,r,s,t,u,v,w,x,y,z}';
  result TEXT := '';
  i INTEGER := 0;
  length2 INTEGER := (SELECT TRUNC(RANDOM() * length + 1));
BEGIN
  IF length2 < 0 THEN
    RAISE EXCEPTION 'Given length cannot be less than 0';
  END IF;
  FOR i IN 1..length2 LOOP
    result := result || chars[1+RANDOM()*(ARRAY_LENGTH(chars, 1)-1)];
  END LOOP;
  RETURN result;
END;
$$ LANGUAGE plpgsql;

-- 3. Insert data (will take time)
INSERT INTO employees(name)(SELECT random_string(10) FROM generate_series(0, 1000000));

-- 4. Verify
SELECT COUNT(*) FROM employees;
SELECT * FROM employees LIMIT 5;

-- 5. Test queries
\timing on
EXPLAIN ANALYZE SELECT id FROM employees WHERE name = 'xyz';

-- 6. Create index
CREATE INDEX idx_employees_name ON employees(name);

-- 7. Test again
EXPLAIN ANALYZE SELECT id FROM employees WHERE name = 'xyz';
```

---

## 🎓 Summary of Hussein's Demo

| Step | Action | Time | Scan Type |
|------|--------|------|-----------|
| 1 | Query by ID (PK) | 0.6 ms | Index Scan |
| 2 | Query name by ID | 2.5 ms | Index Scan + Heap Fetch |
| 3 | Query by name (no index) | 3024 ms | Parallel Seq Scan |
| 4 | Create index on name | ~5 min | - |
| 5 | Query by name (with index) | 47 ms | Bitmap Index Scan |
| 6 | Query with LIKE '%x' | 1124 ms | Seq Scan (index not used!) |

**Key Lesson:** Index exists ≠ Index used. Query pattern matters!

---

## ➡️ Next Steps

Practice these queries yourself to internalize the concepts!
