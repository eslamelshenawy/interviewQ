# أسئلة المقابلات الشخصية - API Design

## جدول المحتويات
1. [أسئلة أساسية](#أسئلة-أساسية)
2. [REST API Design](#rest-api-design)
3. [Authentication & Security](#authentication--security)
4. [Performance & Scalability](#performance--scalability)
5. [Error Handling](#error-handling)
6. [Versioning](#versioning)
7. [أسئلة عملية - Design](#أسئلة-عملية---design)

---

## أسئلة أساسية

### 1. ما الفرق بين REST و SOAP؟

**الإجابة:**

| المقارنة | REST | SOAP |
|----------|------|------|
| **النوع** | Architectural Style | Protocol |
| **التنسيق** | JSON, XML, HTML | XML فقط |
| **البروتوكول** | HTTP | HTTP, SMTP, TCP |
| **الأداء** | أسرع وأخف | أثقل |
| **State** | Stateless | Stateful أو Stateless |
| **Caching** | يدعم | لا يدعم |
| **الأمان** | HTTPS, OAuth | WS-Security |
| **الاستخدام** | Web APIs, Mobile | Enterprise, Banking |

**مثال REST:**
```java
@RestController
@RequestMapping("/api/v1/products")
public class ProductRestController {

    @GetMapping("/{id}")
    public ResponseEntity<Product> getProduct(@PathVariable Long id) {
        Product product = productService.findById(id);
        return ResponseEntity.ok(product);
    }

    @PostMapping
    public ResponseEntity<Product> createProduct(@RequestBody Product product) {
        Product created = productService.create(product);
        return ResponseEntity.created(location).body(created);
    }
}
```

**Request/Response:**
```http
GET /api/v1/products/1 HTTP/1.1
Host: api.example.com
Accept: application/json

HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 1,
  "name": "Laptop",
  "price": 999.99
}
```

**مثال SOAP:**
```xml
<!-- Request -->
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <GetProduct xmlns="http://example.com/">
      <ProductId>1</ProductId>
    </GetProduct>
  </soap:Body>
</soap:Envelope>

<!-- Response -->
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <GetProductResponse>
      <Product>
        <Id>1</Id>
        <Name>Laptop</Name>
        <Price>999.99</Price>
      </Product>
    </GetProductResponse>
  </soap:Body>
</soap:Envelope>
```

---

### 2. ما هي REST Constraints؟

**الإجابة:**
REST يعتمد على 6 قيود (constraints):

#### 1. Client-Server
فصل بين Client و Server.

```java
// Client يطلب
GET /api/products

// Server يستجيب
{
  "products": [...]
}
```

#### 2. Stateless
كل request مستقل.

```java
// ❌ Stateful
// Request 1: login
// Request 2: get products (يعتمد على session)

// ✅ Stateless
GET /api/products
Authorization: Bearer token123
```

#### 3. Cacheable
Responses يمكن تخزينها مؤقتاً.

```java
@GetMapping("/products/{id}")
public ResponseEntity<Product> getProduct(@PathVariable Long id) {
    return ResponseEntity.ok()
        .cacheControl(CacheControl.maxAge(60, TimeUnit.MINUTES))
        .body(product);
}
```

#### 4. Uniform Interface
واجهة موحدة.

```
GET    /api/products      - Get all
GET    /api/products/1    - Get one
POST   /api/products      - Create
PUT    /api/products/1    - Update
DELETE /api/products/1    - Delete
```

#### 5. Layered System
طبقات متعددة.

```
Client → CDN → Load Balancer → API Gateway → Microservice → Database
```

#### 6. Code on Demand (اختياري)
Server يمكن أن يرسل كود للـ client.

```javascript
// Server يرسل JavaScript
{
  "script": "function calculate() { ... }"
}
```

---

### 3. متى تستخدم PUT vs PATCH vs POST؟

**الإجابة:**

```java
@RestController
@RequestMapping("/api/v1/products")
public class ProductController {

    // POST - إنشاء مورد جديد
    // ❌ ليس Idempotent (تكراره ينشئ موارد متعددة)
    @PostMapping
    public ResponseEntity<Product> createProduct(@RequestBody Product product) {
        Product created = productService.create(product);
        // Status: 201 Created
        return ResponseEntity.created(location).body(created);
    }

    // PUT - استبدال كامل
    // ✅ Idempotent (تكراره ينتج نفس النتيجة)
    @PutMapping("/{id}")
    public ResponseEntity<Product> replaceProduct(
            @PathVariable Long id,
            @RequestBody Product product) {

        // يستبدل المورد بالكامل
        // يجب إرسال جميع الحقول
        Product updated = productService.replace(id, product);
        // Status: 200 OK
        return ResponseEntity.ok(updated);
    }

    // PATCH - تحديث جزئي
    // ✅ Idempotent
    @PatchMapping("/{id}")
    public ResponseEntity<Product> updateProduct(
            @PathVariable Long id,
            @RequestBody Map<String, Object> updates) {

        // يحدث حقول محددة فقط
        Product updated = productService.partialUpdate(id, updates);
        // Status: 200 OK
        return ResponseEntity.ok(updated);
    }
}
```

**أمثلة الطلبات:**

```bash
# POST - إنشاء منتج جديد
POST /api/v1/products
{
  "name": "Laptop",
  "price": 999.99,
  "category": "Electronics"
}
# Response: 201 Created + Location header

# PUT - استبدال كامل (يجب إرسال كل الحقول)
PUT /api/v1/products/1
{
  "name": "Laptop Pro",
  "price": 1299.99,
  "description": "Updated description",
  "category": "Electronics",
  "stockQuantity": 50
}
# Response: 200 OK

# PATCH - تحديث جزئي (فقط الحقول المطلوبة)
PATCH /api/v1/products/1
{
  "price": 899.99,
  "stockQuantity": 45
}
# Response: 200 OK
```

**متى تستخدم كل منها:**

| Method | الاستخدام | Idempotent |
|--------|----------|------------|
| **POST** | إنشاء مورد جديد | ❌ |
| **PUT** | استبدال كامل | ✅ |
| **PATCH** | تحديث جزئي | ✅ |

---

### 4. كيف تصمم API للـ Pagination؟

**الإجابة:**

```java
// 1. Request Parameters
@GetMapping("/products")
public ResponseEntity<PageResponse<Product>> getProducts(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(defaultValue = "id") String sort,
        @RequestParam(defaultValue = "ASC") String direction) {

    Sort.Direction dir = Sort.Direction.fromString(direction);
    Pageable pageable = PageRequest.of(page, size, Sort.by(dir, sort));

    Page<Product> productPage = productService.findAll(pageable);

    PageResponse<Product> response = buildPageResponse(productPage);

    return ResponseEntity.ok(response);
}

// 2. Response DTO
@Data
public class PageResponse<T> {
    private List<T> content;
    private PageMetadata page;
    private Links links;

    @Data
    public static class PageMetadata {
        private int number;          // الصفحة الحالية
        private int size;            // حجم الصفحة
        private long totalElements;  // إجمالي العناصر
        private int totalPages;      // إجمالي الصفحات
        private boolean first;       // هل أول صفحة
        private boolean last;        // هل آخر صفحة
    }

    @Data
    public static class Links {
        private String self;
        private String first;
        private String last;
        private String next;
        private String prev;
    }
}

// 3. Build Response
private PageResponse<Product> buildPageResponse(Page<Product> page) {
    PageResponse<Product> response = new PageResponse<>();

    // Content
    response.setContent(page.getContent());

    // Metadata
    PageResponse.PageMetadata metadata = new PageResponse.PageMetadata();
    metadata.setNumber(page.getNumber());
    metadata.setSize(page.getSize());
    metadata.setTotalElements(page.getTotalElements());
    metadata.setTotalPages(page.getTotalPages());
    metadata.setFirst(page.isFirst());
    metadata.setLast(page.isLast());
    response.setPage(metadata);

    // Links (HATEOAS)
    PageResponse.Links links = new PageResponse.Links();
    String baseUrl = "/api/v1/products?page=%d&size=%d&sort=%s";

    links.setSelf(String.format(baseUrl, page.getNumber(), page.getSize(), "id"));
    links.setFirst(String.format(baseUrl, 0, page.getSize(), "id"));
    links.setLast(String.format(baseUrl, page.getTotalPages() - 1, page.getSize(), "id"));

    if (!page.isFirst()) {
        links.setPrev(String.format(baseUrl, page.getNumber() - 1, page.getSize(), "id"));
    }
    if (!page.isLast()) {
        links.setNext(String.format(baseUrl, page.getNumber() + 1, page.getSize(), "id"));
    }

    response.setLinks(links);

    return response;
}
```

**مثال Response:**

```json
{
  "content": [
    {
      "id": 1,
      "name": "Laptop",
      "price": 999.99
    },
    {
      "id": 2,
      "name": "Mouse",
      "price": 29.99
    }
  ],
  "page": {
    "number": 0,
    "size": 10,
    "totalElements": 100,
    "totalPages": 10,
    "first": true,
    "last": false
  },
  "links": {
    "self": "/api/v1/products?page=0&size=10&sort=id",
    "first": "/api/v1/products?page=0&size=10&sort=id",
    "last": "/api/v1/products?page=9&size=10&sort=id",
    "next": "/api/v1/products?page=1&size=10&sort=id"
  }
}
```

**أنواع Pagination:**

#### 1. Offset-Based Pagination
```
GET /api/products?page=0&size=10
GET /api/products?offset=0&limit=10
```

**المميزات:**
- ✅ بسيط
- ✅ يدعم القفز لصفحة معينة

**العيوب:**
- ❌ بطيء مع قواعد بيانات كبيرة
- ❌ مشاكل مع البيانات المتغيرة

#### 2. Cursor-Based Pagination
```java
@GetMapping("/products")
public ResponseEntity<CursorPageResponse<Product>> getProducts(
        @RequestParam(required = false) String cursor,
        @RequestParam(defaultValue = "10") int size) {

    List<Product> products;
    String nextCursor = null;

    if (cursor == null) {
        products = productService.findFirst(size + 1);
    } else {
        Long lastId = decodeCursor(cursor);
        products = productService.findAfter(lastId, size + 1);
    }

    boolean hasNext = products.size() > size;
    if (hasNext) {
        products = products.subList(0, size);
        nextCursor = encodeCursor(products.get(products.size() - 1).getId());
    }

    CursorPageResponse<Product> response = new CursorPageResponse<>();
    response.setData(products);
    response.setNextCursor(nextCursor);
    response.setHasNext(hasNext);

    return ResponseEntity.ok(response);
}
```

**الاستخدام:**
```
GET /api/products?size=10
GET /api/products?cursor=eyJpZCI6MTB9&size=10
```

**المميزات:**
- ✅ أداء ممتاز
- ✅ لا مشاكل مع البيانات المتغيرة

**العيوب:**
- ❌ لا يمكن القفز لصفحة معينة

---

### 5. كيف تصمم API للـ Filtering & Sorting؟

**الإجابة:**

```java
@RestController
@RequestMapping("/api/v1/products")
public class ProductController {

    // 1. Simple Filtering
    @GetMapping
    public ResponseEntity<List<Product>> getProducts(
            @RequestParam(required = false) String category,
            @RequestParam(required = false) Double minPrice,
            @RequestParam(required = false) Double maxPrice,
            @RequestParam(required = false) String status) {

        List<Product> products = productService.search(category, minPrice, maxPrice, status);
        return ResponseEntity.ok(products);
    }

    // 2. Advanced Filtering with Specification
    @GetMapping("/search")
    public ResponseEntity<Page<Product>> searchProducts(
            @RequestParam(required = false) String name,
            @RequestParam(required = false) String category,
            @RequestParam(required = false) Double minPrice,
            @RequestParam(required = false) Double maxPrice,
            @RequestParam(required = false) List<String> tags,
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(defaultValue = "id") String sortBy,
            @RequestParam(defaultValue = "ASC") String sortDir) {

        // Build Specification
        Specification<Product> spec = Specification.where(null);

        if (name != null) {
            spec = spec.and((root, query, cb) ->
                cb.like(cb.lower(root.get("name")), "%" + name.toLowerCase() + "%"));
        }

        if (category != null) {
            spec = spec.and((root, query, cb) ->
                cb.equal(root.get("category").get("name"), category));
        }

        if (minPrice != null) {
            spec = spec.and((root, query, cb) ->
                cb.greaterThanOrEqualTo(root.get("price"), minPrice));
        }

        if (maxPrice != null) {
            spec = spec.and((root, query, cb) ->
                cb.lessThanOrEqualTo(root.get("price"), maxPrice));
        }

        if (tags != null && !tags.isEmpty()) {
            spec = spec.and((root, query, cb) ->
                root.join("tags").get("name").in(tags));
        }

        // Sorting
        Sort sort = Sort.by(Sort.Direction.fromString(sortDir), sortBy);
        Pageable pageable = PageRequest.of(page, size, sort);

        Page<Product> products = productRepository.findAll(spec, pageable);

        return ResponseEntity.ok(products);
    }

    // 3. Query String Examples
    /**
     * GET /api/v1/products?category=electronics
     * GET /api/v1/products?minPrice=100&maxPrice=500
     * GET /api/v1/products?name=laptop&category=electronics&minPrice=500
     * GET /api/v1/products?tags=new,featured
     * GET /api/v1/products?sortBy=price&sortDir=DESC
     * GET /api/v1/products?name=laptop&sortBy=price&sortDir=ASC&page=0&size=10
     */
}

// 4. Advanced Sorting (Multiple fields)
@GetMapping("/advanced-sort")
public ResponseEntity<List<Product>> getProductsWithMultiSort(
        @RequestParam(required = false) String[] sort) {

    List<Sort.Order> orders = new ArrayList<>();

    if (sort != null) {
        for (String sortParam : sort) {
            String[] parts = sortParam.split(",");
            String field = parts[0];
            Sort.Direction direction = parts.length > 1 ?
                Sort.Direction.fromString(parts[1]) : Sort.Direction.ASC;
            orders.add(new Sort.Order(direction, field));
        }
    }

    Sort sorting = Sort.by(orders);
    List<Product> products = productRepository.findAll(sorting);

    return ResponseEntity.ok(products);
}

/**
 * Usage:
 * GET /api/v1/products/advanced-sort?sort=category,asc&sort=price,desc
 * Sorts by category ASC, then by price DESC
 */
```

**أمثلة الاستخدام:**

```bash
# Simple filtering
GET /api/v1/products?category=electronics
GET /api/v1/products?minPrice=100&maxPrice=500

# Combined filtering and sorting
GET /api/v1/products?category=electronics&minPrice=100&sortBy=price&sortDir=DESC

# Multiple tags
GET /api/v1/products?tags=new,featured,sale

# Full search with pagination
GET /api/v1/products/search?name=laptop&category=electronics&minPrice=500&maxPrice=2000&sortBy=price&sortDir=ASC&page=0&size=20

# Multi-field sorting
GET /api/v1/products/advanced-sort?sort=category,asc&sort=price,desc&sort=name,asc
```

---

## Authentication & Security

### 6. كيف تصمم Authentication في API؟

**الإجابة:**

#### 1. JWT (JSON Web Token) - الأكثر شيوعاً

```java
// JWT Configuration
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .and()
            .authorizeHttpRequests()
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            .and()
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}

// JWT Utility
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
            .signWith(SignatureAlgorithm.HS256, secret)
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
        try {
            Authentication authentication = authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(
                    request.getUsername(),
                    request.getPassword()
                )
            );

            UserDetails userDetails = (UserDetails) authentication.getPrincipal();
            String token = jwtService.generateToken(userDetails);

            AuthResponse response = new AuthResponse(
                token,
                "Bearer",
                expiration,
                userDetails.getUsername()
            );

            return ResponseEntity.ok(response);

        } catch (BadCredentialsException e) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
                .body(new AuthResponse("Invalid credentials"));
        }
    }

    @PostMapping("/register")
    public ResponseEntity<User> register(@Valid @RequestBody RegisterRequest request) {
        if (userService.existsByUsername(request.getUsername())) {
            return ResponseEntity.status(HttpStatus.CONFLICT)
                .body(null);
        }

        User user = userService.create(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(user);
    }

    @PostMapping("/refresh")
    public ResponseEntity<AuthResponse> refreshToken(
            @RequestHeader("Authorization") String authHeader) {

        String token = authHeader.substring(7); // Remove "Bearer "
        String username = jwtService.extractUsername(token);
        UserDetails userDetails = userService.loadUserByUsername(username);

        if (jwtService.validateToken(token, userDetails)) {
            String newToken = jwtService.generateToken(userDetails);
            return ResponseEntity.ok(new AuthResponse(newToken));
        }

        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
    }
}

// JWT Filter
@Component
public class JwtAuthFilter extends OncePerRequestFilter {

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

        if (authHeader != null && authHeader.startsWith("Bearer ")) {
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
        }

        filterChain.doFilter(request, response);
    }
}
```

**الاستخدام:**

```bash
# 1. Register
POST /api/auth/register
{
  "username": "john",
  "email": "john@example.com",
  "password": "password123"
}

# 2. Login
POST /api/auth/login
{
  "username": "john",
  "password": "password123"
}

# Response:
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "type": "Bearer",
  "expiresIn": 86400,
  "username": "john"
}

# 3. استخدام Token
GET /api/products
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# 4. Refresh Token
POST /api/auth/refresh
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

### 7. كيف تحمي API من الهجمات؟

**الإجابة:**

#### 1. Rate Limiting

```java
@Configuration
public class RateLimitConfig {

    @Bean
    public RateLimiter rateLimiter() {
        return RateLimiter.of("api", RateLimiterConfig.custom()
            .limitRefreshPeriod(Duration.ofMinutes(1))
            .limitForPeriod(100)
            .timeoutDuration(Duration.ofSeconds(5))
            .build());
    }
}

@RestController
@RequestMapping("/api/products")
public class ProductController {

    @Autowired
    private RateLimiter rateLimiter;

    @GetMapping
    @RateLimiter(name = "products", fallbackMethod = "rateLimitFallback")
    public ResponseEntity<List<Product>> getProducts() {
        return ResponseEntity.ok(productService.findAll());
    }

    public ResponseEntity<ErrorResponse> rateLimitFallback(Exception e) {
        return ResponseEntity
            .status(HttpStatus.TOO_MANY_REQUESTS)
            .body(new ErrorResponse("Rate limit exceeded. Try again later."));
    }
}

// Custom Rate Limiter with Redis
@Component
public class RateLimitFilter extends OncePerRequestFilter {

    @Autowired
    private RedisTemplate<String, Integer> redisTemplate;

    private static final int MAX_REQUESTS = 100;
    private static final long TIME_WINDOW = 60; // seconds

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain) throws ServletException, IOException {

        String clientId = getClientId(request);
        String key = "rate_limit:" + clientId;

        Integer requests = redisTemplate.opsForValue().get(key);

        if (requests == null) {
            redisTemplate.opsForValue().set(key, 1, TIME_WINDOW, TimeUnit.SECONDS);
        } else if (requests < MAX_REQUESTS) {
            redisTemplate.opsForValue().increment(key);
        } else {
            response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
            response.getWriter().write("{\"error\": \"Rate limit exceeded\"}");
            return;
        }

        filterChain.doFilter(request, response);
    }

    private String getClientId(HttpServletRequest request) {
        // يمكن استخدام IP, User ID, API Key
        return request.getRemoteAddr();
    }
}
```

#### 2. CORS Protection

```java
@Configuration
public class CorsConfig {

    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**")
                    .allowedOrigins("https://example.com", "https://app.example.com")
                    .allowedMethods("GET", "POST", "PUT", "DELETE", "PATCH")
                    .allowedHeaders("*")
                    .exposedHeaders("Authorization")
                    .allowCredentials(true)
                    .maxAge(3600);
            }
        };
    }
}
```

#### 3. SQL Injection Prevention

```java
// ✅ استخدم Prepared Statements / JPA
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    // ✅ Safe - يستخدم Prepared Statement
    @Query("SELECT u FROM User u WHERE u.username = :username")
    User findByUsername(@Param("username") String username);

    // ✅ Safe - JPA Query Methods
    Optional<User> findByEmail(String email);
}

// ❌ لا تستخدم String concatenation
@Query(value = "SELECT * FROM users WHERE username = '" + username + "'", nativeQuery = true)
// Vulnerable to SQL Injection!
```

#### 4. XSS Protection

```java
// Input Validation
@Data
public class CommentRequest {

    @NotBlank
    @Size(max = 500)
    @Pattern(regexp = "^[a-zA-Z0-9\\s.,!?'-]*$",
             message = "Comment contains invalid characters")
    private String content;
}

// Output Encoding
@Service
public class CommentService {

    public Comment create(CommentRequest request) {
        Comment comment = new Comment();
        // Sanitize input
        comment.setContent(HtmlUtils.htmlEscape(request.getContent()));
        return commentRepository.save(comment);
    }
}

// Response Headers
@Configuration
public class SecurityHeadersConfig {

    @Bean
    public FilterRegistrationBean<SecurityHeadersFilter> securityHeaders() {
        FilterRegistrationBean<SecurityHeadersFilter> bean = new FilterRegistrationBean<>();
        bean.setFilter(new SecurityHeadersFilter());
        bean.addUrlPatterns("/*");
        return bean;
    }
}

public class SecurityHeadersFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain) throws ServletException, IOException {

        response.setHeader("X-Content-Type-Options", "nosniff");
        response.setHeader("X-Frame-Options", "DENY");
        response.setHeader("X-XSS-Protection", "1; mode=block");
        response.setHeader("Content-Security-Policy", "default-src 'self'");

        filterChain.doFilter(request, response);
    }
}
```

#### 5. HTTPS Only

```yaml
# application.yml
server:
  ssl:
    enabled: true
    key-store: classpath:keystore.p12
    key-store-password: password
    key-store-type: PKCS12
  port: 8443

# Force HTTPS
security:
  require-ssl: true
```

```java
@Configuration
public class HttpsConfig {

    @Bean
    public TomcatServletWebServerFactory servletContainer() {
        TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory() {
            @Override
            protected void postProcessContext(Context context) {
                SecurityConstraint securityConstraint = new SecurityConstraint();
                securityConstraint.setUserConstraint("CONFIDENTIAL");
                SecurityCollection collection = new SecurityCollection();
                collection.addPattern("/*");
                securityConstraint.addCollection(collection);
                context.addConstraint(securityConstraint);
            }
        };
        return tomcat;
    }
}
```

---

تم إنشاء 7 أسئلة حتى الآن. هل تريد المتابعة أم عمل commit و push؟
