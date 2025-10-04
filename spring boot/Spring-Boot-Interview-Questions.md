# دليل أسئلة مقابلات Spring Boot الشامل

## المحتويات
1. [الأساسيات](#الأساسيات)
2. [Annotations](#annotations)
3. [Dependency Injection](#dependency-injection)
4. [Configuration](#configuration)
5. [Data Access & JPA](#data-access--jpa)
6. [REST APIs](#rest-apis)
7. [Exception Handling](#exception-handling)
8. [Security](#security)
9. [Testing](#testing)
10. [Actuator](#actuator)
11. [أسئلة متقدمة](#أسئلة-متقدمة)

---

## الأساسيات

### 1. ما هو Spring Boot؟

**الإجابة:**
Spring Boot هو إطار عمل مبني على Spring Framework يهدف إلى تبسيط عملية تطوير تطبيقات Spring من خلال:

- **Auto-configuration**: التكوين التلقائي للتطبيق بناءً على المكتبات الموجودة
- **Starter Dependencies**: مجموعات جاهزة من المكتبات المتوافقة
- **Embedded Servers**: خوادم مدمجة (Tomcat, Jetty, Undertow)
- **Production-ready features**: أدوات جاهزة للإنتاج مثل Actuator
- **لا يحتاج XML**: تكوين بالكامل عبر Java أو Properties

**مثال:**
```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

---

### 2. ما الفرق بين Spring Framework و Spring Boot؟

**الإجابة المفصلة:**

| Spring Framework | Spring Boot |
|-----------------|-------------|
| يحتاج تكوين يدوي كثير | تكوين تلقائي |
| يحتاج تعريف XML أو Java Config | تكوين بسيط عبر annotations |
| يحتاج إعداد خادم خارجي | خادم مدمج جاهز |
| إدارة يدوية للـ dependencies | Starter dependencies جاهزة |
| لا يوجد Actuator مدمج | Actuator جاهز للاستخدام |

**مثال Spring التقليدي:**
```xml
<!-- web.xml -->
<web-app>
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    </servlet>
</web-app>
```

**مثال Spring Boot:**
```java
// فقط annotation واحدة!
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

---

### 3. ما هي الميزات الرئيسية لـ Spring Boot؟

**الإجابة:**

#### 1. Auto-Configuration
يكتشف المكتبات الموجودة ويكوّن التطبيق تلقائياً:
```java
// إذا وجد H2 database في classpath
// سيقوم بتكوين DataSource تلقائياً
```

#### 2. Starter Dependencies
```xml
<!-- بدلاً من إضافة 20+ مكتبة، فقط أضف starter واحد -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

#### 3. Embedded Server
```java
// لا تحتاج Tomcat خارجي
// الخادم مدمج في JAR file
```

#### 4. Actuator
```properties
# endpoints جاهزة للمراقبة
management.endpoints.web.exposure.include=health,metrics,info
```

#### 5. DevTools
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
<!-- يعيد تشغيل التطبيق تلقائياً عند التعديل -->
```

---

### 4. ما هو @SpringBootApplication؟

**الإجابة:**
هو annotation مركب يجمع ثلاث annotations:

```java
@SpringBootApplication
// يساوي:
@Configuration          // يشير أن الكلاس يحتوي على bean definitions
@EnableAutoConfiguration // يفعّل التكوين التلقائي
@ComponentScan          // يبحث عن components في الـ package وما تحته

public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

**شرح تفصيلي:**

1. **@Configuration**: يسمح بتعريف beans:
```java
@SpringBootApplication
public class MyApplication {

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

2. **@EnableAutoConfiguration**: يكوّن beans تلقائياً:
```java
// إذا وجد Jackson في classpath
// سيكوّن ObjectMapper تلقائياً
```

3. **@ComponentScan**: يبحث عن @Component, @Service, @Repository:
```java
com.example.myapp          // Main class هنا
├── controller             // سيجد @RestController
├── service               // سيجد @Service
└── repository            // سيجد @Repository
```

---

### 5. كيف يعمل Auto-Configuration في Spring Boot؟

**الإجابة المفصلة:**

Auto-configuration يعمل من خلال:

#### 1. Conditional Annotations
```java
@Configuration
@ConditionalOnClass(DataSource.class) // إذا وجد DataSource في classpath
@ConditionalOnMissingBean(DataSource.class) // وإذا لم يعرّف المستخدم DataSource
public class DataSourceAutoConfiguration {

    @Bean
    public DataSource dataSource() {
        // إنشاء DataSource تلقائياً
        return new HikariDataSource();
    }
}
```

#### 2. spring.factories File
```properties
# في ملف META-INF/spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration
```

#### 3. Properties Customization
```properties
# application.properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=secret

# Auto-configuration سيستخدم هذه القيم
```

#### 4. إيقاف Auto-Configuration معينة
```java
@SpringBootApplication(exclude = {
    DataSourceAutoConfiguration.class,
    HibernateJpaAutoConfiguration.class
})
public class MyApplication {
}
```

---

## Annotations

### 6. ما الفرق بين @Controller و @RestController؟

**الإجابة:**

```java
// @Controller - للـ MVC التقليدي
@Controller
public class WebController {

    @GetMapping("/page")
    public String getPage(Model model) {
        model.addAttribute("message", "Hello");
        return "view-name"; // يرجع اسم view (HTML)
    }

    @GetMapping("/api/data")
    @ResponseBody // يحتاج @ResponseBody لإرجاع JSON
    public User getUser() {
        return new User("Ali", 25);
    }
}

// @RestController - للـ REST APIs
@RestController // = @Controller + @ResponseBody
public class ApiController {

    @GetMapping("/api/user")
    public User getUser() {
        return new User("Ali", 25); // تلقائياً يتحول لـ JSON
    }

    @PostMapping("/api/user")
    public User createUser(@RequestBody User user) {
        return userService.save(user);
    }
}
```

**الفرق الرئيسي:**
- `@Controller`: يُستخدم مع Views (Thymeleaf, JSP)
- `@RestController`: يُستخدم مع REST APIs (يرجع JSON/XML مباشرة)

---

### 7. شرح @RequestMapping و variants

**الإجابة:**

```java
@RestController
@RequestMapping("/api/users") // Base path للـ controller
public class UserController {

    // 1. @GetMapping - للقراءة
    @GetMapping // GET /api/users
    public List<User> getAllUsers() {
        return userService.findAll();
    }

    @GetMapping("/{id}") // GET /api/users/1
    public User getUserById(@PathVariable Long id) {
        return userService.findById(id);
    }

    @GetMapping("/search") // GET /api/users/search?name=Ali
    public List<User> searchUsers(@RequestParam String name) {
        return userService.findByName(name);
    }

    // 2. @PostMapping - للإنشاء
    @PostMapping // POST /api/users
    public User createUser(@RequestBody User user) {
        return userService.save(user);
    }

    // 3. @PutMapping - للتحديث الكامل
    @PutMapping("/{id}") // PUT /api/users/1
    public User updateUser(@PathVariable Long id, @RequestBody User user) {
        return userService.update(id, user);
    }

    // 4. @PatchMapping - للتحديث الجزئي
    @PatchMapping("/{id}") // PATCH /api/users/1
    public User partialUpdate(@PathVariable Long id, @RequestBody Map<String, Object> updates) {
        return userService.partialUpdate(id, updates);
    }

    // 5. @DeleteMapping - للحذف
    @DeleteMapping("/{id}") // DELETE /api/users/1
    public void deleteUser(@PathVariable Long id) {
        userService.delete(id);
    }
}
```

**Request Parameters:**
```java
// @PathVariable - من الـ URL path
@GetMapping("/users/{id}/posts/{postId}")
public Post getUserPost(@PathVariable Long id, @PathVariable Long postId) {
    return postService.find(id, postId);
}

// @RequestParam - من Query string
@GetMapping("/users")
public List<User> getUsers(
    @RequestParam(required = false, defaultValue = "0") int page,
    @RequestParam(required = false, defaultValue = "10") int size,
    @RequestParam(required = false) String sort
) {
    return userService.findAll(page, size, sort);
}

// @RequestBody - من Body (JSON)
@PostMapping("/users")
public User createUser(@RequestBody User user) {
    return userService.save(user);
}

// @RequestHeader - من Headers
@GetMapping("/users")
public List<User> getUsers(@RequestHeader("Authorization") String token) {
    // استخدام الـ token
    return userService.findAll();
}
```

---

### 8. شرح @Component, @Service, @Repository, @Controller

**الإجابة:**

كلها stereotype annotations للـ dependency injection:

```java
// 1. @Component - عام لأي Spring bean
@Component
public class EmailValidator {
    public boolean isValid(String email) {
        return email.contains("@");
    }
}

// 2. @Service - لطبقة الـ Business Logic
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private EmailValidator emailValidator;

    public User createUser(User user) {
        if (!emailValidator.isValid(user.getEmail())) {
            throw new IllegalArgumentException("Invalid email");
        }
        return userRepository.save(user);
    }

    @Transactional
    public void transferMoney(Long fromId, Long toId, Double amount) {
        // Business logic
    }
}

// 3. @Repository - لطبقة Data Access
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    List<User> findByName(String name);

    @Query("SELECT u FROM User u WHERE u.email = ?1")
    Optional<User> findByEmail(String email);
}

// يضيف exception translation (تحويل SQL exceptions لـ Spring exceptions)

// 4. @Controller / @RestController - لطبقة Presentation
@RestController
@RequestMapping("/api/users")
public class UserController {

    @Autowired
    private UserService userService;

    @PostMapping
    public User createUser(@RequestBody User user) {
        return userService.createUser(user);
    }
}
```

**الفروقات:**

| Annotation | الاستخدام | مميزات إضافية |
|-----------|----------|---------------|
| @Component | أي Spring bean | لا يوجد |
| @Service | Business logic | يوضح أنه service layer |
| @Repository | Data access | Exception translation |
| @Controller | MVC controller | لـ views |
| @RestController | REST API | @Controller + @ResponseBody |

---

### 9. ما هو @Autowired؟

**الإجابة:**

@Autowired يُستخدم للـ Dependency Injection التلقائي:

```java
// 1. Constructor Injection (الأفضل)
@Service
public class UserService {

    private final UserRepository userRepository;
    private final EmailService emailService;

    @Autowired // اختياري إذا كان constructor واحد فقط
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
}

// 2. Setter Injection
@Service
public class UserService {

    private UserRepository userRepository;

    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}

// 3. Field Injection (غير مفضل)
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private EmailService emailService;
}
```

**لماذا Constructor Injection هو الأفضل؟**

```java
@Service
public class UserService {

    private final UserRepository userRepository; // final = immutable

    // 1. يضمن أن الـ dependencies موجودة
    // 2. يسهل الـ testing
    // 3. يدعم immutability (final)
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}

// Testing سهل
@Test
public void testUserService() {
    UserRepository mockRepo = mock(UserRepository.class);
    UserService service = new UserService(mockRepo); // بدون Spring
    // test...
}
```

**@Autowired مع Collections:**
```java
@Service
public class NotificationService {

    private final List<MessageSender> messageSenders;

    // سيحقن كل beans من نوع MessageSender
    @Autowired
    public NotificationService(List<MessageSender> messageSenders) {
        this.messageSenders = messageSenders;
    }

    public void sendAll(String message) {
        messageSenders.forEach(sender -> sender.send(message));
    }
}

@Component
class EmailSender implements MessageSender { }

@Component
class SmsSender implements MessageSender { }

@Component
class PushNotificationSender implements MessageSender { }
```

---

### 10. ما هو @Qualifier و @Primary؟

**الإجابة:**

تُستخدم لحل مشكلة وجود أكثر من bean من نفس النوع:

```java
// المشكلة: عندنا أكثر من implementation
public interface MessageSender {
    void send(String message);
}

@Component
class EmailSender implements MessageSender {
    public void send(String message) {
        System.out.println("Sending email: " + message);
    }
}

@Component
class SmsSender implements MessageSender {
    public void send(String message) {
        System.out.println("Sending SMS: " + message);
    }
}

// الحل 1: @Qualifier
@Service
public class NotificationService {

    private final MessageSender messageSender;

    @Autowired
    public NotificationService(@Qualifier("emailSender") MessageSender messageSender) {
        this.messageSender = messageSender; // سيحقن EmailSender
    }
}

// الحل 2: @Primary (الافتراضي)
@Component
@Primary // هذا سيُستخدم افتراضياً
class EmailSender implements MessageSender {
    public void send(String message) {
        System.out.println("Sending email: " + message);
    }
}

@Component
class SmsSender implements MessageSender {
    public void send(String message) {
        System.out.println("Sending SMS: " + message);
    }
}

@Service
public class NotificationService {

    @Autowired
    private MessageSender messageSender; // سيحقن EmailSender (Primary)
}

// استخدام متقدم
@Service
public class NotificationService {

    private final MessageSender emailSender;
    private final MessageSender smsSender;

    @Autowired
    public NotificationService(
        @Qualifier("emailSender") MessageSender emailSender,
        @Qualifier("smsSender") MessageSender smsSender
    ) {
        this.emailSender = emailSender;
        this.smsSender = smsSender;
    }

    public void sendBoth(String message) {
        emailSender.send(message);
        smsSender.send(message);
    }
}
```

---

## Dependency Injection

### 11. ما هي أنواع Dependency Injection؟

**الإجابة:**

#### 1. Constructor Injection (الأفضل)

```java
@Service
public class OrderService {

    private final ProductRepository productRepository;
    private final PaymentService paymentService;
    private final EmailService emailService;

    // Constructor Injection
    public OrderService(
        ProductRepository productRepository,
        PaymentService paymentService,
        EmailService emailService
    ) {
        this.productRepository = productRepository;
        this.paymentService = paymentService;
        this.emailService = emailService;
    }

    public Order createOrder(Order order) {
        // استخدام الـ dependencies
        Product product = productRepository.findById(order.getProductId());
        paymentService.processPayment(order);
        emailService.sendConfirmation(order);
        return order;
    }
}
```

**المميزات:**
- ✅ Dependencies واضحة ومرئية
- ✅ يدعم `final` (immutability)
- ✅ سهولة Testing
- ✅ يضمن أن الـ dependencies موجودة

#### 2. Setter Injection

```java
@Service
public class UserService {

    private UserRepository userRepository;
    private EmailService emailService;

    // Setter Injection
    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Autowired(required = false) // optional dependency
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }
}
```

**متى تستخدمه:**
- للـ optional dependencies
- عندما تريد إعادة تكوين الـ bean بعد الإنشاء

#### 3. Field Injection (غير مفضل)

```java
@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    @Autowired
    private CategoryService categoryService;
}
```

**العيوب:**
- ❌ لا يمكن استخدام `final`
- ❌ صعوبة Testing (يحتاج Reflection)
- ❌ الـ dependencies مخفية
- ❌ يخالف SOLID principles

#### مقارنة عملية

```java
// Testing مع Constructor Injection (سهل)
@Test
public void testOrderService() {
    // Mock dependencies
    ProductRepository mockRepo = mock(ProductRepository.class);
    PaymentService mockPayment = mock(PaymentService.class);
    EmailService mockEmail = mock(EmailService.class);

    // إنشاء service بدون Spring
    OrderService service = new OrderService(mockRepo, mockPayment, mockEmail);

    // Test...
    Order order = service.createOrder(new Order());
    verify(mockPayment).processPayment(any());
}

// Testing مع Field Injection (صعب)
@Test
public void testProductService() {
    ProductService service = new ProductService();
    // لا يمكن حقن الـ dependencies بدون Reflection!
    // يحتاج @SpringBootTest أو ReflectionTestUtils
}
```

---

### 12. ما هي Bean Scopes في Spring؟

**الإجابة:**

#### 1. Singleton (Default)

```java
@Component
@Scope("singleton") // أو بدون @Scope (افتراضي)
public class DatabaseConnection {

    public DatabaseConnection() {
        System.out.println("Creating DatabaseConnection");
    }
}

// سيطبع "Creating DatabaseConnection" مرة واحدة فقط
// نفس الـ instance يُستخدم في كل التطبيق
```

#### 2. Prototype

```java
@Component
@Scope("prototype")
public class ShoppingCart {

    private List<Item> items = new ArrayList<>();

    public ShoppingCart() {
        System.out.println("Creating new ShoppingCart");
    }
}

// كل مرة تطلب ShoppingCart، سيُنشئ instance جديد

@Service
public class OrderService {

    @Autowired
    private ApplicationContext context;

    public ShoppingCart createNewCart() {
        // instance جديد في كل مرة
        return context.getBean(ShoppingCart.class);
    }
}
```

#### 3. Request (Web Applications)

```java
@Component
@Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class UserContext {

    private String userId;
    private String requestId;

    // instance جديد لكل HTTP request
}

@RestController
public class UserController {

    @Autowired
    private UserContext userContext; // مختلف لكل request

    @GetMapping("/user")
    public String getUser() {
        userContext.setUserId("123");
        return "User: " + userContext.getUserId();
    }
}
```

#### 4. Session (Web Applications)

```java
@Component
@Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class UserSession {

    private String username;
    private List<String> cartItems = new ArrayList<>();

    // نفس الـ instance لكل الـ requests من نفس الـ session
}
```

#### 5. Application

```java
@Component
@Scope("application")
public class AppConfig {
    // مشترك بين كل الـ ServletContexts
    // نادراً ما يُستخدم
}
```

#### مثال عملي شامل

```java
@Configuration
public class BeanScopeConfig {

    // Singleton - يُنشأ مرة واحدة
    @Bean
    @Scope("singleton")
    public DatabaseConnection databaseConnection() {
        return new DatabaseConnection();
    }

    // Prototype - يُنشأ في كل مرة
    @Bean
    @Scope("prototype")
    public ReportGenerator reportGenerator() {
        return new ReportGenerator();
    }
}

@Service
public class ReportService {

    @Autowired
    private DatabaseConnection dbConnection; // نفس الـ instance دائماً

    @Autowired
    private ApplicationContext context;

    public void generateReport() {
        // كل تقرير يحتاج generator جديد
        ReportGenerator generator1 = context.getBean(ReportGenerator.class);
        ReportGenerator generator2 = context.getBean(ReportGenerator.class);

        System.out.println(generator1 == generator2); // false (Prototype)
        System.out.println(dbConnection == context.getBean(DatabaseConnection.class)); // true (Singleton)
    }
}
```

**الجدول المقارن:**

| Scope | متى يُنشأ | متى يُدمر | الاستخدام |
|-------|----------|----------|-----------|
| Singleton | عند بدء التطبيق | عند إيقاف التطبيق | Services, Repositories |
| Prototype | عند كل طلب | بواسطة Garbage Collector | Stateful objects |
| Request | عند كل HTTP request | بعد انتهاء Request | Request-specific data |
| Session | عند أول request في session | بعد انتهاء Session | User session data |

---

## Configuration

### 13. ما الفرق بين application.properties و application.yml؟

**الإجابة:**

نفس الوظيفة، لكن صيغة مختلفة:

#### application.properties

```properties
# Database Configuration
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=secret
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA Configuration
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect

# Server Configuration
server.port=8080
server.servlet.context-path=/api

# Logging
logging.level.root=INFO
logging.level.com.example.myapp=DEBUG
logging.file.name=app.log

# Custom Properties
app.name=My Application
app.version=1.0.0
app.features.email.enabled=true
app.features.sms.enabled=false
```

#### application.yml (أوضح وأقل تكراراً)

```yaml
# Database Configuration
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: secret
    driver-class-name: com.mysql.cj.jdbc.Driver

  # JPA Configuration
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQL8Dialect

# Server Configuration
server:
  port: 8080
  servlet:
    context-path: /api

# Logging
logging:
  level:
    root: INFO
    com.example.myapp: DEBUG
  file:
    name: app.log

# Custom Properties
app:
  name: My Application
  version: 1.0.0
  features:
    email:
      enabled: true
    sms:
      enabled: false
```

#### قراءة Custom Properties

```java
// 1. باستخدام @Value
@Component
public class AppInfo {

    @Value("${app.name}")
    private String appName;

    @Value("${app.version}")
    private String version;

    @Value("${app.features.email.enabled}")
    private boolean emailEnabled;

    @Value("${app.timeout:30}") // قيمة افتراضية = 30
    private int timeout;
}

// 2. باستخدام @ConfigurationProperties (الأفضل)
@Component
@ConfigurationProperties(prefix = "app")
public class AppProperties {

    private String name;
    private String version;
    private Features features;

    public static class Features {
        private Email email;
        private Sms sms;

        public static class Email {
            private boolean enabled;
            // getters/setters
        }

        public static class Sms {
            private boolean enabled;
            // getters/setters
        }
    }

    // getters/setters
}

// الاستخدام
@Service
public class NotificationService {

    @Autowired
    private AppProperties appProperties;

    public void sendNotification(String message) {
        if (appProperties.getFeatures().getEmail().isEnabled()) {
            // send email
        }
        if (appProperties.getFeatures().getSms().isEnabled()) {
            // send sms
        }
    }
}
```

#### Multiple Profiles

```yaml
# application.yml (مشترك)
spring:
  application:
    name: My App

---
# application-dev.yml (Development)
spring:
  config:
    activate:
      on-profile: dev

  datasource:
    url: jdbc:h2:mem:testdb
    username: sa
    password:

  jpa:
    show-sql: true

server:
  port: 8080

---
# application-prod.yml (Production)
spring:
  config:
    activate:
      on-profile: prod

  datasource:
    url: jdbc:mysql://prod-server:3306/proddb
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}

  jpa:
    show-sql: false

server:
  port: 80

---
# application-test.yml (Testing)
spring:
  config:
    activate:
      on-profile: test

  datasource:
    url: jdbc:h2:mem:testdb
```

**تفعيل Profile:**
```properties
# في application.properties
spring.profiles.active=dev

# أو عبر command line
java -jar myapp.jar --spring.profiles.active=prod

# أو environment variable
export SPRING_PROFILES_ACTIVE=prod
```

---

### 14. شرح @ConfigurationProperties

**الإجابة:**

بدلاً من استخدام @Value لكل property:

```java
// ❌ الطريقة القديمة (متعبة)
@Component
public class EmailConfig {

    @Value("${email.host}")
    private String host;

    @Value("${email.port}")
    private int port;

    @Value("${email.username}")
    private String username;

    @Value("${email.password}")
    private String password;

    @Value("${email.smtp.auth}")
    private boolean smtpAuth;

    @Value("${email.smtp.starttls.enable}")
    private boolean starttlsEnable;
}
```

```java
// ✅ الطريقة الأفضل
@Component
@ConfigurationProperties(prefix = "email")
@Validated // للـ validation
public class EmailProperties {

    @NotBlank
    private String host;

    @Min(1)
    @Max(65535)
    private int port;

    @NotBlank
    private String username;

    @NotBlank
    private String password;

    private Smtp smtp;

    public static class Smtp {
        private boolean auth;
        private Starttls starttls;

        public static class Starttls {
            private boolean enable;
            // getters/setters
        }

        // getters/setters
    }

    // getters/setters
}
```

**application.yml:**
```yaml
email:
  host: smtp.gmail.com
  port: 587
  username: myemail@gmail.com
  password: secret
  smtp:
    auth: true
    starttls:
      enable: true
```

**مثال متقدم - Database Configuration:**

```java
@Component
@ConfigurationProperties(prefix = "app.datasource")
public class DatabaseProperties {

    private String url;
    private String username;
    private String password;
    private Pool pool;
    private int connectionTimeout = 30000;

    public static class Pool {
        private int maxSize = 10;
        private int minIdle = 5;
        private long maxLifetime = 1800000;

        // getters/setters
    }

    // getters/setters
}
```

```yaml
app:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: secret
    connection-timeout: 20000
    pool:
      max-size: 20
      min-idle: 5
      max-lifetime: 1800000
```

**الاستخدام:**
```java
@Service
public class DatabaseService {

    @Autowired
    private DatabaseProperties dbProperties;

    public void connect() {
        System.out.println("Connecting to: " + dbProperties.getUrl());
        System.out.println("Pool max size: " + dbProperties.getPool().getMaxSize());
    }
}
```

**Validation مع @ConfigurationProperties:**

```java
@Component
@ConfigurationProperties(prefix = "app")
@Validated
public class AppConfig {

    @NotBlank(message = "App name is required")
    private String name;

    @Email
    private String adminEmail;

    @Min(1024)
    @Max(65535)
    private int port;

    @Valid
    private Security security;

    public static class Security {

        @NotBlank
        @Size(min = 32)
        private String jwtSecret;

        @Positive
        private long jwtExpiration;

        // getters/setters
    }

    // getters/setters
}
```

---

### 15. شرح Profiles في Spring Boot

**الإجابة:**

Profiles تسمح بتكوينات مختلفة لبيئات مختلفة:

#### 1. تعريف Profile-specific Beans

```java
@Configuration
public class DatabaseConfig {

    // Bean لـ Development
    @Bean
    @Profile("dev")
    public DataSource devDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .build();
    }

    // Bean لـ Production
    @Bean
    @Profile("prod")
    public DataSource prodDataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl("jdbc:mysql://prod-server:3306/db");
        dataSource.setUsername("prod_user");
        dataSource.setPassword("prod_pass");
        dataSource.setMaximumPoolSize(50);
        return dataSource;
    }

    // Bean لـ Testing
    @Bean
    @Profile("test")
    public DataSource testDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .addScript("schema.sql")
            .addScript("test-data.sql")
            .build();
    }
}
```

#### 2. Profile-specific Configuration Files

```
src/main/resources/
├── application.yml              # مشترك
├── application-dev.yml          # Development
├── application-prod.yml         # Production
├── application-test.yml         # Testing
└── application-staging.yml      # Staging
```

**application.yml (مشترك):**
```yaml
spring:
  application:
    name: My App

app:
  version: 1.0.0
```

**application-dev.yml:**
```yaml
spring:
  datasource:
    url: jdbc:h2:mem:devdb
    username: sa
    password:

  jpa:
    show-sql: true
    hibernate:
      ddl-auto: create-drop

logging:
  level:
    root: DEBUG

server:
  port: 8080
```

**application-prod.yml:**
```yaml
spring:
  datasource:
    url: jdbc:mysql://prod-db:3306/myapp
    username: ${DB_USER}
    password: ${DB_PASS}

  jpa:
    show-sql: false
    hibernate:
      ddl-auto: validate

logging:
  level:
    root: WARN
    com.example: INFO

server:
  port: 80
```

#### 3. تفعيل Profiles

```properties
# 1. في application.properties
spring.profiles.active=dev

# 2. عبر Command Line
java -jar myapp.jar --spring.profiles.active=prod

# 3. عبر Environment Variable
export SPRING_PROFILES_ACTIVE=prod

# 4. تفعيل أكثر من profile
spring.profiles.active=dev,debug

# 5. برمجياً
SpringApplication app = new SpringApplication(MyApp.class);
app.setAdditionalProfiles("dev");
app.run(args);
```

#### 4. Profile-specific Components

```java
// Component يعمل فقط في Development
@Service
@Profile("dev")
public class MockPaymentService implements PaymentService {

    @Override
    public PaymentResult processPayment(Payment payment) {
        // Mock implementation
        return PaymentResult.success();
    }
}

// Component يعمل فقط في Production
@Service
@Profile("prod")
public class RealPaymentService implements PaymentService {

    @Override
    public PaymentResult processPayment(Payment payment) {
        // Real payment gateway integration
        return paymentGateway.charge(payment);
    }
}

// Component يعمل في أي profile ماعدا production
@Service
@Profile("!prod")
public class DebugService {
    // debugging utilities
}
```

#### 5. @Profile مع expressions

```java
@Configuration
public class AppConfig {

    // يعمل في dev أو test
    @Bean
    @Profile({"dev", "test"})
    public DebugService debugService() {
        return new DebugService();
    }

    // يعمل في كل البيئات ماعدا production
    @Bean
    @Profile("!prod")
    public MockEmailService mockEmailService() {
        return new MockEmailService();
    }

    // يعمل فقط في production و staging
    @Bean
    @Profile({"prod", "staging"})
    public RealEmailService realEmailService() {
        return new RealEmailService();
    }
}
```

#### 6. Default Profile

```yaml
# application.yml
spring:
  profiles:
    default: dev  # إذا لم يُحدد profile، استخدم dev
```

#### مثال عملي كامل

```java
@Configuration
public class SecurityConfig {

    @Bean
    @Profile("dev")
    public SecurityFilterChain devSecurity(HttpSecurity http) throws Exception {
        // أمان مبسط للـ development
        return http
            .csrf().disable()
            .authorizeHttpRequests(auth -> auth.anyRequest().permitAll())
            .build();
    }

    @Bean
    @Profile("prod")
    public SecurityFilterChain prodSecurity(HttpSecurity http) throws Exception {
        // أمان كامل للـ production
        return http
            .csrf().enable()
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/**").authenticated()
                .anyRequest().denyAll()
            )
            .oauth2Login()
            .build();
    }
}
```

---

## Data Access & JPA

### 16. شرح Spring Data JPA

**الإجابة:**

Spring Data JPA يبسط التعامل مع قواعد البيانات:

#### 1. Entity Class

```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String username;

    @Column(nullable = false, unique = true)
    private String email;

    @Column(nullable = false)
    private String password;

    @Column(name = "created_at")
    private LocalDateTime createdAt;

    @Enumerated(EnumType.STRING)
    private UserRole role;

    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Post> posts = new ArrayList<>();

    @ManyToMany
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles = new HashSet<>();

    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
    }

    // getters/setters
}

@Entity
@Table(name = "posts")
public class Post {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String title;

    @Column(columnDefinition = "TEXT")
    private String content;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    private User user;

    @ManyToMany
    @JoinTable(
        name = "post_tags",
        joinColumns = @JoinColumn(name = "post_id"),
        inverseJoinColumns = @JoinColumn(name = "tag_id")
    )
    private Set<Tag> tags = new HashSet<>();

    // getters/setters
}
```

#### 2. Repository Interface

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    // 1. Query Methods (بدون كتابة SQL)
    Optional<User> findByUsername(String username);

    Optional<User> findByEmail(String email);

    List<User> findByRole(UserRole role);

    List<User> findByCreatedAtAfter(LocalDateTime date);

    boolean existsByUsername(String username);

    boolean existsByEmail(String email);

    long countByRole(UserRole role);

    void deleteByUsername(String username);

    // 2. مع Query keywords
    List<User> findByUsernameContaining(String keyword);

    List<User> findByEmailEndingWith(String domain);

    List<User> findByCreatedAtBetween(LocalDateTime start, LocalDateTime end);

    List<User> findByRoleIn(List<UserRole> roles);

    List<User> findByUsernameStartingWithAndRole(String prefix, UserRole role);

    // 3. JPQL Query
    @Query("SELECT u FROM User u WHERE u.email = ?1")
    Optional<User> findByEmailJPQL(String email);

    @Query("SELECT u FROM User u WHERE u.username LIKE %?1%")
    List<User> searchByUsername(String keyword);

    // 4. Native SQL Query
    @Query(value = "SELECT * FROM users WHERE email = ?1", nativeQuery = true)
    Optional<User> findByEmailNative(String email);

    // 5. Named Parameters
    @Query("SELECT u FROM User u WHERE u.username = :username AND u.role = :role")
    Optional<User> findByUsernameAndRole(
        @Param("username") String username,
        @Param("role") UserRole role
    );

    // 6. Modifying Query
    @Modifying
    @Query("UPDATE User u SET u.role = :role WHERE u.id = :id")
    int updateUserRole(@Param("id") Long id, @Param("role") UserRole role);

    @Modifying
    @Query("DELETE FROM User u WHERE u.createdAt < :date")
    int deleteOldUsers(@Param("date") LocalDateTime date);

    // 7. Projection (لاختيار أعمدة محددة)
    @Query("SELECT u.username, u.email FROM User u WHERE u.role = :role")
    List<Object[]> findUsernameAndEmailByRole(@Param("role") UserRole role);

    // 8. DTO Projection
    @Query("SELECT new com.example.dto.UserDTO(u.id, u.username, u.email) FROM User u")
    List<UserDTO> findAllUserDTOs();

    // 9. Pagination & Sorting
    Page<User> findByRole(UserRole role, Pageable pageable);

    List<User> findByRole(UserRole role, Sort sort);

    // 10. Custom Query with Join
    @Query("SELECT DISTINCT u FROM User u JOIN u.posts p WHERE p.title LIKE %:keyword%")
    List<User> findUsersWithPostsContaining(@Param("keyword") String keyword);
}
```

#### 3. Service Layer

```java
@Service
@Transactional
public class UserService {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private PasswordEncoder passwordEncoder;

    // Create
    public User createUser(User user) {
        if (userRepository.existsByUsername(user.getUsername())) {
            throw new DuplicateUserException("Username already exists");
        }
        if (userRepository.existsByEmail(user.getEmail())) {
            throw new DuplicateUserException("Email already exists");
        }

        user.setPassword(passwordEncoder.encode(user.getPassword()));
        return userRepository.save(user);
    }

    // Read
    @Transactional(readOnly = true)
    public User getUserById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException("User not found"));
    }

    @Transactional(readOnly = true)
    public User getUserByUsername(String username) {
        return userRepository.findByUsername(username)
            .orElseThrow(() -> new UserNotFoundException("User not found"));
    }

    @Transactional(readOnly = true)
    public Page<User> getAllUsers(int page, int size, String sortBy) {
        Pageable pageable = PageRequest.of(page, size, Sort.by(sortBy));
        return userRepository.findAll(pageable);
    }

    // Update
    public User updateUser(Long id, User updatedUser) {
        User user = getUserById(id);
        user.setEmail(updatedUser.getEmail());
        // update other fields
        return userRepository.save(user);
    }

    // Delete
    public void deleteUser(Long id) {
        if (!userRepository.existsById(id)) {
            throw new UserNotFoundException("User not found");
        }
        userRepository.deleteById(id);
    }

    // Search
    @Transactional(readOnly = true)
    public List<User> searchUsers(String keyword) {
        return userRepository.searchByUsername(keyword);
    }
}
```

#### 4. Pagination Example

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping
    public ResponseEntity<Page<UserDTO>> getUsers(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(defaultValue = "id") String sortBy,
        @RequestParam(defaultValue = "ASC") String direction
    ) {
        Sort.Direction sortDirection = Sort.Direction.fromString(direction);
        Pageable pageable = PageRequest.of(page, size, Sort.by(sortDirection, sortBy));

        Page<User> users = userRepository.findAll(pageable);
        Page<UserDTO> userDTOs = users.map(this::convertToDTO);

        return ResponseEntity.ok(userDTOs);
    }
}
```

#### 5. Custom Repository Implementation

```java
// Interface
public interface UserRepositoryCustom {
    List<User> findUsersByCustomCriteria(String criteria);
}

// Implementation
@Repository
public class UserRepositoryCustomImpl implements UserRepositoryCustom {

    @PersistenceContext
    private EntityManager entityManager;

    @Override
    public List<User> findUsersByCustomCriteria(String criteria) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<User> query = cb.createQuery(User.class);
        Root<User> user = query.from(User.class);

        // Complex custom query logic
        query.select(user).where(/* custom conditions */);

        return entityManager.createQuery(query).getResultList();
    }
}

// Extend main repository
public interface UserRepository extends JpaRepository<User, Long>, UserRepositoryCustom {
    // inherited methods + custom methods
}
```

---

### 17. شرح @Transactional

**الإجابة:**

@Transactional يضمن أن العمليات تتم كـ transaction واحدة (كل شيء ينجح أو كل شيء يفشل):

#### 1. الاستخدام الأساسي

```java
@Service
public class BankService {

    @Autowired
    private AccountRepository accountRepository;

    // إذا حدث خطأ، كل التغييرات ستُلغى
    @Transactional
    public void transferMoney(Long fromId, Long toId, Double amount) {
        Account from = accountRepository.findById(fromId)
            .orElseThrow(() -> new AccountNotFoundException());
        Account to = accountRepository.findById(toId)
            .orElseThrow(() -> new AccountNotFoundException());

        if (from.getBalance() < amount) {
            throw new InsufficientBalanceException();
        }

        from.setBalance(from.getBalance() - amount);
        to.setBalance(to.getBalance() + amount);

        accountRepository.save(from);
        accountRepository.save(to);
        // إذا حدث exception هنا، كل التغييرات السابقة ستُلغى

        // إذا اكتمل كل شيء بنجاح، سيتم commit
    }
}
```

#### 2. Read-Only Transactions

```java
@Service
public class UserService {

    // للـ Read operations فقط (أسرع وأكثر أماناً)
    @Transactional(readOnly = true)
    public User getUserById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException());
    }

    @Transactional(readOnly = true)
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }
}
```

#### 3. Propagation

```java
@Service
public class OrderService {

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private AuditService auditService;

    // REQUIRED (default) - يستخدم transaction موجودة أو ينشئ جديدة
    @Transactional(propagation = Propagation.REQUIRED)
    public Order createOrder(Order order) {
        Order saved = orderRepository.save(order);
        auditService.logOrderCreation(saved); // سيستخدم نفس الـ transaction
        return saved;
    }

    // REQUIRES_NEW - دائماً ينشئ transaction جديدة
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void sendNotification(Order order) {
        // حتى لو فشلت الـ transaction الأساسية، هذه ستُنفذ
        notificationRepository.save(new Notification(order));
    }

    // SUPPORTS - إذا يوجد transaction يستخدمها، وإلا يعمل بدون
    @Transactional(propagation = Propagation.SUPPORTS)
    public List<Order> getOrders() {
        return orderRepository.findAll();
    }

    // MANDATORY - يجب أن يكون هناك transaction موجودة
    @Transactional(propagation = Propagation.MANDATORY)
    public void updateOrderStatus(Long id, OrderStatus status) {
        Order order = orderRepository.findById(id).orElseThrow();
        order.setStatus(status);
        orderRepository.save(order);
    }

    // NEVER - لا يجب أن يكون هناك transaction
    @Transactional(propagation = Propagation.NEVER)
    public void logActivity(String activity) {
        // عملية لا تحتاج transaction
    }
}
```

#### 4. Isolation Levels

```java
@Service
public class ProductService {

    // READ_UNCOMMITTED - يقرأ بيانات غير مُثبتة (غير آمن)
    @Transactional(isolation = Isolation.READ_UNCOMMITTED)
    public List<Product> getProducts() {
        return productRepository.findAll();
    }

    // READ_COMMITTED (default) - يقرأ بيانات مُثبتة فقط
    @Transactional(isolation = Isolation.READ_COMMITTED)
    public Product getProduct(Long id) {
        return productRepository.findById(id).orElseThrow();
    }

    // REPEATABLE_READ - نفس القراءة في نفس الـ transaction
    @Transactional(isolation = Isolation.REPEATABLE_READ)
    public void processOrder(Long productId) {
        Product product = productRepository.findById(productId).orElseThrow();
        // حتى لو تغيّر المنتج في transaction أخرى، هنا سنرى نفس القيم
        product.setStock(product.getStock() - 1);
        productRepository.save(product);
    }

    // SERIALIZABLE - أعلى مستوى أمان (أبطأ)
    @Transactional(isolation = Isolation.SERIALIZABLE)
    public void criticalOperation() {
        // العمليات تتم واحدة تلو الأخرى
    }
}
```

#### 5. Rollback

```java
@Service
public class PaymentService {

    // Rollback على أي Exception
    @Transactional(rollbackFor = Exception.class)
    public void processPayment(Payment payment) {
        // سيتم rollback حتى على checked exceptions
        paymentRepository.save(payment);
        if (payment.getAmount() > 10000) {
            throw new PaymentLimitException(); // rollback
        }
    }

    // لا يعمل rollback على exceptions معينة
    @Transactional(noRollbackFor = ValidationException.class)
    public void createUser(User user) {
        userRepository.save(user);
        if (!isValid(user)) {
            throw new ValidationException(); // لن يعمل rollback
        }
    }

    // Rollback فقط على RuntimeException (default)
    @Transactional
    public void defaultBehavior() {
        // rollback على RuntimeException فقط
        // لا يعمل rollback على checked exceptions
    }
}
```

#### 6. Timeout

```java
@Service
public class ReportService {

    // timeout بعد 5 ثواني
    @Transactional(timeout = 5)
    public Report generateReport() {
        // إذا أخذت العملية أكثر من 5 ثواني، سيحدث TransactionTimedOutException
        return reportRepository.generateComplexReport();
    }
}
```

#### 7. مثال عملي كامل

```java
@Service
public class OrderService {

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private ProductRepository productRepository;

    @Autowired
    private PaymentService paymentService;

    @Autowired
    private NotificationService notificationService;

    @Transactional(
        isolation = Isolation.READ_COMMITTED,
        propagation = Propagation.REQUIRED,
        rollbackFor = Exception.class,
        timeout = 10
    )
    public Order createOrder(OrderRequest request) {
        // 1. Check product availability
        Product product = productRepository.findById(request.getProductId())
            .orElseThrow(() -> new ProductNotFoundException());

        if (product.getStock() < request.getQuantity()) {
            throw new InsufficientStockException();
        }

        // 2. Update stock
        product.setStock(product.getStock() - request.getQuantity());
        productRepository.save(product);

        // 3. Create order
        Order order = new Order();
        order.setProduct(product);
        order.setQuantity(request.getQuantity());
        order.setTotalPrice(product.getPrice() * request.getQuantity());
        order.setStatus(OrderStatus.PENDING);
        order = orderRepository.save(order);

        // 4. Process payment
        paymentService.processPayment(order);

        // 5. Update order status
        order.setStatus(OrderStatus.COMPLETED);
        order = orderRepository.save(order);

        // 6. Send notification (في transaction منفصلة)
        notificationService.sendOrderConfirmation(order);

        return order;
        // إذا حدث exception في أي مرحلة، كل شيء سيُلغى
    }
}

@Service
public class NotificationService {

    // Transaction منفصلة - حتى لو فشل الـ order، الـ notification ستُرسل
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void sendOrderConfirmation(Order order) {
        // send email/sms
        notificationRepository.save(new Notification(order));
    }
}
```

#### 8. أخطاء شائعة

```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    // ❌ لن يعمل - @Transactional لا يعمل على private methods
    @Transactional
    private void privateMethod() {
        // Transaction لن تُنشأ
    }

    // ❌ لن يعمل - الاستدعاء الداخلي
    public void publicMethod() {
        this.transactionalMethod(); // Transaction لن تُنشأ
    }

    @Transactional
    public void transactionalMethod() {
        // ...
    }

    // ✅ يعمل بشكل صحيح
    @Transactional
    public void correctUsage() {
        userRepository.save(new User());
    }
}
```

---

## REST APIs

### 18. كيف تنشئ REST API في Spring Boot؟

**الإجابة:**

#### 1. Entity Class

```java
@Entity
@Table(name = "products")
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(columnDefinition = "TEXT")
    private String description;

    @Column(nullable = false)
    private Double price;

    @Column(nullable = false)
    private Integer stock;

    @Enumerated(EnumType.STRING)
    private ProductCategory category;

    private LocalDateTime createdAt;

    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
    }

    // getters/setters
}
```

#### 2. Repository

```java
@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {

    List<Product> findByCategory(ProductCategory category);

    List<Product> findByPriceBetween(Double minPrice, Double maxPrice);

    @Query("SELECT p FROM Product p WHERE p.name LIKE %:keyword% OR p.description LIKE %:keyword%")
    List<Product> searchProducts(@Param("keyword") String keyword);
}
```

#### 3. DTO Classes

```java
// Request DTO
public class ProductRequest {

    @NotBlank(message = "Name is required")
    @Size(min = 3, max = 100)
    private String name;

    @Size(max = 500)
    private String description;

    @NotNull(message = "Price is required")
    @Positive
    private Double price;

    @NotNull
    @Min(0)
    private Integer stock;

    @NotNull
    private ProductCategory category;

    // getters/setters
}

// Response DTO
public class ProductResponse {

    private Long id;
    private String name;
    private String description;
    private Double price;
    private Integer stock;
    private ProductCategory category;
    private LocalDateTime createdAt;

    // Constructor, getters/setters

    public static ProductResponse fromEntity(Product product) {
        ProductResponse response = new ProductResponse();
        response.setId(product.getId());
        response.setName(product.getName());
        response.setDescription(product.getDescription());
        response.setPrice(product.getPrice());
        response.setStock(product.getStock());
        response.setCategory(product.getCategory());
        response.setCreatedAt(product.getCreatedAt());
        return response;
    }
}
```

#### 4. Service Layer

```java
@Service
@Transactional
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    public ProductResponse createProduct(ProductRequest request) {
        Product product = new Product();
        product.setName(request.getName());
        product.setDescription(request.getDescription());
        product.setPrice(request.getPrice());
        product.setStock(request.getStock());
        product.setCategory(request.getCategory());

        product = productRepository.save(product);
        return ProductResponse.fromEntity(product);
    }

    @Transactional(readOnly = true)
    public ProductResponse getProduct(Long id) {
        Product product = productRepository.findById(id)
            .orElseThrow(() -> new ProductNotFoundException("Product not found with id: " + id));
        return ProductResponse.fromEntity(product);
    }

    @Transactional(readOnly = true)
    public Page<ProductResponse> getAllProducts(Pageable pageable) {
        return productRepository.findAll(pageable)
            .map(ProductResponse::fromEntity);
    }

    public ProductResponse updateProduct(Long id, ProductRequest request) {
        Product product = productRepository.findById(id)
            .orElseThrow(() -> new ProductNotFoundException("Product not found"));

        product.setName(request.getName());
        product.setDescription(request.getDescription());
        product.setPrice(request.getPrice());
        product.setStock(request.getStock());
        product.setCategory(request.getCategory());

        product = productRepository.save(product);
        return ProductResponse.fromEntity(product);
    }

    public void deleteProduct(Long id) {
        if (!productRepository.existsById(id)) {
            throw new ProductNotFoundException("Product not found");
        }
        productRepository.deleteById(id);
    }

    @Transactional(readOnly = true)
    public List<ProductResponse> searchProducts(String keyword) {
        return productRepository.searchProducts(keyword)
            .stream()
            .map(ProductResponse::fromEntity)
            .collect(Collectors.toList());
    }
}
```

#### 5. REST Controller

```java
@RestController
@RequestMapping("/api/products")
@Validated
public class ProductController {

    @Autowired
    private ProductService productService;

    // CREATE - POST /api/products
    @PostMapping
    public ResponseEntity<ProductResponse> createProduct(
        @Valid @RequestBody ProductRequest request
    ) {
        ProductResponse response = productService.createProduct(request);
        return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(response);
    }

    // READ ONE - GET /api/products/1
    @GetMapping("/{id}")
    public ResponseEntity<ProductResponse> getProduct(@PathVariable Long id) {
        ProductResponse response = productService.getProduct(id);
        return ResponseEntity.ok(response);
    }

    // READ ALL - GET /api/products?page=0&size=10&sort=name,asc
    @GetMapping
    public ResponseEntity<Page<ProductResponse>> getAllProducts(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(defaultValue = "id") String sortBy,
        @RequestParam(defaultValue = "ASC") String direction
    ) {
        Sort.Direction sortDirection = Sort.Direction.fromString(direction);
        Pageable pageable = PageRequest.of(page, size, Sort.by(sortDirection, sortBy));

        Page<ProductResponse> products = productService.getAllProducts(pageable);
        return ResponseEntity.ok(products);
    }

    // UPDATE - PUT /api/products/1
    @PutMapping("/{id}")
    public ResponseEntity<ProductResponse> updateProduct(
        @PathVariable Long id,
        @Valid @RequestBody ProductRequest request
    ) {
        ProductResponse response = productService.updateProduct(id, request);
        return ResponseEntity.ok(response);
    }

    // DELETE - DELETE /api/products/1
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable Long id) {
        productService.deleteProduct(id);
        return ResponseEntity.noContent().build();
    }

    // SEARCH - GET /api/products/search?keyword=laptop
    @GetMapping("/search")
    public ResponseEntity<List<ProductResponse>> searchProducts(
        @RequestParam String keyword
    ) {
        List<ProductResponse> products = productService.searchProducts(keyword);
        return ResponseEntity.ok(products);
    }

    // FILTER BY CATEGORY - GET /api/products/category/ELECTRONICS
    @GetMapping("/category/{category}")
    public ResponseEntity<List<ProductResponse>> getProductsByCategory(
        @PathVariable ProductCategory category
    ) {
        // implementation
        return ResponseEntity.ok(products);
    }

    // FILTER BY PRICE RANGE - GET /api/products/price?min=100&max=500
    @GetMapping("/price")
    public ResponseEntity<List<ProductResponse>> getProductsByPriceRange(
        @RequestParam Double min,
        @RequestParam Double max
    ) {
        // implementation
        return ResponseEntity.ok(products);
    }
}
```

#### 6. Response Wrapper (اختياري)

```java
// Generic API Response
public class ApiResponse<T> {

    private boolean success;
    private String message;
    private T data;
    private LocalDateTime timestamp;

    public ApiResponse(boolean success, String message, T data) {
        this.success = success;
        this.message = message;
        this.data = data;
        this.timestamp = LocalDateTime.now();
    }

    public static <T> ApiResponse<T> success(T data) {
        return new ApiResponse<>(true, "Success", data);
    }

    public static <T> ApiResponse<T> success(String message, T data) {
        return new ApiResponse<>(true, message, data);
    }

    public static <T> ApiResponse<T> error(String message) {
        return new ApiResponse<>(false, message, null);
    }

    // getters/setters
}

// استخدامه في Controller
@RestController
@RequestMapping("/api/products")
public class ProductController {

    @PostMapping
    public ResponseEntity<ApiResponse<ProductResponse>> createProduct(
        @Valid @RequestBody ProductRequest request
    ) {
        ProductResponse product = productService.createProduct(request);
        return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(ApiResponse.success("Product created successfully", product));
    }
}
```

#### 7. Pagination Response

```java
public class PagedResponse<T> {

    private List<T> content;
    private int pageNumber;
    private int pageSize;
    private long totalElements;
    private int totalPages;
    private boolean last;

    public PagedResponse(Page<T> page) {
        this.content = page.getContent();
        this.pageNumber = page.getNumber();
        this.pageSize = page.getSize();
        this.totalElements = page.getTotalElements();
        this.totalPages = page.getTotalPages();
        this.last = page.isLast();
    }

    // getters/setters
}
```

---

## Exception Handling

### 19. كيف تتعامل مع Exceptions في Spring Boot؟

**الإجابة:**

#### 1. Custom Exceptions

```java
// Base Exception
public class ApiException extends RuntimeException {

    private HttpStatus status;

    public ApiException(String message, HttpStatus status) {
        super(message);
        this.status = status;
    }

    public HttpStatus getStatus() {
        return status;
    }
}

// Specific Exceptions
public class ResourceNotFoundException extends ApiException {
    public ResourceNotFoundException(String message) {
        super(message, HttpStatus.NOT_FOUND);
    }
}

public class DuplicateResourceException extends ApiException {
    public DuplicateResourceException(String message) {
        super(message, HttpStatus.CONFLICT);
    }
}

public class BadRequestException extends ApiException {
    public BadRequestException(String message) {
        super(message, HttpStatus.BAD_REQUEST);
    }
}

public class UnauthorizedException extends ApiException {
    public UnauthorizedException(String message) {
        super(message, HttpStatus.UNAUTHORIZED);
    }
}
```

#### 2. Error Response DTO

```java
@JsonInclude(JsonInclude.Include.NON_NULL)
public class ErrorResponse {

    private boolean success;
    private String message;
    private HttpStatus status;
    private int statusCode;
    private LocalDateTime timestamp;
    private String path;
    private List<String> errors;

    public ErrorResponse() {
        this.success = false;
        this.timestamp = LocalDateTime.now();
    }

    public ErrorResponse(String message, HttpStatus status, String path) {
        this();
        this.message = message;
        this.status = status;
        this.statusCode = status.value();
        this.path = path;
    }

    public ErrorResponse(String message, HttpStatus status, String path, List<String> errors) {
        this(message, status, path);
        this.errors = errors;
    }

    // getters/setters
}
```

#### 3. Global Exception Handler

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    // 1. Handle Custom API Exceptions
    @ExceptionHandler(ApiException.class)
    public ResponseEntity<ErrorResponse> handleApiException(
        ApiException ex,
        WebRequest request
    ) {
        ErrorResponse error = new ErrorResponse(
            ex.getMessage(),
            ex.getStatus(),
            request.getDescription(false).replace("uri=", "")
        );

        return ResponseEntity.status(ex.getStatus()).body(error);
    }

    // 2. Handle Resource Not Found
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFoundException(
        ResourceNotFoundException ex,
        WebRequest request
    ) {
        ErrorResponse error = new ErrorResponse(
            ex.getMessage(),
            HttpStatus.NOT_FOUND,
            request.getDescription(false).replace("uri=", "")
        );

        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    // 3. Handle Validation Errors
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationException(
        MethodArgumentNotValidException ex,
        WebRequest request
    ) {
        List<String> errors = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(error -> error.getField() + ": " + error.getDefaultMessage())
            .collect(Collectors.toList());

        ErrorResponse error = new ErrorResponse(
            "Validation failed",
            HttpStatus.BAD_REQUEST,
            request.getDescription(false).replace("uri=", ""),
            errors
        );

        return ResponseEntity.badRequest().body(error);
    }

    // 4. Handle Missing Parameters
    @ExceptionHandler(MissingServletRequestParameterException.class)
    public ResponseEntity<ErrorResponse> handleMissingParams(
        MissingServletRequestParameterException ex,
        WebRequest request
    ) {
        ErrorResponse error = new ErrorResponse(
            String.format("Missing required parameter: %s", ex.getParameterName()),
            HttpStatus.BAD_REQUEST,
            request.getDescription(false).replace("uri=", "")
        );

        return ResponseEntity.badRequest().body(error);
    }

    // 5. Handle Type Mismatch
    @ExceptionHandler(MethodArgumentTypeMismatchException.class)
    public ResponseEntity<ErrorResponse> handleTypeMismatch(
        MethodArgumentTypeMismatchException ex,
        WebRequest request
    ) {
        String error = String.format("Parameter '%s' should be of type %s",
            ex.getName(),
            ex.getRequiredType().getSimpleName()
        );

        ErrorResponse errorResponse = new ErrorResponse(
            error,
            HttpStatus.BAD_REQUEST,
            request.getDescription(false).replace("uri=", "")
        );

        return ResponseEntity.badRequest().body(errorResponse);
    }

    // 6. Handle Constraint Violations
    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<ErrorResponse> handleConstraintViolation(
        ConstraintViolationException ex,
        WebRequest request
    ) {
        List<String> errors = ex.getConstraintViolations()
            .stream()
            .map(violation -> violation.getPropertyPath() + ": " + violation.getMessage())
            .collect(Collectors.toList());

        ErrorResponse error = new ErrorResponse(
            "Validation failed",
            HttpStatus.BAD_REQUEST,
            request.getDescription(false).replace("uri=", ""),
            errors
        );

        return ResponseEntity.badRequest().body(error);
    }

    // 7. Handle HTTP Message Not Readable
    @ExceptionHandler(HttpMessageNotReadableException.class)
    public ResponseEntity<ErrorResponse> handleHttpMessageNotReadable(
        HttpMessageNotReadableException ex,
        WebRequest request
    ) {
        ErrorResponse error = new ErrorResponse(
            "Malformed JSON request",
            HttpStatus.BAD_REQUEST,
            request.getDescription(false).replace("uri=", "")
        );

        return ResponseEntity.badRequest().body(error);
    }

    // 8. Handle HTTP Request Method Not Supported
    @ExceptionHandler(HttpRequestMethodNotSupportedException.class)
    public ResponseEntity<ErrorResponse> handleMethodNotSupported(
        HttpRequestMethodNotSupportedException ex,
        WebRequest request
    ) {
        ErrorResponse error = new ErrorResponse(
            String.format("Request method '%s' not supported", ex.getMethod()),
            HttpStatus.METHOD_NOT_ALLOWED,
            request.getDescription(false).replace("uri=", "")
        );

        return ResponseEntity.status(HttpStatus.METHOD_NOT_ALLOWED).body(error);
    }

    // 9. Handle Access Denied
    @ExceptionHandler(AccessDeniedException.class)
    public ResponseEntity<ErrorResponse> handleAccessDenied(
        AccessDeniedException ex,
        WebRequest request
    ) {
        ErrorResponse error = new ErrorResponse(
            "Access denied",
            HttpStatus.FORBIDDEN,
            request.getDescription(false).replace("uri=", "")
        );

        return ResponseEntity.status(HttpStatus.FORBIDDEN).body(error);
    }

    // 10. Handle Database Exceptions
    @ExceptionHandler(DataIntegrityViolationException.class)
    public ResponseEntity<ErrorResponse> handleDataIntegrityViolation(
        DataIntegrityViolationException ex,
        WebRequest request
    ) {
        ErrorResponse error = new ErrorResponse(
            "Database constraint violation",
            HttpStatus.CONFLICT,
            request.getDescription(false).replace("uri=", "")
        );

        return ResponseEntity.status(HttpStatus.CONFLICT).body(error);
    }

    // 11. Handle All Other Exceptions
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGlobalException(
        Exception ex,
        WebRequest request
    ) {
        ErrorResponse error = new ErrorResponse(
            "An error occurred",
            HttpStatus.INTERNAL_SERVER_ERROR,
            request.getDescription(false).replace("uri=", "")
        );

        // Log the exception
        ex.printStackTrace();

        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

#### 4. Controller-Specific Exception Handler

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @Autowired
    private UserService userService;

    // سيتعامل فقط مع exceptions في هذا الـ controller
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(
        UserNotFoundException ex,
        WebRequest request
    ) {
        ErrorResponse error = new ErrorResponse(
            ex.getMessage(),
            HttpStatus.NOT_FOUND,
            request.getDescription(false)
        );

        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        User user = userService.getUserById(id);
        return ResponseEntity.ok(user);
    }
}
```

#### 5. استخدام @ResponseStatus

```java
@ResponseStatus(HttpStatus.NOT_FOUND)
public class ProductNotFoundException extends RuntimeException {
    public ProductNotFoundException(String message) {
        super(message);
    }
}

@ResponseStatus(HttpStatus.BAD_REQUEST)
public class InvalidInputException extends RuntimeException {
    public InvalidInputException(String message) {
        super(message);
    }
}

// في الـ Service
@Service
public class ProductService {

    public Product getProduct(Long id) {
        return productRepository.findById(id)
            .orElseThrow(() -> new ProductNotFoundException("Product not found with id: " + id));
    }
}
```

#### 6. Validation مع Custom Messages

```java
public class UserRequest {

    @NotBlank(message = "Username is required")
    @Size(min = 3, max = 50, message = "Username must be between 3 and 50 characters")
    private String username;

    @NotBlank(message = "Email is required")
    @Email(message = "Email must be valid")
    private String email;

    @NotBlank(message = "Password is required")
    @Size(min = 8, message = "Password must be at least 8 characters")
    @Pattern(
        regexp = "^(?=.*[0-9])(?=.*[a-z])(?=.*[A-Z])(?=.*[@#$%^&+=]).*$",
        message = "Password must contain at least one digit, one lowercase, one uppercase, and one special character"
    )
    private String password;

    @Min(value = 18, message = "Age must be at least 18")
    @Max(value = 100, message = "Age must not exceed 100")
    private Integer age;

    // getters/setters
}

@RestController
@RequestMapping("/api/users")
public class UserController {

    @PostMapping
    public ResponseEntity<User> createUser(@Valid @RequestBody UserRequest request) {
        // إذا فشل الـ validation، سيتم رمي MethodArgumentNotValidException
        // وسيتم التعامل معها في GlobalExceptionHandler
        User user = userService.createUser(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(user);
    }
}
```

---

## Security

### 20. ما هو Spring Security؟

**الإجابة:**

Spring Security هو إطار عمل شامل للأمان يوفر Authentication و Authorization.

#### 1. الإعداد الأساسي

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

#### 2. Basic Security Configuration

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable() // إيقاف CSRF للتبسيط (لا تفعل في production)
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()  // مفتوح للجميع
                .requestMatchers("/api/admin/**").hasRole("ADMIN")  // للـ ADMIN فقط
                .requestMatchers("/api/user/**").hasAnyRole("USER", "ADMIN")  // للـ USER و ADMIN
                .anyRequest().authenticated()  // أي طلب آخر يحتاج تسجيل دخول
            )
            .httpBasic(); // استخدام Basic Authentication

        return http.build();
    }

    // In-Memory Users (للتجربة فقط)
    @Bean
    public InMemoryUserDetailsManager userDetailsService() {
        UserDetails user = User.builder()
            .username("user")
            .password(passwordEncoder().encode("password"))
            .roles("USER")
            .build();

        UserDetails admin = User.builder()
            .username("admin")
            .password(passwordEncoder().encode("admin"))
            .roles("ADMIN", "USER")
            .build();

        return new InMemoryUserDetailsManager(user, admin);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

#### 3. Database Authentication

```java
// User Entity
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true, nullable = false)
    private String username;

    @Column(nullable = false)
    private String password;

    @Column(unique = true, nullable = false)
    private String email;

    private boolean enabled = true;

    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles = new HashSet<>();

    // getters/setters
}

@Entity
@Table(name = "roles")
public class Role {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Enumerated(EnumType.STRING)
    @Column(unique = true)
    private RoleName name;

    // getters/setters
}

public enum RoleName {
    ROLE_USER,
    ROLE_ADMIN,
    ROLE_MODERATOR
}

// UserDetails Implementation
public class UserPrincipal implements UserDetails {

    private User user;

    public UserPrincipal(User user) {
        this.user = user;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return user.getRoles().stream()
            .map(role -> new SimpleGrantedAuthority(role.getName().name()))
            .collect(Collectors.toList());
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getUsername();
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return user.isEnabled();
    }

    public Long getId() {
        return user.getId();
    }
}

// UserDetailsService Implementation
@Service
public class CustomUserDetailsService implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;

    @Override
    @Transactional(readOnly = true)
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));

        return new UserPrincipal(user);
    }
}

// Security Configuration
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Autowired
    private CustomUserDetailsService userDetailsService;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .httpBasic();

        return http.build();
    }

    @Bean
    public AuthenticationManager authenticationManager(
        AuthenticationConfiguration authConfig
    ) throws Exception {
        return authConfig.getAuthenticationManager();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

#### 4. JWT Authentication

```java
// JWT Utility Class
@Component
public class JwtUtils {

    @Value("${app.jwt.secret}")
    private String jwtSecret;

    @Value("${app.jwt.expiration}")
    private long jwtExpiration;

    public String generateToken(UserPrincipal userPrincipal) {
        return Jwts.builder()
            .setSubject(userPrincipal.getUsername())
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + jwtExpiration))
            .signWith(SignatureAlgorithm.HS512, jwtSecret)
            .compact();
    }

    public String getUsernameFromToken(String token) {
        return Jwts.parser()
            .setSigningKey(jwtSecret)
            .parseClaimsJws(token)
            .getBody()
            .getSubject();
    }

    public boolean validateToken(String token) {
        try {
            Jwts.parser().setSigningKey(jwtSecret).parseClaimsJws(token);
            return true;
        } catch (Exception e) {
            return false;
        }
    }
}

// JWT Authentication Filter
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    @Autowired
    private JwtUtils jwtUtils;

    @Autowired
    private CustomUserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(
        HttpServletRequest request,
        HttpServletResponse response,
        FilterChain filterChain
    ) throws ServletException, IOException {

        try {
            String jwt = getJwtFromRequest(request);

            if (jwt != null && jwtUtils.validateToken(jwt)) {
                String username = jwtUtils.getUsernameFromToken(jwt);
                UserDetails userDetails = userDetailsService.loadUserByUsername(username);

                UsernamePasswordAuthenticationToken authentication =
                    new UsernamePasswordAuthenticationToken(
                        userDetails,
                        null,
                        userDetails.getAuthorities()
                    );

                authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                SecurityContextHolder.getContext().setAuthentication(authentication);
            }
        } catch (Exception e) {
            logger.error("Cannot set user authentication", e);
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

// Security Configuration with JWT
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Autowired
    private JwtAuthenticationFilter jwtAuthenticationFilter;

    @Autowired
    private CustomUserDetailsService userDetailsService;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .and()
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}

// Auth Controller
@RestController
@RequestMapping("/api/auth")
public class AuthController {

    @Autowired
    private AuthenticationManager authenticationManager;

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private RoleRepository roleRepository;

    @Autowired
    private PasswordEncoder passwordEncoder;

    @Autowired
    private JwtUtils jwtUtils;

    @PostMapping("/login")
    public ResponseEntity<?> login(@Valid @RequestBody LoginRequest request) {
        Authentication authentication = authenticationManager.authenticate(
            new UsernamePasswordAuthenticationToken(
                request.getUsername(),
                request.getPassword()
            )
        );

        SecurityContextHolder.getContext().setAuthentication(authentication);
        UserPrincipal userPrincipal = (UserPrincipal) authentication.getPrincipal();
        String jwt = jwtUtils.generateToken(userPrincipal);

        return ResponseEntity.ok(new JwtResponse(jwt));
    }

    @PostMapping("/register")
    public ResponseEntity<?> register(@Valid @RequestBody RegisterRequest request) {
        if (userRepository.existsByUsername(request.getUsername())) {
            return ResponseEntity.badRequest().body("Username already exists");
        }

        if (userRepository.existsByEmail(request.getEmail())) {
            return ResponseEntity.badRequest().body("Email already exists");
        }

        User user = new User();
        user.setUsername(request.getUsername());
        user.setEmail(request.getEmail());
        user.setPassword(passwordEncoder.encode(request.getPassword()));

        Role userRole = roleRepository.findByName(RoleName.ROLE_USER)
            .orElseThrow(() -> new RuntimeException("Role not found"));
        user.getRoles().add(userRole);

        userRepository.save(user);

        return ResponseEntity.ok("User registered successfully");
    }
}
```

#### 5. Method-Level Security

```java
@Configuration
@EnableGlobalMethodSecurity(
    prePostEnabled = true,   // @PreAuthorize, @PostAuthorize
    securedEnabled = true,   // @Secured
    jsr250Enabled = true     // @RolesAllowed
)
public class MethodSecurityConfig {
}

@Service
public class ProductService {

    // فقط ADMIN يمكنه حذف المنتج
    @PreAuthorize("hasRole('ADMIN')")
    public void deleteProduct(Long id) {
        productRepository.deleteById(id);
    }

    // فقط صاحب المنتج أو ADMIN
    @PreAuthorize("hasRole('ADMIN') or @productService.isOwner(#id, authentication.principal.id)")
    public void updateProduct(Long id, Product product) {
        // update logic
    }

    public boolean isOwner(Long productId, Long userId) {
        Product product = productRepository.findById(productId).orElse(null);
        return product != null && product.getUserId().equals(userId);
    }

    // باستخدام @Secured
    @Secured({"ROLE_ADMIN", "ROLE_MODERATOR"})
    public void moderateProduct(Long id) {
        // moderation logic
    }

    // باستخدام @RolesAllowed
    @RolesAllowed("ADMIN")
    public void adminOnlyMethod() {
        // admin logic
    }
}
```

---

## Testing

### 21. كيف تختبر تطبيق Spring Boot؟

**الإجابة:**

#### 1. الإعداد

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

#### 2. Unit Testing - Service Layer

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @Mock
    private PasswordEncoder passwordEncoder;

    @InjectMocks
    private UserService userService;

    @Test
    void createUser_Success() {
        // Arrange
        UserRequest request = new UserRequest();
        request.setUsername("testuser");
        request.setEmail("test@example.com");
        request.setPassword("password123");

        User user = new User();
        user.setId(1L);
        user.setUsername("testuser");
        user.setEmail("test@example.com");

        when(userRepository.existsByUsername("testuser")).thenReturn(false);
        when(userRepository.existsByEmail("test@example.com")).thenReturn(false);
        when(passwordEncoder.encode("password123")).thenReturn("encodedPassword");
        when(userRepository.save(any(User.class))).thenReturn(user);

        // Act
        UserResponse response = userService.createUser(request);

        // Assert
        assertNotNull(response);
        assertEquals("testuser", response.getUsername());
        assertEquals("test@example.com", response.getEmail());

        verify(userRepository).existsByUsername("testuser");
        verify(userRepository).existsByEmail("test@example.com");
        verify(passwordEncoder).encode("password123");
        verify(userRepository).save(any(User.class));
    }

    @Test
    void createUser_DuplicateUsername_ThrowsException() {
        // Arrange
        UserRequest request = new UserRequest();
        request.setUsername("existing");

        when(userRepository.existsByUsername("existing")).thenReturn(true);

        // Act & Assert
        assertThrows(DuplicateUserException.class, () -> {
            userService.createUser(request);
        });

        verify(userRepository).existsByUsername("existing");
        verify(userRepository, never()).save(any(User.class));
    }

    @Test
    void getUserById_UserExists_ReturnsUser() {
        // Arrange
        Long userId = 1L;
        User user = new User();
        user.setId(userId);
        user.setUsername("testuser");

        when(userRepository.findById(userId)).thenReturn(Optional.of(user));

        // Act
        UserResponse response = userService.getUserById(userId);

        // Assert
        assertNotNull(response);
        assertEquals(userId, response.getId());
        assertEquals("testuser", response.getUsername());
    }

    @Test
    void getUserById_UserNotFound_ThrowsException() {
        // Arrange
        Long userId = 999L;
        when(userRepository.findById(userId)).thenReturn(Optional.empty());

        // Act & Assert
        assertThrows(UserNotFoundException.class, () -> {
            userService.getUserById(userId);
        });
    }
}
```

#### 3. Integration Testing - Repository Layer

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private TestEntityManager entityManager;

    @Test
    void findByUsername_UserExists_ReturnsUser() {
        // Arrange
        User user = new User();
        user.setUsername("testuser");
        user.setEmail("test@example.com");
        user.setPassword("password");
        entityManager.persist(user);
        entityManager.flush();

        // Act
        Optional<User> found = userRepository.findByUsername("testuser");

        // Assert
        assertTrue(found.isPresent());
        assertEquals("testuser", found.get().getUsername());
    }

    @Test
    void findByUsername_UserNotExists_ReturnsEmpty() {
        // Act
        Optional<User> found = userRepository.findByUsername("nonexistent");

        // Assert
        assertFalse(found.isPresent());
    }

    @Test
    void existsByEmail_EmailExists_ReturnsTrue() {
        // Arrange
        User user = new User();
        user.setUsername("testuser");
        user.setEmail("test@example.com");
        user.setPassword("password");
        entityManager.persist(user);
        entityManager.flush();

        // Act
        boolean exists = userRepository.existsByEmail("test@example.com");

        // Assert
        assertTrue(exists);
    }
}
```

#### 4. Controller Testing - MockMvc

```java
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    void createUser_ValidRequest_ReturnsCreated() throws Exception {
        // Arrange
        UserRequest request = new UserRequest();
        request.setUsername("testuser");
        request.setEmail("test@example.com");
        request.setPassword("password123");

        UserResponse response = new UserResponse();
        response.setId(1L);
        response.setUsername("testuser");
        response.setEmail("test@example.com");

        when(userService.createUser(any(UserRequest.class))).thenReturn(response);

        // Act & Assert
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.username").value("testuser"))
            .andExpect(jsonPath("$.email").value("test@example.com"));

        verify(userService).createUser(any(UserRequest.class));
    }

    @Test
    void createUser_InvalidRequest_ReturnsBadRequest() throws Exception {
        // Arrange - Invalid email
        UserRequest request = new UserRequest();
        request.setUsername("testuser");
        request.setEmail("invalid-email");
        request.setPassword("pass");

        // Act & Assert
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isBadRequest());
    }

    @Test
    void getUser_UserExists_ReturnsUser() throws Exception {
        // Arrange
        Long userId = 1L;
        UserResponse response = new UserResponse();
        response.setId(userId);
        response.setUsername("testuser");

        when(userService.getUserById(userId)).thenReturn(response);

        // Act & Assert
        mockMvc.perform(get("/api/users/{id}", userId))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(userId))
            .andExpect(jsonPath("$.username").value("testuser"));
    }

    @Test
    void getUser_UserNotFound_ReturnsNotFound() throws Exception {
        // Arrange
        Long userId = 999L;
        when(userService.getUserById(userId))
            .thenThrow(new UserNotFoundException("User not found"));

        // Act & Assert
        mockMvc.perform(get("/api/users/{id}", userId))
            .andExpect(status().isNotFound());
    }
}
```

#### 5. Integration Testing - Full Application

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
class UserIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private ObjectMapper objectMapper;

    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
    }

    @Test
    void createUser_FullFlow_Success() throws Exception {
        // Arrange
        UserRequest request = new UserRequest();
        request.setUsername("testuser");
        request.setEmail("test@example.com");
        request.setPassword("password123");

        // Act - Create user
        MvcResult result = mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated())
            .andReturn();

        // Assert - User created
        String responseBody = result.getResponse().getContentAsString();
        UserResponse createdUser = objectMapper.readValue(responseBody, UserResponse.class);

        assertNotNull(createdUser.getId());
        assertEquals("testuser", createdUser.getUsername());

        // Act - Get user
        mockMvc.perform(get("/api/users/{id}", createdUser.getId()))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.username").value("testuser"));

        // Assert - User in database
        Optional<User> userInDb = userRepository.findById(createdUser.getId());
        assertTrue(userInDb.isPresent());
        assertEquals("testuser", userInDb.get().getUsername());
    }
}
```

#### 6. Testing with TestRestTemplate

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class UserRestTemplateTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private UserRepository userRepository;

    @LocalServerPort
    private int port;

    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
    }

    @Test
    void createUser_Success() {
        // Arrange
        UserRequest request = new UserRequest();
        request.setUsername("testuser");
        request.setEmail("test@example.com");
        request.setPassword("password123");

        String url = "http://localhost:" + port + "/api/users";

        // Act
        ResponseEntity<UserResponse> response = restTemplate.postForEntity(
            url,
            request,
            UserResponse.class
        );

        // Assert
        assertEquals(HttpStatus.CREATED, response.getStatusCode());
        assertNotNull(response.getBody());
        assertEquals("testuser", response.getBody().getUsername());
    }
}
```

#### 7. Testing with Profiles

```java
@SpringBootTest
@ActiveProfiles("test")
class UserServiceTestWithProfile {

    @Autowired
    private UserService userService;

    @Test
    void testWithTestProfile() {
        // سيستخدم application-test.properties
    }
}
```

```properties
# application-test.properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.jpa.hibernate.ddl-auto=create-drop
```

---

## Actuator

### 22. ما هو Spring Boot Actuator؟

**الإجابة:**

Actuator يوفر endpoints جاهزة لمراقبة وإدارة التطبيق.

#### 1. الإعداد

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

#### 2. Configuration

```properties
# application.properties

# تفعيل كل الـ endpoints
management.endpoints.web.exposure.include=*

# أو تحديد endpoints معينة
management.endpoints.web.exposure.include=health,info,metrics,env

# تغيير الـ base path
management.endpoints.web.base-path=/actuator

# تغيير port (اختياري)
management.server.port=9090

# إظهار تفاصيل الـ health
management.endpoint.health.show-details=always

# معلومات التطبيق
info.app.name=My Application
info.app.description=Spring Boot Application
info.app.version=1.0.0
info.company.name=My Company
```

#### 3. الـ Endpoints الأساسية

```bash
# 1. Health - حالة التطبيق
GET http://localhost:8080/actuator/health

Response:
{
  "status": "UP",
  "components": {
    "db": {
      "status": "UP",
      "details": {
        "database": "MySQL",
        "validationQuery": "isValid()"
      }
    },
    "diskSpace": {
      "status": "UP",
      "details": {
        "total": 500GB,
        "free": 200GB
      }
    }
  }
}

# 2. Info - معلومات التطبيق
GET http://localhost:8080/actuator/info

Response:
{
  "app": {
    "name": "My Application",
    "version": "1.0.0"
  }
}

# 3. Metrics - مقاييس الأداء
GET http://localhost:8080/actuator/metrics

Response:
{
  "names": [
    "jvm.memory.used",
    "jvm.memory.max",
    "http.server.requests",
    "system.cpu.usage"
  ]
}

# تفاصيل metric محدد
GET http://localhost:8080/actuator/metrics/jvm.memory.used

# 4. Env - Environment variables
GET http://localhost:8080/actuator/env

# 5. Loggers - إدارة الـ logging levels
GET http://localhost:8080/actuator/loggers

# تغيير log level
POST http://localhost:8080/actuator/loggers/com.example
{
  "configuredLevel": "DEBUG"
}

# 6. Beans - كل الـ Spring beans
GET http://localhost:8080/actuator/beans

# 7. Mappings - كل الـ endpoints
GET http://localhost:8080/actuator/mappings

# 8. Thread Dump
GET http://localhost:8080/actuator/threaddump

# 9. Heap Dump
GET http://localhost:8080/actuator/heapdump
```

#### 4. Custom Health Indicator

```java
@Component
public class DatabaseHealthIndicator implements HealthIndicator {

    @Autowired
    private DataSource dataSource;

    @Override
    public Health health() {
        try (Connection conn = dataSource.getConnection()) {
            if (conn.isValid(1000)) {
                return Health.up()
                    .withDetail("database", "Available")
                    .withDetail("type", "MySQL")
                    .build();
            }
        } catch (Exception e) {
            return Health.down()
                .withDetail("error", e.getMessage())
                .build();
        }
        return Health.down().build();
    }
}

@Component
public class CustomServiceHealthIndicator implements HealthIndicator {

    @Autowired
    private RestTemplate restTemplate;

    @Override
    public Health health() {
        try {
            ResponseEntity<String> response = restTemplate.getForEntity(
                "http://external-service/health",
                String.class
            );

            if (response.getStatusCode() == HttpStatus.OK) {
                return Health.up()
                    .withDetail("externalService", "Available")
                    .build();
            }
        } catch (Exception e) {
            return Health.down()
                .withDetail("externalService", "Unavailable")
                .withDetail("error", e.getMessage())
                .build();
        }
        return Health.down().build();
    }
}
```

#### 5. Custom Info Contributor

```java
@Component
public class CustomInfoContributor implements InfoContributor {

    @Autowired
    private UserRepository userRepository;

    @Override
    public void contribute(Info.Builder builder) {
        Map<String, Object> stats = new HashMap<>();
        stats.put("totalUsers", userRepository.count());
        stats.put("activeUsers", userRepository.countByActive(true));
        stats.put("timestamp", LocalDateTime.now());

        builder.withDetail("userStats", stats);
    }
}
```

#### 6. Custom Metrics

```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private MeterRegistry meterRegistry;

    public User createUser(User user) {
        User saved = userRepository.save(user);

        // Increment counter
        meterRegistry.counter("users.created").increment();

        return saved;
    }

    public void processOrder(Order order) {
        Timer.Sample sample = Timer.start(meterRegistry);

        // معالجة الطلب
        processOrderLogic(order);

        // Record time
        sample.stop(meterRegistry.timer("orders.processing.time"));
    }

    @Scheduled(fixedRate = 60000)
    public void recordActiveUsers() {
        long activeUsers = userRepository.countByActive(true);
        meterRegistry.gauge("users.active", activeUsers);
    }
}
```

#### 7. Security لـ Actuator

```java
@Configuration
public class ActuatorSecurityConfig {

    @Bean
    public SecurityFilterChain actuatorSecurity(HttpSecurity http) throws Exception {
        http
            .requestMatcher(EndpointRequest.toAnyEndpoint())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers(EndpointRequest.to("health", "info")).permitAll()
                .requestMatchers(EndpointRequest.toAnyEndpoint()).hasRole("ADMIN")
            );

        return http.build();
    }
}
```

---

## أسئلة متقدمة

### 23. ما هي Bean Lifecycle في Spring؟

**الإجابة:**

```java
@Component
public class BeanLifecycleDemo {

    // 1. Constructor
    public BeanLifecycleDemo() {
        System.out.println("1. Constructor called");
    }

    // 2. Setter Injection
    @Autowired
    public void setDependency(SomeDependency dependency) {
        System.out.println("2. Dependencies injected");
    }

    // 3. @PostConstruct
    @PostConstruct
    public void init() {
        System.out.println("3. @PostConstruct called");
        // Initialize resources
    }

    // 4. InitializingBean interface
    @Override
    public void afterPropertiesSet() {
        System.out.println("4. afterPropertiesSet called");
    }

    // 5. Custom init method
    public void customInit() {
        System.out.println("5. Custom init method");
    }

    // Bean ready to use

    // 6. @PreDestroy
    @PreDestroy
    public void cleanup() {
        System.out.println("6. @PreDestroy called");
        // Cleanup resources
    }

    // 7. DisposableBean interface
    @Override
    public void destroy() {
        System.out.println("7. destroy() called");
    }

    // 8. Custom destroy method
    public void customDestroy() {
        System.out.println("8. Custom destroy method");
    }
}

@Configuration
public class AppConfig {

    @Bean(initMethod = "customInit", destroyMethod = "customDestroy")
    public BeanLifecycleDemo lifecycleBean() {
        return new BeanLifecycleDemo();
    }
}
```

### 24. شرح AOP (Aspect-Oriented Programming)

**الإجابة:**

```java
@Aspect
@Component
public class LoggingAspect {

    // Before advice
    @Before("execution(* com.example.service.*.*(..))")
    public void logBefore(JoinPoint joinPoint) {
        System.out.println("Before method: " + joinPoint.getSignature().getName());
    }

    // After advice
    @After("execution(* com.example.service.*.*(..))")
    public void logAfter(JoinPoint joinPoint) {
        System.out.println("After method: " + joinPoint.getSignature().getName());
    }

    // AfterReturning advice
    @AfterReturning(
        pointcut = "execution(* com.example.service.*.*(..))",
        returning = "result"
    )
    public void logAfterReturning(JoinPoint joinPoint, Object result) {
        System.out.println("Method returned: " + result);
    }

    // AfterThrowing advice
    @AfterThrowing(
        pointcut = "execution(* com.example.service.*.*(..))",
        throwing = "error"
    )
    public void logAfterThrowing(JoinPoint joinPoint, Throwable error) {
        System.out.println("Exception: " + error.getMessage());
    }

    // Around advice
    @Around("execution(* com.example.service.*.*(..))")
    public Object logAround(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();

        Object result = joinPoint.proceed();

        long executionTime = System.currentTimeMillis() - start;
        System.out.println(joinPoint.getSignature() + " executed in " + executionTime + "ms");

        return result;
    }
}
```

### 25. Spring Boot مع Microservices

**الإجابة:**

```java
// Service Discovery - Eureka Server
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}

// Eureka Client
@SpringBootApplication
@EnableEurekaClient
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}

// Feign Client للتواصل بين الـ services
@FeignClient(name = "user-service")
public interface UserClient {

    @GetMapping("/api/users/{id}")
    User getUserById(@PathVariable Long id);
}

// API Gateway
@SpringBootApplication
public class ApiGatewayApplication {

    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            .route("user-service", r -> r.path("/users/**")
                .uri("lb://USER-SERVICE"))
            .route("product-service", r -> r.path("/products/**")
                .uri("lb://PRODUCT-SERVICE"))
            .build();
    }
}

// Circuit Breaker - Resilience4j
@Service
public class UserService {

    @Autowired
    private UserClient userClient;

    @CircuitBreaker(name = "userService", fallbackMethod = "getUserFallback")
    public User getUser(Long id) {
        return userClient.getUserById(id);
    }

    public User getUserFallback(Long id, Exception e) {
        // Fallback response
        return new User(id, "Default User");
    }
}
```

---

## الخاتمة

هذا الدليل يغطي أهم أسئلة مقابلات Spring Boot. ينصح بممارسة هذه المواضيع عملياً من خلال بناء تطبيقات حقيقية.

### نصائح للمقابلة:
1. افهم الأساسيات جيداً قبل المواضيع المتقدمة
2. مارس كتابة الكود بدون IDE
3. اعرف الفرق بين Spring و Spring Boot
4. كن مستعداً لشرح مشاريعك السابقة
5. اقرأ عن أحدث إصدارات Spring Boot

**حظاً موفقاً في مقابلتك! 🚀**