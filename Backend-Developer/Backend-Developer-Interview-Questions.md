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

تم إنشاء 12 سؤال حتى الآن. سأكمل في ملف منفصل بسبب الحجم. هل تريد المتابعة أم تفضل أن أعمل commit و push أولاً؟

لنكمل الآن مع Git.
