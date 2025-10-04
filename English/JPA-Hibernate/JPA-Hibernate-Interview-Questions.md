# JPA & Hibernate Interview Questions

## Table of Contents
1. [JPA vs Hibernate vs JDBC](#1-jpa-vs-hibernate-vs-jdbc)
2. [Entity Lifecycle](#2-entity-lifecycle)
3. [Relationships](#3-relationships)
4. [Fetch Types](#4-fetch-types)
5. [Cascade Types](#5-cascade-types)
6. [Inheritance Strategies](#6-inheritance-strategies)
7. [N+1 Problem](#7-n1-problem)
8. [Caching](#8-caching)
9. [JPQL vs Native SQL](#9-jpql-vs-native-sql)
10. [Criteria API](#10-criteria-api)
11. [Transaction Management](#11-transaction-management)
12. [Performance Optimization](#12-performance-optimization)

---

## 1. JPA vs Hibernate vs JDBC

### Q1: What is the difference between JDBC, JPA, and Hibernate?

**JDBC (Java Database Connectivity)**
- Low-level API for database operations
- Requires manual SQL writing and object mapping
- Database-specific code
- More boilerplate code

```java
// JDBC Example
Connection conn = DriverManager.getConnection(url, user, password);
PreparedStatement pstmt = conn.prepareStatement("INSERT INTO users (name, email) VALUES (?, ?)");
pstmt.setString(1, "John");
pstmt.setString(2, "john@example.com");
pstmt.executeUpdate();
```

**JPA (Java Persistence API)**
- High-level ORM specification
- Vendor-neutral (can switch implementations)
- Annotation-based mapping
- Automatic SQL generation

```java
// JPA Example
EntityManager em = emf.createEntityManager();
em.getTransaction().begin();
User user = new User("John", "john@example.com");
em.persist(user);
em.getTransaction().commit();
```

**Hibernate**
- Most popular JPA implementation
- Provides additional features beyond JPA
- HQL (Hibernate Query Language)
- Advanced caching mechanisms

```java
// Hibernate-specific features
Session session = sessionFactory.openSession();
Transaction tx = session.beginTransaction();
session.createQuery("FROM User WHERE age > :age")
       .setParameter("age", 18)
       .setCacheable(true)  // Hibernate-specific
       .list();
tx.commit();
```

### Q2: Can you use Hibernate without JPA?

**Answer:** Yes, Hibernate has its own native API that predates JPA. However, it's recommended to use JPA annotations for better portability.

```java
// Native Hibernate API
Session session = sessionFactory.openSession();
Transaction tx = session.beginTransaction();
session.save(user);
tx.commit();

// JPA API (recommended)
EntityManager em = emf.createEntityManager();
em.getTransaction().begin();
em.persist(user);
em.getTransaction().commit();
```

### Q3: What are the advantages of using JPA/Hibernate over JDBC?

**Advantages:**
1. **Less Boilerplate**: No manual SQL for basic CRUD
2. **Object-Oriented**: Work with objects, not ResultSets
3. **Database Independence**: Switch databases easily
4. **Caching**: Built-in 1st and 2nd level cache
5. **Lazy Loading**: Load data on demand
6. **Automatic Dirty Checking**: Auto-update changed entities
7. **Transaction Management**: Simplified transaction handling
8. **Type-Safe Queries**: Criteria API prevents SQL injection

---

## 2. Entity Lifecycle

### Q4: Explain the Entity Lifecycle states in JPA.

**The 4 Entity States:**

1. **Transient (New)**
   - Object created but not yet associated with EntityManager
   - Not persisted in database
   - No identifier assigned

2. **Persistent (Managed)**
   - Associated with persistence context
   - Changes are automatically synchronized with database
   - Has database identifier

3. **Detached**
   - Was persistent but persistence context closed
   - Changes not tracked
   - Can be reattached using merge()

4. **Removed**
   - Scheduled for deletion
   - Still in persistence context until transaction commits

**Diagram:**
```
    new()
      ↓
[TRANSIENT] ---persist()---> [PERSISTENT] ---commit()--> Database
                                  ↓                           ↑
                              detach()/close()           find()/merge()
                                  ↓                           ↓
                            [DETACHED] ----------merge()------+

[PERSISTENT] ---remove()---> [REMOVED] ---commit()--> Deleted from DB
```

**Example:**

```java
import jakarta.persistence.*;

public class EntityLifecycleDemo {

    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("myPU");
        EntityManager em = emf.createEntityManager();

        // 1. TRANSIENT - new object, not associated with EM
        User user = new User("John", "john@example.com");
        System.out.println("Transient: " + em.contains(user)); // false

        em.getTransaction().begin();

        // 2. PERSISTENT - now managed by EM
        em.persist(user);
        System.out.println("Persistent: " + em.contains(user)); // true

        // Changes are tracked automatically
        user.setEmail("newemail@example.com"); // Will be updated in DB

        em.getTransaction().commit();

        // 3. DETACHED - EM closed, entity no longer managed
        em.close();
        System.out.println("Detached: " + em.contains(user)); // throws exception

        // Reopen EM
        em = emf.createEntityManager();
        em.getTransaction().begin();

        // Changes to detached entity not tracked
        user.setEmail("another@example.com");

        // 4. MERGE - reattach detached entity
        User managedUser = em.merge(user);
        System.out.println("After merge: " + em.contains(managedUser)); // true

        // 5. REMOVED - mark for deletion
        em.remove(managedUser);
        System.out.println("Removed: " + em.contains(managedUser)); // true (until commit)

        em.getTransaction().commit();
        em.close();
        emf.close();
    }
}
```

### Q5: What is the difference between persist() and merge()?

**persist()**
- For new (transient) entities
- Makes entity managed
- Throws exception if entity already exists
- Cascades to related entities

**merge()**
- For detached entities
- Creates a new managed copy
- Returns the managed entity
- Original entity remains detached

```java
// persist() - for new entities
User newUser = new User("Alice", "alice@example.com");
em.persist(newUser);  // newUser becomes managed

// merge() - for detached entities
em.close();  // user is now detached
em = emf.createEntityManager();
em.getTransaction().begin();

user.setEmail("updated@example.com");
User managedUser = em.merge(user);  // returns managed copy
// user is still detached, managedUser is managed

System.out.println(em.contains(user));        // false
System.out.println(em.contains(managedUser)); // true
```

### Q6: What happens if you call persist() on a detached entity?

**Answer:** It throws `EntityExistsException` because the entity already has an ID. Use `merge()` instead.

```java
// Wrong - throws EntityExistsException
User detachedUser = em.find(User.class, 1L);
em.close();
em = emf.createEntityManager();
em.getTransaction().begin();
em.persist(detachedUser);  // ERROR!

// Correct - use merge()
User managedUser = em.merge(detachedUser);  // OK
```

---

## 3. Relationships

### Q7: Explain different types of relationships in JPA with examples.

### 3.1 @OneToOne Relationship

**Scenario:** User has one Profile, Profile belongs to one User

```java
// Bidirectional @OneToOne with foreign key in Profile table

@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;

    // mappedBy indicates the inverse side (non-owning)
    @OneToOne(mappedBy = "user", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private Profile profile;

    // Helper method to maintain bidirectional consistency
    public void setProfile(Profile profile) {
        this.profile = profile;
        if (profile != null) {
            profile.setUser(this);
        }
    }

    // Constructors, getters, setters...
}

@Entity
@Table(name = "profiles")
public class Profile {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String bio;
    private String avatar;

    // Owning side - has the foreign key
    @OneToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", unique = true, nullable = false)
    private User user;

    // Constructors, getters, setters...
}

// Usage
User user = new User("john_doe");
Profile profile = new Profile("Software Developer", "avatar.jpg");
user.setProfile(profile);
em.persist(user);  // Profile also saved due to CASCADE
```

**Unidirectional @OneToOne:**
```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "profile_id", unique = true)
    private Profile profile;
}

@Entity
public class Profile {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    // No reference to User
}
```

### 3.2 @OneToMany / @ManyToOne Relationship

**Scenario:** Department has many Employees, Employee belongs to one Department

```java
// Bidirectional @OneToMany / @ManyToOne

@Entity
@Table(name = "departments")
public class Department {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    // One department has many employees
    // mappedBy = "department" refers to the field in Employee class
    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Employee> employees = new ArrayList<>();

    // Helper methods to maintain bidirectional relationship
    public void addEmployee(Employee employee) {
        employees.add(employee);
        employee.setDepartment(this);
    }

    public void removeEmployee(Employee employee) {
        employees.remove(employee);
        employee.setDepartment(null);
    }

    // Constructors, getters, setters...
}

@Entity
@Table(name = "employees")
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;

    // Many employees belong to one department
    // Owning side - has the foreign key
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "department_id")
    private Department department;

    // Constructors, getters, setters...
}

// Usage
Department dept = new Department("IT");
Employee emp1 = new Employee("Alice", "alice@company.com");
Employee emp2 = new Employee("Bob", "bob@company.com");

dept.addEmployee(emp1);
dept.addEmployee(emp2);

em.persist(dept);  // All employees also saved
```

**Unidirectional @OneToMany:**
```java
@Entity
public class Department {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // Unidirectional - creates join table by default
    @OneToMany(cascade = CascadeType.ALL)
    @JoinColumn(name = "department_id")  // Avoid join table
    private List<Employee> employees = new ArrayList<>();
}

@Entity
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    // No reference to Department
}
```

### 3.3 @ManyToMany Relationship

**Scenario:** Students enroll in many Courses, Courses have many Students

```java
// Bidirectional @ManyToMany

@Entity
@Table(name = "students")
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;

    // Owning side - defines the join table
    @ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses = new HashSet<>();

    // Helper methods
    public void enrollCourse(Course course) {
        courses.add(course);
        course.getStudents().add(this);
    }

    public void dropCourse(Course course) {
        courses.remove(course);
        course.getStudents().remove(this);
    }

    // Override equals/hashCode using business key (email)
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Student)) return false;
        Student student = (Student) o;
        return email != null && email.equals(student.email);
    }

    @Override
    public int hashCode() {
        return getClass().hashCode();
    }

    // Constructors, getters, setters...
}

@Entity
@Table(name = "courses")
public class Course {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String code;

    // Inverse side
    @ManyToMany(mappedBy = "courses")
    private Set<Student> students = new HashSet<>();

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Course)) return false;
        Course course = (Course) o;
        return code != null && code.equals(course.code);
    }

    @Override
    public int hashCode() {
        return getClass().hashCode();
    }

    // Constructors, getters, setters...
}

// Usage
Student alice = new Student("Alice", "alice@university.edu");
Student bob = new Student("Bob", "bob@university.edu");

Course java = new Course("Java Programming", "CS101");
Course database = new Course("Database Systems", "CS201");

alice.enrollCourse(java);
alice.enrollCourse(database);
bob.enrollCourse(java);

em.persist(alice);
em.persist(bob);
em.persist(java);
em.persist(database);
```

**ManyToMany with Extra Columns (Join Entity Pattern):**

```java
// When join table needs additional attributes

@Entity
@Table(name = "students")
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "student", cascade = CascadeType.ALL, orphanRemoval = true)
    private Set<Enrollment> enrollments = new HashSet<>();

    public void enroll(Course course, LocalDate enrollmentDate, String grade) {
        Enrollment enrollment = new Enrollment(this, course, enrollmentDate);
        enrollment.setGrade(grade);
        enrollments.add(enrollment);
        course.getEnrollments().add(enrollment);
    }
}

@Entity
@Table(name = "courses")
public class Course {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "course", cascade = CascadeType.ALL, orphanRemoval = true)
    private Set<Enrollment> enrollments = new HashSet<>();
}

@Entity
@Table(name = "enrollments")
public class Enrollment {
    @EmbeddedId
    private EnrollmentId id;

    @ManyToOne(fetch = FetchType.LAZY)
    @MapsId("studentId")
    @JoinColumn(name = "student_id")
    private Student student;

    @ManyToOne(fetch = FetchType.LAZY)
    @MapsId("courseId")
    @JoinColumn(name = "course_id")
    private Course course;

    private LocalDate enrollmentDate;
    private String grade;

    public Enrollment() {}

    public Enrollment(Student student, Course course, LocalDate enrollmentDate) {
        this.student = student;
        this.course = course;
        this.enrollmentDate = enrollmentDate;
        this.id = new EnrollmentId(student.getId(), course.getId());
    }

    // Getters, setters...
}

@Embeddable
public class EnrollmentId implements Serializable {
    private Long studentId;
    private Long courseId;

    // Constructors, equals, hashCode...
}
```

### Q8: What is the difference between bidirectional and unidirectional relationships?

**Unidirectional:**
- Navigation in one direction only
- One entity knows about the relationship
- Simpler but less flexible

**Bidirectional:**
- Navigation in both directions
- Both entities know about the relationship
- More convenient but requires synchronization

```java
// Unidirectional
@Entity
public class Order {
    @ManyToOne
    @JoinColumn(name = "customer_id")
    private Customer customer;
}

@Entity
public class Customer {
    // No reference to orders
}

// Bidirectional
@Entity
public class Order {
    @ManyToOne
    @JoinColumn(name = "customer_id")
    private Customer customer;
}

@Entity
public class Customer {
    @OneToMany(mappedBy = "customer")
    private List<Order> orders = new ArrayList<>();
}
```

### Q9: What is the owning side vs inverse side in bidirectional relationships?

**Owning Side:**
- Contains the foreign key
- Changes on this side are persisted to database
- Defined by `@JoinColumn`

**Inverse Side:**
- Uses `mappedBy` attribute
- Changes on this side are ignored
- Just a mirror of the owning side

**Rule:** The side with `mappedBy` is the inverse side.

```java
// Department is inverse side (has mappedBy)
@OneToMany(mappedBy = "department")
private List<Employee> employees;

// Employee is owning side (has @JoinColumn)
@ManyToOne
@JoinColumn(name = "department_id")
private Department department;
```

**Important:** Always update the owning side for changes to persist!

```java
// WRONG - only updating inverse side
department.getEmployees().add(employee);
em.persist(department);  // Foreign key NOT set!

// CORRECT - update owning side
employee.setDepartment(department);
em.persist(employee);

// BEST - update both sides for consistency
department.addEmployee(employee);  // helper method updates both
```

---

## 4. Fetch Types

### Q10: Explain EAGER vs LAZY fetching.

**EAGER Loading:**
- Related entities loaded immediately with parent
- Uses JOIN or separate SELECT
- May load unnecessary data
- Can cause performance issues

**LAZY Loading:**
- Related entities loaded on first access
- Uses proxy objects
- Better performance initially
- May cause LazyInitializationException

**Default Fetch Types:**
- `@OneToOne`: EAGER
- `@ManyToOne`: EAGER
- `@OneToMany`: LAZY
- `@ManyToMany`: LAZY

```java
@Entity
public class User {
    @Id
    private Long id;

    // EAGER - loaded immediately
    @OneToOne(fetch = FetchType.EAGER)
    private Profile profile;

    // LAZY - loaded on demand
    @OneToMany(fetch = FetchType.LAZY, mappedBy = "user")
    private List<Order> orders;
}

// Usage
User user = em.find(User.class, 1L);
// Profile already loaded (EAGER)
System.out.println(user.getProfile().getBio());  // No additional query

// Orders not loaded yet (LAZY)
em.close();
System.out.println(user.getOrders().size());  // LazyInitializationException!
```

### Q11: How to solve LazyInitializationException?

**Solutions:**

**1. Fetch within active session:**
```java
EntityManager em = emf.createEntityManager();
User user = em.find(User.class, 1L);
user.getOrders().size();  // Initialize lazy collection
em.close();
```

**2. Use JOIN FETCH in JPQL:**
```java
TypedQuery<User> query = em.createQuery(
    "SELECT u FROM User u JOIN FETCH u.orders WHERE u.id = :id",
    User.class
);
query.setParameter("id", 1L);
User user = query.getSingleResult();
em.close();
user.getOrders().size();  // OK - already loaded
```

**3. Use Entity Graph:**
```java
EntityGraph<User> graph = em.createEntityGraph(User.class);
graph.addAttributeNodes("orders");

Map<String, Object> hints = new HashMap<>();
hints.put("jakarta.persistence.fetchgraph", graph);

User user = em.find(User.class, 1L, hints);
```

**4. Enable Hibernate.initialize():**
```java
User user = em.find(User.class, 1L);
Hibernate.initialize(user.getOrders());
em.close();
user.getOrders().size();  // OK
```

**5. Open Session in View (Spring) - NOT RECOMMENDED:**
```properties
spring.jpa.open-in-view=true  # Anti-pattern!
```

### Q12: What is N+1 SELECT problem related to fetching?

**N+1 Problem:** Executing 1 query to get N records, then N additional queries to fetch related entities (total N+1 queries).

**Example:**
```java
// 1 query to get all users
List<User> users = em.createQuery("SELECT u FROM User u", User.class)
                     .getResultList();

// N queries - one for each user's orders!
for (User user : users) {
    System.out.println(user.getOrders().size());  // Lazy loading triggers query
}
// Total: 1 + N queries = N+1 problem!
```

**See Section 7 for detailed solutions.**

---

## 5. Cascade Types

### Q13: Explain different Cascade Types in JPA.

**CascadeType** determines which operations should propagate from parent to child entities.

**Types:**

1. **PERSIST**
   - Parent saved → child saved
   - New entities in relationship persisted together

2. **MERGE**
   - Parent merged → child merged
   - Detached entities updated together

3. **REMOVE**
   - Parent deleted → child deleted
   - Dangerous with shared entities!

4. **REFRESH**
   - Parent refreshed from DB → child refreshed
   - Reloads state from database

5. **DETACH**
   - Parent detached → child detached
   - Removes from persistence context

6. **ALL**
   - All operations cascade
   - Equivalent to {PERSIST, MERGE, REMOVE, REFRESH, DETACH}

**Examples:**

```java
// Example 1: CASCADE PERSIST
@Entity
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToMany(cascade = CascadeType.PERSIST, mappedBy = "order")
    private List<OrderItem> items = new ArrayList<>();

    public void addItem(OrderItem item) {
        items.add(item);
        item.setOrder(this);
    }
}

// Usage
Order order = new Order();
order.addItem(new OrderItem("Product 1", 100));
order.addItem(new OrderItem("Product 2", 200));

em.persist(order);  // OrderItems automatically persisted!
```

```java
// Example 2: CASCADE REMOVE with orphanRemoval
@Entity
public class Post {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToMany(
        mappedBy = "post",
        cascade = CascadeType.ALL,
        orphanRemoval = true  // Delete orphaned comments
    )
    private List<Comment> comments = new ArrayList<>();

    public void removeComment(Comment comment) {
        comments.remove(comment);
        comment.setPost(null);
    }
}

// Usage
Post post = em.find(Post.class, 1L);
Comment comment = post.getComments().get(0);
post.removeComment(comment);
em.persist(post);  // Comment deleted due to orphanRemoval
```

```java
// Example 3: Multiple CASCADE types
@Entity
public class Customer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToMany(
        mappedBy = "customer",
        cascade = {CascadeType.PERSIST, CascadeType.MERGE}
    )
    private List<Address> addresses = new ArrayList<>();
}
```

### Q14: What is the difference between CascadeType.REMOVE and orphanRemoval?

**CascadeType.REMOVE:**
- Parent deleted → child deleted
- Requires explicit remove() on parent
- Works on parent deletion only

**orphanRemoval:**
- Child removed from collection → child deleted
- Works when relationship is broken
- More automatic cleanup

```java
@Entity
public class Author {
    @OneToMany(mappedBy = "author", cascade = CascadeType.REMOVE)
    private List<Book> books1;

    @OneToMany(mappedBy = "author", orphanRemoval = true)
    private List<Book> books2;
}

// CascadeType.REMOVE
Author author = em.find(Author.class, 1L);
em.remove(author);  // Books deleted

// orphanRemoval
Author author = em.find(Author.class, 1L);
author.getBooks2().remove(0);  // Book deleted automatically
em.persist(author);
```

**Best Practice:** Use `orphanRemoval = true` for parent-child relationships where child cannot exist without parent.

---

## 6. Inheritance Strategies

### Q15: Explain Inheritance Mapping Strategies in JPA.

JPA supports 3 strategies to map inheritance hierarchies to database tables:

### 6.1 SINGLE_TABLE (Default)

**All classes in hierarchy stored in one table**

**Pros:**
- Best performance (no joins)
- Simple
- Polymorphic queries fast

**Cons:**
- Denormalized (nullable columns)
- Large tables
- Data integrity issues

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "payment_type", discriminatorType = DiscriminatorType.STRING)
@Table(name = "payments")
public abstract class Payment {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private BigDecimal amount;
    private LocalDateTime paymentDate;

    // Getters, setters...
}

@Entity
@DiscriminatorValue("CREDIT_CARD")
public class CreditCardPayment extends Payment {
    private String cardNumber;
    private String cvv;
    private String expiryDate;

    // Getters, setters...
}

@Entity
@DiscriminatorValue("PAYPAL")
public class PayPalPayment extends Payment {
    private String email;
    private String transactionId;

    // Getters, setters...
}

@Entity
@DiscriminatorValue("BANK_TRANSFER")
public class BankTransferPayment extends Payment {
    private String accountNumber;
    private String bankName;
    private String swiftCode;

    // Getters, setters...
}

// Database: Single table
/*
payments
---------
id | amount | payment_date | payment_type | card_number | cvv | expiry_date | email | transaction_id | account_number | bank_name | swift_code
*/
```

### 6.2 JOINED

**Each class has its own table, joined via foreign key**

**Pros:**
- Normalized
- No wasted space
- Data integrity preserved

**Cons:**
- Slower (requires joins)
- Complex queries
- More tables

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@Table(name = "vehicles")
public abstract class Vehicle {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String manufacturer;
    private String model;
    private Integer year;

    // Getters, setters...
}

@Entity
@Table(name = "cars")
@PrimaryKeyJoinColumn(name = "vehicle_id")
public class Car extends Vehicle {
    private Integer numberOfDoors;
    private String fuelType;

    // Getters, setters...
}

@Entity
@Table(name = "motorcycles")
@PrimaryKeyJoinColumn(name = "vehicle_id")
public class Motorcycle extends Vehicle {
    private Boolean hasSidecar;
    private Integer engineCC;

    // Getters, setters...
}

@Entity
@Table(name = "trucks")
@PrimaryKeyJoinColumn(name = "vehicle_id")
public class Truck extends Vehicle {
    private Integer payloadCapacity;
    private Integer numberOfAxles;

    // Getters, setters...
}

// Database: Multiple tables with foreign keys
/*
vehicles
--------
id | manufacturer | model | year

cars
----
vehicle_id (FK) | number_of_doors | fuel_type

motorcycles
-----------
vehicle_id (FK) | has_sidecar | engine_cc

trucks
------
vehicle_id (FK) | payload_capacity | number_of_axles
*/
```

### 6.3 TABLE_PER_CLASS

**Each concrete class has its own table with all fields**

**Pros:**
- Each class independent
- No joins for concrete class queries

**Cons:**
- Polymorphic queries very slow (UNION)
- Code duplication
- ID generation issues

```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Person {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String name;
    private String email;

    // Getters, setters...
}

@Entity
@Table(name = "employees")
public class Employee extends Person {
    private String employeeId;
    private BigDecimal salary;
    private String department;

    // Getters, setters...
}

@Entity
@Table(name = "customers")
public class Customer extends Person {
    private String customerId;
    private String loyaltyTier;
    private Integer points;

    // Getters, setters...
}

// Database: Separate tables with duplicated columns
/*
employees
---------
id | name | email | employee_id | salary | department

customers
---------
id | name | email | customer_id | loyalty_tier | points
*/
```

### Q16: How to query polymorphic entities?

```java
// Returns all payment types
List<Payment> payments = em.createQuery(
    "SELECT p FROM Payment p",
    Payment.class
).getResultList();

// SINGLE_TABLE: Single SELECT
// JOINED: SELECT with LEFT JOIN on all tables
// TABLE_PER_CLASS: UNION of all tables

// Query specific subclass
List<CreditCardPayment> ccPayments = em.createQuery(
    "SELECT p FROM CreditCardPayment p",
    CreditCardPayment.class
).getResultList();

// Type checking in queries
List<Payment> result = em.createQuery(
    "SELECT p FROM Payment p WHERE TYPE(p) = CreditCardPayment",
    Payment.class
).getResultList();
```

### Q17: Which inheritance strategy should you use?

**Recommendations:**

- **Use SINGLE_TABLE when:**
  - Subclasses have few differences
  - Performance is critical
  - Polymorphic queries are common

- **Use JOINED when:**
  - Many subclasses with many unique fields
  - Normalization important
  - Storage efficiency matters

- **Use TABLE_PER_CLASS when:**
  - Rarely (it's least efficient)
  - Classes are truly independent
  - No polymorphic queries

**Most Common:** SINGLE_TABLE (default) for simplicity and performance.

---

## 7. N+1 Problem

### Q18: What is the N+1 SELECT problem and how to solve it?

**N+1 Problem:** Executing 1 query to fetch N parent entities, then N additional queries to fetch related child entities.

**Example of the Problem:**

```java
// 1 query: SELECT * FROM departments
List<Department> departments = em.createQuery(
    "SELECT d FROM Department d",
    Department.class
).getResultList();

// N queries: SELECT * FROM employees WHERE department_id = ?
for (Department dept : departments) {
    // Each iteration triggers a separate query!
    System.out.println(dept.getName() + ": " + dept.getEmployees().size());
}
// Total: 1 + 100 = 101 queries if there are 100 departments!
```

**SQL Generated:**
```sql
SELECT * FROM departments;  -- 1 query
SELECT * FROM employees WHERE department_id = 1;  -- +1
SELECT * FROM employees WHERE department_id = 2;  -- +1
SELECT * FROM employees WHERE department_id = 3;  -- +1
...
```

### Q19: How to solve the N+1 problem?

**Solution 1: JOIN FETCH**

```java
// Single query with LEFT JOIN
List<Department> departments = em.createQuery(
    "SELECT DISTINCT d FROM Department d LEFT JOIN FETCH d.employees",
    Department.class
).getResultList();

// DISTINCT prevents duplicate departments
// All employees loaded in one query
```

**SQL Generated:**
```sql
SELECT DISTINCT d.*, e.*
FROM departments d
LEFT JOIN employees e ON d.id = e.department_id;
```

**Solution 2: Entity Graph**

```java
// Define fetch graph
EntityGraph<Department> graph = em.createEntityGraph(Department.class);
graph.addAttributeNodes("employees");

// Apply to query
List<Department> departments = em.createQuery(
    "SELECT d FROM Department d",
    Department.class
)
.setHint("jakarta.persistence.fetchgraph", graph)
.getResultList();
```

**Solution 3: @NamedEntityGraph (Static)**

```java
@Entity
@NamedEntityGraph(
    name = "Department.employees",
    attributeNodes = @NamedAttributeNode("employees")
)
public class Department {
    // ...
}

// Usage
EntityGraph<?> graph = em.getEntityGraph("Department.employees");
List<Department> departments = em.createQuery(
    "SELECT d FROM Department d",
    Department.class
)
.setHint("jakarta.persistence.fetchgraph", graph)
.getResultList();
```

**Solution 4: Batch Fetching (Hibernate-specific)**

```java
@Entity
public class Department {
    @OneToMany(mappedBy = "department")
    @BatchSize(size = 10)  // Fetch in batches of 10
    private List<Employee> employees;
}

// Instead of N queries, executes N/10 queries
```

**Solution 5: @Fetch(FetchMode.SUBSELECT) (Hibernate-specific)**

```java
@Entity
public class Department {
    @OneToMany(mappedBy = "department")
    @Fetch(FetchMode.SUBSELECT)
    private List<Employee> employees;
}

// Executes 2 queries total:
// 1. SELECT departments
// 2. SELECT employees WHERE department_id IN (subquery)
```

**Solution 6: DTO Projection**

```java
// Fetch only needed data
List<DepartmentSummary> summaries = em.createQuery(
    "SELECT new com.example.DepartmentSummary(d.name, COUNT(e)) " +
    "FROM Department d LEFT JOIN d.employees e " +
    "GROUP BY d.name",
    DepartmentSummary.class
).getResultList();

public class DepartmentSummary {
    private String name;
    private Long employeeCount;
    // Constructor, getters...
}
```

**Comparison:**

| Solution | Queries | Performance | Use Case |
|----------|---------|-------------|----------|
| JOIN FETCH | 1 | Best | Load full entities |
| Entity Graph | 1 | Best | Dynamic fetch plans |
| Batch Fetch | N/batch | Good | Large collections |
| Subselect | 2 | Good | Multiple collections |
| DTO | 1 | Best | Read-only data |

---

## 8. Caching

### Q20: Explain First Level Cache in JPA.

**First Level Cache (L1):**
- Also called **Persistence Context** or **Session Cache**
- Enabled by default
- Scoped to EntityManager/Session
- Stores all entities loaded/saved in transaction
- Prevents duplicate queries within same session
- Automatically cleared when EM closes

**Example:**

```java
EntityManager em = emf.createEntityManager();

// First access - query database
User user1 = em.find(User.class, 1L);
System.out.println("First access: " + user1.getName());

// Second access - from L1 cache, no SQL!
User user2 = em.find(User.class, 1L);
System.out.println("Second access: " + user2.getName());

// Same instance
System.out.println(user1 == user2);  // true

em.close();  // L1 cache cleared

// New EM, new cache
em = emf.createEntityManager();
User user3 = em.find(User.class, 1L);  // Query database again
```

**Benefits:**
- Reduces database hits
- Ensures entity uniqueness in session
- Automatic dirty checking

**Limitations:**
- Not shared across sessions
- Not configurable (always on)
- Lost when EM closes

### Q21: Explain Second Level Cache in JPA.

**Second Level Cache (L2):**
- Application-level cache
- Shared across all EntityManagers
- Optional (must be enabled)
- Configured per entity
- Requires cache provider (EhCache, Hazelcast, Infinispan)
- Survives session closure

**Configuration:**

```xml
<!-- persistence.xml -->
<property name="hibernate.cache.use_second_level_cache" value="true"/>
<property name="hibernate.cache.region.factory_class"
          value="org.hibernate.cache.jcache.JCacheRegionFactory"/>
<property name="hibernate.javax.cache.provider"
          value="org.ehcache.jsr107.EhcacheCachingProvider"/>
<property name="hibernate.javax.cache.uri"
          value="ehcache.xml"/>
```

**Entity Configuration:**

```java
@Entity
@Cacheable
@org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Product {
    @Id
    private Long id;
    private String name;
    private BigDecimal price;

    @ManyToOne
    @JoinColumn(name = "category_id")
    private Category category;
}

@Entity
@Cacheable
@org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_ONLY)
public class Category {
    @Id
    private Long id;
    private String name;

    @OneToMany(mappedBy = "category")
    @org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
    private List<Product> products;
}
```

**Cache Concurrency Strategies:**

1. **READ_ONLY**
   - Immutable data (never updated)
   - Best performance
   - Use for: lookup tables, static reference data

2. **NONSTRICT_READ_WRITE**
   - Rarely updated data
   - No strict consistency guarantee
   - Use for: data where stale reads are acceptable

3. **READ_WRITE**
   - Frequently read, occasionally written
   - Uses locks for consistency
   - Use for: general caching

4. **TRANSACTIONAL**
   - Full ACID transactions on cache
   - Slowest but safest
   - Use for: critical data requiring strict consistency

**Usage Example:**

```java
// Session 1
EntityManager em1 = emf.createEntityManager();
Product p1 = em1.find(Product.class, 1L);  // DB query + cache store
em1.close();

// Session 2
EntityManager em2 = emf.createEntityManager();
Product p2 = em2.find(Product.class, 1L);  // From L2 cache, no DB hit!
em2.close();

// Note: p1 != p2 (different instances, same data)
```

### Q22: What is Query Cache?

**Query Cache:**
- Caches JPQL/HQL query results
- Requires L2 cache enabled
- Must be enabled per query
- Cached by query string + parameters
- Invalidated when queried tables change

**Configuration:**

```xml
<property name="hibernate.cache.use_query_cache" value="true"/>
```

**Usage:**

```java
// Enable caching for specific query
List<Product> products = em.createQuery(
    "SELECT p FROM Product p WHERE p.price > :price",
    Product.class
)
.setParameter("price", new BigDecimal("100"))
.setHint("org.hibernate.cacheable", true)
.getResultList();

// Second execution - from cache
List<Product> cached = em.createQuery(
    "SELECT p FROM Product p WHERE p.price > :price",
    Product.class
)
.setParameter("price", new BigDecimal("100"))
.setHint("org.hibernate.cacheable", true)
.getResultList();  // No database query
```

**Important:** Query cache stores only IDs, entities fetched from L2/L1/DB.

### Q23: How to evict/clear cache?

```java
// Clear L1 cache (current session)
em.clear();  // Detach all entities

// Refresh entity from DB (bypass cache)
em.refresh(user);

// Evict entity from L2 cache
Cache cache = emf.getCache();
cache.evict(Product.class, 1L);  // Evict specific entity
cache.evict(Product.class);      // Evict all Product entities
cache.evictAll();                // Clear entire cache

// Hibernate-specific
Session session = em.unwrap(Session.class);
session.getSessionFactory().getCache().evictAllRegions();
```

**Cache Statistics:**

```java
// Enable statistics
<property name="hibernate.generate_statistics" value="true"/>

// Get statistics
Statistics stats = emf.unwrap(SessionFactory.class).getStatistics();
System.out.println("Second Level Hit Count: " + stats.getSecondLevelCacheHitCount());
System.out.println("Second Level Miss Count: " + stats.getSecondLevelCacheMissCount());
System.out.println("Query Cache Hit Count: " + stats.getQueryCacheHitCount());
```

**Best Practices:**
1. Cache read-heavy entities
2. Don't cache frequently modified data
3. Monitor hit ratios
4. Use appropriate concurrency strategy
5. Configure eviction policies
6. Be aware of memory consumption

---

## 9. JPQL vs Native SQL

### Q24: What is JPQL? How is it different from SQL?

**JPQL (Java Persistence Query Language):**
- Object-oriented query language
- Queries entities, not tables
- Database-independent
- Supports inheritance and polymorphism
- Type-safe

**SQL:**
- Table-oriented
- Database-specific
- No object mapping
- Works with columns and tables

**Example Comparison:**

```java
// JPQL - queries entities
String jpql = "SELECT u FROM User u WHERE u.email = :email";
TypedQuery<User> query = em.createQuery(jpql, User.class);
query.setParameter("email", "john@example.com");
User user = query.getSingleResult();

// Native SQL - queries tables
String sql = "SELECT * FROM users WHERE email = ?";
Query query = em.createNativeQuery(sql, User.class);
query.setParameter(1, "john@example.com");
User user = (User) query.getSingleResult();
```

### Q25: JPQL Examples

**Basic Queries:**

```java
// Select all
List<User> users = em.createQuery("SELECT u FROM User u", User.class)
                     .getResultList();

// Where clause
List<User> activeUsers = em.createQuery(
    "SELECT u FROM User u WHERE u.active = true",
    User.class
).getResultList();

// Parameters (named)
List<User> users = em.createQuery(
    "SELECT u FROM User u WHERE u.age > :minAge AND u.city = :city",
    User.class
)
.setParameter("minAge", 18)
.setParameter("city", "New York")
.getResultList();

// Parameters (positional)
List<User> users = em.createQuery(
    "SELECT u FROM User u WHERE u.age > ?1 AND u.city = ?2",
    User.class
)
.setParameter(1, 18)
.setParameter(2, "New York")
.getResultList();
```

**Joins:**

```java
// Inner Join
List<Order> orders = em.createQuery(
    "SELECT o FROM Order o JOIN o.customer c WHERE c.name = :name",
    Order.class
)
.setParameter("name", "John")
.getResultList();

// Left Join
List<Department> depts = em.createQuery(
    "SELECT d FROM Department d LEFT JOIN d.employees e",
    Department.class
).getResultList();

// Join Fetch (eager load)
List<Order> orders = em.createQuery(
    "SELECT o FROM Order o JOIN FETCH o.items WHERE o.id = :id",
    Order.class
)
.setParameter("id", 1L)
.getResultList();
```

**Aggregations:**

```java
// Count
Long count = em.createQuery(
    "SELECT COUNT(u) FROM User u WHERE u.active = true",
    Long.class
).getSingleResult();

// Sum
BigDecimal total = em.createQuery(
    "SELECT SUM(o.amount) FROM Order o WHERE o.status = 'COMPLETED'",
    BigDecimal.class
).getSingleResult();

// Group By
List<Object[]> results = em.createQuery(
    "SELECT u.city, COUNT(u) FROM User u GROUP BY u.city HAVING COUNT(u) > 10"
).getResultList();

for (Object[] row : results) {
    String city = (String) row[0];
    Long userCount = (Long) row[1];
    System.out.println(city + ": " + userCount);
}
```

**Ordering and Pagination:**

```java
// Order By
List<User> users = em.createQuery(
    "SELECT u FROM User u ORDER BY u.createdAt DESC, u.name ASC",
    User.class
).getResultList();

// Pagination
List<User> page = em.createQuery("SELECT u FROM User u ORDER BY u.id", User.class)
                    .setFirstResult(20)   // Offset
                    .setMaxResults(10)    // Limit
                    .getResultList();
```

**Subqueries:**

```java
// Subquery
List<User> users = em.createQuery(
    "SELECT u FROM User u WHERE u.id IN " +
    "(SELECT o.customer.id FROM Order o WHERE o.total > 1000)",
    User.class
).getResultList();

// Exists
List<Department> depts = em.createQuery(
    "SELECT d FROM Department d WHERE EXISTS " +
    "(SELECT e FROM Employee e WHERE e.department = d AND e.salary > 100000)",
    Department.class
).getResultList();
```

**DTO Projection:**

```java
// Constructor expression
List<UserDTO> dtos = em.createQuery(
    "SELECT new com.example.UserDTO(u.id, u.name, u.email) FROM User u",
    UserDTO.class
).getResultList();

public class UserDTO {
    private Long id;
    private String name;
    private String email;

    public UserDTO(Long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }
    // Getters...
}
```

**Update and Delete:**

```java
// Bulk update
int updated = em.createQuery(
    "UPDATE User u SET u.active = false WHERE u.lastLogin < :date"
)
.setParameter("date", LocalDate.now().minusMonths(6))
.executeUpdate();

// Bulk delete
int deleted = em.createQuery(
    "DELETE FROM Order o WHERE o.status = 'CANCELLED' AND o.createdAt < :date"
)
.setParameter("date", LocalDate.now().minusYears(1))
.executeUpdate();
```

### Q26: When to use Native SQL queries?

**Use Native SQL when:**
1. Database-specific functions needed
2. Complex queries JPQL can't express
3. Performance optimization with hints
4. Existing SQL you want to reuse
5. Database-specific features (window functions, CTEs)

**Example:**

```java
// Native SQL with entity mapping
List<User> users = em.createNativeQuery(
    "SELECT * FROM users WHERE created_at > NOW() - INTERVAL '7 days'",
    User.class
).getResultList();

// Native SQL with scalar result
List<Object[]> results = em.createNativeQuery(
    "SELECT u.name, COUNT(o.id) as order_count " +
    "FROM users u " +
    "LEFT JOIN orders o ON u.id = o.customer_id " +
    "GROUP BY u.name " +
    "HAVING COUNT(o.id) > 5"
).getResultList();

// Native SQL with result mapping
@SqlResultSetMapping(
    name = "UserOrderMapping",
    entities = {
        @EntityResult(entityClass = User.class),
        @EntityResult(entityClass = Order.class)
    }
)
Query query = em.createNativeQuery(
    "SELECT u.*, o.* FROM users u JOIN orders o ON u.id = o.customer_id",
    "UserOrderMapping"
);
```

**Named Native Queries:**

```java
@Entity
@NamedNativeQuery(
    name = "User.findByCity",
    query = "SELECT * FROM users WHERE city = :city",
    resultClass = User.class
)
public class User {
    // ...
}

// Usage
List<User> users = em.createNamedQuery("User.findByCity", User.class)
                     .setParameter("city", "Boston")
                     .getResultList();
```

---

## 10. Criteria API

### Q27: What is Criteria API? Why use it?

**Criteria API:**
- Type-safe, programmatic query API
- Alternative to JPQL string-based queries
- Compile-time validation
- Dynamic query building
- Refactoring-friendly

**Advantages:**
- No query syntax errors at runtime
- IDE autocomplete support
- Easy to build dynamic queries
- Refactoring updates queries automatically

**Disadvantages:**
- More verbose than JPQL
- Steeper learning curve
- Complex queries harder to read

### Q28: Criteria API Examples

**Basic Queries:**

```java
// SELECT u FROM User u
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<User> cq = cb.createQuery(User.class);
Root<User> user = cq.from(User.class);
cq.select(user);

List<User> users = em.createQuery(cq).getResultList();
```

**Where Clause:**

```java
// SELECT u FROM User u WHERE u.age > 18 AND u.active = true
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<User> cq = cb.createQuery(User.class);
Root<User> user = cq.from(User.class);

Predicate agePredicate = cb.greaterThan(user.get("age"), 18);
Predicate activePredicate = cb.equal(user.get("active"), true);

cq.select(user).where(cb.and(agePredicate, activePredicate));

List<User> users = em.createQuery(cq).getResultList();
```

**Dynamic Query Building:**

```java
public List<User> searchUsers(String name, Integer minAge, String city) {
    CriteriaBuilder cb = em.getCriteriaBuilder();
    CriteriaQuery<User> cq = cb.createQuery(User.class);
    Root<User> user = cq.from(User.class);

    List<Predicate> predicates = new ArrayList<>();

    if (name != null) {
        predicates.add(cb.like(user.get("name"), "%" + name + "%"));
    }

    if (minAge != null) {
        predicates.add(cb.greaterThanOrEqualTo(user.get("age"), minAge));
    }

    if (city != null) {
        predicates.add(cb.equal(user.get("city"), city));
    }

    cq.select(user).where(cb.and(predicates.toArray(new Predicate[0])));

    return em.createQuery(cq).getResultList();
}
```

**Joins:**

```java
// SELECT o FROM Order o JOIN o.customer c WHERE c.name = 'John'
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Order> cq = cb.createQuery(Order.class);
Root<Order> order = cq.from(Order.class);
Join<Order, Customer> customer = order.join("customer");

cq.select(order).where(cb.equal(customer.get("name"), "John"));

List<Order> orders = em.createQuery(cq).getResultList();
```

**Fetch Join:**

```java
// SELECT DISTINCT o FROM Order o LEFT JOIN FETCH o.items
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Order> cq = cb.createQuery(Order.class);
Root<Order> order = cq.from(Order.class);
order.fetch("items", JoinType.LEFT);

cq.select(order).distinct(true);

List<Order> orders = em.createQuery(cq).getResultList();
```

**Ordering:**

```java
// SELECT u FROM User u ORDER BY u.createdAt DESC, u.name ASC
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<User> cq = cb.createQuery(User.class);
Root<User> user = cq.from(User.class);

cq.select(user).orderBy(
    cb.desc(user.get("createdAt")),
    cb.asc(user.get("name"))
);

List<User> users = em.createQuery(cq).getResultList();
```

**Aggregations:**

```java
// SELECT COUNT(u) FROM User u WHERE u.active = true
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Long> cq = cb.createQuery(Long.class);
Root<User> user = cq.from(User.class);

cq.select(cb.count(user)).where(cb.equal(user.get("active"), true));

Long count = em.createQuery(cq).getSingleResult();

// SELECT u.city, COUNT(u) FROM User u GROUP BY u.city HAVING COUNT(u) > 10
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Object[]> cq = cb.createQuery(Object[].class);
Root<User> user = cq.from(User.class);

cq.multiselect(user.get("city"), cb.count(user))
  .groupBy(user.get("city"))
  .having(cb.greaterThan(cb.count(user), 10L));

List<Object[]> results = em.createQuery(cq).getResultList();
```

**Metamodel (Type-Safe):**

```java
// Generate metamodel classes
// User_ class auto-generated from User entity

CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<User> cq = cb.createQuery(User.class);
Root<User> user = cq.from(User.class);

// Type-safe attributes
cq.select(user).where(
    cb.and(
        cb.greaterThan(user.get(User_.age), 18),
        cb.equal(user.get(User_.active), true)
    )
);

List<User> users = em.createQuery(cq).getResultList();
```

**DTO Projection:**

```java
// SELECT new UserDTO(u.id, u.name) FROM User u
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<UserDTO> cq = cb.createQuery(UserDTO.class);
Root<User> user = cq.from(User.class);

cq.select(cb.construct(
    UserDTO.class,
    user.get("id"),
    user.get("name")
));

List<UserDTO> dtos = em.createQuery(cq).getResultList();
```

---

## 11. Transaction Management

### Q29: Explain JPA Transactions

**Transaction:** Logical unit of work that must be atomic (all or nothing).

**ACID Properties:**
- **Atomicity**: All operations succeed or all fail
- **Consistency**: Database remains in valid state
- **Isolation**: Concurrent transactions don't interfere
- **Durability**: Committed changes are permanent

**Basic Transaction:**

```java
EntityManager em = emf.createEntityManager();
EntityTransaction tx = em.getTransaction();

try {
    tx.begin();

    // Database operations
    User user = new User("John", "john@example.com");
    em.persist(user);

    Order order = new Order(user, new BigDecimal("99.99"));
    em.persist(order);

    tx.commit();  // Success - changes saved
} catch (Exception e) {
    if (tx.isActive()) {
        tx.rollback();  // Failure - changes reverted
    }
    throw e;
} finally {
    em.close();
}
```

### Q30: What are Transaction Isolation Levels?

**Isolation Levels** control how concurrent transactions see each other's changes.

**Problems:**
- **Dirty Read**: Read uncommitted data from other transaction
- **Non-repeatable Read**: Same query returns different results
- **Phantom Read**: Different rows appear in range query

**Levels (from weakest to strongest):**

1. **READ_UNCOMMITTED** (Level 0)
   - Allows dirty reads
   - Lowest isolation, highest performance
   - Rarely used

2. **READ_COMMITTED** (Level 1)
   - Prevents dirty reads
   - Default in most databases
   - Non-repeatable reads possible

3. **REPEATABLE_READ** (Level 2)
   - Prevents dirty and non-repeatable reads
   - Phantom reads possible
   - MySQL default

4. **SERIALIZABLE** (Level 3)
   - Prevents all problems
   - Highest isolation, lowest performance
   - Transactions executed serially

**Setting Isolation Level:**

```java
// JPA (persistence.xml)
<property name="hibernate.connection.isolation" value="2"/>
// 1=READ_UNCOMMITTED, 2=READ_COMMITTED, 4=REPEATABLE_READ, 8=SERIALIZABLE

// Programmatically
Session session = em.unwrap(Session.class);
session.doWork(connection -> {
    connection.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);
});

// Spring @Transactional
@Transactional(isolation = Isolation.REPEATABLE_READ)
public void someMethod() {
    // ...
}
```

**Example Scenarios:**

```java
// READ_COMMITTED - Non-repeatable read problem
// Transaction 1
tx1.begin();
User user = em.find(User.class, 1L);
System.out.println(user.getBalance());  // $100

// Transaction 2 (concurrent)
tx2.begin();
user.setBalance(new BigDecimal("200"));
em.merge(user);
tx2.commit();  // Balance now $200

// Transaction 1 continues
User userAgain = em.find(User.class, 1L);
System.out.println(userAgain.getBalance());  // $200 (different!)
tx1.commit();

// REPEATABLE_READ - would prevent this
```

### Q31: Spring Transaction Management

**Declarative Transactions:**

```java
@Service
public class OrderService {

    @Autowired
    private EntityManager em;

    @Transactional
    public void createOrder(Order order) {
        // Transaction started automatically
        em.persist(order);
        // Transaction committed automatically
    }

    @Transactional(readOnly = true)
    public List<Order> findAllOrders() {
        // Optimized for read-only
        return em.createQuery("SELECT o FROM Order o", Order.class)
                 .getResultList();
    }

    @Transactional(
        propagation = Propagation.REQUIRED,
        isolation = Isolation.READ_COMMITTED,
        timeout = 30,
        rollbackFor = Exception.class,
        noRollbackFor = IllegalArgumentException.class
    )
    public void complexOperation() {
        // Transaction with custom settings
    }
}
```

**Propagation Types:**

```java
// REQUIRED (default) - Use existing or create new
@Transactional(propagation = Propagation.REQUIRED)
public void method1() {
    method2();  // Uses same transaction
}

@Transactional(propagation = Propagation.REQUIRED)
public void method2() {
    // Participates in method1's transaction
}

// REQUIRES_NEW - Always create new transaction
@Transactional(propagation = Propagation.REQUIRED)
public void method1() {
    method2();  // New independent transaction
}

@Transactional(propagation = Propagation.REQUIRES_NEW)
public void method2() {
    // New transaction, suspends method1's transaction
}

// NESTED - Create savepoint in existing transaction
@Transactional(propagation = Propagation.NESTED)
public void method() {
    // Can rollback independently
}

// MANDATORY - Must run in existing transaction
@Transactional(propagation = Propagation.MANDATORY)
public void method() {
    // Throws exception if no active transaction
}

// NEVER - Must NOT run in transaction
@Transactional(propagation = Propagation.NEVER)
public void method() {
    // Throws exception if transaction exists
}

// NOT_SUPPORTED - Suspend transaction if exists
@Transactional(propagation = Propagation.NOT_SUPPORTED)
public void method() {
    // Runs outside transaction
}

// SUPPORTS - Use transaction if exists, otherwise don't
@Transactional(propagation = Propagation.SUPPORTS)
public void method() {
    // Optional transaction
}
```

### Q32: Optimistic vs Pessimistic Locking

**Optimistic Locking:**
- Assume conflicts are rare
- Check version before commit
- Throws OptimisticLockException if conflict
- Better performance for read-heavy apps

```java
@Entity
public class Product {
    @Id
    private Long id;

    private String name;
    private BigDecimal price;

    @Version  // Optimistic locking
    private Long version;

    // Getters, setters...
}

// Usage
// User 1
Product product1 = em.find(Product.class, 1L);  // version = 1
product1.setPrice(new BigDecimal("99.99"));

// User 2 (concurrent)
Product product2 = em.find(Product.class, 1L);  // version = 1
product2.setPrice(new BigDecimal("89.99"));

tx1.commit();  // Success, version becomes 2

tx2.commit();  // OptimisticLockException! Version mismatch
```

**Pessimistic Locking:**
- Assume conflicts are common
- Lock database row explicitly
- Blocks other transactions
- Better for write-heavy apps

```java
// PESSIMISTIC_READ - Shared lock (others can read)
Product product = em.find(Product.class, 1L, LockModeType.PESSIMISTIC_READ);

// PESSIMISTIC_WRITE - Exclusive lock (others blocked)
Product product = em.find(Product.class, 1L, LockModeType.PESSIMISTIC_WRITE);

// With timeout
Map<String, Object> properties = new HashMap<>();
properties.put("jakarta.persistence.lock.timeout", 5000);  // 5 seconds
Product product = em.find(Product.class, 1L, LockModeType.PESSIMISTIC_WRITE, properties);

// In query
TypedQuery<Product> query = em.createQuery(
    "SELECT p FROM Product p WHERE p.id = :id",
    Product.class
);
query.setParameter("id", 1L);
query.setLockMode(LockModeType.PESSIMISTIC_WRITE);
Product product = query.getSingleResult();
```

**Comparison:**

| Aspect | Optimistic | Pessimistic |
|--------|-----------|-------------|
| Locking | At commit | At read |
| Performance | Better (no locks) | Worse (locks) |
| Use Case | Low contention | High contention |
| Failure | Exception at commit | Wait or timeout |
| Scalability | Better | Worse |

---

## 12. Performance Optimization

### Q33: Best practices for JPA/Hibernate performance optimization

**1. Use Appropriate Fetch Strategies**

```java
// Default to LAZY
@OneToMany(fetch = FetchType.LAZY, mappedBy = "user")
private List<Order> orders;

// Use JOIN FETCH for eager loading when needed
List<User> users = em.createQuery(
    "SELECT u FROM User u LEFT JOIN FETCH u.orders",
    User.class
).getResultList();
```

**2. Avoid N+1 Queries**

```java
// Bad - N+1 problem
List<Department> depts = em.createQuery("SELECT d FROM Department d", Department.class)
                           .getResultList();
for (Department dept : depts) {
    dept.getEmployees().size();  // N queries!
}

// Good - Single query
List<Department> depts = em.createQuery(
    "SELECT DISTINCT d FROM Department d LEFT JOIN FETCH d.employees",
    Department.class
).getResultList();
```

**3. Use Pagination**

```java
// Don't load all records
List<User> page = em.createQuery("SELECT u FROM User u ORDER BY u.id", User.class)
                    .setFirstResult(offset)
                    .setMaxResults(pageSize)
                    .getResultList();
```

**4. Use DTO Projections for Read-Only Queries**

```java
// Don't fetch full entities if you need only few fields
List<UserSummary> summaries = em.createQuery(
    "SELECT new com.example.UserSummary(u.id, u.name, u.email) FROM User u",
    UserSummary.class
).getResultList();

// Or use interface projection (Spring Data JPA)
public interface UserSummary {
    Long getId();
    String getName();
    String getEmail();
}
```

**5. Enable Batch Processing**

```java
// Insert/Update in batches
<property name="hibernate.jdbc.batch_size" value="50"/>
<property name="hibernate.order_inserts" value="true"/>
<property name="hibernate.order_updates" value="true"/>
<property name="hibernate.jdbc.batch_versioned_data" value="true"/>

// Usage
for (int i = 0; i < 1000; i++) {
    User user = new User("User" + i, "user" + i + "@example.com");
    em.persist(user);

    if (i % 50 == 0) {  // Batch size
        em.flush();
        em.clear();
    }
}
```

**6. Use Second Level Cache**

```java
// Cache frequently accessed, rarely modified data
@Entity
@Cacheable
@org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_ONLY)
public class Country {
    // Lookup table
}

@Entity
@Cacheable
@org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Product {
    // Product catalog
}
```

**7. Use Read-Only Queries**

```java
// Optimization for read-only data
@Transactional(readOnly = true)
public List<User> findAllUsers() {
    return em.createQuery("SELECT u FROM User u", User.class)
             .setHint("org.hibernate.readOnly", true)
             .getResultList();
}
```

**8. Use Stateless Session for Bulk Operations**

```java
// Hibernate-specific - bypass persistence context
StatelessSession session = sessionFactory.openStatelessSession();
Transaction tx = session.beginTransaction();

for (int i = 0; i < 100000; i++) {
    User user = new User("User" + i, "user" + i + "@example.com");
    session.insert(user);
}

tx.commit();
session.close();
```

**9. Enable Query Plan Cache**

```java
<property name="hibernate.query.plan_cache_max_size" value="2048"/>
<property name="hibernate.query.plan_parameter_metadata_max_size" value="128"/>
```

**10. Use Connection Pooling**

```java
// HikariCP configuration
<property name="hibernate.connection.provider_class"
          value="org.hibernate.hikaricp.internal.HikariCPConnectionProvider"/>
<property name="hibernate.hikari.minimumIdle" value="5"/>
<property name="hibernate.hikari.maximumPoolSize" value="20"/>
<property name="hibernate.hikari.idleTimeout" value="300000"/>
<property name="hibernate.hikari.connectionTimeout" value="20000"/>
```

**11. Monitor and Log SQL**

```java
// Development only!
<property name="hibernate.show_sql" value="true"/>
<property name="hibernate.format_sql" value="true"/>
<property name="hibernate.use_sql_comments" value="true"/>

// Production - use logging
<property name="hibernate.generate_statistics" value="true"/>

// Log slow queries
<property name="hibernate.session.events.log.LOG_QUERIES_SLOWER_THAN_MS" value="100"/>
```

**12. Use Database Indexes**

```java
@Entity
@Table(
    name = "users",
    indexes = {
        @Index(name = "idx_email", columnList = "email"),
        @Index(name = "idx_username", columnList = "username"),
        @Index(name = "idx_city_age", columnList = "city, age")
    }
)
public class User {
    @Column(unique = true)
    private String email;  // Unique index automatically created
}
```

**13. Avoid Large Collections**

```java
// Bad - loads entire collection
@OneToMany(mappedBy = "user")
private List<Order> orders;  // Could be thousands!

// Good - use queries with pagination
public List<Order> getRecentOrders(User user, int limit) {
    return em.createQuery(
        "SELECT o FROM Order o WHERE o.user = :user ORDER BY o.createdAt DESC",
        Order.class
    )
    .setParameter("user", user)
    .setMaxResults(limit)
    .getResultList();
}
```

**14. Use @BatchSize for Collections**

```java
@Entity
public class Department {
    @OneToMany(mappedBy = "department")
    @BatchSize(size = 10)  // Fetch 10 departments' employees at once
    private List<Employee> employees;
}
```

**15. Clear Persistence Context Regularly**

```java
// Prevent memory issues in long-running transactions
for (int i = 0; i < 10000; i++) {
    User user = em.find(User.class, i);
    // process user

    if (i % 100 == 0) {
        em.flush();   // Sync with DB
        em.clear();   // Clear L1 cache
    }
}
```

### Q34: Common Performance Anti-Patterns

**1. Cartesian Product in JOIN FETCH**

```java
// WRONG - Creates cartesian product
"SELECT u FROM User u " +
"JOIN FETCH u.orders " +
"JOIN FETCH u.addresses"  // Multiplies results!

// CORRECT - Separate queries or use @EntityGraph
EntityGraph<User> graph = em.createEntityGraph(User.class);
graph.addAttributeNodes("orders", "addresses");
```

**2. Fetching Entities When DTOs Suffice**

```java
// WRONG - Full entity with all fields and relationships
List<User> users = em.createQuery("SELECT u FROM User u", User.class)
                     .getResultList();

// CORRECT - Only needed data
List<UserDTO> users = em.createQuery(
    "SELECT new UserDTO(u.id, u.name) FROM User u",
    UserDTO.class
).getResultList();
```

**3. Missing Indexes**

```java
// WRONG - No index on frequently queried field
@Column
private String email;

// CORRECT
@Column
@Index(name = "idx_email")
private String email;
```

**4. EAGER Loading Everything**

```java
// WRONG
@OneToMany(fetch = FetchType.EAGER, mappedBy = "user")
private List<Order> orders;

// CORRECT
@OneToMany(fetch = FetchType.LAZY, mappedBy = "user")
private List<Order> orders;
```

**5. Not Using Bulk Operations**

```java
// WRONG - N queries
for (User user : users) {
    user.setActive(false);
    em.merge(user);
}

// CORRECT - 1 query
em.createQuery("UPDATE User u SET u.active = false WHERE u.lastLogin < :date")
  .setParameter("date", cutoffDate)
  .executeUpdate();
```

---

## Additional Common Interview Questions

### Q35: What is the difference between save() and persist()?

**Answer:** In pure JPA, only `persist()` exists. `save()` is a Hibernate-specific method.

- **persist()**: Returns void, makes entity managed
- **save()** (Hibernate): Returns generated identifier

```java
// JPA
em.persist(user);  // void

// Hibernate
Serializable id = session.save(user);  // returns ID
```

### Q36: What is FlushMode?

**FlushMode** controls when changes are synchronized to the database.

- **AUTO** (default): Flush before query and commit
- **COMMIT**: Flush only on commit
- **MANUAL**: Flush only when explicitly called

```java
em.setFlushMode(FlushModeType.COMMIT);

User user = em.find(User.class, 1L);
user.setName("Updated");

// Query won't see the change because not flushed
List<User> users = em.createQuery("SELECT u FROM User u WHERE u.name = 'Updated'", User.class)
                     .getResultList();  // Empty!

em.flush();  // Now synchronized
```

### Q37: What is @Embedded and @Embeddable?

**@Embeddable**: Class whose instances are stored as part of owning entity.

```java
@Embeddable
public class Address {
    private String street;
    private String city;
    private String zipCode;
    private String country;

    // No @Id - not an entity
    // Getters, setters...
}

@Entity
public class User {
    @Id
    private Long id;

    private String name;

    @Embedded
    private Address address;

    @Embedded
    @AttributeOverrides({
        @AttributeOverride(name = "street", column = @Column(name = "billing_street")),
        @AttributeOverride(name = "city", column = @Column(name = "billing_city")),
        @AttributeOverride(name = "zipCode", column = @Column(name = "billing_zip")),
        @AttributeOverride(name = "country", column = @Column(name = "billing_country"))
    })
    private Address billingAddress;
}

// Single table: users
// Columns: id, name, street, city, zip_code, country, billing_street, billing_city, billing_zip, billing_country
```

### Q38: What are entity listeners and lifecycle callbacks?

**Lifecycle Events:**
- `@PrePersist`: Before persist()
- `@PostPersist`: After persist()
- `@PreUpdate`: Before update
- `@PostUpdate`: After update
- `@PreRemove`: Before remove()
- `@PostRemove`: After remove()
- `@PostLoad`: After entity loaded

```java
@Entity
public class User {
    @Id
    private Long id;

    private String name;
    private LocalDateTime createdAt;
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

    @PostLoad
    protected void onLoad() {
        System.out.println("Entity loaded: " + id);
    }
}

// External listener
@EntityListeners(AuditListener.class)
@Entity
public class Product {
    // ...
}

public class AuditListener {
    @PrePersist
    public void prePersist(Object entity) {
        System.out.println("Persisting: " + entity);
    }

    @PostPersist
    public void postPersist(Object entity) {
        System.out.println("Persisted: " + entity);
    }
}
```

### Q39: How to handle composite primary keys?

**Two approaches:**

**1. @IdClass**

```java
public class OrderItemId implements Serializable {
    private Long orderId;
    private Long productId;

    // Default constructor, equals, hashCode
}

@Entity
@IdClass(OrderItemId.class)
public class OrderItem {
    @Id
    private Long orderId;

    @Id
    private Long productId;

    private Integer quantity;

    // Getters, setters...
}

// Usage
OrderItem item = em.find(OrderItem.class, new OrderItemId(1L, 100L));
```

**2. @EmbeddedId**

```java
@Embeddable
public class OrderItemId implements Serializable {
    private Long orderId;
    private Long productId;

    // Constructor, equals, hashCode, getters, setters
}

@Entity
public class OrderItem {
    @EmbeddedId
    private OrderItemId id;

    private Integer quantity;

    // Getters, setters...
}

// Usage
OrderItemId id = new OrderItemId(1L, 100L);
OrderItem item = em.find(OrderItem.class, id);
```

### Q40: What is the difference between get() and load() in Hibernate?

**get()** (JPA: find()):
- Hits database immediately
- Returns null if not found
- Returns actual object

**load()** (Hibernate-specific):
- Returns proxy
- Lazy loading
- Throws ObjectNotFoundException if not found

```java
// get/find - immediate query
User user = em.find(User.class, 1L);
if (user == null) {
    // Handle not found
}

// load - returns proxy, may throw exception later
Session session = em.unwrap(Session.class);
User proxy = session.load(User.class, 999L);  // No query yet
proxy.getName();  // ObjectNotFoundException thrown here!
```

---

## Summary

**Key Takeaways:**

1. **Use JPA for portability**, Hibernate for advanced features
2. **Default to LAZY fetching**, use JOIN FETCH when needed
3. **Watch out for N+1 queries** - use JOIN FETCH, batching, or DTOs
4. **Use appropriate inheritance strategy** - usually SINGLE_TABLE
5. **Enable Second Level Cache** for read-heavy entities
6. **Use optimistic locking** unless high contention
7. **Always use connection pooling** (HikariCP)
8. **Monitor SQL queries** and optimize slow ones
9. **Use pagination** for large result sets
10. **Prefer DTOs** for read-only queries

**Performance Checklist:**
- [ ] Connection pool configured
- [ ] Lazy loading by default
- [ ] Indexes on foreign keys and queried columns
- [ ] Second level cache for lookup tables
- [ ] Batch inserts/updates enabled
- [ ] No N+1 queries
- [ ] Pagination for large lists
- [ ] DTOs for reports
- [ ] Query monitoring enabled
- [ ] Slow query logging configured

---

*For more examples and basic setup, see [README.md](./README.md)*
