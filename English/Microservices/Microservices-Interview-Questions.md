# أسئلة المقابلات في الخدمات الصغيرة (Microservices)

## جدول المحتويات
1. [الأساسيات والمفاهيم](#الأساسيات-والمفاهيم)
2. [الخدمات الصغيرة مقابل الأحادية](#الخدمات-الصغيرة-مقابل-الأحادية)
3. [أنماط التصميم](#أنماط-التصميم)
4. [التواصل بين الخدمات](#التواصل-بين-الخدمات)
5. [Service Discovery](#service-discovery)
6. [API Gateway](#api-gateway)
7. [Load Balancing](#load-balancing)
8. [المراقبة والتتبع](#المراقبة-والتتبع)
9. [قواعد البيانات](#قواعد-البيانات)
10. [المرونة والاستقرار](#المرونة-والاستقرار)
11. [الأمان](#الأمان)
12. [استراتيجيات النشر](#استراتيجيات-النشر)
13. [السيناريوهات الواقعية](#السيناريوهات-الواقعية)

---

## الأساسيات والمفاهيم

### س1: ما هي الخدمات الصغيرة؟
**الجواب:**

الخدمات الصغيرة هي نمط معماري يقوم بتقسيم التطبيق إلى مجموعة من الخدمات الصغيرة المستقلة، حيث:

**الخصائص الأساسية:**
- كل خدمة تركز على وظيفة عمل واحدة (Single Responsibility)
- تعمل في عملية منفصلة (Separate Process)
- تتواصل عبر APIs خفيفة (REST, gRPC, Message Queues)
- يمكن نشرها وتوسيعها بشكل مستقل
- كل خدمة لها قاعدة بيانات خاصة

**مثال معماري:**

```
┌─────────────────────────────────────────────────────────┐
│                    Client Applications                  │
│              (Web, Mobile, Desktop, IoT)                │
└───────────────────────┬─────────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────────┐
│                     API Gateway                         │
│   (Routing, Auth, Rate Limiting, Aggregation)          │
└───┬────────┬────────┬────────┬────────┬────────┬────────┘
    │        │        │        │        │        │
┌───▼───┐ ┌─▼───┐ ┌──▼──┐ ┌───▼──┐ ┌──▼───┐ ┌──▼────┐
│ User  │ │Prod │ │Order│ │ Pay  │ │ Ship │ │ Email │
│Service│ │Svc  │ │ Svc │ │ Svc  │ │ Svc  │ │  Svc  │
└───┬───┘ └─┬───┘ └──┬──┘ └───┬──┘ └──┬───┘ └───┬───┘
    │       │        │        │       │         │
┌───▼───┐ ┌─▼───┐ ┌──▼──┐ ┌───▼──┐ ┌──▼───┐ ┌──▼────┐
│MySQL  │ │ Pg  │ │Mongo│ │Redis │ │ Pg   │ │ Queue │
└───────┘ └─────┘ └─────┘ └──────┘ └──────┘ └───────┘
```

---

### س2: ما هي المكونات الأساسية لمعمارية الخدمات الصغيرة؟
**الجواب:**

#### 1. Service Registry (سجل الخدمات)
```java
// Eureka Server
@SpringBootApplication
@EnableEurekaServer
public class ServiceRegistryApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServiceRegistryApplication.class, args);
    }
}
```

#### 2. API Gateway (بوابة API)
```java
@Configuration
public class GatewayConfig {
    @Bean
    public RouteLocator routes(RouteLocatorBuilder builder) {
        return builder.routes()
            .route("users", r -> r.path("/api/users/**")
                .uri("lb://USER-SERVICE"))
            .route("products", r -> r.path("/api/products/**")
                .uri("lb://PRODUCT-SERVICE"))
            .build();
    }
}
```

#### 3. Config Server (خادم الإعدادات)
```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    // Centralized configuration management
}
```

#### 4. Load Balancer (موزع الحمل)
```java
@LoadBalanced
@Bean
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

#### 5. Circuit Breaker (قاطع الدائرة)
```java
@CircuitBreaker(name = "productService", fallbackMethod = "fallback")
public Product getProduct(Long id) {
    return restTemplate.getForObject(
        "http://PRODUCT-SERVICE/products/" + id,
        Product.class
    );
}
```

#### 6. Distributed Tracing (التتبع الموزع)
```yaml
spring:
  zipkin:
    base-url: http://localhost:9411
  sleuth:
    sampler:
      probability: 1.0
```

---

## الخدمات الصغيرة مقابل الأحادية

### س3: ما الفرق بين Microservices و Monolithic؟
**الجواب:**

**جدول المقارنة:**

| الجانب | Monolithic | Microservices |
|--------|-----------|---------------|
| **البنية** | تطبيق واحد كبير | مجموعة خدمات صغيرة |
| **النشر** | يتم نشر كل شيء معاً | نشر مستقل لكل خدمة |
| **التوسع** | توسيع التطبيق كاملاً | توسيع الخدمات الفردية |
| **التكنولوجيا** | تقنية واحدة للكل | تقنيات مختلفة لكل خدمة |
| **قاعدة البيانات** | قاعدة بيانات مشتركة | قاعدة بيانات لكل خدمة |
| **الفشل** | فشل جزء يؤثر على الكل | فشل معزول في الخدمة |
| **التطوير** | فريق واحد كبير | فرق صغيرة مستقلة |
| **التعقيد** | بسيط في البداية | معقد من البداية |

**مثال عملي - متجر إلكتروني:**

#### Monolithic Architecture:
```
┌─────────────────────────────────────────────┐
│         E-commerce Application              │
│                                             │
│  ┌──────────────────────────────────────┐  │
│  │  User Management Module              │  │
│  ├──────────────────────────────────────┤  │
│  │  Product Catalog Module              │  │
│  ├──────────────────────────────────────┤  │
│  │  Shopping Cart Module                │  │
│  ├──────────────────────────────────────┤  │
│  │  Order Management Module             │  │
│  ├──────────────────────────────────────┤  │
│  │  Payment Processing Module           │  │
│  ├──────────────────────────────────────┤  │
│  │  Shipping & Tracking Module          │  │
│  └──────────────────────────────────────┘  │
│                    │                        │
│         ┌──────────▼──────────┐            │
│         │  Single Database    │            │
│         └─────────────────────┘            │
└─────────────────────────────────────────────┘

المشاكل:
- تحديث بسيط يتطلب نشر التطبيق كاملاً
- لا يمكن توسيع جزء معين (مثل Payment فقط)
- خطأ واحد قد يوقف التطبيق كاملاً
- صعوبة استخدام تقنيات مختلفة
```

#### Microservices Architecture:
```
                    ┌─────────────┐
                    │ API Gateway │
                    └──────┬──────┘
         ┌──────────┬──────┼──────┬──────────┐
         │          │      │      │          │
    ┌────▼───┐ ┌───▼───┐ ┌▼───┐ ┌▼────┐ ┌──▼────┐
    │ User   │ │Product│ │Cart│ │Order│ │Payment│
    │Service │ │Service│ │Svc │ │ Svc │ │  Svc  │
    │        │ │       │ │    │ │     │ │       │
    │ Java   │ │Python │ │Node│ │Java │ │  Go   │
    └────┬───┘ └───┬───┘ └─┬──┘ └─┬───┘ └───┬───┘
         │         │       │      │         │
    ┌────▼───┐ ┌───▼───┐ ┌─▼──┐ ┌─▼───┐ ┌───▼───┐
    │ MySQL  │ │Postgre│ │Redis│ │Mongo│ │ MySQL │
    └────────┘ └───────┘ └────┘ └─────┘ └───────┘

المزايا:
✓ نشر مستقل
✓ توسيع انتقائي (scale Payment فقط في Black Friday)
✓ عزل الأخطاء
✓ مرونة تكنولوجية
```

**كود مقارن:**

```java
// Monolithic - كل شيء في مكان واحد
@RestController
public class EcommerceController {

    @Autowired
    private UserService userService;

    @Autowired
    private ProductService productService;

    @Autowired
    private OrderService orderService;

    @Autowired
    private PaymentService paymentService;

    @PostMapping("/checkout")
    public Order checkout(@RequestBody CheckoutRequest request) {
        // كل شيء في معاملة واحدة
        User user = userService.getUser(request.getUserId());
        Product product = productService.getProduct(request.getProductId());
        Payment payment = paymentService.process(request.getPayment());
        Order order = orderService.create(user, product, payment);
        return order;
    }
}

// Microservices - خدمات منفصلة
// User Service
@RestController
@RequestMapping("/users")
public class UserController {
    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
}

// Product Service
@RestController
@RequestMapping("/products")
public class ProductController {
    @GetMapping("/{id}")
    public Product getProduct(@PathVariable Long id) {
        return productService.findById(id);
    }
}

// Order Service - يستدعي الخدمات الأخرى
@Service
public class OrderService {

    @Autowired
    private WebClient.Builder webClient;

    public Order createOrder(OrderRequest request) {
        // استدعاء User Service
        User user = webClient.build()
            .get()
            .uri("http://USER-SERVICE/users/" + request.getUserId())
            .retrieve()
            .bodyToMono(User.class)
            .block();

        // استدعاء Product Service
        Product product = webClient.build()
            .get()
            .uri("http://PRODUCT-SERVICE/products/" + request.getProductId())
            .retrieve()
            .bodyToMono(Product.class)
            .block();

        // استدعاء Payment Service
        Payment payment = webClient.build()
            .post()
            .uri("http://PAYMENT-SERVICE/payments")
            .bodyValue(request.getPaymentInfo())
            .retrieve()
            .bodyToMono(Payment.class)
            .block();

        // إنشاء الطلب
        return orderRepository.save(new Order(user, product, payment));
    }
}
```

---

### س4: متى يجب استخدام Microservices؟
**الجواب:**

**استخدم Microservices عندما:**

#### 1. التطبيق كبير ومعقد
```
مثال: Amazon, Netflix, Uber
- مئات الميزات
- ملايين المستخدمين
- فرق تطوير كبيرة
```

#### 2. حاجة لتوسع انتقائي
```java
// Black Friday Scenario
┌─────────────────────────────────────┐
│ Payment Service: 20 instances       │ ← توسيع عالي
│ Product Service: 10 instances       │ ← توسيع متوسط
│ User Service: 5 instances           │ ← توسيع منخفض
│ Email Service: 2 instances          │ ← لا توسيع
└─────────────────────────────────────┘
```

#### 3. فرق متعددة ومستقلة
```
Team A → User Service (Java)
Team B → Product Service (Python)
Team C → Order Service (Node.js)
Team D → Payment Service (Go)
```

#### 4. إصدارات متكررة
```
Day 1: Update Payment Service v1.2
Day 2: Update Product Service v2.0
Day 3: Update User Service v1.5
// بدون التأثير على الخدمات الأخرى
```

**لا تستخدم Microservices عندما:**

#### 1. التطبيق صغير وبسيط
```
مثال: Blog, Portfolio, Small CMS
- ميزات محدودة
- مستخدمين قليلين
- فريق صغير
→ Monolithic أفضل
```

#### 2. فريق صغير
```
فريق من 2-5 مطورين
→ تعقيد Microservices أكثر من الفائدة
```

#### 3. ميزانية محدودة
```
Microservices تحتاج:
- بنية تحتية أكبر
- أدوات مراقبة
- DevOps خبرة
→ تكلفة عالية
```

---

## أنماط التصميم

### س5: اشرح API Gateway Pattern
**الجواب:**

**التعريف:**
API Gateway هو نقطة دخول موحدة لجميع الخدمات في معمارية Microservices.

**الوظائف الأساسية:**

```
                     Client Request
                           │
                           ▼
        ┌──────────────────────────────────┐
        │         API Gateway              │
        │                                  │
        │  1. Authentication & Authorization│
        │  2. Request Routing              │
        │  3. Rate Limiting                │
        │  4. Response Aggregation         │
        │  5. Protocol Translation         │
        │  6. Load Balancing               │
        │  7. Caching                      │
        │  8. Logging & Monitoring         │
        └──────┬────────┬────────┬─────────┘
               │        │        │
        ┌──────▼──┐ ┌───▼───┐ ┌─▼──────┐
        │ Service │ │Service│ │Service │
        │    A    │ │   B   │ │   C    │
        └─────────┘ └───────┘ └────────┘
```

**مثال عملي - Spring Cloud Gateway:**

```java
@Configuration
public class GatewayConfiguration {

    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            // User Service Routes
            .route("user-service", r -> r
                .path("/api/users/**")
                .filters(f -> f
                    // إضافة header
                    .addRequestHeader("X-Gateway", "Spring-Cloud-Gateway")
                    // إعادة كتابة المسار
                    .rewritePath("/api/users/(?<segment>.*)", "/${segment}")
                    // التحقق من الصلاحيات
                    .filter(new AuthenticationFilter())
                    // Rate Limiting
                    .requestRateLimiter(config -> config
                        .setRateLimiter(redisRateLimiter())
                        .setKeyResolver(userKeyResolver()))
                    // Retry
                    .retry(config -> config
                        .setRetries(3)
                        .setBackoff(Duration.ofMillis(100),
                                   Duration.ofMillis(1000), 2, true)))
                .uri("lb://USER-SERVICE"))

            // Product Service Routes
            .route("product-service", r -> r
                .path("/api/products/**")
                .filters(f -> f
                    // Circuit Breaker
                    .circuitBreaker(config -> config
                        .setName("productServiceCB")
                        .setFallbackUri("forward:/fallback/products"))
                    // Response Caching
                    .cache(config -> config
                        .setTimeToLive(Duration.ofMinutes(5)))
                    // Request Timeout
                    .filter((exchange, chain) -> {
                        exchange.getAttributes()
                            .put(RESPONSE_TIMEOUT_ATTR,
                                 Duration.ofSeconds(3));
                        return chain.filter(exchange);
                    }))
                .uri("lb://PRODUCT-SERVICE"))

            // Order Service Routes
            .route("order-service", r -> r
                .path("/api/orders/**")
                .filters(f -> f
                    // تجميع الاستجابات من عدة خدمات
                    .filter(new AggregationFilter())
                    // تسجيل الطلبات
                    .filter(new LoggingFilter()))
                .uri("lb://ORDER-SERVICE"))

            .build();
    }

    // Rate Limiter Configuration
    @Bean
    public RedisRateLimiter redisRateLimiter() {
        return new RedisRateLimiter(10, 20); // 10 requests per second
    }

    @Bean
    public KeyResolver userKeyResolver() {
        return exchange -> Mono.just(
            exchange.getRequest()
                .getHeaders()
                .getFirst("X-User-ID")
        );
    }
}

// Authentication Filter
public class AuthenticationFilter implements GatewayFilter {

    @Autowired
    private JwtUtil jwtUtil;

    @Override
    public Mono<Void> filter(ServerWebExchange exchange,
                             GatewayFilterChain chain) {

        String token = exchange.getRequest()
            .getHeaders()
            .getFirst("Authorization");

        if (token == null || !token.startsWith("Bearer ")) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }

        try {
            String jwt = token.substring(7);
            jwtUtil.validateToken(jwt);
            return chain.filter(exchange);
        } catch (Exception e) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }
    }
}

// Response Aggregation Filter
public class AggregationFilter implements GatewayFilter {

    @Autowired
    private WebClient.Builder webClient;

    @Override
    public Mono<Void> filter(ServerWebExchange exchange,
                             GatewayFilterChain chain) {

        String orderId = exchange.getRequest()
            .getQueryParams()
            .getFirst("orderId");

        // الحصول على معلومات الطلب
        Mono<Order> orderMono = webClient.build()
            .get()
            .uri("http://ORDER-SERVICE/orders/" + orderId)
            .retrieve()
            .bodyToMono(Order.class);

        // الحصول على معلومات المستخدم
        Mono<User> userMono = orderMono.flatMap(order ->
            webClient.build()
                .get()
                .uri("http://USER-SERVICE/users/" + order.getUserId())
                .retrieve()
                .bodyToMono(User.class)
        );

        // الحصول على معلومات المنتجات
        Mono<List<Product>> productsMono = orderMono.flatMap(order ->
            Flux.fromIterable(order.getProductIds())
                .flatMap(productId ->
                    webClient.build()
                        .get()
                        .uri("http://PRODUCT-SERVICE/products/" + productId)
                        .retrieve()
                        .bodyToMono(Product.class))
                .collectList()
        );

        // دمج جميع الاستجابات
        return Mono.zip(orderMono, userMono, productsMono)
            .flatMap(tuple -> {
                OrderDetails details = new OrderDetails(
                    tuple.getT1(), // Order
                    tuple.getT2(), // User
                    tuple.getT3()  // Products
                );

                return exchange.getResponse()
                    .writeWith(Mono.just(
                        exchange.getResponse()
                            .bufferFactory()
                            .wrap(toJson(details).getBytes())
                    ));
            });
    }
}
```

**مثال - Kong API Gateway:**

```yaml
# kong.yml
_format_version: "2.1"

services:
  - name: user-service
    url: http://user-service:8080
    routes:
      - name: user-routes
        paths:
          - /api/users
        methods:
          - GET
          - POST
          - PUT
          - DELETE
    plugins:
      - name: jwt
        config:
          secret_is_base64: false
      - name: rate-limiting
        config:
          minute: 100
          hour: 1000
      - name: cors
        config:
          origins:
            - "*"
          methods:
            - GET
            - POST
      - name: request-transformer
        config:
          add:
            headers:
              - X-Gateway:Kong

  - name: product-service
    url: http://product-service:8080
    routes:
      - name: product-routes
        paths:
          - /api/products
    plugins:
      - name: response-caching
        config:
          cache_ttl: 300
      - name: proxy-cache
        config:
          content_type:
            - application/json
          cache_ttl: 300
```

**الفوائد:**
1. نقطة دخول موحدة
2. تبسيط الاتصال للعملاء
3. أمان مركزي
4. تقليل عدد الطلبات (Response Aggregation)
5. تحويل البروتوكولات

---

### س6: اشرح Circuit Breaker Pattern
**الجواب:**

**التعريف:**
Circuit Breaker يمنع انتشار الأعطال في النظام الموزع من خلال إيقاف المحاولات المتكررة للخدمة الفاشلة.

**حالات Circuit Breaker:**

```
                    ┌──────────┐
          success   │          │  timeout/failure
         ┌─────────►│  CLOSED  ├──────────┐
         │          │          │          │
         │          └──────────┘          │
         │                                │
         │                                ▼
    ┌────┴────┐                    ┌──────────┐
    │  HALF   │                    │   OPEN   │
    │  OPEN   │◄───────────────────┤          │
    └─────────┘     after timeout  └──────────┘
    success │
    count   │ failure
    reached │
            ▼
         success

States:
- CLOSED: طبيعي، الطلبات تمر
- OPEN: الخدمة فاشلة، رفض الطلبات
- HALF-OPEN: محاولة اختبار إذا تعافت الخدمة
```

**مثال - Resilience4j:**

```java
// Application Configuration
@Configuration
public class Resilience4jConfig {

    @Bean
    public Customizer<Resilience4JCircuitBreakerFactory> defaultCustomizer() {
        return factory -> factory.configureDefault(id -> new Resilience4JConfigBuilder(id)
            .circuitBreakerConfig(CircuitBreakerConfig.custom()
                // فتح الدائرة بعد 50% فشل
                .failureRateThreshold(50)
                // الحد الأدنى للطلبات قبل حساب معدل الفشل
                .minimumNumberOfCalls(5)
                // حجم النافذة لحساب معدل الفشل
                .slidingWindowSize(10)
                .slidingWindowType(SlidingWindowType.COUNT_BASED)
                // المدة في حالة OPEN
                .waitDurationInOpenState(Duration.ofSeconds(30))
                // عدد الطلبات المسموحة في HALF-OPEN
                .permittedNumberOfCallsInHalfOpenState(3)
                // تسجيل الأحداث
                .recordExceptions(IOException.class, TimeoutException.class)
                .ignoreExceptions(BusinessException.class)
                .build())
            .timeLimiterConfig(TimeLimiterConfig.custom()
                .timeoutDuration(Duration.ofSeconds(3))
                .build())
            .build());
    }
}

// Service Implementation
@Service
@Slf4j
public class ProductService {

    @Autowired
    private RestTemplate restTemplate;

    @Autowired
    private CircuitBreakerFactory circuitBreakerFactory;

    // استخدام Circuit Breaker مع Fallback
    @CircuitBreaker(name = "productService", fallbackMethod = "getProductFallback")
    @Retry(name = "productService", fallbackMethod = "getProductFallback")
    @RateLimiter(name = "productService")
    @Bulkhead(name = "productService")
    public Product getProduct(Long id) {
        log.info("Calling product service for id: {}", id);

        String url = "http://PRODUCT-SERVICE/products/" + id;
        return restTemplate.getForObject(url, Product.class);
    }

    // Fallback Method
    private Product getProductFallback(Long id, Exception ex) {
        log.error("Product service failed, returning fallback for id: {}", id, ex);

        return Product.builder()
            .id(id)
            .name("Product Temporarily Unavailable")
            .description("Service is currently down, please try again later")
            .price(0.0)
            .available(false)
            .build();
    }

    // طريقة برمجية للتحكم في Circuit Breaker
    public Product getProductWithManualCircuitBreaker(Long id) {
        CircuitBreaker circuitBreaker = circuitBreakerFactory.create("product-service");

        return circuitBreaker.run(
            // Supplier - العملية الأساسية
            () -> {
                String url = "http://PRODUCT-SERVICE/products/" + id;
                return restTemplate.getForObject(url, Product.class);
            },
            // Fallback - عند الفشل
            throwable -> {
                log.error("Circuit breaker triggered", throwable);
                return getProductFallback(id, (Exception) throwable);
            }
        );
    }
}

// application.yml
resilience4j:
  circuitbreaker:
    instances:
      productService:
        register-health-indicator: true
        sliding-window-size: 10
        minimum-number-of-calls: 5
        permitted-number-of-calls-in-half-open-state: 3
        automatic-transition-from-open-to-half-open-enabled: true
        wait-duration-in-open-state: 30s
        failure-rate-threshold: 50
        event-consumer-buffer-size: 10
        record-exceptions:
          - org.springframework.web.client.HttpServerErrorException
          - java.io.IOException
          - java.util.concurrent.TimeoutException
        ignore-exceptions:
          - com.example.BusinessException

  retry:
    instances:
      productService:
        max-attempts: 3
        wait-duration: 1s
        enable-exponential-backoff: true
        exponential-backoff-multiplier: 2
        retry-exceptions:
          - java.io.IOException

  ratelimiter:
    instances:
      productService:
        limit-for-period: 10
        limit-refresh-period: 1s
        timeout-duration: 0

  bulkhead:
    instances:
      productService:
        max-concurrent-calls: 10
        max-wait-duration: 0

# Monitoring Circuit Breaker Events
@Component
@Slf4j
public class CircuitBreakerEventListener {

    @Autowired
    private CircuitBreakerRegistry circuitBreakerRegistry;

    @PostConstruct
    public void registerEventListener() {
        circuitBreakerRegistry.circuitBreaker("productService")
            .getEventPublisher()
            .onSuccess(event -> log.info("Circuit Breaker Success: {}", event))
            .onError(event -> log.error("Circuit Breaker Error: {}", event))
            .onStateTransition(event -> log.warn("Circuit Breaker State Changed: {} -> {}",
                event.getStateTransition().getFromState(),
                event.getStateTransition().getToState()))
            .onSlowCallRateExceeded(event -> log.warn("Slow Call Rate Exceeded: {}", event))
            .onFailureRateExceeded(event -> log.error("Failure Rate Exceeded: {}", event));
    }
}
```

**مثال سيناريو كامل:**

```java
@RestController
@RequestMapping("/orders")
public class OrderController {

    @Autowired
    private OrderService orderService;

    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(@RequestBody OrderRequest request) {
        try {
            OrderResponse response = orderService.createOrder(request);
            return ResponseEntity.ok(response);
        } catch (CallNotPermittedException e) {
            // Circuit Breaker is OPEN
            return ResponseEntity.status(HttpStatus.SERVICE_UNAVAILABLE)
                .body(OrderResponse.builder()
                    .message("Service temporarily unavailable")
                    .build());
        }
    }
}

@Service
public class OrderService {

    @CircuitBreaker(name = "payment", fallbackMethod = "paymentFallback")
    private Payment processPayment(PaymentRequest request) {
        return paymentClient.process(request);
    }

    @CircuitBreaker(name = "inventory", fallbackMethod = "inventoryFallback")
    private void reserveInventory(List<OrderItem> items) {
        inventoryClient.reserve(items);
    }

    @CircuitBreaker(name = "shipping", fallbackMethod = "shippingFallback")
    private Shipping arrangeShipping(ShippingRequest request) {
        return shippingClient.arrange(request);
    }

    public OrderResponse createOrder(OrderRequest request) {
        try {
            // 1. حجز المخزون
            reserveInventory(request.getItems());

            // 2. معالجة الدفع
            Payment payment = processPayment(request.getPayment());

            // 3. ترتيب الشحن
            Shipping shipping = arrangeShipping(request.getShipping());

            // 4. إنشاء الطلب
            Order order = orderRepository.save(new Order(request, payment, shipping));

            return OrderResponse.success(order);

        } catch (Exception e) {
            // Rollback/Compensation
            compensate(request);
            throw e;
        }
    }

    // Fallback methods
    private Payment paymentFallback(PaymentRequest request, Exception ex) {
        log.error("Payment service failed", ex);
        throw new ServiceUnavailableException("Payment service is down");
    }

    private void inventoryFallback(List<OrderItem> items, Exception ex) {
        log.error("Inventory service failed", ex);
        throw new ServiceUnavailableException("Inventory service is down");
    }

    private Shipping shippingFallback(ShippingRequest request, Exception ex) {
        log.error("Shipping service failed", ex);
        // يمكن معالجة الشحن لاحقاً
        return Shipping.delayed();
    }
}
```

---

### س7: اشرح Saga Pattern
**الجواب:**

**التعريف:**
Saga Pattern هو نمط لإدارة المعاملات الموزعة (Distributed Transactions) في Microservices من خلال تقسيمها إلى سلسلة من المعاملات المحلية.

**النوعان الأساسيان:**

#### 1. Choreography-based Saga (قائم على الأحداث)

```
الخدمات تتواصل مباشرة عبر الأحداث

Order Created
     │
     ▼
┌─────────────┐     Payment      ┌─────────────┐
│   Order     │─────Requested───►│   Payment   │
│  Service    │                  │   Service   │
└─────────────┘                  └─────────────┘
     ▲                                   │
     │                          Payment Completed
     │                                   │
     │                                   ▼
┌─────────────┐                  ┌─────────────┐
│  Shipping   │◄───Ship─────────│  Inventory  │
│  Service    │    Requested     │   Service   │
└─────────────┘                  └─────────────┘
```

**مثال - Event-Driven Saga:**

```java
// Order Service - بداية Saga
@Service
public class OrderService {

    @Autowired
    private ApplicationEventPublisher eventPublisher;

    @Autowired
    private OrderRepository orderRepository;

    @Transactional
    public Order createOrder(OrderRequest request) {
        // 1. إنشاء الطلب بحالة PENDING
        Order order = Order.builder()
            .userId(request.getUserId())
            .items(request.getItems())
            .totalAmount(calculateTotal(request.getItems()))
            .status(OrderStatus.PENDING)
            .build();

        order = orderRepository.save(order);

        // 2. نشر حدث Order Created
        eventPublisher.publishEvent(new OrderCreatedEvent(
            order.getId(),
            order.getUserId(),
            order.getItems(),
            order.getTotalAmount()
        ));

        return order;
    }

    // الاستماع لأحداث النجاح
    @EventListener
    @Transactional
    public void handlePaymentCompleted(PaymentCompletedEvent event) {
        Order order = orderRepository.findById(event.getOrderId())
            .orElseThrow();
        order.setPaymentId(event.getPaymentId());
        order.setStatus(OrderStatus.PAYMENT_COMPLETED);
        orderRepository.save(order);
    }

    @EventListener
    @Transactional
    public void handleInventoryReserved(InventoryReservedEvent event) {
        Order order = orderRepository.findById(event.getOrderId())
            .orElseThrow();
        order.setStatus(OrderStatus.INVENTORY_RESERVED);
        orderRepository.save(order);
    }

    @EventListener
    @Transactional
    public void handleShippingArranged(ShippingArrangedEvent event) {
        Order order = orderRepository.findById(event.getOrderId())
            .orElseThrow();
        order.setShippingId(event.getShippingId());
        order.setStatus(OrderStatus.CONFIRMED);
        orderRepository.save(order);
    }

    // Compensation - الاستماع لأحداث الفشل
    @EventListener
    @Transactional
    public void handlePaymentFailed(PaymentFailedEvent event) {
        Order order = orderRepository.findById(event.getOrderId())
            .orElseThrow();
        order.setStatus(OrderStatus.CANCELLED);
        order.setFailureReason(event.getReason());
        orderRepository.save(order);

        // نشر حدث Order Cancelled
        eventPublisher.publishEvent(new OrderCancelledEvent(order.getId()));
    }
}

// Payment Service - خطوة في Saga
@Service
public class PaymentService {

    @Autowired
    private ApplicationEventPublisher eventPublisher;

    @Autowired
    private PaymentRepository paymentRepository;

    // الاستماع لحدث Order Created
    @EventListener
    @Transactional
    public void handleOrderCreated(OrderCreatedEvent event) {
        try {
            // معالجة الدفع
            Payment payment = Payment.builder()
                .orderId(event.getOrderId())
                .userId(event.getUserId())
                .amount(event.getTotalAmount())
                .status(PaymentStatus.PROCESSING)
                .build();

            payment = paymentRepository.save(payment);

            // محاكاة معالجة الدفع
            boolean success = processPaymentWithGateway(payment);

            if (success) {
                payment.setStatus(PaymentStatus.COMPLETED);
                paymentRepository.save(payment);

                // نشر حدث Payment Completed
                eventPublisher.publishEvent(new PaymentCompletedEvent(
                    event.getOrderId(),
                    payment.getId(),
                    payment.getAmount()
                ));
            } else {
                throw new PaymentProcessingException("Payment declined");
            }

        } catch (Exception e) {
            // نشر حدث Payment Failed
            eventPublisher.publishEvent(new PaymentFailedEvent(
                event.getOrderId(),
                e.getMessage()
            ));
        }
    }

    // Compensation - إلغاء الدفع
    @EventListener
    @Transactional
    public void handleOrderCancelled(OrderCancelledEvent event) {
        Payment payment = paymentRepository.findByOrderId(event.getOrderId())
            .orElse(null);

        if (payment != null && payment.getStatus() == PaymentStatus.COMPLETED) {
            // استرجاع المبلغ
            refundPayment(payment);
            payment.setStatus(PaymentStatus.REFUNDED);
            paymentRepository.save(payment);
        }
    }
}

// Inventory Service
@Service
public class InventoryService {

    @EventListener
    @Transactional
    public void handlePaymentCompleted(PaymentCompletedEvent event) {
        try {
            // حجز المخزون
            List<OrderItem> items = getOrderItems(event.getOrderId());
            reserveItems(items);

            // نشر حدث Inventory Reserved
            eventPublisher.publishEvent(new InventoryReservedEvent(
                event.getOrderId(),
                items
            ));

        } catch (InsufficientStockException e) {
            // نشر حدث Inventory Failed
            eventPublisher.publishEvent(new InventoryFailedEvent(
                event.getOrderId(),
                e.getMessage()
            ));
        }
    }

    // Compensation
    @EventListener
    @Transactional
    public void handleOrderCancelled(OrderCancelledEvent event) {
        // إلغاء حجز المخزون
        releaseReservedItems(event.getOrderId());
    }
}

// Event Classes
@Data
@AllArgsConstructor
public class OrderCreatedEvent {
    private Long orderId;
    private Long userId;
    private List<OrderItem> items;
    private BigDecimal totalAmount;
}

@Data
@AllArgsConstructor
public class PaymentCompletedEvent {
    private Long orderId;
    private Long paymentId;
    private BigDecimal amount;
}

@Data
@AllArgsConstructor
public class PaymentFailedEvent {
    private Long orderId;
    private String reason;
}
```

#### 2. Orchestration-based Saga (قائم على التنسيق)

```
منسق مركزي يدير جميع الخطوات

        ┌───────────────────┐
        │  Saga Orchestrator │
        └─────────┬──────────┘
                  │
         ┌────────┼────────┬──────────┐
         │        │        │          │
    ┌────▼───┐ ┌─▼────┐ ┌─▼──────┐ ┌─▼───────┐
    │ Order  │ │Payment│ │Inventory│ │Shipping │
    │Service │ │Service│ │ Service │ │ Service │
    └────────┘ └───────┘ └─────────┘ └─────────┘
```

**مثال - Orchestrator Pattern:**

```java
// Saga Orchestrator
@Service
@Slf4j
public class OrderSagaOrchestrator {

    @Autowired
    private OrderService orderService;

    @Autowired
    private PaymentService paymentService;

    @Autowired
    private InventoryService inventoryService;

    @Autowired
    private ShippingService shippingService;

    @Autowired
    private SagaStateRepository sagaStateRepository;

    public SagaResult executeOrderSaga(OrderRequest request) {
        // 1. إنشاء حالة Saga
        SagaState sagaState = SagaState.builder()
            .sagaId(UUID.randomUUID().toString())
            .status(SagaStatus.STARTED)
            .currentStep(SagaStep.ORDER_CREATION)
            .build();

        sagaStateRepository.save(sagaState);

        try {
            // Step 1: إنشاء الطلب
            log.info("Saga {}: Creating order", sagaState.getSagaId());
            Order order = orderService.createOrder(request);
            sagaState.setOrderId(order.getId());
            sagaState.setCurrentStep(SagaStep.ORDER_CREATED);
            sagaStateRepository.save(sagaState);

            // Step 2: معالجة الدفع
            log.info("Saga {}: Processing payment", sagaState.getSagaId());
            Payment payment = paymentService.processPayment(
                order.getId(),
                order.getTotalAmount()
            );
            sagaState.setPaymentId(payment.getId());
            sagaState.setCurrentStep(SagaStep.PAYMENT_COMPLETED);
            sagaStateRepository.save(sagaState);

            // Step 3: حجز المخزون
            log.info("Saga {}: Reserving inventory", sagaState.getSagaId());
            inventoryService.reserveItems(order.getItems());
            sagaState.setCurrentStep(SagaStep.INVENTORY_RESERVED);
            sagaStateRepository.save(sagaState);

            // Step 4: ترتيب الشحن
            log.info("Saga {}: Arranging shipping", sagaState.getSagaId());
            Shipping shipping = shippingService.arrangeShipping(
                order.getId(),
                request.getShippingAddress()
            );
            sagaState.setShippingId(shipping.getId());
            sagaState.setCurrentStep(SagaStep.SHIPPING_ARRANGED);
            sagaStateRepository.save(sagaState);

            // Success: تأكيد الطلب
            orderService.confirmOrder(order.getId());
            sagaState.setStatus(SagaStatus.COMPLETED);
            sagaStateRepository.save(sagaState);

            log.info("Saga {}: Completed successfully", sagaState.getSagaId());
            return SagaResult.success(order);

        } catch (Exception e) {
            log.error("Saga {}: Failed at step {}",
                sagaState.getSagaId(),
                sagaState.getCurrentStep(),
                e);

            // Compensation: التراجع عن الخطوات المكتملة
            compensate(sagaState);

            sagaState.setStatus(SagaStatus.COMPENSATED);
            sagaState.setFailureReason(e.getMessage());
            sagaStateRepository.save(sagaState);

            return SagaResult.failure(e.getMessage());
        }
    }

    private void compensate(SagaState sagaState) {
        log.info("Saga {}: Starting compensation from step {}",
            sagaState.getSagaId(),
            sagaState.getCurrentStep());

        try {
            switch (sagaState.getCurrentStep()) {
                case SHIPPING_ARRANGED:
                    // إلغاء الشحن
                    shippingService.cancelShipping(sagaState.getShippingId());
                    log.info("Saga {}: Shipping cancelled", sagaState.getSagaId());
                    // intentional fall-through

                case INVENTORY_RESERVED:
                    // إلغاء حجز المخزون
                    inventoryService.releaseItems(sagaState.getOrderId());
                    log.info("Saga {}: Inventory released", sagaState.getSagaId());
                    // intentional fall-through

                case PAYMENT_COMPLETED:
                    // استرجاع المبلغ
                    paymentService.refund(sagaState.getPaymentId());
                    log.info("Saga {}: Payment refunded", sagaState.getSagaId());
                    // intentional fall-through

                case ORDER_CREATED:
                    // إلغاء الطلب
                    orderService.cancelOrder(sagaState.getOrderId());
                    log.info("Saga {}: Order cancelled", sagaState.getSagaId());
                    break;

                default:
                    break;
            }

            log.info("Saga {}: Compensation completed", sagaState.getSagaId());

        } catch (Exception e) {
            log.error("Saga {}: Compensation failed", sagaState.getSagaId(), e);
            // يجب التعامل يدوياً أو إعادة المحاولة
            sagaState.setStatus(SagaStatus.COMPENSATION_FAILED);
            sagaStateRepository.save(sagaState);
        }
    }

    // استرجاع واستكمال Sagas الفاشلة
    @Scheduled(fixedDelay = 60000) // كل دقيقة
    public void recoverFailedSagas() {
        List<SagaState> failedSagas = sagaStateRepository
            .findByStatusAndUpdatedAtBefore(
                SagaStatus.STARTED,
                LocalDateTime.now().minusMinutes(5)
            );

        for (SagaState saga : failedSagas) {
            log.warn("Recovering failed saga: {}", saga.getSagaId());
            compensate(saga);
        }
    }
}

// Saga State Entity
@Entity
@Data
@Builder
public class SagaState {
    @Id
    private String sagaId;

    @Enumerated(EnumType.STRING)
    private SagaStatus status;

    @Enumerated(EnumType.STRING)
    private SagaStep currentStep;

    private Long orderId;
    private Long paymentId;
    private Long shippingId;

    private String failureReason;

    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
}

enum SagaStatus {
    STARTED,
    COMPLETED,
    COMPENSATED,
    COMPENSATION_FAILED
}

enum SagaStep {
    ORDER_CREATION,
    ORDER_CREATED,
    PAYMENT_COMPLETED,
    INVENTORY_RESERVED,
    SHIPPING_ARRANGED
}
```

**المقارنة:**

| الجانب | Choreography | Orchestration |
|--------|--------------|---------------|
| **التعقيد** | توزيع منطق Saga | تجميع منطق Saga |
| **الربط** | ربط ضعيف (Loose Coupling) | ربط قوي مع Orchestrator |
| **التتبع** | صعب | سهل |
| **الفشل** | صعب التشخيص | سهل التشخيص |
| **المرونة** | سهل إضافة خدمات | تحديث Orchestrator مطلوب |

---

### س8: اشرح CQRS Pattern
**الجواب:**

**التعريف:**
CQRS (Command Query Responsibility Segregation) هو نمط يفصل عمليات القراءة (Queries) عن عمليات الكتابة (Commands).

**البنية الأساسية:**

```
                 ┌──────────────┐
                 │   Client     │
                 └──────┬───────┘
                        │
            ┌───────────┴────────────┐
            │                        │
    ┌───────▼────────┐      ┌───────▼────────┐
    │   Commands     │      │    Queries     │
    │  (Write Side)  │      │  (Read Side)   │
    └───────┬────────┘      └───────┬────────┘
            │                        │
    ┌───────▼────────┐      ┌───────▼────────┐
    │ Write Database │      │  Read Database │
    │  (Normalized)  │      │ (Denormalized) │
    └───────┬────────┘      └───────▲────────┘
            │                        │
            └──────►Events───────────┘
```

**مثال كامل:**

```java
// Commands (Write Operations)
public interface Command {}

@Data
@AllArgsConstructor
public class CreateProductCommand implements Command {
    private String name;
    private String description;
    private BigDecimal price;
    private Integer stock;
}

@Data
@AllArgsConstructor
public class UpdateProductPriceCommand implements Command {
    private Long productId;
    private BigDecimal newPrice;
}

@Data
@AllArgsConstructor
public class UpdateProductStockCommand implements Command {
    private Long productId;
    private Integer quantity;
}

// Command Handlers
@Service
@Transactional
public class ProductCommandService {

    @Autowired
    private ProductWriteRepository writeRepository;

    @Autowired
    private EventPublisher eventPublisher;

    public Long handle(CreateProductCommand command) {
        // التحقق من صحة البيانات
        validateProduct(command);

        // إنشاء المنتج
        ProductEntity product = ProductEntity.builder()
            .name(command.getName())
            .description(command.getDescription())
            .price(command.getPrice())
            .stock(command.getStock())
            .createdAt(LocalDateTime.now())
            .build();

        product = writeRepository.save(product);

        // نشر حدث ProductCreated
        eventPublisher.publish(new ProductCreatedEvent(
            product.getId(),
            product.getName(),
            product.getDescription(),
            product.getPrice(),
            product.getStock()
        ));

        return product.getId();
    }

    public void handle(UpdateProductPriceCommand command) {
        ProductEntity product = writeRepository.findById(command.getProductId())
            .orElseThrow(() -> new ProductNotFoundException(command.getProductId()));

        BigDecimal oldPrice = product.getPrice();
        product.setPrice(command.getNewPrice());
        product.setUpdatedAt(LocalDateTime.now());

        writeRepository.save(product);

        // نشر حدث PriceUpdated
        eventPublisher.publish(new ProductPriceUpdatedEvent(
            product.getId(),
            oldPrice,
            command.getNewPrice()
        ));
    }

    public void handle(UpdateProductStockCommand command) {
        ProductEntity product = writeRepository.findById(command.getProductId())
            .orElseThrow(() -> new ProductNotFoundException(command.getProductId()));

        if (product.getStock() + command.getQuantity() < 0) {
            throw new InsufficientStockException(product.getId());
        }

        Integer oldStock = product.getStock();
        product.setStock(product.getStock() + command.getQuantity());
        product.setUpdatedAt(LocalDateTime.now());

        writeRepository.save(product);

        // نشر حدث StockUpdated
        eventPublisher.publish(new ProductStockUpdatedEvent(
            product.getId(),
            oldStock,
            product.getStock()
        ));
    }
}

// Queries (Read Operations)
public interface Query<T> {}

@Data
@AllArgsConstructor
public class GetProductByIdQuery implements Query<ProductDTO> {
    private Long productId;
}

@Data
@AllArgsConstructor
public class SearchProductsQuery implements Query<List<ProductDTO>> {
    private String keyword;
    private BigDecimal minPrice;
    private BigDecimal maxPrice;
    private Integer page;
    private Integer size;
}

@Data
@AllArgsConstructor
public class GetProductsByCategoryQuery implements Query<List<ProductDTO>> {
    private String category;
}

// Query Handlers
@Service
@Transactional(readOnly = true)
public class ProductQueryService {

    @Autowired
    private ProductReadRepository readRepository;

    public ProductDTO handle(GetProductByIdQuery query) {
        ProductReadModel product = readRepository.findById(query.getProductId())
            .orElseThrow(() -> new ProductNotFoundException(query.getProductId()));

        return toDTO(product);
    }

    public List<ProductDTO> handle(SearchProductsQuery query) {
        Pageable pageable = PageRequest.of(query.getPage(), query.getSize());

        Page<ProductReadModel> products = readRepository.searchProducts(
            query.getKeyword(),
            query.getMinPrice(),
            query.getMaxPrice(),
            pageable
        );

        return products.stream()
            .map(this::toDTO)
            .collect(Collectors.toList());
    }

    public List<ProductDTO> handle(GetProductsByCategoryQuery query) {
        List<ProductReadModel> products = readRepository
            .findByCategory(query.getCategory());

        return products.stream()
            .map(this::toDTO)
            .collect(Collectors.toList());
    }

    private ProductDTO toDTO(ProductReadModel product) {
        return ProductDTO.builder()
            .id(product.getId())
            .name(product.getName())
            .description(product.getDescription())
            .price(product.getPrice())
            .stock(product.getStock())
            .category(product.getCategory())
            .rating(product.getRating())
            .reviewCount(product.getReviewCount())
            .build();
    }
}

// Write Model (Normalized)
@Entity
@Table(name = "products")
@Data
@Builder
public class ProductEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String description;
    private BigDecimal price;
    private Integer stock;

    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
}

// Read Model (Denormalized for optimal querying)
@Entity
@Table(name = "product_read_model")
@Data
@Builder
public class ProductReadModel {
    @Id
    private Long id;

    private String name;
    private String description;
    private BigDecimal price;
    private Integer stock;
    private String category;
    private Double rating;
    private Integer reviewCount;

    // Denormalized fields for faster queries
    private String brandName;
    private String supplierName;
    private LocalDateTime lastPriceUpdate;
    private LocalDateTime lastStockUpdate;

    // Search optimization
    @Column(name = "search_vector")
    private String searchVector; // Full-text search
}

// Event Handlers - Synchronize Read Model
@Component
public class ProductReadModelUpdater {

    @Autowired
    private ProductReadRepository readRepository;

    @EventListener
    @Transactional
    public void on(ProductCreatedEvent event) {
        ProductReadModel readModel = ProductReadModel.builder()
            .id(event.getProductId())
            .name(event.getName())
            .description(event.getDescription())
            .price(event.getPrice())
            .stock(event.getStock())
            .rating(0.0)
            .reviewCount(0)
            .build();

        readRepository.save(readModel);
    }

    @EventListener
    @Transactional
    public void on(ProductPriceUpdatedEvent event) {
        ProductReadModel readModel = readRepository.findById(event.getProductId())
            .orElseThrow();

        readModel.setPrice(event.getNewPrice());
        readModel.setLastPriceUpdate(LocalDateTime.now());

        readRepository.save(readModel);
    }

    @EventListener
    @Transactional
    public void on(ProductStockUpdatedEvent event) {
        ProductReadModel readModel = readRepository.findById(event.getProductId())
            .orElseThrow();

        readModel.setStock(event.getNewStock());
        readModel.setLastStockUpdate(LocalDateTime.now());

        readRepository.save(readModel);
    }
}

// Controller - CQRS Interface
@RestController
@RequestMapping("/api/products")
public class ProductController {

    @Autowired
    private ProductCommandService commandService;

    @Autowired
    private ProductQueryService queryService;

    // Command Endpoint
    @PostMapping
    public ResponseEntity<Long> createProduct(@RequestBody CreateProductCommand command) {
        Long productId = commandService.handle(command);
        return ResponseEntity.status(HttpStatus.CREATED).body(productId);
    }

    // Command Endpoint
    @PutMapping("/{id}/price")
    public ResponseEntity<Void> updatePrice(
            @PathVariable Long id,
            @RequestBody UpdateProductPriceCommand command) {
        command.setProductId(id);
        commandService.handle(command);
        return ResponseEntity.ok().build();
    }

    // Query Endpoint
    @GetMapping("/{id}")
    public ResponseEntity<ProductDTO> getProduct(@PathVariable Long id) {
        ProductDTO product = queryService.handle(new GetProductByIdQuery(id));
        return ResponseEntity.ok(product);
    }

    // Query Endpoint
    @GetMapping("/search")
    public ResponseEntity<List<ProductDTO>> searchProducts(
            @RequestParam String keyword,
            @RequestParam(required = false) BigDecimal minPrice,
            @RequestParam(required = false) BigDecimal maxPrice,
            @RequestParam(defaultValue = "0") Integer page,
            @RequestParam(defaultValue = "20") Integer size) {

        List<ProductDTO> products = queryService.handle(new SearchProductsQuery(
            keyword, minPrice, maxPrice, page, size
        ));

        return ResponseEntity.ok(products);
    }
}

// Repositories
@Repository
public interface ProductWriteRepository extends JpaRepository<ProductEntity, Long> {
    // Simple CRUD operations
}

@Repository
public interface ProductReadRepository extends JpaRepository<ProductReadModel, Long> {

    // Optimized read queries
    @Query("SELECT p FROM ProductReadModel p WHERE " +
           "(:keyword IS NULL OR p.searchVector LIKE %:keyword%) AND " +
           "(:minPrice IS NULL OR p.price >= :minPrice) AND " +
           "(:maxPrice IS NULL OR p.price <= :maxPrice)")
    Page<ProductReadModel> searchProducts(
        @Param("keyword") String keyword,
        @Param("minPrice") BigDecimal minPrice,
        @Param("maxPrice") BigDecimal maxPrice,
        Pageable pageable
    );

    List<ProductReadModel> findByCategory(String category);

    List<ProductReadModel> findTop10ByOrderByRatingDesc();
}
```

**المزايا:**
1. **الأداء**: نماذج قراءة محسّنة
2. **المرونة**: تصميم مختلف للقراءة والكتابة
3. **التوسع**: توسيع القراءة والكتابة بشكل مستقل
4. **البساطة**: استعلامات بسيطة على جانب القراءة

**العيوب:**
1. **التعقيد**: نماذج بيانات مكررة
2. **التأخير**: eventual consistency بين النماذج
3. **التزامن**: مزيد من الكود للصيانة

---

### س9: اشرح Event Sourcing Pattern
**الجواب:**

**التعريف:**
Event Sourcing هو نمط يخزن جميع التغييرات على حالة التطبيق كسلسلة من الأحداث بدلاً من تخزين الحالة الحالية فقط.

**المبدأ الأساسي:**

```
التخزين التقليدي:
Account Balance = 1000 (الحالة الحالية فقط)

Event Sourcing:
1. AccountCreated: balance = 0
2. MoneyDeposited: +500
3. MoneyDeposited: +700
4. MoneyWithdrawn: -200
→ Current Balance = 1000 (محسوبة من الأحداث)
```

**مثال كامل - Banking System:**

```java
// Event Base Class
@Data
@MappedSuperclass
public abstract class DomainEvent {
    @Id
    private String eventId;
    private String aggregateId;
    private Long version;
    private LocalDateTime occurredAt;
    private String eventType;

    public DomainEvent() {
        this.eventId = UUID.randomUUID().toString();
        this.occurredAt = LocalDateTime.now();
        this.eventType = this.getClass().getSimpleName();
    }
}

// Domain Events
@Entity
@Table(name = "account_events")
@Data
@EqualsAndHashCode(callSuper = true)
public class AccountCreatedEvent extends DomainEvent {
    private String accountNumber;
    private String ownerName;
    private BigDecimal initialBalance;

    public AccountCreatedEvent(String aggregateId, String accountNumber,
                              String ownerName, BigDecimal initialBalance) {
        super();
        setAggregateId(aggregateId);
        this.accountNumber = accountNumber;
        this.ownerName = ownerName;
        this.initialBalance = initialBalance;
    }
}

@Entity
@Table(name = "account_events")
@Data
@EqualsAndHashCode(callSuper = true)
public class MoneyDepositedEvent extends DomainEvent {
    private BigDecimal amount;
    private String description;

    public MoneyDepositedEvent(String aggregateId, BigDecimal amount, String description) {
        super();
        setAggregateId(aggregateId);
        this.amount = amount;
        this.description = description;
    }
}

@Entity
@Table(name = "account_events")
@Data
@EqualsAndHashCode(callSuper = true)
public class MoneyWithdrawnEvent extends DomainEvent {
    private BigDecimal amount;
    private String description;

    public MoneyWithdrawnEvent(String aggregateId, BigDecimal amount, String description) {
        super();
        setAggregateId(aggregateId);
        this.amount = amount;
        this.description = description;
    }
}

@Entity
@Table(name = "account_events")
@Data
@EqualsAndHashCode(callSuper = true)
public class AccountClosedEvent extends DomainEvent {
    private String reason;

    public AccountClosedEvent(String aggregateId, String reason) {
        super();
        setAggregateId(aggregateId);
        this.reason = reason;
    }
}

// Aggregate Root
@Data
public class Account {
    private String accountId;
    private String accountNumber;
    private String ownerName;
    private BigDecimal balance;
    private AccountStatus status;
    private Long version;

    private List<DomainEvent> uncommittedEvents = new ArrayList<>();

    // Constructor for new account
    public Account(String accountNumber, String ownerName, BigDecimal initialBalance) {
        this.accountId = UUID.randomUUID().toString();
        this.version = 0L;

        // Create and apply event
        AccountCreatedEvent event = new AccountCreatedEvent(
            accountId, accountNumber, ownerName, initialBalance
        );
        applyEvent(event);
    }

    // Constructor for rebuilding from events
    public Account(String accountId, List<DomainEvent> events) {
        this.accountId = accountId;
        this.version = 0L;

        // Replay all events
        events.forEach(this::applyEvent);
    }

    // Business Methods
    public void deposit(BigDecimal amount, String description) {
        if (status != AccountStatus.ACTIVE) {
            throw new AccountNotActiveException(accountId);
        }

        if (amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new InvalidAmountException("Deposit amount must be positive");
        }

        MoneyDepositedEvent event = new MoneyDepositedEvent(accountId, amount, description);
        applyEvent(event);
    }

    public void withdraw(BigDecimal amount, String description) {
        if (status != AccountStatus.ACTIVE) {
            throw new AccountNotActiveException(accountId);
        }

        if (amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new InvalidAmountException("Withdrawal amount must be positive");
        }

        if (balance.compareTo(amount) < 0) {
            throw new InsufficientFundsException(accountId, balance, amount);
        }

        MoneyWithdrawnEvent event = new MoneyWithdrawnEvent(accountId, amount, description);
        applyEvent(event);
    }

    public void close(String reason) {
        if (status == AccountStatus.CLOSED) {
            throw new AccountAlreadyClosedException(accountId);
        }

        if (balance.compareTo(BigDecimal.ZERO) != 0) {
            throw new AccountHasBalanceException(accountId, balance);
        }

        AccountClosedEvent event = new AccountClosedEvent(accountId, reason);
        applyEvent(event);
    }

    // Apply Event and add to uncommitted events
    private void applyEvent(DomainEvent event) {
        event.setVersion(++version);
        apply(event);
        uncommittedEvents.add(event);
    }

    // Apply Event (State Change)
    private void apply(DomainEvent event) {
        if (event instanceof AccountCreatedEvent) {
            apply((AccountCreatedEvent) event);
        } else if (event instanceof MoneyDepositedEvent) {
            apply((MoneyDepositedEvent) event);
        } else if (event instanceof MoneyWithdrawnEvent) {
            apply((MoneyWithdrawnEvent) event);
        } else if (event instanceof AccountClosedEvent) {
            apply((AccountClosedEvent) event);
        }
    }

    private void apply(AccountCreatedEvent event) {
        this.accountNumber = event.getAccountNumber();
        this.ownerName = event.getOwnerName();
        this.balance = event.getInitialBalance();
        this.status = AccountStatus.ACTIVE;
    }

    private void apply(MoneyDepositedEvent event) {
        this.balance = this.balance.add(event.getAmount());
    }

    private void apply(MoneyWithdrawnEvent event) {
        this.balance = this.balance.subtract(event.getAmount());
    }

    private void apply(AccountClosedEvent event) {
        this.status = AccountStatus.CLOSED;
    }

    public List<DomainEvent> getUncommittedEvents() {
        return new ArrayList<>(uncommittedEvents);
    }

    public void markEventsAsCommitted() {
        uncommittedEvents.clear();
    }
}

// Event Store
@Repository
public interface EventStoreRepository extends JpaRepository<DomainEvent, String> {
    List<DomainEvent> findByAggregateIdOrderByVersionAsc(String aggregateId);
    Long countByAggregateId(String aggregateId);
}

// Event Store Service
@Service
public class EventStore {

    @Autowired
    private EventStoreRepository repository;

    @Autowired
    private ApplicationEventPublisher eventPublisher;

    @Transactional
    public void saveEvents(String aggregateId, List<DomainEvent> events, Long expectedVersion) {
        // Optimistic Concurrency Check
        Long currentVersion = repository.countByAggregateId(aggregateId);

        if (!currentVersion.equals(expectedVersion)) {
            throw new ConcurrencyException(
                String.format("Aggregate %s has been modified. Expected version: %d, Current: %d",
                    aggregateId, expectedVersion, currentVersion)
            );
        }

        // Save events
        repository.saveAll(events);

        // Publish events for projections/read models
        events.forEach(eventPublisher::publishEvent);
    }

    @Transactional(readOnly = true)
    public List<DomainEvent> getEvents(String aggregateId) {
        return repository.findByAggregateIdOrderByVersionAsc(aggregateId);
    }

    @Transactional(readOnly = true)
    public Account getAggregate(String aggregateId) {
        List<DomainEvent> events = getEvents(aggregateId);

        if (events.isEmpty()) {
            throw new AggregateNotFoundException(aggregateId);
        }

        return new Account(aggregateId, events);
    }
}

// Account Service
@Service
public class AccountService {

    @Autowired
    private EventStore eventStore;

    public String createAccount(String accountNumber, String ownerName, BigDecimal initialBalance) {
        Account account = new Account(accountNumber, ownerName, initialBalance);

        eventStore.saveEvents(
            account.getAccountId(),
            account.getUncommittedEvents(),
            0L
        );

        account.markEventsAsCommitted();
        return account.getAccountId();
    }

    public void deposit(String accountId, BigDecimal amount, String description) {
        Account account = eventStore.getAggregate(accountId);
        Long currentVersion = account.getVersion();

        account.deposit(amount, description);

        eventStore.saveEvents(
            accountId,
            account.getUncommittedEvents(),
            currentVersion
        );

        account.markEventsAsCommitted();
    }

    public void withdraw(String accountId, BigDecimal amount, String description) {
        Account account = eventStore.getAggregate(accountId);
        Long currentVersion = account.getVersion();

        account.withdraw(amount, description);

        eventStore.saveEvents(
            accountId,
            account.getUncommittedEvents(),
            currentVersion
        );

        account.markEventsAsCommitted();
    }

    public Account getAccount(String accountId) {
        return eventStore.getAggregate(accountId);
    }

    public List<DomainEvent> getAccountHistory(String accountId) {
        return eventStore.getEvents(accountId);
    }
}

// Read Model (Projection)
@Entity
@Table(name = "account_read_model")
@Data
public class AccountReadModel {
    @Id
    private String accountId;
    private String accountNumber;
    private String ownerName;
    private BigDecimal balance;
    private AccountStatus status;
    private LocalDateTime createdAt;
    private LocalDateTime lastTransactionAt;
    private Integer transactionCount;
}

// Projection Builder
@Component
public class AccountProjection {

    @Autowired
    private AccountReadModelRepository readModelRepository;

    @EventListener
    @Transactional
    public void on(AccountCreatedEvent event) {
        AccountReadModel readModel = new AccountReadModel();
        readModel.setAccountId(event.getAggregateId());
        readModel.setAccountNumber(event.getAccountNumber());
        readModel.setOwnerName(event.getOwnerName());
        readModel.setBalance(event.getInitialBalance());
        readModel.setStatus(AccountStatus.ACTIVE);
        readModel.setCreatedAt(event.getOccurredAt());
        readModel.setTransactionCount(0);

        readModelRepository.save(readModel);
    }

    @EventListener
    @Transactional
    public void on(MoneyDepositedEvent event) {
        AccountReadModel readModel = readModelRepository
            .findById(event.getAggregateId())
            .orElseThrow();

        readModel.setBalance(readModel.getBalance().add(event.getAmount()));
        readModel.setLastTransactionAt(event.getOccurredAt());
        readModel.setTransactionCount(readModel.getTransactionCount() + 1);

        readModelRepository.save(readModel);
    }

    @EventListener
    @Transactional
    public void on(MoneyWithdrawnEvent event) {
        AccountReadModel readModel = readModelRepository
            .findById(event.getAggregateId())
            .orElseThrow();

        readModel.setBalance(readModel.getBalance().subtract(event.getAmount()));
        readModel.setLastTransactionAt(event.getOccurredAt());
        readModel.setTransactionCount(readModel.getTransactionCount() + 1);

        readModelRepository.save(readModel);
    }

    @EventListener
    @Transactional
    public void on(AccountClosedEvent event) {
        AccountReadModel readModel = readModelRepository
            .findById(event.getAggregateId())
            .orElseThrow();

        readModel.setStatus(AccountStatus.CLOSED);
        readModelRepository.save(readModel);
    }
}

// Snapshots for Performance
@Entity
@Table(name = "account_snapshots")
@Data
public class AccountSnapshot {
    @Id
    private String snapshotId;
    private String aggregateId;
    private Long version;
    private String accountNumber;
    private String ownerName;
    private BigDecimal balance;
    private AccountStatus status;
    private LocalDateTime createdAt;
}

@Service
public class SnapshotService {

    @Autowired
    private SnapshotRepository snapshotRepository;

    @Autowired
    private EventStore eventStore;

    private static final int SNAPSHOT_FREQUENCY = 50; // كل 50 حدث

    public Account loadAggregate(String aggregateId) {
        // Load latest snapshot
        Optional<AccountSnapshot> snapshotOpt = snapshotRepository
            .findTopByAggregateIdOrderByVersionDesc(aggregateId);

        if (snapshotOpt.isPresent()) {
            AccountSnapshot snapshot = snapshotOpt.get();

            // Load events after snapshot
            List<DomainEvent> events = eventStore.getEventsAfterVersion(
                aggregateId,
                snapshot.getVersion()
            );

            // Rebuild from snapshot + new events
            Account account = fromSnapshot(snapshot);
            events.forEach(account::apply);

            return account;
        } else {
            // No snapshot, load all events
            return eventStore.getAggregate(aggregateId);
        }
    }

    @Async
    public void createSnapshotIfNeeded(String aggregateId, Long version) {
        if (version % SNAPSHOT_FREQUENCY == 0) {
            Account account = eventStore.getAggregate(aggregateId);
            saveSnapshot(account);
        }
    }

    private void saveSnapshot(Account account) {
        AccountSnapshot snapshot = new AccountSnapshot();
        snapshot.setSnapshotId(UUID.randomUUID().toString());
        snapshot.setAggregateId(account.getAccountId());
        snapshot.setVersion(account.getVersion());
        snapshot.setAccountNumber(account.getAccountNumber());
        snapshot.setOwnerName(account.getOwnerName());
        snapshot.setBalance(account.getBalance());
        snapshot.setStatus(account.getStatus());
        snapshot.setCreatedAt(LocalDateTime.now());

        snapshotRepository.save(snapshot);
    }

    private Account fromSnapshot(AccountSnapshot snapshot) {
        Account account = new Account();
        account.setAccountId(snapshot.getAggregateId());
        account.setVersion(snapshot.getVersion());
        account.setAccountNumber(snapshot.getAccountNumber());
        account.setOwnerName(snapshot.getOwnerName());
        account.setBalance(snapshot.getBalance());
        account.setStatus(snapshot.getStatus());
        return account;
    }
}
```

**المزايا:**
1. **التدقيق الكامل**: سجل كامل لكل التغييرات
2. **السفر عبر الزمن**: إعادة بناء الحالة في أي نقطة زمنية
3. **تصحيح الأخطاء**: سهولة تتبع الأخطاء
4. **Event-Driven**: سهولة بناء Projections

**العيوب:**
1. **التعقيد**: منحنى تعلم عالٍ
2. **الأداء**: إعادة بناء الحالة قد تكون بطيئة (يُحل بـ Snapshots)
3. **التخزين**: حجم تخزين أكبر

---

## التواصل بين الخدمات

### س10: ما هي طرق التواصل بين الخدمات؟
**الجواب:**

#### 1. REST API (Synchronous)

**المزايا:**
- بسيط وشائع
- قابل للتخزين المؤقت (HTTP Caching)
- دعم واسع

**العيوب:**
- Coupling عالي
- Latency في الطلبات المتعددة
- معالجة الأخطاء معقدة

**مثال:**

```java
@Service
public class OrderService {

    @Autowired
    private RestTemplate restTemplate;

    @Autowired
    private WebClient.Builder webClientBuilder;

    // Using RestTemplate (Blocking)
    public Order createOrderWithRestTemplate(OrderRequest request) {
        // Get User
        User user = restTemplate.getForObject(
            "http://USER-SERVICE/users/" + request.getUserId(),
            User.class
        );

        // Get Product
        Product product = restTemplate.getForObject(
            "http://PRODUCT-SERVICE/products/" + request.getProductId(),
            Product.class
        );

        // Process Payment
        Payment payment = restTemplate.postForObject(
            "http://PAYMENT-SERVICE/payments",
            request.getPaymentInfo(),
            Payment.class
        );

        // Create Order
        Order order = new Order(user, product, payment);
        return orderRepository.save(order);
    }

    // Using WebClient (Non-Blocking/Reactive)
    public Mono<Order> createOrderWithWebClient(OrderRequest request) {
        WebClient webClient = webClientBuilder.build();

        // Parallel calls using Mono.zip
        Mono<User> userMono = webClient.get()
            .uri("http://USER-SERVICE/users/" + request.getUserId())
            .retrieve()
            .bodyToMono(User.class);

        Mono<Product> productMono = webClient.get()
            .uri("http://PRODUCT-SERVICE/products/" + request.getProductId())
            .retrieve()
            .bodyToMono(Product.class);

        Mono<Payment> paymentMono = webClient.post()
            .uri("http://PAYMENT-SERVICE/payments")
            .bodyValue(request.getPaymentInfo())
            .retrieve()
            .bodyToMono(Payment.class);

        return Mono.zip(userMono, productMono, paymentMono)
            .map(tuple -> {
                Order order = new Order(tuple.getT1(), tuple.getT2(), tuple.getT3());
                return orderRepository.save(order);
            });
    }
}

// Error Handling
@ControllerAdvice
public class RestClientErrorHandler {

    @ExceptionHandler(HttpClientErrorException.class)
    public ResponseEntity<ErrorResponse> handleClientError(HttpClientErrorException ex) {
        ErrorResponse error = new ErrorResponse(
            ex.getStatusCode().value(),
            ex.getMessage()
        );
        return ResponseEntity.status(ex.getStatusCode()).body(error);
    }

    @ExceptionHandler(HttpServerErrorException.class)
    public ResponseEntity<ErrorResponse> handleServerError(HttpServerErrorException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.SERVICE_UNAVAILABLE.value(),
            "External service is currently unavailable"
        );
        return ResponseEntity.status(HttpStatus.SERVICE_UNAVAILABLE).body(error);
    }
}
```

#### 2. gRPC (Synchronous - High Performance)

**المزايا:**
- أداء عالي (Protocol Buffers)
- Streaming support
- Strong typing

**العيوب:**
- منحنى تعلم أعلى
- HTTP/2 required
- أقل قابلية للقراءة البشرية

**مثال:**

```protobuf
// product.proto
syntax = "proto3";

package com.example.product;

service ProductService {
    rpc GetProduct (GetProductRequest) returns (ProductResponse);
    rpc ListProducts (ListProductsRequest) returns (stream ProductResponse);
    rpc CreateProduct (CreateProductRequest) returns (ProductResponse);
    rpc UpdateProduct (UpdateProductRequest) returns (ProductResponse);
}

message GetProductRequest {
    int64 id = 1;
}

message ProductResponse {
    int64 id = 1;
    string name = 2;
    string description = 3;
    double price = 4;
    int32 stock = 5;
}

message ListProductsRequest {
    int32 page = 1;
    int32 size = 2;
}

message CreateProductRequest {
    string name = 1;
    string description = 2;
    double price = 3;
    int32 stock = 4;
}

message UpdateProductRequest {
    int64 id = 1;
    string name = 2;
    string description = 3;
    double price = 4;
    int32 stock = 5;
}
```

```java
// Server Implementation
@GrpcService
public class ProductGrpcService extends ProductServiceGrpc.ProductServiceImplBase {

    @Autowired
    private ProductService productService;

    @Override
    public void getProduct(GetProductRequest request,
                          StreamObserver<ProductResponse> responseObserver) {
        try {
            Product product = productService.findById(request.getId());

            ProductResponse response = ProductResponse.newBuilder()
                .setId(product.getId())
                .setName(product.getName())
                .setDescription(product.getDescription())
                .setPrice(product.getPrice().doubleValue())
                .setStock(product.getStock())
                .build();

            responseObserver.onNext(response);
            responseObserver.onCompleted();

        } catch (ProductNotFoundException e) {
            responseObserver.onError(
                Status.NOT_FOUND
                    .withDescription("Product not found: " + request.getId())
                    .asRuntimeException()
            );
        } catch (Exception e) {
            responseObserver.onError(
                Status.INTERNAL
                    .withDescription("Internal error: " + e.getMessage())
                    .asRuntimeException()
            );
        }
    }

    @Override
    public void listProducts(ListProductsRequest request,
                            StreamObserver<ProductResponse> responseObserver) {
        List<Product> products = productService.findAll(request.getPage(), request.getSize());

        products.forEach(product -> {
            ProductResponse response = ProductResponse.newBuilder()
                .setId(product.getId())
                .setName(product.getName())
                .setDescription(product.getDescription())
                .setPrice(product.getPrice().doubleValue())
                .setStock(product.getStock())
                .build();

            responseObserver.onNext(response);
        });

        responseObserver.onCompleted();
    }
}

// Client Implementation
@Service
public class ProductGrpcClient {

    @GrpcClient("product-service")
    private ProductServiceGrpc.ProductServiceBlockingStub productServiceStub;

    public Product getProduct(Long id) {
        GetProductRequest request = GetProductRequest.newBuilder()
            .setId(id)
            .build();

        try {
            ProductResponse response = productServiceStub.getProduct(request);

            return Product.builder()
                .id(response.getId())
                .name(response.getName())
                .description(response.getDescription())
                .price(BigDecimal.valueOf(response.getPrice()))
                .stock(response.getStock())
                .build();

        } catch (StatusRuntimeException e) {
            if (e.getStatus().getCode() == Status.Code.NOT_FOUND) {
                throw new ProductNotFoundException(id);
            }
            throw new GrpcServiceException("Failed to get product", e);
        }
    }

    public List<Product> listProducts(int page, int size) {
        ListProductsRequest request = ListProductsRequest.newBuilder()
            .setPage(page)
            .setSize(size)
            .build();

        List<Product> products = new ArrayList<>();

        Iterator<ProductResponse> iterator = productServiceStub.listProducts(request);

        iterator.forEachRemaining(response -> {
            products.add(Product.builder()
                .id(response.getId())
                .name(response.getName())
                .description(response.getDescription())
                .price(BigDecimal.valueOf(response.getPrice()))
                .stock(response.getStock())
                .build());
        });

        return products;
    }
}
```

#### 3. Message Queues (Asynchronous)

**RabbitMQ Example:**

```java
// Configuration
@Configuration
public class RabbitMQConfig {

    // Direct Exchange
    @Bean
    public Queue orderQueue() {
        return QueueBuilder.durable("order.queue")
            .withArgument("x-message-ttl", 60000) // 60 seconds TTL
            .withArgument("x-dead-letter-exchange", "dlx.exchange")
            .build();
    }

    @Bean
    public DirectExchange orderExchange() {
        return new DirectExchange("order.exchange");
    }

    @Bean
    public Binding orderBinding() {
        return BindingBuilder
            .bind(orderQueue())
            .to(orderExchange())
            .with("order.created");
    }

    // Topic Exchange
    @Bean
    public TopicExchange notificationExchange() {
        return new TopicExchange("notification.exchange");
    }

    @Bean
    public Queue emailQueue() {
        return new Queue("notification.email.queue", true);
    }

    @Bean
    public Queue smsQueue() {
        return new Queue("notification.sms.queue", true);
    }

    @Bean
    public Binding emailBinding() {
        return BindingBuilder
            .bind(emailQueue())
            .to(notificationExchange())
            .with("notification.email.#");
    }

    @Bean
    public Binding smsBinding() {
        return BindingBuilder
            .bind(smsQueue())
            .to(notificationExchange())
            .with("notification.sms.#");
    }

    // Dead Letter Queue
    @Bean
    public Queue deadLetterQueue() {
        return new Queue("dlq.queue", true);
    }

    @Bean
    public DirectExchange deadLetterExchange() {
        return new DirectExchange("dlx.exchange");
    }

    @Bean
    public Binding deadLetterBinding() {
        return BindingBuilder
            .bind(deadLetterQueue())
            .to(deadLetterExchange())
            .with("");
    }

    // Message Converter
    @Bean
    public MessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }
}

// Publisher
@Service
@Slf4j
public class OrderEventPublisher {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void publishOrderCreated(Order order) {
        OrderCreatedEvent event = new OrderCreatedEvent(
            order.getId(),
            order.getUserId(),
            order.getItems(),
            order.getTotalAmount(),
            LocalDateTime.now()
        );

        log.info("Publishing OrderCreatedEvent: {}", event);

        rabbitTemplate.convertAndSend(
            "order.exchange",
            "order.created",
            event,
            message -> {
                message.getMessageProperties().setContentType("application/json");
                message.getMessageProperties().setHeader("priority", 5);
                return message;
            }
        );
    }

    public void publishPaymentRequired(Long orderId, BigDecimal amount) {
        PaymentRequiredEvent event = new PaymentRequiredEvent(orderId, amount);

        rabbitTemplate.convertAndSend(
            "order.exchange",
            "payment.required",
            event
        );
    }
}

// Consumer
@Component
@Slf4j
public class OrderEventListener {

    @Autowired
    private PaymentService paymentService;

    @Autowired
    private InventoryService inventoryService;

    @Autowired
    private NotificationService notificationService;

    @RabbitListener(queues = "order.queue")
    public void handleOrderCreated(OrderCreatedEvent event) {
        log.info("Received OrderCreatedEvent: {}", event);

        try {
            // Process order
            inventoryService.reserveItems(event.getItems());
            paymentService.initiatePayment(event.getOrderId(), event.getTotalAmount());

            log.info("Order processed successfully: {}", event.getOrderId());

        } catch (Exception e) {
            log.error("Failed to process order: {}", event.getOrderId(), e);
            throw new AmqpRejectAndDontRequeueException("Processing failed", e);
        }
    }

    @RabbitListener(queues = "notification.email.queue")
    public void handleEmailNotification(NotificationEvent event) {
        log.info("Sending email notification: {}", event);
        notificationService.sendEmail(event);
    }

    @RabbitListener(queues = "notification.sms.queue")
    public void handleSmsNotification(NotificationEvent event) {
        log.info("Sending SMS notification: {}", event);
        notificationService.sendSms(event);
    }

    // Dead Letter Queue Handler
    @RabbitListener(queues = "dlq.queue")
    public void handleDeadLetterQueue(Message message) {
        log.error("Message in DLQ: {}", new String(message.getBody()));
        // Log to monitoring system, alert team, etc.
    }
}

// Retry and Error Handling
@Configuration
public class RabbitMQRetryConfig {

    @Bean
    public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory(
            ConnectionFactory connectionFactory) {

        SimpleRabbitListenerContainerFactory factory =
            new SimpleRabbitListenerContainerFactory();

        factory.setConnectionFactory(connectionFactory);
        factory.setMessageConverter(new Jackson2JsonMessageConverter());
        factory.setDefaultRequeueRejected(false);
        factory.setAdviceChain(retryInterceptor());

        return factory;
    }

    @Bean
    public RetryOperationsInterceptor retryInterceptor() {
        return RetryInterceptorBuilder.stateless()
            .maxAttempts(3)
            .backOffOptions(1000, 2.0, 10000) // initial, multiplier, max
            .recoverer(new RejectAndDontRequeueRecoverer())
            .build();
    }
}
```

**Apache Kafka Example:**

```java
// Configuration
@Configuration
public class KafkaConfig {

    @Bean
    public ProducerFactory<String, Object> producerFactory() {
        Map<String, Object> config = new HashMap<>();
        config.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        config.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        config.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);
        config.put(ProducerConfig.ACKS_CONFIG, "all");
        config.put(ProducerConfig.RETRIES_CONFIG, 3);
        config.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true);

        return new DefaultKafkaProducerFactory<>(config);
    }

    @Bean
    public KafkaTemplate<String, Object> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }

    @Bean
    public ConsumerFactory<String, Object> consumerFactory() {
        Map<String, Object> config = new HashMap<>();
        config.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        config.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        config.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class);
        config.put(ConsumerConfig.GROUP_ID_CONFIG, "order-service-group");
        config.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        config.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
        config.put(JsonDeserializer.TRUSTED_PACKAGES, "*");

        return new DefaultKafkaConsumerFactory<>(config);
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, Object> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, Object> factory =
            new ConcurrentKafkaListenerContainerFactory<>();

        factory.setConsumerFactory(consumerFactory());
        factory.setConcurrency(3); // 3 threads
        factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.MANUAL);

        return factory;
    }

    // Topic Configuration
    @Bean
    public NewTopic orderTopic() {
        return TopicBuilder.name("order-events")
            .partitions(3)
            .replicas(2)
            .config(TopicConfig.RETENTION_MS_CONFIG, "86400000") // 24 hours
            .build();
    }
}

// Producer
@Service
@Slf4j
public class KafkaEventPublisher {

    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;

    public CompletableFuture<SendResult<String, Object>> publishOrderEvent(OrderEvent event) {
        log.info("Publishing event to Kafka: {}", event);

        return kafkaTemplate.send("order-events", event.getOrderId().toString(), event)
            .whenComplete((result, ex) -> {
                if (ex != null) {
                    log.error("Failed to publish event: {}", event, ex);
                } else {
                    log.info("Event published successfully: offset={}, partition={}",
                        result.getRecordMetadata().offset(),
                        result.getRecordMetadata().partition());
                }
            });
    }
}

// Consumer
@Component
@Slf4j
public class KafkaEventListener {

    @KafkaListener(
        topics = "order-events",
        groupId = "payment-service-group",
        containerFactory = "kafkaListenerContainerFactory"
    )
    public void handleOrderEvent(
            @Payload OrderEvent event,
            @Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition,
            @Header(KafkaHeaders.OFFSET) long offset,
            Acknowledgment acknowledgment) {

        log.info("Received event: partition={}, offset={}, event={}",
            partition, offset, event);

        try {
            // Process event
            processEvent(event);

            // Manual acknowledgment
            acknowledgment.acknowledge();
            log.info("Event processed successfully");

        } catch (Exception e) {
            log.error("Failed to process event", e);
            // Don't acknowledge - message will be reprocessed
        }
    }
}
```

**المقارنة:**

| الطريقة | متى تُستخدم | المزايا | العيوب |
|---------|-------------|---------|--------|
| **REST** | عمليات synchronous، CRUD | بسيط، شائع | Coupling، Latency |
| **gRPC** | High performance، Streaming | سريع، Strong typing | منحنى تعلم |
| **Message Queue** | Async، Event-driven | Decoupling، المرونة | Eventual consistency |

---

## Service Discovery

### س11: ما هو Service Discovery ولماذا نحتاجه؟
**الجواب:**

**التعريف:**
Service Discovery هو آلية تسمح للخدمات باكتشاف بعضها البعض تلقائياً في بيئة ديناميكية حيث عناوين IP والمنافذ تتغير باستمرار.

**المشكلة:**

```
سيناريو بدون Service Discovery:
- Order Service يحتاج استدعاء Payment Service
- Payment Service على: http://192.168.1.10:8080
- ماذا لو تم نقل Payment Service إلى خادم آخر؟
- ماذا لو تم توسيع Payment Service إلى 5 نسخ؟
→ صيانة يدوية مكلفة وعرضة للأخطاء
```

**الحل - Service Discovery:**

```
                    ┌──────────────────┐
                    │ Service Registry │
                    │    (Eureka)      │
                    └────────┬─────────┘
                             │
              ┌──────────────┼──────────────┐
              │ Register     │ Discover     │
              │              │              │
         ┌────▼────┐    ┌───▼────┐    ┌───▼────┐
         │Service A│    │Service │    │Service │
         │Instance1│    │   B    │    │   C    │
         └─────────┘    └────────┘    └────────┘
         ┌─────────┐
         │Service A│
         │Instance2│
         └─────────┘
```

### س12: اشرح Netflix Eureka
**الجواب:**

**Eureka Server:**

```java
// Eureka Server Application
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}

// application.yml
server:
  port: 8761

spring:
  application:
    name: eureka-server

eureka:
  instance:
    hostname: localhost
  client:
    register-with-eureka: false  # لا تسجل نفسك
    fetch-registry: false         # لا تجلب السجل
  server:
    enable-self-preservation: true
    eviction-interval-timer-in-ms: 60000
```

**Eureka Client (Microservice):**

```java
// User Service
@SpringBootApplication
@EnableEurekaClient
// أو @EnableDiscoveryClient (أكثر عمومية)
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}

// application.yml
spring:
  application:
    name: user-service

server:
  port: ${PORT:0}  # منفذ عشوائي للسماح بنسخ متعددة

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
    registry-fetch-interval-seconds: 30
  instance:
    prefer-ip-address: true
    instance-id: ${spring.application.name}:${random.value}
    lease-renewal-interval-in-seconds: 30    # Heartbeat
    lease-expiration-duration-in-seconds: 90 # إذا لم يرسل heartbeat
    metadata-map:
      zone: zone1
      version: 1.0.0
```

**استخدام Eureka للاتصال:**

```java
@Service
public class OrderService {

    @Autowired
    private DiscoveryClient discoveryClient;

    @Autowired
    @LoadBalanced  // مهم للتوازن
    private RestTemplate restTemplate;

    // الطريقة 1: استخدام DiscoveryClient مباشرة
    public User getUserManually(Long userId) {
        List<ServiceInstance> instances =
            discoveryClient.getInstances("user-service");

        if (instances.isEmpty()) {
            throw new ServiceNotFoundException("user-service");
        }

        // اختيار instance (يدوياً - ليس موصى به)
        ServiceInstance instance = instances.get(0);
        String url = instance.getUri() + "/users/" + userId;

        return restTemplate.getForObject(url, User.class);
    }

    // الطريقة 2: استخدام Load Balancer (موصى به)
    public User getUser(Long userId) {
        // اسم الخدمة بدلاً من URL
        return restTemplate.getForObject(
            "http://USER-SERVICE/users/" + userId,
            User.class
        );
    }

    // الطريقة 3: باستخدام Feign Client (الأفضل)
    @Autowired
    private UserFeignClient userClient;

    public User getUserWithFeign(Long userId) {
        return userClient.getUser(userId);
    }

    // معلومات عن الخدمات المسجلة
    public Map<String, List<ServiceInstance>> getAllServices() {
        Map<String, List<ServiceInstance>> services = new HashMap<>();

        discoveryClient.getServices().forEach(serviceName -> {
            List<ServiceInstance> instances =
                discoveryClient.getInstances(serviceName);
            services.put(serviceName, instances);
        });

        return services;
    }
}

// Feign Client
@FeignClient(name = "user-service")
public interface UserFeignClient {

    @GetMapping("/users/{id}")
    User getUser(@PathVariable("id") Long id);

    @PostMapping("/users")
    User createUser(@RequestBody UserRequest request);

    @GetMapping("/users")
    List<User> getAllUsers();
}
```

**Health Checks:**

```java
@Component
public class CustomHealthIndicator implements HealthIndicator {

    @Autowired
    private DataSource dataSource;

    @Override
    public Health health() {
        try {
            // فحص الاتصال بقاعدة البيانات
            Connection connection = dataSource.getConnection();
            connection.close();

            return Health.up()
                .withDetail("database", "Available")
                .withDetail("status", "Healthy")
                .build();

        } catch (Exception e) {
            return Health.down()
                .withDetail("database", "Unavailable")
                .withDetail("error", e.getMessage())
                .build();
        }
    }
}

// application.yml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics
  endpoint:
    health:
      show-details: always
  health:
    circuitbreakers:
      enabled: true

info:
  app:
    name: User Service
    version: 1.0.0
    description: User management microservice
```

### س13: ما الفرق بين Eureka و Consul؟
**الجواب:**

**Consul Configuration:**

```java
// Consul Client Configuration
@Configuration
public class ConsulConfig {

    @Bean
    public ConsulClient consulClient() {
        return new ConsulClient("localhost", 8500);
    }
}

// Service Registration
@Component
public class ConsulServiceRegistration implements ApplicationListener<WebServerInitializedEvent> {

    @Autowired
    private ConsulClient consulClient;

    @Value("${spring.application.name}")
    private String serviceName;

    @Override
    public void onApplicationEvent(WebServerInitializedEvent event) {
        int port = event.getWebServer().getPort();

        NewService newService = new NewService();
        newService.setId(serviceName + "-" + port);
        newService.setName(serviceName);
        newService.setPort(port);
        newService.setAddress("localhost");

        // Health Check
        NewService.Check check = new NewService.Check();
        check.setHttp("http://localhost:" + port + "/actuator/health");
        check.setInterval("10s");
        check.setTimeout("5s");
        newService.setCheck(check);

        consulClient.agentServiceRegister(newService);
    }
}

// Service Discovery
@Service
public class ConsulServiceDiscovery {

    @Autowired
    private ConsulClient consulClient;

    public List<ServiceHealth> getHealthyServices(String serviceName) {
        HealthServicesRequest request = HealthServicesRequest.newBuilder()
            .setPassing(true)  // فقط الخدمات الصحية
            .build();

        Response<List<HealthService>> response =
            consulClient.getHealthServices(serviceName, request);

        return response.getValue().stream()
            .map(HealthService::getService)
            .collect(Collectors.toList());
    }

    public String getServiceUrl(String serviceName) {
        List<ServiceHealth> services = getHealthyServices(serviceName);

        if (services.isEmpty()) {
            throw new ServiceNotFoundException(serviceName);
        }

        // اختيار instance عشوائية (يمكن تحسينها)
        ServiceHealth service = services.get(
            new Random().nextInt(services.size())
        );

        return String.format("http://%s:%d",
            service.getAddress(),
            service.getPort());
    }
}
```

**المقارنة:**

| الميزة | Eureka | Consul |
|--------|--------|--------|
| **المطور** | Netflix | HashiCorp |
| **اللغة** | Java | Go |
| **الاتساق** | AP (Eventually Consistent) | CP (Strongly Consistent) |
| **Health Checks** | محدودة | متقدمة جداً |
| **Key-Value Store** | لا | نعم |
| **DNS Support** | لا | نعم |
| **Multi-Datacenter** | يحتاج إعداد | مدمج |
| **UI** | بسيط | متقدم |

---

## API Gateway

### س14: ما هي مسؤوليات API Gateway؟
**الجواب:**

**المسؤوليات الأساسية:**

#### 1. Request Routing (توجيه الطلبات)

```java
@Configuration
public class GatewayRouting {

    @Bean
    public RouteLocator customRoutes(RouteLocatorBuilder builder) {
        return builder.routes()
            // Path-based routing
            .route("user-route", r -> r
                .path("/api/v1/users/**")
                .uri("lb://USER-SERVICE"))

            // Header-based routing
            .route("mobile-route", r -> r
                .path("/api/**")
                .and()
                .header("X-Client-Type", "mobile")
                .uri("lb://MOBILE-API-SERVICE"))

            // Query parameter routing
            .route("beta-route", r -> r
                .path("/api/**")
                .and()
                .query("version", "beta")
                .uri("lb://BETA-SERVICE"))

            // Method-based routing
            .route("write-route", r -> r
                .path("/api/products/**")
                .and()
                .method(HttpMethod.POST, HttpMethod.PUT, HttpMethod.DELETE)
                .uri("lb://PRODUCT-WRITE-SERVICE"))

            .route("read-route", r -> r
                .path("/api/products/**")
                .and()
                .method(HttpMethod.GET)
                .uri("lb://PRODUCT-READ-SERVICE"))

            .build();
    }
}
```

#### 2. Authentication & Authorization

```java
@Component
public class AuthenticationGatewayFilter implements GlobalFilter, Ordered {

    @Autowired
    private JwtTokenProvider jwtTokenProvider;

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();

        // Public endpoints - skip authentication
        if (isPublicEndpoint(request.getPath().toString())) {
            return chain.filter(exchange);
        }

        // Get token from header
        String token = extractToken(request);

        if (token == null) {
            return onError(exchange, "Missing authentication token", HttpStatus.UNAUTHORIZED);
        }

        try {
            // Validate token
            if (!jwtTokenProvider.validateToken(token)) {
                return onError(exchange, "Invalid token", HttpStatus.UNAUTHORIZED);
            }

            // Check if token is blacklisted (logged out)
            if (isTokenBlacklisted(token)) {
                return onError(exchange, "Token has been revoked", HttpStatus.UNAUTHORIZED);
            }

            // Extract user info
            String userId = jwtTokenProvider.getUserId(token);
            List<String> roles = jwtTokenProvider.getRoles(token);

            // Add user info to request headers
            ServerHttpRequest mutatedRequest = request.mutate()
                .header("X-User-ID", userId)
                .header("X-User-Roles", String.join(",", roles))
                .build();

            // Check authorization for specific endpoints
            if (!hasRequiredRole(request.getPath().toString(), roles)) {
                return onError(exchange, "Insufficient permissions", HttpStatus.FORBIDDEN);
            }

            return chain.filter(exchange.mutate().request(mutatedRequest).build());

        } catch (Exception e) {
            return onError(exchange, "Authentication failed: " + e.getMessage(),
                          HttpStatus.UNAUTHORIZED);
        }
    }

    private String extractToken(ServerHttpRequest request) {
        String bearerToken = request.getHeaders().getFirst("Authorization");

        if (bearerToken != null && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);
        }

        return null;
    }

    private boolean isPublicEndpoint(String path) {
        List<String> publicPaths = Arrays.asList(
            "/api/auth/login",
            "/api/auth/register",
            "/api/public/**",
            "/actuator/health"
        );

        return publicPaths.stream()
            .anyMatch(p -> new AntPathMatcher().match(p, path));
    }

    private boolean isTokenBlacklisted(String token) {
        return redisTemplate.hasKey("blacklist:" + token);
    }

    private boolean hasRequiredRole(String path, List<String> userRoles) {
        // Admin endpoints
        if (path.startsWith("/api/admin")) {
            return userRoles.contains("ADMIN");
        }

        // Manager endpoints
        if (path.startsWith("/api/management")) {
            return userRoles.contains("ADMIN") || userRoles.contains("MANAGER");
        }

        return true; // authenticated users
    }

    private Mono<Void> onError(ServerWebExchange exchange, String message, HttpStatus status) {
        ServerHttpResponse response = exchange.getResponse();
        response.setStatusCode(status);
        response.getHeaders().setContentType(MediaType.APPLICATION_JSON);

        ErrorResponse error = new ErrorResponse(status.value(), message);
        DataBuffer buffer = response.bufferFactory()
            .wrap(toJson(error).getBytes(StandardCharsets.UTF_8));

        return response.writeWith(Mono.just(buffer));
    }

    @Override
    public int getOrder() {
        return -100; // Execute before other filters
    }
}
```

#### 3. Rate Limiting

```java
@Configuration
public class RateLimitingConfig {

    @Bean
    public RouteLocator rateLimitedRoutes(RouteLocatorBuilder builder) {
        return builder.routes()
            .route("rate-limited-api", r -> r
                .path("/api/**")
                .filters(f -> f
                    .requestRateLimiter(config -> config
                        .setRateLimiter(redisRateLimiter())
                        .setKeyResolver(userKeyResolver())
                        .setDenyEmptyKey(false)))
                .uri("lb://API-SERVICE"))
            .build();
    }

    @Bean
    public RedisRateLimiter redisRateLimiter() {
        // replenishRate: عدد الطلبات المسموحة في الثانية
        // burstCapacity: الحد الأقصى للطلبات المتزامنة
        return new RedisRateLimiter(10, 20, 1);
    }

    // Rate limit per user
    @Bean
    public KeyResolver userKeyResolver() {
        return exchange -> Mono.just(
            exchange.getRequest()
                .getHeaders()
                .getFirst("X-User-ID")
        );
    }

    // Rate limit per IP
    @Bean
    public KeyResolver ipKeyResolver() {
        return exchange -> Mono.just(
            exchange.getRequest()
                .getRemoteAddress()
                .getAddress()
                .getHostAddress()
        );
    }

    // Rate limit per API key
    @Bean
    public KeyResolver apiKeyResolver() {
        return exchange -> Mono.just(
            exchange.getRequest()
                .getHeaders()
                .getFirst("X-API-Key")
        );
    }
}

// Custom Rate Limiter
@Component
public class CustomRateLimiter implements RateLimiter<RedisRateLimiter.Config> {

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    @Override
    public Mono<Response> isAllowed(String routeId, String id) {
        String key = "rate-limit:" + routeId + ":" + id;

        return Mono.fromSupplier(() -> {
            Long count = redisTemplate.opsForValue().increment(key);

            if (count == 1) {
                redisTemplate.expire(key, 1, TimeUnit.MINUTES);
            }

            int limit = 100; // 100 requests per minute
            boolean allowed = count <= limit;

            return new Response(allowed, getHeaders(count, limit));
        });
    }

    private Map<String, String> getHeaders(Long current, int limit) {
        Map<String, String> headers = new HashMap<>();
        headers.put("X-RateLimit-Limit", String.valueOf(limit));
        headers.put("X-RateLimit-Remaining", String.valueOf(Math.max(0, limit - current)));
        headers.put("X-RateLimit-Reset", String.valueOf(System.currentTimeMillis() / 1000 + 60));
        return headers;
    }
}
```

#### 4. Response Aggregation

```java
@Component
public class ResponseAggregationFilter implements GatewayFilter {

    @Autowired
    private WebClient.Builder webClientBuilder;

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String orderId = exchange.getRequest().getQueryParams().getFirst("orderId");

        if (orderId == null) {
            return chain.filter(exchange);
        }

        // جمع البيانات من خدمات متعددة بالتوازي
        Mono<Order> orderMono = fetchOrder(orderId);
        Mono<User> userMono = orderMono.flatMap(order -> fetchUser(order.getUserId()));
        Mono<List<Product>> productsMono = orderMono
            .flatMap(order -> fetchProducts(order.getProductIds()));
        Mono<Payment> paymentMono = fetchPayment(orderId);
        Mono<Shipping> shippingMono = fetchShipping(orderId);

        // دمج جميع النتائج
        return Mono.zip(orderMono, userMono, productsMono, paymentMono, shippingMono)
            .flatMap(tuple -> {
                OrderAggregateResponse response = OrderAggregateResponse.builder()
                    .order(tuple.getT1())
                    .user(tuple.getT2())
                    .products(tuple.getT3())
                    .payment(tuple.getT4())
                    .shipping(tuple.getT5())
                    .build();

                return writeResponse(exchange, response);
            })
            .onErrorResume(error -> {
                log.error("Error aggregating response", error);
                return chain.filter(exchange);
            });
    }

    private Mono<Order> fetchOrder(String orderId) {
        return webClientBuilder.build()
            .get()
            .uri("http://ORDER-SERVICE/orders/" + orderId)
            .retrieve()
            .bodyToMono(Order.class)
            .timeout(Duration.ofSeconds(2));
    }

    private Mono<User> fetchUser(Long userId) {
        return webClientBuilder.build()
            .get()
            .uri("http://USER-SERVICE/users/" + userId)
            .retrieve()
            .bodyToMono(User.class)
            .timeout(Duration.ofSeconds(2));
    }

    private Mono<List<Product>> fetchProducts(List<Long> productIds) {
        return Flux.fromIterable(productIds)
            .flatMap(id -> webClientBuilder.build()
                .get()
                .uri("http://PRODUCT-SERVICE/products/" + id)
                .retrieve()
                .bodyToMono(Product.class))
            .collectList()
            .timeout(Duration.ofSeconds(3));
    }

    private Mono<Void> writeResponse(ServerWebExchange exchange, Object response) {
        ServerHttpResponse httpResponse = exchange.getResponse();
        httpResponse.getHeaders().setContentType(MediaType.APPLICATION_JSON);

        DataBuffer buffer = httpResponse.bufferFactory()
            .wrap(toJson(response).getBytes(StandardCharsets.UTF_8));

        return httpResponse.writeWith(Mono.just(buffer));
    }
}
```

#### 5. Request/Response Transformation

```java
@Component
public class RequestTransformationFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();

        // Transform request
        ServerHttpRequest transformedRequest = request.mutate()
            // Add headers
            .header("X-Gateway-Timestamp", String.valueOf(System.currentTimeMillis()))
            .header("X-Request-ID", UUID.randomUUID().toString())
            .header("X-Forwarded-For", getClientIp(request))

            // Modify path
            .path(transformPath(request.getPath().toString()))

            .build();

        // Transform response
        ServerHttpResponse response = exchange.getResponse();
        DataBufferFactory bufferFactory = response.bufferFactory();

        ServerHttpResponseDecorator decoratedResponse = new ServerHttpResponseDecorator(response) {
            @Override
            public Mono<Void> writeWith(Publisher<? extends DataBuffer> body) {
                if (body instanceof Flux) {
                    Flux<? extends DataBuffer> fluxBody = (Flux<? extends DataBuffer>) body;

                    return super.writeWith(fluxBody.map(dataBuffer -> {
                        byte[] content = new byte[dataBuffer.readableByteCount()];
                        dataBuffer.read(content);
                        DataBufferUtils.release(dataBuffer);

                        // Transform response body
                        String transformedContent = transformResponse(new String(content));

                        // Add custom headers
                        getDelegate().getHeaders().add("X-Response-Time",
                            String.valueOf(System.currentTimeMillis()));

                        return bufferFactory.wrap(transformedContent.getBytes());
                    }));
                }

                return super.writeWith(body);
            }
        };

        return chain.filter(exchange.mutate()
            .request(transformedRequest)
            .response(decoratedResponse)
            .build());
    }

    private String transformPath(String originalPath) {
        // Remove /api prefix for backend services
        return originalPath.replaceFirst("/api", "");
    }

    private String transformResponse(String response) {
        // Add metadata to response
        try {
            JsonNode jsonNode = new ObjectMapper().readTree(response);
            ObjectNode objectNode = (ObjectNode) jsonNode;
            objectNode.put("_gateway", "spring-cloud-gateway");
            objectNode.put("_timestamp", System.currentTimeMillis());

            return new ObjectMapper().writeValueAsString(objectNode);
        } catch (Exception e) {
            return response;
        }
    }

    private String getClientIp(ServerHttpRequest request) {
        String xForwardedFor = request.getHeaders().getFirst("X-Forwarded-For");
        if (xForwardedFor != null) {
            return xForwardedFor.split(",")[0].trim();
        }
        return request.getRemoteAddress().getAddress().getHostAddress();
    }

    @Override
    public int getOrder() {
        return -50;
    }
}
```

---

## Load Balancing

### س15: اشرح Load Balancing في Microservices
**الجواب:**

**أنواع Load Balancing:**

#### 1. Client-Side Load Balancing (Spring Cloud LoadBalancer)

```java
// Configuration
@Configuration
public class LoadBalancerConfig {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    @Bean
    @LoadBalanced
    public WebClient.Builder webClientBuilder() {
        return WebClient.builder();
    }
}

// Custom Load Balancer
@Configuration
public class CustomLoadBalancerConfiguration {

    @Bean
    public ServiceInstanceListSupplier serviceInstanceListSupplier(
            ConfigurableApplicationContext context) {
        return ServiceInstanceListSupplier.builder()
            .withDiscoveryClient()
            .withHealthChecks()
            .withCaching()
            .build(context);
    }

    // Random Load Balancer
    @Bean
    public ReactorLoadBalancer<ServiceInstance> randomLoadBalancer(
            Environment environment,
            LoadBalancerClientFactory loadBalancerClientFactory) {

        String name = environment.getProperty(LoadBalancerClientFactory.PROPERTY_NAME);

        return new RandomLoadBalancer(
            loadBalancerClientFactory.getLazyProvider(name, ServiceInstanceListSupplier.class),
            name
        );
    }

    // Weighted Load Balancer
    @Bean
    public ReactorLoadBalancer<ServiceInstance> weightedLoadBalancer(
            Environment environment,
            LoadBalancerClientFactory loadBalancerClientFactory) {

        String name = environment.getProperty(LoadBalancerClientFactory.PROPERTY_NAME);

        return new WeightedLoadBalancer(
            loadBalancerClientFactory.getLazyProvider(name, ServiceInstanceListSupplier.class),
            name
        );
    }
}

// Usage
@Service
public class ProductService {

    @Autowired
    private RestTemplate restTemplate;

    @Autowired
    private WebClient.Builder webClientBuilder;

    // Using RestTemplate with LoadBalancer
    public Product getProduct(Long id) {
        // "PRODUCT-SERVICE" سيتم resolve تلقائياً
        return restTemplate.getForObject(
            "http://PRODUCT-SERVICE/products/" + id,
            Product.class
        );
    }

    // Using WebClient with LoadBalancer
    public Mono<Product> getProductAsync(Long id) {
        return webClientBuilder.build()
            .get()
            .uri("http://PRODUCT-SERVICE/products/" + id)
            .retrieve()
            .bodyToMono(Product.class);
    }
}

// Custom Load Balancing Strategy
public class CustomLoadBalancer implements ReactorServiceInstanceLoadBalancer {

    private final ObjectProvider<ServiceInstanceListSupplier> serviceInstanceListSupplierProvider;
    private final String serviceId;

    public CustomLoadBalancer(
            ObjectProvider<ServiceInstanceListSupplier> serviceInstanceListSupplierProvider,
            String serviceId) {
        this.serviceInstanceListSupplierProvider = serviceInstanceListSupplierProvider;
        this.serviceId = serviceId;
    }

    @Override
    public Mono<Response<ServiceInstance>> choose(Request request) {
        ServiceInstanceListSupplier supplier =
            serviceInstanceListSupplierProvider.getIfAvailable();

        return supplier.get(request).next()
            .map(serviceInstances -> {
                if (serviceInstances.isEmpty()) {
                    return new EmptyResponse();
                }

                // Custom selection logic
                ServiceInstance instance = selectInstance(serviceInstances, request);
                return new DefaultResponse(instance);
            });
    }

    private ServiceInstance selectInstance(
            List<ServiceInstance> instances,
            Request request) {

        // 1. Least Connection Strategy
        // 2. Response Time-based
        // 3. Zone-aware
        // 4. Custom metadata-based

        // Example: Zone-aware selection
        String preferredZone = getPreferredZone(request);

        List<ServiceInstance> zoneInstances = instances.stream()
            .filter(i -> preferredZone.equals(i.getMetadata().get("zone")))
            .collect(Collectors.toList());

        if (!zoneInstances.isEmpty()) {
            return zoneInstances.get(new Random().nextInt(zoneInstances.size()));
        }

        // Fallback to any instance
        return instances.get(new Random().nextInt(instances.size()));
    }

    private String getPreferredZone(Request request) {
        // Logic to determine preferred zone
        return "zone1";
    }
}
```

#### 2. Server-Side Load Balancing (Nginx, HAProxy)

**Nginx Configuration:**

```nginx
# nginx.conf
http {
    upstream user_service {
        # Round Robin (default)
        server user-service-1:8080;
        server user-service-2:8080;
        server user-service-3:8080;
    }

    upstream product_service {
        # Least Connections
        least_conn;
        server product-service-1:8080;
        server product-service-2:8080 weight=2;  # 2x traffic
        server product-service-3:8080;
    }

    upstream order_service {
        # IP Hash (session persistence)
        ip_hash;
        server order-service-1:8080;
        server order-service-2:8080;
    }

    upstream payment_service {
        # Weighted Round Robin
        server payment-service-1:8080 weight=3;
        server payment-service-2:8080 weight=2;
        server payment-service-3:8080 weight=1;
    }

    # Health Checks
    upstream backend {
        server backend-1:8080 max_fails=3 fail_timeout=30s;
        server backend-2:8080 max_fails=3 fail_timeout=30s;
        server backend-3:8080 backup;  # backup server
    }

    server {
        listen 80;

        location /api/users/ {
            proxy_pass http://user_service;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

            # Timeouts
            proxy_connect_timeout 5s;
            proxy_send_timeout 10s;
            proxy_read_timeout 10s;

            # Retry logic
            proxy_next_upstream error timeout http_500 http_502 http_503;
            proxy_next_upstream_tries 3;
        }

        location /api/products/ {
            proxy_pass http://product_service;
            proxy_set_header Host $host;
        }

        location /api/orders/ {
            proxy_pass http://order_service;
            proxy_set_header Host $host;

            # Session stickiness with cookies
            sticky cookie srv_id expires=1h;
        }

        # Circuit Breaker pattern
        location /api/payments/ {
            proxy_pass http://payment_service;

            error_page 502 503 504 = @payment_fallback;
        }

        location @payment_fallback {
            return 503 '{"error": "Payment service temporarily unavailable"}';
            add_header Content-Type application/json;
        }
    }

    # Rate Limiting
    limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;

    server {
        listen 80;

        location /api/ {
            limit_req zone=api_limit burst=20 nodelay;
            proxy_pass http://backend;
        }
    }
}
```

**HAProxy Configuration:**

```haproxy
# haproxy.cfg
global
    maxconn 4096
    log 127.0.0.1 local0

defaults
    mode http
    log global
    option httplog
    option dontlognull
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

# Frontend
frontend http_front
    bind *:80

    # ACLs for routing
    acl is_user_service path_beg /api/users
    acl is_product_service path_beg /api/products
    acl is_order_service path_beg /api/orders

    # Rate limiting
    stick-table type ip size 100k expire 30s store http_req_rate(10s)
    http-request track-sc0 src
    http-request deny if { sc_http_req_rate(0) gt 100 }

    # Routing
    use_backend user_backend if is_user_service
    use_backend product_backend if is_product_service
    use_backend order_backend if is_order_service
    default_backend default_backend

# User Service Backend - Round Robin
backend user_backend
    balance roundrobin
    option httpchk GET /actuator/health
    http-check expect status 200

    server user1 user-service-1:8080 check
    server user2 user-service-2:8080 check
    server user3 user-service-3:8080 check

# Product Service Backend - Least Connections
backend product_backend
    balance leastconn
    option httpchk GET /actuator/health

    server product1 product-service-1:8080 check weight 1
    server product2 product-service-2:8080 check weight 2
    server product3 product-service-3:8080 check weight 1

# Order Service Backend - Source IP Hash (Session Persistence)
backend order_backend
    balance source
    option httpchk GET /actuator/health

    server order1 order-service-1:8080 check
    server order2 order-service-2:8080 check

# Statistics
listen stats
    bind *:8404
    stats enable
    stats uri /stats
    stats refresh 30s
    stats admin if TRUE
```

**استراتيجيات Load Balancing:**

| الاستراتيجية | الوصف | متى تُستخدم |
|--------------|--------|-------------|
| **Round Robin** | توزيع متساوٍ بالتناوب | خدمات متطابقة |
| **Weighted RR** | توزيع بأوزان مختلفة | خوادم بقدرات مختلفة |
| **Least Connections** | إلى الخادم بأقل اتصالات | طلبات طويلة المدة |
| **IP Hash** | نفس العميل → نفس الخادم | Session persistence |
| **Random** | اختيار عشوائي | توزيع بسيط |
| **Response Time** | أسرع خادم | تحسين الأداء |

---

## المراقبة والتتبع

### س16: كيف تتم المراقبة في Microservices؟
**الجواب:**

**المكونات الأساسية:**

```
┌─────────────────────────────────────────────┐
│         Application Layer                   │
│  (Microservices with instrumentation)      │
└─────────────┬───────────────────────────────┘
              │
    ┌─────────┼──────────┬──────────────┐
    │         │          │              │
┌───▼────┐ ┌──▼─────┐ ┌─▼──────┐ ┌─────▼─────┐
│ Logs   │ │Metrics │ │ Traces │ │ Health    │
│(ELK)   │ │(Prom)  │ │(Zipkin)│ │ Checks    │
└───┬────┘ └──┬─────┘ └─┬──────┘ └─────┬─────┘
    │         │          │              │
┌───▼─────────▼──────────▼──────────────▼─────┐
│         Visualization Layer                  │
│  (Grafana, Kibana, Zipkin UI)               │
└──────────────────────────────────────────────┘
```

#### 1. Distributed Tracing (Zipkin + Sleuth)

```xml
<!-- Dependencies -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-sleuth-zipkin</artifactId>
</dependency>
```

```yaml
# application.yml
spring:
  application:
    name: order-service

  sleuth:
    sampler:
      probability: 1.0  # 100% of requests (للتطوير، استخدم 0.1 في الإنتاج)

  zipkin:
    base-url: http://localhost:9411
    sender:
      type: web  # أو kafka أو rabbitmq

  # Async processing
  sleuth:
    async:
      enabled: true
    scheduled:
      enabled: true

logging:
  pattern:
    level: "%5p [${spring.application.name:},%X{traceId:-},%X{spanId:-}]"
```

```java
// تتبع تلقائي للطلبات
@RestController
@RequestMapping("/orders")
@Slf4j
public class OrderController {

    @Autowired
    private OrderService orderService;

    @Autowired
    private Tracer tracer;  // Brave Tracer

    @GetMapping("/{id}")
    public Order getOrder(@PathVariable Long id) {
        // Sleuth يضيف traceId و spanId تلقائياً
        log.info("Fetching order: {}", id);

        // إضافة tags مخصصة
        Span span = tracer.currentSpan();
        if (span != null) {
            span.tag("order.id", id.toString());
            span.tag("user.type", "premium");
        }

        return orderService.getOrder(id);
    }

    @PostMapping
    public Order createOrder(@RequestBody OrderRequest request) {
        // إنشاء span جديد
        Span newSpan = tracer.nextSpan().name("create-order").start();

        try (Tracer.SpanInScope ws = tracer.withSpanInScope(newSpan)) {
            newSpan.tag("order.items", String.valueOf(request.getItems().size()));
            newSpan.tag("order.amount", request.getTotalAmount().toString());

            log.info("Creating order");
            return orderService.createOrder(request);

        } catch (Exception e) {
            newSpan.tag("error", e.getMessage());
            newSpan.error(e);
            throw e;
        } finally {
            newSpan.finish();
        }
    }
}

// Service Layer
@Service
@Slf4j
public class OrderService {

    @Autowired
    private RestTemplate restTemplate;

    @Autowired
    private UserService userService;

    @NewSpan(name = "getOrder")  // إنشاء span جديد
    public Order getOrder(Long id) {
        log.info("Getting order from database");

        Order order = orderRepository.findById(id)
            .orElseThrow(() -> new OrderNotFoundException(id));

        // سيتم تتبع هذا الاستدعاء تلقائياً
        User user = userService.getUser(order.getUserId());
        order.setUser(user);

        return order;
    }

    @NewSpan
    public Order createOrder(OrderRequest request) {
        // كل استدعاء HTTP سيُنشئ span جديد
        User user = restTemplate.getForObject(
            "http://USER-SERVICE/users/" + request.getUserId(),
            User.class
        );

        Product product = restTemplate.getForObject(
            "http://PRODUCT-SERVICE/products/" + request.getProductId(),
            Product.class
        );

        Payment payment = restTemplate.postForObject(
            "http://PAYMENT-SERVICE/payments",
            request.getPaymentInfo(),
            Payment.class
        );

        Order order = new Order(user, product, payment);
        return orderRepository.save(order);
    }
}

// Async methods tracing
@Service
public class NotificationService {

    @Async
    @NewSpan
    public CompletableFuture<Void> sendEmailAsync(String email, String message) {
        log.info("Sending email to: {}", email);
        // Send email logic
        return CompletableFuture.completedFuture(null);
    }
}

// Custom Span
@Component
public class PaymentProcessor {

    @Autowired
    private Tracer tracer;

    public Payment processPayment(PaymentRequest request) {
        Span span = tracer.nextSpan().name("payment-processing");

        try (Tracer.SpanInScope ws = tracer.withSpanInScope(span.start())) {
            span.tag("payment.method", request.getMethod());
            span.tag("payment.amount", request.getAmount().toString());

            // Simulate payment processing
            Thread.sleep(500);

            span.annotate("payment.authorized");

            Payment payment = new Payment(request);

            span.annotate("payment.completed");

            return payment;

        } catch (Exception e) {
            span.tag("error", "true");
            span.tag("error.message", e.getMessage());
            throw new PaymentProcessingException(e);
        } finally {
            span.finish();
        }
    }
}
```

**Zipkin UI:**
```
Trace Visualization:
═══════════════════════════════════════════════════════
order-service: POST /orders              [200ms]
├── user-service: GET /users/123         [50ms]
├── product-service: GET /products/456   [30ms]
├── inventory-service: POST /reserve     [40ms]
└── payment-service: POST /payments      [80ms]
    └── payment-gateway: POST /charge    [60ms]
═══════════════════════════════════════════════════════
```

#### 2. Metrics (Prometheus + Micrometer)

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: always
    metrics:
      enabled: true
    prometheus:
      enabled: true
  metrics:
    export:
      prometheus:
        enabled: true
    distribution:
      percentiles-histogram:
        http.server.requests: true
    tags:
      application: ${spring.application.name}
      environment: ${ENVIRONMENT:dev}
```

```java
// Custom Metrics
@Service
public class OrderMetricsService {

    private final Counter orderCounter;
    private final Timer orderProcessingTimer;
    private final Gauge activeOrders;
    private final DistributionSummary orderAmountSummary;

    public OrderMetricsService(MeterRegistry meterRegistry) {
        // Counter - عدد الطلبات
        this.orderCounter = Counter.builder("orders.created")
            .description("Total number of orders created")
            .tag("service", "order-service")
            .register(meterRegistry);

        // Timer - وقت معالجة الطلبات
        this.orderProcessingTimer = Timer.builder("orders.processing.time")
            .description("Time taken to process orders")
            .tag("service", "order-service")
            .register(meterRegistry);

        // Gauge - الطلبات النشطة حالياً
        this.activeOrders = Gauge.builder("orders.active", this, OrderMetricsService::getActiveOrderCount)
            .description("Number of active orders")
            .register(meterRegistry);

        // Distribution Summary - توزيع قيم الطلبات
        this.orderAmountSummary = DistributionSummary.builder("orders.amount")
            .description("Distribution of order amounts")
            .baseUnit("USD")
            .register(meterRegistry);
    }

    public Order createOrder(OrderRequest request) {
        return orderProcessingTimer.record(() -> {
            Order order = processOrder(request);

            // زيادة العداد
            orderCounter.increment();

            // تسجيل قيمة الطلب
            orderAmountSummary.record(order.getTotalAmount().doubleValue());

            return order;
        });
    }

    private double getActiveOrderCount() {
        return orderRepository.countByStatus(OrderStatus.PROCESSING);
    }
}

// تسجيل Metrics مخصص باستخدام Annotations
@Timed(value = "api.orders.get", description = "Time to get an order")
@Counted(value = "api.orders.get.count", description = "Number of get order calls")
@GetMapping("/{id}")
public Order getOrder(@PathVariable Long id) {
    return orderService.getOrder(id);
}

// Metrics للـ Cache
@Configuration
public class CacheMetricsConfig {

    @Bean
    public CacheMetricsRegistrar cacheMetricsRegistrar(
            MeterRegistry meterRegistry,
            CacheManager cacheManager) {

        cacheManager.getCacheNames().forEach(cacheName -> {
            Cache cache = cacheManager.getCache(cacheName);

            if (cache instanceof CaffeineCache) {
                CaffeineCache caffeineCache = (CaffeineCache) cache;
                CacheMetricsCollector.monitor(
                    meterRegistry,
                    caffeineCache.getNativeCache(),
                    cacheName
                );
            }
        });

        return new CacheMetricsRegistrar();
    }
}

// Database Metrics
@Component
public class DatabaseMetrics {

    public DatabaseMetrics(MeterRegistry meterRegistry, DataSource dataSource) {
        // Connection Pool Metrics
        new DataSourcePoolMetrics(dataSource, "hikaricp", Collections.emptyList())
            .bindTo(meterRegistry);
    }
}

// JVM Metrics
@Configuration
public class JvmMetricsConfig {

    @Bean
    public MeterBinder jvmMetrics() {
        return (registry) -> {
            new JvmMemoryMetrics().bindTo(registry);
            new JvmGcMetrics().bindTo(registry);
            new JvmThreadMetrics().bindTo(registry);
            new ClassLoaderMetrics().bindTo(registry);
        };
    }
}
```

**Prometheus Configuration:**

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'spring-boot-apps'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets:
        - 'user-service:8080'
        - 'product-service:8080'
        - 'order-service:8080'
        labels:
          environment: 'production'

  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)

# Alerting rules
rule_files:
  - 'alerts.yml'

alerting:
  alertmanagers:
    - static_configs:
      - targets:
        - 'alertmanager:9093'
```

**Alert Rules:**

```yaml
# alerts.yml
groups:
  - name: microservices_alerts
    interval: 30s
    rules:
      # High Error Rate
      - alert: HighErrorRate
        expr: rate(http_server_requests_seconds_count{status=~"5.."}[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate on {{ $labels.application }}"
          description: "Error rate is {{ $value }} requests/second"

      # High Response Time
      - alert: HighResponseTime
        expr: histogram_quantile(0.95, http_server_requests_seconds_bucket) > 1
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High response time on {{ $labels.application }}"
          description: "95th percentile response time is {{ $value }}s"

      # Service Down
      - alert: ServiceDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Service {{ $labels.job }} is down"
          description: "{{ $labels.instance }} has been down for more than 1 minute"

      # High Memory Usage
      - alert: HighMemoryUsage
        expr: (jvm_memory_used_bytes{area="heap"} / jvm_memory_max_bytes{area="heap"}) > 0.9
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage on {{ $labels.application }}"
          description: "Memory usage is {{ $value | humanizePercentage }}"

      # Circuit Breaker Open
      - alert: CircuitBreakerOpen
        expr: resilience4j_circuitbreaker_state{state="open"} == 1
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "Circuit breaker open for {{ $labels.name }}"
          description: "Circuit breaker has been open for 2 minutes"
```

#### 3. Logging (ELK Stack)

**Logback Configuration:**

```xml
<!-- logback-spring.xml -->
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>

    <springProperty scope="context" name="appName" source="spring.application.name"/>

    <!-- Console Appender -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <customFields>{"app":"${appName}"}</customFields>
        </encoder>
    </appender>

    <!-- Logstash Appender -->
    <appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>logstash:5000</destination>
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <customFields>{"app":"${appName}","environment":"production"}</customFields>
        </encoder>
    </appender>

    <!-- File Appender -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/${appName}.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/${appName}-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy
                class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} [%X{traceId},%X{spanId}] - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="LOGSTASH"/>
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

```java
// Structured Logging
@Slf4j
@Service
public class OrderService {

    public Order createOrder(OrderRequest request) {
        // Basic logging
        log.info("Creating order for user: {}", request.getUserId());

        // Structured logging with MDC
        MDC.put("userId", request.getUserId().toString());
        MDC.put("orderAmount", request.getTotalAmount().toString());

        try {
            Order order = processOrder(request);

            log.info("Order created successfully: orderId={}, amount={}",
                order.getId(),
                order.getTotalAmount());

            return order;

        } catch (Exception e) {
            log.error("Failed to create order", e);
            throw e;
        } finally {
            MDC.clear();
        }
    }

    // Using Markers for filtering
    private static final Marker SECURITY_MARKER = MarkerFactory.getMarker("SECURITY");
    private static final Marker BUSINESS_MARKER = MarkerFactory.getMarker("BUSINESS");

    public void logSecurityEvent(String event) {
        log.warn(SECURITY_MARKER, "Security event: {}", event);
    }

    public void logBusinessEvent(Order order) {
        log.info(BUSINESS_MARKER, "Business event: order created - {}", order.getId());
    }
}

// Custom Log Filter
@Component
public class LoggingFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {

        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;

        String requestId = UUID.randomUUID().toString();
        MDC.put("requestId", requestId);
        MDC.put("method", httpRequest.getMethod());
        MDC.put("path", httpRequest.getRequestURI());

        long startTime = System.currentTimeMillis();

        try {
            chain.doFilter(request, response);
        } finally {
            long duration = System.currentTimeMillis() - startTime;

            log.info("Request completed: status={}, duration={}ms",
                httpResponse.getStatus(),
                duration);

            MDC.clear();
        }
    }
}
```

**Elasticsearch Query Examples:**

```json
// البحث عن أخطاء في آخر ساعة
GET /logs-*/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "level": "ERROR"
          }
        },
        {
          "range": {
            "@timestamp": {
              "gte": "now-1h"
            }
          }
        }
      ]
    }
  },
  "size": 100,
  "sort": [
    {
      "@timestamp": {
        "order": "desc"
      }
    }
  ]
}

// تجميع الأخطاء حسب الخدمة
GET /logs-*/_search
{
  "query": {
    "match": {
      "level": "ERROR"
    }
  },
  "aggs": {
    "errors_by_service": {
      "terms": {
        "field": "app.keyword",
        "size": 10
      }
    }
  }
}

// تتبع طلب عبر الخدمات باستخدام traceId
GET /logs-*/_search
{
  "query": {
    "match": {
      "traceId": "abc123xyz"
    }
  },
  "sort": [
    {
      "@timestamp": {
        "order": "asc"
      }
    }
  ]
}
```

**Grafana Dashboard:**

```json
{
  "dashboard": {
    "title": "Microservices Overview",
    "panels": [
      {
        "title": "Request Rate",
        "targets": [
          {
            "expr": "rate(http_server_requests_seconds_count[5m])"
          }
        ]
      },
      {
        "title": "Error Rate",
        "targets": [
          {
            "expr": "rate(http_server_requests_seconds_count{status=~\"5..\"}[5m])"
          }
        ]
      },
      {
        "title": "Response Time (95th percentile)",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, http_server_requests_seconds_bucket)"
          }
        ]
      },
      {
        "title": "JVM Memory Usage",
        "targets": [
          {
            "expr": "jvm_memory_used_bytes{area=\"heap\"} / jvm_memory_max_bytes{area=\"heap\"}"
          }
        ]
      }
    ]
  }
}
```

---

## قواعد البيانات

### س17: اشرح نمط Database per Service
**الجواب:**

**المبدأ:**
كل خدمة microservice لها قاعدة بيانات خاصة بها ولا يمكن للخدمات الأخرى الوصول إليها مباشرة.

```
❌ مشاركة قاعدة البيانات (خطأ):
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Service  │  │ Service  │  │ Service  │
│    A     │  │    B     │  │    C     │
└────┬─────┘  └────┬─────┘  └────┬─────┘
     │            │            │
     └────────────┼────────────┘
                  │
            ┌─────▼─────┐
            │  Shared   │
            │ Database  │
            └───────────┘

✓ قاعدة بيانات لكل خدمة (صحيح):
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Service  │  │ Service  │  │ Service  │
│    A     │  │    B     │  │    C     │
└────┬─────┘  └────┬─────┘  └────┬─────┘
     │            │            │
┌────▼─────┐ ┌───▼──────┐ ┌───▼──────┐
│Database A│ │Database B│ │Database C│
└──────────┘ └──────────┘ └──────────┘
```

**أمثلة عملية:**

```yaml
# User Service - MySQL
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/user_db
    username: user_service
    password: ${DB_PASSWORD}
  jpa:
    database-platform: org.hibernate.dialect.MySQL8Dialect
    hibernate:
      ddl-auto: validate
  flyway:
    enabled: true
    locations: classpath:db/migration/user

# Product Service - PostgreSQL
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/product_db
    username: product_service
    password: ${DB_PASSWORD}
  jpa:
    database-platform: org.hibernate.dialect.PostgreSQLDialect
  flyway:
    enabled: true
    locations: classpath:db/migration/product

# Order Service - MongoDB
spring:
  data:
    mongodb:
      uri: mongodb://localhost:27017/order_db
      username: order_service
      password: ${DB_PASSWORD}

# Inventory Service - Redis
spring:
  redis:
    host: localhost
    port: 6379
    database: 0
    password: ${REDIS_PASSWORD}

# Analytics Service - Cassandra
spring:
  data:
    cassandra:
      contact-points: localhost
      port: 9042
      keyspace-name: analytics_db
      local-datacenter: datacenter1
```

### س18: كيف تحافظ على اتساق البيانات عبر الخدمات؟
**الجواب:**

#### 1. Saga Pattern (شرحناه سابقاً)

#### 2. Event-Driven Data Consistency

```java
// Order Service - نشر الأحداث
@Service
@Transactional
public class OrderService {

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private ApplicationEventPublisher eventPublisher;

    public Order createOrder(OrderRequest request) {
        // 1. حفظ في قاعدة البيانات المحلية
        Order order = Order.builder()
            .userId(request.getUserId())
            .items(request.getItems())
            .status(OrderStatus.PENDING)
            .build();

        order = orderRepository.save(order);

        // 2. نشر الحدث
        OrderCreatedEvent event = new OrderCreatedEvent(
            order.getId(),
            order.getUserId(),
            order.getItems(),
            order.getTotalAmount()
        );

        eventPublisher.publishEvent(event);

        return order;
    }
}

// Event Publisher to Message Queue
@Component
public class EventPublisher {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void handleOrderCreated(OrderCreatedEvent event) {
        // نشر الحدث بعد commit ناجح فقط
        rabbitTemplate.convertAndSend(
            "order.exchange",
            "order.created",
            event
        );
    }
}

// Inventory Service - الاستماع وتحديث البيانات المحلية
@Service
public class InventoryEventListener {

    @Autowired
    private InventoryRepository inventoryRepository;

    @RabbitListener(queues = "inventory.order.queue")
    @Transactional
    public void handleOrderCreated(OrderCreatedEvent event) {
        // تحديث المخزون المحلي
        event.getItems().forEach(item -> {
            Inventory inventory = inventoryRepository
                .findByProductId(item.getProductId())
                .orElseThrow();

            inventory.setReservedQuantity(
                inventory.getReservedQuantity() + item.getQuantity()
            );

            inventoryRepository.save(inventory);
        });
    }
}
```

#### 3. CQRS with Event Sourcing

```java
// Write Side (Command) - Order Service
@Service
public class OrderCommandService {

    @Autowired
    private EventStore eventStore;

    public String createOrder(CreateOrderCommand command) {
        String orderId = UUID.randomUUID().toString();

        // إنشاء الأحداث
        List<DomainEvent> events = Arrays.asList(
            new OrderCreatedEvent(orderId, command.getUserId(), command.getItems()),
            new OrderTotalCalculatedEvent(orderId, calculateTotal(command.getItems()))
        );

        // حفظ الأحداث
        eventStore.saveEvents(orderId, events);

        return orderId;
    }
}

// Read Side (Query) - Order Query Service
@Service
public class OrderQueryService {

    @Autowired
    private OrderReadRepository readRepository;

    // يتم تحديث Read Model من خلال Event Listeners
    @EventListener
    @Transactional
    public void on(OrderCreatedEvent event) {
        OrderReadModel readModel = new OrderReadModel();
        readModel.setOrderId(event.getOrderId());
        readModel.setUserId(event.getUserId());
        readModel.setStatus(OrderStatus.PENDING);

        readRepository.save(readModel);
    }

    public OrderDTO getOrder(String orderId) {
        return readRepository.findById(orderId)
            .map(this::toDTO)
            .orElseThrow();
    }
}

// User Service - نسخة محلية من بيانات الطلبات
@Service
public class UserOrderProjection {

    @Autowired
    private UserOrderReadRepository repository;

    @EventListener
    @Transactional
    public void on(OrderCreatedEvent event) {
        // الاحتفاظ بنسخة محلية من طلبات المستخدم
        UserOrderReadModel model = UserOrderReadModel.builder()
            .userId(event.getUserId())
            .orderId(event.getOrderId())
            .totalAmount(event.getTotalAmount())
            .status(OrderStatus.PENDING)
            .createdAt(LocalDateTime.now())
            .build();

        repository.save(model);
    }

    public List<UserOrderDTO> getUserOrders(Long userId) {
        return repository.findByUserId(userId)
            .stream()
            .map(this::toDTO)
            .collect(Collectors.toList());
    }
}
```

#### 4. API Composition

```java
// API Gateway or Composite Service
@Service
public class OrderCompositeService {

    @Autowired
    private WebClient.Builder webClient;

    public OrderDetailsDTO getOrderDetails(String orderId) {
        // استدعاء خدمات متعددة
        Mono<Order> orderMono = webClient.build()
            .get()
            .uri("http://ORDER-SERVICE/orders/" + orderId)
            .retrieve()
            .bodyToMono(Order.class);

        // الحصول على معلومات المستخدم
        Mono<User> userMono = orderMono.flatMap(order ->
            webClient.build()
                .get()
                .uri("http://USER-SERVICE/users/" + order.getUserId())
                .retrieve()
                .bodyToMono(User.class)
        );

        // الحصول على معلومات المنتجات
        Mono<List<Product>> productsMono = orderMono.flatMap(order ->
            Flux.fromIterable(order.getProductIds())
                .flatMap(productId ->
                    webClient.build()
                        .get()
                        .uri("http://PRODUCT-SERVICE/products/" + productId)
                        .retrieve()
                        .bodyToMono(Product.class))
                .collectList()
        );

        // دمج البيانات
        return Mono.zip(orderMono, userMono, productsMono)
            .map(tuple -> OrderDetailsDTO.builder()
                .order(tuple.getT1())
                .user(tuple.getT2())
                .products(tuple.getT3())
                .build())
            .block();
    }
}
```

**جدول المقارنة:**

| النمط | الاتساق | التعقيد | الأداء | متى يُستخدم |
|-------|---------|---------|--------|--------------|
| **Saga** | Eventual | متوسط | جيد | Transactions موزعة |
| **Event Sourcing** | Eventual | عالي | ممتاز | Audit trail مطلوب |
| **CQRS** | Eventual | عالي | ممتاز | Read/Write مختلفة |
| **API Composition** | Strong | منخفض | متوسط | Queries فقط |

---

## المرونة والاستقرار

### س19: كيف تضمن مرونة النظام؟
**الجواب:**

**الأنماط الأساسية:**

#### 1. Circuit Breaker (شرحناه سابقاً)

#### 2. Retry Pattern

```java
@Configuration
public class RetryConfig {

    @Bean
    public Retry orderServiceRetry() {
        RetryConfig config = RetryConfig.custom()
            .maxAttempts(3)
            .waitDuration(Duration.ofMillis(500))
            .retryExceptions(IOException.class, TimeoutException.class)
            .ignoreExceptions(BusinessException.class)
            .build();

        return Retry.of("order-service", config);
    }

    // Exponential Backoff
    @Bean
    public Retry exponentialRetry() {
        IntervalFunction intervalFn = IntervalFunction
            .ofExponentialBackoff(100, 2); // 100ms, 200ms, 400ms, 800ms...

        RetryConfig config = RetryConfig.custom()
            .maxAttempts(5)
            .intervalFunction(intervalFn)
            .build();

        return Retry.of("exponential-retry", config);
    }
}

@Service
public class PaymentService {

    @Autowired
    private Retry retry;

    @Retry(name = "payment-service", fallbackMethod = "paymentFallback")
    public Payment processPayment(PaymentRequest request) {
        return retry.executeSupplier(() -> {
            log.info("Attempting payment processing");
            return paymentGateway.charge(request);
        });
    }

    private Payment paymentFallback(PaymentRequest request, Exception ex) {
        log.error("Payment failed after retries", ex);
        return Payment.failed(request, ex.getMessage());
    }
}
```

#### 3. Timeout Pattern

```java
@Configuration
public class TimeoutConfig {

    @Bean
    public TimeLimiter timeLimiter() {
        TimeLimiterConfig config = TimeLimiterConfig.custom()
            .timeoutDuration(Duration.ofSeconds(3))
            .cancelRunningFuture(true)
            .build();

        return TimeLimiter.of("default", config);
    }
}

@Service
public class ProductService {

    @Autowired
    private TimeLimiter timeLimiter;

    @TimeLimiter(name = "product-service", fallbackMethod = "getProductFallback")
    public CompletableFuture<Product> getProductAsync(Long id) {
        return CompletableFuture.supplyAsync(() -> {
            // Long-running operation
            return productRepository.findById(id).orElseThrow();
        });
    }

    private CompletableFuture<Product> getProductFallback(Long id, Exception ex) {
        return CompletableFuture.completedFuture(
            Product.unavailable(id)
        );
    }
}
```

#### 4. Bulkhead Pattern

```java
// عزل الموارد لمنع انتشار الفشل
@Configuration
public class BulkheadConfig {

    @Bean
    public ThreadPoolBulkhead orderBulkhead() {
        ThreadPoolBulkheadConfig config = ThreadPoolBulkheadConfig.custom()
            .maxThreadPoolSize(10)
            .coreThreadPoolSize(5)
            .queueCapacity(20)
            .keepAliveDuration(Duration.ofMillis(1000))
            .build();

        return ThreadPoolBulkhead.of("order-bulkhead", config);
    }

    @Bean
    public Bulkhead paymentBulkhead() {
        BulkheadConfig config = BulkheadConfig.custom()
            .maxConcurrentCalls(25)
            .maxWaitDuration(Duration.ofMillis(500))
            .build();

        return Bulkhead.of("payment-bulkhead", config);
    }
}

@Service
public class OrderService {

    @Bulkhead(name = "order-bulkhead", type = Bulkhead.Type.THREADPOOL)
    public CompletableFuture<Order> createOrderAsync(OrderRequest request) {
        return CompletableFuture.supplyAsync(() -> createOrder(request));
    }

    @Bulkhead(name = "payment-bulkhead", fallbackMethod = "paymentFallback")
    public Payment processPayment(PaymentRequest request) {
        return paymentService.process(request);
    }
}
```

#### 5. Health Checks

```java
@Component
public class CustomHealthIndicators {

    // Database Health
    @Bean
    public HealthIndicator databaseHealth(DataSource dataSource) {
        return () -> {
            try (Connection conn = dataSource.getConnection()) {
                return Health.up()
                    .withDetail("database", "Available")
                    .build();
            } catch (Exception e) {
                return Health.down()
                    .withDetail("database", "Unavailable")
                    .withException(e)
                    .build();
            }
        };
    }

    // External Service Health
    @Bean
    public HealthIndicator paymentServiceHealth(WebClient.Builder webClient) {
        return () -> {
            try {
                String response = webClient.build()
                    .get()
                    .uri("http://PAYMENT-SERVICE/actuator/health")
                    .retrieve()
                    .bodyToMono(String.class)
                    .timeout(Duration.ofSeconds(2))
                    .block();

                return Health.up()
                    .withDetail("payment-service", "Available")
                    .build();

            } catch (Exception e) {
                return Health.down()
                    .withDetail("payment-service", "Unavailable")
                    .build();
            }
        };
    }

    // Disk Space Health
    @Bean
    public HealthIndicator diskSpaceHealth() {
        return () -> {
            File root = new File("/");
            long freeSpace = root.getFreeSpace();
            long totalSpace = root.getTotalSpace();
            double percentFree = (double) freeSpace / totalSpace * 100;

            if (percentFree < 10) {
                return Health.down()
                    .withDetail("disk-space", "Low")
                    .withDetail("free", freeSpace)
                    .withDetail("percent-free", percentFree)
                    .build();
            }

            return Health.up()
                .withDetail("disk-space", "Sufficient")
                .withDetail("free", freeSpace)
                .withDetail("percent-free", percentFree)
                .build();
        };
    }
}

// Liveness & Readiness Probes
@Component
public class ApplicationAvailability {

    @Bean
    public AvailabilityChangeEvent.Builder livenessState() {
        return AvailabilityChangeEvent.publish(
            applicationContext,
            LivenessState.CORRECT
        );
    }

    @Bean
    public AvailabilityChangeEvent.Builder readinessState() {
        return AvailabilityChangeEvent.publish(
            applicationContext,
            ReadinessState.ACCEPTING_TRAFFIC
        );
    }
}
```

#### 6. Graceful Degradation

```java
@Service
public class RecommendationService {

    @Autowired
    private MLServiceClient mlService;

    @Autowired
    private CacheManager cacheManager;

    public List<Product> getRecommendations(Long userId) {
        try {
            // محاولة الحصول على توصيات ML
            return mlService.getPersonalizedRecommendations(userId);

        } catch (MLServiceException e) {
            log.warn("ML service unavailable, falling back to cached recommendations");

            // الانتقال إلى التوصيات المخزنة
            List<Product> cached = getCachedRecommendations(userId);
            if (cached != null && !cached.isEmpty()) {
                return cached;
            }

            // الانتقال إلى التوصيات الشائعة
            return getPopularProducts();
        }
    }

    private List<Product> getCachedRecommendations(Long userId) {
        Cache cache = cacheManager.getCache("recommendations");
        return cache != null ? cache.get(userId, List.class) : null;
    }

    private List<Product> getPopularProducts() {
        // توصيات افتراضية بسيطة
        return productRepository.findTop10ByOrderBySalesDesc();
    }
}
```

---

## الأمان

### س20: كيف تؤمن Microservices؟
**الجواب:**

#### 1. OAuth2 & JWT

```java
// Authorization Server
@Configuration
@EnableAuthorizationServer
public class AuthServerConfig extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private AuthenticationManager authenticationManager;

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
            .withClient("web-app")
            .secret("{noop}secret")
            .authorizedGrantTypes("password", "refresh_token", "authorization_code")
            .scopes("read", "write")
            .accessTokenValiditySeconds(3600)  // 1 hour
            .refreshTokenValiditySeconds(86400); // 24 hours
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
        endpoints
            .authenticationManager(authenticationManager)
            .tokenStore(tokenStore())
            .accessTokenConverter(accessTokenConverter());
    }

    @Bean
    public TokenStore tokenStore() {
        return new JwtTokenStore(accessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter accessTokenConverter() {
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        converter.setSigningKey("my-secret-key");  // استخدم RSA في الإنتاج
        return converter;
    }
}

// Resource Server (Microservice)
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {

    @Override
    public void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
            .antMatchers("/public/**").permitAll()
            .antMatchers("/api/admin/**").hasRole("ADMIN")
            .antMatchers("/api/**").authenticated()
            .and()
            .csrf().disable();
    }

    @Bean
    public TokenStore tokenStore() {
        return new JwtTokenStore(accessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter accessTokenConverter() {
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        converter.setSigningKey("my-secret-key");
        return converter;
    }
}

// JWT Token Provider
@Component
public class JwtTokenProvider {

    @Value("${jwt.secret}")
    private String secret;

    @Value("${jwt.expiration}")
    private Long expiration;

    public String generateToken(Authentication authentication) {
        UserPrincipal userPrincipal = (UserPrincipal) authentication.getPrincipal();

        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + expiration);

        return Jwts.builder()
            .setSubject(Long.toString(userPrincipal.getId()))
            .claim("email", userPrincipal.getEmail())
            .claim("roles", userPrincipal.getAuthorities().stream()
                .map(GrantedAuthority::getAuthority)
                .collect(Collectors.toList()))
            .setIssuedAt(now)
            .setExpiration(expiryDate)
            .signWith(SignatureAlgorithm.HS512, secret)
            .compact();
    }

    public Long getUserIdFromToken(String token) {
        Claims claims = Jwts.parser()
            .setSigningKey(secret)
            .parseClaimsJws(token)
            .getBody();

        return Long.parseLong(claims.getSubject());
    }

    public boolean validateToken(String token) {
        try {
            Jwts.parser().setSigningKey(secret).parseClaimsJws(token);
            return true;
        } catch (SignatureException ex) {
            log.error("Invalid JWT signature");
        } catch (MalformedJwtException ex) {
            log.error("Invalid JWT token");
        } catch (ExpiredJwtException ex) {
            log.error("Expired JWT token");
        } catch (UnsupportedJwtException ex) {
            log.error("Unsupported JWT token");
        } catch (IllegalArgumentException ex) {
            log.error("JWT claims string is empty");
        }
        return false;
    }
}

// JWT Authentication Filter
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    @Autowired
    private JwtTokenProvider tokenProvider;

    @Autowired
    private UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain)
            throws ServletException, IOException {

        try {
            String jwt = getJwtFromRequest(request);

            if (jwt != null && tokenProvider.validateToken(jwt)) {
                Long userId = tokenProvider.getUserIdFromToken(jwt);

                UserDetails userDetails = userDetailsService.loadUserById(userId);
                UsernamePasswordAuthenticationToken authentication =
                    new UsernamePasswordAuthenticationToken(
                        userDetails,
                        null,
                        userDetails.getAuthorities()
                    );

                authentication.setDetails(
                    new WebAuthenticationDetailsSource().buildDetails(request)
                );

                SecurityContextHolder.getContext().setAuthentication(authentication);
            }
        } catch (Exception ex) {
            log.error("Could not set user authentication", ex);
        }

        filterChain.doFilter(request, response);
    }

    private String getJwtFromRequest(HttpServletRequest request) {
        String bearerToken = request.getHeader("Authorization");
        if (bearerToken != null && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);
        }
        return null;
    }
}
```

#### 2. Service-to-Service Authentication

```java
// Feign Client with OAuth2
@Configuration
public class FeignClientConfig {

    @Bean
    public RequestInterceptor oauth2FeignRequestInterceptor() {
        return requestTemplate -> {
            // الحصول على token من OAuth2
            OAuth2AccessToken accessToken = getAccessToken();
            requestTemplate.header("Authorization", "Bearer " + accessToken.getValue());
        };
    }

    private OAuth2AccessToken getAccessToken() {
        // Client Credentials Grant
        // Implementation here
        return null;
    }
}

// mTLS (Mutual TLS)
@Configuration
public class MutualTLSConfig {

    @Bean
    public RestTemplate mutualTlsRestTemplate() throws Exception {
        // Load client certificate
        KeyStore keyStore = KeyStore.getInstance("PKCS12");
        keyStore.load(
            new FileInputStream("client-cert.p12"),
            "password".toCharArray()
        );

        // Load trusted certificates
        KeyStore trustStore = KeyStore.getInstance("JKS");
        trustStore.load(
            new FileInputStream("truststore.jks"),
            "password".toCharArray()
        );

        SSLContext sslContext = SSLContexts.custom()
            .loadKeyMaterial(keyStore, "password".toCharArray())
            .loadTrustMaterial(trustStore, null)
            .build();

        CloseableHttpClient httpClient = HttpClients.custom()
            .setSSLContext(sslContext)
            .build();

        HttpComponentsClientHttpRequestFactory factory =
            new HttpComponentsClientHttpRequestFactory(httpClient);

        return new RestTemplate(factory);
    }
}
```

#### 3. API Security

```java
// Rate Limiting
@Component
public class RateLimitingFilter extends OncePerRequestFilter {

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain)
            throws ServletException, IOException {

        String clientId = getClientId(request);
        String key = "rate-limit:" + clientId;

        Long requests = redisTemplate.opsForValue().increment(key);

        if (requests == 1) {
            redisTemplate.expire(key, 1, TimeUnit.MINUTES);
        }

        if (requests > 100) { // 100 requests per minute
            response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
            response.getWriter().write("Rate limit exceeded");
            return;
        }

        response.setHeader("X-RateLimit-Limit", "100");
        response.setHeader("X-RateLimit-Remaining", String.valueOf(100 - requests));

        filterChain.doFilter(request, response);
    }

    private String getClientId(HttpServletRequest request) {
        // يمكن استخدام API Key أو User ID أو IP
        String apiKey = request.getHeader("X-API-Key");
        return apiKey != null ? apiKey : request.getRemoteAddr();
    }
}

// Input Validation
@RestController
@Validated
public class UserController {

    @PostMapping("/users")
    public User createUser(@Valid @RequestBody UserRequest request) {
        return userService.createUser(request);
    }
}

@Data
public class UserRequest {

    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 50, message = "Name must be between 2 and 50 characters")
    private String name;

    @NotBlank(message = "Email is required")
    @Email(message = "Email should be valid")
    private String email;

    @NotBlank(message = "Password is required")
    @Pattern(
        regexp = "^(?=.*[0-9])(?=.*[a-z])(?=.*[A-Z])(?=.*[@#$%^&+=])(?=\\S+$).{8,}$",
        message = "Password must be at least 8 characters and contain uppercase, lowercase, digit and special character"
    )
    private String password;

    @Min(value = 18, message = "Age must be at least 18")
    @Max(value = 120, message = "Age must be less than 120")
    private Integer age;
}

// SQL Injection Prevention
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    // ✓ Safe - Parameterized query
    @Query("SELECT u FROM User u WHERE u.email = :email")
    Optional<User> findByEmail(@Param("email") String email);

    // ✓ Safe - Method name query
    Optional<User> findByEmailAndActiveTrue(String email);

    // ❌ Unsafe - Native query with concatenation
    // @Query(value = "SELECT * FROM users WHERE email = '" + email + "'", nativeQuery = true)
    // Don't do this!
}

// XSS Prevention
@Component
public class XSSFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain)
            throws ServletException, IOException {

        XSSRequestWrapper wrappedRequest = new XSSRequestWrapper(request);
        filterChain.doFilter(wrappedRequest, response);
    }
}

public class XSSRequestWrapper extends HttpServletRequestWrapper {

    public XSSRequestWrapper(HttpServletRequest request) {
        super(request);
    }

    @Override
    public String getParameter(String parameter) {
        return cleanXSS(super.getParameter(parameter));
    }

    @Override
    public String[] getParameterValues(String parameter) {
        String[] values = super.getParameterValues(parameter);
        if (values == null) {
            return null;
        }

        return Arrays.stream(values)
            .map(this::cleanXSS)
            .toArray(String[]::new);
    }

    private String cleanXSS(String value) {
        if (value == null) {
            return null;
        }

        value = value.replaceAll("<", "&lt;").replaceAll(">", "&gt;");
        value = value.replaceAll("\\(", "&#40;").replaceAll("\\)", "&#41;");
        value = value.replaceAll("'", "&#39;");
        value = value.replaceAll("eval\\((.*)\\)", "");
        value = value.replaceAll("[\\\"\\\'][\\s]*javascript:(.*)[\\\"\\\']", "\"\"");
        value = value.replaceAll("script", "");

        return value;
    }
}
```

---

## استراتيجيات النشر

### س21: ما هي استراتيجيات نشر Microservices؟
**الجواب:**

#### 1. Blue-Green Deployment

```
Blue Environment (الحالي)      Green Environment (الجديد)
┌──────────────────┐           ┌──────────────────┐
│   Version 1.0    │           │   Version 2.0    │
│                  │           │                  │
│  [Service A]     │           │  [Service A]     │
│  [Service B]     │           │  [Service B]     │
│  [Service C]     │           │  [Service C]     │
└────────┬─────────┘           └────────┬─────────┘
         │                              │
    ┌────▼──────────────────────────────▼────┐
    │         Load Balancer / Router         │
    │    (يوجه 100% traffic إلى Blue)       │
    └───────────────┬────────────────────────┘
                    │
              ┌─────▼──────┐
              │   Users    │
              └────────────┘

بعد التبديل:
    ┌────────────────────────────────────────┐
    │         Load Balancer / Router         │
    │    (يوجه 100% traffic إلى Green)      │
    └───────────────┬────────────────────────┘
```

**مثال - Kubernetes:**

```yaml
# blue-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service-blue
  labels:
    app: user-service
    version: v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: user-service
      version: v1
  template:
    metadata:
      labels:
        app: user-service
        version: v1
    spec:
      containers:
      - name: user-service
        image: user-service:1.0.0
        ports:
        - containerPort: 8080

---
# green-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service-green
  labels:
    app: user-service
    version: v2
spec:
  replicas: 3
  selector:
    matchLabels:
      app: user-service
      version: v2
  template:
    metadata:
      labels:
        app: user-service
        version: v2
    spec:
      containers:
      - name: user-service
        image: user-service:2.0.0
        ports:
        - containerPort: 8080

---
# Service - توجيه Traffic
apiVersion: v1
kind: Service
metadata:
  name: user-service
spec:
  selector:
    app: user-service
    version: v1  # تغيير إلى v2 للتبديل إلى Green
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

**التبديل:**

```bash
# 1. نشر Green environment
kubectl apply -f green-deployment.yaml

# 2. اختبار Green environment
kubectl port-forward deployment/user-service-green 8080:8080
# Test the new version

# 3. التبديل إلى Green
kubectl patch service user-service -p '{"spec":{"selector":{"version":"v2"}}}'

# 4. في حالة المشاكل - Rollback إلى Blue
kubectl patch service user-service -p '{"spec":{"selector":{"version":"v1"}}}'

# 5. بعد التأكد - حذف Blue
kubectl delete deployment user-service-blue
```

#### 2. Canary Deployment

```
تدريجياً زيادة traffic إلى الإصدار الجديد:

Stage 1: 90% → v1.0, 10% → v2.0
┌──────────────┐      ┌──────────────┐
│   v1.0 (9)   │      │   v2.0 (1)   │
└──────┬───────┘      └──────┬───────┘
       │                     │
  ┌────▼─────────────────────▼────┐
  │      Load Balancer            │
  └───────────────────────────────┘

Stage 2: 50% → v1.0, 50% → v2.0

Stage 3: 10% → v1.0, 90% → v2.0

Stage 4: 0% → v1.0, 100% → v2.0
```

**مثال - Istio:**

```yaml
# Virtual Service - Canary Routing
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: user-service
spec:
  hosts:
  - user-service
  http:
  - match:
    - headers:
        canary:
          exact: "true"
    route:
    - destination:
        host: user-service
        subset: v2
      weight: 100

  - route:
    - destination:
        host: user-service
        subset: v1
      weight: 90
    - destination:
        host: user-service
        subset: v2
      weight: 10  # 10% canary traffic

---
# Destination Rule
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: user-service
spec:
  host: user-service
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

**تدريجي Canary:**

```bash
# Stage 1: 10% canary
kubectl apply -f canary-10-percent.yaml

# مراقبة الأداء والأخطاء
# إذا كان كل شيء جيد...

# Stage 2: 50% canary
kubectl apply -f canary-50-percent.yaml

# Stage 3: 100% canary
kubectl apply -f canary-100-percent.yaml
```

#### 3. Rolling Update

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
spec:
  replicas: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2        # عدد pods إضافية أثناء التحديث
      maxUnavailable: 1  # عدد pods يمكن أن تكون غير متاحة
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
    spec:
      containers:
      - name: user-service
        image: user-service:2.0.0
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10
```

```bash
# نشر التحديث
kubectl set image deployment/user-service user-service=user-service:2.0.0

# مراقبة التحديث
kubectl rollout status deployment/user-service

# في حالة المشاكل - Rollback
kubectl rollout undo deployment/user-service

# Rollback إلى نسخة محددة
kubectl rollout undo deployment/user-service --to-revision=2
```

#### 4. A/B Testing

```yaml
# A/B Testing with Istio
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: product-service
spec:
  hosts:
  - product-service
  http:
  - match:
    - headers:
        user-type:
          exact: "premium"
    route:
    - destination:
        host: product-service
        subset: v2-premium

  - match:
    - headers:
        region:
          exact: "us-east"
    route:
    - destination:
        host: product-service
        subset: v2-optimized

  - route:
    - destination:
        host: product-service
        subset: v1
      weight: 70
    - destination:
        host: product-service
        subset: v2
      weight: 30
```

**جدول المقارنة:**

| الاستراتيجية | الوقت | المخاطر | Rollback | متى تُستخدم |
|--------------|-------|---------|----------|--------------|
| **Blue-Green** | سريع | متوسط | فوري | تحديثات كبيرة |
| **Canary** | تدريجي | منخفض | سهل | إصدارات جديدة |
| **Rolling** | متوسط | منخفض | بطيء | تحديثات عادية |
| **A/B Testing** | طويل | منخفض | انتقائي | تجربة ميزات |

---

## السيناريوهات الواقعية

### س22: صمم نظام E-commerce باستخدام Microservices
**الجواب:**

**المعمارية الكاملة:**

```
                        ┌─────────────┐
                        │  Frontend   │
                        │  (React)    │
                        └──────┬──────┘
                               │
                    ┌──────────▼───────────┐
                    │    API Gateway       │
                    │ (Spring Cloud GW)    │
                    └──┬────────┬─────┬────┘
                       │        │     │
        ┌──────────────┼────────┼─────┼────────────┐
        │              │        │     │            │
   ┌────▼─────┐  ┌────▼────┐ ┌─▼──┐ ┌▼──────┐ ┌──▼────┐
   │   User   │  │ Product │ │Cart│ │ Order │ │Payment│
   │ Service  │  │ Service │ │Svc │ │Service│ │Service│
   └────┬─────┘  └────┬────┘ └─┬──┘ └───┬───┘ └───┬───┘
        │             │        │        │         │
   ┌────▼─────┐  ┌───▼────┐ ┌─▼──┐ ┌───▼───┐ ┌───▼───┐
   │ User DB  │  │Prod DB │ │Redis│ │Order │ │Payment│
   │ (MySQL)  │  │ (Pg)   │ │     │ │ DB   │ │  DB   │
   └──────────┘  └────────┘ └────┘ └───┬───┘ └───────┘
                                        │
   ┌────────────┐  ┌──────────┐  ┌─────▼──────┐
   │ Inventory  │  │ Shipping │  │Notification│
   │  Service   │  │ Service  │  │  Service   │
   └────┬───────┘  └────┬─────┘  └─────┬──────┘
        │               │              │
   ┌────▼───────┐  ┌───▼─────┐  ┌─────▼──────┐
   │Inventory DB│  │Shipping │  │   Email    │
   │            │  │   DB    │  │   Queue    │
   └────────────┘  └─────────┘  └────────────┘

   ┌──────────────────────────────────────────┐
   │         Infrastructure Services          │
   ├──────────┬──────────┬──────────┬─────────┤
   │  Eureka  │  Config  │  Zipkin  │ Grafana │
   │          │  Server  │          │         │
   └──────────┴──────────┴──────────┴─────────┘
```

**تفاصيل الخدمات:**

```java
// 1. User Service
@RestController
@RequestMapping("/users")
public class UserController {

    @PostMapping("/register")
    public ResponseEntity<UserDTO> register(@Valid @RequestBody RegisterRequest request) {
        User user = userService.register(request);
        return ResponseEntity.ok(userMapper.toDTO(user));
    }

    @PostMapping("/login")
    public ResponseEntity<LoginResponse> login(@Valid @RequestBody LoginRequest request) {
        String token = authService.authenticate(request.getEmail(), request.getPassword());
        return ResponseEntity.ok(new LoginResponse(token));
    }

    @GetMapping("/{id}")
    public ResponseEntity<UserDTO> getUser(@PathVariable Long id) {
        User user = userService.findById(id);
        return ResponseEntity.ok(userMapper.toDTO(user));
    }
}

// 2. Product Service
@RestController
@RequestMapping("/products")
public class ProductController {

    @GetMapping
    public ResponseEntity<Page<ProductDTO>> searchProducts(
            @RequestParam(required = false) String keyword,
            @RequestParam(required = false) String category,
            @RequestParam(required = false) BigDecimal minPrice,
            @RequestParam(required = false) BigDecimal maxPrice,
            Pageable pageable) {

        Page<Product> products = productService.search(
            keyword, category, minPrice, maxPrice, pageable
        );

        return ResponseEntity.ok(products.map(productMapper::toDTO));
    }

    @GetMapping("/{id}")
    @Cacheable(value = "products", key = "#id")
    public ResponseEntity<ProductDTO> getProduct(@PathVariable Long id) {
        Product product = productService.findById(id);
        return ResponseEntity.ok(productMapper.toDTO(product));
    }
}

// 3. Cart Service
@RestController
@RequestMapping("/cart")
public class CartController {

    @PostMapping("/items")
    public ResponseEntity<CartDTO> addItem(
            @RequestHeader("X-User-ID") Long userId,
            @Valid @RequestBody AddItemRequest request) {

        Cart cart = cartService.addItem(userId, request.getProductId(), request.getQuantity());
        return ResponseEntity.ok(cartMapper.toDTO(cart));
    }

    @DeleteMapping("/items/{productId}")
    public ResponseEntity<Void> removeItem(
            @RequestHeader("X-User-ID") Long userId,
            @PathVariable Long productId) {

        cartService.removeItem(userId, productId);
        return ResponseEntity.noContent().build();
    }

    @GetMapping
    public ResponseEntity<CartDTO> getCart(@RequestHeader("X-User-ID") Long userId) {
        Cart cart = cartService.getCart(userId);
        return ResponseEntity.ok(cartMapper.toDTO(cart));
    }
}

// 4. Order Service - Saga Orchestrator
@Service
public class OrderService {

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private InventoryClient inventoryClient;

    @Autowired
    private PaymentClient paymentClient;

    @Autowired
    private ShippingClient shippingClient;

    @Autowired
    private ApplicationEventPublisher eventPublisher;

    @Transactional
    public Order createOrder(CreateOrderRequest request) {
        // 1. إنشاء الطلب
        Order order = Order.builder()
            .userId(request.getUserId())
            .items(request.getItems())
            .totalAmount(calculateTotal(request.getItems()))
            .status(OrderStatus.PENDING)
            .build();

        order = orderRepository.save(order);

        try {
            // 2. حجز المخزون
            InventoryReservation reservation = inventoryClient.reserve(
                new ReserveRequest(order.getId(), order.getItems())
            );

            order.setInventoryReservationId(reservation.getId());
            order.setStatus(OrderStatus.INVENTORY_RESERVED);
            orderRepository.save(order);

            // 3. معالجة الدفع
            Payment payment = paymentClient.process(
                new PaymentRequest(
                    order.getId(),
                    order.getTotalAmount(),
                    request.getPaymentMethod()
                )
            );

            order.setPaymentId(payment.getId());
            order.setStatus(OrderStatus.PAYMENT_COMPLETED);
            orderRepository.save(order);

            // 4. ترتيب الشحن
            Shipment shipment = shippingClient.arrange(
                new ShipmentRequest(
                    order.getId(),
                    request.getShippingAddress()
                )
            );

            order.setShipmentId(shipment.getId());
            order.setStatus(OrderStatus.CONFIRMED);
            orderRepository.save(order);

            // 5. نشر حدث Order Confirmed
            eventPublisher.publishEvent(new OrderConfirmedEvent(order));

            return order;

        } catch (InventoryException e) {
            order.setStatus(OrderStatus.FAILED);
            order.setFailureReason("Insufficient inventory");
            orderRepository.save(order);
            throw new OrderCreationException("Failed to reserve inventory", e);

        } catch (PaymentException e) {
            // تعويض: إلغاء حجز المخزون
            compensateInventory(order);
            order.setStatus(OrderStatus.FAILED);
            order.setFailureReason("Payment failed");
            orderRepository.save(order);
            throw new OrderCreationException("Payment processing failed", e);

        } catch (ShippingException e) {
            // تعويض: استرجاع الدفع وإلغاء المخزون
            compensatePayment(order);
            compensateInventory(order);
            order.setStatus(OrderStatus.FAILED);
            order.setFailureReason("Shipping arrangement failed");
            orderRepository.save(order);
            throw new OrderCreationException("Failed to arrange shipping", e);
        }
    }

    private void compensateInventory(Order order) {
        if (order.getInventoryReservationId() != null) {
            inventoryClient.cancelReservation(order.getInventoryReservationId());
        }
    }

    private void compensatePayment(Order order) {
        if (order.getPaymentId() != null) {
            paymentClient.refund(order.getPaymentId());
        }
    }

    private BigDecimal calculateTotal(List<OrderItem> items) {
        return items.stream()
            .map(item -> item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}

// 5. Inventory Service
@Service
public class InventoryService {

    @Autowired
    private InventoryRepository inventoryRepository;

    @Transactional
    public InventoryReservation reserve(ReserveRequest request) {
        for (OrderItem item : request.getItems()) {
            Inventory inventory = inventoryRepository
                .findByProductIdWithLock(item.getProductId())
                .orElseThrow(() -> new ProductNotFoundException(item.getProductId()));

            if (inventory.getAvailableQuantity() < item.getQuantity()) {
                throw new InsufficientInventoryException(
                    item.getProductId(),
                    inventory.getAvailableQuantity(),
                    item.getQuantity()
                );
            }

            inventory.setAvailableQuantity(
                inventory.getAvailableQuantity() - item.getQuantity()
            );
            inventory.setReservedQuantity(
                inventory.getReservedQuantity() + item.getQuantity()
            );

            inventoryRepository.save(inventory);
        }

        return InventoryReservation.builder()
            .id(UUID.randomUUID().toString())
            .orderId(request.getOrderId())
            .items(request.getItems())
            .status(ReservationStatus.RESERVED)
            .build();
    }

    @Transactional
    public void cancelReservation(String reservationId) {
        InventoryReservation reservation = reservationRepository
            .findById(reservationId)
            .orElseThrow();

        for (OrderItem item : reservation.getItems()) {
            Inventory inventory = inventoryRepository
                .findByProductId(item.getProductId())
                .orElseThrow();

            inventory.setAvailableQuantity(
                inventory.getAvailableQuantity() + item.getQuantity()
            );
            inventory.setReservedQuantity(
                inventory.getReservedQuantity() - item.getQuantity()
            );

            inventoryRepository.save(inventory);
        }

        reservation.setStatus(ReservationStatus.CANCELLED);
        reservationRepository.save(reservation);
    }
}

// 6. Payment Service
@Service
public class PaymentService {

    @Autowired
    private PaymentRepository paymentRepository;

    @Autowired
    private PaymentGateway paymentGateway;

    @CircuitBreaker(name = "payment-gateway", fallbackMethod = "processPaymentFallback")
    @Retry(name = "payment-gateway")
    @Transactional
    public Payment process(PaymentRequest request) {
        Payment payment = Payment.builder()
            .orderId(request.getOrderId())
            .amount(request.getAmount())
            .method(request.getPaymentMethod())
            .status(PaymentStatus.PROCESSING)
            .build();

        payment = paymentRepository.save(payment);

        try {
            // استدعاء Payment Gateway الخارجي
            PaymentGatewayResponse response = paymentGateway.charge(
                request.getAmount(),
                request.getPaymentMethod()
            );

            if (response.isSuccess()) {
                payment.setStatus(PaymentStatus.COMPLETED);
                payment.setTransactionId(response.getTransactionId());
            } else {
                payment.setStatus(PaymentStatus.FAILED);
                payment.setFailureReason(response.getErrorMessage());
                throw new PaymentException(response.getErrorMessage());
            }

            paymentRepository.save(payment);
            return payment;

        } catch (Exception e) {
            payment.setStatus(PaymentStatus.FAILED);
            payment.setFailureReason(e.getMessage());
            paymentRepository.save(payment);
            throw new PaymentException("Payment processing failed", e);
        }
    }

    private Payment processPaymentFallback(PaymentRequest request, Exception ex) {
        log.error("Payment gateway unavailable, using fallback", ex);
        throw new PaymentGatewayUnavailableException(
            "Payment service is temporarily unavailable. Please try again later."
        );
    }

    @Transactional
    public void refund(Long paymentId) {
        Payment payment = paymentRepository.findById(paymentId)
            .orElseThrow();

        if (payment.getStatus() != PaymentStatus.COMPLETED) {
            throw new IllegalStateException("Cannot refund non-completed payment");
        }

        try {
            paymentGateway.refund(payment.getTransactionId());

            payment.setStatus(PaymentStatus.REFUNDED);
            paymentRepository.save(payment);

        } catch (Exception e) {
            throw new RefundException("Refund failed", e);
        }
    }
}

// 7. Notification Service - Event-Driven
@Component
public class NotificationEventListener {

    @Autowired
    private EmailService emailService;

    @Autowired
    private SmsService smsService;

    @RabbitListener(queues = "notification.order.queue")
    public void handleOrderConfirmed(OrderConfirmedEvent event) {
        // إرسال بريد إلكتروني
        emailService.sendOrderConfirmation(
            event.getUserEmail(),
            event.getOrderId(),
            event.getOrderDetails()
        );

        // إرسال SMS
        smsService.sendOrderConfirmation(
            event.getUserPhone(),
            event.getOrderId()
        );
    }

    @RabbitListener(queues = "notification.shipping.queue")
    public void handleShipmentDispatched(ShipmentDispatchedEvent event) {
        emailService.sendShippingNotification(
            event.getUserEmail(),
            event.getTrackingNumber()
        );
    }
}
```

**Docker Compose للبيئة المحلية:**

```yaml
version: '3.8'

services:
  # Infrastructure
  eureka:
    image: eureka-server:latest
    ports:
      - "8761:8761"

  config-server:
    image: config-server:latest
    ports:
      - "8888:8888"
    environment:
      - EUREKA_URL=http://eureka:8761/eureka

  api-gateway:
    image: api-gateway:latest
    ports:
      - "8080:8080"
    environment:
      - EUREKA_URL=http://eureka:8761/eureka
    depends_on:
      - eureka

  # Services
  user-service:
    image: user-service:latest
    environment:
      - SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/user_db
      - EUREKA_URL=http://eureka:8761/eureka
    depends_on:
      - mysql
      - eureka

  product-service:
    image: product-service:latest
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/product_db
      - EUREKA_URL=http://eureka:8761/eureka
    depends_on:
      - postgres
      - eureka

  order-service:
    image: order-service:latest
    environment:
      - SPRING_DATA_MONGODB_URI=mongodb://mongo:27017/order_db
      - EUREKA_URL=http://eureka:8761/eureka
    depends_on:
      - mongo
      - eureka

  # Databases
  mysql:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=user_db
    ports:
      - "3306:3306"

  postgres:
    image: postgres:14
    environment:
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=product_db
    ports:
      - "5432:5432"

  mongo:
    image: mongo:5
    ports:
      - "27017:27017"

  redis:
    image: redis:7
    ports:
      - "6379:6379"

  # Message Queue
  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"
      - "15672:15672"

  # Monitoring
  zipkin:
    image: openzipkin/zipkin
    ports:
      - "9411:9411"

  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
```

---

## الخلاصة والنصائح النهائية

### نصائح للمقابلات:

1. **فهم المبادئ قبل التفاصيل**
   - افهم لماذا نستخدم pattern معين
   - متى نستخدمه ومتى لا نستخدمه

2. **التطبيق العملي**
   - اذكر أمثلة من تجاربك
   - كن مستعداً لكتابة كود

3. **Trade-offs**
   - كل قرار له مزايا وعيوب
   - ناقش البدائل

4. **المقياس (Scale)**
   - كيف يتصرف النظام تحت الضغط؟
   - استراتيجيات التوسع

5. **الأمان دائماً**
   - لا تنسَ الأمان في التصميم
   - Authentication, Authorization, Encryption

### الأسئلة الأكثر شيوعاً:

1. ما الفرق بين Microservices و Monolithic؟
2. كيف تتعامل مع Distributed Transactions؟
3. اشرح Service Discovery
4. ما هو Circuit Breaker ولماذا نحتاجه؟
5. كيف تراقب Microservices في Production؟
6. اشرح Database per Service pattern
7. كيف تؤمن التواصل بين الخدمات؟
8. ما هي استراتيجيات النشر؟
9. صمم نظام [E-commerce/Social Media/...] باستخدام Microservices
10. كيف تتعامل مع Eventual Consistency؟

### موارد للتعلم:

1. **كتب:**
   - Building Microservices - Sam Newman
   - Microservices Patterns - Chris Richardson
   - Domain-Driven Design - Eric Evans

2. **مواقع:**
   - microservices.io
   - martinfowler.com
   - Netflix Tech Blog

3. **تقنيات للممارسة:**
   - Spring Boot / Spring Cloud
   - Docker & Kubernetes
   - RabbitMQ / Kafka
   - Prometheus & Grafana

---

**حظاً موفقاً في مقابلاتك!**
