# Spring Boot - Bean Scopes & Dependency Injection

## 1. Bean Scopes في Spring Boot

### ما هو Bean Scope؟
Bean Scope يحدد دورة حياة (lifecycle) وإمكانية الوصول (visibility) للـ beans في Spring container. يحدد متى يتم إنشاء instance جديد من الـ bean.

## الأنواع المختلفة لـ Bean Scopes

### 1. Singleton Scope (Default)
- **instance واحد فقط** لكل Spring IoC container
- الـ scope الافتراضي إذا لم تحدد scope
- يتم إنشاء الـ bean عند بدء التطبيق (eager initialization) أو عند أول طلب (lazy initialization)

```java
@Component
@Scope("singleton")  // أو @Scope(ConfigurableBeanFactory.SCOPE_SINGLETON)
public class SingletonBean {
    private static int instanceCount = 0;
    private int instanceId;

    public SingletonBean() {
        instanceId = ++instanceCount;
        System.out.println("Creating SingletonBean instance #" + instanceId);
    }

    public void printInstanceId() {
        System.out.println("Instance ID: " + instanceId);
    }
}

// أو باستخدام @Configuration
@Configuration
public class AppConfig {

    @Bean
    @Scope("singleton")  // Optional - singleton is default
    public DatabaseConnection databaseConnection() {
        return new DatabaseConnection();
    }
}

// الاستخدام
@RestController
public class TestController {
    @Autowired
    private SingletonBean bean1;

    @Autowired
    private SingletonBean bean2;

    @GetMapping("/test-singleton")
    public String testSingleton() {
        bean1.printInstanceId();  // Instance ID: 1
        bean2.printInstanceId();  // Instance ID: 1 (نفس الـ instance)
        return "Check console";
    }
}
```

### 2. Prototype Scope
- **instance جديد** كل مرة يُطلب فيها الـ bean
- Spring لا يدير complete lifecycle (لا يستدعي destroy methods)

```java
@Component
@Scope("prototype")  // أو @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class PrototypeBean {
    private static int instanceCount = 0;
    private int instanceId;

    public PrototypeBean() {
        instanceId = ++instanceCount;
        System.out.println("Creating PrototypeBean instance #" + instanceId);
    }

    public void printInstanceId() {
        System.out.println("Instance ID: " + instanceId);
    }
}

@RestController
public class PrototypeController {
    @Autowired
    private ApplicationContext context;

    @GetMapping("/test-prototype")
    public String testPrototype() {
        PrototypeBean bean1 = context.getBean(PrototypeBean.class);
        PrototypeBean bean2 = context.getBean(PrototypeBean.class);

        bean1.printInstanceId();  // Instance ID: 1
        bean2.printInstanceId();  // Instance ID: 2 (instance مختلف)

        System.out.println("Same instance? " + (bean1 == bean2));  // false
        return "Check console";
    }
}
```

### 3. Request Scope (Web Applications Only)
- **instance جديد لكل HTTP request**
- الـ bean يعيش طوال فترة الـ HTTP request

```java
@Component
@Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RequestScopedBean {
    private String requestId;
    private long creationTime;

    public RequestScopedBean() {
        this.requestId = UUID.randomUUID().toString();
        this.creationTime = System.currentTimeMillis();
        System.out.println("Creating RequestScopedBean: " + requestId);
    }

    public String getRequestInfo() {
        return "Request ID: " + requestId + ", Created at: " + creationTime;
    }
}

@RestController
public class RequestController {
    @Autowired
    private RequestScopedBean requestBean;

    @GetMapping("/test-request")
    public Map<String, String> testRequest() {
        Map<String, String> response = new HashMap<>();
        response.put("first_call", requestBean.getRequestInfo());

        // حتى لو استدعيناها مرة أخرى في نفس الـ request
        response.put("second_call", requestBean.getRequestInfo());  // نفس الـ instance

        return response;
        // عند request جديد، سيتم إنشاء instance جديد
    }
}
```

### 4. Session Scope (Web Applications Only)
- **instance واحد لكل HTTP session**
- مفيد لتخزين معلومات المستخدم طوال الـ session

```java
@Component
@Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class SessionScopedBean {
    private String sessionId;
    private String username;
    private List<String> cartItems = new ArrayList<>();

    public SessionScopedBean() {
        this.sessionId = UUID.randomUUID().toString();
        System.out.println("Creating SessionScopedBean: " + sessionId);
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public void addToCart(String item) {
        cartItems.add(item);
    }

    public List<String> getCartItems() {
        return cartItems;
    }

    public String getSessionInfo() {
        return "Session ID: " + sessionId + ", User: " + username;
    }
}

@RestController
public class SessionController {
    @Autowired
    private SessionScopedBean sessionBean;

    @PostMapping("/login")
    public String login(@RequestParam String username) {
        sessionBean.setUsername(username);
        return "Logged in as: " + username;
    }

    @PostMapping("/add-to-cart")
    public String addToCart(@RequestParam String item) {
        sessionBean.addToCart(item);
        return "Added " + item + " to cart. Total items: " + sessionBean.getCartItems().size();
    }

    @GetMapping("/cart")
    public List<String> getCart() {
        return sessionBean.getCartItems();  // نفس الـ cart طوال الـ session
    }
}
```

### 5. Application Scope (Web Applications Only)
- **instance واحد لكل ServletContext**
- مشترك بين جميع sessions و requests في نفس التطبيق

```java
@Component
@Scope(value = WebApplicationContext.SCOPE_APPLICATION, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class ApplicationScopedBean {
    private int totalVisitors = 0;
    private Map<String, Integer> pageViews = new HashMap<>();

    public synchronized void incrementVisitors() {
        totalVisitors++;
    }

    public synchronized void recordPageView(String page) {
        pageViews.merge(page, 1, Integer::sum);
    }

    public int getTotalVisitors() {
        return totalVisitors;
    }

    public Map<String, Integer> getPageViews() {
        return new HashMap<>(pageViews);
    }
}

@RestController
public class ApplicationController {
    @Autowired
    private ApplicationScopedBean appBean;

    @GetMapping("/visit")
    public String visit() {
        appBean.incrementVisitors();
        appBean.recordPageView("/visit");
        return "Total visitors: " + appBean.getTotalVisitors();
    }

    @GetMapping("/stats")
    public Map<String, Object> getStats() {
        Map<String, Object> stats = new HashMap<>();
        stats.put("totalVisitors", appBean.getTotalVisitors());
        stats.put("pageViews", appBean.getPageViews());
        return stats;
    }
}
```

### 6. WebSocket Scope (Spring WebSocket)
- **instance لكل WebSocket session**

```java
@Component
@Scope(scopeName = "websocket", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class WebSocketScopedBean {
    private String socketSessionId;
    private List<String> messages = new ArrayList<>();

    public WebSocketScopedBean() {
        this.socketSessionId = UUID.randomUUID().toString();
    }

    public void addMessage(String message) {
        messages.add(message);
    }

    public List<String> getMessages() {
        return messages;
    }
}
```

### Custom Scope
يمكنك إنشاء scope خاص بك:

```java
public class CustomScope implements Scope {
    private final Map<String, Object> scopedObjects = new ConcurrentHashMap<>();
    private final Map<String, Runnable> destructionCallbacks = new ConcurrentHashMap<>();

    @Override
    public Object get(String name, ObjectFactory<?> objectFactory) {
        Object scopedObject = scopedObjects.get(name);
        if (scopedObject == null) {
            scopedObject = objectFactory.getObject();
            scopedObjects.put(name, scopedObject);
        }
        return scopedObject;
    }

    @Override
    public Object remove(String name) {
        scopedObjects.remove(name);
        Runnable callback = destructionCallbacks.remove(name);
        if (callback != null) {
            callback.run();
        }
        return null;
    }

    @Override
    public void registerDestructionCallback(String name, Runnable callback) {
        destructionCallbacks.put(name, callback);
    }

    @Override
    public Object resolveContextualObject(String key) {
        return null;
    }

    @Override
    public String getConversationId() {
        return "custom";
    }

    public void clear() {
        scopedObjects.clear();
        destructionCallbacks.values().forEach(Runnable::run);
        destructionCallbacks.clear();
    }
}

// تسجيل الـ Custom Scope
@Configuration
public class CustomScopeConfig {
    @Bean
    public static CustomScopeConfigurer customScopeConfigurer() {
        CustomScopeConfigurer configurer = new CustomScopeConfigurer();
        configurer.addScope("custom", new CustomScope());
        return configurer;
    }
}

// استخدام الـ Custom Scope
@Component
@Scope("custom")
public class CustomScopedBean {
    // Implementation
}
```

## مشاكل وحلول Scope

### 1. Injecting Prototype into Singleton
```java
@Component
public class SingletonBean {
    // مشكلة: سيتم inject نفس الـ prototype instance دائماً
    // @Autowired
    // private PrototypeBean prototypeBean;

    // حل 1: استخدام ApplicationContext
    @Autowired
    private ApplicationContext context;

    public void usePrototype() {
        PrototypeBean prototype = context.getBean(PrototypeBean.class);
        prototype.doSomething();
    }

    // حل 2: استخدام @Lookup
    @Lookup
    public PrototypeBean getPrototypeBean() {
        return null;  // Spring will override this
    }

    // حل 3: استخدام ObjectFactory
    @Autowired
    private ObjectFactory<PrototypeBean> prototypeBeanFactory;

    public void usePrototypeWithFactory() {
        PrototypeBean prototype = prototypeBeanFactory.getObject();
        prototype.doSomething();
    }

    // حل 4: استخدام Provider (JSR-330)
    @Autowired
    private Provider<PrototypeBean> prototypeBeanProvider;

    public void usePrototypeWithProvider() {
        PrototypeBean prototype = prototypeBeanProvider.get();
        prototype.doSomething();
    }
}
```

## 2. Dependency Injection (DI)

### ما هو Dependency Injection؟
Dependency Injection هو design pattern حيث objects تستقبل dependencies من مصدر خارجي بدلاً من إنشائها بنفسها. في Spring، الـ IoC container يدير ويحقن الـ dependencies.

### فوائد Dependency Injection
1. **Loose Coupling**: تقليل الاعتماد المباشر بين الـ classes
2. **Testability**: سهولة كتابة unit tests باستخدام mocks
3. **Maintainability**: سهولة تغيير implementations
4. **Reusability**: إمكانية إعادة استخدام الـ components

## أنواع Dependency Injection

### 1. Constructor Injection (Recommended)
أفضل طريقة - dependencies تُحقن من خلال constructor

```java
@Service
public class UserService {
    private final UserRepository userRepository;
    private final EmailService emailService;

    // Constructor Injection - أفضل طريقة
    @Autowired  // Optional من Spring 4.3+ إذا كان constructor واحد
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }

    public User createUser(User user) {
        User savedUser = userRepository.save(user);
        emailService.sendWelcomeEmail(savedUser);
        return savedUser;
    }
}

// مع Lombok
@Service
@RequiredArgsConstructor  // Lombok creates constructor
public class OrderService {
    private final OrderRepository orderRepository;
    private final PaymentService paymentService;
    private final NotificationService notificationService;

    public Order processOrder(Order order) {
        // Implementation
    }
}

// Configuration class example
@Configuration
public class ServiceConfig {

    @Bean
    public UserService userService(UserRepository repo, EmailService email) {
        return new UserService(repo, email);
    }
}
```

**مميزات Constructor Injection:**
- Immutability (final fields)
- Required dependencies واضحة
- سهولة testing
- يمنع circular dependencies
- Thread-safe

### 2. Setter Injection
Dependencies تُحقن من خلال setter methods

```java
@Service
public class ProductService {
    private ProductRepository productRepository;
    private InventoryService inventoryService;

    // Setter Injection
    @Autowired
    public void setProductRepository(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    @Autowired
    public void setInventoryService(InventoryService inventoryService) {
        this.inventoryService = inventoryService;
    }

    // أو setter واحد لعدة dependencies
    @Autowired
    public void setDependencies(ProductRepository repo, InventoryService inventory) {
        this.productRepository = repo;
        this.inventoryService = inventory;
    }

    public Product addProduct(Product product) {
        inventoryService.checkAvailability(product);
        return productRepository.save(product);
    }
}

// Optional dependencies
@Component
public class LoggingService {
    private Logger customLogger;

    @Autowired(required = false)  // Optional dependency
    public void setCustomLogger(Logger customLogger) {
        this.customLogger = customLogger;
    }

    public void log(String message) {
        if (customLogger != null) {
            customLogger.log(message);
        } else {
            System.out.println(message);  // Default behavior
        }
    }
}
```

**متى نستخدم Setter Injection:**
- Optional dependencies
- عندما نحتاج تغيير dependency بعد الإنشاء
- Legacy code compatibility

### 3. Field Injection
Dependencies تُحقن مباشرة في fields (غير محبذ)

```java
@Service
public class CustomerService {
    // Field Injection - غير محبذ
    @Autowired
    private CustomerRepository customerRepository;

    @Autowired
    private EmailService emailService;

    @Autowired
    @Qualifier("premiumDiscountService")  // تحديد implementation معين
    private DiscountService discountService;

    public Customer registerCustomer(Customer customer) {
        Customer saved = customerRepository.save(customer);
        emailService.sendRegistrationEmail(saved);
        return saved;
    }
}
```

**عيوب Field Injection:**
- صعوبة في testing (تحتاج reflection)
- Hidden dependencies
- لا يمكن استخدام final fields
- يشجع على God classes
- يخفي circular dependencies

### 4. Method Injection
حقن dependencies في methods عادية (نادر الاستخدام)

```java
@Component
public class NotificationService {
    private MessageFormatter formatter;
    private MessageSender sender;

    // Method Injection
    @Autowired
    public void configure(MessageFormatter formatter, MessageSender sender) {
        this.formatter = formatter;
        this.sender = sender;
        // يمكن إضافة initialization logic
        initialize();
    }

    private void initialize() {
        // Setup code
    }

    public void notify(String message) {
        String formatted = formatter.format(message);
        sender.send(formatted);
    }
}
```

## أمثلة متقدمة لـ DI

### 1. Conditional Injection
```java
@Service
public class PaymentService {
    private final PaymentProcessor processor;

    @Autowired
    public PaymentService(
        @Qualifier("paypal") PaymentProcessor paypalProcessor,
        @Qualifier("stripe") PaymentProcessor stripeProcessor,
        @Value("${payment.processor.default}") String defaultProcessor) {

        this.processor = "paypal".equals(defaultProcessor) ?
                        paypalProcessor : stripeProcessor;
    }
}

// أو باستخدام @Conditional
@Component
@ConditionalOnProperty(name = "payment.processor", havingValue = "paypal")
public class PayPalProcessor implements PaymentProcessor {
    // Implementation
}
```

### 2. List/Map Injection
```java
@Service
public class ValidationService {
    // Inject all implementations of Validator
    private final List<Validator> validators;
    private final Map<String, Validator> validatorMap;

    @Autowired
    public ValidationService(List<Validator> validators,
                           Map<String, Validator> validatorMap) {
        this.validators = validators;
        this.validatorMap = validatorMap;
    }

    public void validateAll(Object obj) {
        validators.forEach(v -> v.validate(obj));
    }

    public void validateWithType(String type, Object obj) {
        Validator validator = validatorMap.get(type);
        if (validator != null) {
            validator.validate(obj);
        }
    }
}
```

### 3. Circular Dependencies
```java
// مشكلة: Circular Dependency
@Component
public class ServiceA {
    @Autowired
    private ServiceB serviceB;  // A depends on B
}

@Component
public class ServiceB {
    @Autowired
    private ServiceA serviceA;  // B depends on A - Circular!
}

// حل 1: استخدام @Lazy
@Component
public class ServiceA {
    private final ServiceB serviceB;

    @Autowired
    public ServiceA(@Lazy ServiceB serviceB) {
        this.serviceB = serviceB;
    }
}

// حل 2: Setter Injection
@Component
public class ServiceA {
    private ServiceB serviceB;

    @Autowired
    public void setServiceB(ServiceB serviceB) {
        this.serviceB = serviceB;
    }
}

// حل 3 (الأفضل): إعادة تصميم لتجنب circular dependency
@Component
public class ServiceC {
    // Shared logic that both A and B need
}

@Component
public class ServiceA {
    @Autowired
    private ServiceC serviceC;
}

@Component
public class ServiceB {
    @Autowired
    private ServiceC serviceC;
}
```

### 4. Primary و Qualifier
```java
// Multiple implementations
interface DataSource {
    String getData();
}

@Component
@Primary  // Default choice
public class MySQLDataSource implements DataSource {
    public String getData() {
        return "MySQL Data";
    }
}

@Component
@Qualifier("mongo")
public class MongoDataSource implements DataSource {
    public String getData() {
        return "MongoDB Data";
    }
}

@Service
public class DataService {
    @Autowired
    private DataSource defaultDataSource;  // MySQLDataSource (Primary)

    @Autowired
    @Qualifier("mongo")
    private DataSource mongoDataSource;  // MongoDataSource

    public void processData() {
        System.out.println(defaultDataSource.getData());
        System.out.println(mongoDataSource.getData());
    }
}
```

### 5. Configuration Properties Injection
```java
@ConfigurationProperties(prefix = "app.database")
@Component
public class DatabaseProperties {
    private String url;
    private String username;
    private String password;
    private int maxConnections;

    // Getters and Setters
}

@Service
public class DatabaseService {
    private final DatabaseProperties dbProperties;

    @Autowired
    public DatabaseService(DatabaseProperties dbProperties) {
        this.dbProperties = dbProperties;
    }

    public void connect() {
        // Use dbProperties.getUrl(), etc.
    }
}
```

## Best Practices

### 1. Prefer Constructor Injection
```java
@Service
@RequiredArgsConstructor
public class BestPracticeService {
    // Final fields - immutable
    private final UserRepository userRepository;
    private final EmailService emailService;
    private final SecurityService securityService;

    // No need for @Autowired with single constructor
    // Dependencies are clear and required
    // Easy to test with mocks
}
```

### 2. Use @Configuration Classes
```java
@Configuration
public class ApplicationConfig {

    @Bean
    @Profile("production")
    public DataSource productionDataSource() {
        // Production configuration
    }

    @Bean
    @Profile("development")
    public DataSource developmentDataSource() {
        // Development configuration
    }

    @Bean
    @ConditionalOnProperty(name = "cache.enabled", havingValue = "true")
    public CacheManager cacheManager() {
        return new RedisCacheManager();
    }
}
```

### 3. Avoid Field Injection in Production Code
```java
// Bad - Field Injection
@Service
public class BadService {
    @Autowired
    private Repository repository;  // Hard to test, hidden dependency
}

// Good - Constructor Injection
@Service
public class GoodService {
    private final Repository repository;

    public GoodService(Repository repository) {
        this.repository = repository;  // Clear dependency, easy to test
    }
}
```

## Testing with DI

```java
@SpringBootTest
class UserServiceTest {

    @MockBean
    private UserRepository userRepository;

    @MockBean
    private EmailService emailService;

    @Autowired
    private UserService userService;

    @Test
    void testCreateUser() {
        // Given
        User user = new User("John", "john@example.com");
        when(userRepository.save(any(User.class))).thenReturn(user);

        // When
        User result = userService.createUser(user);

        // Then
        assertNotNull(result);
        verify(emailService).sendWelcomeEmail(user);
    }
}

// Unit test without Spring
class UserServiceUnitTest {

    @Mock
    private UserRepository userRepository;

    @Mock
    private EmailService emailService;

    private UserService userService;

    @BeforeEach
    void setUp() {
        MockitoAnnotations.openMocks(this);
        // Constructor injection makes testing easy
        userService = new UserService(userRepository, emailService);
    }

    @Test
    void testCreateUser() {
        // Test implementation
    }
}
```

## خلاصة

### Bean Scopes Summary
| Scope | Description | Use Case |
|-------|-------------|----------|
| **Singleton** | Instance واحد per container | Stateless services, repositories |
| **Prototype** | Instance جديد كل مرة | Stateful objects, temporary objects |
| **Request** | Instance per HTTP request | Request-specific data |
| **Session** | Instance per HTTP session | User session data |
| **Application** | Instance per ServletContext | Application-wide cache |

### DI Types Summary
| Type | Pros | Cons | When to Use |
|------|------|------|-------------|
| **Constructor** | Immutable, clear deps, testable | Verbose with many deps | Always (preferred) |
| **Setter** | Optional deps, flexible | Mutable, partial injection | Optional dependencies |
| **Field** | Concise | Hard to test, hidden deps | Avoid in production |
| **Method** | Initialization logic | Rarely needed | Special initialization |