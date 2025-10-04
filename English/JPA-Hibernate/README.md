# JPA & Hibernate Documentation

## Table of Contents
- [Introduction](#introduction)
- [JDBC vs JPA vs Hibernate](#jdbc-vs-jpa-vs-hibernate)
- [Architecture Overview](#architecture-overview)
- [Getting Started](#getting-started)
- [Core Concepts](#core-concepts)
- [Resources](#resources)

## Introduction

### What is JDBC?
**JDBC (Java Database Connectivity)** is a Java API that defines how a client may access a database. It provides methods to query and update data in a database and is oriented towards relational databases.

**Key Characteristics:**
- Low-level API for database operations
- Requires manual SQL query writing
- Developer must handle connection management
- Manual mapping between ResultSet and Java objects
- Database-specific code

**Example:**
```java
Connection conn = DriverManager.getConnection(url, username, password);
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery("SELECT * FROM users WHERE id = 1");
while(rs.next()) {
    User user = new User();
    user.setId(rs.getInt("id"));
    user.setName(rs.getString("name"));
    // Manual mapping...
}
```

### What is JPA?
**JPA (Java Persistence API)** is a specification for accessing, persisting, and managing data between Java objects and relational databases. It's part of the Java EE specification.

**Key Characteristics:**
- ORM (Object-Relational Mapping) specification
- Vendor-neutral API
- Annotation-based or XML configuration
- Automatic SQL generation
- Object-oriented query language (JPQL)
- Implementations: Hibernate, EclipseLink, OpenJPA

### What is Hibernate?
**Hibernate** is the most popular implementation of the JPA specification. It's a powerful, high-performance ORM framework for Java.

**Key Characteristics:**
- Full JPA implementation + additional features
- Mature and feature-rich
- Excellent documentation and community support
- Caching mechanisms (1st and 2nd level)
- HQL (Hibernate Query Language) - superset of JPQL
- Criteria API for type-safe queries

## JDBC vs JPA vs Hibernate

| Feature | JDBC | JPA | Hibernate |
|---------|------|-----|-----------|
| Type | API | Specification | Framework/Implementation |
| Level | Low-level | High-level | High-level |
| SQL | Manual | Auto-generated | Auto-generated + HQL |
| Mapping | Manual | Automatic (ORM) | Automatic (ORM) |
| Boilerplate | High | Low | Low |
| Portability | Database-specific | Vendor-neutral | JPA-compliant + extras |
| Learning Curve | Easy | Moderate | Moderate to High |
| Performance | Fast (when optimized) | Good | Excellent (with caching) |
| Caching | Manual | Provider-specific | Built-in (1st & 2nd level) |
| Transaction | Manual | Declarative | Declarative |

## Architecture Overview

### JDBC Architecture
```
Java Application → JDBC API → JDBC Driver → Database
```

### JPA Architecture
```
Java Application
    ↓
JPA API (javax.persistence)
    ↓
JPA Provider (Hibernate/EclipseLink/OpenJPA)
    ↓
JDBC
    ↓
Database
```

### Hibernate Architecture Layers

1. **Application Layer** - Your Java application code
2. **Hibernate API Layer** - Session, Transaction, Query, Criteria
3. **Hibernate Core** - Configuration, SessionFactory
4. **JDBC Layer** - Connection pooling, transaction management
5. **Database Layer** - Actual database

**Key Hibernate Components:**

- **Configuration**: Bootstrap Hibernate
- **SessionFactory**: Thread-safe, expensive to create (one per application)
- **Session**: Not thread-safe, lightweight (one per request/transaction)
- **Transaction**: Unit of work
- **Query/Criteria**: For querying data
- **EntityManager** (JPA): Similar to Session

## Getting Started

### 1. Maven Dependencies

```xml
<!-- JPA API -->
<dependency>
    <groupId>jakarta.persistence</groupId>
    <artifactId>jakarta.persistence-api</artifactId>
    <version>3.1.0</version>
</dependency>

<!-- Hibernate (JPA Implementation) -->
<dependency>
    <groupId>org.hibernate.orm</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>6.2.7.Final</version>
</dependency>

<!-- Database Driver (example: MySQL) -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.33</version>
</dependency>

<!-- Connection Pool (optional but recommended) -->
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>5.0.1</version>
</dependency>
```

### 2. Configuration (persistence.xml)

**Location:** `src/main/resources/META-INF/persistence.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="https://jakarta.ee/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="https://jakarta.ee/xml/ns/persistence
             https://jakarta.ee/xml/ns/persistence/persistence_3_0.xsd"
             version="3.0">

    <persistence-unit name="myPersistenceUnit" transaction-type="RESOURCE_LOCAL">
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>

        <properties>
            <!-- Database Connection -->
            <property name="jakarta.persistence.jdbc.driver" value="com.mysql.cj.jdbc.Driver"/>
            <property name="jakarta.persistence.jdbc.url" value="jdbc:mysql://localhost:3306/mydb"/>
            <property name="jakarta.persistence.jdbc.user" value="root"/>
            <property name="jakarta.persistence.jdbc.password" value="password"/>

            <!-- Hibernate Properties -->
            <property name="hibernate.dialect" value="org.hibernate.dialect.MySQLDialect"/>
            <property name="hibernate.hbm2ddl.auto" value="update"/>
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>

            <!-- Connection Pool -->
            <property name="hibernate.hikari.connectionTimeout" value="20000"/>
            <property name="hibernate.hikari.minimumIdle" value="5"/>
            <property name="hibernate.hikari.maximumPoolSize" value="10"/>

            <!-- Cache -->
            <property name="hibernate.cache.use_second_level_cache" value="true"/>
            <property name="hibernate.cache.region.factory_class"
                      value="org.hibernate.cache.jcache.JCacheRegionFactory"/>
        </properties>
    </persistence-unit>
</persistence>
```

### 3. Alternative: application.properties (Spring Boot)

```properties
# Database
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA/Hibernate
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect

# Connection Pool
spring.datasource.hikari.connection-timeout=20000
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.maximum-pool-size=10

# Cache
spring.jpa.properties.hibernate.cache.use_second_level_cache=true
spring.jpa.properties.hibernate.cache.region.factory_class=org.hibernate.cache.jcache.JCacheRegionFactory
```

### 4. Basic Entity Example

```java
import jakarta.persistence.*;
import java.time.LocalDateTime;

@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "username", unique = true, nullable = false, length = 50)
    private String username;

    @Column(name = "email", unique = true, nullable = false)
    private String email;

    @Column(name = "created_at", updatable = false)
    private LocalDateTime createdAt;

    @Column(name = "updated_at")
    private LocalDateTime updatedAt;

    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }

    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }

    // Constructors, Getters, Setters
    public User() {}

    public User(String username, String email) {
        this.username = username;
        this.email = email;
    }

    // Getters and Setters...
}
```

### 5. Basic CRUD Operations

```java
import jakarta.persistence.*;

public class UserRepository {

    private EntityManagerFactory emf = Persistence.createEntityManagerFactory("myPersistenceUnit");

    // Create
    public void saveUser(User user) {
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();
        try {
            tx.begin();
            em.persist(user);
            tx.commit();
        } catch (Exception e) {
            if (tx.isActive()) tx.rollback();
            throw e;
        } finally {
            em.close();
        }
    }

    // Read
    public User findUserById(Long id) {
        EntityManager em = emf.createEntityManager();
        try {
            return em.find(User.class, id);
        } finally {
            em.close();
        }
    }

    // Update
    public void updateUser(User user) {
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();
        try {
            tx.begin();
            em.merge(user);
            tx.commit();
        } catch (Exception e) {
            if (tx.isActive()) tx.rollback();
            throw e;
        } finally {
            em.close();
        }
    }

    // Delete
    public void deleteUser(Long id) {
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();
        try {
            tx.begin();
            User user = em.find(User.class, id);
            if (user != null) {
                em.remove(user);
            }
            tx.commit();
        } catch (Exception e) {
            if (tx.isActive()) tx.rollback();
            throw e;
        } finally {
            em.close();
        }
    }

    // Query
    public List<User> findAllUsers() {
        EntityManager em = emf.createEntityManager();
        try {
            return em.createQuery("SELECT u FROM User u", User.class)
                    .getResultList();
        } finally {
            em.close();
        }
    }

    public void close() {
        if (emf != null && emf.isOpen()) {
            emf.close();
        }
    }
}
```

## Core Concepts

### 1. Entity Lifecycle States

- **Transient**: New object, not associated with persistence context
- **Persistent/Managed**: Associated with persistence context, changes tracked
- **Detached**: Was managed, but persistence context closed
- **Removed**: Marked for deletion

### 2. Primary Key Generation Strategies

```java
// AUTO - Provider chooses strategy
@GeneratedValue(strategy = GenerationType.AUTO)

// IDENTITY - Database auto-increment
@GeneratedValue(strategy = GenerationType.IDENTITY)

// SEQUENCE - Database sequence
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "user_seq")
@SequenceGenerator(name = "user_seq", sequenceName = "user_sequence", allocationSize = 1)

// TABLE - Separate table for ID generation
@GeneratedValue(strategy = GenerationType.TABLE, generator = "user_gen")
@TableGenerator(name = "user_gen", table = "id_generator",
                pkColumnName = "gen_name", valueColumnName = "gen_value")

// UUID
@GeneratedValue(strategy = GenerationType.UUID)
```

### 3. Relationships

- **@OneToOne**: One entity instance relates to one instance of another
- **@OneToMany / @ManyToOne**: One entity relates to many instances
- **@ManyToMany**: Many instances relate to many instances

### 4. Fetch Types

- **EAGER**: Load related entities immediately
- **LAZY**: Load related entities on-demand (default for collections)

### 5. Cascade Types

- **PERSIST**: Save operations cascade to related entities
- **MERGE**: Update operations cascade
- **REMOVE**: Delete operations cascade
- **REFRESH**: Reload operations cascade
- **DETACH**: Detach operations cascade
- **ALL**: All operations cascade

## Resources

### Official Documentation
- [JPA Specification](https://jakarta.ee/specifications/persistence/)
- [Hibernate Documentation](https://hibernate.org/orm/documentation/)
- [Spring Data JPA Reference](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/)

### Books
- "Java Persistence with Hibernate" by Christian Bauer & Gavin King
- "Pro JPA 2" by Mike Keith & Merrick Schincariol
- "High-Performance Java Persistence" by Vlad Mihalcea

### Online Resources
- [Baeldung JPA/Hibernate Tutorials](https://www.baeldung.com/learn-jpa-hibernate)
- [Hibernate Community](https://hibernate.org/community/)
- [Vlad Mihalcea's Blog](https://vladmihalcea.com/)

## Best Practices

1. **Always use connection pooling** (HikariCP recommended)
2. **Use LAZY fetching** by default for collections
3. **Avoid N+1 queries** - use JOIN FETCH or entity graphs
4. **Enable 2nd level cache** for read-heavy data
5. **Use pagination** for large result sets
6. **Use DTOs** for read-only queries
7. **Monitor SQL queries** in development (show_sql=true)
8. **Use appropriate cascade types** - be careful with REMOVE
9. **Index foreign keys** and frequently queried columns
10. **Use batch processing** for bulk operations

---

*For detailed interview questions and advanced topics, see [JPA-Hibernate-Interview-Questions.md](./JPA-Hibernate-Interview-Questions.md)*
