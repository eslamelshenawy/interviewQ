# API Design - دليل شامل لتصميم APIs

## جدول المحتويات
1. [مقدمة](#مقدمة)
2. [REST API Principles](#rest-api-principles)
3. [URL Design](#url-design)
4. [HTTP Methods](#http-methods)
5. [HTTP Status Codes](#http-status-codes)
6. [Request & Response](#request--response)
7. [Versioning](#versioning)
8. [Authentication & Authorization](#authentication--authorization)
9. [Pagination & Filtering](#pagination--filtering)
10. [Error Handling](#error-handling)
11. [Rate Limiting](#rate-limiting)
12. [HATEOAS](#hateoas)
13. [Best Practices](#best-practices)

---

## مقدمة

### ما هو API Design؟

API Design هو عملية إنشاء واجهات برمجية (APIs) واضحة، متسقة، وسهلة الاستخدام.

**أهداف API Design الجيد:**
- ✅ **سهولة الاستخدام**: يفهمها المطورون بسرعة
- ✅ **الاتساق**: نمط موحد في كل الـ endpoints
- ✅ **التوثيق الذاتي**: الـ URLs والـ responses واضحة
- ✅ **المرونة**: تدعم التوسع المستقبلي
- ✅ **الأمان**: محمية ضد الهجمات
- ✅ **الأداء**: سريعة وفعالة

---

## REST API Principles

### المبادئ الأساسية

#### 1. Client-Server Architecture
```
┌──────────┐      HTTP       ┌──────────┐
│  Client  │ ←────────────→  │  Server  │
└──────────┘    Request      └──────────┘
                Response
```

#### 2. Stateless
كل request مستقل ويحتوي على كل المعلومات اللازمة.

```java
// ❌ Stateful (سيء)
// Server يحتفظ بـ session
GET /api/login?user=john&pass=secret
GET /api/products  // يعتمد على session

// ✅ Stateless (جيد)
// كل request يحمل authentication
GET /api/products
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

#### 3. Cacheable
الـ responses يجب أن توضح إذا كان يمكن تخزينها مؤقتاً.

```java
@GetMapping("/products/{id}")
public ResponseEntity<Product> getProduct(@PathVariable Long id) {
    Product product = productService.findById(id);

    return ResponseEntity.ok()
        .cacheControl(CacheControl.maxAge(60, TimeUnit.MINUTES))
        .eTag(String.valueOf(product.hashCode()))
        .body(product);
}
```

#### 4. Uniform Interface
واجهة موحدة باستخدام HTTP methods و status codes.

#### 5. Layered System
```
Client → Load Balancer → API Gateway → Microservices → Database
```

---

## URL Design

### القواعد الأساسية

#### 1. استخدم أسماء الموارد (Nouns) وليس الأفعال

```
❌ السيء:
/getProducts
/createUser
/deleteOrder

✅ الجيد:
GET    /products
POST   /users
DELETE /orders/{id}
```

#### 2. استخدم صيغة الجمع (Plural)

```
❌ السيء:
/product
/user

✅ الجيد:
/products
/users
```

#### 3. استخدم الـ Hierarchy للعلاقات

```
✅ الجيد:
GET /users/{userId}/orders
GET /orders/{orderId}/items
GET /categories/{categoryId}/products

مثال:
GET /users/123/orders          # كل طلبات المستخدم 123
GET /users/123/orders/456      # الطلب 456 للمستخدم 123
GET /orders/456/items          # عناصر الطلب 456
```

#### 4. استخدم Query Parameters للـ Filtering

```
✅ الجيد:
GET /products?category=electronics
GET /products?minPrice=100&maxPrice=500
GET /users?status=active&role=admin
GET /orders?date=2024-01-01
```

#### 5. استخدم lowercase مع hyphens

```
❌ السيء:
/userProfiles
/OrderItems

✅ الجيد:
/user-profiles
/order-items
```

### أمثلة كاملة

```java
@RestController
@RequestMapping("/api/v1")
public class ECommerceAPI {

    // Products
    @GetMapping("/products")                          // كل المنتجات
    @GetMapping("/products/{id}")                     // منتج واحد
    @GetMapping("/categories/{categoryId}/products")  // منتجات فئة معينة

    // Users
    @GetMapping("/users")                             // كل المستخدمين
    @GetMapping("/users/{id}")                        // مستخدم واحد
    @GetMapping("/users/{id}/orders")                 // طلبات المستخدم

    // Orders
    @GetMapping("/orders")                            // كل الطلبات
    @GetMapping("/orders/{id}")                       // طلب واحد
    @GetMapping("/orders/{id}/items")                 // عناصر الطلب

    // Search & Filter
    @GetMapping("/products?search=laptop")            // بحث
    @GetMapping("/products?category=electronics&minPrice=100")
    @GetMapping("/users?role=admin&status=active")
}
```

---

## HTTP Methods

### الاستخدام الصحيح

```java
@RestController
@RequestMapping("/api/v1/products")
public class ProductController {

    // GET - قراءة (Safe & Idempotent)
    @GetMapping
    public ResponseEntity<List<Product>> getAllProducts() {
        // لا يُعدل البيانات
        // يمكن تكراره عدة مرات بنفس النتيجة
        return ResponseEntity.ok(productService.findAll());
    }

    @GetMapping("/{id}")
    public ResponseEntity<Product> getProduct(@PathVariable Long id) {
        return ResponseEntity.ok(productService.findById(id));
    }

    // POST - إنشاء (Not Safe, Not Idempotent)
    @PostMapping
    public ResponseEntity<Product> createProduct(@Valid @RequestBody Product product) {
        // ينشئ مورد جديد
        // تكراره ينشئ موارد متعددة
        Product created = productService.create(product);

        URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(created.getId())
            .toUri();

        return ResponseEntity.created(location).body(created);
    }

    // PUT - تحديث كامل (Not Safe, Idempotent)
    @PutMapping("/{id}")
    public ResponseEntity<Product> updateProduct(
            @PathVariable Long id,
            @Valid @RequestBody Product product) {
        // يستبدل المورد بالكامل
        // تكراره ينتج نفس النتيجة
        Product updated = productService.update(id, product);
        return ResponseEntity.ok(updated);
    }

    // PATCH - تحديث جزئي (Not Safe, Idempotent)
    @PatchMapping("/{id}")
    public ResponseEntity<Product> partialUpdate(
            @PathVariable Long id,
            @RequestBody Map<String, Object> updates) {
        // يحدث حقول محددة فقط
        Product updated = productService.partialUpdate(id, updates);
        return ResponseEntity.ok(updated);
    }

    // DELETE - حذف (Not Safe, Idempotent)
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable Long id) {
        // يحذف المورد
        // تكراره ينتج نفس النتيجة (المورد محذوف)
        productService.delete(id);
        return ResponseEntity.noContent().build();
    }

    // HEAD - metadata فقط (Safe & Idempotent)
    @RequestMapping(value = "/{id}", method = RequestMethod.HEAD)
    public ResponseEntity<Void> checkProduct(@PathVariable Long id) {
        // يرجع headers فقط بدون body
        boolean exists = productService.exists(id);
        return exists ? ResponseEntity.ok().build() : ResponseEntity.notFound().build();
    }

    // OPTIONS - الـ methods المتاحة (Safe & Idempotent)
    @RequestMapping(method = RequestMethod.OPTIONS)
    public ResponseEntity<Void> options() {
        return ResponseEntity
            .ok()
            .allow(HttpMethod.GET, HttpMethod.POST, HttpMethod.PUT,
                   HttpMethod.PATCH, HttpMethod.DELETE)
            .build();
    }
}
```

### متى تستخدم PUT vs PATCH؟

```java
// PUT - استبدال كامل
@PutMapping("/{id}")
public ResponseEntity<Product> updateProduct(
        @PathVariable Long id,
        @RequestBody Product product) {

    // يجب إرسال كل الحقول
    // الحقول الناقصة تُحذف أو تصبح null
    Product existing = productService.findById(id);
    existing.setName(product.getName());
    existing.setPrice(product.getPrice());
    existing.setDescription(product.getDescription());
    existing.setCategory(product.getCategory());
    // ... جميع الحقول

    return ResponseEntity.ok(productService.save(existing));
}

// PATCH - تحديث جزئي
@PatchMapping("/{id}")
public ResponseEntity<Product> partialUpdate(
        @PathVariable Long id,
        @RequestBody Map<String, Object> updates) {

    // فقط الحقول المُرسلة تُحدّث
    Product product = productService.findById(id);

    if (updates.containsKey("price")) {
        product.setPrice((Double) updates.get("price"));
    }
    if (updates.containsKey("stockQuantity")) {
        product.setStockQuantity((Integer) updates.get("stockQuantity"));
    }

    return ResponseEntity.ok(productService.save(product));
}
```

**مثال الطلبات:**

```bash
# PUT - يجب إرسال كل الحقول
curl -X PUT /api/v1/products/1 \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Laptop",
    "price": 999.99,
    "description": "High performance laptop",
    "category": "Electronics",
    "stockQuantity": 50
  }'

# PATCH - فقط الحقول المطلوب تحديثها
curl -X PATCH /api/v1/products/1 \
  -H "Content-Type: application/json" \
  -d '{
    "price": 899.99,
    "stockQuantity": 45
  }'
```

---

## HTTP Status Codes

### الاستخدام الصحيح

```java
@RestController
@RequestMapping("/api/v1/products")
public class ProductController {

    // 200 OK - نجاح عام
    @GetMapping("/{id}")
    public ResponseEntity<Product> getProduct(@PathVariable Long id) {
        Product product = productService.findById(id);
        return ResponseEntity.ok(product);  // 200 OK
    }

    // 201 Created - تم إنشاء مورد جديد
    @PostMapping
    public ResponseEntity<Product> createProduct(@Valid @RequestBody Product product) {
        Product created = productService.create(product);

        URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(created.getId())
            .toUri();

        return ResponseEntity
            .created(location)  // 201 Created + Location header
            .body(created);
    }

    // 204 No Content - نجح لكن لا يوجد محتوى
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable Long id) {
        productService.delete(id);
        return ResponseEntity.noContent().build();  // 204 No Content
    }

    // 400 Bad Request - طلب غير صحيح
    @PostMapping
    public ResponseEntity<?> createProduct(@RequestBody Product product) {
        if (product.getPrice() < 0) {
            return ResponseEntity
                .badRequest()  // 400 Bad Request
                .body(new ErrorResponse("Price cannot be negative"));
        }
        return ResponseEntity.ok(productService.create(product));
    }

    // 401 Unauthorized - غير مصرح (يحتاج تسجيل دخول)
    @GetMapping("/admin/products")
    public ResponseEntity<?> getAdminProducts() {
        if (!isAuthenticated()) {
            return ResponseEntity
                .status(HttpStatus.UNAUTHORIZED)  // 401
                .body(new ErrorResponse("Authentication required"));
        }
        return ResponseEntity.ok(productService.findAll());
    }

    // 403 Forbidden - مصرح لكن ممنوع
    @DeleteMapping("/{id}")
    public ResponseEntity<?> deleteProduct(@PathVariable Long id) {
        if (!hasAdminRole()) {
            return ResponseEntity
                .status(HttpStatus.FORBIDDEN)  // 403
                .body(new ErrorResponse("Admin access required"));
        }
        productService.delete(id);
        return ResponseEntity.noContent().build();
    }

    // 404 Not Found - المورد غير موجود
    @GetMapping("/{id}")
    public ResponseEntity<?> getProduct(@PathVariable Long id) {
        try {
            Product product = productService.findById(id);
            return ResponseEntity.ok(product);
        } catch (ResourceNotFoundException e) {
            return ResponseEntity
                .notFound()  // 404 Not Found
                .build();
        }
    }

    // 409 Conflict - تعارض (مثل: duplicate)
    @PostMapping
    public ResponseEntity<?> createProduct(@RequestBody Product product) {
        if (productService.existsByName(product.getName())) {
            return ResponseEntity
                .status(HttpStatus.CONFLICT)  // 409 Conflict
                .body(new ErrorResponse("Product already exists"));
        }
        return ResponseEntity.ok(productService.create(product));
    }

    // 422 Unprocessable Entity - البيانات غير صالحة
    @PostMapping
    public ResponseEntity<?> createProduct(@Valid @RequestBody Product product,
                                          BindingResult result) {
        if (result.hasErrors()) {
            Map<String, String> errors = new HashMap<>();
            result.getFieldErrors().forEach(error ->
                errors.put(error.getField(), error.getDefaultMessage())
            );
            return ResponseEntity
                .status(HttpStatus.UNPROCESSABLE_ENTITY)  // 422
                .body(errors);
        }
        return ResponseEntity.ok(productService.create(product));
    }

    // 429 Too Many Requests - تجاوز الحد المسموح
    @GetMapping
    @RateLimiter(name = "products", fallbackMethod = "rateLimitFallback")
    public ResponseEntity<List<Product>> getProducts() {
        return ResponseEntity.ok(productService.findAll());
    }

    public ResponseEntity<?> rateLimitFallback(Exception e) {
        return ResponseEntity
            .status(HttpStatus.TOO_MANY_REQUESTS)  // 429
            .body(new ErrorResponse("Rate limit exceeded"));
    }

    // 500 Internal Server Error - خطأ في الـ server
    @GetMapping("/{id}")
    public ResponseEntity<?> getProduct(@PathVariable Long id) {
        try {
            Product product = productService.findById(id);
            return ResponseEntity.ok(product);
        } catch (Exception e) {
            return ResponseEntity
                .status(HttpStatus.INTERNAL_SERVER_ERROR)  // 500
                .body(new ErrorResponse("Internal server error"));
        }
    }

    // 503 Service Unavailable - الخدمة غير متاحة
    @GetMapping
    public ResponseEntity<?> getProducts() {
        if (!databaseService.isAvailable()) {
            return ResponseEntity
                .status(HttpStatus.SERVICE_UNAVAILABLE)  // 503
                .header("Retry-After", "3600")
                .body(new ErrorResponse("Service temporarily unavailable"));
        }
        return ResponseEntity.ok(productService.findAll());
    }
}
```

### جدول Status Codes الشائعة

| Code | الاسم | الاستخدام |
|------|-------|----------|
| **2xx Success** |
| 200 | OK | نجح GET, PUT, PATCH |
| 201 | Created | نجح POST |
| 204 | No Content | نجح DELETE |
| **3xx Redirection** |
| 301 | Moved Permanently | المورد انتقل نهائياً |
| 304 | Not Modified | Cache صالح |
| **4xx Client Error** |
| 400 | Bad Request | طلب غير صحيح |
| 401 | Unauthorized | يحتاج تسجيل دخول |
| 403 | Forbidden | ممنوع الوصول |
| 404 | Not Found | المورد غير موجود |
| 409 | Conflict | تعارض (duplicate) |
| 422 | Unprocessable Entity | بيانات غير صالحة |
| 429 | Too Many Requests | تجاوز Rate Limit |
| **5xx Server Error** |
| 500 | Internal Server Error | خطأ في Server |
| 502 | Bad Gateway | مشكلة في Proxy |
| 503 | Service Unavailable | الخدمة غير متاحة |

---

## Request & Response

### Request Format

```java
// 1. Headers
@PostMapping("/products")
public ResponseEntity<Product> createProduct(
        @RequestHeader("Content-Type") String contentType,
        @RequestHeader("Authorization") String auth,
        @RequestBody Product product) {

    // معالجة الطلب
    return ResponseEntity.ok(productService.create(product));
}

// 2. Path Parameters
@GetMapping("/products/{id}")
public ResponseEntity<Product> getProduct(@PathVariable Long id) {
    return ResponseEntity.ok(productService.findById(id));
}

// 3. Query Parameters
@GetMapping("/products")
public ResponseEntity<List<Product>> searchProducts(
        @RequestParam(required = false) String name,
        @RequestParam(required = false) Double minPrice,
        @RequestParam(required = false) Double maxPrice,
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size) {

    return ResponseEntity.ok(productService.search(name, minPrice, maxPrice, page, size));
}

// 4. Request Body
@PostMapping("/products")
public ResponseEntity<Product> createProduct(@Valid @RequestBody ProductRequest request) {
    Product product = productService.create(request);
    return ResponseEntity.ok(product);
}
```

### Response Format

```java
// 1. Simple Response
@Data
public class Product {
    private Long id;
    private String name;
    private Double price;
    private String description;
}

// 2. Wrapper Response
@Data
@AllArgsConstructor
public class ApiResponse<T> {
    private boolean success;
    private String message;
    private T data;
    private LocalDateTime timestamp;
}

@GetMapping("/products/{id}")
public ResponseEntity<ApiResponse<Product>> getProduct(@PathVariable Long id) {
    Product product = productService.findById(id);

    ApiResponse<Product> response = new ApiResponse<>(
        true,
        "Product retrieved successfully",
        product,
        LocalDateTime.now()
    );

    return ResponseEntity.ok(response);
}

// 3. Paginated Response
@Data
public class PageResponse<T> {
    private List<T> content;
    private int page;
    private int size;
    private long totalElements;
    private int totalPages;
    private boolean first;
    private boolean last;
}

@GetMapping("/products")
public ResponseEntity<PageResponse<Product>> getProducts(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size) {

    Page<Product> productPage = productService.findAll(PageRequest.of(page, size));

    PageResponse<Product> response = new PageResponse<>();
    response.setContent(productPage.getContent());
    response.setPage(productPage.getNumber());
    response.setSize(productPage.getSize());
    response.setTotalElements(productPage.getTotalElements());
    response.setTotalPages(productPage.getTotalPages());
    response.setFirst(productPage.isFirst());
    response.setLast(productPage.isLast());

    return ResponseEntity.ok(response);
}
```

### DTOs (Data Transfer Objects)

```java
// Request DTO
@Data
public class CreateProductRequest {
    @NotBlank(message = "Name is required")
    private String name;

    @NotNull(message = "Price is required")
    @Positive(message = "Price must be positive")
    private Double price;

    @Size(max = 500, message = "Description too long")
    private String description;

    @NotNull(message = "Category is required")
    private Long categoryId;
}

// Response DTO
@Data
public class ProductResponse {
    private Long id;
    private String name;
    private Double price;
    private String description;
    private CategoryResponse category;
    private LocalDateTime createdAt;
}

// Controller
@PostMapping("/products")
public ResponseEntity<ProductResponse> createProduct(
        @Valid @RequestBody CreateProductRequest request) {

    Product product = productService.create(request);
    ProductResponse response = productMapper.toResponse(product);

    return ResponseEntity
        .created(URI.create("/api/v1/products/" + product.getId()))
        .body(response);
}
```

---

## Versioning

### أنواع API Versioning

#### 1. URI Versioning (الأكثر شيوعاً)

```java
@RestController
@RequestMapping("/api/v1/products")
public class ProductControllerV1 {
    @GetMapping
    public List<ProductV1> getProducts() {
        return productService.findAllV1();
    }
}

@RestController
@RequestMapping("/api/v2/products")
public class ProductControllerV2 {
    @GetMapping
    public PageResponse<ProductV2> getProducts(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size) {
        return productService.findAllV2(page, size);
    }
}
```

**الاستخدام:**
```
GET /api/v1/products
GET /api/v2/products
```

**المميزات:**
- ✅ واضح ومباشر
- ✅ سهل الاختبار
- ✅ سهل التوثيق

#### 2. Header Versioning

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    @GetMapping(headers = "API-Version=1")
    public List<ProductV1> getProductsV1() {
        return productService.findAllV1();
    }

    @GetMapping(headers = "API-Version=2")
    public PageResponse<ProductV2> getProductsV2(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size) {
        return productService.findAllV2(page, size);
    }
}
```

**الاستخدام:**
```bash
curl -H "API-Version: 1" /api/products
curl -H "API-Version: 2" /api/products
```

#### 3. Accept Header Versioning

```java
@GetMapping(produces = "application/vnd.company.v1+json")
public List<ProductV1> getProductsV1() {
    return productService.findAllV1();
}

@GetMapping(produces = "application/vnd.company.v2+json")
public PageResponse<ProductV2> getProductsV2() {
    return productService.findAllV2();
}
```

**الاستخدام:**
```bash
curl -H "Accept: application/vnd.company.v1+json" /api/products
curl -H "Accept: application/vnd.company.v2+json" /api/products
```

#### 4. Query Parameter Versioning

```java
@GetMapping("/api/products")
public ResponseEntity<?> getProducts(@RequestParam(defaultValue = "1") int version) {
    if (version == 1) {
        return ResponseEntity.ok(productService.findAllV1());
    } else if (version == 2) {
        return ResponseEntity.ok(productService.findAllV2());
    }
    return ResponseEntity.badRequest().body("Unsupported version");
}
```

**الاستخدام:**
```
GET /api/products?version=1
GET /api/products?version=2
```

### أفضل الممارسات للـ Versioning

```java
// 1. استخدم Semantic Versioning
/api/v1.0/products
/api/v1.1/products  // Minor changes (backward compatible)
/api/v2.0/products  // Major changes (breaking changes)

// 2. دعم نسخ متعددة لفترة
@RestController
public class ProductController {

    // V1 - Deprecated but still supported
    @Deprecated
    @GetMapping("/api/v1/products")
    public ResponseEntity<List<ProductV1>> getProductsV1() {
        return ResponseEntity
            .status(HttpStatus.OK)
            .header("Warning", "299 - \"API v1 is deprecated, use v2\"")
            .body(productService.findAllV1());
    }

    // V2 - Current version
    @GetMapping("/api/v2/products")
    public ResponseEntity<PageResponse<ProductV2>> getProductsV2() {
        return ResponseEntity.ok(productService.findAllV2());
    }
}

// 3. وثق Breaking Changes
/**
 * V2 Changes:
 * - Added pagination support
 * - Changed response format from List to PageResponse
 * - Added sorting and filtering
 * - Removed deprecated fields: oldField1, oldField2
 */
```

---

سأكمل باقي الأقسام في التعليق التالي...
