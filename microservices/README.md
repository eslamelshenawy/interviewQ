# دليل معمارية الخدمات الصغيرة (Microservices)

## جدول المحتويات
- [مقدمة](#مقدمة)
- [ما هي الخدمات الصغيرة؟](#ما-هي-الخدمات-الصغيرة)
- [المبادئ الأساسية](#المبادئ-الأساسية)
- [المزايا والعيوب](#المزايا-والعيوب)
- [العمارة والمكونات](#العمارة-والمكونات)
- [الأنماط الأساسية](#الأنماط-الأساسية)
- [التقنيات والأدوات](#التقنيات-والأدوات)
- [أفضل الممارسات](#أفضل-الممارسات)

---

## مقدمة

معمارية الخدمات الصغيرة (Microservices Architecture) هي نهج لتطوير التطبيقات حيث يتم بناء التطبيق كمجموعة من الخدمات الصغيرة المستقلة، كل منها يعمل في عملية خاصة به ويتواصل مع الآخرين عبر آليات خفيفة الوزن.

## ما هي الخدمات الصغيرة؟

### التعريف
الخدمات الصغيرة هي نمط معماري يقوم بتقسيم التطبيق إلى مجموعة من الخدمات الصغيرة المستقلة، حيث:
- كل خدمة تركز على وظيفة عمل محددة
- تعمل كل خدمة بشكل مستقل
- يمكن نشر وتوسيع كل خدمة بشكل مستقل
- تتواصل الخدمات مع بعضها عبر APIs خفيفة الوزن

### مثال توضيحي

```
التطبيق الأحادي (Monolithic):
┌─────────────────────────────────────┐
│                                     │
│  User Management + Products +      │
│  Orders + Payments + Shipping      │
│  (كل شيء في تطبيق واحد)            │
│                                     │
└─────────────────────────────────────┘

الخدمات الصغيرة (Microservices):
┌──────────┐  ┌──────────┐  ┌──────────┐
│  Users   │  │ Products │  │  Orders  │
│ Service  │  │ Service  │  │ Service  │
└──────────┘  └──────────┘  └──────────┘
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Payment  │  │ Shipping │  │  Email   │
│ Service  │  │ Service  │  │ Service  │
└──────────┘  └──────────┘  └──────────┘
```

## المبادئ الأساسية

### 1. الاستقلالية (Autonomy)
كل خدمة:
- تمتلك قاعدة بيانات خاصة بها
- يمكن تطويرها بلغة برمجة مختلفة
- لها دورة حياة نشر مستقلة

### 2. التركيز على المجال (Domain-Driven)
- كل خدمة تمثل قدرة عمل محددة
- حدود واضحة بين الخدمات
- تطبيق مبادئ Domain-Driven Design (DDD)

### 3. اللامركزية (Decentralization)
- لا توجد نقطة مركزية للفشل
- إدارة البيانات اللامركزية
- اتخاذ القرارات اللامركزي

### 4. المرونة (Resilience)
- القدرة على التعامل مع الأخطاء
- العزل بين الخدمات
- آليات التعافي التلقائي

## المزايا والعيوب

### المزايا

#### 1. قابلية التوسع المستقلة
```
مثال: موقع تجارة إلكترونية
- خدمة المنتجات: 2 instances
- خدمة الطلبات: 5 instances (ذروة الطلبات)
- خدمة الدفع: 3 instances
```

#### 2. المرونة التكنولوجية
```java
// User Service - Java/Spring Boot
@RestController
public class UserController {
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);
    }
}
```

```python
# Product Service - Python/FastAPI
@app.get("/products/{product_id}")
async def get_product(product_id: int):
    return await product_service.get(product_id)
```

```javascript
// Order Service - Node.js/Express
app.get('/orders/:id', async (req, res) => {
    const order = await orderService.findById(req.params.id);
    res.json(order);
});
```

#### 3. سهولة النشر والتحديث
- تحديث خدمة واحدة دون التأثير على الباقي
- rollback سريع في حالة الفشل
- تقليل وقت التوقف

#### 4. الفرق المستقلة
- كل فريق مسؤول عن خدمة محددة
- تطوير أسرع
- ملكية واضحة

### العيوب

#### 1. التعقيد الموزع
- إدارة الاتصال بين الخدمات
- صعوبة تتبع الأخطاء
- الحاجة لأدوات مراقبة متقدمة

#### 2. اتساق البيانات
- صعوبة تنفيذ المعاملات الموزعة
- الحاجة لأنماط مثل Saga Pattern
- eventual consistency بدلاً من strong consistency

#### 3. زيادة استهلاك الموارد
- كل خدمة تحتاج موارد خاصة
- زيادة في الذاكرة والمعالجة
- تكلفة البنية التحتية

#### 4. تحديات الاختبار
- اختبارات integration معقدة
- الحاجة لبيئات اختبار كاملة
- صعوبة اختبار السيناريوهات الشاملة

## العمارة والمكونات

### المكونات الأساسية

```
                     ┌─────────────────┐
                     │   API Gateway   │
                     └────────┬────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
   ┌────▼────┐          ┌────▼────┐          ┌────▼────┐
   │ Service │          │ Service │          │ Service │
   │    A    │◄────────►│    B    │◄────────►│    C    │
   └────┬────┘          └────┬────┘          └────┬────┘
        │                    │                    │
   ┌────▼────┐          ┌────▼────┐          ┌────▼────┐
   │   DB    │          │   DB    │          │   DB    │
   │    A    │          │    B    │          │    C    │
   └─────────┘          └─────────┘          └─────────┘

        ┌──────────────────────────────────┐
        │   Service Discovery (Eureka)     │
        └──────────────────────────────────┘

        ┌──────────────────────────────────┐
        │   Config Server                  │
        └──────────────────────────────────┘
```

### 1. API Gateway
البوابة الموحدة للدخول إلى جميع الخدمات

**المسؤوليات:**
- توجيه الطلبات (Routing)
- المصادقة والتفويض (Authentication & Authorization)
- تحديد المعدل (Rate Limiting)
- تجميع الاستجابات (Response Aggregation)
- تحويل البروتوكولات

**مثال: Spring Cloud Gateway**

```java
@Configuration
public class GatewayConfig {

    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            // User Service Route
            .route("user-service", r -> r.path("/api/users/**")
                .filters(f -> f
                    .rewritePath("/api/users/(?<segment>.*)", "/${segment}")
                    .addRequestHeader("X-Request-Source", "Gateway"))
                .uri("lb://USER-SERVICE"))

            // Product Service Route
            .route("product-service", r -> r.path("/api/products/**")
                .filters(f -> f
                    .circuitBreaker(config -> config
                        .setName("productServiceCircuitBreaker")
                        .setFallbackUri("forward:/fallback/products"))
                    .retry(config -> config.setRetries(3)))
                .uri("lb://PRODUCT-SERVICE"))

            // Order Service Route
            .route("order-service", r -> r.path("/api/orders/**")
                .filters(f -> f
                    .requestRateLimiter(config -> config
                        .setRateLimiter(redisRateLimiter())
                        .setKeyResolver(userKeyResolver())))
                .uri("lb://ORDER-SERVICE"))
            .build();
    }
}
```

### 2. Service Discovery
اكتشاف وتسجيل الخدمات تلقائياً

**مثال: Netflix Eureka**

```java
// Eureka Server
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

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
  server:
    enable-self-preservation: false
```

```java
// Microservice Client
@SpringBootApplication
@EnableEurekaClient
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}

// application.yml
spring:
  application:
    name: user-service

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
    lease-renewal-interval-in-seconds: 30
```

### 3. Config Server
إدارة الإعدادات المركزية

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}

// application.yml
server:
  port: 8888

spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/your-org/config-repo
          default-label: main
          search-paths: '{application}'
```

### 4. Load Balancer
توزيع الحمل بين نسخ الخدمات

```java
@Configuration
public class LoadBalancerConfig {

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    @LoadBalanced
    @Bean
    public WebClient.Builder webClientBuilder() {
        return WebClient.builder();
    }
}

// الاستخدام
@Service
public class OrderService {

    @Autowired
    private WebClient.Builder webClientBuilder;

    public User getUserById(Long userId) {
        return webClientBuilder.build()
            .get()
            .uri("http://USER-SERVICE/users/" + userId)
            .retrieve()
            .bodyToMono(User.class)
            .block();
    }
}
```

## الأنماط الأساسية

### 1. API Gateway Pattern
بوابة موحدة للوصول إلى الخدمات

### 2. Circuit Breaker Pattern
منع انتشار الأعطال

```java
@Service
public class ProductService {

    @CircuitBreaker(name = "productService", fallbackMethod = "getProductFallback")
    @Retry(name = "productService", fallbackMethod = "getProductFallback")
    @RateLimiter(name = "productService")
    public Product getProduct(Long id) {
        return restTemplate.getForObject(
            "http://PRODUCT-SERVICE/products/" + id,
            Product.class
        );
    }

    private Product getProductFallback(Long id, Exception ex) {
        return Product.builder()
            .id(id)
            .name("Product Temporarily Unavailable")
            .price(0.0)
            .build();
    }
}
```

### 3. Saga Pattern
إدارة المعاملات الموزعة

```java
// Orchestration-based Saga
@Service
public class OrderSagaOrchestrator {

    public void createOrder(OrderRequest request) {
        try {
            // Step 1: Create Order
            Order order = orderService.createOrder(request);

            // Step 2: Reserve Inventory
            inventoryService.reserveItems(order.getItems());

            // Step 3: Process Payment
            paymentService.processPayment(order.getPayment());

            // Step 4: Arrange Shipping
            shippingService.arrangeShipping(order.getShippingInfo());

            // Success: Confirm Order
            orderService.confirmOrder(order.getId());

        } catch (Exception e) {
            // Compensating Transactions
            compensate(order);
        }
    }

    private void compensate(Order order) {
        try {
            shippingService.cancelShipping(order.getId());
            paymentService.refund(order.getPayment());
            inventoryService.releaseItems(order.getItems());
            orderService.cancelOrder(order.getId());
        } catch (Exception e) {
            log.error("Compensation failed", e);
        }
    }
}
```

### 4. CQRS Pattern
فصل القراءة عن الكتابة

```java
// Command Side
@Service
public class UserCommandService {

    @Autowired
    private UserWriteRepository writeRepository;

    @Autowired
    private EventPublisher eventPublisher;

    public void createUser(CreateUserCommand command) {
        User user = new User(command.getName(), command.getEmail());
        writeRepository.save(user);

        eventPublisher.publish(new UserCreatedEvent(user.getId(), user.getName()));
    }
}

// Query Side
@Service
public class UserQueryService {

    @Autowired
    private UserReadRepository readRepository;

    public UserDTO getUserById(Long id) {
        return readRepository.findById(id)
            .map(this::toDTO)
            .orElseThrow(() -> new UserNotFoundException(id));
    }

    public List<UserDTO> searchUsers(String query) {
        return readRepository.searchByNameOrEmail(query)
            .stream()
            .map(this::toDTO)
            .collect(Collectors.toList());
    }
}
```

### 5. Event Sourcing Pattern
تخزين التغييرات كأحداث

```java
@Entity
public class EventStore {
    @Id
    private String eventId;
    private String aggregateId;
    private String eventType;
    private String eventData;
    private LocalDateTime timestamp;
}

@Service
public class AccountEventSourcingService {

    public void deposit(String accountId, BigDecimal amount) {
        AccountDepositedEvent event = new AccountDepositedEvent(
            accountId, amount, LocalDateTime.now()
        );
        eventStore.save(event);
        applyEvent(event);
    }

    public Account reconstructAccount(String accountId) {
        List<Event> events = eventStore.findByAggregateId(accountId);
        Account account = new Account(accountId);

        events.forEach(event -> {
            if (event instanceof AccountDepositedEvent) {
                account.apply((AccountDepositedEvent) event);
            } else if (event instanceof AccountWithdrawnEvent) {
                account.apply((AccountWithdrawnEvent) event);
            }
        });

        return account;
    }
}
```

## التقنيات والأدوات

### 1. أطر العمل (Frameworks)

#### Spring Boot / Spring Cloud
```xml
<dependencies>
    <!-- Spring Boot Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Eureka Client -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>

    <!-- Config Client -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>

    <!-- Circuit Breaker -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
    </dependency>
</dependencies>
```

### 2. التواصل بين الخدمات

#### REST API
```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    @GetMapping("/{id}")
    public ResponseEntity<Product> getProduct(@PathVariable Long id) {
        return ResponseEntity.ok(productService.findById(id));
    }

    @PostMapping
    public ResponseEntity<Product> createProduct(@RequestBody Product product) {
        Product created = productService.create(product);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }
}
```

#### gRPC
```protobuf
syntax = "proto3";

service ProductService {
    rpc GetProduct (ProductRequest) returns (ProductResponse);
    rpc ListProducts (ListProductsRequest) returns (stream ProductResponse);
}

message ProductRequest {
    int64 id = 1;
}

message ProductResponse {
    int64 id = 1;
    string name = 2;
    double price = 3;
}
```

#### Message Queues (RabbitMQ)
```java
@Configuration
public class RabbitMQConfig {

    @Bean
    public Queue orderQueue() {
        return new Queue("order.queue", true);
    }

    @Bean
    public TopicExchange orderExchange() {
        return new TopicExchange("order.exchange");
    }

    @Bean
    public Binding orderBinding() {
        return BindingBuilder
            .bind(orderQueue())
            .to(orderExchange())
            .with("order.created");
    }
}

@Service
public class OrderEventPublisher {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void publishOrderCreated(Order order) {
        rabbitTemplate.convertAndSend(
            "order.exchange",
            "order.created",
            order
        );
    }
}

@Component
public class OrderEventListener {

    @RabbitListener(queues = "order.queue")
    public void handleOrderCreated(Order order) {
        log.info("Received order: {}", order.getId());
        // Process order
    }
}
```

### 3. قواعد البيانات

#### Database per Service
```yaml
# User Service
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/user_db
    username: user_service
    password: ${DB_PASSWORD}

# Product Service
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/product_db
    username: product_service
    password: ${DB_PASSWORD}

# Order Service
spring:
  data:
    mongodb:
      uri: mongodb://localhost:27017/order_db
```

### 4. المراقبة والتتبع

#### Distributed Tracing (Sleuth + Zipkin)
```xml
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
spring:
  sleuth:
    sampler:
      probability: 1.0
  zipkin:
    base-url: http://localhost:9411
```

#### Metrics (Prometheus + Grafana)
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  metrics:
    export:
      prometheus:
        enabled: true
```

## أفضل الممارسات

### 1. تصميم الخدمات
- اتبع مبدأ Single Responsibility
- حافظ على حجم الخدمات صغيراً
- صمم للفشل (Design for Failure)
- استخدم Domain-Driven Design

### 2. إدارة البيانات
- قاعدة بيانات لكل خدمة
- تجنب الاعتماديات المباشرة بين قواعد البيانات
- استخدم Event-Driven Architecture للاتساق
- طبق CQRS عند الحاجة

### 3. الأمان
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .oauth2ResourceServer()
            .jwt()
            .jwtAuthenticationConverter(jwtAuthConverter());
    }

    @Bean
    public JwtAuthenticationConverter jwtAuthConverter() {
        JwtAuthenticationConverter converter = new JwtAuthenticationConverter();
        converter.setJwtGrantedAuthoritiesConverter(new KeycloakRoleConverter());
        return converter;
    }
}
```

### 4. المراقبة والسجلات
```java
@Slf4j
@RestController
public class UserController {

    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        log.info("Fetching user with id: {}", id);

        try {
            User user = userService.findById(id);
            log.info("User found: {}", user.getEmail());
            return user;
        } catch (UserNotFoundException e) {
            log.error("User not found: {}", id, e);
            throw e;
        }
    }
}
```

### 5. النشر والتوسع
```yaml
# Kubernetes Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
spec:
  replicas: 3
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
        image: user-service:1.0.0
        ports:
        - containerPort: 8080
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: production
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
```

---

## الخلاصة

معمارية الخدمات الصغيرة توفر مرونة وقابلية توسع عالية، لكنها تأتي مع تعقيدات إضافية. النجاح في تطبيقها يتطلب:

1. فهم عميق للمبادئ والأنماط
2. استخدام الأدوات المناسبة
3. تطبيق أفضل الممارسات
4. مراقبة وصيانة مستمرة

للمزيد من التفاصيل والأسئلة الشائعة، راجع ملف [Microservices-Interview-Questions.md](./Microservices-Interview-Questions.md)
