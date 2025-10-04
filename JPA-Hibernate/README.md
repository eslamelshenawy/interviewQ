# JPA, Hibernate و JDBC - دليل شامل

## المحتويات
- [نظرة عامة](#نظرة-عامة)
- [JDBC](#jdbc)
- [JPA](#jpa)
- [Hibernate](#hibernate)
- [المقارنة بين JDBC و JPA و Hibernate](#المقارنة-بين-jdbc-و-jpa-و-hibernate)
- [البنية الأساسية](#البنية-الأساسية)
- [الإعداد والتكوين](#الإعداد-والتكوين)

---

## نظرة عامة

### ما هو JDBC؟
**JDBC (Java Database Connectivity)** هو API قياسي في Java يوفر واجهة للاتصال وتنفيذ الاستعلامات على قواعد البيانات العلائقية.

#### الميزات:
- واجهة منخفضة المستوى للتعامل مع قاعدة البيانات
- يتطلب كتابة استعلامات SQL يدوياً
- إدارة الاتصالات والموارد يدوياً
- لا يوجد تعيين تلقائي بين الكائنات والجداول

#### مثال JDBC:
```java
// تحميل درايفر قاعدة البيانات
Class.forName("com.mysql.cj.jdbc.Driver");

// إنشاء الاتصال
Connection conn = DriverManager.getConnection(
    "jdbc:mysql://localhost:3306/mydb",
    "username",
    "password"
);

// إنشاء Statement
Statement stmt = conn.createStatement();

// تنفيذ الاستعلام
ResultSet rs = stmt.executeQuery("SELECT * FROM users");

// معالجة النتائج
while(rs.next()) {
    int id = rs.getInt("id");
    String name = rs.getString("name");
    System.out.println("ID: " + id + ", Name: " + name);
}

// إغلاق الموارد
rs.close();
stmt.close();
conn.close();
```

### مشاكل JDBC:
1. **كود متكرر (Boilerplate Code)**: الكثير من الكود لإدارة الاتصالات والموارد
2. **كتابة SQL يدوياً**: عرضة للأخطاء
3. **عدم وجود ORM**: لا يوجد تعيين تلقائي بين الكائنات والجداول
4. **إدارة الاستثناءات معقدة**: التعامل مع SQLException
5. **صعوبة الصيانة**: تغيير هيكل قاعدة البيانات يتطلب تعديلات كثيرة

---

## JPA

### ما هو JPA؟
**JPA (Java Persistence API)** هو مواصفات قياسية (Specification) في Java توفر نموذج Object-Relational Mapping (ORM) لتعيين الكائنات الجافا إلى جداول قاعدة البيانات.

#### الميزات:
- مواصفات قياسية، ليست تطبيقاً (Implementation)
- توفر Annotations للتعيين بين الكائنات والجداول
- تدعم JPQL (Java Persistence Query Language)
- إدارة دورة حياة الكائنات (Entity Lifecycle)
- دعم التعامل مع العلاقات بين الجداول

#### مكونات JPA الرئيسية:
1. **Entity**: كلاس Java يمثل جدول في قاعدة البيانات
2. **EntityManager**: واجهة لإدارة العمليات على الـ Entities
3. **Persistence Context**: مساحة تخزين مؤقت لإدارة حالة الـ Entities
4. **EntityManagerFactory**: مصنع لإنشاء EntityManager
5. **Persistence Unit**: تكوين في ملف persistence.xml

#### مثال JPA Entity:
```java
import javax.persistence.*;

@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "username", nullable = false, unique = true)
    private String username;

    @Column(name = "email")
    private String email;

    @Temporal(TemporalType.TIMESTAMP)
    @Column(name = "created_at")
    private Date createdAt;

    // Constructors, Getters, Setters
    public User() {}

    public User(String username, String email) {
        this.username = username;
        this.email = email;
        this.createdAt = new Date();
    }

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }

    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }

    public Date getCreatedAt() { return createdAt; }
    public void setCreatedAt(Date createdAt) { this.createdAt = createdAt; }
}
```

#### استخدام EntityManager:
```java
// إنشاء EntityManagerFactory
EntityManagerFactory emf = Persistence.createEntityManagerFactory("my-persistence-unit");

// إنشاء EntityManager
EntityManager em = emf.createEntityManager();

// بدء Transaction
em.getTransaction().begin();

// إنشاء كائن جديد
User user = new User("ahmed", "ahmed@example.com");
em.persist(user); // حفظ في قاعدة البيانات

// تنفيذ Transaction
em.getTransaction().commit();

// البحث عن كائن
User foundUser = em.find(User.class, 1L);

// تحديث كائن
em.getTransaction().begin();
foundUser.setEmail("new-email@example.com");
em.getTransaction().commit();

// حذف كائن
em.getTransaction().begin();
em.remove(foundUser);
em.getTransaction().commit();

// إغلاق الموارد
em.close();
emf.close();
```

---

## Hibernate

### ما هو Hibernate؟
**Hibernate** هو إطار عمل (Framework) يعمل كتطبيق (Implementation) لمواصفات JPA، ويوفر ميزات إضافية تتجاوز المواصفات القياسية.

#### الميزات:
- تطبيق كامل لمواصفات JPA
- يوفر ميزات إضافية (HQL, Criteria API, Second Level Cache)
- دعم أفضل للأداء والتحسينات
- Session بدلاً من EntityManager (مع دعم JPA أيضاً)
- SessionFactory بدلاً من EntityManagerFactory

#### Hibernate vs JPA:
| JPA | Hibernate |
|-----|-----------|
| مواصفات قياسية | تطبيق للمواصفات |
| EntityManager | Session (و EntityManager) |
| EntityManagerFactory | SessionFactory (و EntityManagerFactory) |
| JPQL | HQL و JPQL |
| Annotations محدودة | Annotations إضافية |

#### مثال Hibernate Configuration:
```java
// hibernate.cfg.xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
    "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
    <session-factory>
        <!-- Database connection settings -->
        <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/mydb</property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">password</property>

        <!-- JDBC connection pool settings -->
        <property name="hibernate.c3p0.min_size">5</property>
        <property name="hibernate.c3p0.max_size">20</property>
        <property name="hibernate.c3p0.timeout">300</property>

        <!-- SQL dialect -->
        <property name="hibernate.dialect">org.hibernate.dialect.MySQL8Dialect</property>

        <!-- Echo SQL to stdout -->
        <property name="hibernate.show_sql">true</property>
        <property name="hibernate.format_sql">true</property>

        <!-- Drop and re-create the database schema on startup -->
        <property name="hibernate.hbm2ddl.auto">update</property>

        <!-- Enable Second Level Cache -->
        <property name="hibernate.cache.use_second_level_cache">true</property>
        <property name="hibernate.cache.region.factory_class">
            org.hibernate.cache.ehcache.EhCacheRegionFactory
        </property>

        <!-- Mapping classes -->
        <mapping class="com.example.model.User"/>
    </session-factory>
</hibernate-configuration>
```

#### استخدام Hibernate Session:
```java
// إنشاء SessionFactory
Configuration configuration = new Configuration();
configuration.configure("hibernate.cfg.xml");
SessionFactory sessionFactory = configuration.buildSessionFactory();

// إنشاء Session
Session session = sessionFactory.openSession();

// بدء Transaction
Transaction transaction = session.beginTransaction();

// حفظ كائن
User user = new User("ahmed", "ahmed@example.com");
session.save(user);

// تنفيذ Transaction
transaction.commit();

// البحث باستخدام HQL
List<User> users = session.createQuery("FROM User WHERE username LIKE :name", User.class)
                          .setParameter("name", "ahmed%")
                          .list();

// إغلاق Session
session.close();
sessionFactory.close();
```

---

## المقارنة بين JDBC و JPA و Hibernate

| الخاصية | JDBC | JPA | Hibernate |
|---------|------|-----|-----------|
| **النوع** | API قياسي | مواصفات قياسية | Framework (تطبيق JPA) |
| **المستوى** | منخفض المستوى | عالي المستوى | عالي المستوى |
| **ORM** | لا يوجد | نعم | نعم |
| **كتابة SQL** | يدوياً | اختياري (JPQL) | اختياري (HQL/JPQL) |
| **Mapping** | يدوي | تلقائي (Annotations) | تلقائي (Annotations) |
| **الأداء** | سريع (لكن يحتاج كود كثير) | جيد | ممتاز (مع optimizations) |
| **Caching** | يدوي | First Level | First & Second Level |
| **إدارة الاتصالات** | يدوية | تلقائية | تلقائية (مع Connection Pool) |
| **دورة الحياة** | لا يوجد | نعم | نعم |
| **العلاقات** | يدوية | تلقائية | تلقائية |
| **Lazy Loading** | لا يوجد | نعم | نعم |
| **سهولة الاستخدام** | صعب | سهل | سهل |
| **البائع** | لا يعتمد | لا يعتمد | يعتمد (لكن يدعم JPA) |

---

## البنية الأساسية

### معمارية Hibernate/JPA:

```
┌─────────────────────────────────────────┐
│        Application Layer                │
│    (Business Logic & Services)          │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────▼───────────────────────┐
│         JPA Interface                   │
│  (EntityManager, Entity, Query)         │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────▼───────────────────────┐
│      Hibernate Implementation           │
│  (Session, SessionFactory, etc.)        │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────▼───────────────────────┐
│       Persistence Context               │
│   (First Level Cache - Session)         │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────▼───────────────────────┐
│      Second Level Cache                 │
│   (Optional - SessionFactory Level)     │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────▼───────────────────────┐
│         JDBC Layer                      │
│   (Connection Pool, Drivers)            │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────▼───────────────────────┐
│         Database                        │
│   (MySQL, PostgreSQL, Oracle, etc.)     │
└─────────────────────────────────────────┘
```

---

## الإعداد والتكوين

### 1. إضافة Dependencies (Maven):

```xml
<dependencies>
    <!-- Hibernate Core -->
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-core</artifactId>
        <version>5.6.15.Final</version>
    </dependency>

    <!-- JPA API -->
    <dependency>
        <groupId>javax.persistence</groupId>
        <artifactId>javax.persistence-api</artifactId>
        <version>2.2</version>
    </dependency>

    <!-- MySQL Driver -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.33</version>
    </dependency>

    <!-- Connection Pool (HikariCP) -->
    <dependency>
        <groupId>com.zaxxer</groupId>
        <artifactId>HikariCP</artifactId>
        <version>5.0.1</version>
    </dependency>

    <!-- Second Level Cache (EhCache) -->
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-ehcache</artifactId>
        <version>5.6.15.Final</version>
    </dependency>
</dependencies>
```

### 2. ملف persistence.xml (JPA):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence
             http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd"
             version="2.2">

    <persistence-unit name="my-persistence-unit" transaction-type="RESOURCE_LOCAL">
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>

        <class>com.example.model.User</class>
        <class>com.example.model.Product</class>

        <properties>
            <!-- Database Connection -->
            <property name="javax.persistence.jdbc.driver" value="com.mysql.cj.jdbc.Driver"/>
            <property name="javax.persistence.jdbc.url" value="jdbc:mysql://localhost:3306/mydb"/>
            <property name="javax.persistence.jdbc.user" value="root"/>
            <property name="javax.persistence.jdbc.password" value="password"/>

            <!-- Hibernate Properties -->
            <property name="hibernate.dialect" value="org.hibernate.dialect.MySQL8Dialect"/>
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.hbm2ddl.auto" value="update"/>

            <!-- Connection Pool -->
            <property name="hibernate.hikari.minimumIdle" value="5"/>
            <property name="hibernate.hikari.maximumPoolSize" value="20"/>
            <property name="hibernate.hikari.idleTimeout" value="300000"/>
        </properties>
    </persistence-unit>
</persistence>
```

### 3. Hibernate Configuration (Programmatic):

```java
public class HibernateUtil {

    private static SessionFactory sessionFactory;

    static {
        try {
            Configuration configuration = new Configuration();

            // Database settings
            configuration.setProperty("hibernate.connection.driver_class", "com.mysql.cj.jdbc.Driver");
            configuration.setProperty("hibernate.connection.url", "jdbc:mysql://localhost:3306/mydb");
            configuration.setProperty("hibernate.connection.username", "root");
            configuration.setProperty("hibernate.connection.password", "password");

            // Hibernate settings
            configuration.setProperty("hibernate.dialect", "org.hibernate.dialect.MySQL8Dialect");
            configuration.setProperty("hibernate.show_sql", "true");
            configuration.setProperty("hibernate.format_sql", "true");
            configuration.setProperty("hibernate.hbm2ddl.auto", "update");

            // Connection pool
            configuration.setProperty("hibernate.c3p0.min_size", "5");
            configuration.setProperty("hibernate.c3p0.max_size", "20");

            // Add annotated classes
            configuration.addAnnotatedClass(User.class);
            configuration.addAnnotatedClass(Product.class);

            // Build SessionFactory
            sessionFactory = configuration.buildSessionFactory();

        } catch (Exception e) {
            throw new ExceptionInInitializerError(e);
        }
    }

    public static SessionFactory getSessionFactory() {
        return sessionFactory;
    }

    public static void shutdown() {
        getSessionFactory().close();
    }
}
```

---

## متى نستخدم كل منهم؟

### استخدم JDBC عندما:
- تحتاج أقصى أداء وتحكم دقيق
- استعلامات بسيطة وقليلة
- مشروع صغير جداً
- تحتاج تنفيذ Batch Operations كبيرة

### استخدم JPA عندما:
- تريد كود قياسي وقابل للنقل بين Implementations مختلفة
- تحتاج ORM مع دعم للعلاقات
- تريد المرونة في تغيير الـ Provider (Hibernate, EclipseLink, etc.)

### استخدم Hibernate عندما:
- تحتاج ميزات متقدمة (HQL, Criteria API, Caching)
- مشروع كبير ومعقد
- تحتاج أداء محسّن وإدارة ذاكرة متقدمة
- تريد استخدام Second Level Cache

---

## الخلاصة

- **JDBC**: منخفض المستوى، مزيد من التحكم، مزيد من الكود
- **JPA**: مواصفات قياسية للـ ORM، واجهة موحدة
- **Hibernate**: تطبيق قوي لـ JPA مع ميزات إضافية

معظم المشاريع الحديثة تستخدم **Hibernate** كـ JPA Provider للحصول على أفضل توازن بين الأداء والإنتاجية.
