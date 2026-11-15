# Round 6: PostgreSQL Database Deep Dive

## **Overview**
- **Duration**: 45-60 minutes
- **Format**: Video call, may include live SQL or discussion
- **Goal**: Evaluate SQL expertise, schema design, optimization, and database concepts
- **Interviewer**: Senior backend engineer or DBA
- **Level**: Medium to Hard

---

## **SQL Fundamentals & Advanced Queries**

### **1. Basic & Intermediate Queries**

```sql
-- SELECT with multiple conditions
SELECT id, name, email, age 
FROM users 
WHERE age > 18 AND status = 'active'
ORDER BY created_at DESC
LIMIT 10 OFFSET 0;

-- JOINs
-- INNER JOIN: only matching rows
SELECT u.id, u.name, o.id, o.total_price
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- LEFT JOIN: all from left table + matching from right
SELECT u.id, u.name, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id
HAVING COUNT(o.id) > 5;

-- Self JOIN
SELECT e1.id, e1.name, e2.name as manager_name
FROM employees e1
LEFT JOIN employees e2 ON e1.manager_id = e2.id;

-- Subquery
SELECT * FROM users 
WHERE id IN (
  SELECT user_id FROM orders 
  WHERE total_price > 1000
);

-- CTE (Common Table Expression)
WITH high_value_orders AS (
  SELECT user_id, COUNT(*) as order_count, SUM(total_price) as total
  FROM orders
  WHERE created_at > NOW() - INTERVAL '30 days'
  GROUP BY user_id
  HAVING SUM(total_price) > 5000
)
SELECT u.*, hvo.order_count, hvo.total
FROM users u
JOIN high_value_orders hvo ON u.id = hvo.user_id;

-- Window functions
SELECT 
  id, name, salary,
  AVG(salary) OVER (PARTITION BY department) as dept_avg,
  ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) as rank
FROM employees;

-- RANK vs DENSE_RANK vs ROW_NUMBER
SELECT 
  score,
  ROW_NUMBER() OVER (ORDER BY score DESC) as row_num,
  RANK() OVER (ORDER BY score DESC) as rank,
  DENSE_RANK() OVER (ORDER BY score DESC) as dense_rank
FROM scores;

-- UNION (removes duplicates) vs UNION ALL (keeps duplicates)
SELECT id, name FROM users
UNION
SELECT id, title FROM products;

-- EXCEPT: rows in first query but not in second
SELECT user_id FROM orders
EXCEPT
SELECT user_id FROM refunds;

-- INTERSECT: rows in both queries
SELECT user_id FROM subscribers
INTERSECT
SELECT user_id FROM active_users;
```

### **2. Aggregate & Group By**

```sql
-- COUNT, SUM, AVG, MIN, MAX
SELECT 
  category,
  COUNT(*) as product_count,
  AVG(price) as avg_price,
  SUM(stock) as total_stock,
  MIN(price) as min_price,
  MAX(price) as max_price
FROM products
GROUP BY category
HAVING COUNT(*) > 5
ORDER BY avg_price DESC;

-- STRING_AGG (PostgreSQL)
SELECT 
  category,
  STRING_AGG(DISTINCT name, ', ') as product_names
FROM products
GROUP BY category;

-- ARRAY_AGG
SELECT 
  user_id,
  ARRAY_AGG(product_id) as purchased_products
FROM orders
GROUP BY user_id;
```

### **3. Full-Text Search (PostgreSQL)**

```sql
-- Create full-text search index
CREATE INDEX idx_products_search ON products USING GIN (to_tsvector('english', name || ' ' || description));

-- Full-text search query
SELECT id, name, ts_rank(to_tsvector('english', name || ' ' || description), query) as rank
FROM products, 
  plainto_tsquery('english', 'laptop computer') as query
WHERE to_tsvector('english', name || ' ' || description) @@ query
ORDER BY rank DESC;

-- Phrase search
SELECT * FROM products 
WHERE to_tsvector('english', description) @@ phraseto_tsquery('english', 'high performance');
```

---

## **Schema Design**

### **1. Normalization**

```sql
-- BAD: Denormalized, data redundancy
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INT,
  user_name VARCHAR(100),
  user_email VARCHAR(100),
  product_id INT,
  product_name VARCHAR(100),
  product_price DECIMAL(10,2),
  quantity INT
);

-- GOOD: Normalized (3NF)
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  price DECIMAL(10,2) NOT NULL,
  category_id INT REFERENCES categories(id)
);

CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INT NOT NULL REFERENCES users(id),
  total_price DECIMAL(10,2),
  status VARCHAR(50),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE order_items (
  id SERIAL PRIMARY KEY,
  order_id INT NOT NULL REFERENCES orders(id),
  product_id INT NOT NULL REFERENCES products(id),
  quantity INT NOT NULL,
  unit_price DECIMAL(10,2) NOT NULL
);
```

**Normalization Forms:**
- **1NF**: Atomic values (no repeating groups)
- **2NF**: 1NF + all non-key attributes depend on entire key
- **3NF**: 2NF + no transitive dependencies
- **BCNF**: Stricter version of 3NF

### **2. Key Types**

```sql
-- Primary Key: uniquely identifies each row
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  ...
);

-- Alternative: UUID
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  ...
);

-- Foreign Key: reference another table
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  ...
);

-- Composite Primary Key
CREATE TABLE order_items (
  order_id INT NOT NULL REFERENCES orders(id),
  product_id INT NOT NULL REFERENCES products(id),
  quantity INT NOT NULL,
  PRIMARY KEY (order_id, product_id)
);

-- Unique constraint
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(100) UNIQUE NOT NULL,
  username VARCHAR(50) UNIQUE NOT NULL
);

-- Composite unique constraint
CREATE TABLE user_follows (
  follower_id INT NOT NULL REFERENCES users(id),
  following_id INT NOT NULL REFERENCES users(id),
  UNIQUE(follower_id, following_id)
);
```

### **3. Data Types**

```sql
-- Numeric
SMALLINT (16-bit), INT (32-bit), BIGINT (64-bit)
DECIMAL(precision, scale)  -- e.g., DECIMAL(10,2) for money
FLOAT, DOUBLE PRECISION
SERIAL, BIGSERIAL  -- Auto-incrementing

-- String
CHAR(n)           -- Fixed length
VARCHAR(n)        -- Variable length
TEXT              -- Unlimited length

-- Date/Time
DATE              -- Only date
TIME              -- Only time
TIMESTAMP         -- Date + time with timezone
INTERVAL          -- Duration

-- Boolean
BOOLEAN           -- true/false

-- JSON (PostgreSQL)
JSON              -- Slower but exact representation
JSONB             -- Faster, deduplicated

-- Arrays (PostgreSQL)
INT[] or INT ARRAY
VARCHAR[] or VARCHAR ARRAY

-- UUID (PostgreSQL)
UUID

-- Enums
CREATE TYPE user_status AS ENUM ('active', 'inactive', 'banned');
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  status user_status DEFAULT 'active'
);
```

---

## **Indexing & Performance**

### **1. Index Types**

```sql
-- B-tree index (default, best for most cases)
CREATE INDEX idx_users_email ON users(email);

-- Query using index
SELECT * FROM users WHERE email = 'test@example.com';

-- Composite index (for queries with multiple conditions)
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at);

-- Query using composite index
SELECT * FROM orders 
WHERE user_id = 1 AND created_at > '2024-01-01';

-- Partial index (only index certain rows)
CREATE INDEX idx_active_users ON users(email) WHERE status = 'active';

-- Hash index (good for equality)
CREATE INDEX idx_product_id ON order_items USING HASH(product_id);

-- BRIN index (Block Range Index, good for large tables with correlation)
CREATE INDEX idx_orders_created ON orders USING BRIN(created_at);

-- GIN index (for JSON, arrays, full-text search)
CREATE INDEX idx_jsonb_data ON tables USING GIN(data);

-- GIST index (for geometric data, full-text search)
CREATE INDEX idx_users_search ON users USING GIST(to_tsvector('english', name));
```

### **2. Query Optimization**

```sql
-- EXPLAIN to see query plan
EXPLAIN ANALYZE
SELECT * FROM orders 
WHERE user_id = 1 AND created_at > '2024-01-01';

-- Look for:
-- - Sequential Scan vs Index Scan
-- - Nested Loops vs Hash Join vs Merge Join
-- - Estimated vs Actual rows

-- Optimize slow query example:
-- BEFORE (Slow: full table scan)
SELECT * FROM orders WHERE total_price * quantity > 1000;

-- AFTER (Faster: pre-calculated column)
ALTER TABLE orders ADD COLUMN line_total DECIMAL(10,2);
CREATE INDEX idx_orders_line_total ON orders(line_total);
SELECT * FROM orders WHERE line_total > 1000;

-- Using LIMIT with pagination
SELECT * FROM products 
ORDER BY created_at DESC
LIMIT 20 OFFSET 100000; -- Slow for large offset

-- Better: use WHERE instead of OFFSET
SELECT * FROM products 
WHERE created_at < (SELECT created_at FROM products ORDER BY created_at DESC LIMIT 1 OFFSET 100000)
ORDER BY created_at DESC
LIMIT 20;
```

### **3. Connection Pooling**

```sql
-- Check current connections
SELECT * FROM pg_stat_activity;

-- Connection pooling with PgBouncer config
[databases]
mydb = host=localhost port=5432 dbname=mydb

[pgbouncer]
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 25
min_pool_size = 10
reserve_pool_size = 5
reserve_pool_timeout = 3
```

---

## **Transactions & ACID**

### **1. Transaction Basics**

```sql
-- Explicit transaction
BEGIN;
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;

-- Or ROLLBACK on error
BEGIN;
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  SAVEPOINT sp1;
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;
  -- If error occurs:
  ROLLBACK TO sp1;
COMMIT;
```

### **2. Isolation Levels**

```sql
-- From lowest to highest isolation
-- 1. READ UNCOMMITTED (reads dirty data)
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

-- 2. READ COMMITTED (default, prevents dirty reads)
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- 3. REPEATABLE READ (prevents non-repeatable reads)
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- 4. SERIALIZABLE (highest, serializes transactions)
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

### **3. Locking**

```sql
-- Row-level lock
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
-- Update can proceed, other transactions wait
UPDATE accounts SET balance = balance + 100 WHERE id = 1;
COMMIT;

-- Advisory locks (application-level)
SELECT pg_advisory_lock(1);
-- Do something critical
SELECT pg_advisory_unlock(1);
```

---

## **Advanced Features**

### **1. Views & Materialized Views**

```sql
-- Regular view (query executed each time)
CREATE VIEW active_users_view AS
SELECT id, name, email 
FROM users 
WHERE status = 'active';

-- Query the view
SELECT * FROM active_users_view;

-- Materialized view (data stored, needs refresh)
CREATE MATERIALIZED VIEW user_order_summary AS
SELECT 
  u.id,
  u.name,
  COUNT(o.id) as order_count,
  SUM(o.total_price) as total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id;

-- Refresh materialized view
REFRESH MATERIALIZED VIEW user_order_summary;
REFRESH MATERIALIZED VIEW CONCURRENTLY user_order_summary;
```

### **2. Triggers**

```sql
-- Audit log example
CREATE TABLE audit_log (
  id SERIAL PRIMARY KEY,
  table_name VARCHAR(50),
  operation VARCHAR(10),
  changed_data JSONB,
  changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE OR REPLACE FUNCTION audit_trigger()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO audit_log (table_name, operation, changed_data)
  VALUES (TG_TABLE_NAME, TG_OP, row_to_json(NEW));
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER users_audit_trigger
AFTER INSERT OR UPDATE OR DELETE ON users
FOR EACH ROW
EXECUTE FUNCTION audit_trigger();
```

### **3. Partitioning**

```sql
-- Range partitioning (by date)
CREATE TABLE orders (
  id BIGSERIAL,
  user_id INT,
  created_at DATE,
  total_price DECIMAL(10,2)
) PARTITION BY RANGE (created_at);

CREATE TABLE orders_2024_q1 PARTITION OF orders
  FOR VALUES FROM ('2024-01-01') TO ('2024-04-01');

CREATE TABLE orders_2024_q2 PARTITION OF orders
  FOR VALUES FROM ('2024-04-01') TO ('2024-07-01');

-- List partitioning (by values)
CREATE TABLE orders_by_region (
  id SERIAL,
  region VARCHAR(50),
  total_price DECIMAL(10,2)
) PARTITION BY LIST (region);

CREATE TABLE orders_north PARTITION OF orders_by_region
  FOR VALUES IN ('north', 'northeast');

CREATE TABLE orders_south PARTITION OF orders_by_region
  FOR VALUES IN ('south', 'southeast');

-- Hash partitioning
CREATE TABLE order_details (
  id SERIAL,
  order_id INT,
  details JSONB
) PARTITION BY HASH (order_id);
```

---

## **Common Interview Problems**

### **1. N+1 Query Problem**

```sql
-- PROBLEM: Multiple queries
SELECT * FROM users;  -- Query 1
-- Then for each user:
SELECT * FROM orders WHERE user_id = ?;  -- Query N+1

-- SOLUTION: Use JOIN
SELECT u.*, o.*
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;

-- Or aggregation
SELECT u.id, u.name, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id;
```

### **2. Finding Slow Queries**

```sql
-- Enable query logging
ALTER SYSTEM SET log_min_duration_statement = 1000; -- Log queries > 1 second
SELECT pg_reload_conf();

-- Find slow queries
SELECT query, calls, mean_exec_time, max_exec_time
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;
```

### **3. Duplicate Removal**

```sql
-- Find duplicates
SELECT email, COUNT(*) 
FROM users 
GROUP BY email 
HAVING COUNT(*) > 1;

-- Remove duplicates, keeping oldest
DELETE FROM users u1
WHERE id NOT IN (
  SELECT MIN(id) FROM users u2 
  WHERE u2.email = u1.email
  GROUP BY email
);
```

---

## **What They're Evaluating**

✅ SQL query writing ability
✅ Schema design understanding
✅ Indexing strategy
✅ Query optimization skills
✅ Understanding of transactions & isolation
✅ PostgreSQL-specific features knowledge
✅ Performance problem-solving
✅ Normalization understanding

---

## **Common Mistakes to Avoid**

❌ Not using indexes for frequently queried columns
❌ Using SELECT * instead of specific columns
❌ Not joining tables, using N+1 queries
❌ Over-normalizing (leading to complex queries)
❌ Ignoring query execution plans
❌ Not using appropriate data types
❌ Forgetting about NULL handling
❌ Not considering connection pooling

---

## **Tips**

✅ **Use EXPLAIN**: Understand query plans before optimizing
✅ **Index strategically**: Don't index every column
✅ **Normalize appropriately**: Balance between performance and consistency
✅ **Monitor**: Use pg_stat_statements to find real bottlenecks
✅ **Consider partitioning**: For very large tables
✅ **Connection pooling**: Essential in production
✅ **Test with realistic data**: Small datasets don't show real performance issues
✅ **Backup strategy**: Essential for production databases

---

## **PostgreSQL Advantages**

- ACID compliance
- Advanced indexing options
- Full-text search
- JSONB support
- Window functions
- CTEs (WITH clause)
- Excellent query planner
- Strong community
