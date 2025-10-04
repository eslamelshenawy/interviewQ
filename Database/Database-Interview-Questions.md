# أسئلة مقابلات Database الشاملة

## المحتويات
1. [SQL - أساسيات](#sql---أساسيات)
2. [SQL - متقدم](#sql---متقدم)
3. [Joins](#joins)
4. [Indexes](#indexes)
5. [Transactions](#transactions)
6. [Normalization](#normalization)
7. [NoSQL](#nosql)
8. [MongoDB](#mongodb)
9. [Redis](#redis)
10. [Database Design](#database-design)

---

## SQL - أساسيات

### 1. ما الفرق بين WHERE و HAVING؟

**الإجابة:**

| WHERE | HAVING |
|-------|--------|
| يُستخدم مع الصفوف (rows) | يُستخدم مع المجموعات (groups) |
| قبل GROUP BY | بعد GROUP BY |
| لا يمكن استخدام aggregate functions | يمكن استخدام aggregate functions |

**مثال:**
```sql
-- WHERE - تصفية الصفوف
SELECT department, AVG(salary)
FROM employees
WHERE age > 25  -- تصفية قبل التجميع
GROUP BY department;

-- HAVING - تصفية المجموعات
SELECT department, AVG(salary) as avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000;  -- تصفية بعد التجميع

-- استخدام الاثنين معاً
SELECT department, AVG(salary) as avg_salary
FROM employees
WHERE age > 25                    -- تصفية الصفوف
GROUP BY department
HAVING AVG(salary) > 50000;       -- تصفية المجموعات
```

---

### 2. ما الفرق بين DELETE و TRUNCATE و DROP؟

**الإجابة:**

#### DELETE
```sql
-- حذف صفوف محددة
DELETE FROM employees WHERE id = 5;

-- حذف كل الصفوف
DELETE FROM employees;
```
- ✅ يمكن استخدام WHERE
- ✅ يمكن ROLLBACK
- ✅ يشغل Triggers
- ❌ أبطأ

#### TRUNCATE
```sql
-- حذف كل الصفوف
TRUNCATE TABLE employees;
```
- ✅ أسرع من DELETE
- ✅ يعيد Auto-increment لـ 0
- ❌ لا يمكن استخدام WHERE
- ❌ لا يمكن ROLLBACK (في معظم الحالات)
- ❌ لا يشغل Triggers

#### DROP
```sql
-- حذف الجدول بالكامل
DROP TABLE employees;
```
- ❌ يحذف الجدول والبنية
- ❌ لا رجوع فيه
- ❌ يحذف كل شيء (بيانات + بنية)

**مثال عملي:**
```sql
-- Scenario 1: حذف موظف واحد
DELETE FROM employees WHERE id = 100;

-- Scenario 2: حذف كل البيانات والبدء من جديد
TRUNCATE TABLE test_data;

-- Scenario 3: الجدول لم يعد مطلوباً
DROP TABLE old_logs;
```

---

### 3. ما الفرق بين UNION و UNION ALL؟

**الإجابة:**

#### UNION
```sql
-- يحذف المكرر
SELECT name FROM employees
UNION
SELECT name FROM contractors;
```
- ✅ يحذف الصفوف المكررة
- ❌ أبطأ (يحتاج لفحص التكرار)

#### UNION ALL
```sql
-- يبقي المكرر
SELECT name FROM employees
UNION ALL
SELECT name FROM contractors;
```
- ✅ أسرع
- ✅ يبقي كل الصفوف حتى المكررة
- ✅ استخدمه إذا كنت متأكد أنه لا يوجد تكرار

**مثال:**
```sql
-- Table 1
SELECT 'Ahmed' as name
UNION
SELECT 'Sara'
UNION
SELECT 'Ahmed';  -- مكرر

-- النتيجة: Ahmed, Sara (حذف التكرار)

-- مع UNION ALL
SELECT 'Ahmed'
UNION ALL
SELECT 'Sara'
UNION ALL
SELECT 'Ahmed';

-- النتيجة: Ahmed, Sara, Ahmed (كل الصفوف)
```

---

### 4. ما هو Primary Key؟ وما الفرق بينه وبين Unique Key؟

**الإجابة:**

#### Primary Key
```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,        -- معرّف فريد
    email VARCHAR(100)
);
```
- ✅ فريد (Unique)
- ✅ لا يقبل NULL
- ✅ جدول واحد فقط per table
- ✅ يُنشئ index تلقائياً

#### Unique Key
```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    email VARCHAR(100) UNIQUE,  -- فريد لكن يمكن NULL
    phone VARCHAR(20) UNIQUE
);
```
- ✅ فريد (Unique)
- ✅ يقبل NULL (مرة واحدة أو أكثر حسب DBMS)
- ✅ يمكن عدة unique keys per table

**الفرق:**
```sql
-- Primary Key
INSERT INTO employees VALUES (1, 'ahmed@email.com');
INSERT INTO employees VALUES (NULL, 'sara@email.com');  -- ❌ Error

-- Unique Key
INSERT INTO employees VALUES (1, 'ahmed@email.com', '123');
INSERT INTO employees VALUES (2, 'sara@email.com', NULL);   -- ✅ OK
INSERT INTO employees VALUES (3, 'ali@email.com', NULL);    -- ✅ OK (في MySQL)
```

---

### 5. ما هو Foreign Key؟

**الإجابة:**

Foreign Key يُنشئ علاقة بين جدولين.

**مثال:**
```sql
-- الجدول الأساسي (Parent)
CREATE TABLE departments (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

-- الجدول الفرعي (Child)
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(id)
);

-- إدخال بيانات
INSERT INTO departments VALUES (1, 'IT');
INSERT INTO departments VALUES (2, 'HR');

INSERT INTO employees VALUES (1, 'Ahmed', 1);   -- ✅ OK
INSERT INTO employees VALUES (2, 'Sara', 5);    -- ❌ Error: department 5 لا يوجد
```

**Cascade Options:**
```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(id)
        ON DELETE CASCADE      -- عند حذف department، احذف الموظفين
        ON UPDATE CASCADE      -- عند تحديث id، حدث الموظفين
);

-- حذف department
DELETE FROM departments WHERE id = 1;
-- سيحذف تلقائياً كل employees في department 1
```

**أنواع Cascade:**
- `CASCADE`: ينفذ نفس العملية
- `SET NULL`: يضع NULL
- `RESTRICT`: يمنع الحذف/التحديث
- `NO ACTION`: مثل RESTRICT

---

## SQL - متقدم

### 6. ما هي Subqueries؟

**الإجابة:**

Subquery هو استعلام داخل استعلام آخر.

#### أنواع Subqueries:

**1. Single Row Subquery**
```sql
-- أوجد الموظف الذي راتبه يساوي متوسط الرواتب
SELECT name, salary
FROM employees
WHERE salary = (SELECT AVG(salary) FROM employees);
```

**2. Multi Row Subquery**
```sql
-- أوجد الموظفين في أقسام معينة
SELECT name
FROM employees
WHERE department_id IN (
    SELECT id
    FROM departments
    WHERE location = 'Cairo'
);
```

**3. Correlated Subquery**
```sql
-- أوجد الموظفين الذين رواتبهم أعلى من متوسط قسمهم
SELECT e1.name, e1.salary, e1.department_id
FROM employees e1
WHERE e1.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e1.department_id
);
```

**4. Subquery في SELECT**
```sql
SELECT
    name,
    salary,
    (SELECT AVG(salary) FROM employees) as avg_salary,
    salary - (SELECT AVG(salary) FROM employees) as difference
FROM employees;
```

**5. Subquery في FROM**
```sql
SELECT dept_name, avg_salary
FROM (
    SELECT department_id, AVG(salary) as avg_salary
    FROM employees
    GROUP BY department_id
) as dept_salaries
JOIN departments ON departments.id = dept_salaries.department_id;
```

---

### 7. ما الفرق بين IN و EXISTS؟

**الإجابة:**

#### IN
```sql
-- يُرجع كل القيم ثم يقارن
SELECT name
FROM employees
WHERE department_id IN (
    SELECT id FROM departments WHERE location = 'Cairo'
);
```
- يُنفذ الـ subquery أولاً
- يُرجع كل النتائج
- جيد للبيانات الصغيرة

#### EXISTS
```sql
-- يتوقف عند أول match
SELECT name
FROM employees e
WHERE EXISTS (
    SELECT 1
    FROM departments d
    WHERE d.id = e.department_id
    AND d.location = 'Cairo'
);
```
- يتوقف عند أول نتيجة true
- أسرع للبيانات الكبيرة
- يرجع true/false فقط

**الفرق في الأداء:**
```sql
-- IN - للقوائم الصغيرة
WHERE department_id IN (1, 2, 3)

-- EXISTS - للبيانات الكبيرة
WHERE EXISTS (subquery)
```

---

## Joins

### 8. اشرح أنواع Joins

**الإجابة:**

#### 1. INNER JOIN
```sql
-- فقط الصفوف المتطابقة في الجدولين
SELECT e.name, d.name as department
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;
```

**مثال:**
```
Employees:              Departments:
id | name    | dept_id   id | name
1  | Ahmed   | 1         1  | IT
2  | Sara    | 2         2  | HR
3  | Ali     | NULL      3  | Sales

النتيجة (INNER JOIN):
Ahmed  | IT
Sara   | HR
```

#### 2. LEFT JOIN (LEFT OUTER JOIN)
```sql
-- كل الصفوف من الجدول الأيسر
SELECT e.name, d.name as department
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;
```

**النتيجة:**
```
Ahmed  | IT
Sara   | HR
Ali    | NULL
```

#### 3. RIGHT JOIN (RIGHT OUTER JOIN)
```sql
-- كل الصفوف من الجدول الأيمن
SELECT e.name, d.name as department
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

**النتيجة:**
```
Ahmed  | IT
Sara   | HR
NULL   | Sales
```

#### 4. FULL OUTER JOIN
```sql
-- كل الصفوف من الجدولين
SELECT e.name, d.name as department
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.id;
```

**النتيجة:**
```
Ahmed  | IT
Sara   | HR
Ali    | NULL
NULL   | Sales
```

#### 5. CROSS JOIN
```sql
-- كل الاحتمالات (Cartesian Product)
SELECT e.name, d.name
FROM employees e
CROSS JOIN departments d;
```

**النتيجة:** 3 employees × 3 departments = 9 rows

#### 6. SELF JOIN
```sql
-- Join الجدول مع نفسه
SELECT
    e1.name as employee,
    e2.name as manager
FROM employees e1
LEFT JOIN employees e2 ON e1.manager_id = e2.id;
```

---

### 9. ما الفرق بين INNER JOIN و WHERE؟

**الإجابة:**

```sql
-- INNER JOIN (الطريقة الحديثة)
SELECT e.name, d.name
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;

-- WHERE (الطريقة القديمة)
SELECT e.name, d.name
FROM employees e, departments d
WHERE e.department_id = d.id;
```

**الفرق:**
- ✅ INNER JOIN أوضح وأسهل قراءة
- ✅ INNER JOIN يفصل شرط الـ join عن شرط الـ filter
- ✅ WHERE قديمة ولا يُنصح بها

**مثال يوضح الفرق:**
```sql
-- مع INNER JOIN - واضح
SELECT e.name, d.name
FROM employees e
INNER JOIN departments d ON e.department_id = d.id
WHERE e.salary > 50000;

-- مع WHERE - غير واضح
SELECT e.name, d.name
FROM employees e, departments d
WHERE e.department_id = d.id
AND e.salary > 50000;
```

---

## Indexes

### 10. ما هو Index؟ ومتى نستخدمه؟

**الإجابة:**

Index هو بنية بيانات تُسرّع عمليات البحث.

**بدون Index:**
```sql
-- Full Table Scan
SELECT * FROM employees WHERE email = 'ahmed@email.com';
-- يفحص كل الصفوف واحدة واحدة
```

**مع Index:**
```sql
-- إنشاء index
CREATE INDEX idx_email ON employees(email);

-- الآن البحث أسرع
SELECT * FROM employees WHERE email = 'ahmed@email.com';
-- يستخدم Index للوصول مباشرة
```

#### أنواع Indexes:

**1. Single Column Index**
```sql
CREATE INDEX idx_name ON employees(name);
```

**2. Composite Index**
```sql
CREATE INDEX idx_name_dept ON employees(name, department_id);
```

**3. Unique Index**
```sql
CREATE UNIQUE INDEX idx_email ON employees(email);
```

**4. Full-Text Index**
```sql
CREATE FULLTEXT INDEX idx_description ON products(description);
```

#### متى نستخدم Index:

✅ **استخدم Index عندما:**
- عمود يُستخدم كثيراً في WHERE
- عمود يُستخدم في JOIN
- عمود يُستخدم في ORDER BY
- عمود يُستخدم في GROUP BY

❌ **لا تستخدم Index عندما:**
- الجدول صغير
- عمود يتغير كثيراً (INSERT/UPDATE كثيرة)
- عمود له قيم قليلة متكررة (مثل gender: M/F)

**مثال:**
```sql
-- جيد للـ index
CREATE INDEX idx_email ON users(email);
-- email فريد ويُستخدم كثيراً في WHERE

-- سيء للـ index
CREATE INDEX idx_gender ON users(gender);
-- gender له قيمتين فقط (M/F)
```

#### مشاكل Indexes:

```sql
-- مشكلة 1: تبطئ INSERT/UPDATE/DELETE
INSERT INTO employees VALUES (...);
-- يجب تحديث الـ index أيضاً

-- مشكلة 2: تستهلك مساحة تخزين
-- كل index يأخذ مساحة إضافية
```

---

### 11. ما هو Composite Index؟ وما ترتيب الأعمدة؟

**الإجابة:**

Composite Index هو index على عدة أعمدة.

```sql
-- إنشاء composite index
CREATE INDEX idx_name_age ON employees(name, age);
```

**ترتيب الأعمدة مهم:**

```sql
-- Index: (name, age)

-- ✅ سيستخدم Index
SELECT * FROM employees WHERE name = 'Ahmed' AND age = 25;
SELECT * FROM employees WHERE name = 'Ahmed';

-- ❌ لن يستخدم Index بكفاءة
SELECT * FROM employees WHERE age = 25;
```

**القاعدة:**
- ضع العمود الأكثر استخداماً أولاً
- ضع العمود الأكثر تحديداً أولاً

**مثال:**
```sql
-- خيار 1
CREATE INDEX idx_dept_name ON employees(department_id, name);
-- جيد إذا كنت تبحث غالباً بـ department أولاً

-- خيار 2
CREATE INDEX idx_name_dept ON employees(name, department_id);
-- جيد إذا كنت تبحث غالباً بـ name أولاً
```

---

## Transactions

### 12. ما هو Transaction؟ وما هي ACID؟

**الإجابة:**

Transaction هو مجموعة من العمليات تُنفذ كوحدة واحدة.

```sql
-- Transaction
START TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;  -- نجح
-- أو
ROLLBACK;  -- فشل
```

#### ACID Properties:

**A - Atomicity (الذرية)**
```sql
-- كل العمليات تنفذ أو لا شيء
BEGIN TRANSACTION;
    UPDATE account SET balance = balance - 100 WHERE id = 1;
    UPDATE account SET balance = balance + 100 WHERE id = 2;
    -- إذا فشلت أي عملية، كل شيء يُلغى
COMMIT;
```

**C - Consistency (الاتساق)**
```sql
-- البيانات صحيحة قبل وبعد
-- مثال: مجموع الأموال لا يتغير
Balance قبل: Account1 = 1000, Account2 = 500 (Total = 1500)
Balance بعد: Account1 = 900, Account2 = 600 (Total = 1500) ✓
```

**I - Isolation (العزل)**
```sql
-- Transactions معزولة عن بعضها
-- Transaction 1
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- Transaction 2 لا يرى التغيير حتى COMMIT
COMMIT;
```

**D - Durability (الديمومة)**
```sql
-- بعد COMMIT، البيانات محفوظة حتى لو انقطع الكهرباء
COMMIT;  -- ← البيانات آمنة الآن
```

---

### 13. ما هي Isolation Levels؟

**الإجابة:**

Isolation Levels تحدد كيف تعزل transactions بعضها عن بعض.

#### 1. READ UNCOMMITTED (الأضعف)
```sql
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

-- يمكن قراءة بيانات لم تُحفظ بعد (Dirty Read)
-- Transaction 1
UPDATE accounts SET balance = 1000 WHERE id = 1;
-- Transaction 2 يمكنه قراءة 1000 حتى قبل COMMIT

-- مشكلة: لو Transaction 1 عمل ROLLBACK؟
```

#### 2. READ COMMITTED
```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- يقرأ فقط البيانات المحفوظة
-- مشكلة: Non-Repeatable Read
```

#### 3. REPEATABLE READ
```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- قراءة متسقة طوال الـ transaction
-- مشكلة: Phantom Read
```

#### 4. SERIALIZABLE (الأقوى)
```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- عزل كامل
-- أبطأ ولكن الأكثر أماناً
```

**الترتيب:**
```
READ UNCOMMITTED < READ COMMITTED < REPEATABLE READ < SERIALIZABLE
سريع/غير آمن <--------------------------> بطيء/آمن
```

---

## Normalization

### 14. ما هو Normalization؟ ولماذا نستخدمه؟

**الإجابة:**

Normalization هو تنظيم البيانات لتقليل التكرار.

#### قبل Normalization (مشاكل):
```sql
CREATE TABLE orders (
    order_id INT,
    customer_name VARCHAR(100),
    customer_email VARCHAR(100),
    customer_phone VARCHAR(20),
    product_name VARCHAR(100),
    product_price DECIMAL(10,2)
);

-- مشاكل:
❌ تكرار بيانات العميل
❌ تكرار بيانات المنتج
❌ صعوبة التحديث
❌ Anomalies
```

#### بعد Normalization:
```sql
-- جدول العملاء
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    phone VARCHAR(20)
);

-- جدول المنتجات
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10,2)
);

-- جدول الطلبات
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    product_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

-- فوائد:
✅ لا تكرار
✅ سهل التحديث
✅ data integrity
```

---

### 15. اشرح Normal Forms

**الإجابة:**

#### 1NF (First Normal Form)
- كل عمود يحتوي على قيمة atomic (لا قوائم)
- كل صف فريد

**قبل 1NF:**
```sql
CREATE TABLE students (
    id INT,
    name VARCHAR(100),
    courses VARCHAR(200)  -- "Math, Physics, Chemistry" ❌
);
```

**بعد 1NF:**
```sql
CREATE TABLE student_courses (
    student_id INT,
    course VARCHAR(100)  -- صف لكل course ✓
);
```

#### 2NF (Second Normal Form)
- 1NF ✓
- لا يوجد Partial Dependency

**قبل 2NF:**
```sql
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    product_name VARCHAR(100),    -- يعتمد فقط على product_id ❌
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);
```

**بعد 2NF:**
```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100)  -- منفصل ✓
);

CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);
```

#### 3NF (Third Normal Form)
- 2NF ✓
- لا يوجد Transitive Dependency

**قبل 3NF:**
```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    department_name VARCHAR(100)  -- يعتمد على department_id ❌
);
```

**بعد 3NF:**
```sql
CREATE TABLE departments (
    id INT PRIMARY KEY,
    name VARCHAR(100)  -- منفصل ✓
);

CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(id)
);
```

---

## NoSQL

### 16. ما الفرق بين SQL و NoSQL؟

**الإجابة:**

راجع ملف [SQL-vs-NoSQL.md](SQL-vs-NoSQL.md) للتفاصيل الكاملة.

**ملخص سريع:**

| SQL | NoSQL |
|-----|-------|
| جداول وعلاقات | Documents, Key-Value, etc. |
| Schema محدد | مرن |
| ACID | BASE |
| Vertical Scaling | Horizontal Scaling |
| MySQL, PostgreSQL | MongoDB, Redis, Cassandra |

---

### 17. ما أنواع NoSQL Databases؟

**الإجابة:**

#### 1. Document Databases (MongoDB)
```javascript
{
    "_id": 1,
    "name": "Ahmed",
    "email": "ahmed@email.com",
    "orders": [
        {"product": "Book", "price": 20},
        {"product": "Pen", "price": 5}
    ]
}
```

**Use Case:** CMS, E-commerce, User Profiles

#### 2. Key-Value Stores (Redis)
```javascript
SET user:1000:session "session_data"
GET user:1000:session

SET product:500:views 1250
INCR product:500:views
```

**Use Case:** Caching, Session Management, Real-time Analytics

#### 3. Column-Family Stores (Cassandra)
```
Row Key: user_123
├── name: "Ahmed"
├── email: "ahmed@email.com"
└── age: 25
```

**Use Case:** Time-series data, IoT, Analytics

#### 4. Graph Databases (Neo4j)
```cypher
(Ahmed)-[:FRIEND]->(Sara)
(Ahmed)-[:LIKES]->(Product)
```

**Use Case:** Social Networks, Recommendation Engines

---

## MongoDB

### 18. أسئلة MongoDB الأساسية

**السؤال 1: كيف تُدخل document في MongoDB؟**

```javascript
// إدخال واحد
db.users.insertOne({
    name: "Ahmed",
    email: "ahmed@email.com",
    age: 25
});

// إدخال عدة
db.users.insertMany([
    {name: "Sara", age: 22},
    {name: "Ali", age: 30}
]);
```

**السؤال 2: كيف تبحث عن documents؟**

```javascript
// كل الـ documents
db.users.find();

// مع شرط
db.users.find({age: {$gt: 25}});

// أول document
db.users.findOne({name: "Ahmed"});

// مع projection
db.users.find({}, {name: 1, email: 1, _id: 0});
```

**السؤال 3: كيف تُحدث documents؟**

```javascript
// تحديث واحد
db.users.updateOne(
    {name: "Ahmed"},
    {$set: {age: 26}}
);

// تحديث عدة
db.users.updateMany(
    {age: {$lt: 18}},
    {$set: {status: "minor"}}
);

// Operators
db.users.updateOne(
    {name: "Ahmed"},
    {
        $set: {email: "new@email.com"},
        $inc: {age: 1},
        $push: {hobbies: "reading"}
    }
);
```

**السؤال 4: كيف تحذف documents؟**

```javascript
// حذف واحد
db.users.deleteOne({name: "Ahmed"});

// حذف عدة
db.users.deleteMany({age: {$lt: 18}});
```

---

### 19. MongoDB Aggregation Pipeline

**السؤال:** اشرح Aggregation Pipeline.

**الإجابة:**

```javascript
db.orders.aggregate([
    // Stage 1: Filter
    {
        $match: {
            status: "completed",
            date: {$gte: new Date("2024-01-01")}
        }
    },

    // Stage 2: Group
    {
        $group: {
            _id: "$customer_id",
            total_spent: {$sum: "$amount"},
            order_count: {$sum: 1}
        }
    },

    // Stage 3: Filter groups
    {
        $match: {
            total_spent: {$gt: 1000}
        }
    },

    // Stage 4: Sort
    {
        $sort: {total_spent: -1}
    },

    // Stage 5: Limit
    {
        $limit: 10
    },

    // Stage 6: Lookup (Join)
    {
        $lookup: {
            from: "customers",
            localField: "_id",
            foreignField: "customer_id",
            as: "customer_info"
        }
    }
]);
```

**Operators الشائعة:**
```javascript
// $match - تصفية
{$match: {age: {$gt: 25}}}

// $group - تجميع
{$group: {_id: "$category", count: {$sum: 1}}}

// $project - اختيار حقول
{$project: {name: 1, age: 1}}

// $sort - ترتيب
{$sort: {age: -1}}

// $limit - تحديد
{$limit: 10}

// $skip - تخطي
{$skip: 20}

// $unwind - فك array
{$unwind: "$items"}

// $lookup - join
{$lookup: {from: "orders", localField: "id", foreignField: "user_id", as: "orders"}}
```

---

## Redis

### 20. أسئلة Redis الأساسية

**السؤال 1: ما أنواع البيانات في Redis؟**

**الإجابة:**

#### 1. Strings
```redis
SET key "value"
GET key
INCR counter
DECR counter
```

#### 2. Lists
```redis
LPUSH mylist "first"
RPUSH mylist "last"
LRANGE mylist 0 -1
```

#### 3. Sets
```redis
SADD myset "item1"
SADD myset "item2"
SMEMBERS myset
```

#### 4. Sorted Sets
```redis
ZADD leaderboard 100 "player1"
ZADD leaderboard 200 "player2"
ZRANGE leaderboard 0 -1 WITHSCORES
```

#### 5. Hashes
```redis
HSET user:1000 name "Ahmed"
HSET user:1000 email "ahmed@email.com"
HGETALL user:1000
```

**السؤال 2: كيف تستخدم Redis للـ Caching؟**

```java
// Java + Redis
Jedis redis = new Jedis("localhost");

// محاولة القراءة من Cache
String user = redis.get("user:1000");

if (user == null) {
    // لا يوجد في Cache، اقرأ من Database
    user = database.getUser(1000);

    // احفظ في Cache لمدة ساعة
    redis.setex("user:1000", 3600, user);
}

return user;
```

**السؤال 3: ما الفرق بين Redis و Memcached؟**

| Redis | Memcached |
|-------|-----------|
| أنواع بيانات متعددة | String فقط |
| Persistence | لا يوجد |
| Single-threaded | Multi-threaded |
| أكثر مرونة | أبسط وأسرع قليلاً |

---

## Database Design

### 21. كيف تصمم Database لتطبيق معين؟

**السؤال:** صمم database لتطبيق E-Commerce.

**الإجابة:**

```sql
-- 1. جدول المستخدمين
CREATE TABLE users (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 2. جدول العناوين
CREATE TABLE addresses (
    address_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT,
    street VARCHAR(200),
    city VARCHAR(100),
    country VARCHAR(100),
    postal_code VARCHAR(20),
    is_default BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

-- 3. جدول الفئات
CREATE TABLE categories (
    category_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    parent_category_id INT NULL,
    FOREIGN KEY (parent_category_id) REFERENCES categories(category_id)
);

-- 4. جدول المنتجات
CREATE TABLE products (
    product_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    stock_quantity INT DEFAULT 0,
    category_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (category_id) REFERENCES categories(category_id)
);

-- 5. جدول الطلبات
CREATE TABLE orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT,
    address_id INT,
    total_amount DECIMAL(10,2),
    status ENUM('pending', 'processing', 'shipped', 'delivered', 'cancelled'),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (address_id) REFERENCES addresses(address_id)
);

-- 6. جدول تفاصيل الطلبات
CREATE TABLE order_items (
    order_item_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT,
    product_id INT,
    quantity INT NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

-- 7. جدول المراجعات
CREATE TABLE reviews (
    review_id INT PRIMARY KEY AUTO_INCREMENT,
    product_id INT,
    user_id INT,
    rating INT CHECK (rating BETWEEN 1 AND 5),
    comment TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (product_id) REFERENCES products(product_id),
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

-- Indexes للأداء
CREATE INDEX idx_products_category ON products(category_id);
CREATE INDEX idx_orders_user ON orders(user_id);
CREATE INDEX idx_order_items_order ON order_items(order_id);
CREATE INDEX idx_reviews_product ON reviews(product_id);
```

---

## نصائح للمقابلات

### استراتيجية الإجابة:

1. **افهم السؤال**
   - اسأل عن التفاصيل
   - تحقق من المتطلبات

2. **ابدأ بالأساسيات**
   - اشرح المفهوم الأساسي
   - أعط مثال بسيط

3. **أعط أمثلة عملية**
   - اكتب SQL واضح
   - استخدم أمثلة من الحياة الواقعية

4. **ناقش Trade-offs**
   - المزايا والعيوب
   - متى تستخدم كل approach

5. **تكلم عن الأداء**
   - Indexes
   - Query Optimization
   - Caching

---

**حظاً موفقاً في المقابلة! 🚀**
