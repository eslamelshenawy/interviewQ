# أسئلة المقابلات الشخصية - Backend Developer

## جدول المحتويات
1. [المفاهيم الأساسية](#المفاهيم-الأساسية)
2. [Java & Spring Boot](#java--spring-boot)
3. [قواعد البيانات](#قواعد-البيانات)
4. [APIs](#apis)
5. [الأمان](#الأمان)
6. [الأداء والتوسع](#الأداء-والتوسع)
7. [Message Queues](#message-queues)
8. [الاختبارات](#الاختبارات)
9. [DevOps والأدوات](#devops-والأدوات)
10. [أسئلة عملية - Coding](#أسئلة-عملية---coding)

---

## المفاهيم الأساسية

### 1. ما هو Backend Development؟

**الإجابة:**
Backend Development هو الجزء من تطوير التطبيقات الذي يتعامل مع:
- **Server-side Logic**: معالجة البيانات والمنطق
- **Database**: تخزين واسترجاع البيانات
- **APIs**: التواصل مع Frontend
- **Authentication & Authorization**: الأمان
- **Business Logic**: القواعد الأساسية للتطبيق

**المكونات الرئيسية:**
```
Client (Frontend) → API (Backend) → Database
                    ↓
               Business Logic
                    ↓
            External Services
```

**مثال بسيط - Spring Boot:**
```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        User user = userService.findById(id);
        return ResponseEntity.ok(user);
    }

    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody User user) {
        User created = userService.create(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }
}
```

---

### 2. ما الفرق بين Monolithic و Microservices؟

**الإجابة:**

#### Monolithic Architecture
تطبيق واحد يحتوي على كل المكونات.

```
┌─────────────────────────────┐
│   Monolithic Application    │
│  ┌─────────────────────┐   │
│  │   User Module       │   │
│  ├─────────────────────┤   │
│  │   Order Module      │   │
│  ├─────────────────────┤   │
│  │   Payment Module    │   │
│  ├─────────────────────┤   │
│  │   Database          │   │
│  └─────────────────────┘   │
└─────────────────────────────┘
```

**المميزات:**
- ✅ بسيط في البداية
- ✅ سهل النشر
- ✅ Testing أسهل

**العيوب:**
- ❌ صعب التوسع
- ❌ تغيير بسيط يتطلب نشر كل التطبيق
- ❌ صعب الصيانة مع النمو

#### Microservices Architecture
تطبيقات صغيرة مستقلة تتواصل عبر APIs.

```
┌──────────┐  ┌──────────┐  ┌──────────┐
│  User    │  │  Order   │  │ Payment  │
│ Service  │  │ Service  │  │ Service  │
│    +     │  │    +     │  │    +     │
│   DB     │  │   DB     │  │   DB     │
└──────────┘  └──────────┘  └──────────┘
```

**المميزات:**
- ✅ توسع مستقل
- ✅ نشر مستقل
- ✅ تقنيات مختلفة لكل service

**العيوب:**
- ❌ معقد
- ❌ Distributed system challenges
- ❌ Testing أصعب

**متى تستخدم كل منهما:**
- **Monolithic**: تطبيقات صغيرة، فريق صغير، MVP
- **Microservices**: تطبيقات كبيرة، فرق متعددة، حاجة للتوسع

---

### 3. اشرح MVC Pattern

**الإجابة:**
MVC (Model-View-Controller) هو نمط معماري يفصل التطبيق إلى 3 مكونات.

```
Client Request
      ↓
┌──────────┐
│Controller│ ← يستقبل الطلب
└────┬─────┘
     ↓
┌──────────┐
│  Model   │ ← يتعامل مع البيانات
└────┬─────┘
     ↓
┌──────────┐
│   View   │ ← يعرض البيانات
└──────────┘
```

**مثال Spring MVC:**

```java
// Model
@Entity
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private double price;

    // Getters, Setters
}

// Controller
@Controller
@RequestMapping("/products")
public class ProductController {

    @Autowired
    private ProductService productService;

    @GetMapping
    public String listProducts(Model model) {
        List<Product> products = productService.findAll();
        model.addAttribute("products", products);
        return "product-list"; // View name
    }

    @GetMapping("/{id}")
    public String viewProduct(@PathVariable Long id, Model model) {
        Product product = productService.findById(id);
        model.addAttribute("product", product);
        return "product-detail";
    }
}

// Service (Business Logic)
@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    public List<Product> findAll() {
        return productRepository.findAll();
    }

    public Product findById(Long id) {
        return productRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Product not found"));
    }
}

// Repository (Data Access)
public interface ProductRepository extends JpaRepository<Product, Long> {
    List<Product> findByPriceGreaterThan(double price);
}
```

**الفوائد:**
- ✅ فصل المسؤوليات (Separation of Concerns)
- ✅ سهل الاختبار
- ✅ سهل الصيانة
- ✅ إعادة استخدام الكود

---

### 4. ما هو RESTful API؟

**الإجابة:**
REST (Representational State Transfer) هو architectural style لتصميم web services.

**المبادئ الأساسية:**

1. **Stateless**: كل طلب مستقل
2. **Client-Server**: فصل بين Client و Server
3. **Cacheable**: الردود يمكن تخزينها مؤقتاً
4. **Uniform Interface**: واجهة موحدة
5. **Layered System**: نظام طبقي

**RESTful API Design:**

```java
@RestController
@RequestMapping("/api/v1/products")
public class ProductRestController {

    @Autowired
    private ProductService productService;

    // GET /api/v1/products - Get all products
    @GetMapping
    public ResponseEntity<List<Product>> getAllProducts() {
        List<Product> products = productService.findAll();
        return ResponseEntity.ok(products);
    }

    // GET /api/v1/products/{id} - Get single product
    @GetMapping("/{id}")
    public ResponseEntity<Product> getProduct(@PathVariable Long id) {
        Product product = productService.findById(id);
        return ResponseEntity.ok(product);
    }

    // POST /api/v1/products - Create product
    @PostMapping
    public ResponseEntity<Product> createProduct(@Valid @RequestBody Product product) {
        Product created = productService.create(product);
        URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(created.getId())
            .toUri();
        return ResponseEntity.created(location).body(created);
    }

    // PUT /api/v1/products/{id} - Update product
    @PutMapping("/{id}")
    public ResponseEntity<Product> updateProduct(
            @PathVariable Long id,
            @Valid @RequestBody Product product) {
        Product updated = productService.update(id, product);
        return ResponseEntity.ok(updated);
    }

    // DELETE /api/v1/products/{id} - Delete product
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable Long id) {
        productService.delete(id);
        return ResponseEntity.noContent().build();
    }

    // PATCH /api/v1/products/{id} - Partial update
    @PatchMapping("/{id}")
    public ResponseEntity<Product> partialUpdate(
            @PathVariable Long id,
            @RequestBody Map<String, Object> updates) {
        Product updated = productService.partialUpdate(id, updates);
        return ResponseEntity.ok(updated);
    }
}
```

**Best Practices:**
- ✅ استخدم أسماء جمع للموارد (products, users)
- ✅ استخدم HTTP methods بشكل صحيح
- ✅ استخدم HTTP status codes المناسبة
- ✅ نسخة API (/api/v1/)
- ✅ استخدم filtering, sorting, pagination

```java
// Pagination & Sorting
@GetMapping
public ResponseEntity<Page<Product>> getProducts(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(defaultValue = "id") String sort) {

    Pageable pageable = PageRequest.of(page, size, Sort.by(sort));
    Page<Product> products = productService.findAll(pageable);
    return ResponseEntity.ok(products);
}

// Filtering
@GetMapping("/search")
public ResponseEntity<List<Product>> searchProducts(
        @RequestParam(required = false) String name,
        @RequestParam(required = false) Double minPrice,
        @RequestParam(required = false) Double maxPrice) {

    List<Product> products = productService.search(name, minPrice, maxPrice);
    return ResponseEntity.ok(products);
}
```

---

## Java & Spring Boot

### 5. ما هو Dependency Injection (DI)؟

**الإجابة:**
Dependency Injection هو design pattern حيث يتم توفير dependencies من الخارج بدلاً من إنشائها داخل الclass.

**بدون DI:**
```java
// ❌ Tight coupling
public class UserService {
    private UserRepository userRepository = new UserRepository(); // Hard-coded dependency

    public User findById(Long id) {
        return userRepository.findById(id);
    }
}
```

**مع DI:**
```java
// ✅ Loose coupling
@Service
public class UserService {

    private final UserRepository userRepository;

    // Constructor Injection (الأفضل)
    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User findById(Long id) {
        return userRepository.findById(id);
    }
}
```

**أنواع DI في Spring:**

#### 1. Constructor Injection (الموصى به)
```java
@Service
public class OrderService {
    private final OrderRepository orderRepository;
    private final EmailService emailService;

    @Autowired // اختياري في Spring 4.3+
    public OrderService(OrderRepository orderRepository, EmailService emailService) {
        this.orderRepository = orderRepository;
        this.emailService = emailService;
    }
}
```

#### 2. Setter Injection
```java
@Service
public class OrderService {
    private OrderRepository orderRepository;

    @Autowired
    public void setOrderRepository(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }
}
```

#### 3. Field Injection (غير موصى به)
```java
@Service
public class OrderService {
    @Autowired
    private OrderRepository orderRepository; // صعب الاختبار
}
```

**الفوائد:**
- ✅ Loose Coupling
- ✅ سهل الاختبار (Mock dependencies)
- ✅ إعادة استخدام الكود
- ✅ سهل الصيانة

---

### 6. اشرح Spring Boot Auto-Configuration

**الإجابة:**
Auto-Configuration يقوم Spring Boot بإعداد beans تلقائياً بناءً على dependencies الموجودة.

**مثال:**

```java
// 1. إضافة Dependency
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
```

**Spring Boot يقوم تلقائياً بـ:**
- ✅ إنشاء DataSource
- ✅ إنشاء EntityManagerFactory
- ✅ إنشاء TransactionManager
- ✅ إعداد JPA repositories

**application.yml:**
```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: secret
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```

**كيف يعمل:**

```java
@SpringBootApplication
// يحتوي على:
// @Configuration - يحدد أن هذا configuration class
// @EnableAutoConfiguration - يفعل auto-configuration
// @ComponentScan - يبحث عن components
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

**تخصيص Auto-Configuration:**

```java
// تعطيل auto-configuration معينة
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
public class Application {
}

// Custom configuration
@Configuration
public class DatabaseConfig {

    @Bean
    @Primary
    public DataSource customDataSource() {
        // Custom DataSource configuration
        return DataSourceBuilder.create()
            .url("jdbc:mysql://localhost:3306/custom")
            .username("custom_user")
            .password("custom_pass")
            .build();
    }
}
```

---

### 7. اشرح Spring Bean Scopes

**الإجابة:**

```java
// 1. Singleton (الافتراضي) - نسخة واحدة فقط
@Service
@Scope("singleton")
public class SingletonService {
    // نفس النسخة لجميع الطلبات
}

// 2. Prototype - نسخة جديدة في كل مرة
@Service
@Scope("prototype")
public class PrototypeService {
    // نسخة جديدة عند كل طلب
}

// 3. Request - نسخة واحدة لكل HTTP request
@Service
@Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RequestScopedService {
    // نسخة جديدة لكل HTTP request
}

// 4. Session - نسخة واحدة لكل HTTP session
@Service
@Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class SessionScopedService {
    // نسخة واحدة لكل user session
}

// 5. Application - نسخة واحدة لكل ServletContext
@Service
@Scope(value = WebApplicationContext.SCOPE_APPLICATION, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class ApplicationScopedService {
    // نسخة واحدة للتطبيق كله
}
```

**مثال عملي:**

```java
@RestController
@RequestMapping("/api/cart")
public class ShoppingCartController {

    @Autowired
    private ShoppingCartService cartService; // Session scope

    @PostMapping("/add")
    public ResponseEntity<Void> addToCart(@RequestBody CartItem item) {
        cartService.addItem(item);
        return ResponseEntity.ok().build();
    }

    @GetMapping
    public ResponseEntity<List<CartItem>> getCart() {
        return ResponseEntity.ok(cartService.getItems());
    }
}

@Service
@Scope(value = "session", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class ShoppingCartService {
    private List<CartItem> items = new ArrayList<>();

    public void addItem(CartItem item) {
        items.add(item);
    }

    public List<CartItem> getItems() {
        return items;
    }
}
```

---

### 8. كيف تتعامل مع Exceptions في Spring Boot؟

**الإجابة:**

#### 1. @ControllerAdvice (Global Exception Handler)

```java
// Custom Exceptions
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}

public class BadRequestException extends RuntimeException {
    public BadRequestException(String message) {
        super(message);
    }
}

// Error Response DTO
@Data
@AllArgsConstructor
public class ErrorResponse {
    private LocalDateTime timestamp;
    private int status;
    private String error;
    private String message;
    private String path;
}

// Global Exception Handler
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(
            ResourceNotFoundException ex,
            WebRequest request) {

        ErrorResponse error = new ErrorResponse(
            LocalDateTime.now(),
            HttpStatus.NOT_FOUND.value(),
            "Not Found",
            ex.getMessage(),
            request.getDescription(false).replace("uri=", "")
        );

        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }

    @ExceptionHandler(BadRequestException.class)
    public ResponseEntity<ErrorResponse> handleBadRequest(
            BadRequestException ex,
            WebRequest request) {

        ErrorResponse error = new ErrorResponse(
            LocalDateTime.now(),
            HttpStatus.BAD_REQUEST.value(),
            "Bad Request",
            ex.getMessage(),
            request.getDescription(false).replace("uri=", "")
        );

        return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, Object>> handleValidationErrors(
            MethodArgumentNotValidException ex) {

        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
            errors.put(error.getField(), error.getDefaultMessage())
        );

        Map<String, Object> response = new HashMap<>();
        response.put("timestamp", LocalDateTime.now());
        response.put("status", HttpStatus.BAD_REQUEST.value());
        response.put("errors", errors);

        return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGlobalException(
            Exception ex,
            WebRequest request) {

        ErrorResponse error = new ErrorResponse(
            LocalDateTime.now(),
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "Internal Server Error",
            ex.getMessage(),
            request.getDescription(false).replace("uri=", "")
        );

        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

#### 2. استخدام في Service

```java
@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    public Product findById(Long id) {
        return productRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException(
                "Product not found with id: " + id
            ));
    }

    public Product create(Product product) {
        if (product.getPrice() < 0) {
            throw new BadRequestException("Price cannot be negative");
        }
        return productRepository.save(product);
    }
}
```

---

### 9. ما هو Spring Data JPA؟

**الإجابة:**
Spring Data JPA يبسط التعامل مع قواعد البيانات بتوفير repositories جاهزة.

```java
// Entity
@Entity
@Table(name = "products")
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    private String description;

    @Column(nullable = false)
    private Double price;

    @Column(name = "stock_quantity")
    private Integer stockQuantity;

    @Enumerated(EnumType.STRING)
    private ProductStatus status;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "category_id")
    private Category category;

    @CreationTimestamp
    private LocalDateTime createdAt;

    @UpdateTimestamp
    private LocalDateTime updatedAt;

    // Getters, Setters
}

// Repository
public interface ProductRepository extends JpaRepository<Product, Long> {

    // Query Methods - Spring Data ينشئ الاستعلام تلقائياً
    List<Product> findByName(String name);

    List<Product> findByPriceGreaterThan(Double price);

    List<Product> findByPriceBetween(Double minPrice, Double maxPrice);

    List<Product> findByNameContainingIgnoreCase(String name);

    List<Product> findByCategoryId(Long categoryId);

    List<Product> findByStatus(ProductStatus status);

    // @Query - JPQL
    @Query("SELECT p FROM Product p WHERE p.price > :price AND p.stockQuantity > 0")
    List<Product> findAvailableProductsAbovePrice(@Param("price") Double price);

    // Native Query
    @Query(value = "SELECT * FROM products WHERE price BETWEEN :min AND :max",
           nativeQuery = true)
    List<Product> findProductsInPriceRange(
        @Param("min") Double min,
        @Param("max") Double max
    );

    // Modifying Query
    @Modifying
    @Query("UPDATE Product p SET p.price = p.price * :factor WHERE p.category.id = :categoryId")
    int updatePricesByCategoryId(
        @Param("categoryId") Long categoryId,
        @Param("factor") Double factor
    );

    // Custom method with Pageable
    Page<Product> findByCategory(Category category, Pageable pageable);

    // Projection
    @Query("SELECT p.name as name, p.price as price FROM Product p WHERE p.id = :id")
    ProductProjection findProductProjectionById(@Param("id") Long id);
}

// Projection Interface
interface ProductProjection {
    String getName();
    Double getPrice();
}
```

**Service Layer:**

```java
@Service
@Transactional
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    public Page<Product> findAll(int page, int size, String sortBy) {
        Pageable pageable = PageRequest.of(page, size, Sort.by(sortBy));
        return productRepository.findAll(pageable);
    }

    public Product findById(Long id) {
        return productRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Product not found"));
    }

    public Product create(Product product) {
        product.setStatus(ProductStatus.ACTIVE);
        return productRepository.save(product);
    }

    public Product update(Long id, Product productDetails) {
        Product product = findById(id);
        product.setName(productDetails.getName());
        product.setPrice(productDetails.getPrice());
        product.setStockQuantity(productDetails.getStockQuantity());
        return productRepository.save(product);
    }

    public void delete(Long id) {
        Product product = findById(id);
        productRepository.delete(product);
    }

    public List<Product> search(String keyword, Double minPrice, Double maxPrice) {
        if (keyword != null && minPrice != null && maxPrice != null) {
            return productRepository.findAll((root, query, cb) -> {
                return cb.and(
                    cb.like(cb.lower(root.get("name")), "%" + keyword.toLowerCase() + "%"),
                    cb.between(root.get("price"), minPrice, maxPrice)
                );
            });
        }
        return productRepository.findAll();
    }
}
```

---

## قواعد البيانات

### 10. ما هي ACID Properties؟

**الإجابة:**
ACID هي خصائص تضمن موثوقية المعاملات (Transactions) في قواعد البيانات.

#### 1. Atomicity (الذرية)
المعاملة تنجح كاملة أو تفشل كاملة.

```java
@Service
@Transactional
public class BankService {

    @Autowired
    private AccountRepository accountRepository;

    // ✅ Atomicity - كلاهما ينجح أو كلاهما يفشل
    public void transferMoney(Long fromId, Long toId, Double amount) {
        Account from = accountRepository.findById(fromId)
            .orElseThrow(() -> new ResourceNotFoundException("Account not found"));
        Account to = accountRepository.findById(toId)
            .orElseThrow(() -> new ResourceNotFoundException("Account not found"));

        // خصم من الحساب الأول
        from.setBalance(from.getBalance() - amount);
        accountRepository.save(from);

        // إضافة للحساب الثاني
        to.setBalance(to.getBalance() + amount);
        accountRepository.save(to);

        // إذا حدث خطأ في أي خطوة، كل شيء يُلغى
    }
}
```

#### 2. Consistency (الاتساق)
البيانات تنتقل من حالة صحيحة إلى حالة صحيحة.

```java
@Entity
public class Account {
    @Id
    private Long id;

    @Column(nullable = false)
    private Double balance;

    @PreUpdate
    public void validateBalance() {
        if (balance < 0) {
            throw new IllegalStateException("Balance cannot be negative");
        }
    }
}
```

#### 3. Isolation (العزل)
المعاملات المتزامنة لا تتداخل مع بعضها.

```java
@Transactional(isolation = Isolation.READ_COMMITTED)
public void updateAccount(Long id, Double amount) {
    // مستوى العزل يمنع قراءة بيانات غير مُثبتة
    Account account = accountRepository.findById(id).orElseThrow();
    account.setBalance(account.getBalance() + amount);
    accountRepository.save(account);
}
```

**Isolation Levels:**
```java
// 1. READ_UNCOMMITTED - أقل عزل (Dirty Read ممكن)
@Transactional(isolation = Isolation.READ_UNCOMMITTED)

// 2. READ_COMMITTED - يمنع Dirty Read
@Transactional(isolation = Isolation.READ_COMMITTED)

// 3. REPEATABLE_READ - يمنع Non-repeatable Read
@Transactional(isolation = Isolation.REPEATABLE_READ)

// 4. SERIALIZABLE - أعلى عزل (أبطأ)
@Transactional(isolation = Isolation.SERIALIZABLE)
```

#### 4. Durability (الاستمرارية)
البيانات المُثبتة تبقى حتى بعد إعادة تشغيل النظام.

```yaml
# application.yml
spring:
  jpa:
    properties:
      hibernate:
        jdbc:
          batch_size: 20
        order_inserts: true
        order_updates: true
```

---

### 11. ما هي Database Indexes ومتى تستخدمها؟

**الإجابة:**
Index هو data structure يحسن سرعة استرجاع البيانات.

```java
@Entity
@Table(name = "users", indexes = {
    @Index(name = "idx_email", columnList = "email"),
    @Index(name = "idx_username", columnList = "username"),
    @Index(name = "idx_created_at", columnList = "created_at")
})
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true, nullable = false)
    private String email;

    @Column(unique = true, nullable = false)
    private String username;

    @Column(name = "created_at")
    private LocalDateTime createdAt;
}

// Composite Index
@Table(indexes = {
    @Index(name = "idx_user_status", columnList = "user_id, status")
})
public class Order {
    @ManyToOne
    @JoinColumn(name = "user_id")
    private User user;

    @Enumerated(EnumType.STRING)
    private OrderStatus status;
}
```

**متى تستخدم Index:**
- ✅ أعمدة في WHERE clause
- ✅ أعمدة في JOIN
- ✅ أعمدة في ORDER BY
- ✅ Foreign Keys

**متى لا تستخدم Index:**
- ❌ جداول صغيرة
- ❌ أعمدة يُكتب عليها كثيراً
- ❌ أعمدة ذات قيم قليلة (مثل: gender)

---

### 12. اشرح N+1 Query Problem وحلها

**الإجابة:**
N+1 Problem يحدث عند تنفيذ استعلام إضافي لكل عنصر في النتيجة.

**المشكلة:**
```java
// ❌ N+1 Problem
@Entity
public class User {
    @Id
    private Long id;
    private String name;

    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
    private List<Order> orders;
}

// هذا الكود ينفذ 1 + N استعلام
List<User> users = userRepository.findAll(); // 1 استعلام
for (User user : users) {
    System.out.println(user.getOrders().size()); // N استعلام (استعلام لكل user)
}
```

**الحلول:**

#### 1. JOIN FETCH
```java
public interface UserRepository extends JpaRepository<User, Long> {

    @Query("SELECT u FROM User u JOIN FETCH u.orders")
    List<User> findAllWithOrders();
}

// الآن استعلام واحد فقط
List<User> users = userRepository.findAllWithOrders();
for (User user : users) {
    System.out.println(user.getOrders().size()); // لا استعلام إضافي
}
```

#### 2. @EntityGraph
```java
public interface UserRepository extends JpaRepository<User, Long> {

    @EntityGraph(attributePaths = {"orders"})
    List<User> findAll();
}
```

#### 3. Batch Fetching
```java
@Entity
public class User {
    @Id
    private Long id;

    @OneToMany(mappedBy = "user")
    @BatchSize(size = 10)
    private List<Order> orders;
}
```

---

### 13. ما الفرق بين SQL و NoSQL؟

**الإجابة:**

#### SQL (Relational Databases)

**الخصائص:**
- Schema محدد مسبقاً
- علاقات بين الجداول (Relationships)
- ACID compliance
- Vertical Scaling
- استعلامات معقدة (Joins)

**أمثلة:** MySQL, PostgreSQL, Oracle, SQL Server

```java
// SQL Example - Spring Data JPA
@Entity
@Table(name = "customers")
public class Customer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;

    @OneToMany(mappedBy = "customer")
    private List<Order> orders;
}

@Entity
@Table(name = "orders")
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "customer_id")
    private Customer customer;

    private Double totalAmount;
}
```

#### NoSQL (Non-Relational Databases)

**الأنواع:**
1. **Document Store** (MongoDB)
2. **Key-Value** (Redis)
3. **Column-Family** (Cassandra)
4. **Graph** (Neo4j)

**الخصائص:**
- Schema مرن
- Horizontal Scaling
- BASE (Basically Available, Soft state, Eventually consistent)
- أداء عالي للقراءة/الكتابة

```java
// NoSQL Example - MongoDB
@Document(collection = "customers")
public class Customer {
    @Id
    private String id;

    private String name;
    private String email;

    // Embedded Documents
    private List<Order> orders; // لا حاجة لـ JOIN

    @Data
    public static class Order {
        private String orderId;
        private Double totalAmount;
        private LocalDateTime orderDate;
        private List<OrderItem> items;
    }
}

// Repository
public interface CustomerRepository extends MongoRepository<Customer, String> {
    List<Customer> findByEmail(String email);

    @Query("{'orders.totalAmount': {$gt: ?0}}")
    List<Customer> findCustomersWithOrdersAbove(Double amount);
}
```

**متى تستخدم كل منهما:**

| **SQL** | **NoSQL** |
|---------|-----------|
| بيانات منظمة وعلاقات معقدة | بيانات غير منظمة أو شبه منظمة |
| ACID مهم | Scalability أهم من Consistency |
| استعلامات معقدة | قراءة/كتابة بسيطة وسريعة |
| Banking, E-commerce | Social Media, IoT, Real-time Analytics |

---

### 14. اشرح Database Transactions

**الإجابة:**

```java
@Service
@Transactional
public class OrderService {

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private ProductRepository productRepository;

    @Autowired
    private PaymentService paymentService;

    // Transaction يشمل كل العمليات
    public Order createOrder(OrderRequest request) {
        // 1. التحقق من المخزون
        Product product = productRepository.findById(request.getProductId())
            .orElseThrow(() -> new ResourceNotFoundException("Product not found"));

        if (product.getStockQuantity() < request.getQuantity()) {
            throw new BadRequestException("Insufficient stock");
        }

        // 2. خصم المخزون
        product.setStockQuantity(product.getStockQuantity() - request.getQuantity());
        productRepository.save(product);

        // 3. إنشاء الطلب
        Order order = new Order();
        order.setProduct(product);
        order.setQuantity(request.getQuantity());
        order.setTotalAmount(product.getPrice() * request.getQuantity());
        orderRepository.save(order);

        // 4. معالجة الدفع
        paymentService.processPayment(order.getTotalAmount());

        return order;
        // إذا فشلت أي خطوة، كل شيء يُلغى (Rollback)
    }

    // Transaction مع Propagation
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void logOrderCreation(Order order) {
        // هذا ينشئ transaction منفصلة
        // حتى لو فشلت transaction الرئيسية، هذا يُحفظ
    }

    // Read-only Transaction (تحسين الأداء)
    @Transactional(readOnly = true)
    public List<Order> findAllOrders() {
        return orderRepository.findAll();
    }

    // Transaction مع Timeout
    @Transactional(timeout = 30)
    public void longRunningOperation() {
        // إذا استغرق أكثر من 30 ثانية، rollback
    }
}
```

**Propagation Types:**

```java
// 1. REQUIRED (الافتراضي) - يستخدم transaction موجودة أو ينشئ جديدة
@Transactional(propagation = Propagation.REQUIRED)

// 2. REQUIRES_NEW - ينشئ transaction جديدة دائماً
@Transactional(propagation = Propagation.REQUIRES_NEW)

// 3. MANDATORY - يتطلب transaction موجودة
@Transactional(propagation = Propagation.MANDATORY)

// 4. NOT_SUPPORTED - يعلق transaction الحالية
@Transactional(propagation = Propagation.NOT_SUPPORTED)

// 5. NEVER - يرفض وجود transaction
@Transactional(propagation = Propagation.NEVER)

// 6. SUPPORTS - يستخدم transaction إن وجدت
@Transactional(propagation = Propagation.SUPPORTS)
```

---

## APIs

### 15. ما الفرق بين REST و GraphQL؟

**الإجابة:**

#### REST

```java
// REST - Multiple endpoints
@RestController
@RequestMapping("/api/v1")
public class RestApiController {

    // GET /api/v1/users/1
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);
    }

    // GET /api/v1/users/1/posts
    @GetMapping("/users/{id}/posts")
    public List<Post> getUserPosts(@PathVariable Long id) {
        return postService.findByUserId(id);
    }

    // GET /api/v1/posts/1/comments
    @GetMapping("/posts/{id}/comments")
    public List<Comment> getPostComments(@PathVariable Long id) {
        return commentService.findByPostId(id);
    }
}
```

#### GraphQL

```java
// GraphQL - Single endpoint
@Component
public class GraphQLResolver implements GraphQLQueryResolver {

    @Autowired
    private UserService userService;

    // Query - يسترجع فقط ما طلبه Client
    public User user(Long id) {
        return userService.findById(id);
    }
}

// GraphQL Schema
/*
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post]
}

type Post {
  id: ID!
  title: String!
  content: String!
  comments: [Comment]
}

type Query {
  user(id: ID!): User
}
*/

// Client Query - يحدد الحقول المطلوبة
/*
{
  user(id: 1) {
    name
    email
    posts {
      title
      comments {
        text
      }
    }
  }
}
*/
```

**المقارنة:**

| **REST** | **GraphQL** |
|----------|-------------|
| Multiple endpoints | Single endpoint |
| Over-fetching/Under-fetching | بيانات دقيقة فقط |
| Versioning (/v1, /v2) | Schema evolution |
| HTTP caching سهل | Caching معقد |
| أبسط للفهم | منحنى تعلم أعلى |

---

### 16. كيف تؤمن APIs؟

**الإجابة:**

#### 1. JWT Authentication

```java
// JWT Token Service
@Service
public class JwtService {

    @Value("${jwt.secret}")
    private String secret;

    @Value("${jwt.expiration}")
    private Long expiration;

    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("roles", userDetails.getAuthorities());

        return Jwts.builder()
            .setClaims(claims)
            .setSubject(userDetails.getUsername())
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + expiration))
            .signWith(SignatureAlgorithm.HS512, secret)
            .compact();
    }

    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }

    public boolean validateToken(String token, UserDetails userDetails) {
        String username = extractUsername(token);
        return username.equals(userDetails.getUsername()) && !isTokenExpired(token);
    }

    private boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
    }

    private Date extractExpiration(String token) {
        return extractClaim(token, Claims::getExpiration);
    }

    private <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {
        Claims claims = Jwts.parser()
            .setSigningKey(secret)
            .parseClaimsJws(token)
            .getBody();
        return claimsResolver.apply(claims);
    }
}

// JWT Filter
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    @Autowired
    private JwtService jwtService;

    @Autowired
    private UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain) throws ServletException, IOException {

        String authHeader = request.getHeader("Authorization");

        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }

        String token = authHeader.substring(7);
        String username = jwtService.extractUsername(token);

        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);

            if (jwtService.validateToken(token, userDetails)) {
                UsernamePasswordAuthenticationToken authToken =
                    new UsernamePasswordAuthenticationToken(
                        userDetails,
                        null,
                        userDetails.getAuthorities()
                    );

                authToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }

        filterChain.doFilter(request, response);
    }
}

// Security Configuration
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Autowired
    private JwtAuthenticationFilter jwtAuthFilter;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .authorizeHttpRequests()
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            .and()
            .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .and()
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}

// Auth Controller
@RestController
@RequestMapping("/api/auth")
public class AuthController {

    @Autowired
    private AuthenticationManager authenticationManager;

    @Autowired
    private JwtService jwtService;

    @Autowired
    private UserService userService;

    @PostMapping("/login")
    public ResponseEntity<AuthResponse> login(@RequestBody LoginRequest request) {
        authenticationManager.authenticate(
            new UsernamePasswordAuthenticationToken(
                request.getUsername(),
                request.getPassword()
            )
        );

        UserDetails user = userService.loadUserByUsername(request.getUsername());
        String token = jwtService.generateToken(user);

        return ResponseEntity.ok(new AuthResponse(token));
    }

    @PostMapping("/register")
    public ResponseEntity<User> register(@RequestBody RegisterRequest request) {
        User user = userService.register(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(user);
    }
}
```

#### 2. API Rate Limiting

```java
@Component
public class RateLimitingFilter extends OncePerRequestFilter {

    private final Map<String, List<Long>> requestCounts = new ConcurrentHashMap<>();
    private static final int MAX_REQUESTS = 100;
    private static final long TIME_WINDOW = 60000; // 1 minute

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain) throws ServletException, IOException {

        String clientId = getClientId(request);

        if (isRateLimited(clientId)) {
            response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
            response.getWriter().write("Rate limit exceeded");
            return;
        }

        filterChain.doFilter(request, response);
    }

    private boolean isRateLimited(String clientId) {
        long now = System.currentTimeMillis();
        requestCounts.putIfAbsent(clientId, new ArrayList<>());

        List<Long> timestamps = requestCounts.get(clientId);
        timestamps.removeIf(timestamp -> now - timestamp > TIME_WINDOW);

        if (timestamps.size() >= MAX_REQUESTS) {
            return true;
        }

        timestamps.add(now);
        return false;
    }

    private String getClientId(HttpServletRequest request) {
        String apiKey = request.getHeader("X-API-Key");
        return apiKey != null ? apiKey : request.getRemoteAddr();
    }
}
```

#### 3. Input Validation

```java
// DTO مع Validation
@Data
public class CreateUserRequest {

    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 50, message = "Name must be between 2 and 50 characters")
    private String name;

    @NotBlank(message = "Email is required")
    @Email(message = "Email must be valid")
    private String email;

    @NotBlank(message = "Password is required")
    @Size(min = 8, message = "Password must be at least 8 characters")
    @Pattern(
        regexp = "^(?=.*[A-Z])(?=.*[a-z])(?=.*\\d)(?=.*[@$!%*?&])[A-Za-z\\d@$!%*?&]{8,}$",
        message = "Password must contain uppercase, lowercase, digit and special character"
    )
    private String password;

    @Min(value = 18, message = "Age must be at least 18")
    @Max(value = 120, message = "Age must be less than 120")
    private Integer age;
}

@RestController
@RequestMapping("/api/users")
public class UserController {

    @PostMapping
    public ResponseEntity<User> createUser(@Valid @RequestBody CreateUserRequest request) {
        User user = userService.create(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(user);
    }
}
```

---

### 17. ما هو API Versioning؟

**الإجابة:**

```java
// 1. URI Versioning (الأكثر شيوعاً)
@RestController
@RequestMapping("/api/v1/products")
public class ProductV1Controller {

    @GetMapping("/{id}")
    public ProductV1 getProduct(@PathVariable Long id) {
        return productService.findByIdV1(id);
    }
}

@RestController
@RequestMapping("/api/v2/products")
public class ProductV2Controller {

    @GetMapping("/{id}")
    public ProductV2 getProduct(@PathVariable Long id) {
        // V2 مع حقول إضافية
        return productService.findByIdV2(id);
    }
}

// 2. Header Versioning
@RestController
@RequestMapping("/api/products")
public class ProductController {

    @GetMapping(value = "/{id}", headers = "API-Version=1")
    public ProductV1 getProductV1(@PathVariable Long id) {
        return productService.findByIdV1(id);
    }

    @GetMapping(value = "/{id}", headers = "API-Version=2")
    public ProductV2 getProductV2(@PathVariable Long id) {
        return productService.findByIdV2(id);
    }
}

// 3. Accept Header Versioning
@RestController
@RequestMapping("/api/products")
public class ProductController {

    @GetMapping(value = "/{id}", produces = "application/vnd.company.v1+json")
    public ProductV1 getProductV1(@PathVariable Long id) {
        return productService.findByIdV1(id);
    }

    @GetMapping(value = "/{id}", produces = "application/vnd.company.v2+json")
    public ProductV2 getProductV2(@PathVariable Long id) {
        return productService.findByIdV2(id);
    }
}

// 4. Query Parameter Versioning
@RestController
@RequestMapping("/api/products")
public class ProductController {

    @GetMapping(value = "/{id}", params = "version=1")
    public ProductV1 getProductV1(@PathVariable Long id) {
        return productService.findByIdV1(id);
    }

    @GetMapping(value = "/{id}", params = "version=2")
    public ProductV2 getProductV2(@PathVariable Long id) {
        return productService.findByIdV2(id);
    }
}
```

---

## الأمان

### 18. اشرح OAuth 2.0

**الإجابة:**

OAuth 2.0 هو بروتوكول للتخويل (Authorization) يسمح للتطبيقات بالوصول المحدود لموارد المستخدم.

**الأدوار:**
- **Resource Owner**: المستخدم
- **Client**: التطبيق
- **Authorization Server**: يصدر Tokens
- **Resource Server**: يحمي الموارد

**Grant Types:**

#### 1. Authorization Code Flow

```java
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private AuthenticationManager authenticationManager;

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
            .withClient("client-app")
            .secret("{noop}secret")
            .authorizedGrantTypes("authorization_code", "refresh_token")
            .scopes("read", "write")
            .redirectUris("http://localhost:8080/callback");
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
        endpoints.authenticationManager(authenticationManager);
    }
}

// Resource Server
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {

    @Override
    public void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
            .antMatchers("/api/public/**").permitAll()
            .antMatchers("/api/**").authenticated();
    }
}
```

#### 2. Client Credentials Flow

```java
// للتواصل بين Services
@Service
public class ExternalApiClient {

    @Value("${oauth.client.id}")
    private String clientId;

    @Value("${oauth.client.secret}")
    private String clientSecret;

    @Value("${oauth.token.url}")
    private String tokenUrl;

    public String getAccessToken() {
        RestTemplate restTemplate = new RestTemplate();

        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
        headers.setBasicAuth(clientId, clientSecret);

        MultiValueMap<String, String> body = new LinkedMultiValueMap<>();
        body.add("grant_type", "client_credentials");
        body.add("scope", "read write");

        HttpEntity<MultiValueMap<String, String>> request = new HttpEntity<>(body, headers);

        TokenResponse response = restTemplate.postForObject(
            tokenUrl,
            request,
            TokenResponse.class
        );

        return response.getAccessToken();
    }
}
```

---

### 19. ما هو CORS وكيف تتعامل معه؟

**الإجابة:**

CORS (Cross-Origin Resource Sharing) يتحكم في طلبات HTTP من domains مختلفة.

```java
@Configuration
public class CorsConfig {

    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**")
                    .allowedOrigins("http://localhost:3000", "https://example.com")
                    .allowedMethods("GET", "POST", "PUT", "DELETE", "PATCH")
                    .allowedHeaders("*")
                    .allowCredentials(true)
                    .maxAge(3600);
            }
        };
    }
}

// أو استخدام @CrossOrigin
@RestController
@RequestMapping("/api/products")
@CrossOrigin(
    origins = "http://localhost:3000",
    methods = {RequestMethod.GET, RequestMethod.POST},
    maxAge = 3600
)
public class ProductController {

    @GetMapping
    public List<Product> getAllProducts() {
        return productService.findAll();
    }
}

// Security Config مع CORS
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .cors().and()
            .csrf().disable()
            .authorizeHttpRequests()
                .anyRequest().authenticated();

        return http.build();
    }

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(Arrays.asList("http://localhost:3000"));
        configuration.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "DELETE"));
        configuration.setAllowedHeaders(Arrays.asList("*"));
        configuration.setAllowCredentials(true);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/api/**", configuration);
        return source;
    }
}
```

---

### 20. كيف تحمي من SQL Injection؟

**الإجابة:**

```java
// ❌ خطر - SQL Injection
@Repository
public class UserRepositoryBad {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    // ❌ NEVER DO THIS
    public User findByUsername(String username) {
        String sql = "SELECT * FROM users WHERE username = '" + username + "'";
        return jdbcTemplate.queryForObject(sql, new UserRowMapper());
        // Hacker: username = "' OR '1'='1"
        // Result: SELECT * FROM users WHERE username = '' OR '1'='1'
    }
}

// ✅ آمن - Prepared Statements
@Repository
public class UserRepositoryGood {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    // ✅ استخدم Prepared Statements
    public User findByUsername(String username) {
        String sql = "SELECT * FROM users WHERE username = ?";
        return jdbcTemplate.queryForObject(sql, new UserRowMapper(), username);
    }
}

// ✅ Spring Data JPA (آمن تلقائياً)
public interface UserRepository extends JpaRepository<User, Long> {

    // ✅ Query Methods - آمنة
    Optional<User> findByUsername(String username);

    // ✅ @Query مع Parameters - آمنة
    @Query("SELECT u FROM User u WHERE u.username = :username")
    Optional<User> findByUsernameCustom(@Param("username") String username);

    // ✅ Native Query مع Parameters - آمنة
    @Query(value = "SELECT * FROM users WHERE username = :username", nativeQuery = true)
    Optional<User> findByUsernameNative(@Param("username") String username);
}

// Input Validation
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping("/search")
    public ResponseEntity<List<User>> searchUsers(
            @RequestParam @Pattern(regexp = "^[a-zA-Z0-9_]+$") String username) {
        // Pattern يسمح فقط بأحرف وأرقام و underscore
        List<User> users = userService.findByUsername(username);
        return ResponseEntity.ok(users);
    }
}
```

---

## الأداء والتوسع

### 21. ما هو Caching وكيف تستخدمه؟

**الإجابة:**

```java
// 1. تفعيل Caching
@SpringBootApplication
@EnableCaching
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

// 2. Cache Configuration
@Configuration
public class CacheConfig {

    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager(
            "products", "users", "categories"
        );
        cacheManager.setCaffeine(Caffeine.newBuilder()
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .maximumSize(1000)
            .recordStats()
        );
        return cacheManager;
    }
}

// 3. استخدام Cache
@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    // تخزين النتيجة في Cache
    @Cacheable(value = "products", key = "#id")
    public Product findById(Long id) {
        System.out.println("Fetching from database...");
        return productRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Product not found"));
    }

    // تحديث Cache
    @CachePut(value = "products", key = "#product.id")
    public Product update(Product product) {
        return productRepository.save(product);
    }

    // حذف من Cache
    @CacheEvict(value = "products", key = "#id")
    public void delete(Long id) {
        productRepository.deleteById(id);
    }

    // حذف كل Cache
    @CacheEvict(value = "products", allEntries = true)
    public void deleteAll() {
        productRepository.deleteAll();
    }

    // Cache شرطي
    @Cacheable(value = "products", key = "#id", condition = "#id > 10")
    public Product findByIdConditional(Long id) {
        return productRepository.findById(id).orElse(null);
    }
}

// Redis Cache
@Configuration
public class RedisCacheConfig {

    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(10))
            .serializeKeysWith(
                RedisSerializationContext.SerializationPair.fromSerializer(
                    new StringRedisSerializer()
                )
            )
            .serializeValuesWith(
                RedisSerializationContext.SerializationPair.fromSerializer(
                    new GenericJackson2JsonRedisSerializer()
                )
            );

        return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(config)
            .build();
    }
}

// application.yml
/*
spring:
  cache:
    type: redis
  redis:
    host: localhost
    port: 6379
*/
```

---

### 22. اشرح Database Connection Pooling

**الإجابة:**

Connection Pooling يعيد استخدام اتصالات Database بدلاً من إنشاء جديدة في كل مرة.

```java
// HikariCP (الافتراضي في Spring Boot)
# application.yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: secret
    hikari:
      # الحد الأقصى للاتصالات
      maximum-pool-size: 10
      # الحد الأدنى للاتصالات الجاهزة
      minimum-idle: 5
      # Timeout للحصول على اتصال
      connection-timeout: 30000
      # Timeout لاتصال خامل
      idle-timeout: 600000
      # أقصى عمر لاتصال
      max-lifetime: 1800000
      # اسم Pool
      pool-name: MyAppHikariCP
      # Auto-commit
      auto-commit: true

// Custom Configuration
@Configuration
public class DatabaseConfig {

    @Bean
    public DataSource dataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:mysql://localhost:3306/mydb");
        config.setUsername("root");
        config.setPassword("secret");
        config.setMaximumPoolSize(20);
        config.setMinimumIdle(5);
        config.setConnectionTimeout(30000);
        config.setIdleTimeout(600000);
        config.setMaxLifetime(1800000);

        // Connection Test Query
        config.setConnectionTestQuery("SELECT 1");

        // Pool Optimization
        config.addDataSourceProperty("cachePrepStmts", "true");
        config.addDataSourceProperty("prepStmtCacheSize", "250");
        config.addDataSourceProperty("prepStmtCacheSqlLimit", "2048");

        return new HikariDataSource(config);
    }
}

// Monitoring
@Component
public class DataSourceMonitor {

    @Autowired
    private DataSource dataSource;

    @Scheduled(fixedRate = 60000) // كل دقيقة
    public void logPoolStats() {
        if (dataSource instanceof HikariDataSource) {
            HikariDataSource hikariDS = (HikariDataSource) dataSource;
            HikariPoolMXBean poolMXBean = hikariDS.getHikariPoolMXBean();

            System.out.println("Active Connections: " + poolMXBean.getActiveConnections());
            System.out.println("Idle Connections: " + poolMXBean.getIdleConnections());
            System.out.println("Total Connections: " + poolMXBean.getTotalConnections());
            System.out.println("Threads Awaiting: " + poolMXBean.getThreadsAwaitingConnection());
        }
    }
}
```

---

### 23. كيف تحسن أداء التطبيق؟

**الإجابة:**

#### 1. Database Optimization

```java
// ✅ استخدم Pagination
@GetMapping("/products")
public Page<Product> getProducts(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size) {

    return productRepository.findAll(PageRequest.of(page, size));
}

// ✅ استخدم Indexes
@Entity
@Table(indexes = {
    @Index(name = "idx_name", columnList = "name"),
    @Index(name = "idx_category", columnList = "category_id")
})
public class Product {
    // ...
}

// ✅ Lazy Loading
@OneToMany(fetch = FetchType.LAZY, mappedBy = "product")
private List<Review> reviews;

// ✅ Batch Operations
@Transactional
public void saveAll(List<Product> products) {
    productRepository.saveAll(products); // Batch insert
}
```

#### 2. Async Processing

```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}

@Service
public class EmailService {

    @Async
    public CompletableFuture<Void> sendEmail(String to, String subject, String body) {
        // عملية طويلة
        System.out.println("Sending email to: " + to);
        // ... send email
        return CompletableFuture.completedFuture(null);
    }
}

@Service
public class OrderService {

    @Autowired
    private EmailService emailService;

    public Order createOrder(OrderRequest request) {
        Order order = orderRepository.save(new Order(request));

        // إرسال Email بشكل غير متزامن
        emailService.sendEmail(
            order.getCustomerEmail(),
            "Order Confirmation",
            "Your order has been placed"
        );

        return order;
    }
}
```

#### 3. HTTP Compression

```yaml
# application.yml
server:
  compression:
    enabled: true
    mime-types:
      - application/json
      - application/xml
      - text/html
      - text/plain
    min-response-size: 1024
```

#### 4. Response Compression with Gzip

```java
@Configuration
public class WebConfig {

    @Bean
    public FilterRegistrationBean<GzipFilter> gzipFilter() {
        FilterRegistrationBean<GzipFilter> registration = new FilterRegistrationBean<>();
        registration.setFilter(new GzipFilter());
        registration.addUrlPatterns("/api/*");
        return registration;
    }
}
```

---

## Message Queues

### 24. اشرح RabbitMQ ومتى تستخدمه

**الإجابة:**

RabbitMQ هو message broker يستخدم لـ asynchronous communication بين services.

```java
// Configuration
@Configuration
public class RabbitMQConfig {

    public static final String QUEUE_NAME = "order.queue";
    public static final String EXCHANGE_NAME = "order.exchange";
    public static final String ROUTING_KEY = "order.routing.key";

    @Bean
    public Queue queue() {
        return new Queue(QUEUE_NAME, true); // durable = true
    }

    @Bean
    public TopicExchange exchange() {
        return new TopicExchange(EXCHANGE_NAME);
    }

    @Bean
    public Binding binding(Queue queue, TopicExchange exchange) {
        return BindingBuilder
            .bind(queue)
            .to(exchange)
            .with(ROUTING_KEY);
    }

    @Bean
    public MessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }
}

// Producer
@Service
public class OrderProducer {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void sendOrder(Order order) {
        rabbitTemplate.convertAndSend(
            RabbitMQConfig.EXCHANGE_NAME,
            RabbitMQConfig.ROUTING_KEY,
            order
        );
        System.out.println("Order sent: " + order.getId());
    }
}

// Consumer
@Component
public class OrderConsumer {

    @RabbitListener(queues = RabbitMQConfig.QUEUE_NAME)
    public void handleOrder(Order order) {
        System.out.println("Received order: " + order.getId());
        // معالجة الطلب
        processOrder(order);
    }

    private void processOrder(Order order) {
        // Business logic
    }
}

// Controller
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    @Autowired
    private OrderProducer orderProducer;

    @PostMapping
    public ResponseEntity<String> createOrder(@RequestBody Order order) {
        orderProducer.sendOrder(order);
        return ResponseEntity.accepted()
            .body("Order accepted and will be processed");
    }
}

// Dead Letter Queue (للفشل)
@Configuration
public class DeadLetterQueueConfig {

    @Bean
    public Queue deadLetterQueue() {
        return new Queue("order.dlq", true);
    }

    @Bean
    public Queue orderQueueWithDLQ() {
        return QueueBuilder.durable("order.queue")
            .withArgument("x-dead-letter-exchange", "")
            .withArgument("x-dead-letter-routing-key", "order.dlq")
            .build();
    }
}
```

**متى تستخدم RabbitMQ:**
- ✅ Decoupling services
- ✅ Asynchronous processing
- ✅ Load balancing
- ✅ Event-driven architecture
- ✅ Retry mechanism

---

## الاختبارات

### 25. اشرح أنواع الاختبارات في Spring Boot

**الإجابة:**

#### 1. Unit Tests

```java
@ExtendWith(MockitoExtension.class)
public class ProductServiceTest {

    @Mock
    private ProductRepository productRepository;

    @InjectMocks
    private ProductService productService;

    @Test
    void testFindById_Success() {
        // Arrange
        Long productId = 1L;
        Product product = new Product();
        product.setId(productId);
        product.setName("Test Product");

        when(productRepository.findById(productId)).thenReturn(Optional.of(product));

        // Act
        Product result = productService.findById(productId);

        // Assert
        assertNotNull(result);
        assertEquals("Test Product", result.getName());
        verify(productRepository, times(1)).findById(productId);
    }

    @Test
    void testFindById_NotFound() {
        // Arrange
        Long productId = 999L;
        when(productRepository.findById(productId)).thenReturn(Optional.empty());

        // Act & Assert
        assertThrows(ResourceNotFoundException.class, () -> {
            productService.findById(productId);
        });
    }

    @Test
    void testCreate_Success() {
        // Arrange
        Product product = new Product();
        product.setName("New Product");
        product.setPrice(99.99);

        Product savedProduct = new Product();
        savedProduct.setId(1L);
        savedProduct.setName("New Product");
        savedProduct.setPrice(99.99);

        when(productRepository.save(any(Product.class))).thenReturn(savedProduct);

        // Act
        Product result = productService.create(product);

        // Assert
        assertNotNull(result.getId());
        assertEquals("New Product", result.getName());
        verify(productRepository).save(product);
    }
}
```

#### 2. Integration Tests

```java
@SpringBootTest
@AutoConfigureMockMvc
@Transactional
public class ProductControllerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @Autowired
    private ProductRepository productRepository;

    @BeforeEach
    void setUp() {
        productRepository.deleteAll();
    }

    @Test
    void testGetProduct_Success() throws Exception {
        // Arrange
        Product product = new Product();
        product.setName("Test Product");
        product.setPrice(99.99);
        product = productRepository.save(product);

        // Act & Assert
        mockMvc.perform(get("/api/v1/products/" + product.getId()))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("Test Product"))
            .andExpect(jsonPath("$.price").value(99.99));
    }

    @Test
    void testCreateProduct_Success() throws Exception {
        // Arrange
        Product product = new Product();
        product.setName("New Product");
        product.setPrice(149.99);

        // Act & Assert
        mockMvc.perform(post("/api/v1/products")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(product)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.name").value("New Product"))
            .andExpect(jsonPath("$.price").value(149.99));

        // Verify
        assertEquals(1, productRepository.count());
    }

    @Test
    void testGetProduct_NotFound() throws Exception {
        mockMvc.perform(get("/api/v1/products/999"))
            .andExpect(status().isNotFound());
    }
}
```

#### 3. Repository Tests

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
public class ProductRepositoryTest {

    @Autowired
    private ProductRepository productRepository;

    @Autowired
    private TestEntityManager entityManager;

    @Test
    void testFindByName() {
        // Arrange
        Product product = new Product();
        product.setName("Test Product");
        product.setPrice(99.99);
        entityManager.persist(product);
        entityManager.flush();

        // Act
        List<Product> found = productRepository.findByName("Test Product");

        // Assert
        assertEquals(1, found.size());
        assertEquals("Test Product", found.get(0).getName());
    }

    @Test
    void testFindByPriceGreaterThan() {
        // Arrange
        Product p1 = new Product();
        p1.setName("Cheap");
        p1.setPrice(50.0);

        Product p2 = new Product();
        p2.setName("Expensive");
        p2.setPrice(150.0);

        entityManager.persist(p1);
        entityManager.persist(p2);
        entityManager.flush();

        // Act
        List<Product> found = productRepository.findByPriceGreaterThan(100.0);

        // Assert
        assertEquals(1, found.size());
        assertEquals("Expensive", found.get(0).getName());
    }
}
```

#### 4. Test Configuration

```yaml
# application-test.yml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
  h2:
    console:
      enabled: true
```

---

## DevOps والأدوات

### 26. كيف تستخدم Docker مع Spring Boot؟

**الإجابة:**

```dockerfile
# Dockerfile - Multi-stage build
FROM maven:3.8.4-openjdk-17 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

FROM openjdk:17-jdk-slim
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=prod
      - SPRING_DATASOURCE_URL=jdbc:mysql://db:3306/mydb
      - SPRING_DATASOURCE_USERNAME=root
      - SPRING_DATASOURCE_PASSWORD=secret
    depends_on:
      - db
      - redis
    networks:
      - app-network

  db:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=secret
      - MYSQL_DATABASE=mydb
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    networks:
      - app-network

volumes:
  mysql-data:

networks:
  app-network:
    driver: bridge
```

```bash
# Build and run
docker-compose up -d

# View logs
docker-compose logs -f app

# Stop
docker-compose down

# Remove volumes
docker-compose down -v
```

---

### 27. ما هي أفضل ممارسات Logging؟

**الإجابة:**

```java
// استخدام SLF4J مع Logback
@Service
@Slf4j // Lombok annotation
public class ProductService {

    public Product create(Product product) {
        log.info("Creating product: {}", product.getName());

        try {
            Product created = productRepository.save(product);
            log.info("Product created successfully with ID: {}", created.getId());
            return created;
        } catch (Exception e) {
            log.error("Error creating product: {}", product.getName(), e);
            throw e;
        }
    }

    public Product findById(Long id) {
        log.debug("Finding product with ID: {}", id);

        return productRepository.findById(id)
            .orElseThrow(() -> {
                log.warn("Product not found with ID: {}", id);
                return new ResourceNotFoundException("Product not found");
            });
    }
}

// Logging Configuration - logback-spring.xml
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <!-- Console Appender -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- File Appender -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/application.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/application-%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
            <totalSizeCap>3GB</totalSizeCap>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- Error File Appender -->
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/error.log</file>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/error-%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>90</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- Root Logger -->
    <root level="INFO">
        <appender-ref ref="CONSOLE" />
        <appender-ref ref="FILE" />
        <appender-ref ref="ERROR_FILE" />
    </root>

    <!-- Package Level Logging -->
    <logger name="com.example.myapp" level="DEBUG" />
    <logger name="org.springframework.web" level="INFO" />
    <logger name="org.hibernate" level="WARN" />

</configuration>
```

```yaml
# application.yml
logging:
  level:
    root: INFO
    com.example.myapp: DEBUG
    org.springframework.web: DEBUG
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"
    file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
  file:
    name: logs/application.log
```

---

## أسئلة عملية - Coding

### 28. اكتب REST API كامل لنظام E-Commerce

**الإجابة:**

```java
// 1. Entities
@Entity
@Table(name = "products")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    private String description;

    @Column(nullable = false)
    private Double price;

    private Integer stockQuantity;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "category_id")
    private Category category;

    @OneToMany(mappedBy = "product", cascade = CascadeType.ALL)
    private List<Review> reviews = new ArrayList<>();

    @CreationTimestamp
    private LocalDateTime createdAt;

    @UpdateTimestamp
    private LocalDateTime updatedAt;
}

@Entity
@Table(name = "orders")
@Data
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    private User user;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> items = new ArrayList<>();

    private Double totalAmount;

    @Enumerated(EnumType.STRING)
    private OrderStatus status;

    @CreationTimestamp
    private LocalDateTime createdAt;
}

@Entity
@Table(name = "order_items")
@Data
public class OrderItem {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "order_id")
    private Order order;

    @ManyToOne
    @JoinColumn(name = "product_id")
    private Product product;

    private Integer quantity;
    private Double price;
}

// 2. DTOs
@Data
public class ProductDTO {
    private Long id;
    private String name;
    private String description;
    private Double price;
    private Integer stockQuantity;
    private Long categoryId;
}

@Data
public class CreateOrderRequest {
    private Long userId;
    private List<OrderItemRequest> items;

    @Data
    public static class OrderItemRequest {
        private Long productId;
        private Integer quantity;
    }
}

// 3. Repositories
public interface ProductRepository extends JpaRepository<Product, Long> {
    List<Product> findByCategoryId(Long categoryId);
    List<Product> findByPriceBetween(Double min, Double max);
    Page<Product> findByNameContainingIgnoreCase(String name, Pageable pageable);
}

public interface OrderRepository extends JpaRepository<Order, Long> {
    List<Order> findByUserId(Long userId);
    List<Order> findByStatus(OrderStatus status);
}

// 4. Services
@Service
@Transactional
@Slf4j
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    public Page<Product> findAll(int page, int size, String sortBy) {
        return productRepository.findAll(
            PageRequest.of(page, size, Sort.by(sortBy))
        );
    }

    public Product findById(Long id) {
        return productRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Product not found"));
    }

    public Product create(ProductDTO dto) {
        Product product = new Product();
        product.setName(dto.getName());
        product.setDescription(dto.getDescription());
        product.setPrice(dto.getPrice());
        product.setStockQuantity(dto.getStockQuantity());

        return productRepository.save(product);
    }

    public Product update(Long id, ProductDTO dto) {
        Product product = findById(id);
        product.setName(dto.getName());
        product.setDescription(dto.getDescription());
        product.setPrice(dto.getPrice());
        product.setStockQuantity(dto.getStockQuantity());

        return productRepository.save(product);
    }

    public void delete(Long id) {
        Product product = findById(id);
        productRepository.delete(product);
    }
}

@Service
@Transactional
@Slf4j
public class OrderService {

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private ProductRepository productRepository;

    @Autowired
    private UserRepository userRepository;

    public Order createOrder(CreateOrderRequest request) {
        User user = userRepository.findById(request.getUserId())
            .orElseThrow(() -> new ResourceNotFoundException("User not found"));

        Order order = new Order();
        order.setUser(user);
        order.setStatus(OrderStatus.PENDING);

        double totalAmount = 0.0;

        for (CreateOrderRequest.OrderItemRequest itemReq : request.getItems()) {
            Product product = productRepository.findById(itemReq.getProductId())
                .orElseThrow(() -> new ResourceNotFoundException("Product not found"));

            if (product.getStockQuantity() < itemReq.getQuantity()) {
                throw new BadRequestException("Insufficient stock for: " + product.getName());
            }

            OrderItem orderItem = new OrderItem();
            orderItem.setOrder(order);
            orderItem.setProduct(product);
            orderItem.setQuantity(itemReq.getQuantity());
            orderItem.setPrice(product.getPrice());

            order.getItems().add(orderItem);
            totalAmount += product.getPrice() * itemReq.getQuantity();

            // Update stock
            product.setStockQuantity(product.getStockQuantity() - itemReq.getQuantity());
            productRepository.save(product);
        }

        order.setTotalAmount(totalAmount);
        return orderRepository.save(order);
    }

    public List<Order> findByUserId(Long userId) {
        return orderRepository.findByUserId(userId);
    }
}

// 5. Controllers
@RestController
@RequestMapping("/api/v1/products")
@Slf4j
public class ProductController {

    @Autowired
    private ProductService productService;

    @GetMapping
    public ResponseEntity<Page<Product>> getAllProducts(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size,
            @RequestParam(defaultValue = "id") String sort) {

        return ResponseEntity.ok(productService.findAll(page, size, sort));
    }

    @GetMapping("/{id}")
    public ResponseEntity<Product> getProduct(@PathVariable Long id) {
        return ResponseEntity.ok(productService.findById(id));
    }

    @PostMapping
    public ResponseEntity<Product> createProduct(@Valid @RequestBody ProductDTO dto) {
        Product created = productService.create(dto);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }

    @PutMapping("/{id}")
    public ResponseEntity<Product> updateProduct(
            @PathVariable Long id,
            @Valid @RequestBody ProductDTO dto) {

        return ResponseEntity.ok(productService.update(id, dto));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable Long id) {
        productService.delete(id);
        return ResponseEntity.noContent().build();
    }
}

@RestController
@RequestMapping("/api/v1/orders")
@Slf4j
public class OrderController {

    @Autowired
    private OrderService orderService;

    @PostMapping
    public ResponseEntity<Order> createOrder(@Valid @RequestBody CreateOrderRequest request) {
        Order order = orderService.createOrder(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(order);
    }

    @GetMapping("/user/{userId}")
    public ResponseEntity<List<Order>> getUserOrders(@PathVariable Long userId) {
        return ResponseEntity.ok(orderService.findByUserId(userId));
    }
}
```

---

## نصائح للمقابلات

### نصائح عامة:

1. **فهم السؤال جيداً** - اطلب التوضيح إذا لزم الأمر
2. **فكر بصوت عالٍ** - اشرح طريقة تفكيرك
3. **ابدأ بسيط** - ثم حسّن الحل تدريجياً
4. **اسأل عن المتطلبات** - Scale, Performance, Security
5. **اختبر كودك** - فكر في Edge Cases
6. **ناقش Trade-offs** - لكل حل مميزات وعيوب

### أسئلة لطرحها على المُقابل:

- ما هو tech stack المستخدم؟
- كيف تتعاملون مع Scaling؟
- ما هي عملية Deployment؟
- كيف تتعاملون مع Monitoring والأخطاء؟
- ما هو حجم الفريق وطريقة العمل؟

---

**حظاً موفقاً في مقابلتك! 🚀**
