# Spring Cloud - دليل شامل

## المحتويات
1. [ما هو Spring Cloud](#ما-هو-spring-cloud)
2. [المكونات الرئيسية](#المكونات-الرئيسية)
3. [Service Discovery (Eureka)](#service-discovery-eureka)
4. [API Gateway (Spring Cloud Gateway)](#api-gateway-spring-cloud-gateway)
5. [Configuration Management (Config Server)](#configuration-management-config-server)
6. [Load Balancing (Spring Cloud LoadBalancer)](#load-balancing-spring-cloud-loadbalancer)
7. [Circuit Breaker (Resilience4j)](#circuit-breaker-resilience4j)
8. [Distributed Tracing (Sleuth & Zipkin)](#distributed-tracing-sleuth--zipkin)
9. [Message Bus (Spring Cloud Bus)](#message-bus-spring-cloud-bus)
10. [Security (OAuth2)](#security-oauth2)
11. [أمثلة عملية](#أمثلة-عملية)
12. [Best Practices](#best-practices)
13. [أسئلة المقابلات](#أسئلة-المقابلات)

---

## ما هو Spring Cloud

**Spring Cloud** هو مجموعة من الأدوات والمكتبات لبناء **Microservices** و**Distributed Systems**. يوفر حلولاً للتحديات الشائعة مثل:

### المزايا الرئيسية
- **Service Discovery**: اكتشاف وتسجيل الخدمات
- **API Gateway**: نقطة دخول موحدة
- **Configuration Management**: إدارة التهيئة مركزياً
- **Load Balancing**: توزيع الحمل
- **Circuit Breaker**: منع تعطل متسلسل
- **Distributed Tracing**: تتبع الطلبات عبر Services
- **Message Bus**: تبادل الرسائل بين Services
- **Security**: أمان موحد (OAuth2, JWT)

### Spring Cloud مقابل Spring Boot

| الميزة | Spring Boot | Spring Cloud |
|--------|-------------|--------------|
| **الهدف** | بناء تطبيقات مستقلة | بناء Distributed Systems |
| **المستوى** | Single Application | Multiple Applications |
| **التركيز** | Configuration، Packaging | Communication، Coordination |
| **مثال** | REST API واحد | نظام من Microservices |

---

## المكونات الرئيسية

### 1. Spring Cloud Netflix
- **Eureka**: Service Discovery
- **Zuul**: API Gateway (Deprecated، استخدم Spring Cloud Gateway)
- **Hystrix**: Circuit Breaker (Maintenance Mode، استخدم Resilience4j)
- **Ribbon**: Load Balancing (Deprecated، استخدم Spring Cloud LoadBalancer)

### 2. Spring Cloud Gateway
- API Gateway حديث
- مبني على Spring WebFlux (Reactive)
- Routing، Filtering، Load Balancing

### 3. Spring Cloud Config
- Configuration Server مركزي
- دعم Git، SVN، File System
- Refresh Configuration بدون Restart

### 4. Spring Cloud LoadBalancer
- Client-side Load Balancing
- بديل Ribbon

### 5. Resilience4j
- Circuit Breaker، Retry، Rate Limiter
- بديل Hystrix

### 6. Spring Cloud Sleuth
- Distributed Tracing
- Integration مع Zipkin

### 7. Spring Cloud Bus
- Message Bus للتواصل بين Services
- استخدام RabbitMQ أو Kafka

### 8. Spring Cloud Security
- OAuth2، JWT
- Single Sign-On (SSO)

### 9. Spring Cloud Stream
- Message-driven Microservices
- دعم Kafka، RabbitMQ

### 10. Spring Cloud OpenFeign
- Declarative REST Client
- تسهيل Communication بين Services

---

## Service Discovery (Eureka)

### ما هو Service Discovery؟

**Service Discovery** يسمح للـ Microservices باكتشاف بعضها تلقائياً بدون Hard-coded URLs.

### Eureka Server

**1. Dependencies:**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

**2. Application Class:**
```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

**3. Configuration (application.yml):**
```yaml
server:
  port: 8761

eureka:
  client:
    register-with-eureka: false  # Server لا يسجل نفسه
    fetch-registry: false         # Server لا يجلب Registry
  server:
    enable-self-preservation: false  # للـ Development فقط
```

### Eureka Client

**1. Dependencies:**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

**2. Application Class:**
```java
@SpringBootApplication
@EnableDiscoveryClient  // أو @EnableEurekaClient
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}
```

**3. Configuration:**
```yaml
server:
  port: 8081

spring:
  application:
    name: user-service  # اسم الخدمة في Eureka

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
    hostname: localhost
```

### استخدام Service Discovery

```java
@RestController
public class UserController {

    @Autowired
    private DiscoveryClient discoveryClient;

    @GetMapping("/services")
    public List<String> getServices() {
        return discoveryClient.getServices();
    }

    @GetMapping("/instances/{serviceName}")
    public List<ServiceInstance> getInstances(@PathVariable String serviceName) {
        return discoveryClient.getInstances(serviceName);
    }
}
```

---

## API Gateway (Spring Cloud Gateway)

### ما هو API Gateway؟

**API Gateway** هو نقطة دخول موحدة لجميع Microservices. يوفر:
- **Routing**: توجيه الطلبات
- **Load Balancing**: توزيع الحمل
- **Authentication**: مصادقة موحدة
- **Rate Limiting**: تحديد عدد الطلبات
- **Logging & Monitoring**: تسجيل ومراقبة

### Dependencies

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
</dependencies>
```

### Configuration-based Routing

```yaml
spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      routes:
        # User Service Route
        - id: user-service
          uri: lb://USER-SERVICE  # Load-balanced من Eureka
          predicates:
            - Path=/api/users/**
          filters:
            - StripPrefix=1  # إزالة /api من المسار

        # Order Service Route
        - id: order-service
          uri: lb://ORDER-SERVICE
          predicates:
            - Path=/api/orders/**
          filters:
            - StripPrefix=1

        # Product Service Route
        - id: product-service
          uri: lb://PRODUCT-SERVICE
          predicates:
            - Path=/api/products/**
            - Method=GET,POST
          filters:
            - AddRequestHeader=X-Gateway, SpringCloudGateway
            - AddResponseHeader=X-Response-Time, ${response.time}

      # Global CORS Configuration
      globalcors:
        corsConfigurations:
          '[/**]':
            allowedOrigins: "*"
            allowedMethods:
              - GET
              - POST
              - PUT
              - DELETE
            allowedHeaders: "*"

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/

server:
  port: 8080
```

### Programmatic Routing

```java
@Configuration
public class GatewayConfig {

    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            // User Service
            .route("user-service", r -> r
                .path("/api/users/**")
                .filters(f -> f
                    .stripPrefix(1)
                    .addRequestHeader("X-Gateway", "SpringCloudGateway")
                    .retry(config -> config
                        .setRetries(3)
                        .setStatuses(HttpStatus.INTERNAL_SERVER_ERROR)))
                .uri("lb://USER-SERVICE"))

            // Order Service with Circuit Breaker
            .route("order-service", r -> r
                .path("/api/orders/**")
                .filters(f -> f
                    .stripPrefix(1)
                    .circuitBreaker(config -> config
                        .setName("orderServiceCircuitBreaker")
                        .setFallbackUri("forward:/fallback/orders")))
                .uri("lb://ORDER-SERVICE"))

            // Redirect Route
            .route("redirect-route", r -> r
                .path("/old/**")
                .filters(f -> f
                    .setPath("/new/{segment}"))
                .uri("lb://NEW-SERVICE"))

            .build();
    }
}
```

### Custom Filters

#### Global Filter
```java
@Component
public class LoggingGlobalFilter implements GlobalFilter, Ordered {

    private static final Logger log = LoggerFactory.getLogger(LoggingGlobalFilter.class);

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("Request Path: {}", exchange.getRequest().getPath());
        log.info("Request Method: {}", exchange.getRequest().getMethod());

        return chain.filter(exchange).then(Mono.fromRunnable(() -> {
            log.info("Response Status Code: {}",
                exchange.getResponse().getStatusCode());
        }));
    }

    @Override
    public int getOrder() {
        return -1;  // أول Filter
    }
}
```

#### Gateway Filter Factory
```java
@Component
public class CustomHeaderGatewayFilterFactory
        extends AbstractGatewayFilterFactory<CustomHeaderGatewayFilterFactory.Config> {

    public CustomHeaderGatewayFilterFactory() {
        super(Config.class);
    }

    @Override
    public GatewayFilter apply(Config config) {
        return (exchange, chain) -> {
            exchange.getRequest()
                .mutate()
                .header(config.getHeaderName(), config.getHeaderValue())
                .build();

            return chain.filter(exchange);
        };
    }

    public static class Config {
        private String headerName;
        private String headerValue;

        // getters, setters
    }
}

// استخدام في Configuration
@Bean
public RouteLocator routes(RouteLocatorBuilder builder, CustomHeaderGatewayFilterFactory factory) {
    return builder.routes()
        .route(r -> r
            .path("/api/**")
            .filters(f -> f
                .filter(factory.apply(new CustomHeaderGatewayFilterFactory.Config(
                    "X-Custom-Header", "CustomValue"))))
            .uri("lb://SERVICE"))
        .build();
}
```

### Rate Limiting

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://USER-SERVICE
          predicates:
            - Path=/api/users/**
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10  # طلبات في الثانية
                redis-rate-limiter.burstCapacity: 20  # Max burst
                key-resolver: "#{@userKeyResolver}"
```

```java
@Bean
public KeyResolver userKeyResolver() {
    return exchange -> Mono.just(
        exchange.getRequest()
            .getRemoteAddress()
            .getAddress()
            .getHostAddress()
    );
}
```

---

## Configuration Management (Config Server)

### Config Server

**1. Dependencies:**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

**2. Application Class:**
```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

**3. Configuration:**
```yaml
server:
  port: 8888

spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: https://github.com/username/config-repo
          default-label: main
          search-paths: config
          username: your-username  # إذا private
          password: your-password

        # أو استخدام Native (File System)
        native:
          search-locations: classpath:/config,file:./config
```

**هيكل Config Repository:**
```
config-repo/
├── application.yml            # مشترك لجميع Services
├── user-service.yml           # خاص بـ user-service
├── order-service.yml          # خاص بـ order-service
├── user-service-dev.yml       # user-service في Dev
├── user-service-prod.yml      # user-service في Production
```

### Config Client

**1. Dependencies:**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

**2. bootstrap.yml:**
```yaml
spring:
  application:
    name: user-service  # اسم الملف في Config Server
  profiles:
    active: dev         # Environment
  cloud:
    config:
      uri: http://localhost:8888
      fail-fast: true
      retry:
        max-attempts: 6
        max-interval: 2000

management:
  endpoints:
    web:
      exposure:
        include: refresh  # لـ Dynamic Refresh
```

**3. استخدام Configuration:**
```java
@RestController
@RefreshScope  // للـ Dynamic Refresh
public class ConfigController {

    @Value("${app.message:Default Message}")
    private String message;

    @Value("${app.feature.enabled:false}")
    private boolean featureEnabled;

    @GetMapping("/config")
    public Map<String, Object> getConfig() {
        return Map.of(
            "message", message,
            "featureEnabled", featureEnabled
        );
    }
}
```

### Dynamic Configuration Refresh

**باستخدام Actuator:**
```bash
# إعادة تحميل Configuration
curl -X POST http://localhost:8081/actuator/refresh
```

**باستخدام Spring Cloud Bus:**
```bash
# إعادة تحميل جميع Services مرة واحدة
curl -X POST http://localhost:8888/actuator/bus-refresh
```

---

## Load Balancing (Spring Cloud LoadBalancer)

### Client-Side Load Balancing

**1. Dependencies:**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

**2. استخدام مع RestTemplate:**
```java
@Configuration
public class RestTemplateConfig {

    @Bean
    @LoadBalanced  // تفعيل Load Balancing
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

@Service
public class UserService {

    @Autowired
    @LoadBalanced
    private RestTemplate restTemplate;

    public User getUserById(String id) {
        // استخدام اسم Service من Eureka
        return restTemplate.getForObject(
            "http://USER-SERVICE/users/" + id,
            User.class
        );
    }
}
```

**3. استخدام مع WebClient:**
```java
@Configuration
public class WebClientConfig {

    @Bean
    @LoadBalanced
    public WebClient.Builder webClientBuilder() {
        return WebClient.builder();
    }
}

@Service
public class UserService {

    private final WebClient webClient;

    public UserService(@LoadBalanced WebClient.Builder builder) {
        this.webClient = builder.build();
    }

    public Mono<User> getUserById(String id) {
        return webClient.get()
            .uri("http://USER-SERVICE/users/{id}", id)
            .retrieve()
            .bodyToMono(User.class);
    }
}
```

**4. Custom Load Balancer Configuration:**
```java
@Configuration
public class LoadBalancerConfig {

    @Bean
    public ServiceInstanceListSupplier discoveryClientServiceInstanceListSupplier(
            ConfigurableApplicationContext context) {
        return ServiceInstanceListSupplier.builder()
            .withDiscoveryClient()
            .withHealthChecks()  // فحص الصحة
            .build(context);
    }

    // Custom Load Balancing Strategy
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
}
```

---

## Circuit Breaker (Resilience4j)

### ما هو Circuit Breaker؟

**Circuit Breaker** يمنع التعطل المتسلسل (Cascading Failure) عندما يفشل Service:

**الحالات الثلاثة:**
1. **CLOSED**: الطلبات تمر بشكل طبيعي
2. **OPEN**: الطلبات تُرفض فوراً (Fail Fast)
3. **HALF_OPEN**: يسمح بعدد محدود من الطلبات للاختبار

### Dependencies

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>
</dependencies>
```

### Configuration

```yaml
resilience4j:
  circuitbreaker:
    instances:
      userService:
        register-health-indicator: true
        sliding-window-size: 10              # عدد الطلبات للتقييم
        minimum-number-of-calls: 5           # الحد الأدنى قبل التقييم
        permitted-number-of-calls-in-half-open-state: 3
        wait-duration-in-open-state: 10s     # الوقت في OPEN
        failure-rate-threshold: 50           # نسبة الفشل لفتح Circuit
        slow-call-rate-threshold: 50         # نسبة الطلبات البطيئة
        slow-call-duration-threshold: 2s     # ما يعتبر بطيء

  retry:
    instances:
      userService:
        max-attempts: 3
        wait-duration: 1s
        retry-exceptions:
          - java.net.ConnectException
          - java.io.IOException

  ratelimiter:
    instances:
      userService:
        limit-for-period: 10
        limit-refresh-period: 1s
        timeout-duration: 0

  timelimiter:
    instances:
      userService:
        timeout-duration: 3s
```

### استخدام @CircuitBreaker

```java
@Service
public class UserService {

    @Autowired
    private RestTemplate restTemplate;

    @CircuitBreaker(name = "userService", fallbackMethod = "getUserFallback")
    @Retry(name = "userService")
    @RateLimiter(name = "userService")
    @TimeLimiter(name = "userService")
    public User getUserById(String id) {
        return restTemplate.getForObject(
            "http://USER-SERVICE/users/" + id,
            User.class
        );
    }

    // Fallback Method
    private User getUserFallback(String id, Exception ex) {
        log.error("Fallback triggered for user {}: {}", id, ex.getMessage());

        User fallbackUser = new User();
        fallbackUser.setId(id);
        fallbackUser.setName("Fallback User");
        fallbackUser.setEmail("fallback@example.com");

        return fallbackUser;
    }
}
```

### استخدام Programmatic API

```java
@Service
public class UserService {

    @Autowired
    private RestTemplate restTemplate;

    private final CircuitBreaker circuitBreaker;
    private final Retry retry;

    public UserService(CircuitBreakerFactory circuitBreakerFactory,
                      RetryRegistry retryRegistry) {
        this.circuitBreaker = circuitBreakerFactory.create("userService");
        this.retry = retryRegistry.retry("userService");
    }

    public User getUserById(String id) {
        Supplier<User> supplier = () -> restTemplate.getForObject(
            "http://USER-SERVICE/users/" + id,
            User.class
        );

        // مع Circuit Breaker و Retry
        supplier = CircuitBreaker.decorateSupplier(circuitBreaker, supplier);
        supplier = Retry.decorateSupplier(retry, supplier);

        try {
            return supplier.get();
        } catch (Exception ex) {
            return getUserFallback(id, ex);
        }
    }

    private User getUserFallback(String id, Exception ex) {
        // نفس الـ Fallback
    }
}
```

### Monitoring Circuit Breaker

```java
@RestController
public class CircuitBreakerController {

    @Autowired
    private CircuitBreakerRegistry circuitBreakerRegistry;

    @GetMapping("/circuit-breaker/status")
    public Map<String, Object> getCircuitBreakerStatus() {
        CircuitBreaker circuitBreaker = circuitBreakerRegistry
            .circuitBreaker("userService");

        return Map.of(
            "state", circuitBreaker.getState().name(),
            "metrics", circuitBreaker.getMetrics(),
            "config", circuitBreaker.getCircuitBreakerConfig()
        );
    }

    // Event Listeners
    @PostConstruct
    public void registerEventListeners() {
        CircuitBreaker circuitBreaker = circuitBreakerRegistry
            .circuitBreaker("userService");

        circuitBreaker.getEventPublisher()
            .onStateTransition(event -> {
                log.info("Circuit Breaker State Changed: {} -> {}",
                    event.getStateTransition().getFromState(),
                    event.getStateTransition().getToState());
            })
            .onError(event -> {
                log.error("Circuit Breaker Error: {}",
                    event.getThrowable().getMessage());
            });
    }
}
```

---

## Distributed Tracing (Sleuth & Zipkin)

### Spring Cloud Sleuth

**Sleuth** يضيف Trace IDs و Span IDs للطلبات.

**1. Dependencies:**
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-sleuth</artifactId>
    </dependency>

    <!-- Zipkin Integration -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-sleuth-zipkin</artifactId>
    </dependency>
</dependencies>
```

**2. Configuration:**
```yaml
spring:
  sleuth:
    sampler:
      probability: 1.0  # نسبة الطلبات للتتبع (1.0 = 100%)
  zipkin:
    base-url: http://localhost:9411
    sender:
      type: web  # أو rabbit أو kafka

logging:
  pattern:
    level: "%5p [${spring.application.name:},%X{traceId:-},%X{spanId:-}]"
```

**3. Logs مع Trace IDs:**
```
INFO [user-service,a1b2c3d4e5f6,1234567890] Getting user by ID: 123
INFO [order-service,a1b2c3d4e5f6,9876543210] Creating order for user: 123
```

### Zipkin Server

**Docker:**
```bash
docker run -d -p 9411:9411 openzipkin/zipkin
```

**Dashboard:**
```
http://localhost:9411
```

### Custom Spans

```java
@Service
public class UserService {

    @Autowired
    private Tracer tracer;

    public User getUserById(String id) {
        Span newSpan = tracer.nextSpan().name("getUserById").start();

        try (Tracer.SpanInScope ws = tracer.withSpan(newSpan)) {
            newSpan.tag("userId", id);
            newSpan.annotate("Starting database query");

            User user = userRepository.findById(id);

            newSpan.annotate("Database query completed");
            return user;
        } catch (Exception ex) {
            newSpan.error(ex);
            throw ex;
        } finally {
            newSpan.end();
        }
    }
}
```

---

## Message Bus (Spring Cloud Bus)

### Spring Cloud Bus

**Cloud Bus** يربط جميع Services عبر Message Broker للتواصل.

**1. Dependencies:**
```xml
<dependencies>
    <!-- RabbitMQ -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-bus-amqp</artifactId>
    </dependency>

    <!-- أو Kafka -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-bus-kafka</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

**2. Configuration:**
```yaml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest

  # أو Kafka
  kafka:
    bootstrap-servers: localhost:9092

management:
  endpoints:
    web:
      exposure:
        include: bus-refresh,bus-env
```

**3. Refresh جميع Services:**
```bash
curl -X POST http://localhost:8888/actuator/bus-refresh
```

### Custom Bus Events

```java
// Custom Event
public class UserCreatedEvent extends RemoteApplicationEvent {
    private String userId;
    private String userName;

    public UserCreatedEvent() {}

    public UserCreatedEvent(Object source, String originService,
                           String userId, String userName) {
        super(source, originService);
        this.userId = userId;
        this.userName = userName;
    }

    // getters, setters
}

// Publisher
@Service
public class UserService {

    @Autowired
    private ApplicationEventPublisher publisher;

    public User createUser(User user) {
        User savedUser = userRepository.save(user);

        // نشر Event
        publisher.publishEvent(new UserCreatedEvent(
            this,
            "user-service",
            savedUser.getId(),
            savedUser.getName()
        ));

        return savedUser;
    }
}

// Listener في Service آخر
@Component
public class UserEventListener {

    @EventListener
    public void handleUserCreated(UserCreatedEvent event) {
        log.info("User created: {} - {}", event.getUserId(), event.getUserName());
        // معالجة Event
    }
}
```

---

## Security (OAuth2)

### Authorization Server

**1. Dependencies:**
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-oauth2-authorization-server</artifactId>
    </dependency>
</dependencies>
```

**2. Configuration:**
```java
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig {

    @Bean
    public SecurityFilterChain authorizationServerSecurityFilterChain(HttpSecurity http)
            throws Exception {
        OAuth2AuthorizationServerConfiguration.applyDefaultSecurity(http);

        http.formLogin(Customizer.withDefaults());

        return http.build();
    }

    @Bean
    public RegisteredClientRepository registeredClientRepository() {
        RegisteredClient registeredClient = RegisteredClient.withId(UUID.randomUUID().toString())
            .clientId("client-id")
            .clientSecret("{noop}client-secret")
            .clientAuthenticationMethod(ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
            .authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
            .authorizationGrantType(AuthorizationGrantType.REFRESH_TOKEN)
            .authorizationGrantType(AuthorizationGrantType.CLIENT_CREDENTIALS)
            .redirectUri("http://localhost:8080/login/oauth2/code/myapp")
            .scope("read")
            .scope("write")
            .build();

        return new InMemoryRegisteredClientRepository(registeredClient);
    }

    @Bean
    public JWKSource<SecurityContext> jwkSource() {
        KeyPair keyPair = generateRsaKey();
        RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();
        RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();
        RSAKey rsaKey = new RSAKey.Builder(publicKey)
            .privateKey(privateKey)
            .keyID(UUID.randomUUID().toString())
            .build();
        JWKSet jwkSet = new JWKSet(rsaKey);
        return new ImmutableJWKSet<>(jwkSet);
    }

    private static KeyPair generateRsaKey() {
        KeyPair keyPair;
        try {
            KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("RSA");
            keyPairGenerator.initialize(2048);
            keyPair = keyPairGenerator.generateKeyPair();
        } catch (Exception ex) {
            throw new IllegalStateException(ex);
        }
        return keyPair;
    }
}
```

### Resource Server

**1. Dependencies:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

**2. Configuration:**
```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:9000
          jwk-set-uri: http://localhost:9000/oauth2/jwks
```

**3. Security Configuration:**
```java
@Configuration
@EnableWebSecurity
public class ResourceServerConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authorize -> authorize
                .requestMatchers("/public/**").permitAll()
                .requestMatchers("/api/users/**").hasAuthority("SCOPE_read")
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(Customizer.withDefaults())
            );

        return http.build();
    }

    @Bean
    public JwtAuthenticationConverter jwtAuthenticationConverter() {
        JwtGrantedAuthoritiesConverter grantedAuthoritiesConverter =
            new JwtGrantedAuthoritiesConverter();
        grantedAuthoritiesConverter.setAuthoritiesClaimName("authorities");
        grantedAuthoritiesConverter.setAuthorityPrefix("ROLE_");

        JwtAuthenticationConverter jwtAuthenticationConverter =
            new JwtAuthenticationConverter();
        jwtAuthenticationConverter.setJwtGrantedAuthoritiesConverter(
            grantedAuthoritiesConverter);

        return jwtAuthenticationConverter;
    }
}
```

**4. استخدام:**
```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping
    @PreAuthorize("hasAuthority('SCOPE_read')")
    public List<User> getAllUsers() {
        return userService.findAll();
    }

    @PostMapping
    @PreAuthorize("hasRole('ADMIN')")
    public User createUser(@RequestBody User user) {
        return userService.save(user);
    }

    @GetMapping("/me")
    public Map<String, Object> getCurrentUser(@AuthenticationPrincipal Jwt jwt) {
        return Map.of(
            "sub", jwt.getSubject(),
            "scopes", jwt.getClaimAsStringList("scope"),
            "claims", jwt.getClaims()
        );
    }
}
```

---

## أمثلة عملية

### مثال 1: E-commerce Microservices

**Architecture:**
```
                    ┌─────────────────┐
                    │  API Gateway    │
                    └────────┬────────┘
                             │
            ┌────────────────┼────────────────┐
            │                │                │
     ┌──────▼─────┐   ┌─────▼──────┐  ┌─────▼──────┐
     │   User     │   │  Product   │  │   Order    │
     │  Service   │   │  Service   │  │  Service   │
     └────────────┘   └────────────┘  └────────────┘
            │                │                │
     ┌──────▼─────┐   ┌─────▼──────┐  ┌─────▼──────┐
     │  User DB   │   │ Product DB │  │  Order DB  │
     └────────────┘   └────────────┘  └────────────┘

            ┌────────────────┐
            │ Eureka Server  │
            └────────────────┘

            ┌────────────────┐
            │ Config Server  │
            └────────────────┘
```

**User Service:**
```java
@RestController
@RequestMapping("/users")
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping("/{id}")
    public ResponseEntity<User> getUserById(@PathVariable String id) {
        return userService.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    @PostMapping
    public ResponseEntity<User> createUser(@Valid @RequestBody User user) {
        User savedUser = userService.save(user);
        return ResponseEntity.created(
            URI.create("/users/" + savedUser.getId())
        ).body(savedUser);
    }
}
```

**Order Service (يستدعي User Service و Product Service):**
```java
@Service
public class OrderService {

    @Autowired
    @LoadBalanced
    private RestTemplate restTemplate;

    @Autowired
    private OrderRepository orderRepository;

    @CircuitBreaker(name = "orderService", fallbackMethod = "createOrderFallback")
    public Order createOrder(OrderRequest request) {
        // التحقق من User
        User user = restTemplate.getForObject(
            "http://USER-SERVICE/users/" + request.getUserId(),
            User.class
        );

        if (user == null) {
            throw new UserNotFoundException("User not found");
        }

        // التحقق من Product
        Product product = restTemplate.getForObject(
            "http://PRODUCT-SERVICE/products/" + request.getProductId(),
            Product.class
        );

        if (product == null) {
            throw new ProductNotFoundException("Product not found");
        }

        // إنشاء Order
        Order order = new Order();
        order.setUserId(request.getUserId());
        order.setProductId(request.getProductId());
        order.setQuantity(request.getQuantity());
        order.setTotalPrice(product.getPrice() * request.getQuantity());
        order.setStatus(OrderStatus.PENDING);

        return orderRepository.save(order);
    }

    private Order createOrderFallback(OrderRequest request, Exception ex) {
        log.error("Fallback triggered: {}", ex.getMessage());

        Order fallbackOrder = new Order();
        fallbackOrder.setStatus(OrderStatus.FAILED);
        fallbackOrder.setErrorMessage("Service temporarily unavailable");

        return fallbackOrder;
    }
}
```

### مثال 2: OpenFeign للتواصل بين Services

**Dependencies:**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

**Enable Feign:**
```java
@SpringBootApplication
@EnableFeignClients
@EnableDiscoveryClient
public class OrderServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}
```

**Feign Client:**
```java
@FeignClient(
    name = "USER-SERVICE",
    fallback = UserServiceFallback.class
)
public interface UserServiceClient {

    @GetMapping("/users/{id}")
    User getUserById(@PathVariable String id);

    @PostMapping("/users")
    User createUser(@RequestBody User user);

    @GetMapping("/users/search")
    List<User> searchUsers(@RequestParam String name);
}

// Fallback Implementation
@Component
public class UserServiceFallback implements UserServiceClient {

    @Override
    public User getUserById(String id) {
        User fallbackUser = new User();
        fallbackUser.setId(id);
        fallbackUser.setName("Fallback User");
        return fallbackUser;
    }

    @Override
    public User createUser(User user) {
        throw new ServiceUnavailableException("User service is unavailable");
    }

    @Override
    public List<User> searchUsers(String name) {
        return Collections.emptyList();
    }
}

// استخدام
@Service
public class OrderService {

    @Autowired
    private UserServiceClient userServiceClient;

    public Order createOrder(OrderRequest request) {
        User user = userServiceClient.getUserById(request.getUserId());

        // معالجة Order
    }
}
```

**Feign Configuration:**
```yaml
feign:
  client:
    config:
      default:
        connectTimeout: 5000
        readTimeout: 5000
        loggerLevel: full
  circuitbreaker:
    enabled: true
  compression:
    request:
      enabled: true
    response:
      enabled: true
```

---

## Best Practices

### 1. Service Design

- **Single Responsibility**: كل Service له مسؤولية واحدة
- **Loose Coupling**: Services مستقلة قدر الإمكان
- **Database per Service**: كل Service له Database خاص
- **API Gateway Pattern**: نقطة دخول موحدة
- **Event-Driven**: استخدام Events للتواصل

### 2. Configuration

- **Externalized Configuration**: استخدام Config Server
- **Environment-Specific**: تهيئة منفصلة لكل بيئة
- **Secrets Management**: استخدام Vault أو Secrets Manager
- **Feature Flags**: للتحكم في الميزات

### 3. Resilience

- **Circuit Breaker**: لكل استدعاء خارجي
- **Retry with Backoff**: إعادة محاولة ذكية
- **Timeouts**: تحديد وقت للعمليات
- **Fallbacks**: خطط بديلة عند الفشل

### 4. Monitoring & Observability

- **Distributed Tracing**: Sleuth + Zipkin
- **Centralized Logging**: ELK Stack أو Splunk
- **Metrics**: Prometheus + Grafana
- **Health Checks**: Actuator endpoints

### 5. Security

- **OAuth2/JWT**: للـ Authentication & Authorization
- **API Gateway Security**: تأمين على Gateway
- **Service-to-Service**: Mutual TLS
- **Secrets**: لا تضع في Code أو Config files

### 6. Testing

- **Unit Tests**: لكل Service منفصل
- **Integration Tests**: للتواصل بين Services
- **Contract Testing**: Pact أو Spring Cloud Contract
- **End-to-End Tests**: للسيناريوهات الكاملة

---

## أسئلة المقابلات

### أسئلة أساسية

**1. ما هو Spring Cloud؟**
- **الإجابة**: Spring Cloud هو مجموعة أدوات لبناء Distributed Systems و Microservices، توفر:
  - Service Discovery، API Gateway، Configuration Management
  - Load Balancing، Circuit Breaker، Distributed Tracing

**2. ما الفرق بين Eureka Server و Eureka Client؟**
- **الإجابة**:
  - **Eureka Server**: Service Registry، يسجل ويخزن معلومات Services
  - **Eureka Client**: Service يسجل نفسه في Eureka Server

**3. ما هو API Gateway ولماذا نستخدمه؟**
- **الإجابة**: API Gateway هو نقطة دخول موحدة توفر:
  - Routing، Load Balancing، Authentication
  - Rate Limiting، Logging، Transformation

**4. ما الفرق بين Spring Cloud Gateway و Zuul؟**
- **الإجابة**:
  - **Gateway**: Reactive (WebFlux)، حديث، يُنصح به
  - **Zuul**: Blocking، قديم، Deprecated

**5. ما هو Circuit Breaker ولماذا نستخدمه؟**
- **الإجابة**: Circuit Breaker يمنع Cascading Failure:
  - يفتح عند فشل متكرر
  - يرفض الطلبات فوراً (Fail Fast)
  - يختبر Service بعد فترة

### أسئلة متقدمة

**6. كيف يعمل Service Discovery؟**
- **الإجابة**:
  1. Service يسجل نفسه في Eureka عند البدء
  2. يرسل Heartbeats دورية
  3. Services أخرى تجلب Registry
  4. تستدعي بعضها باستخدام Service Name

**7. ما الفرق بين Client-Side و Server-Side Load Balancing؟**
- **الإجابة**:
  - **Client-Side**: Client يختار Server (Ribbon، LoadBalancer)
  - **Server-Side**: Load Balancer خارجي يوزع (NGINX، HAProxy)

**8. كيف تنفذ Dynamic Configuration Refresh؟**
- **الإجابة**:
  - استخدام `@RefreshScope`
  - Actuator endpoint `/refresh`
  - Spring Cloud Bus للـ Broadcast refresh

**9. ما الفرق بين Hystrix و Resilience4j؟**
- **الإجابة**:
  - **Hystrix**: قديم، Maintenance Mode
  - **Resilience4j**: حديث، lightweight، functional

**10. كيف تتعامل مع Distributed Transactions؟**
- **الإجابة**:
  - **Saga Pattern**: سلسلة من Transactions محلية
  - **Event Sourcing**: تخزين Events بدلاً من State
  - **2-Phase Commit**: نادراً في Microservices

**11. ما هو Distributed Tracing؟**
- **الإجابة**: تتبع Request عبر Services متعددة:
  - Trace ID: فريد للـ Request
  - Span ID: فريد لكل Service
  - Sleuth يضيف IDs، Zipkin يعرضها

**12. كيف تؤمن Microservices؟**
- **الإجابة**:
  - OAuth2/JWT على API Gateway
  - Service-to-Service: Mutual TLS
  - Secret Management: Vault
  - Network Segmentation

**13. ما هو Feign Client؟**
- **الإجابة**: Declarative REST Client:
  - Interface-based
  - يتكامل مع Eureka و LoadBalancer
  - دعم Circuit Breaker

**14. كيف تنفذ API Versioning في Microservices؟**
- **الإجابة**:
  - URI Versioning: `/api/v1/users`
  - Header Versioning: `Accept-Version: v1`
  - Gateway Routing بناءً على Version

**15. ما هي Saga Pattern؟**
- **الإجابة**: نمط لإدارة Distributed Transactions:
  - **Choreography**: Services تتواصل بـ Events
  - **Orchestration**: Orchestrator يدير الـ Flow

### أسئلة عن Best Practices

**16. ما هي Challenges في Microservices؟**
- **الإجابة**:
  - Distributed Data Management
  - Network Latency
  - Service Discovery
  - Monitoring & Debugging
  - Testing Complexity

**17. كيف تتعامل مع Data Consistency؟**
- **الإجابة**:
  - Eventual Consistency
  - Saga Pattern
  - Event Sourcing
  - CQRS

**18. ما هي استراتيجيات Deployment للـ Microservices؟**
- **الإجابة**:
  - Blue-Green Deployment
  - Canary Deployment
  - Rolling Deployment
  - Feature Toggles

**19. كيف تختبر Microservices؟**
- **الإجابة**:
  - Unit Tests لكل Service
  - Contract Testing (Pact)
  - Integration Testing
  - End-to-End Testing
  - Chaos Engineering

**20. متى يجب استخدام Microservices؟**
- **الإجابة**: استخدم Microservices عندما:
  - تطبيق كبير ومعقد
  - فرق متعددة
  - حاجة للـ Scalability المستقلة
  - تقنيات مختلفة لكل Service
  - **لا تستخدم** للتطبيقات الصغيرة

---

## الخلاصة

Spring Cloud هو حل شامل لبناء Microservices:
- ✅ Service Discovery (Eureka)
- ✅ API Gateway (Spring Cloud Gateway)
- ✅ Configuration Management (Config Server)
- ✅ Load Balancing (LoadBalancer)
- ✅ Resilience (Resilience4j)
- ✅ Distributed Tracing (Sleuth & Zipkin)
- ✅ Security (OAuth2 & JWT)

**متى تستخدم Spring Cloud:**
- بناء Microservices Architecture
- تطبيقات موزعة كبيرة
- حاجة لـ High Availability
- فرق متعددة تعمل بشكل مستقل

**Best Practices:**
- تصميم Services بشكل صحيح (Single Responsibility)
- استخدام Circuit Breakers
- Centralized Configuration
- Comprehensive Monitoring
- Automated Testing
- Security First

**الأدوات المكملة:**
- **Kubernetes**: Container Orchestration
- **Docker**: Containerization
- **ELK Stack**: Centralized Logging
- **Prometheus + Grafana**: Monitoring
- **Jenkins**: CI/CD