# Database - قواعد البيانات

## المحتويات
- [SQL](SQL.md)
- [NoSQL](NoSQL.md)
- [SQL vs NoSQL](SQL-vs-NoSQL.md)
- [أسئلة مقابلات Database](Database-Interview-Questions.md)

---

## ما هي قاعدة البيانات؟

قاعدة البيانات (Database) هي مجموعة منظمة من البيانات يمكن الوصول إليها وإدارتها بسهولة.

---

## أنواع قواعد البيانات

### 1. **SQL (Relational Databases)**
قواعد بيانات علائقية تستخدم جداول وعلاقات.

**أمثلة:**
- MySQL
- PostgreSQL
- Oracle
- SQL Server
- SQLite

### 2. **NoSQL (Non-Relational Databases)**
قواعد بيانات غير علائقية لبيانات غير منظمة.

**أنواعها:**
- **Document**: MongoDB, CouchDB
- **Key-Value**: Redis, DynamoDB
- **Column-Family**: Cassandra, HBase
- **Graph**: Neo4j, ArangoDB

---

## متى تستخدم SQL؟

✅ البيانات منظمة ومحددة البنية
✅ تحتاج لـ ACID transactions
✅ علاقات معقدة بين البيانات
✅ استعلامات معقدة (joins, aggregations)

**أمثلة:**
- أنظمة البنوك
- أنظمة المبيعات
- أنظمة إدارة الموظفين

---

## متى تستخدم NoSQL؟

✅ البيانات غير منظمة أو متغيرة البنية
✅ حجم بيانات ضخم (Big Data)
✅ تحتاج لـ scalability أفقي
✅ سرعة القراءة والكتابة مهمة

**أمثلة:**
- وسائل التواصل الاجتماعي
- تطبيقات Real-time
- IoT Applications
- Gaming

---

## الفرق الأساسي

| SQL | NoSQL |
|-----|-------|
| بنية محددة (Schema) | مرن (Schema-less) |
| جداول وعلاقات | Documents, Key-Value, etc. |
| Vertical Scaling | Horizontal Scaling |
| ACID | BASE |
| SQL Language | Various APIs |

---

**للمزيد من التفاصيل، راجع الملفات الأخرى في هذا المجلد**
