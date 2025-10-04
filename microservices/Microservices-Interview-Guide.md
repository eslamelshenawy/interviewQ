# 🚀 دليل أسئلة المقابلات - Microservices مع Java & Spring Boot

## 📚 الفهرس
1. [أسئلة أساسية](#أسئلة-أساسية)
2. [Service Discovery & Registration](#service-discovery--registration)
3. [API Gateway](#api-gateway)
4. [Communication بين Services](#communication-بين-services)
5. [Configuration Management](#configuration-management)
6. [Load Balancing & Resilience](#load-balancing--resilience)
7. [Distributed Tracing & Monitoring](#distributed-tracing--monitoring)
8. [Security](#security)
9. [Database & Transactions](#database--transactions)
10. [Deployment & Containerization](#deployment--containerization)
11. [أسئلة متقدمة](#أسئلة-متقدمة)
12. [أسئلة عملية (Scenario-Based)](#أسئلة-عملية-scenario-based)

---

## أسئلة أساسية

### 1. ما هي الـ Microservices وما الفرق بينها وبين Monolithic Architecture؟

**الشرح:**

**Microservices:**
- معمارية برمجية تقسم التطبيق إلى خدمات صغيرة مستقلة
- كل خدمة تعمل في process منفصل
- كل خدمة مسؤولة عن وظيفة محددة (Single Responsibility)
- التواصل بين الخدمات عبر APIs (REST, gRPC, Messaging)

**Monolithic Architecture:**
- كل التطبيق في codebase واحد
- يعمل كـ process واحد
- جميع المكونات مرتبطة ببعضها

**الفرق:**

| المقارنة | Monolithic | Microservices |
|---------|-----------|---------------|
| الحجم | تطبيق واحد كبير | عدة تطبيقات صغيرة |
| Deployment | نشر التطبيق كله | نشر كل خدمة منفصلة |
| Scalability | تكبير التطبيق كله | تكبير الخدمات المطلوبة فقط |
| Technology | تقنية واحدة غالباً | تقنيات متعددة ممكنة |
| Database | قاعدة بيانات واحدة | قاعدة بيانات لكل خدمة |
| Development | فريق واحد كبير | فرق صغيرة لكل خدمة |

**مثال:**
```
Monolithic: Order + Payment + User + Inventory = تطبيق واحد
Microservices: Order Service | Payment Service | User Service | Inventory Service
```

---

### 2. ما هي مميزات وعيوب Microservices؟

**المميزات:**

1. **Independent Deployment**
   - يمكن نشر كل خدمة بشكل مستقل
   - لا تحتاج لإعادة نشر التطبيق كله

2. **Scalability**
   - تكبير الخدمات التي تحتاج موارد أكثر فقط
   - توفير في التكلفة

3. **Technology Flexibility**
   - كل خدمة يمكن أن تستخدم تقنية مختلفة
   - Java لخدمة، Node.js لخدمة أخرى

4. **Fault Isolation**
   - فشل خدمة لا يعطل التطبيق كله
   - أسهل في تحديد المشاكل

5. **Team Independence**
   - فرق صغيرة تعمل بشكل مستقل
   - أسرع في التطوير

6. **Easy Maintenance**
   - codebase أصغر لكل خدمة
   - أسهل في الفهم والتعديل

**العيوب:**

1. **Complexity**
   - صعوبة في إدارة عدة خدمات
   - حاجة لأدوات إضافية

2. **Network Latency**
   - التواصل بين الخدمات عبر الشبكة
   - أبطأ من الـ Method Calls

3. **Data Consistency**
   - صعوبة في الحفاظ على Consistency
   - Distributed Transactions معقدة

4. **Testing**
   - صعوبة في الاختبار الشامل
   - حاجة لـ Integration Testing

5. **Deployment Complexity**
   - حاجة لـ DevOps قوية
   - أدوات Orchestration (Kubernetes)

6. **Monitoring & Debugging**
   - صعوبة في تتبع الأخطاء
   - حاجة لـ Distributed Tracing

---

### 3. ما هو Spring Boot ولماذا نستخدمه في Microservices؟

**الشرح:**

**Spring Boot:**
- Framework مبني على Spring Framework
- يسهل إنشاء تطبيقات Spring بسرعة
- يوفر Configuration تلقائية

**لماذا نستخدمه في Microservices؟**

1. **Auto-Configuration**
```java
@SpringBootApplication
public class OrderServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}
```

2. **Embedded Server**
   - Tomcat, Jetty مدمج
   - لا حاجة لـ External Server
   - JAR file فقط وشغل

3. **Production-Ready Features**
   - Health Checks
   - Metrics
   - Monitoring

4. **Easy Integration**
   - Spring Cloud integration سهل
   - Database, Security, etc.

5. **Fast Development**
   - Spring Initializr لبدء المشروع
   - Dependencies management

**مثال بسيط:**
```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    @GetMapping("/{id}")
    public Order getOrder(@PathVariable Long id) {
        return orderService.findById(id);
    }

    @PostMapping
    public Order createOrder(@RequestBody Order order) {
        return orderService.save(order);
    }
}
```

**application.yml:**
```yaml
spring:
  application:
    name: order-service
server:
  port: 8081
```

---

### 4. اشرح Spring Cloud وأهم مكوناته

**الشرح:**

**Spring Cloud:**
- مجموعة من الأدوات لبناء Microservices
- مبني على Spring Boot
- يحل مشاكل Distributed Systems

**أهم المكونات:**

#### 1. **Spring Cloud Netflix Eureka (Service Discovery)**
```java
// Eureka Server
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}

// Eureka Client
@EnableEurekaClient
@SpringBootApplication
public class OrderServiceApplication {
    // ...
}
```

#### 2. **Spring Cloud Gateway (API Gateway)**
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: order-service
          uri: lb://ORDER-SERVICE
          predicates:
            - Path=/api/orders/**
```

#### 3. **Spring Cloud Config (Configuration Management)**
```java
@EnableConfigServer
@SpringBootApplication
public class ConfigServerApplication {
    // ...
}
```

#### 4. **Spring Cloud OpenFeign (HTTP Client)**
```java
@FeignClient(name = "payment-service")
public interface PaymentClient {
    @GetMapping("/api/payments/{id}")
    Payment getPayment(@PathVariable Long id);
}
```

#### 5. **Spring Cloud Circuit Breaker (Resilience)**
```java
@CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
public Payment getPayment(Long id) {
    return paymentClient.getPayment(id);
}

public Payment paymentFallback(Long id, Exception ex) {
    return new Payment(); // default response
}
```

#### 6. **Spring Cloud Sleuth (Distributed Tracing)**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
```

#### 7. **Spring Cloud LoadBalancer**
```java
@LoadBalanced
@Bean
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

#### 8. **Spring Cloud Stream (Messaging)**
```java
@EnableBinding(Source.class)
public class MessageProducer {
    @Autowired
    private Source source;

    public void sendMessage(String message) {
        source.output().send(MessageBuilder.withPayload(message).build());
    }
}
```

**الصورة الكاملة:**
```
┌─────────────────────────────────────────────────┐
│          Spring Cloud Ecosystem                  │
├─────────────────────────────────────────────────┤
│ Config Server  → Centralized Configuration      │
│ Eureka         → Service Discovery               │
│ Gateway        → API Gateway & Routing           │
│ Feign          → Declarative HTTP Client         │
│ Resilience4j   → Circuit Breaker & Fault Tolerance│
│ Sleuth/Zipkin  → Distributed Tracing             │
│ LoadBalancer   → Client-Side Load Balancing      │
│ Stream         → Event-Driven Communication      │
└─────────────────────────────────────────────────┘
```

---

## Service Discovery & Registration

### 5. ما هو Service Discovery وكيف يعمل؟

**الشرح:**

**Service Discovery:**
- آلية للعثور على عناوين الخدمات في النظام
- الخدمات تسجل نفسها تلقائياً
- الخدمات الأخرى تكتشفها ديناميكياً

**المشكلة التي يحلها:**
```
بدون Service Discovery:
OrderService → http://192.168.1.10:8081/payments  ❌
- لو IP تغير؟
- لو Port تغير؟
- لو عدة instances؟

مع Service Discovery:
OrderService → PAYMENT-SERVICE ✅
- اكتشاف تلقائي
- تحديث تلقائي
- Load balancing تلقائي
```

**كيف يعمل:**

```
1. Service Registration:
   Payment Service → يسجل نفسه في Eureka Server
   (Name: PAYMENT-SERVICE, IP: 192.168.1.10, Port: 8081)

2. Service Discovery:
   Order Service → يسأل Eureka عن PAYMENT-SERVICE
   Eureka → يرجع القائمة (IP + Port)

3. Health Check:
   Eureka → يفحص الخدمات كل فترة
   إذا Service مات → يحذفه من القائمة
```

**Implementation:**

**Eureka Server:**
```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

```yaml
# application.yml
server:
  port: 8761

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
  server:
    enable-self-preservation: false
```

**Eureka Client (Payment Service):**
```java
@EnableEurekaClient
@SpringBootApplication
public class PaymentServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(PaymentServiceApplication.class, args);
    }
}
```

```yaml
spring:
  application:
    name: payment-service

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
```

**استخدام Discovery (Order Service):**
```java
@Service
public class OrderService {

    @Autowired
    private DiscoveryClient discoveryClient;

    public List<ServiceInstance> getPaymentServiceInstances() {
        return discoveryClient.getInstances("PAYMENT-SERVICE");
    }
}
```

**أو باستخدام Feign:**
```java
@FeignClient(name = "payment-service")
public interface PaymentClient {
    @GetMapping("/api/payments/{id}")
    Payment getPayment(@PathVariable Long id);
}
```

---

### 6. اشرح Eureka Server وكيفية استخدامه

**الشرح:**

**Eureka Server:**
- Service Registry من Netflix
- يحتفظ بمعلومات جميع الخدمات
- يوفر Dashboard لمراقبة الخدمات

**مكونات Eureka:**

1. **Eureka Server** - Registry
2. **Eureka Client** - الخدمات التي تسجل نفسها

**كيف يعمل:**

```
┌─────────────────────────────────────────┐
│         Eureka Server (Port 8761)       │
│  ┌───────────────────────────────────┐  │
│  │ Service Registry                  │  │
│  │ - PAYMENT-SERVICE: [Instance1]    │  │
│  │ - ORDER-SERVICE: [Instance1, 2]   │  │
│  │ - USER-SERVICE: [Instance1]       │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
         ↑                    ↑
    Registration          Discovery
         │                    │
    ┌────┴────┐         ┌────┴────┐
    │ Payment │         │  Order  │
    │ Service │         │ Service │
    └─────────┘         └─────────┘
```

**Setup كامل:**

#### 1. Dependencies (pom.xml)

**Eureka Server:**
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>2023.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

**Eureka Client:**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

#### 2. Eureka Server Configuration

**Main Class:**
```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

**application.yml:**
```yaml
server:
  port: 8761

spring:
  application:
    name: eureka-server

eureka:
  client:
    # لا تسجل Server نفسه كـ Client
    register-with-eureka: false
    fetch-registry: false
  server:
    # تعطيل Self Preservation للتطوير
    enable-self-preservation: false
    # وقت حذف Service الميت
    eviction-interval-timer-in-ms: 3000
```

#### 3. Eureka Client Configuration

**Payment Service:**
```java
@SpringBootApplication
@EnableEurekaClient // أو @EnableDiscoveryClient
public class PaymentServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(PaymentServiceApplication.class, args);
    }
}
```

**application.yml:**
```yaml
server:
  port: 8081

spring:
  application:
    name: payment-service

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
    # تحديث القائمة من Eureka كل فترة
    registry-fetch-interval-seconds: 5
  instance:
    # استخدام IP بدل Hostname
    prefer-ip-address: true
    # Heartbeat interval
    lease-renewal-interval-in-seconds: 5
    # وقت اعتبار Service ميت
    lease-expiration-duration-in-seconds: 10
    # معلومات إضافية
    metadata-map:
      zone: zone1
      version: 1.0
```

#### 4. استخدام Service Discovery

**طريقة 1: Discovery Client**
```java
@Service
public class OrderService {

    @Autowired
    private DiscoveryClient discoveryClient;

    @Autowired
    private RestTemplate restTemplate;

    public Payment getPayment(Long paymentId) {
        // الحصول على instances
        List<ServiceInstance> instances =
            discoveryClient.getInstances("PAYMENT-SERVICE");

        if (instances.isEmpty()) {
            throw new ServiceUnavailableException("Payment Service not available");
        }

        // اختيار أول instance
        ServiceInstance instance = instances.get(0);
        String url = instance.getUri() + "/api/payments/" + paymentId;

        return restTemplate.getForObject(url, Payment.class);
    }
}
```

**طريقة 2: Load Balanced RestTemplate**
```java
@Configuration
public class RestTemplateConfig {

    @LoadBalanced // يضيف Load Balancing تلقائياً
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

@Service
public class OrderService {

    @Autowired
    private RestTemplate restTemplate;

    public Payment getPayment(Long paymentId) {
        // استخدام اسم الخدمة مباشرة
        String url = "http://PAYMENT-SERVICE/api/payments/" + paymentId;
        return restTemplate.getForObject(url, Payment.class);
    }
}
```

**طريقة 3: Feign Client (الأفضل)**
```java
@FeignClient(name = "payment-service")
public interface PaymentClient {

    @GetMapping("/api/payments/{id}")
    Payment getPayment(@PathVariable("id") Long id);

    @PostMapping("/api/payments")
    Payment createPayment(@RequestBody Payment payment);
}

@Service
public class OrderService {

    @Autowired
    private PaymentClient paymentClient;

    public Payment getPayment(Long paymentId) {
        return paymentClient.getPayment(paymentId);
    }
}
```

#### 5. Health Check & Monitoring

```java
@RestController
public class HealthController {

    @Autowired
    private DiscoveryClient discoveryClient;

    @GetMapping("/services")
    public List<String> getAllServices() {
        return discoveryClient.getServices();
    }

    @GetMapping("/services/{serviceName}")
    public List<ServiceInstance> getServiceInstances(@PathVariable String serviceName) {
        return discoveryClient.getInstances(serviceName);
    }
}
```

**Eureka Dashboard:**
- افتح المتصفح: `http://localhost:8761`
- تشاهد جميع الخدمات المسجلة
- حالة كل خدمة (UP/DOWN)
- عدد الـ Instances

**مميزات Eureka:**

1. **Self-Registration**: الخدمات تسجل نفسها
2. **Health Checks**: فحص دوري للخدمات
3. **Load Balancing**: توزيع الطلبات
4. **Fault Tolerance**: لو Service مات، يحذفه
5. **Multi-Zone Support**: دعم مناطق متعددة

**Best Practices:**

```yaml
# Production Configuration
eureka:
  server:
    enable-self-preservation: true  # تفعيل للـ Production
    eviction-interval-timer-in-ms: 60000
  instance:
    lease-renewal-interval-in-seconds: 30
    lease-expiration-duration-in-seconds: 90
```

---

### 7. ما الفرق بين Client-Side و Server-Side Discovery؟

**الشرح:**

#### Client-Side Discovery

**كيف يعمل:**
```
1. Client يسأل Service Registry عن الخدمة
2. Registry يرجع قائمة بالـ Instances
3. Client يختار Instance (Load Balancing)
4. Client يتصل بالـ Instance مباشرة
```

**Flow:**
```
┌────────┐  ① Query    ┌─────────────┐
│ Client │──────────→  │   Eureka    │
│        │  ② List     │  Registry   │
│        │←──────────  └─────────────┘
└────────┘
    │
    │ ③ Direct Call
    ↓
┌────────────┐
│  Service   │
│  Instance  │
└────────────┘
```

**مثال:**
```java
// Client-Side Discovery مع Eureka
@Service
public class OrderService {

    @Autowired
    private DiscoveryClient discoveryClient;

    @Autowired
    private RestTemplate restTemplate;

    public Payment callPaymentService(Long id) {
        // ① الحصول على القائمة من Eureka
        List<ServiceInstance> instances =
            discoveryClient.getInstances("PAYMENT-SERVICE");

        // ② اختيار Instance (Client-Side Load Balancing)
        ServiceInstance instance = chooseInstance(instances);

        // ③ الاتصال المباشر
        String url = instance.getUri() + "/api/payments/" + id;
        return restTemplate.getForObject(url, Payment.class);
    }

    private ServiceInstance chooseInstance(List<ServiceInstance> instances) {
        // Round Robin, Random, etc.
        return instances.get(new Random().nextInt(instances.size()));
    }
}
```

**مميزات:**
- ✅ Client يتحكم في Load Balancing
- ✅ اتصال مباشر (أسرع)
- ✅ لا يحتاج Load Balancer منفصل

**عيوب:**
- ❌ Client أكثر تعقيداً
- ❌ كل Client يحتاج Discovery Logic
- ❌ تحديث Clients صعب

**أدوات:**
- Netflix Eureka + Ribbon
- Spring Cloud LoadBalancer
- Consul

---

#### Server-Side Discovery

**كيف يعمل:**
```
1. Client يرسل Request للـ Load Balancer
2. Load Balancer يسأل Service Registry
3. Load Balancer يختار Instance
4. Load Balancer يوجه Request
```

**Flow:**
```
┌────────┐  ① Request  ┌──────────────┐  ② Query  ┌─────────┐
│ Client │──────────→  │     Load     │──────────→ │ Service │
│        │  ④ Response │   Balancer   │  ③ List   │Registry │
│        │←──────────  │   (Gateway)  │←────────── └─────────┘
└────────┘             └──────────────┘
                              │
                              │ ③ Route
                              ↓
                       ┌────────────┐
                       │  Service   │
                       │  Instance  │
                       └────────────┘
```

**مثال:**
```java
// Server-Side Discovery مع API Gateway
@Configuration
public class GatewayConfig {

    @Bean
    public RouteLocator routes(RouteLocatorBuilder builder) {
        return builder.routes()
            // Client يرسل لـ Gateway فقط
            .route("payment-service", r -> r
                .path("/api/payments/**")
                // Gateway يكتشف ويوجه
                .uri("lb://PAYMENT-SERVICE"))
            .build();
    }
}
```

**Client Code (بسيط):**
```java
@Service
public class OrderService {

    @Autowired
    private RestTemplate restTemplate;

    public Payment callPaymentService(Long id) {
        // Client لا يعرف شيء عن Service Discovery
        // يرسل للـ Gateway فقط
        String url = "http://api-gateway:8080/api/payments/" + id;
        return restTemplate.getForObject(url, Payment.class);
    }
}
```

**مميزات:**
- ✅ Client بسيط جداً
- ✅ Load Balancing مركزي
- ✅ سهل إضافة Security, Rate Limiting
- ✅ تحديثات مركزية

**عيوب:**
- ❌ Load Balancer = Single Point of Failure
- ❌ Hop إضافي (أبطأ قليلاً)
- ❌ حاجة لإدارة Load Balancer

**أدوات:**
- AWS ELB (Elastic Load Balancer)
- Nginx
- HAProxy
- Spring Cloud Gateway
- Kong

---

#### المقارنة الكاملة

| الميزة | Client-Side | Server-Side |
|--------|-------------|-------------|
| **Complexity** | Client معقد | Client بسيط |
| **Performance** | أسرع (Direct) | أبطأ قليلاً (Extra Hop) |
| **Load Balancing** | في Client | في Gateway |
| **Single Point of Failure** | لا | نعم (Gateway) |
| **Language Agnostic** | لا | نعم |
| **Security** | كل Client منفصل | مركزية في Gateway |
| **Maintenance** | صعب | سهل |
| **Examples** | Eureka+Ribbon | API Gateway, ELB |

---

#### متى تستخدم أيهما؟

**Client-Side Discovery:**
- لو كل الخدمات نفس التقنية (مثل كلها Java)
- لو تحتاج Performance عالي
- لو عندك Microservices داخلية فقط

**Server-Side Discovery:**
- لو عندك تقنيات مختلفة (Java, Node.js, Python)
- لو تحتاج Security مركزية
- لو عندك External Clients
- لو تحتاج API Gateway (أغلب الحالات)

---

#### Hybrid Approach (الأفضل)

```
External Clients → Server-Side (API Gateway)
Internal Services → Client-Side (Eureka + Feign)
```

**مثال:**
```
Mobile App ──→ API Gateway ──→ Order Service
                               (uses Feign)
                                    │
                                    ↓
                            Payment Service
```

**Code:**
```java
// External: عبر API Gateway
// gateway-config.yml
spring:
  cloud:
    gateway:
      routes:
        - id: orders
          uri: lb://ORDER-SERVICE
          predicates:
            - Path=/api/orders/**

// Internal: مباشرة مع Feign (Client-Side)
@FeignClient(name = "payment-service")
public interface PaymentClient {
    @GetMapping("/api/payments/{id}")
    Payment getPayment(@PathVariable Long id);
}
```

---

## API Gateway

### 8. ما هو API Gateway ولماذا نحتاجه؟

**الشرح:**

**API Gateway:**
- نقطة دخول واحدة لجميع الطلبات
- يوجه الطلبات للخدمات المناسبة
- Reverse Proxy للـ Microservices

**المشكلة بدون API Gateway:**

```
❌ بدون Gateway:

Mobile App ──→ Order Service (IP1:8081)
           ──→ Payment Service (IP2:8082)
           ──→ User Service (IP3:8083)
           ──→ Product Service (IP4:8084)

المشاكل:
1. Client يعرف كل الخدمات
2. CORS مشاكل
3. Authentication في كل خدمة
4. Security مكررة
5. لو Service انتقل؟
```

**الحل مع API Gateway:**

```
✅ مع Gateway:

Mobile App ──→ API Gateway (gateway.com)
                    │
                    ├──→ Order Service
                    ├──→ Payment Service
                    ├──→ User Service
                    └──→ Product Service
```

**وظائف API Gateway:**

#### 1. **Routing (التوجيه)**
```java
@Configuration
public class GatewayConfig {

    @Bean
    public RouteLocator routes(RouteLocatorBuilder builder) {
        return builder.routes()
            // /api/orders/** → ORDER-SERVICE
            .route("orders", r -> r
                .path("/api/orders/**")
                .uri("lb://ORDER-SERVICE"))

            // /api/payments/** → PAYMENT-SERVICE
            .route("payments", r -> r
                .path("/api/payments/**")
                .uri("lb://PAYMENT-SERVICE"))

            .build();
    }
}
```

#### 2. **Authentication & Authorization**
```java
@Component
public class AuthenticationFilter implements GlobalFilter {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String token = exchange.getRequest()
            .getHeaders()
            .getFirst("Authorization");

        if (token == null || !isValidToken(token)) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }

        return chain.filter(exchange);
    }
}
```

#### 3. **Load Balancing**
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: order-service
          uri: lb://ORDER-SERVICE  # lb = Load Balanced
          predicates:
            - Path=/api/orders/**
```

#### 4. **Rate Limiting**
```java
@Bean
public RouteLocator rateLimitedRoutes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("orders", r -> r
            .path("/api/orders/**")
            .filters(f -> f.requestRateLimiter(c -> c
                .setRateLimiter(redisRateLimiter())
                .setKeyResolver(userKeyResolver())))
            .uri("lb://ORDER-SERVICE"))
        .build();
}
```

#### 5. **Request/Response Transformation**
```java
@Component
public class ModifyRequestFilter implements GlobalFilter {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // إضافة Header
        ServerHttpRequest request = exchange.getRequest()
            .mutate()
            .header("X-Gateway", "API-Gateway")
            .header("X-Request-Time", String.valueOf(System.currentTimeMillis()))
            .build();

        return chain.filter(exchange.mutate().request(request).build());
    }
}
```

#### 6. **Circuit Breaker**
```java
@Bean
public RouteLocator circuitBreakerRoutes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("orders", r -> r
            .path("/api/orders/**")
            .filters(f -> f.circuitBreaker(c -> c
                .setName("orderServiceCircuitBreaker")
                .setFallbackUri("forward:/fallback/orders")))
            .uri("lb://ORDER-SERVICE"))
        .build();
}
```

#### 7. **Logging & Monitoring**
```java
@Component
@Slf4j
public class LoggingFilter implements GlobalFilter {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("Request: {} {}",
            exchange.getRequest().getMethod(),
            exchange.getRequest().getURI());

        long startTime = System.currentTimeMillis();

        return chain.filter(exchange).then(Mono.fromRunnable(() -> {
            long duration = System.currentTimeMillis() - startTime;
            log.info("Response: {} ({}ms)",
                exchange.getResponse().getStatusCode(),
                duration);
        }));
    }
}
```

#### 8. **CORS Handling**
```java
@Bean
public CorsWebFilter corsFilter() {
    CorsConfiguration config = new CorsConfiguration();
    config.setAllowedOrigins(Arrays.asList("http://localhost:3000"));
    config.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "DELETE"));
    config.setAllowedHeaders(Arrays.asList("*"));

    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", config);

    return new CorsWebFilter(source);
}
```

---

**لماذا نحتاج API Gateway؟**

#### 1. **Single Entry Point**
- Client يعرف عنوان واحد فقط
- سهولة في التغيير

#### 2. **Security**
- Authentication مركزية
- SSL/TLS Termination
- API Key Management

#### 3. **Cross-Cutting Concerns**
- Logging
- Monitoring
- Caching
- Compression

#### 4. **Protocol Translation**
- HTTP → gRPC
- REST → WebSocket

#### 5. **API Composition**
```java
// Gateway يجمع من عدة خدمات
@RestController
public class CompositionController {

    @Autowired
    private WebClient.Builder webClient;

    @GetMapping("/api/order-details/{id}")
    public Mono<OrderDetails> getOrderDetails(@PathVariable Long id) {
        Mono<Order> orderMono = webClient.build()
            .get()
            .uri("lb://ORDER-SERVICE/api/orders/" + id)
            .retrieve()
            .bodyToMono(Order.class);

        Mono<Payment> paymentMono = webClient.build()
            .get()
            .uri("lb://PAYMENT-SERVICE/api/payments/order/" + id)
            .retrieve()
            .bodyToMono(Payment.class);

        return Mono.zip(orderMono, paymentMono)
            .map(tuple -> new OrderDetails(tuple.getT1(), tuple.getT2()));
    }
}
```

---

**مثال كامل - Spring Cloud Gateway:**

**pom.xml:**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

**application.yml:**
```yaml
server:
  port: 8080

spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      routes:
        # Order Service Routes
        - id: order-service
          uri: lb://ORDER-SERVICE
          predicates:
            - Path=/api/orders/**
          filters:
            - RewritePath=/api/orders/(?<segment>.*), /${segment}
            - AddRequestHeader=X-Gateway, API-Gateway

        # Payment Service Routes
        - id: payment-service
          uri: lb://PAYMENT-SERVICE
          predicates:
            - Path=/api/payments/**
          filters:
            - name: CircuitBreaker
              args:
                name: paymentCircuitBreaker
                fallbackUri: forward:/fallback/payments

        # User Service Routes (مع Authentication)
        - id: user-service
          uri: lb://USER-SERVICE
          predicates:
            - Path=/api/users/**
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20

      # Default Filters (تطبق على كل Routes)
      default-filters:
        - DedupeResponseHeader=Access-Control-Allow-Credentials Access-Control-Allow-Origin
        - name: Retry
          args:
            retries: 3
            statuses: BAD_GATEWAY,SERVICE_UNAVAILABLE
            methods: GET
            backoff:
              firstBackoff: 10ms
              maxBackoff: 50ms

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

**Main Class:**
```java
@SpringBootApplication
@EnableEurekaClient
public class ApiGatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
    }
}
```

---

**مميزات:**
- ✅ Centralized Security
- ✅ Simplified Client
- ✅ Load Balancing
- ✅ Monitoring & Analytics
- ✅ API Versioning

**عيوب:**
- ❌ Single Point of Failure
- ❌ Extra Network Hop
- ❌ Complexity

---

### 9. اشرح Spring Cloud Gateway أو Zuul

**الشرح:**

#### Spring Cloud Gateway vs Zuul

| Feature | Spring Cloud Gateway | Zuul 1.x | Zuul 2.x |
|---------|---------------------|----------|----------|
| **Model** | Reactive (Non-blocking) | Blocking | Non-blocking |
| **Base** | Spring WebFlux | Servlet API | Netty |
| **Performance** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Support** | Active | Maintenance | Active (Netflix) |
| **Spring Integration** | ممتاز | جيد | متوسط |

**الأفضل حالياً: Spring Cloud Gateway**

---

#### Spring Cloud Gateway - شرح مفصل

**المكونات الأساسية:**

```
┌──────────────────────────────────────────┐
│      Spring Cloud Gateway                │
│  ┌────────────────────────────────────┐  │
│  │  1. Route Predicate Factory        │  │
│  │     (شروط التوجيه)                 │  │
│  ├────────────────────────────────────┤  │
│  │  2. Gateway Filter Factory         │  │
│  │     (تعديل Request/Response)       │  │
│  ├────────────────────────────────────┤  │
│  │  3. Gateway Handler                │  │
│  │     (معالج الطلبات)                │  │
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘
```

---

#### 1. Route Predicate (شروط التوجيه)

**أنواع Predicates:**

```java
@Configuration
public class GatewayRoutes {

    @Bean
    public RouteLocator customRoutes(RouteLocatorBuilder builder) {
        return builder.routes()

            // ① Path Predicate
            .route("path_route", r -> r
                .path("/api/orders/**")
                .uri("lb://ORDER-SERVICE"))

            // ② Method Predicate
            .route("method_route", r -> r
                .path("/api/products/**")
                .and().method(HttpMethod.GET)
                .uri("lb://PRODUCT-SERVICE"))

            // ③ Header Predicate
            .route("header_route", r -> r
                .header("X-Request-Type", "mobile")
                .uri("lb://MOBILE-SERVICE"))

            // ④ Query Predicate
            .route("query_route", r -> r
                .query("version", "v2")
                .uri("lb://SERVICE-V2"))

            // ⑤ Cookie Predicate
            .route("cookie_route", r -> r
                .cookie("user-type", "premium")
                .uri("lb://PREMIUM-SERVICE"))

            // ⑥ Host Predicate
            .route("host_route", r -> r
                .host("*.example.com")
                .uri("lb://MAIN-SERVICE"))

            // ⑦ Time-Based Predicate
            .route("time_route", r -> r
                .between(
                    ZonedDateTime.parse("2024-01-01T00:00:00+00:00"),
                    ZonedDateTime.parse("2024-12-31T23:59:59+00:00"))
                .uri("lb://SEASONAL-SERVICE"))

            // ⑧ Weight-Based (A/B Testing)
            .route("weight_route_a", r -> r
                .weight("group1", 80)  // 80%
                .uri("lb://SERVICE-A"))
            .route("weight_route_b", r -> r
                .weight("group1", 20)  // 20%
                .uri("lb://SERVICE-B"))

            // ⑨ Combined Predicates
            .route("combined_route", r -> r
                .path("/api/admin/**")
                .and().header("Authorization")
                .and().method(HttpMethod.POST, HttpMethod.PUT)
                .uri("lb://ADMIN-SERVICE"))

            .build();
    }
}
```

**YAML Configuration:**
```yaml
spring:
  cloud:
    gateway:
      routes:
        # Path-based routing
        - id: order_service
          uri: lb://ORDER-SERVICE
          predicates:
            - Path=/api/orders/**
            - Method=GET,POST

        # Header-based routing
        - id: mobile_api
          uri: lb://MOBILE-API
          predicates:
            - Path=/api/**
            - Header=X-Platform, mobile

        # Time-based routing
        - id: maintenance
          uri: http://maintenance.page.com
          predicates:
            - Between=2024-12-25T00:00:00+00:00,2024-12-25T06:00:00+00:00
```

---

#### 2. Gateway Filters

**أنواع Filters:**

##### A. Pre Filters (قبل إرسال Request)

```java
@Component
public class PreFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest()
            .mutate()
            // إضافة Headers
            .header("X-Gateway-Time", String.valueOf(System.currentTimeMillis()))
            .header("X-Gateway-Version", "1.0")
            // إضافة User Info من Token
            .header("X-User-Id", extractUserId(exchange))
            .build();

        return chain.filter(exchange.mutate().request(request).build());
    }

    @Override
    public int getOrder() {
        return -1; // يعمل قبل Filters الأخرى
    }
}
```

##### B. Post Filters (بعد استلام Response)

```java
@Component
public class PostFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        return chain.filter(exchange).then(Mono.fromRunnable(() -> {
            ServerHttpResponse response = exchange.getResponse();

            // إضافة Headers للـ Response
            response.getHeaders().add("X-Response-Time",
                String.valueOf(System.currentTimeMillis()));
            response.getHeaders().add("X-Powered-By", "Spring Cloud Gateway");
        }));
    }

    @Override
    public int getOrder() {
        return 1; // يعمل بعد Filters الأخرى
    }
}
```

##### C. Built-in Filters

```java
@Bean
public RouteLocator builtInFiltersRoutes(RouteLocatorBuilder builder) {
    return builder.routes()

        .route("filters_demo", r -> r
            .path("/api/demo/**")
            .filters(f -> f
                // ① إضافة Request Header
                .addRequestHeader("X-Request-Source", "Gateway")

                // ② إضافة Response Header
                .addResponseHeader("X-Response-Source", "Gateway")

                // ③ إعادة كتابة Path
                .rewritePath("/api/demo/(?<segment>.*)", "/${segment}")

                // ④ إزالة Header
                .removeRequestHeader("Cookie")

                // ⑤ إعادة كتابة Response Body
                .modifyResponseBody(String.class, String.class,
                    (exchange, body) -> Mono.just(body.toUpperCase()))

                // ⑥ Retry
                .retry(config -> config
                    .setRetries(3)
                    .setStatuses(HttpStatus.SERVICE_UNAVAILABLE)
                    .setBackoff(Duration.ofMillis(100),
                               Duration.ofMillis(1000),
                               2, true))

                // ⑦ Circuit Breaker
                .circuitBreaker(config -> config
                    .setName("myCircuitBreaker")
                    .setFallbackUri("forward:/fallback"))

                // ⑧ Request Rate Limiter
                .requestRateLimiter(config -> config
                    .setRateLimiter(redisRateLimiter())
                    .setKeyResolver(userKeyResolver()))

                // ⑨ Redirect
                .redirect(HttpStatus.PERMANENT_REDIRECT, "https://new-url.com")

                // ⑩ Set Path
                .setPath("/new-path")

                // ⑪ Set Status
                .setStatus(HttpStatus.OK)

                // ⑫ Preserve Host Header
                .preserveHostHeader()

                // ⑬ Request Size Limiter
                .setRequestSize(1000000L) // 1MB

                // ⑭ Strip Prefix
                .stripPrefix(1) // /api/orders → /orders

                // ⑮ Prefix Path
                .prefixPath("/api")
            )
            .uri("lb://DEMO-SERVICE"))

        .build();
}
```

##### D. Custom Filters

```java
// Custom Filter Factory
@Component
public class CustomFilterFactory extends AbstractGatewayFilterFactory<CustomFilterFactory.Config> {

    public CustomFilterFactory() {
        super(Config.class);
    }

    @Override
    public GatewayFilter apply(Config config) {
        return (exchange, chain) -> {
            ServerHttpRequest request = exchange.getRequest();

            // Custom Logic
            if (config.isPreLogger()) {
                System.out.println("Pre Filter: " + request.getURI());
            }

            return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                if (config.isPostLogger()) {
                    System.out.println("Post Filter: " +
                        exchange.getResponse().getStatusCode());
                }
            }));
        };
    }

    @Data
    public static class Config {
        private boolean preLogger;
        private boolean postLogger;
    }
}

// استخدامه
@Bean
public RouteLocator customFilterRoute(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("custom_filter", r -> r
            .path("/api/custom/**")
            .filters(f -> f.filter(new CustomFilterFactory().apply(
                new CustomFilterFactory.Config() {{
                    setPreLogger(true);
                    setPostLogger(true);
                }}
            )))
            .uri("lb://CUSTOM-SERVICE"))
        .build();
}
```

---

#### 3. Authentication Filter (مثال عملي)

```java
@Component
@Slf4j
public class AuthenticationFilter implements GlobalFilter, Ordered {

    @Autowired
    private JwtUtil jwtUtil;

    private static final List<String> OPEN_ENDPOINTS = Arrays.asList(
        "/api/auth/login",
        "/api/auth/register",
        "/actuator/health"
    );

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        String path = request.getURI().getPath();

        // السماح بالـ Open Endpoints
        if (OPEN_ENDPOINTS.stream().anyMatch(path::startsWith)) {
            return chain.filter(exchange);
        }

        // فحص Token
        String token = extractToken(request);

        if (token == null) {
            return onError(exchange, "Missing authorization header", HttpStatus.UNAUTHORIZED);
        }

        try {
            // التحقق من Token
            if (!jwtUtil.validateToken(token)) {
                return onError(exchange, "Invalid token", HttpStatus.UNAUTHORIZED);
            }

            // استخراج معلومات User
            String userId = jwtUtil.extractUserId(token);
            String role = jwtUtil.extractRole(token);

            // إضافتها للـ Request
            ServerHttpRequest modifiedRequest = request.mutate()
                .header("X-User-Id", userId)
                .header("X-User-Role", role)
                .build();

            return chain.filter(exchange.mutate().request(modifiedRequest).build());

        } catch (Exception e) {
            log.error("Authentication error", e);
            return onError(exchange, "Authentication failed", HttpStatus.UNAUTHORIZED);
        }
    }

    private String extractToken(ServerHttpRequest request) {
        String bearerToken = request.getHeaders().getFirst("Authorization");
        if (bearerToken != null && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);
        }
        return null;
    }

    private Mono<Void> onError(ServerWebExchange exchange, String message, HttpStatus status) {
        ServerHttpResponse response = exchange.getResponse();
        response.setStatusCode(status);
        response.getHeaders().setContentType(MediaType.APPLICATION_JSON);

        ErrorResponse errorResponse = new ErrorResponse(message, status.value());
        DataBuffer buffer = response.bufferFactory().wrap(
            new ObjectMapper().writeValueAsBytes(errorResponse)
        );

        return response.writeWith(Mono.just(buffer));
    }

    @Override
    public int getOrder() {
        return -100; // أول Filter يعمل
    }
}
```

---

#### 4. Rate Limiting

```java
@Configuration
public class RateLimitingConfig {

    // Key Resolver based on User ID
    @Bean
    public KeyResolver userKeyResolver() {
        return exchange -> Mono.just(
            exchange.getRequest()
                .getHeaders()
                .getFirst("X-User-Id")
        );
    }

    // Key Resolver based on IP
    @Bean
    public KeyResolver ipKeyResolver() {
        return exchange -> Mono.just(
            exchange.getRequest()
                .getRemoteAddress()
                .getAddress()
                .getHostAddress()
        );
    }

    @Bean
    public RedisRateLimiter redisRateLimiter() {
        return new RedisRateLimiter(
            10,  // replenishRate: 10 requests per second
            20   // burstCapacity: max 20 requests at once
        );
    }
}
```

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: rate_limited_service
          uri: lb://SERVICE
          predicates:
            - Path=/api/**
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20
                key-resolver: "#{@userKeyResolver}"
```

---

#### 5. Circuit Breaker مع Resilience4j

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
</dependency>
```

```java
@Bean
public RouteLocator circuitBreakerRoutes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("circuit_breaker_route", r -> r
            .path("/api/orders/**")
            .filters(f -> f
                .circuitBreaker(config -> config
                    .setName("orderServiceCircuitBreaker")
                    .setFallbackUri("forward:/fallback/orders")
                    .setStatusCodes("500", "502", "503", "504")))
            .uri("lb://ORDER-SERVICE"))
        .build();
}

@RestController
public class FallbackController {

    @RequestMapping("/fallback/orders")
    public Mono<Map<String, String>> orderFallback() {
        return Mono.just(Map.of(
            "message", "Order service is currently unavailable",
            "status", "Please try again later"
        ));
    }
}
```

```yaml
resilience4j:
  circuitbreaker:
    instances:
      orderServiceCircuitBreaker:
        slidingWindowSize: 10
        failureRateThreshold: 50
        waitDurationInOpenState: 10000
        permittedNumberOfCallsInHalfOpenState: 3
```

---

#### 6. Logging & Monitoring

```java
@Component
@Slf4j
public class LoggingFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();

        // Request Logging
        log.info("Incoming Request: {} {} from {}",
            request.getMethod(),
            request.getURI(),
            request.getRemoteAddress());

        long startTime = System.currentTimeMillis();

        return chain.filter(exchange).then(Mono.fromRunnable(() -> {
            ServerHttpResponse response = exchange.getResponse();
            long duration = System.currentTimeMillis() - startTime;

            // Response Logging
            log.info("Outgoing Response: {} {} - Status: {} - Duration: {}ms",
                request.getMethod(),
                request.getURI(),
                response.getStatusCode(),
                duration);
        }));
    }

    @Override
    public int getOrder() {
        return Ordered.LOWEST_PRECEDENCE;
    }
}
```

---

#### 7. Configuration الكاملة

```yaml
server:
  port: 8080

spring:
  application:
    name: api-gateway

  cloud:
    gateway:
      # Global CORS Configuration
      globalcors:
        cors-configurations:
          '[/**]':
            allowedOrigins: "http://localhost:3000"
            allowedMethods:
              - GET
              - POST
              - PUT
              - DELETE
            allowedHeaders: "*"
            allowCredentials: true

      # Routes
      routes:
        # Order Service
        - id: order-service
          uri: lb://ORDER-SERVICE
          predicates:
            - Path=/api/orders/**
          filters:
            - RewritePath=/api/orders/(?<segment>.*), /${segment}
            - name: CircuitBreaker
              args:
                name: orderCB
                fallbackUri: forward:/fallback/orders
            - name: Retry
              args:
                retries: 3
                statuses: SERVICE_UNAVAILABLE

        # Payment Service
        - id: payment-service
          uri: lb://PAYMENT-SERVICE
          predicates:
            - Path=/api/payments/**
          filters:
            - StripPrefix=2
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20

      # Default Filters (على كل Routes)
      default-filters:
        - AddRequestHeader=X-Gateway, API-Gateway
        - AddResponseHeader=X-Powered-By, Spring Cloud Gateway

# Eureka Client
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true

# Actuator
management:
  endpoints:
    web:
      exposure:
        include: health,info,gateway
  endpoint:
    gateway:
      enabled: true

# Resilience4j
resilience4j:
  circuitbreaker:
    configs:
      default:
        slidingWindowSize: 10
        failureRateThreshold: 50
        waitDurationInOpenState: 10000
  timelimiter:
    configs:
      default:
        timeoutDuration: 3s
```

---

**Spring Cloud Gateway vs Zuul - الخلاصة:**

استخدم **Spring Cloud Gateway** لأنه:
- ✅ أسرع (Reactive)
- ✅ دعم رسمي من Spring
- ✅ Features أكثر
- ✅ Integration أفضل مع Spring Boot

---

### 10. كيف تطبق Authentication/Authorization في API Gateway؟

**الشرح المفصل:**

#### الاستراتيجية العامة

```
┌─────────────┐  ① Login      ┌──────────────┐
│   Client    │──────────────→ │ Auth Service │
│             │  ② JWT Token  │              │
│             │←────────────── └──────────────┘
└─────────────┘
      │
      │ ③ Request + Token
      ↓
┌──────────────────────────────────────────┐
│         API Gateway                       │
│  ┌────────────────────────────────────┐  │
│  │  Authentication Filter             │  │
│  │  - Validate JWT                    │  │
│  │  - Extract User Info               │  │
│  │  - Check Permissions               │  │
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘
      │
      │ ④ Forward (with User Info)
      ↓
┌─────────────┐
│  Services   │
└─────────────┘
```

---

#### Implementation الكامل

##### 1. JWT Utility Class

```java
@Component
public class JwtUtil {

    @Value("${jwt.secret}")
    private String secret;

    @Value("${jwt.expiration}")
    private Long expiration;

    private Key getSigningKey() {
        byte[] keyBytes = Decoders.BASE64.decode(secret);
        return Keys.hmacShaKeyFor(keyBytes);
    }

    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("roles", userDetails.getAuthorities()
            .stream()
            .map(GrantedAuthority::getAuthority)
            .collect(Collectors.toList()));

        return Jwts.builder()
            .setClaims(claims)
            .setSubject(userDetails.getUsername())
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + expiration))
            .signWith(getSigningKey(), SignatureAlgorithm.HS256)
            .compact();
    }

    public boolean validateToken(String token) {
        try {
            Jwts.parserBuilder()
                .setSigningKey(getSigningKey())
                .build()
                .parseClaimsJws(token);
            return true;
        } catch (JwtException | IllegalArgumentException e) {
            return false;
        }
    }

    public Claims extractAllClaims(String token) {
        return Jwts.parserBuilder()
            .setSigningKey(getSigningKey())
            .build()
            .parseClaimsJws(token)
            .getBody();
    }

    public String extractUsername(String token) {
        return extractAllClaims(token).getSubject();
    }

    public List<String> extractRoles(String token) {
        Claims claims = extractAllClaims(token);
        return (List<String>) claims.get("roles");
    }

    public boolean isTokenExpired(String token) {
        return extractAllClaims(token)
            .getExpiration()
            .before(new Date());
    }
}
```

---

##### 2. Authentication Filter

```java
@Component
@Slf4j
public class JwtAuthenticationFilter implements GlobalFilter, Ordered {

    @Autowired
    private JwtUtil jwtUtil;

    @Autowired
    private RouteValidator routeValidator;

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();

        // تخطي المسارات المفتوحة
        if (routeValidator.isSecured.test(request)) {
            // التحقق من وجود Token
            if (!request.getHeaders().containsKey(HttpHeaders.AUTHORIZATION)) {
                return onError(exchange, "Missing authorization header", HttpStatus.UNAUTHORIZED);
            }

            String token = extractToken(request);

            if (token == null) {
                return onError(exchange, "Invalid authorization header format", HttpStatus.UNAUTHORIZED);
            }

            try {
                // التحقق من صحة Token
                if (!jwtUtil.validateToken(token)) {
                    return onError(exchange, "Invalid or expired token", HttpStatus.UNAUTHORIZED);
                }

                // استخراج معلومات المستخدم
                String username = jwtUtil.extractUsername(token);
                List<String> roles = jwtUtil.extractRoles(token);

                // إضافة المعلومات للـ Request Headers
                ServerHttpRequest modifiedRequest = request.mutate()
                    .header("X-User-Name", username)
                    .header("X-User-Roles", String.join(",", roles))
                    .build();

                log.info("Authenticated user: {} with roles: {}", username, roles);

                return chain.filter(exchange.mutate().request(modifiedRequest).build());

            } catch (Exception e) {
                log.error("JWT validation error", e);
                return onError(exchange, "Authentication failed: " + e.getMessage(),
                             HttpStatus.UNAUTHORIZED);
            }
        }

        return chain.filter(exchange);
    }

    private String extractToken(ServerHttpRequest request) {
        String bearerToken = request.getHeaders().getFirst(HttpHeaders.AUTHORIZATION);
        if (bearerToken != null && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);
        }
        return null;
    }

    private Mono<Void> onError(ServerWebExchange exchange, String message, HttpStatus status) {
        ServerHttpResponse response = exchange.getResponse();
        response.setStatusCode(status);
        response.getHeaders().setContentType(MediaType.APPLICATION_JSON);

        ErrorResponse errorResponse = ErrorResponse.builder()
            .timestamp(LocalDateTime.now())
            .status(status.value())
            .error(status.getReasonPhrase())
            .message(message)
            .path(exchange.getRequest().getURI().getPath())
            .build();

        try {
            ObjectMapper mapper = new ObjectMapper();
            mapper.registerModule(new JavaTimeModule());
            byte[] bytes = mapper.writeValueAsBytes(errorResponse);
            DataBuffer buffer = response.bufferFactory().wrap(bytes);
            return response.writeWith(Mono.just(buffer));
        } catch (JsonProcessingException e) {
            log.error("Error creating error response", e);
            return response.setComplete();
        }
    }

    @Override
    public int getOrder() {
        return -100; // أعلى أولوية
    }
}

@Data
@Builder
class ErrorResponse {
    private LocalDateTime timestamp;
    private int status;
    private String error;
    private String message;
    private String path;
}
```

---

##### 3. Route Validator

```java
@Component
public class RouteValidator {

    // المسارات التي لا تحتاج Authentication
    public static final List<String> OPEN_ENDPOINTS = List.of(
        "/api/auth/login",
        "/api/auth/register",
        "/api/auth/refresh",
        "/api/auth/forgot-password",
        "/actuator/health",
        "/actuator/info",
        "/swagger-ui",
        "/v3/api-docs"
    );

    // فحص إذا المسار يحتاج Authentication
    public Predicate<ServerHttpRequest> isSecured =
        request -> OPEN_ENDPOINTS.stream()
            .noneMatch(uri -> request.getURI().getPath().startsWith(uri));
}
```

---

##### 4. Authorization Filter (Role-Based)

```java
@Component
@Slf4j
public class RoleAuthorizationFilter implements GlobalFilter, Ordered {

    // تعريف الصلاحيات المطلوبة لكل Path
    private static final Map<String, List<String>> ROUTE_ROLES = Map.of(
        "/api/admin", List.of("ROLE_ADMIN"),
        "/api/users", List.of("ROLE_USER", "ROLE_ADMIN"),
        "/api/orders/create", List.of("ROLE_USER", "ROLE_ADMIN"),
        "/api/orders/delete", List.of("ROLE_ADMIN"),
        "/api/reports", List.of("ROLE_ADMIN", "ROLE_MANAGER")
    );

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        String path = request.getURI().getPath();

        // التحقق من الصلاحيات
        List<String> requiredRoles = getRequiredRoles(path);

        if (requiredRoles != null && !requiredRoles.isEmpty()) {
            String userRoles = request.getHeaders().getFirst("X-User-Roles");

            if (userRoles == null) {
                return onError(exchange, "User roles not found", HttpStatus.FORBIDDEN);
            }

            List<String> userRolesList = Arrays.asList(userRoles.split(","));

            // فحص إذا User لديه الصلاحية
            boolean hasPermission = userRolesList.stream()
                .anyMatch(requiredRoles::contains);

            if (!hasPermission) {
                log.warn("User with roles {} tried to access {} which requires {}",
                    userRolesList, path, requiredRoles);
                return onError(exchange,
                    "Insufficient permissions to access this resource",
                    HttpStatus.FORBIDDEN);
            }

            log.info("User authorized to access {}", path);
        }

        return chain.filter(exchange);
    }

    private List<String> getRequiredRoles(String path) {
        return ROUTE_ROLES.entrySet().stream()
            .filter(entry -> path.startsWith(entry.getKey()))
            .map(Map.Entry::getValue)
            .findFirst()
            .orElse(null);
    }

    private Mono<Void> onError(ServerWebExchange exchange, String message, HttpStatus status) {
        ServerHttpResponse response = exchange.getResponse();
        response.setStatusCode(status);
        response.getHeaders().setContentType(MediaType.APPLICATION_JSON);

        ErrorResponse errorResponse = ErrorResponse.builder()
            .timestamp(LocalDateTime.now())
            .status(status.value())
            .error(status.getReasonPhrase())
            .message(message)
            .path(exchange.getRequest().getURI().getPath())
            .build();

        try {
            byte[] bytes = new ObjectMapper()
                .registerModule(new JavaTimeModule())
                .writeValueAsBytes(errorResponse);
            DataBuffer buffer = response.bufferFactory().wrap(bytes);
            return response.writeWith(Mono.just(buffer));
        } catch (JsonProcessingException e) {
            return response.setComplete();
        }
    }

    @Override
    public int getOrder() {
        return -99; // بعد Authentication Filter
    }
}
```

---

##### 5. Auth Service (Login Endpoint)

```java
@RestController
@RequestMapping("/api/auth")
public class AuthController {

    @Autowired
    private AuthenticationManager authenticationManager;

    @Autowired
    private JwtUtil jwtUtil;

    @Autowired
    private UserDetailsService userDetailsService;

    @PostMapping("/login")
    public ResponseEntity<?> login(@RequestBody LoginRequest request) {
        try {
            // Authentication
            authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(
                    request.getUsername(),
                    request.getPassword()
                )
            );

            // Load User Details
            UserDetails userDetails = userDetailsService.loadUserByUsername(request.getUsername());

            // Generate Token
            String token = jwtUtil.generateToken(userDetails);

            return ResponseEntity.ok(AuthResponse.builder()
                .token(token)
                .type("Bearer")
                .username(userDetails.getUsername())
                .roles(userDetails.getAuthorities().stream()
                    .map(GrantedAuthority::getAuthority)
                    .collect(Collectors.toList()))
                .expiresIn(jwtUtil.getExpiration())
                .build());

        } catch (BadCredentialsException e) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
                .body("Invalid username or password");
        }
    }

    @PostMapping("/refresh")
    public ResponseEntity<?> refreshToken(@RequestHeader("Authorization") String bearerToken) {
        String token = bearerToken.substring(7);

        if (jwtUtil.validateToken(token)) {
            String username = jwtUtil.extractUsername(token);
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);
            String newToken = jwtUtil.generateToken(userDetails);

            return ResponseEntity.ok(Map.of("token", newToken));
        }

        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body("Invalid token");
    }
}

@Data
class LoginRequest {
    private String username;
    private String password;
}

@Data
@Builder
class AuthResponse {
    private String token;
    private String type;
    private String username;
    private List<String> roles;
    private Long expiresIn;
}
```

---

##### 6. Configuration Files

**application.yml:**
```yaml
server:
  port: 8080

spring:
  application:
    name: api-gateway

  cloud:
    gateway:
      routes:
        # Auth Service (Open)
        - id: auth-service
          uri: lb://AUTH-SERVICE
          predicates:
            - Path=/api/auth/**

        # User Service (Authenticated)
        - id: user-service
          uri: lb://USER-SERVICE
          predicates:
            - Path=/api/users/**

        # Admin Service (Admin Only)
        - id: admin-service
          uri: lb://ADMIN-SERVICE
          predicates:
            - Path=/api/admin/**

        # Order Service (User + Admin)
        - id: order-service
          uri: lb://ORDER-SERVICE
          predicates:
            - Path=/api/orders/**

# JWT Configuration
jwt:
  secret: mySecretKeyThatIsAtLeast256BitsLongForHS256AlgorithmToWorkProperly
  expiration: 86400000  # 24 hours

# Eureka
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

---

##### 7. استخدام المعلومات في Services

```java
// في Order Service مثلاً
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    @GetMapping("/my-orders")
    public List<Order> getMyOrders(@RequestHeader("X-User-Name") String username) {
        // استخدام Username من Gateway
        return orderService.findByUsername(username);
    }

    @PostMapping
    public Order createOrder(
            @RequestBody OrderRequest request,
            @RequestHeader("X-User-Name") String username,
            @RequestHeader("X-User-Roles") String roles) {

        // التحقق من الصلاحيات داخل Service أيضاً
        List<String> userRoles = Arrays.asList(roles.split(","));

        if (request.getAmount() > 10000 && !userRoles.contains("ROLE_PREMIUM")) {
            throw new InsufficientPermissionException("Large orders require premium membership");
        }

        return orderService.create(request, username);
    }
}
```

---

##### 8. OAuth2 / OpenID Connect (Alternative)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

```java
@Configuration
@EnableWebFluxSecurity
public class SecurityConfig {

    @Bean
    public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
        http
            .authorizeExchange(exchanges -> exchanges
                .pathMatchers("/api/auth/**").permitAll()
                .pathMatchers("/api/admin/**").hasRole("ADMIN")
                .pathMatchers("/api/users/**").hasAnyRole("USER", "ADMIN")
                .anyExchange().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt.jwtAuthenticationConverter(jwtAuthenticationConverter()))
            )
            .csrf().disable();

        return http.build();
    }

    @Bean
    public Converter<Jwt, Mono<AbstractAuthenticationToken>> jwtAuthenticationConverter() {
        JwtAuthenticationConverter converter = new JwtAuthenticationConverter();
        converter.setJwtGrantedAuthoritiesConverter(new CustomAuthoritiesConverter());
        return new ReactiveJwtAuthenticationConverterAdapter(converter);
    }
}
```

---

**Best Practices:**

1. ✅ **Token في Authorization Header فقط** (لا تضعه في URL)
2. ✅ **HTTPS دائماً** في Production
3. ✅ **Token Expiration قصير** + Refresh Token
4. ✅ **Validate Token في Gateway فقط** (Services تثق بالـ Gateway)
5. ✅ **Log كل Authentication Failures**
6. ✅ **Rate Limiting على Login Endpoint**
7. ✅ **Blacklist للـ Tokens (Redis)**

---

## Communication بين Services

### 11. ما الفرق بين Synchronous و Asynchronous Communication؟

**الشرح:**

#### Synchronous Communication (الاتصال المتزامن)

**التعريف:**
- Client يرسل Request وينتظر Response
- Blocking Operation
- مثل: REST API, gRPC

**كيف يعمل:**
```
Client ──→ Request  ──→ Service A
   ↓                      ↓
   │                   Processing
   │                      ↓
   ←── Response ←────────┘

⏱️ Client ينتظر حتى يأتي الرد
```

**مثال - REST Call:**
```java
@Service
public class OrderService {

    @Autowired
    private RestTemplate restTemplate;

    public Order createOrder(OrderRequest request) {
        // ① Create Order
        Order order = orderRepository.save(new Order(request));

        // ② Synchronous Call - ينتظر الرد
        Payment payment = restTemplate.postForObject(
            "http://PAYMENT-SERVICE/api/payments",
            new PaymentRequest(order.getId(), order.getAmount()),
            Payment.class
        );

        // ③ ما يكمل إلا لما Payment يرجع
        order.setPaymentId(payment.getId());
        order.setStatus("PAID");

        return orderRepository.save(order);
    }
}
```

**Timeline:**
```
0ms   → Create Order
10ms  → Call Payment Service ⏳
10ms  → Payment Processing...
500ms ← Payment Response ✓
510ms → Update Order
520ms ← Return to Client
```

**مميزات:**
- ✅ بسيط ومباشر
- ✅ Immediate Feedback
- ✅ Strong Consistency
- ✅ سهل في الـ Debugging

**عيوب:**
- ❌ Tight Coupling
- ❌ Blocking (Client ينتظر)
- ❌ Cascading Failures
- ❌ لو Service بطيء، كل شيء بطيء

---

#### Asynchronous Communication (الاتصال غير المتزامن)

**التعريف:**
- Client يرسل Request ولا ينتظر Response
- Non-Blocking Operation
- مثل: Message Queues (RabbitMQ, Kafka)

**كيف يعمل:**
```
Client ──→ Message ──→ Queue ──→ Service A
   ↓                              ↓
   │                          Processing
   ↓                              ↓
Returns Immediately         (في الخلفية)
```

**مثال - Message Queue:**
```java
@Service
public class OrderService {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Autowired
    private OrderRepository orderRepository;

    public Order createOrder(OrderRequest request) {
        // ① Create Order
        Order order = orderRepository.save(new Order(request));
        order.setStatus("PENDING");

        // ② Send Message (لا ينتظر)
        PaymentMessage message = PaymentMessage.builder()
            .orderId(order.getId())
            .amount(order.getAmount())
            .build();

        rabbitTemplate.convertAndSend("payment.exchange", "payment.create", message);

        // ③ يرجع مباشرة
        return order; // status = PENDING
    }
}

// Payment Service يستقبل ويعالج
@Component
public class PaymentListener {

    @RabbitListener(queues = "payment.queue")
    public void handlePayment(PaymentMessage message) {
        // معالجة الدفع في الخلفية
        Payment payment = paymentService.process(message);

        // إرسال رد
        rabbitTemplate.convertAndSend("order.exchange", "payment.completed",
            new PaymentCompletedEvent(message.getOrderId(), payment.getId()));
    }
}

// Order Service يستقبل النتيجة
@Component
public class OrderListener {

    @RabbitListener(queues = "order.queue")
    public void handlePaymentCompleted(PaymentCompletedEvent event) {
        Order order = orderRepository.findById(event.getOrderId());
        order.setPaymentId(event.getPaymentId());
        order.setStatus("PAID");
        orderRepository.save(order);

        // إرسال Notification للـ User
        notificationService.notifyUser(order);
    }
}
```

**Timeline:**
```
0ms   → Create Order
10ms  → Send Message to Queue ✓
15ms  ← Return to Client (status=PENDING)

... في الخلفية ...
100ms → Payment Service يستقبل
500ms → Payment Processing
600ms → Send Completion Event
700ms → Order Service يحدث الحالة
```

**مميزات:**
- ✅ Loose Coupling
- ✅ Non-Blocking (أسرع للـ Client)
- ✅ Scalability أفضل
- ✅ Fault Tolerance (Queue يحفظ الـ Messages)
- ✅ Retry Mechanism

**عيوب:**
- ❌ أكثر تعقيداً
- ❌ Eventual Consistency
- ❌ Debugging صعب
- ❌ Message Order مشاكل محتملة

---

#### المقارنة التفصيلية

| الميزة | Synchronous | Asynchronous |
|--------|-------------|--------------|
| **Response** | Immediate | Eventual |
| **Coupling** | Tight | Loose |
| **Performance** | Client ينتظر | Non-Blocking |
| **Consistency** | Strong | Eventual |
| **Complexity** | بسيط | معقد |
| **Failure Handling** | Immediate Error | Retry + Dead Letter Queue |
| **Scaling** | صعب | سهل |
| **Use Case** | Read Operations | Long Operations |
| **Example** | REST, gRPC | RabbitMQ, Kafka, SQS |

---

#### متى تستخدم أيهما؟

**استخدم Synchronous إذا:**
1. **تحتاج Immediate Response**
   ```java
   // User Login - لازم رد فوري
   @PostMapping("/login")
   public TokenResponse login(@RequestBody LoginRequest request) {
       return authService.authenticate(request); // Synchronous
   }
   ```

2. **Read Operations**
   ```java
   // Get Product Details
   @GetMapping("/products/{id}")
   public Product getProduct(@PathVariable Long id) {
       return productService.findById(id); // Synchronous
   }
   ```

3. **Transaction المهمة**
   ```java
   // Bank Transfer - لازم تتأكد نجح
   public TransferResult transfer(TransferRequest request) {
       return bankService.transfer(request); // Synchronous
   }
   ```

---

**استخدم Asynchronous إذا:**
1. **Long Running Operations**
   ```java
   // Generate Report (يأخذ دقائق)
   @PostMapping("/reports/generate")
   public ReportResponse generateReport(@RequestBody ReportRequest request) {
       String reportId = UUID.randomUUID().toString();

       // Send to Queue
       rabbitTemplate.convertAndSend("report.queue",
           new GenerateReportMessage(reportId, request));

       return new ReportResponse(reportId, "PROCESSING");
   }
   ```

2. **Event-Driven Architecture**
   ```java
   // Order Created → Send Email, Update Inventory, etc.
   public Order createOrder(OrderRequest request) {
       Order order = orderRepository.save(new Order(request));

       // Publish Event
       eventPublisher.publish(new OrderCreatedEvent(order));

       return order;
   }
   ```

3. **Background Processing**
   ```java
   // Send Email (لا تحتاج ترد للـ User فوراً)
   public void sendWelcomeEmail(User user) {
       emailQueue.send(new SendEmailMessage(user.getEmail(), "Welcome!"));
   }
   ```

4. **High Throughput Systems**
   ```java
   // IoT Sensor Data
   public void processSensorData(SensorReading reading) {
       kafkaTemplate.send("sensor-data", reading); // Async
   }
   ```

---

#### Hybrid Approach (الأفضل)

**الجمع بين الاثنين:**

```java
@Service
public class OrderService {

    @Autowired
    private RestTemplate restTemplate; // Sync

    @Autowired
    private RabbitTemplate rabbitTemplate; // Async

    @Transactional
    public OrderResponse createOrder(OrderRequest request) {
        // ① Create Order
        Order order = orderRepository.save(new Order(request));

        // ② Validate Payment (Synchronous - Critical)
        boolean paymentValid = restTemplate.getForObject(
            "http://PAYMENT-SERVICE/api/validate/" + request.getPaymentMethodId(),
            Boolean.class
        );

        if (!paymentValid) {
            throw new InvalidPaymentException();
        }

        // ③ Process Payment (Asynchronous - Long Operation)
        rabbitTemplate.convertAndSend("payment.exchange", "payment.process",
            new ProcessPaymentMessage(order.getId(), order.getAmount()));

        // ④ Send Notifications (Asynchronous - Not Critical)
        rabbitTemplate.convertAndSend("notification.exchange", "order.created",
            new OrderCreatedEvent(order));

        // ⑤ Update Inventory (Asynchronous - Can be eventual)
        rabbitTemplate.convertAndSend("inventory.exchange", "inventory.reserve",
            new ReserveInventoryMessage(order.getItems()));

        return OrderResponse.builder()
            .orderId(order.getId())
            .status("PENDING_PAYMENT")
            .message("Order created successfully")
            .build();
    }
}
```

---

#### Handling Asynchronous Responses

**1. Polling:**
```java
@GetMapping("/orders/{id}/status")
public OrderStatus getOrderStatus(@PathVariable Long id) {
    return orderService.getStatus(id);
}

// Client يسأل كل فترة
setInterval(() => {
    fetch('/api/orders/123/status')
        .then(data => {
            if (data.status === 'COMPLETED') {
                // Stop polling
            }
        });
}, 2000);
```

**2. WebSocket:**
```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws").withSockJS();
    }
}

// عند اكتمال Order
@Autowired
private SimpMessagingTemplate messagingTemplate;

public void notifyOrderComplete(Long orderId, String userId) {
    messagingTemplate.convertAndSendToUser(userId, "/queue/orders",
        new OrderCompletedNotification(orderId));
}
```

**3. Server-Sent Events (SSE):**
```java
@GetMapping(value = "/orders/{id}/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<OrderStatus> streamOrderStatus(@PathVariable Long id) {
    return orderStatusService.getStatusStream(id);
}
```

**4. Webhooks:**
```java
// Client يسجل Webhook URL
@PostMapping("/webhooks/register")
public void registerWebhook(@RequestBody WebhookRequest request) {
    webhookService.register(request.getUserId(), request.getUrl());
}

// عند اكتمال Order
public void notifyViaWebhook(Order order) {
    String webhookUrl = webhookService.getUrl(order.getUserId());
    restTemplate.postForEntity(webhookUrl, new OrderCompletedEvent(order), Void.class);
}
```

---

**Best Practices:**

1. **Use Sync for:**
   - User Authentication
   - Data Retrieval
   - Critical Validations

2. **Use Async for:**
   - Emails/Notifications
   - Report Generation
   - Data Sync between Services
   - Event Broadcasting

3. **Always:**
   - ✅ Handle Timeouts (Sync)
   - ✅ Implement Retries (Async)
   - ✅ Dead Letter Queues (Async)
   - ✅ Idempotency (Both)
   - ✅ Monitoring & Logging

---

### 12. اشرح REST و Feign Client

#### REST (Representational State Transfer)

**التعريف:**
- معيار معماري للـ Web APIs
- يستخدم HTTP Methods (GET, POST, PUT, DELETE)
- Stateless Communication

**HTTP Methods:**

| Method | Use Case | Idempotent | Safe |
|--------|----------|------------|------|
| GET | Read data | ✅ | ✅ |
| POST | Create data | ❌ | ❌ |
| PUT | Update/Replace | ✅ | ❌ |
| PATCH | Partial Update | ❌ | ❌ |
| DELETE | Delete data | ✅ | ❌ |

---

**مثال REST API:**

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    @Autowired
    private ProductService productService;

    // GET - Read
    @GetMapping
    public List<Product> getAllProducts() {
        return productService.findAll();
    }

    @GetMapping("/{id}")
    public ResponseEntity<Product> getProduct(@PathVariable Long id) {
        return productService.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    // POST - Create
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Product createProduct(@RequestBody @Valid ProductRequest request) {
        return productService.create(request);
    }

    // PUT - Update
    @PutMapping("/{id}")
    public Product updateProduct(@PathVariable Long id, @RequestBody ProductRequest request) {
        return productService.update(id, request);
    }

    // PATCH - Partial Update
    @PatchMapping("/{id}")
    public Product patchProduct(@PathVariable Long id, @RequestBody Map<String, Object> updates) {
        return productService.patch(id, updates);
    }

    // DELETE - Delete
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void deleteProduct(@PathVariable Long id) {
        productService.delete(id);
    }
}
```

---

#### طرق استدعاء REST APIs في Microservices

##### 1. RestTemplate (Traditional)

```java
@Configuration
public class RestTemplateConfig {

    @LoadBalanced // للـ Service Discovery
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

@Service
public class OrderService {

    @Autowired
    private RestTemplate restTemplate;

    // GET Request
    public Product getProduct(Long productId) {
        String url = "http://PRODUCT-SERVICE/api/products/" + productId;
        return restTemplate.getForObject(url, Product.class);
    }

    // GET with Response Entity
    public Product getProductWithHeaders(Long productId) {
        String url = "http://PRODUCT-SERVICE/api/products/" + productId;
        ResponseEntity<Product> response = restTemplate.getForEntity(url, Product.class);

        HttpHeaders headers = response.getHeaders();
        HttpStatus status = response.getStatusCode();

        return response.getBody();
    }

    // POST Request
    public Payment createPayment(PaymentRequest request) {
        String url = "http://PAYMENT-SERVICE/api/payments";
        return restTemplate.postForObject(url, request, Payment.class);
    }

    // POST with Headers
    public Payment createPaymentWithAuth(PaymentRequest request, String token) {
        String url = "http://PAYMENT-SERVICE/api/payments";

        HttpHeaders headers = new HttpHeaders();
        headers.set("Authorization", "Bearer " + token);
        headers.setContentType(MediaType.APPLICATION_JSON);

        HttpEntity<PaymentRequest> entity = new HttpEntity<>(request, headers);

        ResponseEntity<Payment> response = restTemplate.postForEntity(url, entity, Payment.class);
        return response.getBody();
    }

    // PUT Request
    public void updateProduct(Long id, ProductRequest request) {
        String url = "http://PRODUCT-SERVICE/api/products/" + id;
        restTemplate.put(url, request);
    }

    // DELETE Request
    public void deleteProduct(Long id) {
        String url = "http://PRODUCT-SERVICE/api/products/" + id;
        restTemplate.delete(url);
    }

    // Exchange (Generic)
    public <T> T exchange(String url, HttpMethod method, Object body, Class<T> responseType) {
        HttpEntity<?> entity = body != null ? new HttpEntity<>(body) : null;

        ResponseEntity<T> response = restTemplate.exchange(
            url,
            method,
            entity,
            responseType
        );

        return response.getBody();
    }
}
```

**مع Error Handling:**
```java
public Product getProductSafe(Long productId) {
    try {
        String url = "http://PRODUCT-SERVICE/api/products/" + productId;
        return restTemplate.getForObject(url, Product.class);

    } catch (HttpClientErrorException.NotFound e) {
        throw new ProductNotFoundException("Product not found: " + productId);

    } catch (HttpServerErrorException e) {
        throw new ServiceUnavailableException("Product service is down");

    } catch (ResourceAccessException e) {
        throw new ServiceTimeoutException("Request timeout");
    }
}
```

---

##### 2. WebClient (Reactive - Recommended)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

```java
@Configuration
public class WebClientConfig {

    @Bean
    @LoadBalanced
    public WebClient.Builder webClientBuilder() {
        return WebClient.builder()
            .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
            .defaultHeader(HttpHeaders.USER_AGENT, "Order-Service/1.0");
    }
}

@Service
public class OrderService {

    @Autowired
    private WebClient.Builder webClientBuilder;

    // GET - Block (Synchronous)
    public Product getProduct(Long productId) {
        return webClientBuilder.build()
            .get()
            .uri("http://PRODUCT-SERVICE/api/products/{id}", productId)
            .retrieve()
            .bodyToMono(Product.class)
            .block();  // يحول Async إلى Sync
    }

    // GET - Async (Reactive)
    public Mono<Product> getProductAsync(Long productId) {
        return webClientBuilder.build()
            .get()
            .uri("http://PRODUCT-SERVICE/api/products/{id}", productId)
            .retrieve()
            .bodyToMono(Product.class);
    }

    // GET List
    public Flux<Product> getAllProducts() {
        return webClientBuilder.build()
            .get()
            .uri("http://PRODUCT-SERVICE/api/products")
            .retrieve()
            .bodyToFlux(Product.class);
    }

    // POST
    public Mono<Payment> createPayment(PaymentRequest request) {
        return webClientBuilder.build()
            .post()
            .uri("http://PAYMENT-SERVICE/api/payments")
            .bodyValue(request)
            .retrieve()
            .bodyToMono(Payment.class);
    }

    // POST with Headers
    public Mono<Payment> createPaymentWithAuth(PaymentRequest request, String token) {
        return webClientBuilder.build()
            .post()
            .uri("http://PAYMENT-SERVICE/api/payments")
            .header(HttpHeaders.AUTHORIZATION, "Bearer " + token)
            .bodyValue(request)
            .retrieve()
            .bodyToMono(Payment.class);
    }

    // Error Handling
    public Mono<Product> getProductWithErrorHandling(Long productId) {
        return webClientBuilder.build()
            .get()
            .uri("http://PRODUCT-SERVICE/api/products/{id}", productId)
            .retrieve()
            .onStatus(HttpStatus::is4xxClientError, response -> {
                return Mono.error(new ProductNotFoundException("Product not found"));
            })
            .onStatus(HttpStatus::is5xxServerError, response -> {
                return Mono.error(new ServiceUnavailableException("Service is down"));
            })
            .bodyToMono(Product.class)
            .timeout(Duration.ofSeconds(3))
            .retry(3);
    }

    // Multiple Calls in Parallel
    public Mono<OrderDetails> getOrderDetails(Long orderId) {
        Mono<Order> orderMono = webClientBuilder.build()
            .get()
            .uri("http://ORDER-SERVICE/api/orders/{id}", orderId)
            .retrieve()
            .bodyToMono(Order.class);

        Mono<Payment> paymentMono = webClientBuilder.build()
            .get()
            .uri("http://PAYMENT-SERVICE/api/payments/order/{orderId}", orderId)
            .retrieve()
            .bodyToMono(Payment.class);

        Mono<User> userMono = webClientBuilder.build()
            .get()
            .uri("http://USER-SERVICE/api/users/{id}", orderId)
            .retrieve()
            .bodyToMono(User.class);

        // Combine all
        return Mono.zip(orderMono, paymentMono, userMono)
            .map(tuple -> new OrderDetails(
                tuple.getT1(),
                tuple.getT2(),
                tuple.getT3()
            ));
    }
}
```

---

##### 3. Feign Client (Declarative - الأسهل)

**Setup:**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableFeignClients  // تفعيل Feign
public class OrderServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}
```

**Feign Client Interface:**
```java
@FeignClient(
    name = "product-service",  // اسم الخدمة في Eureka
    path = "/api/products",    // Base path
    configuration = FeignConfig.class  // Custom config (optional)
)
public interface ProductClient {

    // GET - Single
    @GetMapping("/{id}")
    Product getProduct(@PathVariable("id") Long id);

    // GET - List
    @GetMapping
    List<Product> getAllProducts();

    // GET with Query Params
    @GetMapping("/search")
    List<Product> searchProducts(
        @RequestParam("name") String name,
        @RequestParam("category") String category
    );

    // POST
    @PostMapping
    Product createProduct(@RequestBody ProductRequest request);

    // PUT
    @PutMapping("/{id}")
    Product updateProduct(
        @PathVariable("id") Long id,
        @RequestBody ProductRequest request
    );

    // DELETE
    @DeleteMapping("/{id}")
    void deleteProduct(@PathVariable("id") Long id);

    // With Headers
    @GetMapping("/{id}")
    Product getProductWithAuth(
        @PathVariable("id") Long id,
        @RequestHeader("Authorization") String token
    );
}
```

**استخدام Feign Client:**
```java
@Service
public class OrderService {

    @Autowired
    private ProductClient productClient;  // Autowire Interface

    @Autowired
    private PaymentClient paymentClient;

    public Order createOrder(OrderRequest request) {
        // ① Get Product Info
        Product product = productClient.getProduct(request.getProductId());

        // ② Check Stock
        if (product.getStock() < request.getQuantity()) {
            throw new OutOfStockException();
        }

        // ③ Create Order
        Order order = orderRepository.save(new Order(request, product));

        // ④ Process Payment
        PaymentRequest paymentRequest = new PaymentRequest(
            order.getId(),
            product.getPrice() * request.getQuantity()
        );
        Payment payment = paymentClient.createPayment(paymentRequest);

        // ⑤ Update Order
        order.setPaymentId(payment.getId());
        order.setStatus("PAID");

        return orderRepository.save(order);
    }
}
```

---

#### Feign Configuration

**Custom Configuration:**
```java
@Configuration
public class FeignConfig {

    // Timeouts
    @Bean
    public Request.Options requestOptions() {
        return new Request.Options(
            5000,  // connect timeout (ms)
            10000  // read timeout (ms)
        );
    }

    // Logging
    @Bean
    public Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;  // NONE, BASIC, HEADERS, FULL
    }

    // Error Decoder
    @Bean
    public ErrorDecoder errorDecoder() {
        return new CustomErrorDecoder();
    }

    // Request Interceptor (Add Headers)
    @Bean
    public RequestInterceptor requestInterceptor() {
        return requestTemplate -> {
            requestTemplate.header("X-Service", "Order-Service");
            requestTemplate.header("X-Request-Time", String.valueOf(System.currentTimeMillis()));
        };
    }

    // Retryer
    @Bean
    public Retryer retryer() {
        return new Retryer.Default(
            100,   // period
            1000,  // maxPeriod
            3      // maxAttempts
        );
    }
}
```

**Custom Error Decoder:**
```java
public class CustomErrorDecoder implements ErrorDecoder {

    private final ErrorDecoder defaultDecoder = new Default();

    @Override
    public Exception decode(String methodKey, Response response) {
        switch (response.status()) {
            case 400:
                return new BadRequestException("Bad request to " + methodKey);
            case 404:
                return new NotFoundException("Resource not found");
            case 503:
                return new ServiceUnavailableException("Service is down");
            default:
                return defaultDecoder.decode(methodKey, response);
        }
    }
}
```

---

#### Feign مع Resilience4j (Circuit Breaker)

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
</dependency>
```

```java
@FeignClient(
    name = "product-service",
    fallback = ProductClientFallback.class  // Fallback class
)
public interface ProductClient {
    @GetMapping("/api/products/{id}")
    Product getProduct(@PathVariable Long id);
}

@Component
public class ProductClientFallback implements ProductClient {

    @Override
    public Product getProduct(Long id) {
        // Fallback response
        return Product.builder()
            .id(id)
            .name("Product Unavailable")
            .price(0.0)
            .build();
    }
}
```

**أو FallbackFactory للحصول على Exception:**
```java
@FeignClient(
    name = "product-service",
    fallbackFactory = ProductClientFallbackFactory.class
)
public interface ProductClient {
    @GetMapping("/api/products/{id}")
    Product getProduct(@PathVariable Long id);
}

@Component
public class ProductClientFallbackFactory implements FallbackFactory<ProductClient> {

    @Override
    public ProductClient create(Throwable cause) {
        return new ProductClient() {
            @Override
            public Product getProduct(Long id) {
                log.error("Error calling product service", cause);
                return new Product(); // default product
            }
        };
    }
}
```

---

#### المقارنة بين الطرق

| Feature | RestTemplate | WebClient | Feign Client |
|---------|--------------|-----------|--------------|
| **Model** | Synchronous | Reactive | Synchronous |
| **Code Style** | Imperative | Functional | Declarative |
| **Ease of Use** | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Performance** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Load Balancing** | ✅ @LoadBalanced | ✅ @LoadBalanced | ✅ Built-in |
| **Circuit Breaker** | Manual | Manual | ✅ Easy Integration |
| **Retry** | Manual | ✅ Built-in | ✅ Built-in |
| **Future** | Deprecated | ✅ Recommended | ✅ Good |

---

**Best Practice - استخدم:**

1. **Feign Client** للـ:
   - Service-to-Service calls
   - Clean & Readable code
   - Built-in resilience

2. **WebClient** للـ:
   - Reactive applications
   - High performance requirements
   - Parallel calls

3. **RestTemplate** - لا تستخدمه (Deprecated)

---

**مثال كامل - Feign Client:**

```yaml
# application.yml
feign:
  client:
    config:
      default:  # لكل Clients
        connectTimeout: 5000
        readTimeout: 10000
        loggerLevel: basic
      product-service:  # لـ Client معين
        connectTimeout: 3000
        readTimeout: 5000
  circuitbreaker:
    enabled: true

resilience4j:
  circuitbreaker:
    instances:
      product-service:
        slidingWindowSize: 10
        failureRateThreshold: 50
        waitDurationInOpenState: 10000
```

```java
// Multiple Feign Clients
@Service
@RequiredArgsConstructor
public class OrderService {

    private final ProductClient productClient;
    private final PaymentClient paymentClient;
    private final InventoryClient inventoryClient;
    private final NotificationClient notificationClient;

    @Transactional
    public OrderResponse createOrder(OrderRequest request) {
        // ① Validate Product
        Product product = productClient.getProduct(request.getProductId());

        // ② Check Inventory
        boolean available = inventoryClient.checkAvailability(
            request.getProductId(),
            request.getQuantity()
        );

        if (!available) {
            throw new OutOfStockException();
        }

        // ③ Create Order
        Order order = orderRepository.save(new Order(request, product));

        // ④ Process Payment
        Payment payment = paymentClient.createPayment(
            new PaymentRequest(order.getId(), order.getTotalAmount())
        );

        // ⑤ Update Inventory
        inventoryClient.reduceStock(request.getProductId(), request.getQuantity());

        // ⑥ Send Notification
        notificationClient.sendOrderConfirmation(order.getUserId(), order.getId());

        return new OrderResponse(order, payment);
    }
}
```

---

### 13. ما هو Message Broker (RabbitMQ, Kafka)؟

**الشرح:**

**Message Broker:**
- وسيط لنقل الرسائل بين Services
- يحفظ Messages في Queue أو Topic
- يضمن توصيل الرسائل حتى لو Service مطفي

```
Producer → Message Broker → Consumer
Service A     (Queue)      Service B
```

---

#### RabbitMQ

**التعريف:**
- Message Broker تقليدي
- يستخدم AMQP Protocol
- مناسب للـ Task Queues

**المكونات:**
```
Publisher → Exchange → Queue → Consumer
            (Routing)
```

**Setup:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

```yaml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
```

**Configuration:**
```java
@Configuration
public class RabbitMQConfig {

    // ① Queue
    @Bean
    public Queue orderQueue() {
        return new Queue("order.queue", true); // durable
    }

    @Bean
    public Queue paymentQueue() {
        return new Queue("payment.queue", true);
    }

    // ② Exchange
    @Bean
    public DirectExchange orderExchange() {
        return new DirectExchange("order.exchange");
    }

    // ③ Binding
    @Bean
    public Binding orderBinding(Queue orderQueue, DirectExchange orderExchange) {
        return BindingBuilder
            .bind(orderQueue)
            .to(orderExchange)
            .with("order.created");
    }

    // Topic Exchange (للـ Pattern Matching)
    @Bean
    public TopicExchange notificationExchange() {
        return new TopicExchange("notification.exchange");
    }

    @Bean
    public Queue emailQueue() {
        return new Queue("email.queue");
    }

    @Bean
    public Queue smsQueue() {
        return new Queue("sms.queue");
    }

    @Bean
    public Binding emailBinding() {
        return BindingBuilder
            .bind(emailQueue())
            .to(notificationExchange())
            .with("notification.email.*");
    }

    @Bean
    public Binding smsBinding() {
        return BindingBuilder
            .bind(smsQueue())
            .to(notificationExchange())
            .with("notification.sms.*");
    }

    // Fanout Exchange (Broadcast)
    @Bean
    public FanoutExchange broadcastExchange() {
        return new FanoutExchange("broadcast.exchange");
    }

    // Message Converter
    @Bean
    public MessageConverter jsonMessageConverter() {
        return new Jackson2JsonMessageConverter();
    }
}
```

**Publisher (Producer):**
```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final RabbitTemplate rabbitTemplate;
    private final OrderRepository orderRepository;

    public Order createOrder(OrderRequest request) {
        // Create Order
        Order order = orderRepository.save(new Order(request));

        // Send Message
        OrderCreatedEvent event = OrderCreatedEvent.builder()
            .orderId(order.getId())
            .userId(order.getUserId())
            .amount(order.getAmount())
            .timestamp(LocalDateTime.now())
            .build();

        rabbitTemplate.convertAndSend(
            "order.exchange",
            "order.created",
            event
        );

        return order;
    }

    // مع Custom Headers
    public void publishWithHeaders(Order order) {
        MessageProperties properties = new MessageProperties();
        properties.setHeader("priority", "high");
        properties.setHeader("source", "order-service");

        Message message = rabbitTemplate.getMessageConverter()
            .toMessage(new OrderCreatedEvent(order), properties);

        rabbitTemplate.send("order.exchange", "order.created", message);
    }
}
```

**Consumer (Listener):**
```java
@Component
@Slf4j
public class OrderListener {

    @Autowired
    private PaymentService paymentService;

    // Basic Listener
    @RabbitListener(queues = "order.queue")
    public void handleOrderCreated(OrderCreatedEvent event) {
        log.info("Received order: {}", event.getOrderId());

        // معالجة الطلب
        paymentService.processPayment(event);
    }

    // مع Manual Acknowledgment
    @RabbitListener(queues = "payment.queue", ackMode = "MANUAL")
    public void handlePayment(PaymentMessage message, Channel channel,
                              @Header(AmqpHeaders.DELIVERY_TAG) long tag) {
        try {
            paymentService.process(message);

            // Success - Acknowledge
            channel.basicAck(tag, false);

        } catch (RecoverableException e) {
            // Reject & Requeue
            channel.basicNack(tag, false, true);

        } catch (Exception e) {
            // Reject & Don't Requeue
            channel.basicNack(tag, false, false);
        }
    }

    // مع Error Handler
    @RabbitListener(
        queues = "order.queue",
        errorHandler = "customErrorHandler"
    )
    public void handleOrderWithErrorHandler(OrderCreatedEvent event) {
        if (event.getAmount() < 0) {
            throw new InvalidOrderException("Invalid amount");
        }

        processOrder(event);
    }
}

@Component
public class CustomErrorHandler implements RabbitListenerErrorHandler {

    @Override
    public Object handleError(Message amqpMessage, org.springframework.messaging.Message<?> message,
                             ListenerExecutionFailedException exception) {
        log.error("Error processing message", exception);

        // Send to Dead Letter Queue or Log
        return null;
    }
}
```

**Dead Letter Queue:**
```java
@Configuration
public class DeadLetterConfig {

    @Bean
    public Queue mainQueue() {
        return QueueBuilder.durable("main.queue")
            .withArgument("x-dead-letter-exchange", "dlx.exchange")
            .withArgument("x-dead-letter-routing-key", "dlq")
            .withArgument("x-message-ttl", 60000) // 1 minute
            .build();
    }

    @Bean
    public Queue deadLetterQueue() {
        return new Queue("dead.letter.queue");
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
            .with("dlq");
    }
}
```

---

#### Apache Kafka

**التعريف:**
- Distributed Event Streaming Platform
- High Throughput & Scalability
- مناسب للـ Event Sourcing & Real-time Processing

**المكونات:**
```
Producer → Topic (Partitions) → Consumer Group
                                    ↓
                              Multiple Consumers
```

**Setup:**
```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
    consumer:
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      group-id: order-service-group
      auto-offset-reset: earliest
      properties:
        spring.json.trusted.packages: "*"
```

**Producer:**
```java
@Service
@RequiredArgsConstructor
@Slf4j
public class OrderProducer {

    private final KafkaTemplate<String, OrderEvent> kafkaTemplate;

    public void publishOrderCreated(Order order) {
        OrderEvent event = new OrderEvent(order);

        // Send
        CompletableFuture<SendResult<String, OrderEvent>> future =
            kafkaTemplate.send("order-events", order.getId().toString(), event);

        // Callback
        future.whenComplete((result, ex) -> {
            if (ex == null) {
                log.info("Sent message=[{}] with offset=[{}]",
                    event, result.getRecordMetadata().offset());
            } else {
                log.error("Unable to send message=[{}]", event, ex);
            }
        });
    }

    // مع Headers
    public void publishWithHeaders(Order order) {
        OrderEvent event = new OrderEvent(order);

        ProducerRecord<String, OrderEvent> record = new ProducerRecord<>(
            "order-events",
            order.getId().toString(),
            event
        );

        record.headers().add("source", "order-service".getBytes());
        record.headers().add("version", "1.0".getBytes());

        kafkaTemplate.send(record);
    }

    // مع Partitioning
    public void publishToPartition(Order order, int partition) {
        kafkaTemplate.send("order-events", partition,
            order.getId().toString(), new OrderEvent(order));
    }
}
```

**Consumer:**
```java
@Component
@Slf4j
public class OrderConsumer {

    @KafkaListener(
        topics = "order-events",
        groupId = "payment-service-group"
    )
    public void handleOrderEvent(OrderEvent event) {
        log.info("Received: {}", event);
        processOrder(event);
    }

    // مع Manual Offset Commit
    @KafkaListener(
        topics = "order-events",
        groupId = "inventory-service-group",
        containerFactory = "kafkaListenerContainerFactory"
    )
    public void handleWithManualCommit(
            OrderEvent event,
            Acknowledgment acknowledgment) {

        try {
            processOrder(event);
            acknowledgment.acknowledge(); // Commit offset
        } catch (Exception e) {
            log.error("Error processing event", e);
            // لا تعمل acknowledge = لن يتم commit
        }
    }

    // Multiple Topics
    @KafkaListener(
        topics = {"order-events", "payment-events"},
        groupId = "notification-service-group"
    )
    public void handleMultipleTopics(ConsumerRecord<String, String> record) {
        String topic = record.topic();
        String message = record.value();

        log.info("Received from {}: {}", topic, message);
    }

    // مع Headers
    @KafkaListener(topics = "order-events")
    public void handleWithHeaders(
            OrderEvent event,
            @Header(KafkaHeaders.RECEIVED_TOPIC) String topic,
            @Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition,
            @Header(KafkaHeaders.OFFSET) long offset,
            @Header("source") String source) {

        log.info("Received from topic={}, partition={}, offset={}, source={}",
            topic, partition, offset, source);

        processOrder(event);
    }
}
```

**Configuration:**
```java
@Configuration
public class KafkaConfig {

    @Bean
    public NewTopic orderTopic() {
        return TopicBuilder.name("order-events")
            .partitions(3)
            .replicas(1)
            .config("retention.ms", "604800000") // 7 days
            .build();
    }

    @Bean
    public ConsumerFactory<String, OrderEvent> consumerFactory() {
        Map<String, Object> config = new HashMap<>();
        config.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        config.put(ConsumerConfig.GROUP_ID_CONFIG, "order-service-group");
        config.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        config.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class);
        config.put(JsonDeserializer.TRUSTED_PACKAGES, "*");
        config.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false); // Manual commit

        return new DefaultKafkaConsumerFactory<>(config);
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, OrderEvent> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, OrderEvent> factory =
            new ConcurrentKafkaListenerContainerFactory<>();

        factory.setConsumerFactory(consumerFactory());
        factory.setConcurrency(3); // 3 consumers
        factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.MANUAL);

        // Error Handler
        factory.setCommonErrorHandler(new DefaultErrorHandler(
            new FixedBackOff(1000L, 3L) // 3 retries with 1 second delay
        ));

        return factory;
    }

    // Retry Topic
    @Bean
    public RetryTopicConfiguration retryTopicConfiguration(KafkaTemplate<String, OrderEvent> template) {
        return RetryTopicConfigurationBuilder
            .newInstance()
            .maxAttempts(4)
            .backoff(new FixedBackOffPolicy() {{
                setBackOffPeriod(3000L);
            }})
            .create(template);
    }
}
```

---

#### RabbitMQ vs Kafka

| Feature | RabbitMQ | Kafka |
|---------|----------|-------|
| **Type** | Message Broker | Event Streaming |
| **Protocol** | AMQP | Custom |
| **Throughput** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Latency** | Low | Very Low |
| **Message Retention** | حتى Consumption | Configurable (days) |
| **Message Ordering** | Per Queue | Per Partition |
| **Replay Messages** | ❌ | ✅ |
| **Use Case** | Task Queues | Event Sourcing, Logs |
| **Complexity** | ⭐⭐⭐ | ⭐⭐⭐⭐ |

**استخدم RabbitMQ إذا:**
- Task Queues (Background Jobs)
- Complex Routing
- Priority Queues
- Request/Reply Pattern

**استخدم Kafka إذا:**
- Event Sourcing
- Real-time Analytics
- High Throughput
- Message Replay needed
- Distributed Logs

---

### 14. متى تستخدم REST ومتى تستخدم Messaging؟

#### REST (Synchronous)

**استخدم REST إذا:**

1. **تحتاج رد فوري**
   ```java
   // Get User Profile
   @GetMapping("/users/{id}")
   public User getUser(@PathVariable Long id) {
       return userService.findById(id); // REST
   }
   ```

2. **Read Operations**
   ```java
   // List Products
   @GetMapping("/products")
   public List<Product> getProducts() {
       return productService.findAll(); // REST
   }
   ```

3. **Validation**
   ```java
   // Check Email Availability
   @GetMapping("/users/check-email")
   public boolean checkEmail(@RequestParam String email) {
       return userService.isEmailAvailable(email); // REST
   }
   ```

4. **Simple CRUD**
   ```java
   // Update Profile
   @PutMapping("/users/{id}")
   public User updateUser(@PathVariable Long id, @RequestBody UserRequest request) {
       return userService.update(id, request); // REST
   }
   ```

---

#### Messaging (Asynchronous)

**استخدم Messaging إذا:**

1. **Long Running Operations**
   ```java
   // Generate Report (يأخذ وقت طويل)
   @PostMapping("/reports/generate")
   public ReportResponse generateReport(@RequestBody ReportRequest request) {
       String reportId = UUID.randomUUID().toString();

       // Send to Queue
       rabbitTemplate.convertAndSend("report.queue",
           new GenerateReportMessage(reportId, request));

       return new ReportResponse(reportId, "PROCESSING");
   }
   ```

2. **Event Broadcasting**
   ```java
   // Order Created → Multiple Actions
   public Order createOrder(OrderRequest request) {
       Order order = orderRepository.save(new Order(request));

       // Publish Event
       kafkaTemplate.send("order-events", new OrderCreatedEvent(order));

       // Multiple Consumers:
       // - Send Email
       // - Update Inventory
       // - Send SMS
       // - Update Analytics

       return order;
   }
   ```

3. **Decoupling**
   ```java
   // User Registered → Welcome Email (لا تريد Order Service يعتمد على Email Service)
   public void registerUser(UserRequest request) {
       User user = userRepository.save(new User(request));

       // Publish Event
       rabbitTemplate.convertAndSend("user.exchange", "user.registered",
           new UserRegisteredEvent(user));

       // Email Service يستمع ويرسل Email
   }
   ```

4. **Retry & Reliability**
   ```java
   // Process Payment (لو فشل، يعيد المحاولة)
   public void processPayment(Order order) {
       rabbitTemplate.convertAndSend("payment.queue",
           new ProcessPaymentMessage(order));

       // لو Payment Service مطفي، الـ Message يتحفظ في Queue
       // لما يشتغل، يعالج Messages المتبقية
   }
   ```

5. **High Throughput**
   ```java
   // IoT Sensor Data
   public void processSensorData(SensorReading reading) {
       kafkaTemplate.send("sensor-data", reading);

       // Millions of messages per second
   }
   ```

---

#### Decision Matrix

| Scenario | Use | Why |
|----------|-----|-----|
| Get User Profile | REST | Immediate response needed |
| Login | REST | Must validate credentials now |
| Search Products | REST | Real-time results expected |
| Create Order | REST + Messaging | REST لـ validation, Messaging للـ processing |
| Send Email | Messaging | Can be done later, not blocking |
| Generate Report | Messaging | Takes minutes |
| Update Inventory | Messaging | Multiple services need to know |
| Payment Processing | Messaging | Needs retry mechanism |
| Real-time Chat | WebSocket | Bi-directional communication |
| File Upload | REST | User waits for confirmation |
| Analytics Events | Messaging (Kafka) | High volume, non-critical |

---

#### Hybrid Approach (الأكثر شيوعاً)

```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final ProductClient productClient; // REST
    private final RabbitTemplate rabbitTemplate; // Messaging

    @Transactional
    public OrderResponse createOrder(OrderRequest request) {
        // ① Validate Product (REST - Immediate)
        Product product = productClient.getProduct(request.getProductId());

        if (product == null) {
            throw new ProductNotFoundException();
        }

        // ② Check Stock (REST - Immediate)
        boolean available = productClient.checkStock(
            request.getProductId(),
            request.getQuantity()
        );

        if (!available) {
            throw new OutOfStockException();
        }

        // ③ Create Order
        Order order = orderRepository.save(new Order(request, product));

        // ④ Process Payment (Messaging - Background)
        rabbitTemplate.convertAndSend("payment.exchange", "payment.process",
            new ProcessPaymentMessage(order));

        // ⑤ Send Notification (Messaging - Background)
        rabbitTemplate.convertAndSend("notification.exchange", "order.created",
            new OrderCreatedEvent(order));

        // ⑥ Update Analytics (Messaging - Background)
        kafkaTemplate.send("analytics-events", new OrderAnalyticsEvent(order));

        return OrderResponse.builder()
            .orderId(order.getId())
            .status("PENDING_PAYMENT")
            .message("Order created successfully")
            .build();
    }
}
```

---

## Configuration Management

### 15. ما هو Externalized Configuration؟

**الشرح:**

**Externalized Configuration:**
- فصل الإعدادات عن الكود
- تغيير Configuration بدون إعادة Build
- إعدادات مختلفة لكل بيئة (dev, staging, production)

**المشكلة:**
```java
// ❌ Hard-coded
public class PaymentService {
    private String apiKey = "sk_test_12345"; // في الكود!
    private String baseUrl = "http://localhost:8080";
}
```

**الحل:**
```yaml
# application.yml
payment:
  api-key: ${PAYMENT_API_KEY}
  base-url: ${PAYMENT_BASE_URL}
```

```java
// ✅ Externalized
@Service
public class PaymentService {

    @Value("${payment.api-key}")
    private String apiKey;

    @Value("${payment.base-url}")
    private String baseUrl;
}
```

---

#### Levels of Configuration

**1. Application Level (application.yml)**
```yaml
server:
  port: 8080

spring:
  application:
    name: order-service

  datasource:
    url: jdbc:mysql://localhost:3306/orders
    username: root
    password: password
```

**2. Profile-Specific (application-{profile}.yml)**
```yaml
# application-dev.yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/orders_dev
logging:
  level:
    root: DEBUG

# application-prod.yml
spring:
  datasource:
    url: jdbc:mysql://prod-db:3306/orders
logging:
  level:
    root: WARN
```

**Activation:**
```yaml
# application.yml
spring:
  profiles:
    active: dev
```

Or:
```bash
java -jar app.jar --spring.profiles.active=prod
```

**3. Environment Variables**
```yaml
spring:
  datasource:
    url: ${DB_URL:jdbc:mysql://localhost:3306/orders}
    username: ${DB_USERNAME:root}
    password: ${DB_PASSWORD}
```

**4. Command Line Arguments**
```bash
java -jar app.jar --server.port=9090 --spring.profiles.active=prod
```

**5. Config Server (Centralized)**
```
Config Server → Git Repository → All Services
```

---

#### Configuration Priority (من الأعلى للأقل)

```
1. Command Line Arguments
2. Environment Variables
3. application-{profile}.yml
4. application.yml
5. Default Values
```

---

#### Configuration Properties Class

```java
@ConfigurationProperties(prefix = "payment")
@Component
@Data
public class PaymentProperties {

    private String apiKey;
    private String baseUrl;
    private int timeout = 5000;
    private Retry retry = new Retry();

    @Data
    public static class Retry {
        private int maxAttempts = 3;
        private long delay = 1000;
    }
}
```

```yaml
payment:
  api-key: sk_test_12345
  base-url: https://api.payment.com
  timeout: 10000
  retry:
    max-attempts: 5
    delay: 2000
```

```java
@Service
@RequiredArgsConstructor
public class PaymentService {

    private final PaymentProperties properties;

    public void processPayment() {
        String apiKey = properties.getApiKey();
        int maxAttempts = properties.getRetry().getMaxAttempts();
    }
}
```

---

### 16. اشرح Spring Cloud Config Server

**الشرح:**

**Spring Cloud Config Server:**
- مركز لإدارة Configuration لجميع Services
- يخزن Configuration في Git Repository
- تحديث Configuration بدون Restart

```
┌────────────────────────────────────────┐
│       Git Repository                   │
│  - order-service-dev.yml               │
│  - order-service-prod.yml              │
│  - payment-service-dev.yml             │
│  - application.yml (Shared)            │
└────────────────────────────────────────┘
                ↓
┌────────────────────────────────────────┐
│     Spring Cloud Config Server         │
│         (Port 8888)                    │
└────────────────────────────────────────┘
         ↓              ↓
┌─────────────┐  ┌─────────────┐
│   Order     │  │  Payment    │
│  Service    │  │  Service    │
└─────────────┘  └─────────────┘
```

---

#### Setup

##### 1. Config Server

**Dependencies:**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

**Application Class:**
```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

**application.yml:**
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
          uri: https://github.com/your-org/config-repo
          default-label: main
          search-paths:
            - 'services/{application}'
            - 'shared'
          username: ${GIT_USERNAME}
          password: ${GIT_PASSWORD}

        # أو Local File System
        # native:
        #   search-locations: classpath:/config,file:./config

# Eureka Client (Optional)
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

**Git Repository Structure:**
```
config-repo/
├── application.yml              # Shared across all services
├── application-dev.yml
├── application-prod.yml
├── services/
│   ├── order-service/
│   │   ├── order-service.yml
│   │   ├── order-service-dev.yml
│   │   └── order-service-prod.yml
│   ├── payment-service/
│   │   ├── payment-service.yml
│   │   └── payment-service-prod.yml
```

**application.yml (Shared):**
```yaml
# في Git Repo
management:
  endpoints:
    web:
      exposure:
        include: '*'

logging:
  pattern:
    console: '%d{yyyy-MM-dd HH:mm:ss} - %msg%n'
```

**order-service-dev.yml:**
```yaml
# في Git Repo
server:
  port: 8081

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/orders_dev
    username: root
    password: dev_password

  rabbitmq:
    host: localhost
    port: 5672

logging:
  level:
    com.example: DEBUG
```

**order-service-prod.yml:**
```yaml
# في Git Repo
server:
  port: 8081

spring:
  datasource:
    url: jdbc:mysql://prod-db.example.com:3306/orders
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}

  rabbitmq:
    host: rabbitmq-prod.example.com
    port: 5672
    username: ${RABBITMQ_USERNAME}
    password: ${RABBITMQ_PASSWORD}

logging:
  level:
    root: WARN
    com.example: INFO
```

---

##### 2. Config Client (Services)

**Dependencies:**
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

**application.yml:**
```yaml
spring:
  application:
    name: order-service  # مهم! يطابق اسم الملف في Git

  config:
    import: "optional:configserver:http://localhost:8888"

  # Or using spring.cloud.config (Old way)
  # cloud:
  #   config:
  #     uri: http://localhost:8888
  #     fail-fast: true
  #     retry:
  #       max-attempts: 6
  #       initial-interval: 1000

  profiles:
    active: dev

management:
  endpoints:
    web:
      exposure:
        include: refresh
```

**Using Configuration:**
```java
@RestController
@RefreshScope  // Important for refresh
public class ConfigController {

    @Value("${custom.property}")
    private String customProperty;

    @GetMapping("/config")
    public String getConfig() {
        return customProperty;
    }
}
```

---

#### Refresh Configuration (بدون Restart)

**1. Actuator Refresh Endpoint:**
```bash
# تحديث Git Repo
git commit -m "Update config"
git push

# Refresh Service
curl -X POST http://localhost:8081/actuator/refresh
```

**2. Spring Cloud Bus (Auto Refresh لكل Services):**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

```yaml
spring:
  rabbitmq:
    host: localhost
    port: 5672

management:
  endpoints:
    web:
      exposure:
        include: busrefresh
```

```bash
# Refresh All Services مرة واحدة
curl -X POST http://localhost:8888/actuator/busrefresh
```

**Flow:**
```
① Update Git
② POST /actuator/busrefresh
③ Config Server → RabbitMQ → All Services
④ All Services Refresh Configuration
```

---

#### Security

**Encrypt/Decrypt Secrets:**

**Setup:**
```yaml
# Config Server
encrypt:
  key: my-secret-encryption-key
```

**Encrypt:**
```bash
curl http://localhost:8888/encrypt -d "my-secret-password"
# Returns: {cipher}AQA3esr4t5y6u7i8o9p0...
```

**In Git:**
```yaml
spring:
  datasource:
    password: '{cipher}AQA3esr4t5y6u7i8o9p0...'
```

**Config Server يفك التشفير تلقائياً**

---

#### Testing Config Server

**Endpoints:**
```bash
# Get config for order-service in dev profile
http://localhost:8888/order-service/dev

# Get config for order-service in prod profile
http://localhost:8888/order-service/prod

# Get specific file
http://localhost:8888/order-service/dev/main

# Format: /{application}/{profile}/{label}
# application: service name
# profile: dev, prod, etc.
# label: git branch (default: main)
```

---

#### Best Practices

1. **Separate Sensitive Data**
   ```yaml
   # application.yml
   spring:
     datasource:
       url: jdbc:mysql://prod-db:3306/orders
       username: ${DB_USERNAME}  # From environment
       password: ${DB_PASSWORD}
   ```

2. **Use Profiles**
   ```
   - order-service-dev.yml
   - order-service-staging.yml
   - order-service-prod.yml
   ```

3. **Shared Configuration**
   ```yaml
   # application.yml (for all services)
   management:
     endpoints:
       web:
         exposure:
           include: health,info,metrics
   ```

4. **Versioning**
   ```bash
   # Use Git tags/branches
   http://localhost:8888/order-service/prod/v1.0.0
   ```

5. **Fail Fast**
   ```yaml
   spring:
     cloud:
       config:
         fail-fast: true  # فشل عند Startup لو Config Server مطفي
   ```

---

### 17. كيف تدير Configuration لبيئات مختلفة (dev, staging, prod)؟

#### الاستراتيجيات

##### 1. Spring Profiles (الأساسي)

**Structure:**
```
src/main/resources/
├── application.yml              # Default
├── application-dev.yml          # Development
├── application-staging.yml      # Staging
├── application-prod.yml         # Production
```

**application.yml (Shared):**
```yaml
spring:
  application:
    name: order-service

  jpa:
    show-sql: false
    hibernate:
      ddl-auto: validate

management:
  endpoints:
    web:
      exposure:
        include: health,info
```

**application-dev.yml:**
```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/orders_dev
    username: root
    password: password

  jpa:
    show-sql: true
    hibernate:
      ddl-auto: create-drop

logging:
  level:
    root: DEBUG
    com.example: TRACE

debug: true
```

**application-staging.yml:**
```yaml
spring:
  datasource:
    url: jdbc:mysql://staging-db:3306/orders
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}

logging:
  level:
    root: INFO
    com.example: DEBUG
```

**application-prod.yml:**
```yaml
spring:
  datasource:
    url: jdbc:mysql://prod-db.example.com:3306/orders
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5

logging:
  level:
    root: WARN
    com.example: INFO

  file:
    name: /var/log/order-service/app.log
    max-size: 10MB
    max-history: 30
```

**Activation:**
```bash
# Development
java -jar app.jar --spring.profiles.active=dev

# Staging
java -jar app.jar --spring.profiles.active=staging

# Production
java -jar app.jar --spring.profiles.active=prod
```

---

##### 2. Environment Variables

```yaml
# application.yml
spring:
  datasource:
    url: ${DB_URL:jdbc:mysql://localhost:3306/orders}
    username: ${DB_USERNAME:root}
    password: ${DB_PASSWORD}

  rabbitmq:
    host: ${RABBITMQ_HOST:localhost}
    port: ${RABBITMQ_PORT:5672}
    username: ${RABBITMQ_USERNAME:guest}
    password: ${RABBITMQ_PASSWORD:guest}

server:
  port: ${SERVER_PORT:8080}
```

**Docker Compose (Dev):**
```yaml
version: '3'
services:
  order-service:
    image: order-service:latest
    environment:
      - SPRING_PROFILES_ACTIVE=dev
      - DB_URL=jdbc:mysql://mysql:3306/orders_dev
      - DB_USERNAME=root
      - DB_PASSWORD=dev_password
      - RABBITMQ_HOST=rabbitmq
```

**Kubernetes (Prod):**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: order-service-config
data:
  SPRING_PROFILES_ACTIVE: "prod"
  DB_URL: "jdbc:mysql://prod-db:3306/orders"
  RABBITMQ_HOST: "rabbitmq-prod"

---

apiVersion: v1
kind: Secret
metadata:
  name: order-service-secrets
type: Opaque
data:
  DB_USERNAME: base64encoded
  DB_PASSWORD: base64encoded
  RABBITMQ_PASSWORD: base64encoded

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
spec:
  template:
    spec:
      containers:
        - name: order-service
          image: order-service:latest
          envFrom:
            - configMapRef:
                name: order-service-config
            - secretRef:
                name: order-service-secrets
```

---

##### 3. Config Server (Centralized)

**Git Repository:**
```
config-repo/
├── order-service/
│   ├── order-service-dev.yml
│   ├── order-service-staging.yml
│   ├── order-service-prod.yml
├── payment-service/
│   ├── payment-service-dev.yml
│   ├── payment-service-prod.yml
└── application.yml  # Shared
```

**Benefits:**
- ✅ Centralized management
- ✅ Version control (Git)
- ✅ Refresh without restart
- ✅ Environment-specific secrets
- ✅ Audit trail

---

##### 4. Profile-Specific Beans

```java
@Configuration
public class DataSourceConfig {

    @Bean
    @Profile("dev")
    public DataSource devDataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl("jdbc:h2:mem:testdb");
        dataSource.setMaximumPoolSize(5);
        return dataSource;
    }

    @Bean
    @Profile("prod")
    public DataSource prodDataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl(System.getenv("DB_URL"));
        dataSource.setMaximumPoolSize(20);
        dataSource.setConnectionTimeout(30000);
        return dataSource;
    }
}

@Component
@Profile("dev")
public class DevDataInitializer implements CommandLineRunner {

    @Override
    public void run(String... args) {
        // تعبئة بيانات تجريبية
        System.out.println("Initializing dev data...");
    }
}

@Component
@Profile("!dev")  // كل البيئات إلا dev
public class SecurityConfig {
    // Production security settings
}
```

---

##### 5. Property Matrix

| Property | Dev | Staging | Prod |
|----------|-----|---------|------|
| **Database** | H2/Local MySQL | Staging DB | Production DB |
| **Log Level** | DEBUG | INFO | WARN |
| **Show SQL** | true | false | false |
| **DDL Auto** | create-drop | validate | validate |
| **Pool Size** | 5 | 10 | 20 |
| **Cache** | false | true | true |
| **Metrics** | All | Selected | Selected |
| **Security** | Basic | Full | Full + Audit |

---

##### 6. Secrets Management

**Option 1: Environment Variables**
```bash
export DB_PASSWORD="prod_password"
export PAYMENT_API_KEY="sk_live_12345"
```

**Option 2: Vault (HashiCorp)**
```yaml
spring:
  cloud:
    vault:
      uri: https://vault.example.com
      token: ${VAULT_TOKEN}
      kv:
        enabled: true
        backend: secret
        default-context: order-service
```

```bash
# Store in Vault
vault kv put secret/order-service db.password=secret123

# Application reads from Vault automatically
```

**Option 3: AWS Secrets Manager**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-aws-secrets-manager-config</artifactId>
</dependency>
```

```yaml
aws:
  secretsmanager:
    enabled: true
    prefix: /secret/order-service
```

---

##### 7. Build-Time vs Runtime Configuration

**Build-Time (Maven Profiles):**
```xml
<profiles>
    <profile>
        <id>dev</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <spring.profiles.active>dev</spring.profiles.active>
        </properties>
    </profile>

    <profile>
        <id>prod</id>
        <properties>
            <spring.profiles.active>prod</spring.profiles.active>
        </properties>
    </profile>
</profiles>
```

```bash
# Build for production
mvn clean package -Pprod
```

**Runtime (Better):**
```bash
# Same JAR, different environments
java -jar app.jar --spring.profiles.active=dev
java -jar app.jar --spring.profiles.active=prod
```

---

##### 8. Feature Flags

```yaml
# application.yml
features:
  new-payment-gateway: false
  beta-ui: false

# application-prod.yml
features:
  new-payment-gateway: true
```

```java
@Service
public class PaymentService {

    @Value("${features.new-payment-gateway}")
    private boolean useNewGateway;

    public void processPayment(Payment payment) {
        if (useNewGateway) {
            newPaymentGateway.process(payment);
        } else {
            oldPaymentGateway.process(payment);
        }
    }
}
```

---

**Best Practices:**

1. ✅ **Never commit secrets** to Git
2. ✅ Use **environment variables** for sensitive data
3. ✅ **Default values** للـ Development
4. ✅ **Separate config** من الكود
5. ✅ Use **Config Server** للـ Production
6. ✅ **Version control** للـ Configuration
7. ✅ **Validate configuration** عند Startup
8. ✅ **Document** all properties

---

## Load Balancing & Resilience

### 18. ما هو Load Balancing وكيف يعمل في Microservices؟

**الشرح:**

**Load Balancing:**
- توزيع الطلبات على عدة instances من نفس الخدمة
- يحسن Performance و Availability
- يمنع Overload على instance واحد

```
         Load Balancer
              │
    ┌─────────┼─────────┐
    ↓         ↓         ↓
Instance1  Instance2  Instance3
```

---

#### أنواع Load Balancing

##### 1. Client-Side Load Balancing

**كيف يعمل:**
```
Client → Service Registry → Get Instance List
         ↓
    Choose Instance (Algorithm)
         ↓
    Direct Call to Instance
```

**Implementation مع Spring Cloud LoadBalancer:**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

```java
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
```

**استخدام:**
```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final RestTemplate restTemplate;

    public Payment getPayment(Long id) {
        // "PAYMENT-SERVICE" → Load Balancer يختار Instance
        String url = "http://PAYMENT-SERVICE/api/payments/" + id;
        return restTemplate.getForObject(url, Payment.class);
    }
}
```

---

##### 2. Server-Side Load Balancing

**أدوات:**
- Nginx
- HAProxy
- AWS ELB
- Kubernetes Service

**Nginx Example:**
```nginx
upstream payment-service {
    server 192.168.1.10:8081;
    server 192.168.1.11:8081;
    server 192.168.1.12:8081;
}

server {
    listen 80;

    location /api/payments/ {
        proxy_pass http://payment-service;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

#### Load Balancing Algorithms

##### 1. Round Robin (الافتراضي)

```
Request 1 → Instance 1
Request 2 → Instance 2
Request 3 → Instance 3
Request 4 → Instance 1  (يبدأ من جديد)
```

```java
@Configuration
public class LoadBalancerConfiguration {

    @Bean
    public ServiceInstanceListSupplier serviceInstanceListSupplier(
            ConfigurableApplicationContext context) {
        return ServiceInstanceListSupplier.builder()
            .withDiscoveryClient()
            .withHealthChecks()
            .build(context);
    }
}
```

##### 2. Random

```java
public class RandomLoadBalancerConfig {

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

##### 3. Weighted (بناءً على الأوزان)

```java
@Bean
public ServiceInstanceListSupplier weightedServiceInstanceListSupplier(
        ConfigurableApplicationContext context) {
    return ServiceInstanceListSupplier.builder()
        .withDiscoveryClient()
        .withWeighted()  // Weighted Load Balancing
        .build(context);
}
```

```yaml
# في Eureka metadata
eureka:
  instance:
    metadata-map:
      weight: 2  # Instance هذا يستقبل ضعف الطلبات
```

##### 4. Least Connections

```java
// Custom Implementation
public class LeastConnectionsLoadBalancer implements ReactorServiceInstanceLoadBalancer {

    private final Map<ServiceInstance, AtomicInteger> connections = new ConcurrentHashMap<>();

    @Override
    public Mono<Response<ServiceInstance>> choose(Request request) {
        return Mono.fromCallable(() -> {
            ServiceInstance instance = connections.entrySet().stream()
                .min(Comparator.comparingInt(e -> e.getValue().get()))
                .map(Map.Entry::getKey)
                .orElseThrow();

            connections.get(instance).incrementAndGet();

            return new DefaultResponse(instance);
        });
    }
}
```

##### 5. IP Hash / Sticky Sessions

```java
public class StickySessionLoadBalancer implements ReactorServiceInstanceLoadBalancer {

    @Override
    public Mono<Response<ServiceInstance>> choose(Request request) {
        return Mono.fromCallable(() -> {
            // استخدام IP أو Session ID
            String clientId = getClientId(request);
            int hash = clientId.hashCode();

            List<ServiceInstance> instances = getInstances();
            int index = Math.abs(hash % instances.size());

            return new DefaultResponse(instances.get(index));
        });
    }
}
```

---

#### Health Checks

```java
@RestController
public class HealthController {

    @GetMapping("/actuator/health")
    public ResponseEntity<HealthStatus> health() {
        // Check database, external services, etc.
        boolean healthy = checkDatabaseConnection()
                       && checkExternalServices();

        if (healthy) {
            return ResponseEntity.ok(new HealthStatus("UP"));
        } else {
            return ResponseEntity.status(HttpStatus.SERVICE_UNAVAILABLE)
                .body(new HealthStatus("DOWN"));
        }
    }
}
```

```yaml
# Eureka configuration
eureka:
  instance:
    health-check-url-path: /actuator/health
    lease-renewal-interval-in-seconds: 30
    lease-expiration-duration-in-seconds: 90
```

**Load Balancer يستبعد Instances الغير صحية:**
```java
@Bean
public ServiceInstanceListSupplier healthCheckServiceInstanceListSupplier(
        ConfigurableApplicationContext context) {
    return ServiceInstanceListSupplier.builder()
        .withDiscoveryClient()
        .withHealthChecks()  // يستبعد Instances DOWN
        .build(context);
}
```

---

#### مثال عملي كامل

```java
@Configuration
@LoadBalancerClient(
    name = "payment-service",
    configuration = PaymentServiceLoadBalancerConfig.class
)
public class LoadBalancerConfig {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

@Configuration
public class PaymentServiceLoadBalancerConfig {

    @Bean
    public ReactorLoadBalancer<ServiceInstance> paymentServiceLoadBalancer(
            Environment environment,
            LoadBalancerClientFactory loadBalancerClientFactory) {

        String name = environment.getProperty(LoadBalancerClientFactory.PROPERTY_NAME);

        return new RoundRobinLoadBalancer(
            loadBalancerClientFactory.getLazyProvider(name, ServiceInstanceListSupplier.class),
            name
        );
    }

    @Bean
    public ServiceInstanceListSupplier serviceInstanceListSupplier(
            ConfigurableApplicationContext context) {
        return ServiceInstanceListSupplier.builder()
            .withDiscoveryClient()
            .withHealthChecks()
            .withCaching()  // Cache instance list
            .build(context);
    }
}
```

---

### 19. اشرح Ribbon أو Spring Cloud LoadBalancer

#### Ribbon (قديم - Deprecated)

**ملاحظة:** Ribbon في Maintenance Mode، استخدم Spring Cloud LoadBalancer

```xml
<!-- لا تستخدمه في مشاريع جديدة -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```

---

#### Spring Cloud LoadBalancer (الحديث)

**Setup:**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

**Basic Configuration:**
```java
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
```

---

#### Custom Load Balancer

```java
// Custom Load Balancer Algorithm
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
        ServiceInstanceListSupplier supplier = serviceInstanceListSupplierProvider.getIfAvailable();

        return supplier.get(request).next()
            .map(instances -> {
                if (instances.isEmpty()) {
                    return new EmptyResponse();
                }

                // Custom Logic: Choose based on metadata
                ServiceInstance instance = instances.stream()
                    .filter(i -> "zone1".equals(i.getMetadata().get("zone")))
                    .findFirst()
                    .orElse(instances.get(0));

                return new DefaultResponse(instance);
            });
    }
}
```

**Configuration:**
```java
@Configuration
public class CustomLoadBalancerConfiguration {

    @Bean
    public ReactorLoadBalancer<ServiceInstance> customLoadBalancer(
            Environment environment,
            LoadBalancerClientFactory loadBalancerClientFactory) {

        String name = environment.getProperty(LoadBalancerClientFactory.PROPERTY_NAME);

        return new CustomLoadBalancer(
            loadBalancerClientFactory.getLazyProvider(name, ServiceInstanceListSupplier.class),
            name
        );
    }
}
```

**Apply to specific service:**
```java
@Configuration
@LoadBalancerClient(
    name = "payment-service",
    configuration = CustomLoadBalancerConfiguration.class
)
public class AppConfig {
    // ...
}
```

---

#### Zone-Aware Load Balancing

```java
@Configuration
public class ZonePreferenceLoadBalancerConfig {

    @Bean
    public ServiceInstanceListSupplier zonePreferenceServiceInstanceListSupplier(
            ConfigurableApplicationContext context) {
        return ServiceInstanceListSupplier.builder()
            .withDiscoveryClient()
            .withZonePreference()  // يفضل Instances في نفس الـ Zone
            .withCaching()
            .build(context);
    }
}
```

```yaml
# تحديد Zone
spring:
  cloud:
    loadbalancer:
      zone: zone1

eureka:
  instance:
    metadata-map:
      zone: zone1
```

---

#### Retry Configuration

```xml
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
</dependency>
```

```java
@Configuration
@EnableRetry
public class RetryConfig {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

```yaml
spring:
  cloud:
    loadbalancer:
      retry:
        enabled: true
        max-retries-on-same-service-instance: 1
        max-retries-on-next-service-instance: 2
        retry-on-all-operations: false
```

---

### 20. ما هو Circuit Breaker Pattern؟

**الشرح:**

**Circuit Breaker:**
- نمط لمنع Cascading Failures
- يوقف Calls للخدمة الفاشلة مؤقتاً
- يحمي النظام من التعطل الكامل

**المشكلة:**
```
Order Service → Payment Service (DOWN)
     ↓
Waiting... (Timeout 10s)
     ↓
500 requests × 10s = System Overload
```

**الحل:**
```
Circuit Breaker → يكتشف الفشل
                → يوقف Calls مؤقتاً
                → يرجع Fallback Response
```

---

#### Circuit Breaker States

```
┌─────────┐   Failures > Threshold    ┌──────┐
│ CLOSED  │─────────────────────────→ │ OPEN │
│ (Normal)│                            │(Block)│
└─────────┘                            └──────┘
     ↑                                     │
     │                                     │ After timeout
     │                                     ↓
     │                              ┌────────────┐
     └──────────────────────────────│ HALF_OPEN  │
       Calls succeed                │  (Testing) │
                                    └────────────┘
```

**1. CLOSED (طبيعي):**
- Requests تمر عادي
- يراقب Failures

**2. OPEN (مفتوح):**
- يمنع Requests
- يرجع Fallback فوراً

**3. HALF_OPEN (اختبار):**
- يسمح بعدد محدود من Requests
- لو نجحت → CLOSED
- لو فشلت → OPEN

---

#### Implementation مع Resilience4j

**Dependencies:**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

**Configuration:**
```yaml
resilience4j:
  circuitbreaker:
    configs:
      default:
        # عدد الـ Calls المراقبة
        slidingWindowSize: 10

        # نسبة الفشل المسموحة
        failureRateThreshold: 50

        # عدد الـ Slow Calls المسموحة
        slowCallRateThreshold: 50
        slowCallDurationThreshold: 2s

        # وقت الانتظار في OPEN state
        waitDurationInOpenState: 10s

        # عدد الـ Calls في HALF_OPEN
        permittedNumberOfCallsInHalfOpenState: 3

        # تسجيل Exceptions كـ Failures
        recordExceptions:
          - java.io.IOException
          - java.util.concurrent.TimeoutException

        # تجاهل Exceptions
        ignoreExceptions:
          - com.example.IgnoreException

    instances:
      paymentService:
        baseConfig: default
        slidingWindowSize: 20
        failureRateThreshold: 60

  timelimiter:
    configs:
      default:
        timeoutDuration: 3s
```

---

#### استخدام Circuit Breaker

**1. مع Annotation:**
```java
@Service
@Slf4j
public class OrderService {

    @Autowired
    private PaymentClient paymentClient;

    @CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
    @TimeLimiter(name = "paymentService")
    public CompletableFuture<Payment> getPayment(Long id) {
        log.info("Calling payment service for id: {}", id);
        return CompletableFuture.supplyAsync(() -> paymentClient.getPayment(id));
    }

    // Fallback Method (نفس الـ Signature + Exception parameter)
    public CompletableFuture<Payment> paymentFallback(Long id, Exception ex) {
        log.error("Payment service failed, returning fallback", ex);

        Payment fallbackPayment = Payment.builder()
            .id(id)
            .status("UNAVAILABLE")
            .message("Payment service is currently unavailable")
            .build();

        return CompletableFuture.completedFuture(fallbackPayment);
    }

    // Fallback for specific exception
    public CompletableFuture<Payment> paymentFallback(Long id, TimeoutException ex) {
        log.error("Payment service timeout", ex);
        return CompletableFuture.completedFuture(new Payment(id, "TIMEOUT"));
    }
}
```

**2. Programmatic API:**
```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final PaymentClient paymentClient;
    private final CircuitBreakerRegistry circuitBreakerRegistry;

    public Payment getPayment(Long id) {
        CircuitBreaker circuitBreaker = circuitBreakerRegistry.circuitBreaker("paymentService");

        return circuitBreaker.executeSupplier(() -> {
            return paymentClient.getPayment(id);
        });
    }

    // مع Fallback
    public Payment getPaymentWithFallback(Long id) {
        CircuitBreaker circuitBreaker = circuitBreakerRegistry.circuitBreaker("paymentService");

        return Try.ofSupplier(CircuitBreaker.decorateSupplier(circuitBreaker,
                () -> paymentClient.getPayment(id)))
            .recover(throwable -> {
                log.error("Circuit breaker fallback", throwable);
                return new Payment(id, "FALLBACK");
            })
            .get();
    }
}
```

**3. مع Feign Client:**
```java
@FeignClient(
    name = "payment-service",
    fallback = PaymentClientFallback.class
)
public interface PaymentClient {
    @GetMapping("/api/payments/{id}")
    Payment getPayment(@PathVariable Long id);
}

@Component
public class PaymentClientFallback implements PaymentClient {

    @Override
    public Payment getPayment(Long id) {
        return Payment.builder()
            .id(id)
            .status("UNAVAILABLE")
            .build();
    }
}
```

```yaml
feign:
  circuitbreaker:
    enabled: true
```

---

#### Circuit Breaker Events & Monitoring

**Event Listeners:**
```java
@Component
@Slf4j
public class CircuitBreakerEventListener {

    @Autowired
    public CircuitBreakerEventListener(CircuitBreakerRegistry registry) {
        registry.circuitBreaker("paymentService").getEventPublisher()
            .onStateTransition(event ->
                log.info("Circuit Breaker state changed: {}", event))
            .onFailureRateExceeded(event ->
                log.warn("Failure rate exceeded: {}", event.getFailureRate()))
            .onSlowCallRateExceeded(event ->
                log.warn("Slow call rate exceeded: {}", event.getSlowCallRate()))
            .onCallNotPermitted(event ->
                log.warn("Call not permitted"))
            .onError(event ->
                log.error("Circuit Breaker error: {}", event.getThrowable()));
    }
}
```

**Actuator Endpoints:**
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,circuitbreakers,circuitbreakerevents

  health:
    circuitbreakers:
      enabled: true

  endpoint:
    health:
      show-details: always
```

**Endpoints:**
```bash
# Circuit Breaker Status
GET /actuator/circuitbreakers

# Events
GET /actuator/circuitbreakerevents
GET /actuator/circuitbreakerevents/paymentService

# Health
GET /actuator/health
```

**Response Example:**
```json
{
  "circuitBreakers": {
    "paymentService": {
      "state": "CLOSED",
      "failureRate": "25.0%",
      "slowCallRate": "10.0%",
      "bufferedCalls": 10,
      "failedCalls": 2,
      "slowCalls": 1
    }
  }
}
```

---

#### Best Practices

**1. Timeout Configuration:**
```yaml
# Circuit Breaker Timeout < HTTP Client Timeout
resilience4j:
  timelimiter:
    configs:
      default:
        timeoutDuration: 3s  # أقل من

feign:
  client:
    config:
      default:
        connectTimeout: 2000
        readTimeout: 5000  # هذا
```

**2. Thread Pool Isolation (Optional):**
```yaml
resilience4j:
  thread-pool-bulkhead:
    configs:
      default:
        maxThreadPoolSize: 10
        coreThreadPoolSize: 5
        queueCapacity: 100
```

```java
@Bulkhead(name = "paymentService", type = Bulkhead.Type.THREADPOOL)
@CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
public CompletableFuture<Payment> getPayment(Long id) {
    return CompletableFuture.supplyAsync(() -> paymentClient.getPayment(id));
}
```

**3. Metrics:**
```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

```yaml
management:
  metrics:
    export:
      prometheus:
        enabled: true
    tags:
      application: ${spring.application.name}
```

---

### 21. اشرح Resilience4j أو Hystrix

#### Hystrix vs Resilience4j

| Feature | Hystrix | Resilience4j |
|---------|---------|--------------|
| **Status** | Maintenance Mode | Active Development |
| **Dependencies** | Heavy | Lightweight |
| **Java Version** | 8+ | 8+ (17+ recommended) |
| **Reactive** | Limited | Full Support |
| **Modular** | No | Yes |
| **Performance** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

**النصيحة: استخدم Resilience4j**

---

#### Resilience4j - الشرح الكامل

**Modules:**
```
┌────────────────────────────────────┐
│        Resilience4j Core           │
├────────────────────────────────────┤
│ ① Circuit Breaker                  │
│ ② Rate Limiter                     │
│ ③ Retry                            │
│ ④ Bulkhead (Thread Pool Isolation) │
│ ⑤ Time Limiter (Timeout)           │
│ ⑥ Cache                            │
└────────────────────────────────────┘
```

---

#### 1. Circuit Breaker (شرحناه سابقاً)

```java
@CircuitBreaker(name = "paymentService", fallbackMethod = "fallback")
public Payment getPayment(Long id) {
    return paymentClient.getPayment(id);
}
```

---

#### 2. Retry

**Configuration:**
```yaml
resilience4j:
  retry:
    configs:
      default:
        maxAttempts: 3
        waitDuration: 500ms
        retryExceptions:
          - java.io.IOException
          - java.util.concurrent.TimeoutException
        ignoreExceptions:
          - com.example.BusinessException

    instances:
      paymentService:
        baseConfig: default
        maxAttempts: 5
        waitDuration: 1s
        enableExponentialBackoff: true
        exponentialBackoffMultiplier: 2
        # 1s → 2s → 4s → 8s → 16s
```

**Usage:**
```java
@Service
public class OrderService {

    @Retry(name = "paymentService", fallbackMethod = "retryFallback")
    public Payment processPayment(Payment payment) {
        log.info("Attempting payment processing");
        return paymentClient.process(payment);
    }

    public Payment retryFallback(Payment payment, Exception ex) {
        log.error("All retries failed", ex);
        payment.setStatus("FAILED");
        return payment;
    }
}
```

**Programmatic:**
```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final RetryRegistry retryRegistry;
    private final PaymentClient paymentClient;

    public Payment processPayment(Payment payment) {
        Retry retry = retryRegistry.retry("paymentService");

        return retry.executeSupplier(() -> {
            log.info("Retry attempt");
            return paymentClient.process(payment);
        });
    }
}
```

---

#### 3. Rate Limiter

**Configuration:**
```yaml
resilience4j:
  ratelimiter:
    configs:
      default:
        limitForPeriod: 10  # 10 calls
        limitRefreshPeriod: 1s  # per second
        timeoutDuration: 0  # wait time (0 = fail immediately)

    instances:
      paymentService:
        baseConfig: default
        limitForPeriod: 100
        limitRefreshPeriod: 1s
```

**Usage:**
```java
@Service
public class PaymentService {

    @RateLimiter(name = "paymentService")
    public Payment processPayment(Payment payment) {
        return externalPaymentApi.process(payment);
    }

    // Fallback
    @RateLimiter(name = "paymentService", fallbackMethod = "rateLimitFallback")
    public Payment processPaymentWithFallback(Payment payment) {
        return externalPaymentApi.process(payment);
    }

    public Payment rateLimitFallback(Payment payment, RequestNotPermitted ex) {
        payment.setStatus("RATE_LIMITED");
        payment.setMessage("Too many requests, please try again later");
        return payment;
    }
}
```

---

#### 4. Bulkhead (Thread Pool Isolation)

**Configuration:**
```yaml
resilience4j:
  bulkhead:
    configs:
      default:
        maxConcurrentCalls: 10  # Max parallel calls

  thread-pool-bulkhead:
    configs:
      default:
        maxThreadPoolSize: 10
        coreThreadPoolSize: 5
        queueCapacity: 20
        keepAliveDuration: 20ms
```

**Usage:**
```java
// Semaphore Bulkhead (lightweight)
@Bulkhead(name = "paymentService", type = Bulkhead.Type.SEMAPHORE)
public Payment getPayment(Long id) {
    return paymentClient.getPayment(id);
}

// Thread Pool Bulkhead (isolated threads)
@Bulkhead(name = "paymentService", type = Bulkhead.Type.THREADPOOL)
public CompletableFuture<Payment> getPaymentAsync(Long id) {
    return CompletableFuture.supplyAsync(() -> paymentClient.getPayment(id));
}
```

**Purpose:** منع استهلاك كل الـ Threads

---

#### 5. Time Limiter (Timeout)

**Configuration:**
```yaml
resilience4j:
  timelimiter:
    configs:
      default:
        timeoutDuration: 3s
        cancelRunningFuture: true
```

**Usage:**
```java
@TimeLimiter(name = "paymentService")
@CircuitBreaker(name = "paymentService", fallbackMethod = "timeoutFallback")
public CompletableFuture<Payment> getPayment(Long id) {
    return CompletableFuture.supplyAsync(() -> paymentClient.getPayment(id));
}

public CompletableFuture<Payment> timeoutFallback(Long id, TimeoutException ex) {
    return CompletableFuture.completedFuture(
        Payment.builder()
            .id(id)
            .status("TIMEOUT")
            .message("Request timed out")
            .build()
    );
}
```

---

#### 6. Cache

**Configuration:**
```yaml
resilience4j:
  cache:
    configs:
      default:
        expiresAfterWrite: 10s
        maximumSize: 100
```

**Usage:**
```java
@Service
public class ProductService {

    @Cacheable(value = "products", key = "#id")
    public Product getProduct(Long id) {
        return productClient.getProduct(id);
    }

    @CacheEvict(value = "products", key = "#id")
    public void updateProduct(Long id, Product product) {
        productClient.updateProduct(id, product);
    }
}
```

---

#### Combining Multiple Patterns

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class OrderService {

    private final PaymentClient paymentClient;

    // Combine: Retry + Circuit Breaker + Time Limiter + Rate Limiter
    @Retry(name = "paymentService")
    @CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
    @TimeLimiter(name = "paymentService")
    @RateLimiter(name = "paymentService")
    @Bulkhead(name = "paymentService", type = Bulkhead.Type.SEMAPHORE)
    public CompletableFuture<Payment> processPayment(Payment payment) {
        log.info("Processing payment: {}", payment.getId());
        return CompletableFuture.supplyAsync(() -> paymentClient.process(payment));
    }

    public CompletableFuture<Payment> paymentFallback(Payment payment, Exception ex) {
        log.error("Payment processing failed", ex);

        payment.setStatus("FAILED");
        payment.setMessage("Payment service unavailable: " + ex.getMessage());

        return CompletableFuture.completedFuture(payment);
    }
}
```

**Order of Execution:**
```
Request
  ↓
Rate Limiter  (check quota)
  ↓
Bulkhead      (check concurrent calls)
  ↓
Time Limiter  (start timeout)
  ↓
Circuit Breaker (check state)
  ↓
Retry         (attempt + retries)
  ↓
Actual Call
```

---

#### Monitoring & Metrics

**Event Listeners:**
```java
@Component
@Slf4j
public class ResilienceEventListener {

    @Autowired
    public ResilienceEventListener(
            CircuitBreakerRegistry circuitBreakerRegistry,
            RetryRegistry retryRegistry,
            RateLimiterRegistry rateLimiterRegistry) {

        // Circuit Breaker Events
        circuitBreakerRegistry.circuitBreaker("paymentService")
            .getEventPublisher()
            .onStateTransition(event ->
                log.info("Circuit Breaker state: {} -> {}",
                    event.getStateTransition().getFromState(),
                    event.getStateTransition().getToState()));

        // Retry Events
        retryRegistry.retry("paymentService")
            .getEventPublisher()
            .onRetry(event ->
                log.warn("Retry attempt #{}", event.getNumberOfRetryAttempts()));

        // Rate Limiter Events
        rateLimiterRegistry.rateLimiter("paymentService")
            .getEventPublisher()
            .onSuccess(event ->
                log.debug("Rate limiter succeeded"))
            .onFailure(event ->
                log.warn("Rate limit exceeded"));
    }
}
```

**Actuator Endpoints:**
```bash
# Circuit Breakers
GET /actuator/circuitbreakers
GET /actuator/circuitbreakerevents

# Retries
GET /actuator/retries
GET /actuator/retryevents

# Rate Limiters
GET /actuator/ratelimiters
GET /actuator/ratelimiterevents

# Bulkheads
GET /actuator/bulkheads
GET /actuator/bulkheadevents

# Time Limiters
GET /actuator/timelimiters
```

**Prometheus Metrics:**
```
resilience4j_circuitbreaker_state
resilience4j_circuitbreaker_calls_total
resilience4j_retry_calls_total
resilience4j_ratelimiter_available_permissions
resilience4j_bulkhead_available_concurrent_calls
```

---

### 22. ما هو Retry Pattern و Timeout؟

#### Retry Pattern

**الشرح:**
- إعادة محاولة الطلب عند الفشل
- مناسب للـ Transient Failures (أخطاء مؤقتة)

**متى تستخدمه:**
- ✅ Network timeouts
- ✅ Database connection errors
- ✅ Temporary service unavailability
- ❌ Validation errors (400)
- ❌ Authentication errors (401)
- ❌ Not Found (404)

---

#### Retry Strategies

##### 1. Fixed Delay

```yaml
resilience4j:
  retry:
    instances:
      paymentService:
        maxAttempts: 3
        waitDuration: 1000  # 1 second between retries
```

```
Attempt 1 → Fail → Wait 1s
Attempt 2 → Fail → Wait 1s
Attempt 3 → Fail → Give up
```

##### 2. Exponential Backoff

```yaml
resilience4j:
  retry:
    instances:
      paymentService:
        maxAttempts: 5
        waitDuration: 1000
        enableExponentialBackoff: true
        exponentialBackoffMultiplier: 2
```

```
Attempt 1 → Fail → Wait 1s
Attempt 2 → Fail → Wait 2s
Attempt 3 → Fail → Wait 4s
Attempt 4 → Fail → Wait 8s
Attempt 5 → Fail → Give up
```

##### 3. Randomized (Jitter)

```yaml
resilience4j:
  retry:
    instances:
      paymentService:
        maxAttempts: 3
        waitDuration: 1000
        enableRandomizedWait: true
        randomizedWaitFactor: 0.5
        # Wait between 500ms - 1500ms
```

---

#### Retry Implementation

**Spring Retry:**
```xml
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
</dependency>
```

```java
@Configuration
@EnableRetry
public class RetryConfig {
}

@Service
public class PaymentService {

    @Retryable(
        value = {IOException.class, TimeoutException.class},
        maxAttempts = 3,
        backoff = @Backoff(delay = 1000, multiplier = 2)
    )
    public Payment processPayment(Payment payment) {
        return paymentApi.process(payment);
    }

    @Recover
    public Payment recover(IOException ex, Payment payment) {
        log.error("All retries failed", ex);
        payment.setStatus("FAILED");
        return payment;
    }
}
```

**Resilience4j Retry:**
```java
@Service
public class PaymentService {

    @Retry(name = "paymentService", fallbackMethod = "paymentRetryFallback")
    public Payment processPayment(Payment payment) {
        log.info("Processing payment: {}", payment.getId());
        return paymentApi.process(payment);
    }

    public Payment paymentRetryFallback(Payment payment, Exception ex) {
        log.error("All retries exhausted", ex);
        payment.setStatus("RETRY_FAILED");
        return payment;
    }
}
```

---

#### Timeout Pattern

**الشرح:**
- حد أقصى للانتظار
- يمنع Thread من الانتظار إلى الأبد

**Levels of Timeout:**

##### 1. Connection Timeout
```yaml
# وقت الاتصال بالـ Server
feign:
  client:
    config:
      default:
        connectTimeout: 2000  # 2 seconds
```

##### 2. Read Timeout
```yaml
# وقت انتظار الرد
feign:
  client:
    config:
      default:
        readTimeout: 5000  # 5 seconds
```

##### 3. Circuit Breaker Timeout
```yaml
resilience4j:
  timelimiter:
    configs:
      default:
        timeoutDuration: 3s
```

---

#### Timeout Implementation

**RestTemplate:**
```java
@Bean
public RestTemplate restTemplate() {
    SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
    factory.setConnectTimeout(2000);
    factory.setReadTimeout(5000);
    return new RestTemplate(factory);
}
```

**WebClient:**
```java
@Bean
public WebClient webClient() {
    HttpClient httpClient = HttpClient.create()
        .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 2000)
        .responseTimeout(Duration.ofSeconds(5));

    return WebClient.builder()
        .clientConnector(new ReactorClientHttpConnector(httpClient))
        .build();
}
```

**Feign:**
```yaml
feign:
  client:
    config:
      payment-service:
        connectTimeout: 2000
        readTimeout: 5000
```

**Resilience4j:**
```java
@TimeLimiter(name = "paymentService")
@CircuitBreaker(name = "paymentService", fallbackMethod = "timeoutFallback")
public CompletableFuture<Payment> getPayment(Long id) {
    return CompletableFuture.supplyAsync(() -> paymentClient.getPayment(id));
}

public CompletableFuture<Payment> timeoutFallback(Long id, TimeoutException ex) {
    log.error("Request timed out for payment: {}", id);
    return CompletableFuture.completedFuture(
        Payment.builder()
            .id(id)
            .status("TIMEOUT")
            .build()
    );
}
```

---

#### Best Practices

**1. Idempotent Operations:**
```java
// ✅ Safe to retry
@GetMapping("/payments/{id}")
public Payment getPayment(@PathVariable Long id) {
    return paymentService.getPayment(id);
}

// ❌ خطر! قد يُنشئ أكثر من order
@PostMapping("/orders")
@Retry(name = "orderService")
public Order createOrder(@RequestBody OrderRequest request) {
    return orderService.create(request);
}

// ✅ Idempotent with Idempotency Key
@PostMapping("/orders")
@Retry(name = "orderService")
public Order createOrder(
        @RequestBody OrderRequest request,
        @RequestHeader("Idempotency-Key") String key) {
    return orderService.createIdempotent(request, key);
}
```

**2. Retry on Specific Exceptions:**
```yaml
resilience4j:
  retry:
    instances:
      paymentService:
        retryExceptions:
          - java.io.IOException
          - java.util.concurrent.TimeoutException
          - org.springframework.web.client.ResourceAccessException
        ignoreExceptions:
          - com.example.ValidationException
          - com.example.NotFoundException
```

**3. Timeout Hierarchy:**
```
Circuit Breaker Timeout (3s)
    <
HTTP Read Timeout (5s)
    <
Overall Request Timeout (10s)
```

**4. Monitoring:**
```java
@Component
@Slf4j
public class RetryEventListener {

    @Autowired
    public RetryEventListener(RetryRegistry retryRegistry) {
        retryRegistry.retry("paymentService")
            .getEventPublisher()
            .onRetry(event -> {
                log.warn("Retry #{} for {}: {}",
                    event.getNumberOfRetryAttempts(),
                    event.getName(),
                    event.getLastThrowable().getMessage());
            })
            .onSuccess(event -> {
                log.info("Retry succeeded after {} attempts",
                    event.getNumberOfRetryAttempts());
            })
            .onError(event -> {
                log.error("All retries failed for {}",
                    event.getName(),
                    event.getLastThrowable());
            });
    }
}
```

---

## Distributed Tracing & Monitoring

### 23. كيف تتبع Request عبر عدة Services؟

**المشكلة:**
```
Client → API Gateway → Order Service → Payment Service
                                    → Inventory Service
                     → Notification Service

كيف تتبع طلب واحد عبر كل هذه الخدمات؟
```

**الحل: Distributed Tracing**

---

#### المفاهيم الأساسية

**1. Trace:**
- رحلة Request الكاملة عبر جميع Services
- Unique Trace ID

**2. Span:**
- عملية واحدة في Service واحد
- كل Span له Span ID
- كل Span ينتمي لـ Trace واحد

```
Trace ID: abc123
├─ Span 1: API Gateway (50ms)
├─ Span 2: Order Service (200ms)
│  ├─ Span 3: Payment Service (150ms)
│  └─ Span 4: Inventory Service (100ms)
└─ Span 5: Notification Service (50ms)

Total: 550ms
```

---

#### Implementation

**Tools:**
- Spring Cloud Sleuth (Trace ID generation)
- Zipkin / Jaeger (Visualization)
- Micrometer Tracing (New standard)

---

### 24. اشرح Spring Cloud Sleuth و Zipkin

#### Spring Cloud Sleuth

**التعريف:**
- يضيف Trace ID و Span ID تلقائياً
- يرسل Trace Data لـ Zipkin/Jaeger

**Setup:**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
```

**Automatic Instrumentation:**
```java
@RestController
@Slf4j
public class OrderController {

    @GetMapping("/orders/{id}")
    public Order getOrder(@PathVariable Long id) {
        log.info("Getting order: {}", id);
        // Log includes: [appname,traceId,spanId]
        return orderService.getOrder(id);
    }
}
```

**Log Output:**
```
2024-01-15 10:30:45 INFO [order-service,abc123,def456] Getting order: 123
                                      ↑        ↑      ↑
                                   app name  trace  span
```

---

#### Zipkin Integration

**Setup:**
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-sleuth-zipkin</artifactId>
</dependency>
```

**Configuration:**
```yaml
spring:
  application:
    name: order-service

  sleuth:
    sampler:
      probability: 1.0  # 100% sampling (dev), 0.1 (10%) for prod

  zipkin:
    base-url: http://localhost:9411
    sender:
      type: web  # or kafka, rabbit
```

**Run Zipkin:**
```bash
docker run -d -p 9411:9411 openzipkin/zipkin
```

**Access UI:**
```
http://localhost:9411
```

---

#### Custom Spans

```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final Tracer tracer;
    private final PaymentClient paymentClient;

    public Order createOrder(OrderRequest request) {
        // Custom Span
        Span customSpan = tracer.nextSpan().name("validate-order");

        try (Tracer.SpanInScope ws = tracer.withSpan(customSpan.start())) {
            validateOrder(request);
            customSpan.tag("order.type", request.getType());
            customSpan.tag("order.amount", String.valueOf(request.getAmount()));
        } finally {
            customSpan.end();
        }

        // Another custom span
        Span paymentSpan = tracer.nextSpan().name("process-payment");
        try (Tracer.SpanInScope ws = tracer.withSpan(paymentSpan.start())) {
            Payment payment = paymentClient.process(request.getPayment());
            paymentSpan.tag("payment.id", payment.getId().toString());
            return orderRepository.save(new Order(request, payment));
        } finally {
            paymentSpan.end();
        }
    }
}
```

---

#### Propagating Trace Context

**Automatic (HTTP):**
Sleuth يضيف Headers تلقائياً:
```
X-B3-TraceId: abc123
X-B3-SpanId: def456
X-B3-ParentSpanId: ghi789
X-B3-Sampled: 1
```

**Manual (Messaging):**
```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final RabbitTemplate rabbitTemplate;
    private final Tracer tracer;

    public void publishOrderEvent(Order order) {
        OrderEvent event = new OrderEvent(order);

        rabbitTemplate.convertAndSend("order.exchange", "order.created", event, message -> {
            // Add trace context to message
            Span currentSpan = tracer.currentSpan();
            if (currentSpan != null) {
                message.getMessageProperties().setHeader("X-B3-TraceId",
                    currentSpan.context().traceId());
                message.getMessageProperties().setHeader("X-B3-SpanId",
                    currentSpan.context().spanId());
            }
            return message;
        });
    }
}
```

---

### 25. ما هي أدوات Monitoring (Prometheus, Grafana, ELK)؟

#### 1. Prometheus (Metrics)

**التعريف:**
- Time-series database
- يجمع Metrics من Services

**Setup:**
```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

**Configuration:**
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
    tags:
      application: ${spring.application.name}
      environment: ${spring.profiles.active}
```

**Metrics Endpoint:**
```bash
curl http://localhost:8080/actuator/prometheus
```

**Custom Metrics:**
```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final MeterRegistry meterRegistry;
    private final Counter orderCounter;
    private final Timer orderTimer;

    @PostConstruct
    public void initMetrics() {
        orderCounter = Counter.builder("orders.created")
            .description("Total orders created")
            .tag("type", "online")
            .register(meterRegistry);

        orderTimer = Timer.builder("orders.processing.time")
            .description("Order processing time")
            .register(meterRegistry);
    }

    public Order createOrder(OrderRequest request) {
        return orderTimer.record(() -> {
            Order order = orderRepository.save(new Order(request));
            orderCounter.increment();
            return order;
        });
    }
}
```

**Prometheus Config (prometheus.yml):**
```yaml
scrape_configs:
  - job_name: 'spring-boot-apps'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['localhost:8080', 'localhost:8081', 'localhost:8082']
```

---

#### 2. Grafana (Visualization)

**Setup:**
```bash
docker run -d -p 3000:3000 grafana/grafana
```

**Dashboard Creation:**
1. Add Prometheus as Data Source
2. Create Dashboard
3. Add Panels with PromQL queries

**Sample Queries:**
```promql
# Request Rate
rate(http_server_requests_seconds_count[5m])

# Error Rate
rate(http_server_requests_seconds_count{status=~"5.."}[5m])

# 95th Percentile Latency
histogram_quantile(0.95, http_server_requests_seconds_bucket)

# Orders Created
sum(orders_created_total) by (service)
```

---

#### 3. ELK Stack (Logging)

**Components:**
- **E**lasticsearch: Search & Storage
- **L**ogstash: Log Processing
- **K**ibana: Visualization

**Filebeat Alternative (Lighter):**
```yaml
# docker-compose.yml
version: '3'
services:
  elasticsearch:
    image: elasticsearch:8.5.0
    ports:
      - 9200:9200

  kibana:
    image: kibana:8.5.0
    ports:
      - 5601:5601

  logstash:
    image: logstash:8.5.0
    ports:
      - 5044:5044
```

**Logback Configuration:**
```xml
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
</dependency>
```

```xml
<!-- logback-spring.xml -->
<configuration>
    <appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>localhost:5044</destination>
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <includeMdcKeyName>traceId</includeMdcKeyName>
            <includeMdcKeyName>spanId</includeMdcKeyName>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="LOGSTASH"/>
    </root>
</configuration>
```

---

#### Full Monitoring Stack

```yaml
# docker-compose.yml
version: '3.8'

services:
  # Distributed Tracing
  zipkin:
    image: openzipkin/zipkin
    ports:
      - 9411:9411

  # Metrics
  prometheus:
    image: prom/prometheus
    ports:
      - 9090:9090
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  # Visualization
  grafana:
    image: grafana/grafana
    ports:
      - 3000:3000
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin

  # Logging
  elasticsearch:
    image: elasticsearch:8.5.0
    ports:
      - 9200:9200
    environment:
      - discovery.type=single-node

  kibana:
    image: kibana:8.5.0
    ports:
      - 5601:5601
```

**Access:**
- Zipkin: http://localhost:9411
- Prometheus: http://localhost:9090
- Grafana: http://localhost:3000
- Kibana: http://localhost:5601

---

## Security

### 26. كيف تطبق Security في Microservices؟

**التحديات:**

```
┌─────────────────────────────────────┐
│  Monolithic Security (بسيط)         │
│  - Single entry point               │
│  - Session-based auth               │
│  - One security layer               │
└─────────────────────────────────────┘

vs

┌─────────────────────────────────────┐
│  Microservices Security (معقد)     │
│  - Multiple entry points            │
│  - Stateless auth (JWT)             │
│  - Security at each service         │
│  - API Gateway security             │
│  - Service-to-service auth          │
└─────────────────────────────────────┘
```

---

#### Security Layers

##### 1. Perimeter Security (API Gateway)

```java
@Configuration
@EnableWebFluxSecurity
public class GatewaySecurityConfig {

    @Bean
    public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
        return http
            .csrf().disable()
            .authorizeExchange(exchanges -> exchanges
                // Public endpoints
                .pathMatchers("/api/auth/**").permitAll()
                .pathMatchers("/actuator/health").permitAll()

                // Admin only
                .pathMatchers("/api/admin/**").hasRole("ADMIN")

                // Authenticated
                .anyExchange().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt.jwtAuthenticationConverter(jwtConverter()))
            )
            .build();
    }
}
```

##### 2. Service-Level Security

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .csrf().disable()
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/actuator/**").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2.jwt())
            .build();
    }
}

@Service
public class OrderService {

    @PreAuthorize("hasRole('USER')")
    public Order createOrder(OrderRequest request) {
        return orderRepository.save(new Order(request));
    }

    @PreAuthorize("hasRole('ADMIN') or #userId == authentication.principal.id")
    public List<Order> getUserOrders(Long userId) {
        return orderRepository.findByUserId(userId);
    }

    @PostAuthorize("returnObject.userId == authentication.principal.id")
    public Order getOrder(Long orderId) {
        return orderRepository.findById(orderId)
            .orElseThrow(() -> new NotFoundException());
    }
}
```

##### 3. Method-Level Security

```java
@Service
public class PaymentService {

    @Secured("ROLE_ADMIN")
    public void refundPayment(Long paymentId) {
        // Only admins can refund
    }

    @PreAuthorize("@securityService.canAccessPayment(#paymentId)")
    public Payment getPayment(Long paymentId) {
        return paymentRepository.findById(paymentId).orElseThrow();
    }
}

@Service
public class SecurityService {

    public boolean canAccessPayment(Long paymentId) {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        Payment payment = paymentRepository.findById(paymentId).orElse(null);

        if (payment == null) return false;

        // Check if user owns this payment
        return payment.getUserId().equals(getCurrentUserId(auth));
    }
}
```

---

#### Service-to-Service Authentication

##### 1. JWT Propagation

```java
@Configuration
public class FeignClientConfig {

    @Bean
    public RequestInterceptor requestInterceptor() {
        return requestTemplate -> {
            // Get JWT from SecurityContext
            Authentication authentication = SecurityContextHolder
                .getContext()
                .getAuthentication();

            if (authentication != null && authentication.getCredentials() instanceof String) {
                String token = (String) authentication.getCredentials();
                requestTemplate.header("Authorization", "Bearer " + token);
            }
        };
    }
}
```

##### 2. Service Credentials (OAuth2 Client Credentials)

```java
@Configuration
public class OAuth2Config {

    @Bean
    public OAuth2AuthorizedClientManager authorizedClientManager(
            ClientRegistrationRepository clientRegistrationRepository,
            OAuth2AuthorizedClientRepository authorizedClientRepository) {

        OAuth2AuthorizedClientProvider authorizedClientProvider =
            OAuth2AuthorizedClientProviderBuilder.builder()
                .clientCredentials()
                .build();

        DefaultOAuth2AuthorizedClientManager authorizedClientManager =
            new DefaultOAuth2AuthorizedClientManager(
                clientRegistrationRepository,
                authorizedClientRepository);

        authorizedClientManager.setAuthorizedClientProvider(authorizedClientProvider);

        return authorizedClientManager;
    }
}
```

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          payment-service:
            client-id: order-service
            client-secret: ${CLIENT_SECRET}
            authorization-grant-type: client_credentials
            scope: payment.read,payment.write
        provider:
          payment-service:
            token-uri: http://auth-server/oauth/token
```

##### 3. mTLS (Mutual TLS)

```yaml
# Order Service
server:
  ssl:
    enabled: true
    key-store: classpath:keystore.p12
    key-store-password: ${KEYSTORE_PASSWORD}
    key-store-type: PKCS12
    trust-store: classpath:truststore.p12
    trust-store-password: ${TRUSTSTORE_PASSWORD}
    client-auth: need  # Require client certificate
```

---

#### API Key Management

```java
@Component
public class ApiKeyFilter extends OncePerRequestFilter {

    @Autowired
    private ApiKeyService apiKeyService;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                   HttpServletResponse response,
                                   FilterChain filterChain) throws ServletException, IOException {

        String apiKey = request.getHeader("X-API-Key");

        if (apiKey == null || !apiKeyService.isValid(apiKey)) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            response.getWriter().write("Invalid API Key");
            return;
        }

        // Set authentication
        ApiKeyAuthentication auth = new ApiKeyAuthentication(apiKey);
        SecurityContextHolder.getContext().setAuthentication(auth);

        filterChain.doFilter(request, response);
    }
}
```

---

#### Rate Limiting by User

```java
@Component
public class UserRateLimitFilter extends OncePerRequestFilter {

    @Autowired
    private RateLimiterRegistry rateLimiterRegistry;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                   HttpServletResponse response,
                                   FilterChain filterChain) throws ServletException, IOException {

        String userId = getCurrentUserId();
        RateLimiter rateLimiter = rateLimiterRegistry.rateLimiter(userId);

        try {
            rateLimiter.acquirePermission();
            filterChain.doFilter(request, response);
        } catch (RequestNotPermitted e) {
            response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
            response.getWriter().write("Rate limit exceeded");
        }
    }
}
```

---

#### Data Encryption

##### 1. Database Encryption

```java
@Entity
public class User {

    @Id
    private Long id;

    private String email;

    @Convert(converter = EncryptedStringConverter.class)
    private String ssn;  // Social Security Number

    @Convert(converter = EncryptedStringConverter.class)
    private String creditCard;
}

@Converter
public class EncryptedStringConverter implements AttributeConverter<String, String> {

    @Autowired
    private EncryptionService encryptionService;

    @Override
    public String convertToDatabaseColumn(String attribute) {
        return encryptionService.encrypt(attribute);
    }

    @Override
    public String convertToEntityAttribute(String dbData) {
        return encryptionService.decrypt(dbData);
    }
}
```

##### 2. Secrets Management

```yaml
# Using Vault
spring:
  cloud:
    vault:
      uri: https://vault.example.com
      token: ${VAULT_TOKEN}
      kv:
        enabled: true
        backend: secret
        default-context: order-service

# Using AWS Secrets Manager
aws:
  secretsmanager:
    enabled: true
    prefix: /microservices/order-service
```

---

### 27. اشرح OAuth2 و JWT

#### OAuth2

**التعريف:**
- Authorization Framework
- يسمح للتطبيقات بالوصول لموارد المستخدم بدون معرفة Password

**OAuth2 Roles:**
```
┌──────────────┐
│ Resource     │  User
│ Owner        │
└──────────────┘
       │
       ↓ (1) Authorization Request
┌──────────────┐
│ Client       │  Your Application
│ Application  │
└──────────────┘
       │
       ↓ (2) Authorization Grant
┌──────────────┐
│ Authorization│  OAuth Server (Keycloak, Auth0)
│ Server       │
└──────────────┘
       │
       ↓ (3) Access Token
┌──────────────┐
│ Resource     │  Your API
│ Server       │
└──────────────┘
```

**OAuth2 Grant Types:**

##### 1. Authorization Code (الأكثر أماناً)

```java
@Configuration
public class OAuth2LoginConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .authorizeHttpRequests(auth -> auth
                .anyRequest().authenticated()
            )
            .oauth2Login(oauth2 -> oauth2
                .loginPage("/login")
                .defaultSuccessUrl("/dashboard")
            )
            .build();
    }
}
```

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: ${GOOGLE_CLIENT_ID}
            client-secret: ${GOOGLE_CLIENT_SECRET}
            scope: openid,profile,email
        provider:
          google:
            authorization-uri: https://accounts.google.com/o/oauth2/v2/auth
            token-uri: https://oauth2.googleapis.com/token
            user-info-uri: https://www.googleapis.com/oauth2/v3/userinfo
```

##### 2. Client Credentials (Service-to-Service)

```java
@Configuration
public class OAuth2ClientConfig {

    @Bean
    public WebClient webClient(OAuth2AuthorizedClientManager authorizedClientManager) {
        ServletOAuth2AuthorizedClientExchangeFilterFunction oauth2Client =
            new ServletOAuth2AuthorizedClientExchangeFilterFunction(authorizedClientManager);

        return WebClient.builder()
            .apply(oauth2Client.oauth2Configuration())
            .build();
    }
}

@Service
public class PaymentService {

    @Autowired
    private WebClient webClient;

    public Payment processPayment(PaymentRequest request) {
        return webClient.post()
            .uri("http://payment-api/payments")
            .attributes(clientRegistrationId("payment-client"))
            .bodyValue(request)
            .retrieve()
            .bodyToMono(Payment.class)
            .block();
    }
}
```

##### 3. Password Grant (Legacy - لا يُنصح به)

```java
// ❌ Don't use in production
public TokenResponse login(String username, String password) {
    MultiValueMap<String, String> params = new LinkedMultiValueMap<>();
    params.add("grant_type", "password");
    params.add("username", username);
    params.add("password", password);

    return restTemplate.postForObject(
        "http://auth-server/oauth/token",
        params,
        TokenResponse.class
    );
}
```

---

#### JWT (JSON Web Token)

**Structure:**
```
Header.Payload.Signature

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

**Decoded:**

**Header:**
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Payload:**
```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "email": "john@example.com",
  "roles": ["USER", "ADMIN"],
  "iat": 1516239022,
  "exp": 1516242622
}
```

**Signature:**
```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

---

#### JWT Implementation

**Generate JWT:**

```java
@Service
public class JwtService {

    @Value("${jwt.secret}")
    private String secret;

    @Value("${jwt.expiration}")
    private Long expiration;

    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("roles", userDetails.getAuthorities().stream()
            .map(GrantedAuthority::getAuthority)
            .collect(Collectors.toList()));
        claims.put("email", userDetails.getUsername());

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

    public List<String> extractRoles(String token) {
        Claims claims = extractAllClaims(token);
        return (List<String>) claims.get("roles");
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
        Claims claims = extractAllClaims(token);
        return claimsResolver.apply(claims);
    }

    private Claims extractAllClaims(String token) {
        return Jwts.parser()
            .setSigningKey(secret)
            .parseClaimsJws(token)
            .getBody();
    }
}
```

**JWT Filter:**

```java
@Component
@RequiredArgsConstructor
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                   HttpServletResponse response,
                                   FilterChain filterChain) throws ServletException, IOException {

        String authHeader = request.getHeader("Authorization");

        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }

        String jwt = authHeader.substring(7);
        String username = jwtService.extractUsername(jwt);

        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);

            if (jwtService.validateToken(jwt, userDetails)) {
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
```

**Login Endpoint:**

```java
@RestController
@RequestMapping("/api/auth")
@RequiredArgsConstructor
public class AuthController {

    private final AuthenticationManager authenticationManager;
    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;

    @PostMapping("/login")
    public ResponseEntity<AuthResponse> login(@RequestBody LoginRequest request) {
        authenticationManager.authenticate(
            new UsernamePasswordAuthenticationToken(
                request.getEmail(),
                request.getPassword()
            )
        );

        UserDetails userDetails = userDetailsService.loadUserByUsername(request.getEmail());
        String token = jwtService.generateToken(userDetails);

        return ResponseEntity.ok(AuthResponse.builder()
            .token(token)
            .type("Bearer")
            .expiresIn(86400L)
            .build());
    }

    @PostMapping("/refresh")
    public ResponseEntity<AuthResponse> refresh(@RequestHeader("Authorization") String authHeader) {
        String refreshToken = authHeader.substring(7);
        String username = jwtService.extractUsername(refreshToken);

        if (username != null) {
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);

            if (jwtService.validateToken(refreshToken, userDetails)) {
                String newToken = jwtService.generateToken(userDetails);

                return ResponseEntity.ok(AuthResponse.builder()
                    .token(newToken)
                    .type("Bearer")
                    .expiresIn(86400L)
                    .build());
            }
        }

        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
    }
}
```

---

#### JWT Best Practices

**1. Short Expiration + Refresh Token:**

```java
public class JwtService {

    public String generateAccessToken(UserDetails userDetails) {
        return generateToken(userDetails, 900000); // 15 minutes
    }

    public String generateRefreshToken(UserDetails userDetails) {
        return generateToken(userDetails, 604800000); // 7 days
    }

    private String generateToken(UserDetails userDetails, long expiration) {
        return Jwts.builder()
            .setSubject(userDetails.getUsername())
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + expiration))
            .signWith(SignatureAlgorithm.HS512, secret)
            .compact();
    }
}
```

**2. Token Blacklist (Redis):**

```java
@Service
@RequiredArgsConstructor
public class TokenBlacklistService {

    private final RedisTemplate<String, String> redisTemplate;

    public void blacklist(String token) {
        long expiration = extractExpiration(token).getTime() - System.currentTimeMillis();
        redisTemplate.opsForValue().set(
            "blacklist:" + token,
            "true",
            expiration,
            TimeUnit.MILLISECONDS
        );
    }

    public boolean isBlacklisted(String token) {
        return Boolean.TRUE.equals(
            redisTemplate.hasKey("blacklist:" + token)
        );
    }
}
```

**3. RSA Keys (Production):**

```java
@Configuration
public class JwtConfig {

    @Value("${jwt.private-key}")
    private String privateKeyPath;

    @Value("${jwt.public-key}")
    private String publicKeyPath;

    @Bean
    public PrivateKey privateKey() throws Exception {
        byte[] keyBytes = Files.readAllBytes(Paths.get(privateKeyPath));
        PKCS8EncodedKeySpec spec = new PKCS8EncodedKeySpec(keyBytes);
        KeyFactory kf = KeyFactory.getInstance("RSA");
        return kf.generatePrivate(spec);
    }

    @Bean
    public PublicKey publicKey() throws Exception {
        byte[] keyBytes = Files.readAllBytes(Paths.get(publicKeyPath));
        X509EncodedKeySpec spec = new X509EncodedKeySpec(keyBytes);
        KeyFactory kf = KeyFactory.getInstance("RSA");
        return kf.generatePublic(spec);
    }
}
```

---

### 28. ما هو Spring Security وكيف تستخدمه؟

#### Spring Security Basics

**Setup:**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

**Basic Configuration:**

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .csrf().disable()
            .authorizeHttpRequests(auth -> auth
                // Public endpoints
                .requestMatchers("/api/auth/**", "/actuator/health").permitAll()

                // Role-based
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/users/**").hasAnyRole("USER", "ADMIN")

                // Authenticated
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)
            .build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config)
            throws Exception {
        return config.getAuthenticationManager();
    }
}
```

---

#### UserDetailsService

```java
@Service
@RequiredArgsConstructor
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
        User user = userRepository.findByEmail(email)
            .orElseThrow(() -> new UsernameNotFoundException("User not found: " + email));

        return org.springframework.security.core.userdetails.User.builder()
            .username(user.getEmail())
            .password(user.getPassword())
            .roles(user.getRoles().toArray(new String[0]))
            .accountExpired(!user.isActive())
            .accountLocked(user.isLocked())
            .credentialsExpired(false)
            .disabled(!user.isEnabled())
            .build();
    }
}
```

---

#### Method Security

```java
@Service
public class OrderService {

    // Role-based
    @PreAuthorize("hasRole('ADMIN')")
    public void deleteOrder(Long orderId) {
        orderRepository.deleteById(orderId);
    }

    // Expression-based
    @PreAuthorize("hasRole('USER') and #order.userId == authentication.principal.id")
    public Order updateOrder(Order order) {
        return orderRepository.save(order);
    }

    // Post-authorization
    @PostAuthorize("returnObject.userId == authentication.principal.id")
    public Order getOrder(Long orderId) {
        return orderRepository.findById(orderId).orElseThrow();
    }

    // Filtering collections
    @PostFilter("filterObject.userId == authentication.principal.id")
    public List<Order> getAllOrders() {
        return orderRepository.findAll();
    }

    // Custom security expression
    @PreAuthorize("@securityService.canAccessOrder(#orderId)")
    public Order getOrderSecure(Long orderId) {
        return orderRepository.findById(orderId).orElseThrow();
    }
}
```

---

#### Custom Security Expressions

```java
@Component("securityService")
@RequiredArgsConstructor
public class SecurityService {

    private final OrderRepository orderRepository;

    public boolean canAccessOrder(Long orderId) {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();

        if (auth == null) return false;

        Order order = orderRepository.findById(orderId).orElse(null);
        if (order == null) return false;

        String currentUserId = auth.getName();

        // Admin can access all
        if (hasRole(auth, "ADMIN")) return true;

        // User can access own orders
        return order.getUserId().equals(currentUserId);
    }

    public boolean hasRole(Authentication auth, String role) {
        return auth.getAuthorities().stream()
            .anyMatch(a -> a.getAuthority().equals("ROLE_" + role));
    }
}
```

---

#### Getting Current User

```java
@RestController
@RequestMapping("/api/profile")
public class ProfileController {

    // Method 1: SecurityContext
    @GetMapping("/me")
    public User getCurrentUser() {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        String email = auth.getName();
        return userRepository.findByEmail(email).orElseThrow();
    }

    // Method 2: @AuthenticationPrincipal
    @GetMapping("/me2")
    public User getCurrentUser2(@AuthenticationPrincipal UserDetails userDetails) {
        return userRepository.findByEmail(userDetails.getUsername()).orElseThrow();
    }

    // Method 3: Custom annotation
    @GetMapping("/me3")
    public User getCurrentUser3(@CurrentUser User user) {
        return user;
    }
}

// Custom annotation
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
@AuthenticationPrincipal
public @interface CurrentUser {
}

// Argument resolver
@Component
public class CurrentUserArgumentResolver implements HandlerMethodArgumentResolver {

    @Autowired
    private UserRepository userRepository;

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return parameter.hasParameterAnnotation(CurrentUser.class);
    }

    @Override
    public Object resolveArgument(MethodParameter parameter,
                                  ModelAndViewContainer mavContainer,
                                  NativeWebRequest webRequest,
                                  WebDataBinderFactory binderFactory) {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        return userRepository.findByEmail(auth.getName()).orElseThrow();
    }
}
```

---

#### CORS Configuration

```java
@Configuration
public class CorsConfig {

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();

        configuration.setAllowedOrigins(Arrays.asList(
            "http://localhost:3000",
            "https://myapp.com"
        ));

        configuration.setAllowedMethods(Arrays.asList(
            "GET", "POST", "PUT", "DELETE", "OPTIONS"
        ));

        configuration.setAllowedHeaders(Arrays.asList("*"));
        configuration.setAllowCredentials(true);
        configuration.setMaxAge(3600L);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);

        return source;
    }
}
```

---

## Database & Transactions

### 29. ما هو Database per Service Pattern؟

**الشرح:**

**Database per Service:**
- كل Microservice له قاعدة بيانات خاصة
- لا يمكن لـ Service الوصول لقاعدة بيانات Service آخر
- Data Independence & Isolation

```
┌──────────────────┐     ┌──────────────────┐
│  Order Service   │     │ Payment Service  │
├──────────────────┤     ├──────────────────┤
│ - createOrder()  │     │ - processPayment │
└────────┬─────────┘     └────────┬─────────┘
         │                        │
         ↓                        ↓
┌──────────────────┐     ┌──────────────────┐
│  Orders DB       │     │  Payments DB     │
│  (MySQL)         │     │  (PostgreSQL)    │
└──────────────────┘     └──────────────────┘
```

---

#### مميزات

**1. Technology Diversity:**
```
Order Service    → MySQL
Payment Service  → PostgreSQL
User Service     → MongoDB
Analytics Service → Cassandra
```

**2. Independent Scaling:**
```java
// Orders DB - High traffic
@Configuration
public class OrderDbConfig {
    @Bean
    public DataSource orderDataSource() {
        HikariConfig config = new HikariConfig();
        config.setMaximumPoolSize(50);  // Large pool
        return new HikariDataSource(config);
    }
}

// User DB - Low traffic
@Configuration
public class UserDbConfig {
    @Bean
    public DataSource userDataSource() {
        HikariConfig config = new HikariConfig();
        config.setMaximumPoolSize(10);  // Small pool
        return new HikariDataSource(config);
    }
}
```

**3. Fault Isolation:**
```
Orders DB crashes ❌ → Only Order Service affected
                     → Payment Service still works ✅
```

**4. Schema Evolution:**
```sql
-- Order Service - يمكن تغيير Schema بحرية
ALTER TABLE orders ADD COLUMN notes TEXT;

-- لا يؤثر على Payment Service
```

---

#### عيوب

**1. No Foreign Keys:**

```sql
-- ❌ لا يمكن
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    user_id BIGINT,
    FOREIGN KEY (user_id) REFERENCES users.users(id)
    -- users في DB آخر!
);

-- ✅ الحل: Application-level validation
```

```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final UserClient userClient;
    private final OrderRepository orderRepository;

    @Transactional
    public Order createOrder(OrderRequest request) {
        // Validate user exists
        User user = userClient.getUser(request.getUserId());
        if (user == null) {
            throw new UserNotFoundException();
        }

        return orderRepository.save(new Order(request));
    }
}
```

**2. No Joins:**

```sql
-- ❌ لا يمكن
SELECT o.*, u.name, u.email
FROM orders o
JOIN users u ON o.user_id = u.id;
-- users في DB آخر!

-- ✅ الحل: API Composition
```

```java
public class OrderDetails {
    private Order order;
    private User user;
    private Payment payment;
}

@Service
public class OrderService {

    public OrderDetails getOrderDetails(Long orderId) {
        // Multiple service calls
        Order order = orderRepository.findById(orderId).orElseThrow();
        User user = userClient.getUser(order.getUserId());
        Payment payment = paymentClient.getPayment(order.getPaymentId());

        return new OrderDetails(order, user, payment);
    }
}
```

**3. Distributed Queries:**

```java
// ❌ صعب: Get all orders with user name
public List<OrderWithUser> getAllOrdersWithUsers() {
    List<Order> orders = orderRepository.findAll();

    // N+1 problem!
    return orders.stream()
        .map(order -> {
            User user = userClient.getUser(order.getUserId());
            return new OrderWithUser(order, user);
        })
        .collect(Collectors.toList());
}

// ✅ الحل: CQRS + Read Replicas
```

---

#### Implementation Strategies

##### 1. Schema per Service (Same DB Instance)

```yaml
# Order Service
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/ecommerce
    username: order_user
    password: ${ORDER_DB_PASSWORD}
  jpa:
    properties:
      hibernate:
        default_schema: orders
```

```yaml
# Payment Service
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/ecommerce
    username: payment_user
    password: ${PAYMENT_DB_PASSWORD}
  jpa:
    properties:
      hibernate:
        default_schema: payments
```

```sql
-- Database setup
CREATE SCHEMA orders;
CREATE SCHEMA payments;

-- Restrict access
GRANT SELECT, INSERT, UPDATE, DELETE ON orders.* TO 'order_user'@'%';
GRANT SELECT, INSERT, UPDATE, DELETE ON payments.* TO 'payment_user'@'%';
```

##### 2. Separate Database Instances (Recommended)

```yaml
# Order Service
spring:
  datasource:
    url: jdbc:mysql://order-db:3306/orders
    username: order_user
    password: ${ORDER_DB_PASSWORD}

# Payment Service
spring:
  datasource:
    url: jdbc:postgresql://payment-db:5432/payments
    username: payment_user
    password: ${PAYMENT_DB_PASSWORD}
```

##### 3. Polyglot Persistence

```java
// Order Service - MySQL
@Configuration
public class OrderDbConfig {
    @Bean
    public DataSource dataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:mysql://localhost:3306/orders")
            .build();
    }
}

// User Service - MongoDB
@Configuration
public class UserDbConfig {
    @Bean
    public MongoClient mongoClient() {
        return MongoClients.create("mongodb://localhost:27017/users");
    }
}

// Analytics Service - Cassandra
@Configuration
public class AnalyticsDbConfig {
    @Bean
    public CqlSession session() {
        return CqlSession.builder()
            .addContactPoint(new InetSocketAddress("localhost", 9042))
            .withKeyspace("analytics")
            .build();
    }
}
```

---

### 30. كيف تتعامل مع Distributed Transactions؟

**المشكلة:**

```
Order Service → Create Order ✅
             → Call Payment Service
Payment Service → Process Payment ❌ (Failed)

مشكلة: Order created but payment failed!
```

**حلول:**

---

#### 1. Two-Phase Commit (2PC) - تجنبه!

**كيف يعمل:**
```
Phase 1: Prepare
Coordinator → All Services: "Are you ready?"
All Services → Coordinator: "Yes" or "No"

Phase 2: Commit/Rollback
Coordinator → All Services: "Commit" or "Rollback"
```

**لماذا نتجنبه:**
- ❌ Blocking (Services تنتظر)
- ❌ Single Point of Failure (Coordinator)
- ❌ Performance issues
- ❌ لا يعمل مع NoSQL

---

#### 2. Saga Pattern (الأفضل)

**التعريف:**
- سلسلة من Local Transactions
- كل Transaction لها Compensation

**Types:**

##### A. Choreography-Based Saga

**Events-driven:**

```java
// Order Service
@Service
@RequiredArgsConstructor
public class OrderService {

    private final KafkaTemplate<String, OrderEvent> kafkaTemplate;

    @Transactional
    public Order createOrder(OrderRequest request) {
        // ① Create order (Pending)
        Order order = orderRepository.save(new Order(request, "PENDING"));

        // ② Publish event
        kafkaTemplate.send("order-events",
            new OrderCreatedEvent(order.getId(), order.getAmount()));

        return order;
    }
}

// Payment Service (Listener)
@Component
public class PaymentListener {

    @KafkaListener(topics = "order-events")
    @Transactional
    public void handleOrderCreated(OrderCreatedEvent event) {
        try {
            // ③ Process payment
            Payment payment = paymentService.process(event.getOrderId(), event.getAmount());

            // ④ Publish success
            kafkaTemplate.send("payment-events",
                new PaymentCompletedEvent(event.getOrderId(), payment.getId()));

        } catch (PaymentFailedException e) {
            // ⑤ Publish failure
            kafkaTemplate.send("payment-events",
                new PaymentFailedEvent(event.getOrderId(), e.getMessage()));
        }
    }
}

// Order Service (Listener)
@Component
public class OrderEventListener {

    @KafkaListener(topics = "payment-events")
    @Transactional
    public void handlePaymentCompleted(PaymentCompletedEvent event) {
        // ⑥ Update order status
        Order order = orderRepository.findById(event.getOrderId()).orElseThrow();
        order.setStatus("COMPLETED");
        order.setPaymentId(event.getPaymentId());
        orderRepository.save(order);

        // ⑦ Publish next event (e.g., ship order)
        kafkaTemplate.send("shipping-events",
            new ShipOrderEvent(order.getId()));
    }

    @KafkaListener(topics = "payment-events")
    @Transactional
    public void handlePaymentFailed(PaymentFailedEvent event) {
        // ⑧ Compensate: Cancel order
        Order order = orderRepository.findById(event.getOrderId()).orElseThrow();
        order.setStatus("CANCELLED");
        orderRepository.save(order);
    }
}
```

**Flow:**
```
Order Service:    Create Order (PENDING)
                       ↓ (publish event)
Payment Service:  Process Payment
                       ↓ (success)
Order Service:    Update Order (COMPLETED)
                       ↓ (publish event)
Shipping Service: Ship Order
```

**Compensation Flow:**
```
Order Service:    Create Order (PENDING)
                       ↓
Payment Service:  Process Payment ❌
                       ↓ (failure event)
Order Service:    Cancel Order (Compensate)
```

---

##### B. Orchestration-Based Saga

**Centralized coordinator:**

```java
@Service
@RequiredArgsConstructor
public class OrderSagaOrchestrator {

    private final OrderRepository orderRepository;
    private final PaymentClient paymentClient;
    private final InventoryClient inventoryClient;
    private final ShippingClient shippingClient;

    @Transactional
    public Order createOrder(OrderRequest request) {
        Order order = null;
        Payment payment = null;
        Reservation reservation = null;

        try {
            // ① Create Order
            order = orderRepository.save(new Order(request, "PENDING"));

            // ② Reserve Inventory
            reservation = inventoryClient.reserve(
                order.getProductId(),
                order.getQuantity()
            );

            // ③ Process Payment
            payment = paymentClient.process(
                order.getId(),
                order.getAmount()
            );

            // ④ Ship Order
            shippingClient.ship(order.getId());

            // ⑤ Update Order Status
            order.setStatus("COMPLETED");
            order.setPaymentId(payment.getId());
            orderRepository.save(order);

            return order;

        } catch (Exception e) {
            // ⑥ Compensate (Rollback)
            compensate(order, payment, reservation);
            throw new OrderCreationFailedException("Order creation failed", e);
        }
    }

    private void compensate(Order order, Payment payment, Reservation reservation) {
        // Rollback in reverse order

        try {
            // Cancel shipping (if started)
            shippingClient.cancel(order.getId());
        } catch (Exception e) {
            log.error("Failed to cancel shipping", e);
        }

        if (payment != null) {
            try {
                // Refund payment
                paymentClient.refund(payment.getId());
            } catch (Exception e) {
                log.error("Failed to refund payment", e);
            }
        }

        if (reservation != null) {
            try {
                // Release inventory
                inventoryClient.release(reservation.getId());
            } catch (Exception e) {
                log.error("Failed to release inventory", e);
            }
        }

        if (order != null) {
            // Cancel order
            order.setStatus("CANCELLED");
            orderRepository.save(order);
        }
    }
}
```

**مع State Machine:**

```java
@Configuration
@EnableStateMachine
public class OrderSagaStateMachine {

    @Bean
    public StateMachine<OrderState, OrderEvent> stateMachine() throws Exception {
        StateMachineBuilder.Builder<OrderState, OrderEvent> builder =
            StateMachineBuilder.builder();

        builder.configureStates()
            .withStates()
                .initial(OrderState.CREATED)
                .state(OrderState.INVENTORY_RESERVED)
                .state(OrderState.PAYMENT_PROCESSED)
                .state(OrderState.SHIPPED)
                .end(OrderState.COMPLETED)
                .end(OrderState.CANCELLED);

        builder.configureTransitions()
            .withExternal()
                .source(OrderState.CREATED)
                .target(OrderState.INVENTORY_RESERVED)
                .event(OrderEvent.RESERVE_INVENTORY)
                .action(reserveInventoryAction())
            .and()
            .withExternal()
                .source(OrderState.INVENTORY_RESERVED)
                .target(OrderState.PAYMENT_PROCESSED)
                .event(OrderEvent.PROCESS_PAYMENT)
                .action(processPaymentAction())
            .and()
            .withExternal()
                .source(OrderState.PAYMENT_PROCESSED)
                .target(OrderState.SHIPPED)
                .event(OrderEvent.SHIP_ORDER)
                .action(shipOrderAction());

        return builder.build();
    }
}
```

---

#### 3. Event Sourcing

```java
// Store events instead of state
@Entity
public class OrderEvent {
    @Id
    private Long id;
    private Long orderId;
    private String eventType;  // CREATED, PAYMENT_PROCESSED, SHIPPED
    private String payload;
    private LocalDateTime timestamp;
}

@Service
public class OrderEventStore {

    public void saveEvent(OrderEvent event) {
        eventRepository.save(event);
    }

    public Order rebuildOrderState(Long orderId) {
        List<OrderEvent> events = eventRepository.findByOrderId(orderId);

        Order order = new Order();
        for (OrderEvent event : events) {
            order.apply(event);  // Replay events
        }

        return order;
    }
}
```

---

### 31. اشرح Saga Pattern

(تم شرحه في السؤال السابق بالتفصيل)

**ملخص Saga Pattern:**

**Choreography vs Orchestration:**

| Feature | Choreography | Orchestration |
|---------|--------------|---------------|
| **Control** | Distributed | Centralized |
| **Coupling** | Loose | Tight |
| **Complexity** | Complex (many services) | Simple (one coordinator) |
| **Failure Handling** | Distributed | Centralized |
| **Monitoring** | Difficult | Easy |
| **Best For** | Simple sagas | Complex sagas |

---

### 32. ما هو CQRS و Event Sourcing؟

#### CQRS (Command Query Responsibility Segregation)

**التعريف:**
- فصل القراءة (Query) عن الكتابة (Command)
- Models مختلفة للقراءة والكتابة

**Traditional:**
```
┌─────────────────┐
│  Single Model   │
│  ┌───────────┐  │
│  │  Entity   │  │
│  │  + CRUD   │  │
│  └───────────┘  │
└────────┬────────┘
         ↓
    ┌─────────┐
    │Database │
    └─────────┘
```

**CQRS:**
```
Write Side (Commands):        Read Side (Queries):
┌──────────────┐             ┌──────────────┐
│ Write Model  │             │  Read Model  │
│ (Normalized) │             │ (Denormalized)│
└──────┬───────┘             └──────┬───────┘
       ↓                            ↓
┌──────────────┐  Events    ┌──────────────┐
│  Write DB    │───────────→│   Read DB    │
│   (MySQL)    │             │  (MongoDB)   │
└──────────────┘             └──────────────┘
```

**Implementation:**

```java
// Command Side
@Service
@RequiredArgsConstructor
public class OrderCommandService {

    private final OrderRepository orderRepository;
    private final EventPublisher eventPublisher;

    public void createOrder(CreateOrderCommand command) {
        // Validate
        validateOrder(command);

        // Create
        Order order = new Order(command);
        orderRepository.save(order);

        // Publish event
        eventPublisher.publish(new OrderCreatedEvent(order));
    }

    public void updateOrder(UpdateOrderCommand command) {
        Order order = orderRepository.findById(command.getOrderId())
            .orElseThrow();

        order.update(command);
        orderRepository.save(order);

        eventPublisher.publish(new OrderUpdatedEvent(order));
    }
}

// Query Side
@Service
@RequiredArgsConstructor
public class OrderQueryService {

    private final OrderReadRepository orderReadRepository;

    public OrderDTO getOrder(Long orderId) {
        return orderReadRepository.findById(orderId)
            .orElseThrow();
    }

    public List<OrderDTO> getUserOrders(Long userId) {
        // Optimized for reads
        return orderReadRepository.findByUserId(userId);
    }

    public OrderStatistics getStatistics() {
        // Pre-calculated
        return orderReadRepository.getStatistics();
    }
}

// Event Handler (Updates Read Model)
@Component
@RequiredArgsConstructor
public class OrderEventHandler {

    private final OrderReadRepository orderReadRepository;

    @EventListener
    public void handleOrderCreated(OrderCreatedEvent event) {
        OrderDTO dto = OrderDTO.fromEvent(event);
        orderReadRepository.save(dto);
    }

    @EventListener
    public void handleOrderUpdated(OrderUpdatedEvent event) {
        OrderDTO dto = orderReadRepository.findById(event.getOrderId())
            .orElseThrow();

        dto.update(event);
        orderReadRepository.save(dto);
    }
}
```

**Write Model (Normalized):**
```java
@Entity
@Table(name = "orders")
public class Order {
    @Id
    private Long id;
    private Long userId;
    private String status;
    private BigDecimal total;

    @OneToMany(mappedBy = "order")
    private List<OrderItem> items;
}

@Entity
@Table(name = "order_items")
public class OrderItem {
    @Id
    private Long id;

    @ManyToOne
    private Order order;

    private Long productId;
    private Integer quantity;
    private BigDecimal price;
}
```

**Read Model (Denormalized):**
```java
@Document(collection = "orders_read")
public class OrderDTO {
    private Long id;
    private Long userId;
    private String userName;  // Denormalized
    private String userEmail; // Denormalized
    private String status;
    private BigDecimal total;

    // Denormalized items
    private List<OrderItemDTO> items;

    // Pre-calculated
    private Integer totalItems;
    private LocalDateTime createdAt;
}
```

---

#### Event Sourcing

**التعريف:**
- تخزين Events بدلاً من الحالة الحالية
- إعادة بناء الحالة من Events

**Traditional:**
```sql
-- State-based
UPDATE orders
SET status = 'SHIPPED', shipped_at = NOW()
WHERE id = 123;

-- Lost history!
```

**Event Sourcing:**
```sql
-- Event-based
INSERT INTO order_events (order_id, event_type, data, timestamp)
VALUES (123, 'ORDER_SHIPPED', '{"shippedAt": "2024-01-15"}', NOW());

-- Full history preserved!
```

**Implementation:**

```java
// Event Store
@Entity
@Table(name = "order_events")
public class OrderEventEntity {
    @Id
    @GeneratedValue
    private Long id;

    private Long orderId;
    private String eventType;
    private String payload;  // JSON
    private Long version;
    private LocalDateTime timestamp;
}

@Repository
public interface OrderEventRepository extends JpaRepository<OrderEventEntity, Long> {
    List<OrderEventEntity> findByOrderIdOrderByVersionAsc(Long orderId);
}

// Event Store Service
@Service
@RequiredArgsConstructor
public class OrderEventStore {

    private final OrderEventRepository eventRepository;
    private final ObjectMapper objectMapper;

    public void saveEvent(Long orderId, OrderEvent event) {
        Long version = getNextVersion(orderId);

        OrderEventEntity entity = new OrderEventEntity();
        entity.setOrderId(orderId);
        entity.setEventType(event.getClass().getSimpleName());
        entity.setPayload(objectMapper.writeValueAsString(event));
        entity.setVersion(version);
        entity.setTimestamp(LocalDateTime.now());

        eventRepository.save(entity);
    }

    public Order rebuildOrder(Long orderId) {
        List<OrderEventEntity> events = eventRepository.findByOrderIdOrderByVersionAsc(orderId);

        if (events.isEmpty()) {
            throw new OrderNotFoundException();
        }

        Order order = new Order();
        for (OrderEventEntity eventEntity : events) {
            OrderEvent event = deserializeEvent(eventEntity);
            order.apply(event);  // Apply event to rebuild state
        }

        return order;
    }

    public List<OrderEvent> getEvents(Long orderId) {
        return eventRepository.findByOrderIdOrderByVersionAsc(orderId).stream()
            .map(this::deserializeEvent)
            .collect(Collectors.toList());
    }
}

// Order Aggregate
public class Order {
    private Long id;
    private Long userId;
    private String status;
    private BigDecimal total;
    private List<OrderItem> items = new ArrayList<>();
    private Long version = 0L;

    // Apply events to rebuild state
    public void apply(OrderEvent event) {
        if (event instanceof OrderCreatedEvent) {
            apply((OrderCreatedEvent) event);
        } else if (event instanceof ItemAddedEvent) {
            apply((ItemAddedEvent) event);
        } else if (event instanceof OrderShippedEvent) {
            apply((OrderShippedEvent) event);
        }
        version++;
    }

    private void apply(OrderCreatedEvent event) {
        this.id = event.getOrderId();
        this.userId = event.getUserId();
        this.status = "CREATED";
        this.total = BigDecimal.ZERO;
    }

    private void apply(ItemAddedEvent event) {
        OrderItem item = new OrderItem(
            event.getProductId(),
            event.getQuantity(),
            event.getPrice()
        );
        this.items.add(item);
        this.total = this.total.add(event.getPrice().multiply(
            BigDecimal.valueOf(event.getQuantity())
        ));
    }

    private void apply(OrderShippedEvent event) {
        this.status = "SHIPPED";
    }
}

// Command Handler
@Service
@RequiredArgsConstructor
public class OrderCommandHandler {

    private final OrderEventStore eventStore;

    public void handle(CreateOrderCommand command) {
        OrderCreatedEvent event = new OrderCreatedEvent(
            command.getOrderId(),
            command.getUserId(),
            LocalDateTime.now()
        );

        eventStore.saveEvent(command.getOrderId(), event);
    }

    public void handle(AddItemCommand command) {
        // Load current state
        Order order = eventStore.rebuildOrder(command.getOrderId());

        // Validate
        if (!order.canAddItem()) {
            throw new InvalidOperationException();
        }

        // Create event
        ItemAddedEvent event = new ItemAddedEvent(
            command.getProductId(),
            command.getQuantity(),
            command.getPrice()
        );

        // Save event
        eventStore.saveEvent(command.getOrderId(), event);
    }
}
```

**Snapshots (Optimization):**

```java
// لو Order عنده 10,000 events، إعادة البناء بطيئة!
// الحل: Snapshots

@Entity
@Table(name = "order_snapshots")
public class OrderSnapshot {
    @Id
    private Long orderId;
    private String state;  // JSON of Order
    private Long version;
    private LocalDateTime timestamp;
}

@Service
public class OrderEventStore {

    public Order rebuildOrder(Long orderId) {
        // ① Load latest snapshot
        OrderSnapshot snapshot = snapshotRepository.findLatestByOrderId(orderId);

        Order order;
        Long fromVersion;

        if (snapshot != null) {
            // Start from snapshot
            order = objectMapper.readValue(snapshot.getState(), Order.class);
            fromVersion = snapshot.getVersion() + 1;
        } else {
            // Start from scratch
            order = new Order();
            fromVersion = 0L;
        }

        // ② Load events after snapshot
        List<OrderEventEntity> events = eventRepository
            .findByOrderIdAndVersionGreaterThanOrderByVersionAsc(orderId, fromVersion);

        // ③ Apply remaining events
        for (OrderEventEntity event : events) {
            order.apply(deserializeEvent(event));
        }

        // ④ Create new snapshot if needed
        if (events.size() > 100) {
            createSnapshot(order);
        }

        return order;
    }

    private void createSnapshot(Order order) {
        OrderSnapshot snapshot = new OrderSnapshot();
        snapshot.setOrderId(order.getId());
        snapshot.setState(objectMapper.writeValueAsString(order));
        snapshot.setVersion(order.getVersion());
        snapshot.setTimestamp(LocalDateTime.now());

        snapshotRepository.save(snapshot);
    }
}
```

**مميزات Event Sourcing:**
- ✅ Full audit trail
- ✅ Temporal queries (كيف كان الـ Order في 2023؟)
- ✅ Replay events
- ✅ Debug & Analysis

**عيوب:**
- ❌ Complexity
- ❌ Storage (كل Events)
- ❌ Query complexity
- ❌ Schema evolution

---

## Deployment & Containerization

### 33. كيف تعمل Deploy لـ Microservices؟

**الاستراتيجيات:**

#### 1. Traditional Deployment (Bare Metal / VMs)

```bash
# Build JAR
mvn clean package

# Deploy to server
scp target/order-service.jar user@server:/opt/services/
ssh user@server "systemctl restart order-service"
```

**systemd service:**
```ini
# /etc/systemd/system/order-service.service
[Unit]
Description=Order Service
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/services
ExecStart=/usr/bin/java -jar order-service.jar
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

**عيوب:**
- ❌ يدوي ومعقد
- ❌ لا يوجد Isolation
- ❌ صعوبة في الـ Scaling

---

#### 2. Docker Containers (الأفضل)

**Dockerfile:**
```dockerfile
# Multi-stage build
FROM maven:3.8-openjdk-17 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# Runtime image
FROM openjdk:17-jdk-slim
WORKDIR /app

# Create non-root user
RUN addgroup --system appgroup && adduser --system --ingroup appgroup appuser

# Copy JAR
COPY --from=build /app/target/*.jar app.jar

# Change ownership
RUN chown -R appuser:appgroup /app

# Switch to non-root user
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=60s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

# Expose port
EXPOSE 8080

# Run application
ENTRYPOINT ["java", \
    "-XX:+UseContainerSupport", \
    "-XX:MaxRAMPercentage=75.0", \
    "-Djava.security.egd=file:/dev/./urandom", \
    "-jar", "app.jar"]
```

**Build & Run:**
```bash
# Build image
docker build -t order-service:1.0.0 .

# Run container
docker run -d \
  --name order-service \
  -p 8080:8080 \
  -e SPRING_PROFILES_ACTIVE=prod \
  -e DB_URL=jdbc:mysql://mysql:3306/orders \
  -e DB_USERNAME=root \
  -e DB_PASSWORD=secret \
  --memory=512m \
  --cpus=1 \
  order-service:1.0.0
```

---

#### 3. Docker Compose (Local Development)

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  # Databases
  mysql:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: orders
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

  mongodb:
    image: mongo:6
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  # Message Broker
  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      RABBITMQ_DEFAULT_USER: guest
      RABBITMQ_DEFAULT_PASS: guest

  # Service Discovery
  eureka-server:
    build: ./eureka-server
    ports:
      - "8761:8761"
    environment:
      SPRING_PROFILES_ACTIVE: docker

  # Config Server
  config-server:
    build: ./config-server
    ports:
      - "8888:8888"
    environment:
      SPRING_PROFILES_ACTIVE: docker
    depends_on:
      - eureka-server

  # API Gateway
  api-gateway:
    build: ./api-gateway
    ports:
      - "8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: docker
      EUREKA_CLIENT_SERVICEURL_DEFAULTZONE: http://eureka-server:8761/eureka/
    depends_on:
      - eureka-server
      - config-server

  # Microservices
  order-service:
    build: ./order-service
    ports:
      - "8081:8081"
    environment:
      SPRING_PROFILES_ACTIVE: docker
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/orders
      SPRING_DATASOURCE_USERNAME: root
      SPRING_DATASOURCE_PASSWORD: root
      EUREKA_CLIENT_SERVICEURL_DEFAULTZONE: http://eureka-server:8761/eureka/
    depends_on:
      mysql:
        condition: service_healthy
      eureka-server:
        condition: service_started
      rabbitmq:
        condition: service_started

  payment-service:
    build: ./payment-service
    ports:
      - "8082:8082"
    environment:
      SPRING_PROFILES_ACTIVE: docker
      EUREKA_CLIENT_SERVICEURL_DEFAULTZONE: http://eureka-server:8761/eureka/
    depends_on:
      - eureka-server
      - rabbitmq

  # Monitoring
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ADMIN_PASSWORD: admin
    volumes:
      - grafana-data:/var/lib/grafana

  zipkin:
    image: openzipkin/zipkin
    ports:
      - "9411:9411"

volumes:
  mysql-data:
  mongo-data:
  prometheus-data:
  grafana-data:
```

**Commands:**
```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f order-service

# Scale service
docker-compose up -d --scale order-service=3

# Stop all
docker-compose down

# Remove volumes
docker-compose down -v
```

---

#### 4. Kubernetes (Production)

**كيف يعمل:**
```
┌─────────────────────────────────────┐
│         Kubernetes Cluster          │
│  ┌───────────────────────────────┐  │
│  │  Master Node (Control Plane)  │  │
│  │  - API Server                 │  │
│  │  - Scheduler                  │  │
│  │  - Controller Manager         │  │
│  └───────────────────────────────┘  │
│                                     │
│  ┌───────────┐  ┌───────────┐     │
│  │  Worker   │  │  Worker   │     │
│  │  Node 1   │  │  Node 2   │     │
│  │           │  │           │     │
│  │  Pods     │  │  Pods     │     │
│  └───────────┘  └───────────┘     │
└─────────────────────────────────────┘
```

**Deployment.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
  labels:
    app: order-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
        version: v1
    spec:
      containers:
      - name: order-service
        image: myregistry.com/order-service:1.0.0
        ports:
        - containerPort: 8080
          name: http

        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "prod"
        - name: SPRING_DATASOURCE_URL
          valueFrom:
            configMapKeyRef:
              name: order-service-config
              key: database.url
        - name: SPRING_DATASOURCE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: order-service-secrets
              key: database.password

        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"

        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3

        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3

        volumeMounts:
        - name: config
          mountPath: /config
          readOnly: true

      volumes:
      - name: config
        configMap:
          name: order-service-config

---
apiVersion: v1
kind: Service
metadata:
  name: order-service
  labels:
    app: order-service
spec:
  type: ClusterIP
  selector:
    app: order-service
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
    name: http

---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: order-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

**ConfigMap.yaml:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: order-service-config
data:
  application.yml: |
    spring:
      application:
        name: order-service
    server:
      port: 8080
    management:
      endpoints:
        web:
          exposure:
            include: health,info,metrics,prometheus

  database.url: "jdbc:mysql://mysql-service:3306/orders"
```

**Secret.yaml:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: order-service-secrets
type: Opaque
data:
  database.password: cGFzc3dvcmQxMjM=  # base64 encoded
  jwt.secret: bXlzZWNyZXRrZXk=
```

**Ingress.yaml:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-gateway-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - api.example.com
    secretName: api-tls
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-gateway
            port:
              number: 80
```

**Deploy:**
```bash
# Apply configurations
kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml

# Check status
kubectl get pods
kubectl get services
kubectl get deployments

# View logs
kubectl logs -f deployment/order-service

# Scale manually
kubectl scale deployment order-service --replicas=5

# Update image
kubectl set image deployment/order-service \
  order-service=myregistry.com/order-service:2.0.0

# Rollback
kubectl rollout undo deployment/order-service

# Delete
kubectl delete -f deployment.yaml
```

---

### 34. اشرح Docker و Kubernetes

#### Docker

**التعريف:**
- Container Platform
- يحزم التطبيق مع dependencies
- Lightweight virtualization

**Docker vs VM:**
```
┌─────────────────────┐     ┌─────────────────────┐
│   Virtual Machine   │     │      Docker         │
├─────────────────────┤     ├─────────────────────┤
│  App A │ App B      │     │  App A │ App B      │
│  ────────────       │     │  ────────────       │
│  Bins/Libs          │     │  Bins/Libs          │
│  ────────────       │     │  ────────────       │
│  Guest OS           │     │  Docker Engine      │
│  ────────────       │     │  ────────────       │
│  Hypervisor         │     │  Host OS            │
│  ────────────       │     │  ────────────       │
│  Host OS            │     │  Hardware           │
│  ────────────       │     └─────────────────────┘
│  Hardware           │
└─────────────────────┘

Size: GBs                   Size: MBs
Boot: Minutes               Boot: Seconds
```

**Docker Commands:**
```bash
# Images
docker images                    # List images
docker pull nginx:latest         # Pull image
docker build -t myapp:1.0 .     # Build image
docker rmi myapp:1.0            # Remove image
docker tag myapp:1.0 myapp:latest

# Containers
docker ps                        # Running containers
docker ps -a                     # All containers
docker run -d -p 8080:8080 myapp:1.0
docker stop container_id
docker start container_id
docker restart container_id
docker rm container_id
docker logs -f container_id
docker exec -it container_id bash

# Networks
docker network create mynetwork
docker network ls
docker network inspect mynetwork

# Volumes
docker volume create mydata
docker volume ls
docker volume inspect mydata

# System
docker system df                 # Disk usage
docker system prune             # Clean up
```

**Best Practices:**
```dockerfile
# ✅ Good Dockerfile
FROM openjdk:17-jdk-slim

# Use specific versions
RUN apt-get update && \
    apt-get install -y curl=7.* && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Create non-root user
RUN useradd -m -u 1000 appuser

# Copy only necessary files
COPY --chown=appuser:appuser target/*.jar app.jar

# Don't run as root
USER appuser

# Use ENTRYPOINT + CMD
ENTRYPOINT ["java"]
CMD ["-jar", "app.jar"]

# Health check
HEALTHCHECK CMD curl -f http://localhost:8080/health || exit 1
```

---

#### Kubernetes (K8s)

**التعريف:**
- Container Orchestration Platform
- Automates deployment, scaling, management

**Core Concepts:**

**1. Pod:**
```yaml
# Smallest deployable unit
apiVersion: v1
kind: Pod
metadata:
  name: order-service-pod
spec:
  containers:
  - name: order-service
    image: order-service:1.0
    ports:
    - containerPort: 8080
```

**2. Deployment:**
```yaml
# Manages Pods + ReplicaSets
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
    spec:
      containers:
      - name: order-service
        image: order-service:1.0
```

**3. Service:**
```yaml
# Network access to Pods
apiVersion: v1
kind: Service
metadata:
  name: order-service
spec:
  selector:
    app: order-service
  ports:
  - port: 80
    targetPort: 8080
  type: LoadBalancer
```

**Service Types:**
- **ClusterIP**: Internal only (default)
- **NodePort**: External access via Node IP
- **LoadBalancer**: Cloud load balancer
- **ExternalName**: DNS CNAME

**4. ConfigMap & Secret:**
```yaml
# ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database.url: "mysql://db:3306"
  log.level: "INFO"

---
# Secret
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  password: cGFzc3dvcmQ=  # base64
```

**5. Ingress:**
```yaml
# HTTP/HTTPS routing
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /orders
        pathType: Prefix
        backend:
          service:
            name: order-service
            port:
              number: 80
```

**6. Namespace:**
```yaml
# Isolate resources
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

**kubectl Commands:**
```bash
# Cluster info
kubectl cluster-info
kubectl get nodes

# Pods
kubectl get pods
kubectl get pods -n production
kubectl describe pod pod-name
kubectl logs pod-name
kubectl logs -f pod-name -c container-name
kubectl exec -it pod-name -- bash

# Deployments
kubectl get deployments
kubectl describe deployment deployment-name
kubectl scale deployment order-service --replicas=5
kubectl set image deployment/order-service \
  order-service=order-service:2.0

# Services
kubectl get services
kubectl describe service service-name
kubectl expose deployment order-service --type=LoadBalancer --port=80

# Apply/Delete
kubectl apply -f deployment.yaml
kubectl delete -f deployment.yaml
kubectl delete pod pod-name

# Rollout
kubectl rollout status deployment/order-service
kubectl rollout history deployment/order-service
kubectl rollout undo deployment/order-service

# Debug
kubectl get events
kubectl get all
kubectl top nodes
kubectl top pods
```

---

### 35. ما هو Container Orchestration؟

**التعريف:**
- إدارة تلقائية للـ Containers
- Deployment, Scaling, Networking, Monitoring

**مشاكل بدون Orchestration:**
```
❌ Manual deployment
❌ No auto-scaling
❌ No load balancing
❌ No self-healing
❌ Manual updates
```

**مع Orchestration (Kubernetes):**
```
✅ Automated deployment
✅ Auto-scaling (HPA)
✅ Built-in load balancing
✅ Self-healing (restart failed pods)
✅ Rolling updates & rollbacks
✅ Service discovery
✅ Secret management
```

**Features:**

#### 1. Self-Healing
```yaml
# Kubernetes restarts failed pods automatically
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  failureThreshold: 3  # Restart after 3 failures
```

#### 2. Auto-Scaling

**Horizontal Pod Autoscaler:**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: order-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

**Vertical Pod Autoscaler:**
```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: order-service-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service
  updatePolicy:
    updateMode: "Auto"
```

#### 3. Rolling Updates

```bash
# Update image
kubectl set image deployment/order-service \
  order-service=order-service:2.0

# Kubernetes performs rolling update:
# 1. Create new pods with v2.0
# 2. Wait for them to be ready
# 3. Terminate old pods with v1.0
# 4. Zero downtime!
```

```yaml
# Control update strategy
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Max extra pods during update
      maxUnavailable: 0  # Min available pods
```

#### 4. Service Discovery

```yaml
# Services get DNS names automatically
# order-service.default.svc.cluster.local

# From another pod:
curl http://order-service/api/orders
# Kubernetes routes to available pods
```

#### 5. Load Balancing

```yaml
apiVersion: v1
kind: Service
metadata:
  name: order-service
spec:
  selector:
    app: order-service
  ports:
  - port: 80
    targetPort: 8080
  type: LoadBalancer
# Distributes traffic across all pods
```

---

**Orchestration Platforms:**

| Platform | Type | Best For |
|----------|------|----------|
| **Kubernetes** | Open Source | Production, Large scale |
| **Docker Swarm** | Open Source | Simple, small deployments |
| **AWS ECS** | Managed | AWS-only |
| **AWS EKS** | Managed K8s | AWS + Kubernetes |
| **Azure AKS** | Managed K8s | Azure + Kubernetes |
| **Google GKE** | Managed K8s | GCP + Kubernetes |
| **Nomad** | Open Source | Multi-cloud |

---

### 36. كيف تطبق CI/CD للـ Microservices؟

**CI/CD Pipeline:**

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│   Code   │ →  │  Build   │ →  │   Test   │ →  │  Deploy  │
│  Commit  │    │ & Package│    │          │    │          │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
```

---

#### GitHub Actions Example

**.github/workflows/ci-cd.yml:**
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: docker.io
  IMAGE_NAME: myorg/order-service

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Set up JDK 17
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'
        cache: maven

    - name: Run tests
      run: mvn clean test

    - name: Generate test coverage
      run: mvn jacoco:report

    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3

  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.event_name == 'push'

    steps:
    - uses: actions/checkout@v3

    - name: Set up JDK 17
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'
        cache: maven

    - name: Build with Maven
      run: mvn clean package -DskipTests

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2

    - name: Log in to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v4
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
        tags: |
          type=ref,event=branch
          type=sha,prefix={{branch}}-
          type=semver,pattern={{version}}

    - name: Build and push Docker image
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop'
    environment: staging

    steps:
    - uses: actions/checkout@v3

    - name: Configure kubectl
      uses: azure/k8s-set-context@v3
      with:
        kubeconfig: ${{ secrets.KUBE_CONFIG_STAGING }}

    - name: Deploy to Staging
      run: |
        kubectl set image deployment/order-service \
          order-service=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:develop-${{ github.sha }}
        kubectl rollout status deployment/order-service

    - name: Run smoke tests
      run: |
        curl -f https://staging-api.example.com/health || exit 1

  deploy-production:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production

    steps:
    - uses: actions/checkout@v3

    - name: Configure kubectl
      uses: azure/k8s-set-context@v3
      with:
        kubeconfig: ${{ secrets.KUBE_CONFIG_PROD }}

    - name: Deploy to Production
      run: |
        kubectl set image deployment/order-service \
          order-service=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:main-${{ github.sha }} \
          --namespace=production
        kubectl rollout status deployment/order-service -n production

    - name: Notify Slack
      uses: 8398a7/action-slack@v3
      with:
        status: ${{ job.status }}
        text: 'Order Service deployed to production'
        webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

---

#### GitLab CI Example

**.gitlab-ci.yml:**
```yaml
stages:
  - test
  - build
  - deploy

variables:
  MAVEN_OPTS: "-Dmaven.repo.local=$CI_PROJECT_DIR/.m2/repository"
  DOCKER_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA

cache:
  paths:
    - .m2/repository

test:
  stage: test
  image: maven:3.8-openjdk-17
  script:
    - mvn clean test
    - mvn jacoco:report
  artifacts:
    reports:
      junit: target/surefire-reports/TEST-*.xml
      coverage_report:
        coverage_format: cobertura
        path: target/site/jacoco/jacoco.xml

build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - mvn clean package -DskipTests
    - docker build -t $DOCKER_IMAGE .
    - docker push $DOCKER_IMAGE
  only:
    - main
    - develop

deploy:staging:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl config use-context staging
    - kubectl set image deployment/order-service order-service=$DOCKER_IMAGE
    - kubectl rollout status deployment/order-service
  environment:
    name: staging
    url: https://staging-api.example.com
  only:
    - develop

deploy:production:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl config use-context production
    - kubectl set image deployment/order-service order-service=$DOCKER_IMAGE -n production
    - kubectl rollout status deployment/order-service -n production
  environment:
    name: production
    url: https://api.example.com
  when: manual
  only:
    - main
```

---

#### Jenkins Pipeline

**Jenkinsfile:**
```groovy
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "myorg/order-service"
        DOCKER_TAG = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                sh 'mvn clean test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                    jacoco(
                        execPattern: 'target/jacoco.exec',
                        classPattern: 'target/classes',
                        sourcePattern: 'src/main/java'
                    )
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-credentials') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push('latest')
                    }
                }
            }
        }

        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                sh """
                    kubectl set image deployment/order-service \
                        order-service=${DOCKER_IMAGE}:${DOCKER_TAG}
                    kubectl rollout status deployment/order-service
                """
            }
        }

        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to production?'
                sh """
                    kubectl set image deployment/order-service \
                        order-service=${DOCKER_IMAGE}:${DOCKER_TAG} \
                        --namespace=production
                    kubectl rollout status deployment/order-service -n production
                """
            }
        }
    }

    post {
        success {
            slackSend(
                color: 'good',
                message: "Deployment successful: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
            )
        }
        failure {
            slackSend(
                color: 'danger',
                message: "Deployment failed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
            )
        }
    }
}
```

---

#### Best Practices

**1. Automated Testing:**
```yaml
# Unit tests
- mvn test

# Integration tests
- mvn verify

# Contract tests (Pact)
- mvn pact:verify

# E2E tests
- npm run test:e2e
```

**2. Code Quality:**
```yaml
# SonarQube
- sonar-scanner \
    -Dsonar.projectKey=order-service \
    -Dsonar.sources=src \
    -Dsonar.host.url=$SONAR_URL \
    -Dsonar.login=$SONAR_TOKEN
```

**3. Security Scanning:**
```yaml
# Dependency check
- mvn dependency-check:check

# Container scanning (Trivy)
- trivy image myorg/order-service:latest

# SAST (Static Analysis)
- semgrep --config=auto .
```

**4. Blue/Green Deployment:**
```yaml
# Deploy to green environment
kubectl apply -f deployment-green.yaml

# Test green
curl https://green.example.com/health

# Switch traffic to green
kubectl patch service order-service -p '{"spec":{"selector":{"version":"green"}}}'

# Keep blue for rollback
```

**5. Canary Deployment:**
```yaml
# Istio VirtualService
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: order-service
spec:
  hosts:
  - order-service
  http:
  - route:
    - destination:
        host: order-service
        subset: v1
      weight: 90
    - destination:
        host: order-service
        subset: v2
      weight: 10  # 10% traffic to canary
```

---

## أسئلة متقدمة

### 37. كيف تتعامل مع Data Consistency في Distributed System؟

**التحديات:**

```
Order Service: Create Order ✅
Payment Service: Charge Card ✅
Inventory Service: Reserve Item ❌ (Failed)

مشكلة: Inconsistent state!
```

**الحلول:**

#### 1. Eventual Consistency

**Accept temporary inconsistency:**

```java
@Service
public class OrderService {

    public Order createOrder(OrderRequest request) {
        // ① Create order immediately
        Order order = orderRepository.save(new Order(request, "PENDING"));

        // ② Publish event (async)
        eventPublisher.publish(new OrderCreatedEvent(order));

        // ③ Return (order may not be fully processed yet)
        return order;
    }
}

// Eventually, all services sync
@Component
public class OrderEventHandler {

    @EventListener
    public void onOrderCreated(OrderCreatedEvent event) {
        // Process payment (eventually)
        // Update inventory (eventually)
        // Send email (eventually)
    }
}
```

**Use Cases:**
- ✅ Social media likes/views
- ✅ Analytics
- ✅ Notifications
- ❌ Banking transactions
- ❌ Inventory for limited items

---

#### 2. Saga Pattern (شرحناه سابقاً)

**Compensating Transactions:**

```java
public class OrderSaga {

    public void execute() {
        try {
            createOrder();        // ① T1
            reserveInventory();   // ② T2
            processPayment();     // ③ T3 ❌ Failed
        } catch (Exception e) {
            // Compensate
            releaseInventory();   // ② C2
            cancelOrder();        // ① C1
        }
    }
}
```

---

#### 3. Event Sourcing

**Store all changes:**

```java
// Instead of: UPDATE order SET status='PAID'
// Store: INSERT INTO events (order_id, event_type, data)

@Service
public class OrderEventStore {

    public void apply(OrderEvent event) {
        eventRepository.save(event);

        // Update read model eventually
        updateReadModel(event);
    }

    // Rebuild state from events
    public Order getOrder(Long id) {
        List<OrderEvent> events = eventRepository.findByOrderId(id);

        Order order = new Order();
        events.forEach(order::apply);

        return order;
    }
}
```

---

#### 4. CQRS (Command Query Responsibility Segregation)

**Separate read/write models:**

```java
// Write Model (Strong consistency)
@Service
public class OrderCommandService {
    public void createOrder(CreateOrderCommand cmd) {
        Order order = new Order(cmd);
        orderRepository.save(order);

        eventPublisher.publish(new OrderCreatedEvent(order));
    }
}

// Read Model (Eventual consistency)
@Service
public class OrderQueryService {
    public OrderDTO getOrder(Long id) {
        // Read from denormalized view
        return orderReadRepository.findById(id);
    }
}
```

---

#### 5. Distributed Locks

**For critical sections:**

```java
@Service
public class InventoryService {

    @Autowired
    private RedissonClient redisson;

    public void reserveItem(Long productId, int quantity) {
        RLock lock = redisson.getLock("product:" + productId);

        try {
            // Acquire lock
            boolean acquired = lock.tryLock(10, 30, TimeUnit.SECONDS);

            if (!acquired) {
                throw new LockAcquisitionException();
            }

            // Critical section
            Product product = productRepository.findById(productId).orElseThrow();

            if (product.getStock() < quantity) {
                throw new OutOfStockException();
            }

            product.setStock(product.getStock() - quantity);
            productRepository.save(product);

        } finally {
            lock.unlock();
        }
    }
}
```

---

#### 6. Idempotency

**Handle duplicate requests:**

```java
@Service
public class PaymentService {

    @Autowired
    private RedisTemplate<String, String> redis;

    public Payment processPayment(PaymentRequest request) {
        String idempotencyKey = request.getIdempotencyKey();

        // Check if already processed
        String cachedResult = redis.opsForValue().get("payment:" + idempotencyKey);

        if (cachedResult != null) {
            return objectMapper.readValue(cachedResult, Payment.class);
        }

        // Process payment
        Payment payment = externalPaymentApi.charge(request);

        // Cache result
        redis.opsForValue().set(
            "payment:" + idempotencyKey,
            objectMapper.writeValueAsString(payment),
            24,
            TimeUnit.HOURS
        );

        return payment;
    }
}
```

**Client usage:**
```java
PaymentRequest request = new PaymentRequest();
request.setIdempotencyKey(UUID.randomUUID().toString());
request.setAmount(100.00);

// Safe to retry
Payment payment = paymentService.processPayment(request);
```

---

### 38. ما هو CAP Theorem؟

**التعريف:**

في Distributed System، يمكنك اختيار اثنين فقط من:

- **C**onsistency: كل القراءات ترجع أحدث كتابة
- **A**vailability: كل الطلبات تحصل على رد (لو كان صحيح أو خاطئ)
- **P**artition Tolerance: النظام يستمر رغم انقطاع الشبكة

```
       C
      / \
     /   \
    /     \
   /  CP   \
  / (Banks)\
 /_________\
A     AP     P
   (Social
    Media)

CA: لا يمكن في Distributed Systems
    (Network partitions will happen)
```

**أمثلة:**

#### CP (Consistency + Partition Tolerance)

**التضحية بـ Availability:**

```java
// Banking system
@Service
public class BankingService {

    public void transfer(Transfer transfer) {
        // Must be consistent
        // If partition occurs, system unavailable

        Transaction tx = beginTransaction();

        try {
            Account from = getAccount(transfer.getFromAccount());
            Account to = getAccount(transfer.getToAccount());

            // Both operations must succeed
            from.debit(transfer.getAmount());
            to.credit(transfer.getAmount());

            tx.commit();

        } catch (PartitionException e) {
            // Cannot proceed - return error
            throw new ServiceUnavailableException("Cannot guarantee consistency");
        }
    }
}
```

**Systems:**
- MongoDB (with strong consistency)
- HBase
- Redis (single instance)

---

#### AP (Availability + Partition Tolerance)

**التضحية بـ Consistency:**

```java
// Social media likes
@Service
public class LikeService {

    public void likePost(Long postId, Long userId) {
        // Always accept request
        // Eventual consistency

        likeRepository.save(new Like(postId, userId));

        // Publish event
        eventPublisher.publish(new PostLikedEvent(postId));

        // Count may be inconsistent temporarily
        // But system is always available
    }

    public long getLikeCount(Long postId) {
        // May return stale data
        return likeCountCache.get(postId);
    }
}
```

**Systems:**
- Cassandra
- DynamoDB
- CouchDB
- Riak

---

#### Practical Approach

**Different consistency for different data:**

```java
@Service
public class EcommerceService {

    // CP: Critical data
    @Transactional
    public void checkout(Order order) {
        // Must be consistent
        Payment payment = processPayment(order);
        Inventory inventory = reserveInventory(order);

        // If partition: fail fast
        if (!payment.isSuccessful()) {
            throw new PaymentFailedException();
        }
    }

    // AP: Non-critical data
    public void trackView(Long productId) {
        // Eventual consistency OK
        viewCountService.increment(productId);
    }

    // AP: Recommendations
    public List<Product> getRecommendations(Long userId) {
        // Stale data OK
        return recommendationCache.get(userId);
    }
}
```

---

### 39. اشرح Service Mesh (Istio, Linkerd)

**التعريف:**

**Service Mesh:**
- Infrastructure layer للـ service-to-service communication
- يضيف observability, security, reliability
- بدون تغيير الكود

**بدون Service Mesh:**
```
Service A → Code: Retry, Circuit Breaker, TLS → Service B
           → Code: Metrics, Tracing
```

**مع Service Mesh:**
```
Service A → Sidecar Proxy → Service B
              ↓
        (Handles everything)
```

---

#### Architecture

```
┌─────────────────────────────────────────┐
│          Control Plane                  │
│  - Configuration                        │
│  - Certificate Authority                │
│  - Telemetry Collection                 │
└─────────────────────────────────────────┘
              ↓ (Configuration)
┌──────────────────────┬──────────────────┐
│   Pod 1              │   Pod 2          │
│  ┌────────────┐      │  ┌────────────┐  │
│  │  Service A │      │  │  Service B │  │
│  └────────────┘      │  └────────────┘  │
│        ↕             │        ↕         │
│  ┌────────────┐      │  ┌────────────┐  │
│  │   Proxy    │◄────────►│   Proxy    │  │
│  │ (Envoy)    │      │  │  (Envoy)   │  │
│  └────────────┘      │  └────────────┘  │
└──────────────────────┴──────────────────┘
```

---

#### Istio Example

**Installation:**
```bash
# Install Istio
istioctl install --set profile=demo

# Enable sidecar injection
kubectl label namespace default istio-injection=enabled
```

**Traffic Management:**

```yaml
# VirtualService
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: order-service
spec:
  hosts:
  - order-service
  http:
  # Canary: 90% v1, 10% v2
  - match:
    - headers:
        user-type:
          exact: beta
    route:
    - destination:
        host: order-service
        subset: v2
  - route:
    - destination:
        host: order-service
        subset: v1
      weight: 90
    - destination:
        host: order-service
        subset: v2
      weight: 10

---
# DestinationRule
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: order-service
spec:
  host: order-service
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 50
        maxRequestsPerConnection: 2
    outlierDetection:
      consecutiveErrors: 5
      interval: 30s
      baseEjectionTime: 30s
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

**Circuit Breaker:**
```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: payment-service
spec:
  host: payment-service
  trafficPolicy:
    outlierDetection:
      consecutiveGatewayErrors: 5
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
```

**Retry:**
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: payment-service
spec:
  hosts:
  - payment-service
  http:
  - route:
    - destination:
        host: payment-service
    retries:
      attempts: 3
      perTryTimeout: 2s
      retryOn: 5xx,gateway-error,reset,connect-failure
```

**Timeout:**
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: order-service
spec:
  hosts:
  - order-service
  http:
  - route:
    - destination:
        host: order-service
    timeout: 10s
```

**mTLS:**
```yaml
# Require mTLS for all services
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: default
spec:
  mtls:
    mode: STRICT
```

**Observability:**
```bash
# Metrics (Prometheus)
kubectl apply -f samples/addons/prometheus.yaml

# Tracing (Jaeger)
kubectl apply -f samples/addons/jaeger.yaml

# Visualization (Kiali)
kubectl apply -f samples/addons/kiali.yaml

# Access Kiali
istioctl dashboard kiali
```

---

#### مميزات Service Mesh

**1. Traffic Management:**
- A/B Testing
- Canary Deployments
- Blue/Green
- Circuit Breaking
- Retries & Timeouts

**2. Security:**
- mTLS (automatic)
- Authorization policies
- Certificate rotation

**3. Observability:**
- Metrics (RED - Rate, Errors, Duration)
- Distributed Tracing
- Access Logs

**4. Zero Code Changes:**
```java
// Your code remains simple
@RestController
public class OrderController {

    @GetMapping("/orders/{id}")
    public Order getOrder(@PathVariable Long id) {
        return orderService.getOrder(id);
    }
}

// Service Mesh handles:
// - Retries
// - Circuit breaking
// - TLS
// - Metrics
// - Tracing
```

---

**متى تستخدم Service Mesh؟**

✅ Use if:
- Many microservices (10+)
- Complex traffic management needed
- Need mTLS everywhere
- Want detailed observability

❌ Don't use if:
- Few services (< 5)
- Simple architecture
- Limited resources
- Team unfamiliar with K8s

---

### 40. كيف تقسم Monolithic Application إلى Microservices؟ (Strangler Fig Pattern)

**الخطوات العملية:**

#### 1️⃣ التحليل والتخطيط

```markdown
## خطوات التقسيم

1. **Domain Analysis:**
   - حدد الـ Bounded Contexts
   - استخدم Domain-Driven Design (DDD)
   - حدد الـ Business Capabilities

2. **Identify Seams:**
   - ابحث عن نقاط الفصل الطبيعية
   - Modules قائمة بذاتها
   - Low coupling areas
```

#### 2️⃣ Strangler Fig Pattern

```java
// الـ Monolith الأصلي
@RestController
public class MonolithController {

    @GetMapping("/orders")
    public List<Order> getOrders() {
        return orderService.getAllOrders();
    }

    @GetMapping("/customers")
    public List<Customer> getCustomers() {
        return customerService.getAllCustomers();
    }
}

// المرحلة 1: إنشاء Order Microservice
@Service
public class OrderMigrationService {

    @Value("${order.service.enabled:false}")
    private boolean useOrderService;

    private final RestTemplate restTemplate;
    private final LegacyOrderService legacyService;

    public List<Order> getOrders() {
        if (useOrderService) {
            // استخدم الـ Microservice الجديد
            return restTemplate.getForObject(
                "http://order-service/api/orders",
                List.class
            );
        } else {
            // استخدم الكود القديم
            return legacyService.getAllOrders();
        }
    }
}
```

#### 3️⃣ Database Decomposition

```sql
-- الخطوة 1: نسخ البيانات
-- من Monolith Database إلى Order Service Database

-- Monolith Database
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    customer_id BIGINT,
    total DECIMAL,
    status VARCHAR(50),
    created_at TIMESTAMP
);

-- Order Service Database (جديد)
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    customer_id BIGINT,
    total DECIMAL,
    status VARCHAR(50),
    created_at TIMESTAMP
);

-- الخطوة 2: Data Synchronization
```

```java
@Service
public class DataSyncService {

    @Scheduled(fixedDelay = 60000) // كل دقيقة
    public void syncOrders() {
        // اقرأ من الـ Monolith
        List<Order> monolithOrders = monolithRepository.findAll();

        // اكتب للـ Microservice
        for (Order order : monolithOrders) {
            microserviceRepository.save(order);
        }
    }
}
```

#### 4️⃣ API Gateway للتوجيه

```java
@Configuration
public class GatewayRoutingConfig {

    @Bean
    public RouteLocator routes(RouteLocatorBuilder builder) {
        return builder.routes()
            // توجيه Orders للـ Microservice الجديد
            .route("orders", r -> r.path("/api/orders/**")
                .uri("lb://ORDER-SERVICE"))

            // باقي الطلبات للـ Monolith
            .route("legacy", r -> r.path("/**")
                .uri("lb://MONOLITH-APP"))
            .build();
    }
}
```

#### 5️⃣ استراتيجية الترحيل

```java
// Anti-Corruption Layer (ACL)
@Service
public class OrderAdapter {

    // تحويل من Monolith Model إلى Microservice Model
    public OrderDTO convertToDTO(LegacyOrder legacy) {
        return OrderDTO.builder()
            .orderId(legacy.getId())
            .customerId(legacy.getCustomerId())
            .amount(legacy.getTotalAmount())
            .status(convertStatus(legacy.getStatusCode()))
            .build();
    }

    private OrderStatus convertStatus(int code) {
        return switch(code) {
            case 0 -> OrderStatus.PENDING;
            case 1 -> OrderStatus.CONFIRMED;
            case 2 -> OrderStatus.SHIPPED;
            default -> OrderStatus.UNKNOWN;
        };
    }
}
```

---

### 41. ما هي Best Practices للـ Microservices؟

#### 1️⃣ Design Principles

```markdown
## 1. Single Responsibility
- كل service مسؤول عن business capability واحدة
- High Cohesion, Low Coupling

## 2. Database per Service
- كل service له database خاص
- لا shared database بين services

## 3. API First Design
- صمم الـ API قبل التطبيق
- استخدم OpenAPI/Swagger

## 4. Decentralized Governance
- كل team يختار tech stack المناسب
- Autonomy للفرق
```

#### 2️⃣ Communication Best Practices

```java
// 1. استخدم Async Communication للـ Non-Critical Operations
@Service
public class OrderService {

    private final KafkaTemplate<String, OrderEvent> kafka;

    public Order createOrder(OrderRequest request) {
        Order order = orderRepository.save(new Order(request));

        // Async notification - لا تنتظر
        kafka.send("order-created",
            new OrderEvent(order.getId(), order.getCustomerId())
        );

        return order;
    }
}

// 2. استخدم Circuit Breaker للحماية
@Service
public class PaymentService {

    @CircuitBreaker(name = "payment", fallbackMethod = "paymentFallback")
    @Retry(name = "payment", fallbackMethod = "paymentFallback")
    public Payment processPayment(Long orderId) {
        return paymentClient.process(orderId);
    }

    private Payment paymentFallback(Long orderId, Exception e) {
        log.error("Payment failed for order: {}", orderId, e);
        return Payment.failed(orderId);
    }
}

// 3. استخدم Idempotency للـ APIs
@RestController
public class OrderController {

    @PostMapping("/orders")
    public ResponseEntity<Order> createOrder(
        @RequestHeader("Idempotency-Key") String key,
        @RequestBody OrderRequest request
    ) {
        // تحقق من الـ key أولاً
        Optional<Order> existing = orderRepository.findByIdempotencyKey(key);
        if (existing.isPresent()) {
            return ResponseEntity.ok(existing.get());
        }

        Order order = orderService.createOrder(request, key);
        return ResponseEntity.status(HttpStatus.CREATED).body(order);
    }
}
```

#### 3️⃣ Observability Best Practices

```yaml
# application.yml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  metrics:
    export:
      prometheus:
        enabled: true
  tracing:
    sampling:
      probability: 1.0 # 100% في Dev, 10% في Prod

logging:
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n - [TraceId: %X{traceId}] [SpanId: %X{spanId}]"
```

```java
// Structured Logging
@Slf4j
@Service
public class OrderService {

    public Order createOrder(OrderRequest request) {
        log.info("Creating order: customerId={}, items={}",
            request.getCustomerId(),
            request.getItems().size()
        );

        try {
            Order order = orderRepository.save(new Order(request));

            log.info("Order created successfully: orderId={}, total={}",
                order.getId(),
                order.getTotal()
            );

            return order;
        } catch (Exception e) {
            log.error("Failed to create order: customerId={}, error={}",
                request.getCustomerId(),
                e.getMessage(),
                e
            );
            throw e;
        }
    }
}
```

#### 4️⃣ Security Best Practices

```java
// 1. JWT Validation في كل Service
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/actuator/**").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt.decoder(jwtDecoder()))
            );
        return http.build();
    }
}

// 2. Rate Limiting
@Configuration
public class RateLimitConfig {

    @Bean
    public KeyResolver userKeyResolver() {
        return exchange -> Mono.just(
            exchange.getRequest()
                .getHeaders()
                .getFirst("X-User-Id")
        );
    }
}
```

#### 5️⃣ Testing Best Practices

```java
// 1. Contract Testing مع Pact
@ExtendWith(PactConsumerTestExt.class)
@PactTestFor(providerName = "order-service", port = "8080")
public class PaymentServiceContractTest {

    @Pact(consumer = "payment-service")
    public RequestResponsePact createOrderPact(PactDslWithProvider builder) {
        return builder
            .given("order exists")
            .uponReceiving("get order by id")
            .path("/api/orders/123")
            .method("GET")
            .willRespondWith()
            .status(200)
            .body(new PactDslJsonBody()
                .stringType("id", "123")
                .numberType("total", 100.00)
            )
            .toPact();
    }
}

// 2. Integration Testing مع Testcontainers
@SpringBootTest
@Testcontainers
class OrderServiceIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
        .withDatabaseName("testdb");

    @Container
    static KafkaContainer kafka = new KafkaContainer(
        DockerImageName.parse("confluentinc/cp-kafka:7.4.0")
    );

    @Test
    void shouldCreateOrder() {
        // Test implementation
    }
}
```

---

### 42. كيف تتعامل مع Versioning للـ APIs؟

#### استراتيجية 1️⃣: URI Versioning

```java
@RestController
@RequestMapping("/api/v1/orders")
public class OrderControllerV1 {

    @GetMapping("/{id}")
    public OrderDTOV1 getOrder(@PathVariable Long id) {
        return OrderDTOV1.builder()
            .id(order.getId())
            .total(order.getTotal())
            .build();
    }
}

@RestController
@RequestMapping("/api/v2/orders")
public class OrderControllerV2 {

    @GetMapping("/{id}")
    public OrderDTOV2 getOrder(@PathVariable Long id) {
        return OrderDTOV2.builder()
            .orderId(order.getId())
            .amount(order.getTotal())
            .currency("USD") // حقل جديد
            .items(order.getItems()) // حقل جديد
            .build();
    }
}
```

#### استراتيجية 2️⃣: Header Versioning

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    @GetMapping(value = "/{id}", headers = "API-Version=1")
    public OrderDTOV1 getOrderV1(@PathVariable Long id) {
        return convertToV1(orderService.getOrder(id));
    }

    @GetMapping(value = "/{id}", headers = "API-Version=2")
    public OrderDTOV2 getOrderV2(@PathVariable Long id) {
        return convertToV2(orderService.getOrder(id));
    }
}
```

#### استراتيجية 3️⃣: Content Negotiation

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    @GetMapping(value = "/{id}", produces = "application/vnd.company.v1+json")
    public OrderDTOV1 getOrderV1(@PathVariable Long id) {
        return convertToV1(orderService.getOrder(id));
    }

    @GetMapping(value = "/{id}", produces = "application/vnd.company.v2+json")
    public OrderDTOV2 getOrderV2(@PathVariable Long id) {
        return convertToV2(orderService.getOrder(id));
    }
}
```

#### استراتيجية 4️⃣: Backward Compatibility

```java
// DTO مع Optional Fields للـ Backward Compatibility
public class OrderDTO {

    private Long id; // V1
    private Double total; // V1

    @JsonInclude(JsonInclude.Include.NON_NULL)
    private String currency; // V2 - optional

    @JsonInclude(JsonInclude.Include.NON_NULL)
    private List<OrderItem> items; // V2 - optional

    @JsonInclude(JsonInclude.Include.NON_NULL)
    private CustomerInfo customer; // V3 - optional
}

// استخدام Adapter Pattern
@Service
public class OrderVersionAdapter {

    public Object adaptToVersion(Order order, String version) {
        return switch(version) {
            case "1" -> convertToV1(order);
            case "2" -> convertToV2(order);
            case "3" -> convertToV3(order);
            default -> convertToLatest(order);
        };
    }
}
```

#### Deprecation Strategy

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    /**
     * @deprecated استخدم v2 بدلاً منه
     * سيتم إيقافه في 2024-12-31
     */
    @Deprecated
    @GetMapping(value = "/{id}", headers = "API-Version=1")
    public ResponseEntity<OrderDTOV1> getOrderV1(@PathVariable Long id) {
        return ResponseEntity
            .status(HttpStatus.OK)
            .header("Warning", "299 - \"API v1 is deprecated. Use v2. Sunset: 2024-12-31\"")
            .header("Sunset", "Wed, 31 Dec 2024 23:59:59 GMT")
            .body(convertToV1(orderService.getOrder(id)));
    }
}
```

---

## الأسئلة العملية (Scenario-Based Questions)

### 43. كيف تتعامل مع Service Failure؟

#### السيناريو: Order Service معطل ولا يستجيب

```java
// 1. Circuit Breaker Pattern
@Service
public class OrderService {

    @CircuitBreaker(
        name = "orderService",
        fallbackMethod = "getOrderFallback"
    )
    @Retry(name = "orderService")
    @TimeLimiter(name = "orderService")
    public CompletableFuture<Order> getOrder(Long id) {
        return CompletableFuture.supplyAsync(() ->
            orderClient.getOrder(id)
        );
    }

    private CompletableFuture<Order> getOrderFallback(
        Long id,
        Exception e
    ) {
        log.error("Order service failed, using cached data", e);

        // محاولة 1: استخدم Cache
        return cacheService.getOrder(id)
            .map(CompletableFuture::completedFuture)
            // محاولة 2: استخدم بيانات أساسية
            .orElseGet(() -> CompletableFuture.completedFuture(
                Order.builder()
                    .id(id)
                    .status("UNKNOWN")
                    .message("Order service unavailable")
                    .build()
            ));
    }
}

// 2. Health Checks
@Component
public class OrderServiceHealthIndicator implements HealthIndicator {

    private final OrderClient orderClient;

    @Override
    public Health health() {
        try {
            orderClient.ping();
            return Health.up()
                .withDetail("service", "order-service")
                .withDetail("status", "available")
                .build();
        } catch (Exception e) {
            return Health.down()
                .withDetail("service", "order-service")
                .withDetail("error", e.getMessage())
                .build();
        }
    }
}

// 3. Graceful Degradation
@Service
public class CheckoutService {

    private final OrderService orderService;
    private final PaymentService paymentService;

    public CheckoutResponse checkout(CheckoutRequest request) {
        try {
            // المحاولة الطبيعية
            Order order = orderService.createOrder(request);
            Payment payment = paymentService.process(order);

            return CheckoutResponse.success(order, payment);

        } catch (OrderServiceException e) {
            // Graceful degradation
            log.warn("Order service failed, using alternative flow", e);

            // احفظ في queue للمعالجة لاحقاً
            pendingOrderQueue.send(request);

            return CheckoutResponse.pending(
                "Your order is being processed. You'll receive confirmation soon."
            );
        }
    }
}
```

#### Resilience4j Configuration

```yaml
resilience4j:
  circuitbreaker:
    instances:
      orderService:
        registerHealthIndicator: true
        slidingWindowSize: 10
        minimumNumberOfCalls: 5
        permittedNumberOfCallsInHalfOpenState: 3
        automaticTransitionFromOpenToHalfOpenEnabled: true
        waitDurationInOpenState: 5s
        failureRateThreshold: 50
        eventConsumerBufferSize: 10

  retry:
    instances:
      orderService:
        maxAttempts: 3
        waitDuration: 1s
        exponentialBackoffMultiplier: 2
        retryExceptions:
          - java.net.ConnectException
          - java.io.IOException

  timelimiter:
    instances:
      orderService:
        timeoutDuration: 3s
```

---

### 44. ماذا تفعل إذا كان أحد الـ Services بطيء؟

#### 1️⃣ تحديد المشكلة (Diagnosis)

```java
// تفعيل Metrics
@Configuration
public class MetricsConfig {

    @Bean
    public TimedAspect timedAspect(MeterRegistry registry) {
        return new TimedAspect(registry);
    }
}

@Service
public class OrderService {

    @Timed(
        value = "order.create",
        description = "Time to create order",
        histogram = true
    )
    public Order createOrder(OrderRequest request) {
        long start = System.currentTimeMillis();

        try {
            // Database query - قد تكون بطيئة
            Order order = orderRepository.save(new Order(request));

            // External call - قد يكون بطيء
            customerService.validateCustomer(request.getCustomerId());

            return order;

        } finally {
            long duration = System.currentTimeMillis() - start;
            if (duration > 1000) {
                log.warn("Slow order creation: {}ms", duration);
            }
        }
    }
}
```

#### 2️⃣ Database Optimization

```java
// مشكلة: N+1 Query Problem
// ❌ البطيء
@Service
public class OrderService {

    public List<OrderDTO> getAllOrders() {
        List<Order> orders = orderRepository.findAll();

        return orders.stream()
            .map(order -> {
                // N queries إضافية!
                Customer customer = customerRepository.findById(order.getCustomerId());
                return new OrderDTO(order, customer);
            })
            .collect(Collectors.toList());
    }
}

// ✅ السريع
@Service
public class OrderService {

    public List<OrderDTO> getAllOrders() {
        // استخدم JOIN FETCH
        List<Order> orders = orderRepository.findAllWithCustomers();

        return orders.stream()
            .map(OrderDTO::new)
            .collect(Collectors.toList());
    }
}

@Repository
public interface OrderRepository extends JpaRepository<Order, Long> {

    @Query("SELECT o FROM Order o JOIN FETCH o.customer")
    List<Order> findAllWithCustomers();

    // أو استخدم EntityGraph
    @EntityGraph(attributePaths = {"customer", "items"})
    List<Order> findAll();
}
```

#### 3️⃣ Caching Strategy

```java
@Service
public class ProductService {

    @Cacheable(value = "products", key = "#id")
    public Product getProduct(Long id) {
        log.info("Fetching product from database: {}", id);
        return productRepository.findById(id)
            .orElseThrow(() -> new ProductNotFoundException(id));
    }

    @CacheEvict(value = "products", key = "#product.id")
    public Product updateProduct(Product product) {
        return productRepository.save(product);
    }
}

// Redis Configuration
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory factory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(10))
            .disableCachingNullValues();

        return RedisCacheManager.builder(factory)
            .cacheDefaults(config)
            .build();
    }
}
```

#### 4️⃣ Async Processing

```java
@Service
public class NotificationService {

    // ❌ Synchronous - يبطئ الـ response
    public void sendOrderConfirmation(Order order) {
        emailService.send(order.getCustomerEmail(), "Order Confirmed");
        smsService.send(order.getCustomerPhone(), "Order #" + order.getId());
        pushService.send(order.getCustomerId(), "Your order is confirmed");
    }

    // ✅ Asynchronous - لا يبطئ الـ response
    @Async
    public CompletableFuture<Void> sendOrderConfirmationAsync(Order order) {
        emailService.send(order.getCustomerEmail(), "Order Confirmed");
        smsService.send(order.getCustomerPhone(), "Order #" + order.getId());
        pushService.send(order.getCustomerId(), "Your order is confirmed");
        return CompletableFuture.completedFuture(null);
    }
}

@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean(name = "taskExecutor")
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
```

#### 5️⃣ Connection Pool Tuning

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000

  data:
    redis:
      lettuce:
        pool:
          max-active: 8
          max-idle: 8
          min-idle: 2
```

---

### 45. كيف تطبق Rate Limiting؟

#### 1️⃣ API Gateway Level (Spring Cloud Gateway)

```java
@Configuration
public class RateLimitConfig {

    @Bean
    public RouteLocator routes(RouteLocatorBuilder builder) {
        return builder.routes()
            .route("order-service", r -> r
                .path("/api/orders/**")
                .filters(f -> f
                    .requestRateLimiter(config -> config
                        .setRateLimiter(redisRateLimiter())
                        .setKeyResolver(userKeyResolver())
                    )
                )
                .uri("lb://ORDER-SERVICE")
            )
            .build();
    }

    @Bean
    public RedisRateLimiter redisRateLimiter() {
        return new RedisRateLimiter(
            10,  // replenishRate: 10 requests per second
            20   // burstCapacity: max 20 requests
        );
    }

    @Bean
    public KeyResolver userKeyResolver() {
        return exchange -> Mono.just(
            exchange.getRequest()
                .getHeaders()
                .getFirst("X-User-Id")
        );
    }
}
```

#### 2️⃣ Service Level (Bucket4j)

```java
// Dependencies
// implementation 'com.bucket4j:bucket4j-core:8.1.0'

@Service
public class RateLimitService {

    private final Map<String, Bucket> cache = new ConcurrentHashMap<>();

    public Bucket resolveBucket(String userId) {
        return cache.computeIfAbsent(userId, this::newBucket);
    }

    private Bucket newBucket(String userId) {
        Bandwidth limit = Bandwidth.builder()
            .capacity(20)
            .refillIntervally(20, Duration.ofMinutes(1))
            .build();

        return Bucket.builder()
            .addLimit(limit)
            .build();
    }
}

@RestController
@RequestMapping("/api/orders")
public class OrderController {

    private final RateLimitService rateLimitService;

    @PostMapping
    public ResponseEntity<Order> createOrder(
        @RequestHeader("X-User-Id") String userId,
        @RequestBody OrderRequest request
    ) {
        Bucket bucket = rateLimitService.resolveBucket(userId);

        if (bucket.tryConsume(1)) {
            Order order = orderService.createOrder(request);
            return ResponseEntity.ok(order);
        } else {
            return ResponseEntity
                .status(HttpStatus.TOO_MANY_REQUESTS)
                .header("X-Rate-Limit-Retry-After-Seconds", "60")
                .build();
        }
    }
}
```

#### 3️⃣ Redis-based Rate Limiting

```java
@Service
public class RedisRateLimitService {

    private final StringRedisTemplate redisTemplate;

    public boolean isAllowed(String key, int maxRequests, Duration window) {
        String redisKey = "rate_limit:" + key;
        long currentTime = System.currentTimeMillis();
        long windowStart = currentTime - window.toMillis();

        // استخدم Redis Sorted Set
        redisTemplate.opsForZSet().removeRangeByScore(
            redisKey, 0, windowStart
        );

        Long count = redisTemplate.opsForZSet().zCard(redisKey);

        if (count == null || count < maxRequests) {
            redisTemplate.opsForZSet().add(
                redisKey,
                UUID.randomUUID().toString(),
                currentTime
            );
            redisTemplate.expire(redisKey, window);
            return true;
        }

        return false;
    }
}

@RestController
public class ApiController {

    private final RedisRateLimitService rateLimitService;

    @GetMapping("/api/products")
    public ResponseEntity<List<Product>> getProducts(
        @RequestHeader("X-User-Id") String userId
    ) {
        boolean allowed = rateLimitService.isAllowed(
            userId,
            100,  // 100 requests
            Duration.ofMinutes(1)  // per minute
        );

        if (!allowed) {
            return ResponseEntity.status(HttpStatus.TOO_MANY_REQUESTS)
                .header("X-RateLimit-Limit", "100")
                .header("X-RateLimit-Remaining", "0")
                .header("X-RateLimit-Reset", String.valueOf(
                    System.currentTimeMillis() + 60000
                ))
                .build();
        }

        return ResponseEntity.ok(productService.getAllProducts());
    }
}
```

#### 4️⃣ Annotation-based Rate Limiting

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RateLimit {
    int requests() default 100;
    int perSeconds() default 60;
}

@Aspect
@Component
public class RateLimitAspect {

    private final RedisRateLimitService rateLimitService;

    @Around("@annotation(rateLimit)")
    public Object checkRateLimit(
        ProceedingJoinPoint joinPoint,
        RateLimit rateLimit
    ) throws Throwable {

        HttpServletRequest request =
            ((ServletRequestAttributes) RequestContextHolder.currentRequestAttributes())
                .getRequest();

        String userId = request.getHeader("X-User-Id");

        boolean allowed = rateLimitService.isAllowed(
            userId,
            rateLimit.requests(),
            Duration.ofSeconds(rateLimit.perSeconds())
        );

        if (!allowed) {
            throw new RateLimitExceededException("Too many requests");
        }

        return joinPoint.proceed();
    }
}

// الاستخدام
@RestController
public class OrderController {

    @PostMapping("/api/orders")
    @RateLimit(requests = 10, perSeconds = 60)
    public Order createOrder(@RequestBody OrderRequest request) {
        return orderService.createOrder(request);
    }
}
```

---

### 46. كيف تتعامل مع Duplicate Requests (Idempotency)؟

#### 1️⃣ Idempotency Key Pattern

```java
@Entity
@Table(name = "idempotency_keys")
public class IdempotencyRecord {

    @Id
    private String idempotencyKey;

    private String requestHash;

    @Column(columnDefinition = "TEXT")
    private String response;

    private LocalDateTime createdAt;

    private LocalDateTime expiresAt;
}

@Service
public class IdempotencyService {

    private final IdempotencyRepository repository;
    private final ObjectMapper objectMapper;

    public <T> Optional<T> checkIdempotency(
        String key,
        Class<T> responseType
    ) {
        return repository.findById(key)
            .filter(record -> record.getExpiresAt().isAfter(LocalDateTime.now()))
            .map(record -> {
                try {
                    return objectMapper.readValue(
                        record.getResponse(),
                        responseType
                    );
                } catch (JsonProcessingException e) {
                    return null;
                }
            });
    }

    public <T> void saveIdempotency(
        String key,
        T response,
        Duration ttl
    ) {
        try {
            IdempotencyRecord record = new IdempotencyRecord();
            record.setIdempotencyKey(key);
            record.setResponse(objectMapper.writeValueAsString(response));
            record.setCreatedAt(LocalDateTime.now());
            record.setExpiresAt(LocalDateTime.now().plus(ttl));

            repository.save(record);
        } catch (JsonProcessingException e) {
            log.error("Failed to save idempotency record", e);
        }
    }
}
```

#### 2️⃣ Controller Implementation

```java
@RestController
@RequestMapping("/api/payments")
public class PaymentController {

    private final PaymentService paymentService;
    private final IdempotencyService idempotencyService;

    @PostMapping
    public ResponseEntity<Payment> processPayment(
        @RequestHeader("Idempotency-Key") String idempotencyKey,
        @RequestBody PaymentRequest request
    ) {
        // تحقق من وجود request سابق بنفس الـ key
        Optional<Payment> existing = idempotencyService
            .checkIdempotency(idempotencyKey, Payment.class);

        if (existing.isPresent()) {
            log.info("Duplicate request detected: {}", idempotencyKey);
            return ResponseEntity.ok(existing.get());
        }

        // معالجة الطلب
        Payment payment = paymentService.processPayment(request);

        // حفظ النتيجة
        idempotencyService.saveIdempotency(
            idempotencyKey,
            payment,
            Duration.ofHours(24)
        );

        return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(payment);
    }
}
```

#### 3️⃣ Annotation-based Idempotency

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Idempotent {
    String keyHeader() default "Idempotency-Key";
    long ttlHours() default 24;
}

@Aspect
@Component
public class IdempotencyAspect {

    private final IdempotencyService idempotencyService;

    @Around("@annotation(idempotent)")
    public Object handleIdempotency(
        ProceedingJoinPoint joinPoint,
        Idempotent idempotent
    ) throws Throwable {

        HttpServletRequest request =
            ((ServletRequestAttributes) RequestContextHolder.currentRequestAttributes())
                .getRequest();

        String key = request.getHeader(idempotent.keyHeader());

        if (key == null || key.isEmpty()) {
            throw new BadRequestException("Idempotency-Key header is required");
        }

        // تحقق من وجود response سابق
        Optional<Object> cached = idempotencyService.checkIdempotency(
            key,
            Object.class
        );

        if (cached.isPresent()) {
            return cached.get();
        }

        // تنفيذ العملية
        Object result = joinPoint.proceed();

        // حفظ النتيجة
        idempotencyService.saveIdempotency(
            key,
            result,
            Duration.ofHours(idempotent.ttlHours())
        );

        return result;
    }
}

// الاستخدام
@RestController
public class OrderController {

    @PostMapping("/api/orders")
    @Idempotent(ttlHours = 48)
    public Order createOrder(@RequestBody OrderRequest request) {
        return orderService.createOrder(request);
    }
}
```

#### 4️⃣ Database-level Idempotency

```java
@Entity
@Table(
    name = "orders",
    uniqueConstraints = @UniqueConstraint(
        columnNames = {"customer_id", "idempotency_key"}
    )
)
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private Long customerId;

    @Column(nullable = false)
    private String idempotencyKey;

    private Double total;
    private String status;
}

@Service
public class OrderService {

    @Transactional
    public Order createOrder(OrderRequest request, String idempotencyKey) {
        try {
            Order order = new Order();
            order.setCustomerId(request.getCustomerId());
            order.setIdempotencyKey(idempotencyKey);
            order.setTotal(request.getTotal());
            order.setStatus("PENDING");

            return orderRepository.save(order);

        } catch (DataIntegrityViolationException e) {
            // Duplicate key - ابحث عن الـ order الموجود
            log.info("Duplicate order detected: {}", idempotencyKey);

            return orderRepository
                .findByCustomerIdAndIdempotencyKey(
                    request.getCustomerId(),
                    idempotencyKey
                )
                .orElseThrow(() -> new IllegalStateException(
                    "Order not found after duplicate key error"
                ));
        }
    }
}
```

#### 5️⃣ Distributed Lock للحماية من Race Conditions

```java
@Service
public class PaymentService {

    private final RedissonClient redisson;
    private final PaymentRepository paymentRepository;

    public Payment processPayment(PaymentRequest request, String idempotencyKey) {
        RLock lock = redisson.getLock("payment:" + idempotencyKey);

        try {
            // انتظر 10 ثواني للحصول على الـ lock
            boolean acquired = lock.tryLock(10, 30, TimeUnit.SECONDS);

            if (!acquired) {
                throw new PaymentProcessingException("Could not acquire lock");
            }

            // تحقق مرة أخرى بعد الحصول على الـ lock
            Optional<Payment> existing = paymentRepository
                .findByIdempotencyKey(idempotencyKey);

            if (existing.isPresent()) {
                return existing.get();
            }

            // معالجة الدفع
            Payment payment = new Payment();
            payment.setIdempotencyKey(idempotencyKey);
            payment.setAmount(request.getAmount());
            payment.setStatus("COMPLETED");

            return paymentRepository.save(payment);

        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new PaymentProcessingException("Lock interrupted", e);
        } finally {
            if (lock.isHeldByCurrentThread()) {
                lock.unlock();
            }
        }
    }
}
```

---

## خاتمة

هذا الدليل يغطي **46 سؤال** شامل عن الـ Microservices مع Java و Spring Boot:

✅ **الأساسيات** (1-4)
✅ **Service Discovery** (5-7)
✅ **API Gateway** (8-10)
✅ **Communication** (11-14)
✅ **Configuration** (15-17)
✅ **Resilience** (18-22)
✅ **Monitoring** (23-25)
✅ **Security** (26-28)
✅ **Database & Transactions** (29-32)
✅ **Deployment** (33-36)
✅ **Advanced Topics** (37-42)
✅ **Scenario-Based** (43-46)

---

**نصائح للمقابلة:**

1. **افهم الـ Why قبل الـ How** - اشرح لماذا تستخدم pattern معين
2. **اذكر Trade-offs** - كل حل له مميزات وعيوب
3. **استخدم أمثلة عملية** - من مشاريعك السابقة
4. **كن مستعد للـ Deep Dive** - في أي موضوع
5. **اعرف متى لا تستخدم Microservices** - ليس الحل الأمثل دائماً

---

**🎯 Good Luck في المقابلة! 🚀**