# Database Interview Questions

## Table of Contents
1. [SQL Basics](#sql-basics)
2. [Primary Key vs Foreign Key](#primary-key-vs-foreign-key)
3. [Joins](#joins)
4. [Subqueries](#subqueries)
5. [Indexes](#indexes)
6. [Transactions and ACID](#transactions-and-acid)
7. [Normalization](#normalization)
8. [NoSQL Types](#nosql-types)
9. [MongoDB Queries](#mongodb-queries)
10. [Redis](#redis)
11. [When to Use SQL vs NoSQL](#when-to-use-sql-vs-nosql)
12. [Advanced Topics](#advanced-topics)

---

## SQL Basics

### Q1: What is SQL?

**Answer:**
SQL (Structured Query Language) is a standardized programming language used to manage and manipulate relational databases. It allows you to:
- Query data (SELECT)
- Insert data (INSERT)
- Update data (UPDATE)
- Delete data (DELETE)
- Create and modify database structures (CREATE, ALTER, DROP)
- Control access (GRANT, REVOKE)

**Categories of SQL Commands:**
- **DDL (Data Definition Language)**: CREATE, ALTER, DROP, TRUNCATE
- **DML (Data Manipulation Language)**: SELECT, INSERT, UPDATE, DELETE
- **DCL (Data Control Language)**: GRANT, REVOKE
- **TCL (Transaction Control Language)**: COMMIT, ROLLBACK, SAVEPOINT

---

### Q2: WHERE vs HAVING - What's the Difference?

**Answer:**

| Aspect | WHERE | HAVING |
|--------|-------|--------|
| **Purpose** | Filters rows before grouping | Filters groups after grouping |
| **Used with** | Individual rows | Aggregated data (GROUP BY) |
| **Aggregate functions** | Cannot use aggregate functions | Can use aggregate functions |
| **Execution order** | Executes before GROUP BY | Executes after GROUP BY |
| **Performance** | Generally faster | Slower (operates on grouped data) |

**Example:**

```sql
-- Sample table: employees
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    department VARCHAR(50),
    salary DECIMAL(10,2)
);

INSERT INTO employees VALUES
(1, 'John', 'IT', 60000),
(2, 'Jane', 'IT', 70000),
(3, 'Bob', 'HR', 50000),
(4, 'Alice', 'HR', 55000),
(5, 'Charlie', 'IT', 65000);

-- WHERE: Filter before grouping
SELECT department, COUNT(*) as emp_count
FROM employees
WHERE salary > 55000  -- Filter individual rows
GROUP BY department;

-- Result:
-- IT: 3 employees (John, Jane, Charlie)
-- HR: 0 employees

-- HAVING: Filter after grouping
SELECT department, AVG(salary) as avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 60000;  -- Filter grouped results

-- Result:
-- IT: 65000 (average of 60000, 70000, 65000)

-- Combining WHERE and HAVING
SELECT department, AVG(salary) as avg_salary
FROM employees
WHERE salary > 50000           -- First: filter individual rows
GROUP BY department            -- Then: group the filtered data
HAVING AVG(salary) > 60000;    -- Finally: filter groups

-- Result: Only IT department with avg_salary 65000
```

**Real-World Scenario:**
```sql
-- Find departments where average salary exceeds $80,000
-- but only include employees hired after 2020
SELECT
    department,
    AVG(salary) as avg_salary,
    COUNT(*) as employee_count
FROM employees
WHERE hire_date > '2020-01-01'     -- WHERE: filter rows
GROUP BY department
HAVING AVG(salary) > 80000         -- HAVING: filter groups
ORDER BY avg_salary DESC;
```

---

### Q3: DELETE vs TRUNCATE vs DROP - What are the Differences?

**Answer:**

| Feature | DELETE | TRUNCATE | DROP |
|---------|--------|----------|------|
| **Type** | DML | DDL | DDL |
| **Purpose** | Remove specific rows | Remove all rows | Remove entire table |
| **WHERE clause** | Yes | No | No |
| **Rollback** | Can rollback (in transaction) | Cannot rollback (most DBs) | Cannot rollback |
| **Triggers** | Fires DELETE triggers | Does not fire triggers | Does not fire triggers |
| **Identity/Auto-increment** | Maintains counter | Resets counter | Removes counter |
| **Locks** | Row-level locks | Table-level lock | Removes object |
| **Speed** | Slower (logs each row) | Faster | Fastest |
| **Space** | May not reclaim space | Reclaims space | Reclaims all space |

**Examples:**

```sql
-- Sample table
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    status VARCHAR(20)
);

INSERT INTO users (name, status) VALUES
('John', 'active'),
('Jane', 'inactive'),
('Bob', 'active');

-- DELETE: Remove specific rows
DELETE FROM users WHERE status = 'inactive';
-- Result: Only Jane's row is deleted
-- Next insert will have id = 4

-- DELETE: Remove all rows (but keep structure)
DELETE FROM users;
-- Result: All rows deleted, table structure remains
-- Next insert will have id = 4 (counter not reset)

-- TRUNCATE: Remove all rows, reset auto-increment
TRUNCATE TABLE users;
-- Result: All rows deleted, table structure remains
-- Next insert will have id = 1 (counter reset)

-- DROP: Remove entire table
DROP TABLE users;
-- Result: Table no longer exists
-- Must recreate table before inserting
```

**Transaction Behavior:**

```sql
-- DELETE can be rolled back
BEGIN TRANSACTION;
DELETE FROM users WHERE id = 1;
SELECT * FROM users;  -- Row is gone
ROLLBACK;
SELECT * FROM users;  -- Row is back!

-- TRUNCATE usually cannot be rolled back (database-dependent)
BEGIN TRANSACTION;
TRUNCATE TABLE users;
SELECT * FROM users;  -- All rows gone
ROLLBACK;
SELECT * FROM users;  -- Still empty (in most databases)
```

**Real-World Scenarios:**

```sql
-- Scenario 1: Remove inactive users (use DELETE)
DELETE FROM users WHERE last_login < DATE_SUB(NOW(), INTERVAL 1 YEAR);

-- Scenario 2: Clear test data (use TRUNCATE)
TRUNCATE TABLE test_orders;

-- Scenario 3: Remove deprecated table (use DROP)
DROP TABLE legacy_customers;
```

---

### Q4: What are Constraints in SQL?

**Answer:**

Constraints enforce rules on data in tables to ensure data integrity.

**Types of Constraints:**

1. **NOT NULL**: Column cannot have NULL values
2. **UNIQUE**: All values in column must be unique
3. **PRIMARY KEY**: Uniquely identifies each row (NOT NULL + UNIQUE)
4. **FOREIGN KEY**: Links two tables together
5. **CHECK**: Ensures values meet specific condition
6. **DEFAULT**: Sets default value for column

```sql
CREATE TABLE employees (
    -- PRIMARY KEY: unique identifier
    id INT PRIMARY KEY AUTO_INCREMENT,

    -- NOT NULL: mandatory fields
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,

    -- UNIQUE: no duplicate emails
    email VARCHAR(100) UNIQUE,

    -- CHECK: age must be reasonable
    age INT CHECK (age >= 18 AND age <= 100),

    -- DEFAULT: default value if not specified
    status VARCHAR(20) DEFAULT 'active',
    hire_date DATE DEFAULT CURRENT_DATE,

    -- FOREIGN KEY: must reference existing department
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

-- Examples of constraint violations
INSERT INTO employees (first_name, last_name, email, age)
VALUES (NULL, 'Doe', 'john@example.com', 25);
-- Error: first_name cannot be NULL

INSERT INTO employees (first_name, last_name, email, age)
VALUES ('John', 'Doe', 'existing@example.com', 25);
-- Error: email must be unique (if already exists)

INSERT INTO employees (first_name, last_name, email, age)
VALUES ('John', 'Doe', 'john@example.com', 15);
-- Error: age must be >= 18

INSERT INTO employees (first_name, last_name, department_id)
VALUES ('John', 'Doe', 999);
-- Error: department_id 999 doesn't exist in departments table
```

---

## Primary Key vs Foreign Key

### Q5: What is the difference between Primary Key and Foreign Key?

**Answer:**

| Aspect | Primary Key | Foreign Key |
|--------|------------|-------------|
| **Purpose** | Uniquely identify a row | Create relationship between tables |
| **Uniqueness** | Must be unique | Can have duplicates |
| **NULL values** | Cannot be NULL | Can be NULL (unless NOT NULL specified) |
| **Quantity** | One per table | Multiple per table |
| **Index** | Automatically creates unique index | Creates regular index |
| **Values** | Independent | Must exist in referenced table |

**Visual Example:**

```sql
-- Primary Key Example
CREATE TABLE departments (
    department_id INT PRIMARY KEY,    -- Primary Key
    department_name VARCHAR(100) NOT NULL
);

INSERT INTO departments VALUES
(1, 'IT'),
(2, 'HR'),
(3, 'Sales');

-- Foreign Key Example
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,       -- Primary Key
    name VARCHAR(100),
    department_id INT,                 -- Foreign Key
    FOREIGN KEY (department_id) REFERENCES departments(department_id)
);

INSERT INTO employees VALUES
(101, 'John Doe', 1),      -- Valid: department 1 exists
(102, 'Jane Smith', 2),    -- Valid: department 2 exists
(103, 'Bob Wilson', 1);    -- Valid: duplicate department_id is OK

-- This will fail:
INSERT INTO employees VALUES
(104, 'Alice Brown', 99);  -- Error: department 99 doesn't exist

-- Composite Primary Key
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id),  -- Composite key
    FOREIGN KEY (order_id) REFERENCES orders(id),
    FOREIGN KEY (product_id) REFERENCES products(id)
);
```

**Real-World Scenario:**

```sql
-- E-commerce database design
CREATE TABLE customers (
    customer_id INT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(100) UNIQUE,
    name VARCHAR(100)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT NOT NULL,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    total_amount DECIMAL(10,2),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
        ON DELETE RESTRICT    -- Prevent deleting customer with orders
        ON UPDATE CASCADE     -- Update order if customer_id changes
);

CREATE TABLE order_items (
    item_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    price DECIMAL(10,2),
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
        ON DELETE CASCADE,    -- Delete items if order is deleted
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

-- Query using relationships
SELECT
    c.name as customer_name,
    o.order_id,
    o.order_date,
    SUM(oi.quantity * oi.price) as order_total
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY c.customer_id, o.order_id
ORDER BY o.order_date DESC;
```

**Foreign Key Actions:**

```sql
-- ON DELETE CASCADE: Delete child records when parent is deleted
-- ON DELETE RESTRICT: Prevent deletion of parent if children exist
-- ON DELETE SET NULL: Set foreign key to NULL when parent is deleted
-- ON DELETE NO ACTION: Same as RESTRICT

CREATE TABLE posts (
    id INT PRIMARY KEY,
    user_id INT,
    FOREIGN KEY (user_id) REFERENCES users(id)
        ON DELETE CASCADE  -- Delete all posts when user is deleted
);

CREATE TABLE comments (
    id INT PRIMARY KEY,
    post_id INT,
    FOREIGN KEY (post_id) REFERENCES posts(id)
        ON DELETE SET NULL  -- Keep comments but set post_id to NULL
);
```

---

## Joins

### Q6: Explain Different Types of Joins with Examples

**Answer:**

Joins combine rows from two or more tables based on a related column.

**Types of Joins:**
1. INNER JOIN
2. LEFT JOIN (LEFT OUTER JOIN)
3. RIGHT JOIN (RIGHT OUTER JOIN)
4. FULL OUTER JOIN
5. CROSS JOIN
6. SELF JOIN

**Sample Tables:**

```sql
-- Customers table
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100)
);

INSERT INTO customers VALUES
(1, 'John'),
(2, 'Jane'),
(3, 'Bob'),
(4, 'Alice');

-- Orders table
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    amount DECIMAL(10,2)
);

INSERT INTO orders VALUES
(101, 1, 100.00),
(102, 1, 150.00),
(103, 2, 200.00),
(104, 5, 75.00);   -- customer_id 5 doesn't exist in customers
```

**1. INNER JOIN**

Returns only matching rows from both tables.

```sql
SELECT
    c.customer_id,
    c.name,
    o.order_id,
    o.amount
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id;

-- Result:
-- 1, John,  101, 100.00
-- 1, John,  102, 150.00
-- 2, Jane,  103, 200.00
-- (Bob and Alice excluded - no orders)
-- (Order 104 excluded - no matching customer)
```

**Visual Representation:**
```
Customers: 1, 2, 3, 4
Orders:    1, 1, 2, 5
Result:    1, 1, 2     (intersection)
```

**2. LEFT JOIN (LEFT OUTER JOIN)**

Returns all rows from left table, matching rows from right table (NULL if no match).

```sql
SELECT
    c.customer_id,
    c.name,
    o.order_id,
    o.amount
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id;

-- Result:
-- 1, John,  101,  100.00
-- 1, John,  102,  150.00
-- 2, Jane,  103,  200.00
-- 3, Bob,   NULL, NULL     -- Bob has no orders
-- 4, Alice, NULL, NULL     -- Alice has no orders
```

**Use Case: Find customers with no orders**
```sql
SELECT c.customer_id, c.name
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;

-- Result:
-- 3, Bob
-- 4, Alice
```

**3. RIGHT JOIN (RIGHT OUTER JOIN)**

Returns all rows from right table, matching rows from left table (NULL if no match).

```sql
SELECT
    c.customer_id,
    c.name,
    o.order_id,
    o.amount
FROM customers c
RIGHT JOIN orders o ON c.customer_id = o.customer_id;

-- Result:
-- 1,    John,  101, 100.00
-- 1,    John,  102, 150.00
-- 2,    Jane,  103, 200.00
-- NULL, NULL,  104, 75.00    -- Order with no matching customer
```

**4. FULL OUTER JOIN**

Returns all rows from both tables (NULL where no match).

```sql
SELECT
    c.customer_id,
    c.name,
    o.order_id,
    o.amount
FROM customers c
FULL OUTER JOIN orders o ON c.customer_id = o.customer_id;

-- Result:
-- 1,    John,  101,  100.00
-- 1,    John,  102,  150.00
-- 2,    Jane,  103,  200.00
-- 3,    Bob,   NULL, NULL     -- Customer with no orders
-- 4,    Alice, NULL, NULL     -- Customer with no orders
-- NULL, NULL,  104,  75.00    -- Order with no customer

-- Note: MySQL doesn't support FULL OUTER JOIN directly
-- Workaround using UNION:
SELECT c.customer_id, c.name, o.order_id, o.amount
FROM customers c LEFT JOIN orders o ON c.customer_id = o.customer_id
UNION
SELECT c.customer_id, c.name, o.order_id, o.amount
FROM customers c RIGHT JOIN orders o ON c.customer_id = o.customer_id;
```

**5. CROSS JOIN**

Returns Cartesian product (all possible combinations).

```sql
SELECT
    c.name,
    o.order_id
FROM customers c
CROSS JOIN orders o;

-- Result: 4 customers × 4 orders = 16 rows
-- John,  101
-- John,  102
-- John,  103
-- John,  104
-- Jane,  101
-- Jane,  102
-- ... (16 total combinations)
```

**Use Case: Generate combinations**
```sql
-- Create a calendar of employee shifts
SELECT
    e.name as employee,
    d.day_name
FROM employees e
CROSS JOIN days d
ORDER BY e.name, d.day_id;
```

**6. SELF JOIN**

Join a table to itself.

```sql
-- Employee table with manager relationship
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    manager_id INT
);

INSERT INTO employees VALUES
(1, 'CEO', NULL),
(2, 'Manager A', 1),
(3, 'Manager B', 1),
(4, 'Employee 1', 2),
(5, 'Employee 2', 2);

-- Find each employee and their manager
SELECT
    e.name as employee,
    m.name as manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;

-- Result:
-- CEO,        NULL
-- Manager A,  CEO
-- Manager B,  CEO
-- Employee 1, Manager A
-- Employee 2, Manager A
```

**Real-World Complex Join Example:**

```sql
-- E-commerce order summary
SELECT
    c.name as customer_name,
    o.order_id,
    o.order_date,
    p.product_name,
    oi.quantity,
    oi.price,
    (oi.quantity * oi.price) as line_total
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
INNER JOIN order_items oi ON o.order_id = oi.order_id
INNER JOIN products p ON oi.product_id = p.product_id
WHERE o.order_date >= DATE_SUB(NOW(), INTERVAL 30 DAY)
ORDER BY o.order_date DESC, o.order_id, oi.item_id;
```

**Join Performance Tips:**

```sql
-- Bad: No index on join column
SELECT * FROM customers c
JOIN orders o ON c.email = o.customer_email;  -- Slow without index

-- Good: Indexed join columns
CREATE INDEX idx_customer_id ON orders(customer_id);
SELECT * FROM customers c
JOIN orders o ON c.customer_id = o.customer_id;  -- Fast with index

-- Use EXPLAIN to analyze joins
EXPLAIN SELECT * FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id;
```

---

## Subqueries

### Q7: What are Subqueries? Explain Different Types

**Answer:**

A subquery is a query nested inside another query. Also called inner query or nested query.

**Types of Subqueries:**
1. Scalar Subquery (returns single value)
2. Row Subquery (returns single row)
3. Column Subquery (returns single column)
4. Table Subquery (returns table)
5. Correlated Subquery
6. Nested Subquery

**Sample Tables:**

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    department VARCHAR(50),
    salary DECIMAL(10,2),
    manager_id INT
);

INSERT INTO employees VALUES
(1, 'John', 'IT', 60000, NULL),
(2, 'Jane', 'IT', 70000, 1),
(3, 'Bob', 'HR', 50000, NULL),
(4, 'Alice', 'HR', 55000, 3),
(5, 'Charlie', 'IT', 65000, 1),
(6, 'David', 'Sales', 80000, NULL);
```

**1. Scalar Subquery (Single Value)**

```sql
-- Find employees earning more than average salary
SELECT name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary) FROM employees  -- Returns single value: 63333.33
);

-- Result:
-- Jane,    70000
-- Charlie, 65000
-- David,   80000

-- Find employee with highest salary
SELECT name, salary
FROM employees
WHERE salary = (
    SELECT MAX(salary) FROM employees
);

-- Result: David, 80000
```

**2. Column Subquery (Multiple Values)**

```sql
-- Find employees in departments that have more than 2 people
SELECT name, department
FROM employees
WHERE department IN (
    SELECT department
    FROM employees
    GROUP BY department
    HAVING COUNT(*) > 2
);

-- Result:
-- John,    IT
-- Jane,    IT
-- Charlie, IT
```

**3. Table Subquery (Derived Table)**

```sql
-- Calculate department statistics
SELECT
    dept_stats.department,
    dept_stats.avg_salary,
    dept_stats.emp_count
FROM (
    SELECT
        department,
        AVG(salary) as avg_salary,
        COUNT(*) as emp_count
    FROM employees
    GROUP BY department
) as dept_stats
WHERE dept_stats.avg_salary > 60000;

-- Result:
-- IT, 65000, 3
```

**4. Correlated Subquery**

Subquery references the outer query (executes for each row).

```sql
-- Find employees earning more than their department average
SELECT e1.name, e1.department, e1.salary
FROM employees e1
WHERE e1.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department = e1.department  -- Correlated: references e1
);

-- Result:
-- Jane,  IT, 70000  (avg IT salary: 65000)
-- David, Sales, 80000  (only employee in Sales)

-- Find employees with no subordinates
SELECT e1.name
FROM employees e1
WHERE NOT EXISTS (
    SELECT 1
    FROM employees e2
    WHERE e2.manager_id = e1.id
);

-- Result: Jane, Bob, Alice, Charlie, David
```

**5. Nested Subqueries (Multiple Levels)**

```sql
-- Find employees in departments where
-- someone earns more than the company average
SELECT name, department, salary
FROM employees
WHERE department IN (
    SELECT department
    FROM employees
    WHERE salary > (
        SELECT AVG(salary)
        FROM employees
    )
);
```

**Subquery Operators:**

```sql
-- IN: Check if value exists in subquery results
SELECT name FROM employees
WHERE department IN (SELECT department FROM departments WHERE location = 'New York');

-- NOT IN: Check if value doesn't exist
SELECT name FROM employees
WHERE id NOT IN (SELECT employee_id FROM attendance WHERE date = '2025-10-04');

-- EXISTS: Check if subquery returns any rows
SELECT name FROM customers c
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id);

-- NOT EXISTS: Check if subquery returns no rows
SELECT name FROM customers c
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id);

-- ANY/SOME: Compare to any value returned by subquery
SELECT name FROM employees
WHERE salary > ANY (SELECT salary FROM employees WHERE department = 'HR');
-- (greater than at least one HR salary)

-- ALL: Compare to all values returned by subquery
SELECT name FROM employees
WHERE salary > ALL (SELECT salary FROM employees WHERE department = 'HR');
-- (greater than all HR salaries)
```

**Real-World Examples:**

```sql
-- Example 1: Find products never ordered
SELECT product_id, product_name
FROM products p
WHERE NOT EXISTS (
    SELECT 1
    FROM order_items oi
    WHERE oi.product_id = p.product_id
);

-- Example 2: Find top 3 most expensive products per category
SELECT category, product_name, price
FROM (
    SELECT
        category,
        product_name,
        price,
        ROW_NUMBER() OVER (PARTITION BY category ORDER BY price DESC) as rank
    FROM products
) ranked
WHERE rank <= 3;

-- Example 3: Find customers who spent more than $1000 in last month
SELECT c.name, c.email
FROM customers c
WHERE (
    SELECT SUM(total_amount)
    FROM orders o
    WHERE o.customer_id = c.customer_id
      AND o.order_date >= DATE_SUB(NOW(), INTERVAL 30 DAY)
) > 1000;

-- Example 4: Update salaries based on department average
UPDATE employees e
SET salary = salary * 1.10
WHERE salary < (
    SELECT AVG(salary)
    FROM (SELECT * FROM employees) e2
    WHERE e2.department = e.department
);
```

**Subquery vs JOIN Performance:**

```sql
-- Subquery approach
SELECT name FROM customers
WHERE id IN (SELECT customer_id FROM orders WHERE total > 1000);

-- JOIN approach (often faster)
SELECT DISTINCT c.name
FROM customers c
INNER JOIN orders o ON c.id = o.customer_id
WHERE o.total > 1000;

-- Use EXPLAIN to compare performance
EXPLAIN SELECT name FROM customers
WHERE id IN (SELECT customer_id FROM orders WHERE total > 1000);
```

**Common Pitfalls:**

```sql
-- Pitfall 1: Subquery returns NULL
SELECT name FROM employees
WHERE department NOT IN (
    SELECT department FROM temp_table  -- If this returns NULL
);
-- Result: Empty (NOT IN with NULL returns no rows)
-- Fix: Use NOT EXISTS or filter out NULLs

-- Pitfall 2: Correlated subquery in SELECT (performance)
-- Slow: Executes subquery for each row
SELECT
    e.name,
    (SELECT COUNT(*) FROM orders WHERE employee_id = e.id) as order_count
FROM employees e;

-- Faster: Use JOIN
SELECT e.name, COUNT(o.id) as order_count
FROM employees e
LEFT JOIN orders o ON e.id = o.employee_id
GROUP BY e.id, e.name;
```

---

## Indexes

### Q8: What are Indexes? Types and When to Use Them?

**Answer:**

An index is a database object that improves the speed of data retrieval operations. Think of it like a book index that helps you find information quickly.

**How Indexes Work:**

Without index: Full table scan (check every row)
```
Search for "John" in 1,000,000 rows → Check all 1,000,000 rows
```

With index: Binary search (much faster)
```
Search for "John" in 1,000,000 rows → Check ~20 rows
```

**Types of Indexes:**

**1. Single-Column Index**

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    email VARCHAR(100),
    name VARCHAR(100),
    created_at TIMESTAMP
);

-- Create index on email column
CREATE INDEX idx_email ON users(email);

-- Query using index (fast)
SELECT * FROM users WHERE email = 'john@example.com';
-- Uses idx_email index

-- Query not using index (slow)
SELECT * FROM users WHERE name = 'John';
-- Full table scan
```

**2. Composite Index (Multi-Column)**

```sql
-- Create composite index
CREATE INDEX idx_name_created ON users(name, created_at);

-- These queries use the index:
SELECT * FROM users WHERE name = 'John';
-- Uses leftmost column

SELECT * FROM users WHERE name = 'John' AND created_at > '2025-01-01';
-- Uses both columns

-- These queries DON'T use the index:
SELECT * FROM users WHERE created_at > '2025-01-01';
-- Doesn't start with leftmost column (name)
```

**Leftmost Prefix Rule:**
```
Index on (A, B, C) can be used for:
✓ WHERE A = x
✓ WHERE A = x AND B = y
✓ WHERE A = x AND B = y AND C = z
✗ WHERE B = y
✗ WHERE C = z
✗ WHERE B = y AND C = z
```

**3. Unique Index**

```sql
-- Ensures column values are unique
CREATE UNIQUE INDEX idx_unique_email ON users(email);

-- Attempts to insert duplicate will fail
INSERT INTO users (email) VALUES ('john@example.com');  -- OK
INSERT INTO users (email) VALUES ('john@example.com');  -- Error: Duplicate

-- PRIMARY KEY automatically creates unique index
CREATE TABLE products (
    id INT PRIMARY KEY,  -- Automatically creates unique index
    sku VARCHAR(50) UNIQUE  -- Also creates unique index
);
```

**4. Full-Text Index**

```sql
-- For text searching
CREATE TABLE articles (
    id INT PRIMARY KEY,
    title VARCHAR(200),
    content TEXT
);

-- Create full-text index
CREATE FULLTEXT INDEX idx_content ON articles(content);

-- Full-text search
SELECT * FROM articles
WHERE MATCH(content) AGAINST('database performance' IN NATURAL LANGUAGE MODE);

-- Boolean mode (advanced)
SELECT * FROM articles
WHERE MATCH(content) AGAINST('+database -mongodb' IN BOOLEAN MODE);
-- Must contain "database", must not contain "mongodb"
```

**5. Partial/Filtered Index**

```sql
-- Index only specific rows (PostgreSQL)
CREATE INDEX idx_active_users ON users(email) WHERE status = 'active';

-- Useful for queries like:
SELECT * FROM users WHERE status = 'active' AND email = 'john@example.com';
```

**6. Covering Index**

```sql
-- Index includes all columns needed by query
CREATE INDEX idx_covering ON orders(customer_id, order_date, total_amount);

-- This query can be satisfied entirely from index (no table access needed)
SELECT customer_id, order_date, total_amount
FROM orders
WHERE customer_id = 123;
```

**When to Use Indexes:**

**✓ Create Index When:**
- Column frequently used in WHERE clause
- Column used in JOIN conditions
- Column used in ORDER BY or GROUP BY
- Column has high cardinality (many unique values)
- Table is read-heavy

```sql
-- Good candidates for indexing:
CREATE INDEX idx_email ON users(email);        -- WHERE email = ?
CREATE INDEX idx_customer ON orders(customer_id);  -- JOIN condition
CREATE INDEX idx_date ON orders(order_date);   -- ORDER BY date
CREATE INDEX idx_status ON orders(status);     -- WHERE status = ?
```

**✗ Don't Create Index When:**
- Table is very small
- Column frequently updated
- Column has low cardinality (few unique values)
- Table is write-heavy
- Column not used in queries

```sql
-- Poor candidates for indexing:
-- Gender column (only 2-3 values)
CREATE INDEX idx_gender ON users(gender);  -- Don't do this

-- Boolean flags (only 2 values)
CREATE INDEX idx_active ON users(is_active);  -- Usually not helpful

-- Columns that change frequently
CREATE INDEX idx_last_modified ON logs(last_modified);  -- Expensive to maintain
```

**Index Performance Examples:**

```sql
-- Example 1: Before index
SELECT * FROM users WHERE email = 'john@example.com';
-- Execution time: 2000ms (full table scan of 1M rows)

CREATE INDEX idx_email ON users(email);

SELECT * FROM users WHERE email = 'john@example.com';
-- Execution time: 5ms (index seek)

-- Example 2: Composite index performance
CREATE INDEX idx_dept_salary ON employees(department, salary);

-- Fast: Uses index
SELECT * FROM employees
WHERE department = 'IT' AND salary > 60000;

-- Slower: Can only use first column of index
SELECT * FROM employees
WHERE salary > 60000;

-- Example 3: ORDER BY optimization
CREATE INDEX idx_created_at ON orders(created_at);

-- Fast: Index used for sorting
SELECT * FROM orders ORDER BY created_at DESC LIMIT 10;

-- Slow: No index, requires sorting
SELECT * FROM orders ORDER BY customer_name LIMIT 10;
```

**Analyzing Index Usage:**

```sql
-- Use EXPLAIN to see if index is used
EXPLAIN SELECT * FROM users WHERE email = 'john@example.com';

-- Output shows:
-- type: ref (using index)
-- key: idx_email
-- rows: 1

-- Show all indexes on a table
SHOW INDEXES FROM users;

-- Find unused indexes (PostgreSQL)
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan as index_scans
FROM pg_stat_user_indexes
WHERE idx_scan = 0
  AND indexrelname NOT LIKE 'pg_toast%'
ORDER BY schemaname, tablename;
```

**Index Maintenance:**

```sql
-- Rebuild index (MySQL)
ALTER TABLE users DROP INDEX idx_email, ADD INDEX idx_email(email);

-- Rebuild index (PostgreSQL)
REINDEX INDEX idx_email;

-- Remove unused index
DROP INDEX idx_email ON users;

-- Analyze table statistics (helps query optimizer)
ANALYZE TABLE users;
```

**Real-World Indexing Strategy:**

```sql
-- E-commerce database

-- Orders table
CREATE TABLE orders (
    id INT PRIMARY KEY,              -- Auto-indexed
    customer_id INT,
    order_date TIMESTAMP,
    status VARCHAR(50),
    total_amount DECIMAL(10,2)
);

-- Index for customer's orders
CREATE INDEX idx_customer_date ON orders(customer_id, order_date DESC);

-- Index for order status filtering
CREATE INDEX idx_status_date ON orders(status, order_date);

-- Index for date range queries
CREATE INDEX idx_date ON orders(order_date);

-- Common queries optimized:

-- 1. Customer's recent orders (uses idx_customer_date)
SELECT * FROM orders
WHERE customer_id = 123
ORDER BY order_date DESC
LIMIT 10;

-- 2. Pending orders (uses idx_status_date)
SELECT * FROM orders
WHERE status = 'pending'
ORDER BY order_date;

-- 3. Sales report (uses idx_date)
SELECT DATE(order_date), SUM(total_amount)
FROM orders
WHERE order_date >= '2025-10-01'
GROUP BY DATE(order_date);
```

**Trade-offs:**

**Benefits:**
- Faster SELECT queries
- Faster JOIN operations
- Faster sorting (ORDER BY)
- Enforces uniqueness (UNIQUE indexes)

**Costs:**
- Slower INSERT/UPDATE/DELETE (index must be updated)
- Additional disk space
- Memory usage
- Maintenance overhead

**Best Practices:**

1. Start with primary key and foreign key indexes
2. Add indexes based on query patterns
3. Use EXPLAIN to verify index usage
4. Monitor query performance
5. Remove unused indexes
6. Don't over-index (balance read vs write performance)
7. Consider index size (composite indexes can be large)
8. Regular maintenance (rebuild fragmented indexes)

```sql
-- Good indexing strategy
CREATE INDEX idx_user_email ON users(email);              -- Single column
CREATE INDEX idx_order_customer ON orders(customer_id);   -- Foreign key
CREATE INDEX idx_order_status_date ON orders(status, order_date);  -- Composite

-- Avoid over-indexing
-- Don't do this:
CREATE INDEX idx1 ON users(name);
CREATE INDEX idx2 ON users(name, email);
CREATE INDEX idx3 ON users(name, email, created_at);
-- idx2 and idx3 make idx1 redundant
```

---

## Transactions and ACID

### Q9: What are Transactions? Explain ACID Properties

**Answer:**

A transaction is a sequence of database operations that are treated as a single unit of work. Either all operations succeed (commit) or all fail (rollback).

**ACID Properties:**

### **A - Atomicity**

All operations in a transaction succeed or all fail. No partial completion.

```sql
-- Bank transfer example
START TRANSACTION;

-- Deduct from Account A
UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A';

-- Add to Account B
UPDATE accounts SET balance = balance + 100 WHERE account_id = 'B';

COMMIT;
-- Both updates succeed together

-- If any operation fails:
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A';  -- Success
UPDATE accounts SET balance = balance + 100 WHERE account_id = 'Z';  -- Fail: account doesn't exist
ROLLBACK;
-- Both updates are cancelled (atomicity preserved)
```

### **C - Consistency**

Database moves from one valid state to another. All constraints and rules are maintained.

```sql
-- Example: Total money in system must remain constant
START TRANSACTION;

-- Before:
-- Account A: $1000
-- Account B: $500
-- Total: $1500

UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A';
UPDATE accounts SET balance = balance + 100 WHERE account_id = 'B';

COMMIT;

-- After:
-- Account A: $900
-- Account B: $600
-- Total: $1500 (consistent - money conserved)

-- Constraint violation example
CREATE TABLE accounts (
    account_id VARCHAR(10) PRIMARY KEY,
    balance DECIMAL(10,2) CHECK (balance >= 0)  -- Constraint: no negative balance
);

START TRANSACTION;
UPDATE accounts SET balance = balance - 1000 WHERE account_id = 'A';
-- Error: Check constraint violated (balance would be negative)
ROLLBACK;
-- Database remains in consistent state
```

### **I - Isolation**

Concurrent transactions don't interfere with each other. Each transaction sees a consistent snapshot of data.

```sql
-- Transaction 1 (Transfer $100)
START TRANSACTION;
SELECT balance FROM accounts WHERE account_id = 'A';  -- Reads $1000
UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A';
-- Transaction pauses here...

-- Transaction 2 (runs concurrently)
START TRANSACTION;
SELECT balance FROM accounts WHERE account_id = 'A';  -- Still reads $1000 (isolated)
UPDATE accounts SET balance = balance - 50 WHERE account_id = 'A';
COMMIT;

-- Transaction 1 continues
COMMIT;

-- Final result depends on isolation level
```

**Isolation Levels:**

```sql
-- 1. READ UNCOMMITTED (lowest isolation)
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
-- Can read uncommitted changes from other transactions (dirty reads)

-- 2. READ COMMITTED (default in most DBs)
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
-- Only reads committed changes

-- 3. REPEATABLE READ
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
-- Same query in transaction returns same results

-- 4. SERIALIZABLE (highest isolation)
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
-- Complete isolation, transactions run as if serial
```

**Isolation Problems:**

```sql
-- Problem 1: Dirty Read (reading uncommitted data)
-- Transaction 1
START TRANSACTION;
UPDATE accounts SET balance = 5000 WHERE account_id = 'A';
-- Not committed yet

-- Transaction 2 (with READ UNCOMMITTED)
SELECT balance FROM accounts WHERE account_id = 'A';  -- Reads 5000

-- Transaction 1
ROLLBACK;  -- Change reverted

-- Transaction 2 read wrong value (dirty read)

-- Problem 2: Non-Repeatable Read
-- Transaction 1
START TRANSACTION;
SELECT balance FROM accounts WHERE account_id = 'A';  -- Reads 1000

-- Transaction 2
UPDATE accounts SET balance = 2000 WHERE account_id = 'A';
COMMIT;

-- Transaction 1
SELECT balance FROM accounts WHERE account_id = 'A';  -- Now reads 2000
-- Different value in same transaction (non-repeatable read)

-- Problem 3: Phantom Read
-- Transaction 1
START TRANSACTION;
SELECT COUNT(*) FROM orders WHERE status = 'pending';  -- Returns 10

-- Transaction 2
INSERT INTO orders (status) VALUES ('pending');
COMMIT;

-- Transaction 1
SELECT COUNT(*) FROM orders WHERE status = 'pending';  -- Returns 11
-- New row appeared (phantom read)
```

### **D - Durability**

Committed changes are permanent, even if system crashes.

```sql
START TRANSACTION;
INSERT INTO orders (customer_id, total) VALUES (123, 99.99);
COMMIT;
-- Even if server crashes immediately after COMMIT,
-- this data will persist (written to disk, logged)

-- Database uses:
-- - Write-ahead logging (WAL)
-- - Transaction logs
-- - Checkpoints
-- to ensure durability
```

**Transaction Control Commands:**

```sql
-- START TRANSACTION / BEGIN
START TRANSACTION;  -- or BEGIN;

-- COMMIT: Save all changes
UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A';
UPDATE accounts SET balance = balance + 100 WHERE account_id = 'B';
COMMIT;

-- ROLLBACK: Undo all changes
START TRANSACTION;
DELETE FROM customers WHERE id = 123;
SELECT * FROM customers;  -- Customer gone
ROLLBACK;
SELECT * FROM customers;  -- Customer back

-- SAVEPOINT: Create checkpoint within transaction
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A';
SAVEPOINT sp1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 'B';
SAVEPOINT sp2;
UPDATE accounts SET balance = balance + 50 WHERE account_id = 'C';
-- Error in last update
ROLLBACK TO sp2;  -- Undo only last update
COMMIT;  -- Save first two updates
```

**Real-World Example: E-commerce Order**

```sql
-- Complete order processing transaction
START TRANSACTION;

-- 1. Create order
INSERT INTO orders (customer_id, total_amount, status)
VALUES (123, 299.99, 'processing');

SET @order_id = LAST_INSERT_ID();

-- 2. Add order items
INSERT INTO order_items (order_id, product_id, quantity, price)
VALUES
    (@order_id, 1, 2, 99.99),
    (@order_id, 2, 1, 100.01);

-- 3. Update inventory
UPDATE products SET stock = stock - 2 WHERE id = 1;
UPDATE products SET stock = stock - 1 WHERE id = 2;

-- 4. Check stock availability
SELECT stock FROM products WHERE id IN (1, 2);
-- If any stock < 0, ROLLBACK

-- 5. Process payment
INSERT INTO payments (order_id, amount, status)
VALUES (@order_id, 299.99, 'completed');

-- 6. Update customer loyalty points
UPDATE customers
SET loyalty_points = loyalty_points + 299
WHERE id = 123;

-- All or nothing
COMMIT;
```

**Deadlocks:**

```sql
-- Transaction 1
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A';
-- Waits for lock on B...
UPDATE accounts SET balance = balance + 100 WHERE account_id = 'B';

-- Transaction 2 (runs simultaneously)
START TRANSACTION;
UPDATE accounts SET balance = balance - 50 WHERE account_id = 'B';
-- Waits for lock on A...
UPDATE accounts SET balance = balance + 50 WHERE account_id = 'A';

-- Deadlock! Both waiting for each other
-- Database detects and rolls back one transaction

-- Solution: Always access resources in same order
-- Both transactions should lock A first, then B
```

**Best Practices:**

```sql
-- 1. Keep transactions short
-- Bad: Long-running transaction
START TRANSACTION;
-- Complex calculations...
-- Sleep 10 seconds...
-- More operations...
COMMIT;

-- Good: Quick transaction
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A';
UPDATE accounts SET balance = balance + 100 WHERE account_id = 'B';
COMMIT;

-- 2. Handle errors properly
START TRANSACTION;
BEGIN TRY
    UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A';
    UPDATE accounts SET balance = balance + 100 WHERE account_id = 'B';
    COMMIT;
END TRY
BEGIN CATCH
    ROLLBACK;
    -- Log error
    THROW;
END CATCH

-- 3. Use appropriate isolation level
-- Read-only reporting: READ COMMITTED (faster)
-- Financial transactions: SERIALIZABLE (safer)

-- 4. Avoid nested transactions
-- Don't do this:
START TRANSACTION;
    START TRANSACTION;  -- Problematic in most databases
    COMMIT;
COMMIT;
```

---

## Normalization

### Q10: What is Normalization? Explain Normal Forms

**Answer:**

Normalization is the process of organizing data to reduce redundancy and improve data integrity. It involves decomposing tables into smaller tables and defining relationships.

**Goals:**
- Eliminate redundant data
- Ensure data dependencies make sense
- Reduce insertion, update, and deletion anomalies

### **Un-normalized Table (0NF)**

```sql
-- Bad: Repeating groups, redundancy
CREATE TABLE orders_bad (
    order_id INT,
    customer_name VARCHAR(100),
    customer_email VARCHAR(100),
    customer_phone VARCHAR(20),
    product1_name VARCHAR(100),
    product1_price DECIMAL(10,2),
    product1_quantity INT,
    product2_name VARCHAR(100),
    product2_price DECIMAL(10,2),
    product2_quantity INT
);

-- Problems:
-- 1. Limited number of products per order
-- 2. Wasted space if order has only 1 product
-- 3. Customer info duplicated for each order
```

### **First Normal Form (1NF)**

**Rules:**
- Each column contains atomic (indivisible) values
- No repeating groups or arrays
- Each column has unique name
- Order doesn't matter

```sql
-- Violates 1NF: Non-atomic values
CREATE TABLE customers_bad (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    phones VARCHAR(200)  -- "555-1234, 555-5678, 555-9012"
);

-- Correct 1NF:
CREATE TABLE customers (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE customer_phones (
    customer_id INT,
    phone VARCHAR(20),
    PRIMARY KEY (customer_id, phone),
    FOREIGN KEY (customer_id) REFERENCES customers(id)
);

-- Before 1NF:
| id | name | phones                           |
|----|------|----------------------------------|
| 1  | John | 555-1234, 555-5678, 555-9012    |

-- After 1NF:
Customers:
| id | name |
|----|------|
| 1  | John |

Customer_Phones:
| customer_id | phone    |
|-------------|----------|
| 1           | 555-1234 |
| 1           | 555-5678 |
| 1           | 555-9012 |
```

### **Second Normal Form (2NF)**

**Rules:**
- Must be in 1NF
- No partial dependencies (all non-key columns fully depend on entire primary key)

```sql
-- Violates 2NF: Partial dependency
CREATE TABLE order_items_bad (
    order_id INT,
    product_id INT,
    product_name VARCHAR(100),     -- Depends only on product_id
    product_price DECIMAL(10,2),   -- Depends only on product_id
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);

-- Problems:
-- 1. product_name and product_price depend only on product_id
-- 2. Not dependent on full key (order_id, product_id)

-- Correct 2NF:
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    product_price DECIMAL(10,2)
);

CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

-- Before 2NF:
Order_Items:
| order_id | product_id | product_name | product_price | quantity |
|----------|------------|--------------|---------------|----------|
| 1        | 100        | Widget       | 9.99          | 2        |
| 1        | 101        | Gadget       | 19.99         | 1        |
| 2        | 100        | Widget       | 9.99          | 5        |

-- After 2NF:
Products:
| product_id | product_name | product_price |
|------------|--------------|---------------|
| 100        | Widget       | 9.99          |
| 101        | Gadget       | 19.99         |

Order_Items:
| order_id | product_id | quantity |
|----------|------------|----------|
| 1        | 100        | 2        |
| 1        | 101        | 1        |
| 2        | 100        | 5        |
```

### **Third Normal Form (3NF)**

**Rules:**
- Must be in 2NF
- No transitive dependencies (non-key columns don't depend on other non-key columns)

```sql
-- Violates 3NF: Transitive dependency
CREATE TABLE employees_bad (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    department_name VARCHAR(100),    -- Depends on department_id (transitive)
    department_location VARCHAR(100) -- Depends on department_id (transitive)
);

-- Problems:
-- 1. department_name depends on department_id, not employee_id
-- 2. department_location depends on department_id, not employee_id
-- 3. Redundancy: department info repeated for each employee

-- Correct 3NF:
CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(100),
    department_location VARCHAR(100)
);

CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(department_id)
);

-- Before 3NF:
Employees:
| employee_id | name  | dept_id | dept_name | dept_location |
|-------------|-------|---------|-----------|---------------|
| 1           | John  | 10      | IT        | Building A    |
| 2           | Jane  | 10      | IT        | Building A    |
| 3           | Bob   | 20      | HR        | Building B    |

-- After 3NF:
Departments:
| department_id | department_name | department_location |
|---------------|-----------------|---------------------|
| 10            | IT              | Building A          |
| 20            | HR              | Building B          |

Employees:
| employee_id | name  | department_id |
|-------------|-------|---------------|
| 1           | John  | 10            |
| 2           | Jane  | 10            |
| 3           | Bob   | 20            |
```

### **Boyce-Codd Normal Form (BCNF)**

**Rules:**
- Must be in 3NF
- For every functional dependency X → Y, X must be a super key

```sql
-- Example: Student, Course, Instructor
-- Rule: Each instructor teaches only one course
-- But a course can have multiple instructors

-- Violates BCNF:
CREATE TABLE teaching_bad (
    student_id INT,
    course_id INT,
    instructor_id INT,
    PRIMARY KEY (student_id, course_id)
);
-- Instructor_id → course_id (instructor determines course)
-- But instructor_id is not a super key

-- Correct BCNF:
CREATE TABLE instructor_courses (
    instructor_id INT PRIMARY KEY,
    course_id INT,
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);

CREATE TABLE student_instructors (
    student_id INT,
    instructor_id INT,
    PRIMARY KEY (student_id, instructor_id),
    FOREIGN KEY (instructor_id) REFERENCES instructor_courses(instructor_id)
);
```

### **Fourth Normal Form (4NF)**

**Rules:**
- Must be in BCNF
- No multi-valued dependencies

```sql
-- Violates 4NF: Multi-valued dependencies
CREATE TABLE person_skills_languages_bad (
    person_id INT,
    skill VARCHAR(50),
    language VARCHAR(50),
    PRIMARY KEY (person_id, skill, language)
);

-- Problem: Skills and languages are independent
INSERT VALUES
(1, 'Programming', 'English'),
(1, 'Programming', 'Spanish'),
(1, 'Design', 'English'),
(1, 'Design', 'Spanish');
-- Cartesian product: unnecessary redundancy

-- Correct 4NF: Separate independent multi-valued facts
CREATE TABLE person_skills (
    person_id INT,
    skill VARCHAR(50),
    PRIMARY KEY (person_id, skill)
);

CREATE TABLE person_languages (
    person_id INT,
    language VARCHAR(50),
    PRIMARY KEY (person_id, language)
);

INSERT INTO person_skills VALUES
(1, 'Programming'),
(1, 'Design');

INSERT INTO person_languages VALUES
(1, 'English'),
(1, 'Spanish');
```

### **Fifth Normal Form (5NF)**

**Rules:**
- Must be in 4NF
- No join dependencies (can't be decomposed further without loss)

Rarely needed in practice. Deals with complex cases where table can be reconstructed from multiple smaller tables.

### **Denormalization**

Sometimes we intentionally violate normalization for performance.

```sql
-- Normalized (3NF):
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE
);

CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    price DECIMAL(10,2),
    PRIMARY KEY (order_id, product_id)
);

-- Query requires JOIN to get order total:
SELECT o.order_id, SUM(oi.quantity * oi.price) as total
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY o.order_id;

-- Denormalized (add calculated column):
CREATE TABLE orders_denormalized (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total_amount DECIMAL(10,2)  -- Denormalized: duplicates calculated data
);

-- Faster query (no JOIN needed):
SELECT order_id, total_amount FROM orders_denormalized;

-- Trade-off:
-- + Faster reads
-- - Slower writes (must update total on every item change)
-- - Risk of inconsistency
```

**Real-World Example:**

```sql
-- E-commerce database evolution

-- 0NF (Bad):
CREATE TABLE sales_bad (
    sale_id INT,
    customer_name VARCHAR(100),
    customer_email VARCHAR(100),
    product1 VARCHAR(100),
    price1 DECIMAL(10,2),
    product2 VARCHAR(100),
    price2 DECIMAL(10,2)
);

-- 1NF: Atomic values, no repeating groups
CREATE TABLE sales_1nf (
    sale_id INT,
    customer_name VARCHAR(100),
    customer_email VARCHAR(100),
    product_name VARCHAR(100),
    product_price DECIMAL(10,2)
);

-- 2NF: Remove partial dependencies
CREATE TABLE customers_2nf (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

CREATE TABLE sales_2nf (
    sale_id INT PRIMARY KEY,
    customer_id INT,
    product_name VARCHAR(100),
    product_price DECIMAL(10,2)
);

-- 3NF: Remove transitive dependencies
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10,2)
);

CREATE TABLE sales (
    sale_id INT PRIMARY KEY,
    customer_id INT,
    sale_date DATE,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

CREATE TABLE sale_items (
    sale_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (sale_id, product_id),
    FOREIGN KEY (sale_id) REFERENCES sales(sale_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

**Benefits of Normalization:**
- Eliminates data redundancy
- Ensures data consistency
- Easier to maintain
- Flexible for changes
- Reduces storage space

**Drawbacks:**
- More complex queries (more JOINs)
- Potentially slower performance
- More tables to manage

**When to Denormalize:**
- Read-heavy applications
- Reporting/analytics databases
- Performance critical queries
- Data warehouse scenarios

---

## NoSQL Types

### Q11: What are Different Types of NoSQL Databases?

**Answer:**

NoSQL databases are categorized by their data model. There are four main types:

### **1. Document Databases**

Store data as JSON-like documents.

**Examples:** MongoDB, CouchDB, Firestore

**Data Model:**
```javascript
// MongoDB document
{
    "_id": "507f1f77bcf86cd799439011",
    "name": "John Doe",
    "email": "john@example.com",
    "age": 30,
    "address": {
        "street": "123 Main St",
        "city": "New York",
        "zip": "10001"
    },
    "orders": [
        {
            "orderId": "ORD123",
            "date": "2025-10-01",
            "total": 99.99
        },
        {
            "orderId": "ORD124",
            "date": "2025-10-03",
            "total": 149.99
        }
    ],
    "tags": ["premium", "active"]
}
```

**Use Cases:**
- Content management systems
- User profiles
- Product catalogs
- Mobile applications

**Advantages:**
- Flexible schema
- Natural data representation
- Easy to scale horizontally
- Rich query capabilities

**Disadvantages:**
- No ACID guarantees across documents
- Can lead to data duplication
- Complex queries may be inefficient

### **2. Key-Value Stores**

Simplest NoSQL type. Store data as key-value pairs.

**Examples:** Redis, DynamoDB, Riak

**Data Model:**
```
Key                     Value
-------------------     ---------------------
user:1000               {"name": "John", "email": "john@example.com"}
session:abc123          {"userId": 1000, "expires": 1728000000}
cache:product:50        {"name": "Widget", "price": 9.99}
```

**Redis Examples:**
```redis
-- String
SET user:1000:name "John Doe"
GET user:1000:name

-- Hash (object)
HSET user:1000 name "John Doe" email "john@example.com" age 30
HGET user:1000 email
HGETALL user:1000

-- List
LPUSH notifications:user:1000 "New message"
LPUSH notifications:user:1000 "Friend request"
LRANGE notifications:user:1000 0 -1

-- Set (unique values)
SADD user:1000:tags "premium"
SADD user:1000:tags "active"
SMEMBERS user:1000:tags

-- Sorted Set (with scores)
ZADD leaderboard 1000 "player1"
ZADD leaderboard 1500 "player2"
ZRANGE leaderboard 0 -1 WITHSCORES

-- Expiration
SETEX session:abc123 3600 "user_data"  -- Expires in 1 hour
```

**Use Cases:**
- Caching
- Session storage
- Real-time analytics
- Leaderboards
- Rate limiting

**Advantages:**
- Extremely fast (often in-memory)
- Simple data model
- Easy to scale
- Low latency

**Disadvantages:**
- Limited query capabilities
- Only lookup by key
- No relationships between data

### **3. Column-Family Databases**

Store data in column families (wide columns).

**Examples:** Cassandra, HBase, ScyllaDB

**Data Model:**
```
Row Key: user:1000
Column Family: profile
    name: "John Doe"
    email: "john@example.com"
    created: "2025-01-01"

Column Family: orders
    order:123:date: "2025-10-01"
    order:123:total: 99.99
    order:124:date: "2025-10-03"
    order:124:total: 149.99
```

**Cassandra Example:**
```sql
-- Create keyspace (database)
CREATE KEYSPACE ecommerce
WITH replication = {
    'class': 'SimpleStrategy',
    'replication_factor': 3
};

-- Create column family (table)
CREATE TABLE users (
    user_id UUID PRIMARY KEY,
    name TEXT,
    email TEXT,
    created_at TIMESTAMP
);

CREATE TABLE user_activity (
    user_id UUID,
    activity_time TIMESTAMP,
    activity_type TEXT,
    details TEXT,
    PRIMARY KEY (user_id, activity_time)
) WITH CLUSTERING ORDER BY (activity_time DESC);

-- Insert data
INSERT INTO users (user_id, name, email, created_at)
VALUES (uuid(), 'John Doe', 'john@example.com', toTimestamp(now()));

-- Query by partition key
SELECT * FROM user_activity
WHERE user_id = 123e4567-e89b-12d3-a456-426614174000
  AND activity_time >= '2025-10-01'
  AND activity_time < '2025-10-05';
```

**Use Cases:**
- Time-series data
- IoT sensor data
- Event logging
- Analytics
- Large-scale data warehouses

**Advantages:**
- Excellent for write-heavy workloads
- Highly scalable
- Good for time-series data
- No single point of failure

**Disadvantages:**
- Limited query flexibility
- Must design around query patterns
- No JOINs
- Eventual consistency

### **4. Graph Databases**

Store data as nodes (entities) and edges (relationships).

**Examples:** Neo4j, Amazon Neptune, ArangoDB

**Data Model:**
```
Nodes:
(person:Person {name: "John", age: 30})
(person:Person {name: "Jane", age: 28})
(company:Company {name: "Acme Corp"})

Edges (Relationships):
(john)-[:FRIENDS_WITH {since: 2020}]->(jane)
(john)-[:WORKS_AT {role: "Engineer"}]->(acme)
(jane)-[:WORKS_AT {role: "Manager"}]->(acme)
```

**Neo4j Examples:**
```cypher
-- Create nodes
CREATE (john:Person {name: 'John Doe', age: 30, email: 'john@example.com'})
CREATE (jane:Person {name: 'Jane Smith', age: 28, email: 'jane@example.com'})
CREATE (acme:Company {name: 'Acme Corp', industry: 'Technology'})

-- Create relationships
MATCH (john:Person {name: 'John Doe'})
MATCH (jane:Person {name: 'Jane Smith'})
CREATE (john)-[:FRIENDS_WITH {since: 2020}]->(jane)

MATCH (john:Person {name: 'John Doe'})
MATCH (acme:Company {name: 'Acme Corp'})
CREATE (john)-[:WORKS_AT {role: 'Engineer', since: '2022-01-01'}]->(acme)

-- Find friends
MATCH (person:Person {name: 'John Doe'})-[:FRIENDS_WITH]->(friend)
RETURN friend.name, friend.email

-- Find friends of friends
MATCH (person:Person {name: 'John Doe'})-[:FRIENDS_WITH*2]->(fof)
RETURN DISTINCT fof.name

-- Find colleagues
MATCH (person:Person {name: 'John Doe'})-[:WORKS_AT]->(company)<-[:WORKS_AT]-(colleague)
RETURN colleague.name, colleague.role

-- Complex query: Recommend friends (friends of friends who aren't already friends)
MATCH (user:Person {name: 'John Doe'})-[:FRIENDS_WITH]->(friend)-[:FRIENDS_WITH]->(fof)
WHERE NOT (user)-[:FRIENDS_WITH]->(fof) AND user <> fof
RETURN fof.name, COUNT(*) as mutual_friends
ORDER BY mutual_friends DESC
LIMIT 10

-- Shortest path between two people
MATCH path = shortestPath(
    (john:Person {name: 'John Doe'})-[:FRIENDS_WITH*]-(jane:Person {name: 'Jane Smith'})
)
RETURN length(path), nodes(path)
```

**Use Cases:**
- Social networks
- Recommendation engines
- Fraud detection
- Knowledge graphs
- Network analysis

**Advantages:**
- Excellent for connected data
- Natural representation of relationships
- Fast relationship queries
- Flexible schema

**Disadvantages:**
- Not ideal for simple CRUD operations
- Can be complex to learn
- Limited scalability compared to other NoSQL types

### **Comparison Table**

| Type | Best For | Examples | Query Style |
|------|----------|----------|-------------|
| **Document** | Flexible, hierarchical data | MongoDB, CouchDB | Rich queries, indexes |
| **Key-Value** | Caching, sessions | Redis, DynamoDB | Key lookups only |
| **Column-Family** | Time-series, analytics | Cassandra, HBase | Partition key queries |
| **Graph** | Connected data, relationships | Neo4j, Neptune | Graph traversal |

### **Choosing the Right NoSQL Type**

```
Use Document DB if:
✓ Data has varying structure
✓ Need flexible schema
✓ Hierarchical data (nested objects)
✓ Example: Product catalog with different attributes per category

Use Key-Value Store if:
✓ Simple get/set operations
✓ Need extreme performance
✓ Session or cache storage
✓ Example: User session management

Use Column-Family DB if:
✓ Write-heavy workload
✓ Time-series data
✓ Need massive scalability
✓ Example: IoT sensor data collection

Use Graph DB if:
✓ Many relationships between entities
✓ Need to traverse connections
✓ Relationship queries are important
✓ Example: Social network friend recommendations
```

---

## MongoDB Queries

### Q12: How to Perform CRUD Operations in MongoDB?

**Answer:**

MongoDB is a document database that stores data in JSON-like BSON format.

### **Create (Insert) Operations**

```javascript
// Insert one document
db.users.insertOne({
    name: "John Doe",
    email: "john@example.com",
    age: 30,
    address: {
        city: "New York",
        zip: "10001"
    },
    tags: ["premium", "active"]
})

// Insert multiple documents
db.users.insertMany([
    {
        name: "Jane Smith",
        email: "jane@example.com",
        age: 28
    },
    {
        name: "Bob Wilson",
        email: "bob@example.com",
        age: 35
    }
])

// Insert with specific _id
db.products.insertOne({
    _id: "PROD123",
    name: "Widget",
    price: 9.99,
    category: "Electronics"
})
```

### **Read (Find) Operations**

```javascript
// Find all documents
db.users.find()

// Find with filter
db.users.find({ age: 30 })
db.users.find({ age: { $gt: 25 } })  // age > 25

// Find one document
db.users.findOne({ email: "john@example.com" })

// Projection (select specific fields)
db.users.find(
    { age: { $gt: 25 } },
    { name: 1, email: 1, _id: 0 }  // 1 = include, 0 = exclude
)
// Returns only name and email

// Comparison operators
db.products.find({ price: { $gt: 10 } })        // greater than
db.products.find({ price: { $gte: 10 } })       // greater than or equal
db.products.find({ price: { $lt: 50 } })        // less than
db.products.find({ price: { $lte: 50 } })       // less than or equal
db.products.find({ price: { $ne: 9.99 } })      // not equal
db.products.find({ price: { $in: [9.99, 19.99] } })  // in array
db.products.find({ price: { $nin: [9.99, 19.99] } }) // not in array

// Logical operators
db.products.find({
    $and: [
        { price: { $gt: 10 } },
        { category: "Electronics" }
    ]
})

db.products.find({
    $or: [
        { price: { $lt: 10 } },
        { category: "Books" }
    ]
})

db.products.find({
    price: { $not: { $gt: 100 } }
})

// Array queries
db.users.find({ tags: "premium" })              // has tag "premium"
db.users.find({ tags: { $all: ["premium", "active"] } })  // has both tags
db.users.find({ tags: { $size: 2 } })           // array has exactly 2 elements

// Nested object queries
db.users.find({ "address.city": "New York" })
db.users.find({ "address.zip": { $regex: "^100" } })  // zip starts with 100

// Regular expressions
db.users.find({ email: { $regex: "@example.com$" } })
db.users.find({ name: { $regex: "^John", $options: "i" } })  // case-insensitive

// Exists operator
db.users.find({ phone: { $exists: true } })     // has phone field
db.users.find({ phone: { $exists: false } })    // doesn't have phone field

// Sorting
db.users.find().sort({ age: 1 })                // ascending
db.users.find().sort({ age: -1 })               // descending
db.users.find().sort({ age: -1, name: 1 })      // multi-field sort

// Limit and Skip (pagination)
db.users.find().limit(10)                       // first 10 documents
db.users.find().skip(10).limit(10)              // skip 10, next 10 (page 2)

// Count
db.users.countDocuments({ age: { $gt: 25 } })
db.users.estimatedDocumentCount()               // fast count (all docs)
```

### **Update Operations**

```javascript
// Update one document
db.users.updateOne(
    { email: "john@example.com" },              // filter
    { $set: { age: 31 } }                       // update
)

// Update multiple documents
db.users.updateMany(
    { age: { $lt: 25 } },
    { $set: { status: "young" } }
)

// Update operators
// $set: Set field value
db.users.updateOne(
    { _id: 1 },
    { $set: { name: "John Doe", age: 30 } }
)

// $unset: Remove field
db.users.updateOne(
    { _id: 1 },
    { $unset: { age: "" } }
)

// $inc: Increment numeric value
db.products.updateOne(
    { _id: "PROD123" },
    { $inc: { stock: -5 } }                     // decrease stock by 5
)

db.users.updateOne(
    { _id: 1 },
    { $inc: { age: 1, login_count: 1 } }
)

// $mul: Multiply value
db.products.updateOne(
    { _id: "PROD123" },
    { $mul: { price: 1.1 } }                    // increase price by 10%
)

// $rename: Rename field
db.users.updateOne(
    { _id: 1 },
    { $rename: { "name": "full_name" } }
)

// $min / $max: Update only if smaller/larger
db.products.updateOne(
    { _id: "PROD123" },
    { $min: { price: 8.99 } }                   // update only if 8.99 < current
)

// Array update operators
// $push: Add to array
db.users.updateOne(
    { _id: 1 },
    { $push: { tags: "new_tag" } }
)

// $push with modifiers
db.users.updateOne(
    { _id: 1 },
    {
        $push: {
            orders: {
                $each: [                        // add multiple items
                    { orderId: "ORD123", total: 99.99 },
                    { orderId: "ORD124", total: 149.99 }
                ],
                $sort: { total: -1 },           // sort by total descending
                $slice: 10                      // keep only last 10
            }
        }
    }
)

// $pull: Remove from array
db.users.updateOne(
    { _id: 1 },
    { $pull: { tags: "inactive" } }
)

db.users.updateOne(
    { _id: 1 },
    { $pull: { orders: { total: { $lt: 50 } } } }  // remove orders < $50
)

// $addToSet: Add to array only if doesn't exist
db.users.updateOne(
    { _id: 1 },
    { $addToSet: { tags: "premium" } }          // won't add duplicate
)

// $pop: Remove first or last array element
db.users.updateOne(
    { _id: 1 },
    { $pop: { orders: 1 } }                     // remove last element (-1 for first)
)

// Update array element
db.users.updateOne(
    { _id: 1, "orders.orderId": "ORD123" },
    { $set: { "orders.$.status": "shipped" } }  // $ = matched element
)

// Update all array elements
db.users.updateOne(
    { _id: 1 },
    { $set: { "orders.$[].processed": true } }  // $[] = all elements
)

// Update array elements matching condition
db.users.updateOne(
    { _id: 1 },
    {
        $set: { "orders.$[elem].status": "cancelled" }
    },
    {
        arrayFilters: [{ "elem.total": { $lt: 50 } }]  // only orders < $50
    }
)

// Replace entire document
db.users.replaceOne(
    { _id: 1 },
    {
        name: "John Doe",
        email: "john@example.com",
        age: 30
    }
)

// Upsert: Update if exists, insert if not
db.users.updateOne(
    { email: "new@example.com" },
    { $set: { name: "New User", age: 25 } },
    { upsert: true }                            // creates if doesn't exist
)

// findOneAndUpdate: Update and return document
db.users.findOneAndUpdate(
    { email: "john@example.com" },
    { $inc: { age: 1 } },
    { returnNewDocument: true }                 // return updated document
)
```

### **Delete Operations**

```javascript
// Delete one document
db.users.deleteOne({ email: "john@example.com" })

// Delete multiple documents
db.users.deleteMany({ age: { $lt: 18 } })

// Delete all documents
db.users.deleteMany({})

// findOneAndDelete: Delete and return document
db.users.findOneAndDelete({ email: "john@example.com" })
```

### **Aggregation Pipeline**

```javascript
// Basic aggregation
db.orders.aggregate([
    // Stage 1: Filter
    { $match: { status: "completed" } },

    // Stage 2: Group and calculate
    {
        $group: {
            _id: "$customer_id",
            total_spent: { $sum: "$total_amount" },
            order_count: { $sum: 1 },
            avg_order: { $avg: "$total_amount" }
        }
    },

    // Stage 3: Sort
    { $sort: { total_spent: -1 } },

    // Stage 4: Limit
    { $limit: 10 }
])

// Complex aggregation example
db.orders.aggregate([
    // Unwind array
    { $unwind: "$items" },

    // Join with products collection
    {
        $lookup: {
            from: "products",
            localField: "items.product_id",
            foreignField: "_id",
            as: "product_info"
        }
    },

    // Project specific fields
    {
        $project: {
            customer_id: 1,
            "items.quantity": 1,
            "items.price": 1,
            "product_info.name": 1,
            line_total: {
                $multiply: ["$items.quantity", "$items.price"]
            }
        }
    },

    // Group by customer
    {
        $group: {
            _id: "$customer_id",
            total_items: { $sum: "$items.quantity" },
            total_amount: { $sum: "$line_total" },
            products: { $addToSet: "$product_info.name" }
        }
    },

    // Filter groups
    { $match: { total_amount: { $gt: 1000 } } },

    // Sort
    { $sort: { total_amount: -1 } }
])

// Common aggregation operators
// $sum, $avg, $min, $max, $first, $last
// $push, $addToSet (array operations)
// $concat, $substr, $toLower, $toUpper (string operations)
// $year, $month, $dayOfMonth (date operations)
// $cond, $ifNull (conditional operations)
```

### **Indexes in MongoDB**

```javascript
// Create index
db.users.createIndex({ email: 1 })              // ascending
db.users.createIndex({ age: -1 })               // descending

// Compound index
db.users.createIndex({ age: 1, name: 1 })

// Unique index
db.users.createIndex({ email: 1 }, { unique: true })

// Text index (for full-text search)
db.articles.createIndex({ content: "text" })
db.articles.find({ $text: { $search: "database mongodb" } })

// List indexes
db.users.getIndexes()

// Drop index
db.users.dropIndex("email_1")

// Explain query (analyze performance)
db.users.find({ email: "john@example.com" }).explain("executionStats")
```

### **Real-World Example: E-commerce Queries**

```javascript
// Find products by category with pagination
db.products.find({ category: "Electronics" })
    .sort({ price: 1 })
    .skip(20)
    .limit(10)

// Get user's recent orders with product details
db.orders.aggregate([
    { $match: { customer_id: "CUST123" } },
    { $sort: { order_date: -1 } },
    { $limit: 5 },
    { $unwind: "$items" },
    {
        $lookup: {
            from: "products",
            localField: "items.product_id",
            foreignField: "_id",
            as: "product"
        }
    },
    { $unwind: "$product" },
    {
        $project: {
            order_id: 1,
            order_date: 1,
            product_name: "$product.name",
            quantity: "$items.quantity",
            price: "$items.price",
            subtotal: {
                $multiply: ["$items.quantity", "$items.price"]
            }
        }
    }
])

// Update product stock after purchase
db.products.updateOne(
    { _id: "PROD123", stock: { $gte: 5 } },     // ensure enough stock
    { $inc: { stock: -5 } }
)

// Add review to product
db.products.updateOne(
    { _id: "PROD123" },
    {
        $push: {
            reviews: {
                user_id: "USER456",
                rating: 5,
                comment: "Great product!",
                date: new Date()
            }
        },
        $inc: { review_count: 1 },
        $set: {
            avg_rating: {
                $avg: "$reviews.rating"             // recalculate average
            }
        }
    }
)

// Find top-selling products
db.order_items.aggregate([
    {
        $group: {
            _id: "$product_id",
            total_sold: { $sum: "$quantity" },
            revenue: {
                $sum: { $multiply: ["$quantity", "$price"] }
            }
        }
    },
    { $sort: { total_sold: -1 } },
    { $limit: 10 },
    {
        $lookup: {
            from: "products",
            localField: "_id",
            foreignField: "_id",
            as: "product"
        }
    },
    { $unwind: "$product" },
    {
        $project: {
            product_name: "$product.name",
            total_sold: 1,
            revenue: 1
        }
    }
])
```

---

## Redis

### Q13: What is Redis and Common Use Cases?

**Answer:**

Redis (Remote Dictionary Server) is an in-memory key-value data store known for its speed. It's often used as a cache, message broker, and database.

**Key Features:**
- In-memory storage (extremely fast)
- Data persistence options
- Rich data structures
- Pub/Sub messaging
- Atomic operations
- TTL (Time To Live) support

### **Data Types and Operations**

**1. Strings**

```redis
-- Set value
SET key "value"
SET user:1000:name "John Doe"

-- Get value
GET user:1000:name

-- Set with expiration (seconds)
SETEX session:abc123 3600 "user_data"        -- expires in 1 hour
SET session:abc123 "user_data" EX 3600       -- same as above

-- Set only if not exists
SETNX user:1000:name "John"                  -- won't overwrite if exists

-- Increment/Decrement
SET page_views 100
INCR page_views                              -- now 101
INCRBY page_views 5                          -- now 106
DECR page_views                              -- now 105
DECRBY page_views 3                          -- now 102

-- Multiple operations
MSET key1 "value1" key2 "value2" key3 "value3"
MGET key1 key2 key3

-- Append
SET message "Hello"
APPEND message " World"                      -- now "Hello World"

-- String operations
STRLEN message                               -- 11
GETRANGE message 0 4                         -- "Hello"
```

**2. Hashes (Objects)**

```redis
-- Set hash field
HSET user:1000 name "John Doe"
HSET user:1000 email "john@example.com"
HSET user:1000 age 30

-- Set multiple fields
HMSET user:1000 name "John Doe" email "john@example.com" age 30

-- Get field
HGET user:1000 name

-- Get all fields
HGETALL user:1000

-- Get multiple fields
HMGET user:1000 name email

-- Increment hash field
HINCRBY user:1000 age 1
HINCRBYFLOAT user:1000 balance 10.50

-- Check if field exists
HEXISTS user:1000 name

-- Delete field
HDEL user:1000 age

-- Get all keys or values
HKEYS user:1000
HVALS user:1000

-- Field count
HLEN user:1000
```

**3. Lists (Ordered Collections)**

```redis
-- Push to list (left/right)
LPUSH notifications "message1"               -- add to beginning
RPUSH notifications "message2"               -- add to end

-- Push multiple
LPUSH notifications "msg3" "msg4" "msg5"

-- Pop from list
LPOP notifications                           -- remove from beginning
RPOP notifications                           -- remove from end

-- Get range
LRANGE notifications 0 -1                    -- all items
LRANGE notifications 0 9                     -- first 10 items

-- Get by index
LINDEX notifications 0                       -- first item

-- Set by index
LSET notifications 0 "updated_message"

-- List length
LLEN notifications

-- Trim list
LTRIM notifications 0 99                     -- keep only first 100

-- Blocking pop (wait for element)
BLPOP notifications 10                       -- wait up to 10 seconds

-- Use case: Queue
LPUSH queue "task1"
LPUSH queue "task2"
BRPOP queue 0                                -- blocking pop (wait forever)
```

**4. Sets (Unique Unordered Collections)**

```redis
-- Add members
SADD user:1000:tags "premium"
SADD user:1000:tags "active" "verified"

-- Remove member
SREM user:1000:tags "active"

-- Get all members
SMEMBERS user:1000:tags

-- Check membership
SISMEMBER user:1000:tags "premium"

-- Member count
SCARD user:1000:tags

-- Random member
SRANDMEMBER user:1000:tags
SRANDMEMBER user:1000:tags 2                 -- get 2 random members

-- Pop random member
SPOP user:1000:tags

-- Set operations
SADD set1 "a" "b" "c"
SADD set2 "b" "c" "d"

SINTER set1 set2                             -- intersection: ["b", "c"]
SUNION set1 set2                             -- union: ["a", "b", "c", "d"]
SDIFF set1 set2                              -- difference: ["a"]

-- Store result
SINTERSTORE result set1 set2

-- Move member
SMOVE set1 set2 "a"                          -- move "a" from set1 to set2
```

**5. Sorted Sets (Ordered by Score)**

```redis
-- Add members with scores
ZADD leaderboard 1000 "player1"
ZADD leaderboard 1500 "player2"
ZADD leaderboard 1200 "player3"

-- Add multiple
ZADD leaderboard 800 "player4" 950 "player5"

-- Get rank (0-based)
ZRANK leaderboard "player1"                  -- ascending rank
ZREVRANK leaderboard "player1"               -- descending rank

-- Get score
ZSCORE leaderboard "player1"

-- Increment score
ZINCRBY leaderboard 100 "player1"

-- Get range by rank
ZRANGE leaderboard 0 9                       -- top 10 (ascending)
ZREVRANGE leaderboard 0 9                    -- top 10 (descending)
ZRANGE leaderboard 0 9 WITHSCORES            -- with scores

-- Get range by score
ZRANGEBYSCORE leaderboard 1000 2000
ZRANGEBYSCORE leaderboard -inf +inf          -- all
ZRANGEBYSCORE leaderboard 1000 +inf LIMIT 0 10  -- pagination

-- Count by score range
ZCOUNT leaderboard 1000 2000

-- Remove member
ZREM leaderboard "player1"

-- Remove by rank
ZREMRANGEBYRANK leaderboard 0 2              -- remove lowest 3

-- Remove by score
ZREMRANGEBYSCORE leaderboard 0 999

-- Sorted set size
ZCARD leaderboard
```

### **Advanced Features**

**1. TTL and Expiration**

```redis
-- Set expiration
EXPIRE key 3600                              -- expires in 1 hour
EXPIREAT key 1728000000                      -- expires at timestamp

-- Check TTL
TTL key                                      -- seconds remaining
PTTL key                                     -- milliseconds remaining

-- Remove expiration
PERSIST key

-- Set with expiration
SET session:abc EX 3600 "data"
SETEX session:abc 3600 "data"
```

**2. Transactions**

```redis
-- Multi-Exec transaction
MULTI
SET key1 "value1"
SET key2 "value2"
INCR counter
EXEC

-- Discard transaction
MULTI
SET key "value"
DISCARD

-- Watch (optimistic locking)
WATCH counter
val = GET counter
MULTI
SET counter val+1
EXEC                                         -- fails if counter changed
```

**3. Pub/Sub (Messaging)**

```redis
-- Subscribe to channel
SUBSCRIBE news

-- Subscribe to pattern
PSUBSCRIBE news:*

-- Publish message
PUBLISH news "Breaking news!"
PUBLISH news:sports "Team wins championship!"

-- Unsubscribe
UNSUBSCRIBE news
PUNSUBSCRIBE news:*
```

**4. Lua Scripting**

```redis
-- Execute Lua script
EVAL "return redis.call('SET', KEYS[1], ARGV[1])" 1 mykey myvalue

-- Atomic increment with limit
EVAL "
    local current = redis.call('GET', KEYS[1])
    if current == false or tonumber(current) < tonumber(ARGV[1]) then
        return redis.call('INCR', KEYS[1])
    else
        return nil
    end
" 1 counter 100

-- Load script (returns SHA)
SCRIPT LOAD "return redis.call('GET', KEYS[1])"

-- Execute loaded script
EVALSHA sha1 1 mykey
```

### **Common Use Cases**

**1. Caching**

```redis
-- Cache user data
SET cache:user:1000 "{\"name\":\"John\",\"email\":\"john@example.com\"}" EX 3600

-- Get cached data
GET cache:user:1000

-- Cache with pattern
KEYS cache:user:*                            -- find all user caches
DEL cache:user:1000                          -- invalidate cache
```

**2. Session Management**

```redis
-- Store session
HMSET session:abc123 user_id 1000 username "john" logged_in_at 1728000000
EXPIRE session:abc123 1800                   -- 30 minutes

-- Get session
HGETALL session:abc123

-- Update session expiration on activity
EXPIRE session:abc123 1800

-- Delete session (logout)
DEL session:abc123
```

**3. Rate Limiting**

```redis
-- Simple rate limiting
SET rate:user:1000 0 EX 60                   -- reset every 60 seconds
INCR rate:user:1000
GET rate:user:1000                           -- check count

-- Sliding window rate limiting (Lua script)
EVAL "
    local current = redis.call('INCR', KEYS[1])
    if current == 1 then
        redis.call('EXPIRE', KEYS[1], ARGV[1])
    end
    return current
" 1 rate:user:1000 60

-- Token bucket rate limiting (sorted set)
ZADD rate:user:1000 timestamp1 request1
ZADD rate:user:1000 timestamp2 request2
ZREMRANGEBYSCORE rate:user:1000 0 (now-window)  -- remove old requests
ZCARD rate:user:1000                         -- count requests in window
```

**4. Leaderboards**

```redis
-- Add player score
ZADD leaderboard:global 1500 "player123"

-- Update score
ZINCRBY leaderboard:global 100 "player123"

-- Get top 10
ZREVRANGE leaderboard:global 0 9 WITHSCORES

-- Get player rank
ZREVRANK leaderboard:global "player123"

-- Get player's neighbors
ZREVRANGE leaderboard:global (rank-5) (rank+5) WITHSCORES
```

**5. Real-time Analytics**

```redis
-- Page views counter
INCR analytics:page_views:home
INCRBY analytics:page_views:home 5

-- Unique visitors (HyperLogLog)
PFADD analytics:unique_visitors:2025-10-04 "user1" "user2"
PFADD analytics:unique_visitors:2025-10-04 "user1" "user3"  -- user1 counted once
PFCOUNT analytics:unique_visitors:2025-10-04  -- approximate unique count

-- Time-series data (sorted set)
ZADD analytics:response_times timestamp1 150
ZADD analytics:response_times timestamp2 200
ZRANGEBYSCORE analytics:response_times (now-3600) +inf  -- last hour
```

**6. Distributed Locks**

```redis
-- Acquire lock
SET lock:resource:123 "token" NX EX 30       -- only if not exists, 30s TTL

-- Release lock (Lua script for atomicity)
EVAL "
    if redis.call('GET', KEYS[1]) == ARGV[1] then
        return redis.call('DEL', KEYS[1])
    else
        return 0
    end
" 1 lock:resource:123 "token"
```

**7. Message Queue**

```redis
-- Producer: Add task to queue
LPUSH queue:tasks "{\"task\":\"send_email\",\"to\":\"john@example.com\"}"

-- Consumer: Get task (blocking)
BRPOP queue:tasks 10                         -- wait up to 10 seconds

-- Reliable queue (with processing list)
RPOPLPUSH queue:tasks queue:processing
-- Process task...
LREM queue:processing 1 task                 -- remove after processing
```

### **Performance Tips**

```redis
-- Use pipelining for multiple commands
-- Instead of:
GET key1
GET key2
GET key3

-- Do:
PIPELINE
GET key1
GET key2
GET key3
EXEC

-- Use SCAN instead of KEYS (non-blocking)
-- Bad: KEYS user:*                           -- blocks server
-- Good: SCAN 0 MATCH user:* COUNT 100       -- iterates in chunks

-- Monitor performance
INFO stats
SLOWLOG GET 10                               -- show slow queries
CLIENT LIST                                  -- show connected clients

-- Memory optimization
MEMORY DOCTOR                                -- get memory advice
MEMORY STATS
```

### **Persistence Options**

```
1. RDB (Redis Database):
   - Point-in-time snapshots
   - Compact, good for backups
   - May lose data between snapshots

2. AOF (Append Only File):
   - Logs every write operation
   - Better durability
   - Larger file size

3. Hybrid (RDB + AOF):
   - Best of both worlds
   - Recommended for production
```

**Configuration:**
```
-- RDB
save 900 1       # save if 1 key changed in 900 seconds
save 300 10      # save if 10 keys changed in 300 seconds
save 60 10000    # save if 10000 keys changed in 60 seconds

-- AOF
appendonly yes
appendfsync everysec  # or: always, no
```

---

## When to Use SQL vs NoSQL

### Q14: How to Decide Between SQL and NoSQL?

**Answer:**

The choice depends on multiple factors. Here's a comprehensive decision framework:

### **Choose SQL Database When:**

**1. ACID Compliance is Critical**
```
Use Cases:
- Financial transactions
- Banking systems
- Payment processing
- Order management
- Inventory systems

Example: E-commerce order processing
SQL ensures atomicity: Either entire order succeeds (payment + inventory update)
or entire order fails. No partial states.
```

**2. Complex Relationships and JOINs**
```sql
-- SQL excels at complex relational queries
SELECT
    c.name as customer,
    o.order_date,
    p.product_name,
    oi.quantity,
    (oi.quantity * oi.price) as total
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
WHERE c.city = 'New York'
  AND o.order_date >= '2025-01-01'
ORDER BY o.order_date DESC;

-- NoSQL would require multiple queries or denormalization
```

**3. Structured, Well-Defined Schema**
```
Use Cases:
- Data structure unlikely to change
- Strong data validation required
- Referential integrity important

Examples:
- Employee records
- Product catalogs (consistent attributes)
- Financial records
- Medical records
```

**4. Complex Reporting and Analytics**
```sql
-- SQL is excellent for ad-hoc queries and reporting
SELECT
    department,
    AVG(salary) as avg_salary,
    COUNT(*) as emp_count,
    MAX(salary) - MIN(salary) as salary_range
FROM employees
WHERE hire_date >= '2020-01-01'
GROUP BY department
HAVING AVG(salary) > 60000
ORDER BY avg_salary DESC;

-- With window functions
SELECT
    name,
    salary,
    department,
    AVG(salary) OVER (PARTITION BY department) as dept_avg,
    RANK() OVER (ORDER BY salary DESC) as salary_rank
FROM employees;
```

**5. Need for Strong Consistency**
```
SQL provides immediate consistency:
- Read after write returns latest data
- No eventual consistency delays
- Critical for applications where stale data is unacceptable

Examples:
- Banking balances
- Inventory levels
- Booking systems (seats, rooms, tickets)
```

### **Choose NoSQL Database When:**

**1. Flexible/Evolving Schema**
```javascript
// MongoDB: Different products can have different attributes
{
    "type": "laptop",
    "brand": "Apple",
    "cpu": "M2",
    "ram": "16GB",
    "display": "14-inch"
}

{
    "type": "shirt",
    "brand": "Nike",
    "size": "L",
    "color": "blue",
    "material": "cotton"
}

// SQL would need nullable columns or EAV pattern (complex)
```

**2. High Write/Read Throughput**
```
NoSQL (especially Cassandra, DynamoDB) handles:
- Millions of writes per second
- Horizontal scaling
- Low latency at scale

Use Cases:
- IoT sensor data
- Log aggregation
- Real-time analytics
- Social media feeds
```

**3. Horizontal Scalability Needed**
```
NoSQL sharding is built-in:

Cassandra Example:
Node1: Users A-F
Node2: Users G-M
Node3: Users N-S
Node4: Users T-Z

Adding more nodes is seamless.

SQL requires complex setup for sharding.
```

**4. Simple Access Patterns**
```javascript
// NoSQL excels at key-value lookups
db.users.findOne({ _id: "user123" })         // Very fast

// Redis
GET session:abc123                           // Microsecond latency

// DynamoDB
GetItem({ TableName: "Users", Key: { userId: "123" } })
```

**5. Unstructured/Semi-Structured Data**
```javascript
// Logs with varying fields
{
    "timestamp": "2025-10-04T10:30:00Z",
    "level": "ERROR",
    "message": "Database connection failed",
    "stack_trace": "...",
    "user_id": "123"
}

{
    "timestamp": "2025-10-04T10:31:00Z",
    "level": "INFO",
    "message": "User logged in",
    "user_id": "456",
    "ip_address": "192.168.1.1"
}

// SQL would need JSONB or sparse columns
```

### **Decision Matrix**

| Requirement | SQL | NoSQL |
|-------------|-----|-------|
| ACID transactions | ✅ Excellent | ⚠️ Limited (document-level) |
| Horizontal scaling | ⚠️ Complex | ✅ Built-in |
| Complex queries | ✅ Excellent (JOINs, subqueries) | ⚠️ Limited |
| Schema flexibility | ⚠️ Requires migrations | ✅ Dynamic |
| Consistency | ✅ Strong | ⚠️ Eventual (configurable) |
| Read performance | ⚠️ Good with indexes | ✅ Excellent |
| Write performance | ⚠️ Good | ✅ Excellent |
| Data relationships | ✅ Native (foreign keys) | ⚠️ Embedded or denormalized |
| Learning curve | ✅ Standardized SQL | ⚠️ Database-specific |
| Maturity | ✅ Decades of development | ⚠️ Relatively newer |

### **Real-World Decision Examples**

**Example 1: Social Media Application**

```
Requirements:
- User profiles with flexible attributes
- High write volume (posts, likes, comments)
- Real-time feeds
- Horizontal scaling for millions of users
- Eventual consistency acceptable

Decision: NoSQL (MongoDB + Redis)

Architecture:
- MongoDB: User profiles, posts, comments
- Redis: Session management, real-time feeds, caching
- PostgreSQL: Financial data (ads, payments)

Why:
✓ Flexible schema for user profiles
✓ High write throughput for social activity
✓ Easy horizontal scaling
✓ Fast key-value lookups for sessions
```

**Example 2: E-commerce Platform**

```
Requirements:
- Product catalog with varying attributes
- Order processing with ACID guarantees
- Inventory management
- Complex reporting
- User sessions and caching

Decision: Hybrid (PostgreSQL + MongoDB + Redis)

Architecture:
- PostgreSQL: Orders, inventory, transactions
- MongoDB: Product catalog, user activity logs
- Redis: Sessions, caching, cart data

Why:
✓ PostgreSQL for transactional integrity (orders)
✓ MongoDB for flexible product attributes
✓ Redis for high-performance caching
```

**Example 3: Banking Application**

```
Requirements:
- Strong ACID compliance
- Complex financial calculations
- Regulatory compliance
- Audit trails
- Data integrity critical

Decision: SQL (PostgreSQL or Oracle)

Why:
✅ ACID transactions absolutely critical
✅ Complex relationships (accounts, transactions, customers)
✅ Strong consistency required
✅ Mature ecosystem for compliance
✅ Regulatory requirements often mandate SQL
```

**Example 4: IoT Sensor Platform**

```
Requirements:
- Millions of sensor readings per second
- Time-series data
- Horizontal scalability
- Simple queries (by sensor, by time range)
- Eventual consistency OK

Decision: NoSQL (Cassandra or TimescaleDB)

Architecture:
- Cassandra: Time-series sensor data
- TimescaleDB: If SQL interface preferred
- Redis: Real-time aggregations

Why:
✓ Extreme write throughput
✓ Built-in time-series optimization
✓ Linear horizontal scalability
✓ Simple query patterns
```

### **Migration Considerations**

**SQL → NoSQL Migration**

```
When to Migrate:
1. Hitting scaling limits
2. Schema changes too frequent
3. Need better write performance
4. Global distribution required

Challenges:
- Losing JOINs (need to denormalize)
- Application logic changes
- Transaction handling
- Data modeling differences

Strategy:
1. Identify bounded contexts
2. Start with non-critical components
3. Dual-write pattern (temporary)
4. Gradual migration
```

**NoSQL → SQL Migration**

```
When to Migrate:
1. Need complex queries
2. Require ACID transactions
3. Data relationships becoming important
4. Eventual consistency causing issues

Challenges:
- Schema design
- Normalizing nested data
- Performance during migration

Strategy:
1. Extract schema from documents
2. Design normalized model
3. ETL pipeline for data
4. Update application code
```

### **Polyglot Persistence (Best Approach)**

```
Use multiple databases for different purposes:

┌─────────────────────────────────────┐
│         Application Layer           │
└─────────────────────────────────────┘
         │         │         │
         ↓         ↓         ↓
    ┌────────┐ ┌────────┐ ┌────────┐
    │  SQL   │ │ NoSQL  │ │ Redis  │
    │(Orders)│ │(Catalog)│ │(Cache) │
    └────────┘ └────────┘ └────────┘

Example: E-commerce
- PostgreSQL: Orders, payments, inventory (ACID)
- MongoDB: Product catalog (flexible schema)
- Redis: Sessions, cart, cache (speed)
- Elasticsearch: Product search (full-text)
- Cassandra: User activity logs (high writes)
```

### **Final Decision Framework**

```
Ask these questions:

1. Data Structure:
   Q: Is schema well-defined and stable?
   Yes → SQL | No → NoSQL

2. Scalability:
   Q: Need to scale horizontally to massive size?
   Yes → NoSQL | No → SQL fine

3. Consistency:
   Q: Is strong consistency critical?
   Yes → SQL | No → NoSQL OK

4. Queries:
   Q: Need complex JOINs and ad-hoc queries?
   Yes → SQL | No → NoSQL

5. Transactions:
   Q: Need multi-record ACID transactions?
   Yes → SQL | No → NoSQL

6. Performance:
   Q: Extreme read/write performance needed?
   Yes → NoSQL | No → SQL fine

7. Team:
   Q: What's your team familiar with?
   Consider learning curve

8. Budget:
   Consider licensing, operational costs
```

### **Common Myths**

```
Myth 1: "NoSQL is always faster"
Reality: SQL can be very fast with proper indexing.
         NoSQL trades features for speed in specific scenarios.

Myth 2: "SQL doesn't scale"
Reality: SQL can scale (PostgreSQL, MySQL clusters).
         NoSQL scales more easily, but SQL can handle billions of rows.

Myth 3: "NoSQL means no schema"
Reality: Schema is often enforced at application level.
         Some NoSQL DBs (MongoDB) support schema validation.

Myth 4: "You must choose one"
Reality: Polyglot persistence is common and recommended.
         Use the right tool for each job.
```

---

## Advanced Topics

### Q15: What are Database Sharding and Partitioning?

**Answer:**

**Partitioning**: Dividing a large table into smaller pieces within the same database instance.

**Sharding**: Distributing data across multiple database instances/servers.

**Horizontal Partitioning (Sharding)**
```sql
-- Divide users table by user_id ranges
-- Shard 1: user_id 1-1,000,000
-- Shard 2: user_id 1,000,001-2,000,000
-- Shard 3: user_id 2,000,001-3,000,000

-- Application logic determines which shard
shard_id = user_id / 1_000_000
shard = shards[shard_id]
shard.query("SELECT * FROM users WHERE user_id = ?", user_id)

-- Hash-based sharding
shard_id = hash(user_id) % num_shards
```

**Vertical Partitioning**
```sql
-- Split table by columns
-- Hot data (frequently accessed)
CREATE TABLE users_hot (
    user_id INT PRIMARY KEY,
    email VARCHAR(100),
    name VARCHAR(100),
    last_login TIMESTAMP
);

-- Cold data (rarely accessed)
CREATE TABLE users_cold (
    user_id INT PRIMARY KEY,
    bio TEXT,
    preferences JSON,
    created_at TIMESTAMP
);
```

**Range Partitioning**
```sql
-- PostgreSQL example
CREATE TABLE orders (
    order_id INT,
    order_date DATE,
    customer_id INT,
    total DECIMAL(10,2)
) PARTITION BY RANGE (order_date);

CREATE TABLE orders_2024_q1 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2024-04-01');

CREATE TABLE orders_2024_q2 PARTITION OF orders
    FOR VALUES FROM ('2024-04-01') TO ('2024-07-01');

-- Queries automatically route to correct partition
SELECT * FROM orders WHERE order_date = '2024-05-15';
-- Only queries orders_2024_q2
```

---

### Q16: What is Database Replication?

**Answer:**

Replication copies data from one database (master) to one or more databases (replicas).

**Master-Slave Replication**
```
┌─────────┐
│ Master  │  (Writes)
└────┬────┘
     │
     ├───────┬──────────┐
     ↓       ↓          ↓
 ┌───────┐ ┌───────┐ ┌───────┐
 │Slave 1│ │Slave 2│ │Slave 3│  (Reads)
 └───────┘ └───────┘ └───────┘

Benefits:
✓ Read scalability
✓ Backup/disaster recovery
✓ Reduced load on master

Drawbacks:
⚠️ Replication lag (eventual consistency)
⚠️ Master is single point of failure for writes
```

**Multi-Master Replication**
```
┌─────────┐  ←→  ┌─────────┐
│Master 1 │      │Master 2 │
└─────────┘  ←→  └─────────┘
  (Writes)        (Writes)

Benefits:
✓ No single point of failure
✓ Better write scalability
✓ Geographic distribution

Drawbacks:
⚠️ Conflict resolution needed
⚠️ More complex
```

---

### Q17: What is CAP Theorem?

**Answer:**

CAP Theorem states that a distributed system can only guarantee 2 out of 3:

**C (Consistency)**: All nodes see the same data at the same time
**A (Availability)**: Every request receives a response
**P (Partition Tolerance)**: System continues to operate despite network failures

```
Network partition occurs:
Node 1 ←  X  → Node 2
(Can't communicate)

Choose 2:

CP (Consistency + Partition Tolerance):
- Reject requests until partition healed
- Examples: MongoDB, HBase, Redis
- Use when: Data accuracy critical

AP (Availability + Partition Tolerance):
- Accept requests, allow inconsistency
- Examples: Cassandra, DynamoDB, CouchDB
- Use when: Availability critical

CA (Consistency + Availability):
- Only works if no partitions
- Not realistic in distributed systems
- Examples: Traditional RDBMS in single location
```

**Real-World Example:**
```
Banking App (CP):
ATM withdrawal must check actual balance
Better to reject request than allow overdraft

Social Media (AP):
Like counts can be eventually consistent
Better to show approximate count than fail
```

---

### Q18: Explain EXPLAIN and Query Optimization

**Answer:**

```sql
-- EXPLAIN shows query execution plan
EXPLAIN SELECT * FROM users WHERE email = 'john@example.com';

-- Output interpretation:
-- type: ALL = full table scan (slow)
-- type: index = index scan
-- type: ref = index lookup (fast)
-- type: const = constant lookup (fastest)

-- rows: estimated rows scanned
-- key: which index used (NULL = no index)

-- Example optimization:
-- Before (slow):
EXPLAIN SELECT * FROM orders WHERE customer_id = 123;
-- type: ALL, rows: 1000000

-- Add index:
CREATE INDEX idx_customer ON orders(customer_id);

-- After (fast):
EXPLAIN SELECT * FROM orders WHERE customer_id = 123;
-- type: ref, rows: 10, key: idx_customer

-- EXPLAIN ANALYZE (PostgreSQL) - actual execution
EXPLAIN ANALYZE SELECT * FROM large_table WHERE status = 'active';
-- Shows actual time, not just estimates
```

---

This comprehensive guide covers all major database interview topics with SQL examples and real-world scenarios. Practice these concepts and queries to prepare for database interviews!
