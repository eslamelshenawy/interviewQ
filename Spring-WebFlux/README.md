# Spring WebFlux - دليل شامل

## المحتويات
1. [ما هو Spring WebFlux](#ما-هو-spring-webflux)
2. [Reactive Programming](#reactive-programming)
3. [Project Reactor](#project-reactor)
4. [WebFlux vs Spring MVC](#webflux-vs-spring-mvc)
5. [المكونات الأساسية](#المكونات-الأساسية)
6. [Reactive Controllers](#reactive-controllers)
7. [Functional Endpoints](#functional-endpoints)
8. [Reactive Data Access](#reactive-data-access)
9. [WebClient](#webclient)
10. [أمثلة عملية](#أمثلة-عملية)
11. [Best Practices](#best-practices)
12. [أسئلة المقابلات](#أسئلة-المقابلات)

---

## ما هو Spring WebFlux

**Spring WebFlux** هو إطار عمل Reactive Web Framework من Spring 5، مبني على Reactive Streams و Project Reactor. يوفر نموذج برمجة non-blocking و asynchronous.

### المزايا الرئيسية
- **Non-blocking**: لا يحجز Threads أثناء I/O operations
- **Asynchronous**: العمليات تتم بشكل غير متزامن
- **Reactive Streams**: دعم كامل للـ Reactive Streams API
- **High Scalability**: يدعم آلاف الطلبات المتزامنة
- **Backpressure**: التحكم في تدفق البيانات
- **Functional Programming**: دعم Functional endpoints
- **Event Loop**: مبني على Event Loop model (مثل Node.js)

### متى تستخدم WebFlux؟
- ✅ تطبيقات تحتاج High Concurrency
- ✅ Microservices مع Communication كثيف
- ✅ Real-time Applications
- ✅ Streaming Data
- ✅ عمليات I/O كثيرة (Database, External APIs)
- ✅ عندما تحتاج تقليل استهلاك Resources

### متى تستخدم Spring MVC؟
- ✅ تطبيقات CRUD بسيطة
- ✅ فريق ليس لديه خبرة في Reactive
- ✅ عندما تستخدم Blocking Libraries
- ✅ تطبيقات موجودة بالفعل على MVC

---

## Reactive Programming

### ما هو Reactive Programming؟

**Reactive Programming** هو نموذج برمجة يركز على:
- **Data Streams**: كل شيء عبارة عن Stream
- **Asynchronous**: العمليات غير متزامنة
- **Non-blocking**: لا تحجز Threads
- **Event-driven**: يستجيب للأحداث
- **Backpressure**: التحكم في معدل البيانات

### Reactive Streams API

أربعة Interfaces رئيسية:

```java
// 1. Publisher: ينشر البيانات
public interface Publisher<T> {
    void subscribe(Subscriber<? super T> s);
}

// 2. Subscriber: يستقبل البيانات
public interface Subscriber<T> {
    void onSubscribe(Subscription s);
    void onNext(T t);
    void onError(Throwable t);
    void onComplete();
}

// 3. Subscription: يربط Publisher و Subscriber
public interface Subscription {
    void request(long n);  // Backpressure
    void cancel();
}

// 4. Processor: Publisher و Subscriber معاً
public interface Processor<T, R> extends Subscriber<T>, Publisher<R> {
}
```

### Reactive Manifesto

المبادئ الأربعة:
1. **Responsive**: الاستجابة السريعة
2. **Resilient**: التعامل مع الأخطاء
3. **Elastic**: القابلية للتوسع
4. **Message Driven**: معتمد على الرسائل

---

## Project Reactor

**Project Reactor** هو Reactive Library أساس Spring WebFlux.

### Mono vs Flux

#### Mono<T>
Publisher ينشر **0 أو 1** عنصر.

```java
// إنشاء Mono
Mono<String> mono = Mono.just("Hello");
Mono<String> emptyMono = Mono.empty();
Mono<String> errorMono = Mono.error(new RuntimeException("Error"));

// من Supplier
Mono<String> monoFromSupplier = Mono.fromSupplier(() -> "Hello");

// من Callable
Mono<String> monoFromCallable = Mono.fromCallable(() -> {
    // عملية معقدة
    return "Result";
});
```

#### Flux<T>
Publisher ينشر **0 إلى N** عنصر.

```java
// إنشاء Flux
Flux<String> flux = Flux.just("A", "B", "C");
Flux<Integer> fluxRange = Flux.range(1, 10);
Flux<String> emptyFlux = Flux.empty();

// من Collection
List<String> list = Arrays.asList("A", "B", "C");
Flux<String> fluxFromList = Flux.fromIterable(list);

// من Array
Flux<String> fluxFromArray = Flux.fromArray(new String[]{"A", "B", "C"});

// من Stream
Flux<String> fluxFromStream = Flux.fromStream(Stream.of("A", "B", "C"));
```

### Operators

#### Transformation Operators

```java
// map: تحويل كل عنصر
Flux<Integer> flux = Flux.just(1, 2, 3)
    .map(n -> n * 2);  // [2, 4, 6]

// flatMap: تحويل إلى Publisher آخر
Flux<String> flux = Flux.just("A", "B")
    .flatMap(s -> Flux.just(s + "1", s + "2"));
    // ["A1", "A2", "B1", "B2"]

// flatMapSequential: نفس flatMap لكن يحافظ على الترتيب
Flux<String> flux = Flux.just("A", "B")
    .flatMapSequential(s -> getDataAsync(s));

// concatMap: flatMap متسلسل
Flux<String> flux = Flux.just("A", "B")
    .concatMap(s -> Flux.just(s + "1", s + "2"));

// switchMap: يلغي السابق عند وصول جديد
Flux<String> flux = Flux.just("A", "B")
    .switchMap(s -> getDataAsync(s));
```

#### Filtering Operators

```java
// filter: تصفية العناصر
Flux<Integer> flux = Flux.range(1, 10)
    .filter(n -> n % 2 == 0);  // [2, 4, 6, 8, 10]

// take: أخذ أول N عنصر
Flux<Integer> flux = Flux.range(1, 10)
    .take(5);  // [1, 2, 3, 4, 5]

// skip: تخطي أول N عنصر
Flux<Integer> flux = Flux.range(1, 10)
    .skip(5);  // [6, 7, 8, 9, 10]

// distinct: عناصر فريدة فقط
Flux<Integer> flux = Flux.just(1, 2, 2, 3, 3, 3)
    .distinct();  // [1, 2, 3]

// distinctUntilChanged: فريدة متتالية
Flux<Integer> flux = Flux.just(1, 1, 2, 2, 3)
    .distinctUntilChanged();  // [1, 2, 3]
```

#### Combining Operators

```java
// merge: دمج عدة Publishers
Flux<String> flux1 = Flux.just("A", "B");
Flux<String> flux2 = Flux.just("C", "D");
Flux<String> merged = Flux.merge(flux1, flux2);
// ["A", "B", "C", "D"] (غير مضمون الترتيب)

// concat: دمج متسلسل
Flux<String> concatenated = Flux.concat(flux1, flux2);
// ["A", "B", "C", "D"] (ترتيب مضمون)

// zip: دمج بالتزامن
Flux<String> flux1 = Flux.just("A", "B", "C");
Flux<Integer> flux2 = Flux.just(1, 2, 3);
Flux<String> zipped = Flux.zip(flux1, flux2, (s, i) -> s + i);
// ["A1", "B2", "C3"]

// combineLatest: دمج آخر قيمة من كل Publisher
Flux<String> combined = Flux.combineLatest(
    flux1, flux2,
    (s, i) -> s + i
);
```

#### Error Handling Operators

```java
// onErrorReturn: إرجاع قيمة افتراضية عند الخطأ
Flux<String> flux = Flux.just("A", "B")
    .concatWith(Mono.error(new RuntimeException("Error")))
    .onErrorReturn("DefaultValue");

// onErrorResume: استبدال بـ Publisher آخر
Flux<String> flux = Flux.just("A", "B")
    .concatWith(Mono.error(new RuntimeException("Error")))
    .onErrorResume(e -> Flux.just("Fallback1", "Fallback2"));

// onErrorContinue: استمرار رغم الأخطاء
Flux<Integer> flux = Flux.just(1, 2, 0, 4)
    .map(n -> 10 / n)
    .onErrorContinue((e, obj) -> {
        System.out.println("Error with " + obj + ": " + e.getMessage());
    });

// retry: إعادة المحاولة
Flux<String> flux = getDataFromAPI()
    .retry(3);  // إعادة 3 مرات

// retryWhen: إعادة المحاولة بشروط
Flux<String> flux = getDataFromAPI()
    .retryWhen(Retry.backoff(3, Duration.ofSeconds(1)));
```

#### Timing Operators

```java
// delay: تأخير العناصر
Flux<Integer> flux = Flux.range(1, 5)
    .delayElements(Duration.ofSeconds(1));

// timeout: رمي خطأ بعد وقت محدد
Flux<String> flux = getDataFromAPI()
    .timeout(Duration.ofSeconds(5));

// interval: إصدار قيم بفاصل زمني
Flux<Long> flux = Flux.interval(Duration.ofSeconds(1));
// 0, 1, 2, 3, ... (كل ثانية)
```

#### Utility Operators

```java
// doOnNext: عمل شيء لكل عنصر
Flux<Integer> flux = Flux.range(1, 5)
    .doOnNext(n -> System.out.println("Processing: " + n));

// doOnError: عمل شيء عند الخطأ
Flux<Integer> flux = Flux.range(1, 5)
    .doOnError(e -> log.error("Error occurred", e));

// doOnComplete: عمل شيء عند الانتهاء
Flux<Integer> flux = Flux.range(1, 5)
    .doOnComplete(() -> System.out.println("Completed"));

// log: تسجيل جميع الأحداث
Flux<Integer> flux = Flux.range(1, 5)
    .log();
```

### Subscribing

```java
// subscribe بدون parameters
flux.subscribe();

// subscribe مع Consumer
flux.subscribe(
    value -> System.out.println("Received: " + value)
);

// subscribe مع Consumer و Error Handler
flux.subscribe(
    value -> System.out.println("Received: " + value),
    error -> System.err.println("Error: " + error)
);

// subscribe مع Consumer، Error Handler، و Complete Handler
flux.subscribe(
    value -> System.out.println("Received: " + value),
    error -> System.err.println("Error: " + error),
    () -> System.out.println("Completed")
);

// subscribe مع Subscription Consumer (Backpressure)
flux.subscribe(
    value -> System.out.println("Received: " + value),
    error -> System.err.println("Error: " + error),
    () -> System.out.println("Completed"),
    subscription -> subscription.request(5)  // طلب 5 عناصر فقط
);
```

### Blocking (للاختبار فقط)

```java
// block: انتظار النتيجة (Mono فقط)
String result = Mono.just("Hello")
    .block();

// blockFirst: أول عنصر (Flux)
Integer first = Flux.range(1, 10)
    .blockFirst();

// blockLast: آخر عنصر (Flux)
Integer last = Flux.range(1, 10)
    .blockLast();

// toIterable: تحويل إلى Iterable
Iterable<Integer> iterable = Flux.range(1, 10)
    .toIterable();

// toStream: تحويل إلى Stream
Stream<Integer> stream = Flux.range(1, 10)
    .toStream();
```

---

## WebFlux vs Spring MVC

| الميزة | Spring WebFlux | Spring MVC |
|--------|----------------|------------|
| **Programming Model** | Reactive (Asynchronous) | Imperative (Synchronous) |
| **Threading Model** | Event Loop (قليل من Threads) | Thread per Request |
| **Scalability** | عالي جداً (آلاف الطلبات) | محدود بعدد Threads |
| **Blocking Operations** | غير مدعوم (يجب استخدام Reactive) | مدعوم |
| **Database** | R2DBC, Reactive MongoDB | JPA, JDBC |
| **Server** | Netty, Undertow, Tomcat | Tomcat, Jetty |
| **Learning Curve** | أعلى | أقل |
| **Use Cases** | High Concurrency, Streaming | Traditional Web Apps |
| **Return Types** | Mono, Flux | Object, List, ResponseEntity |
| **Annotations** | @RestController, @GetMapping | @RestController, @GetMapping |

### مثال مقارنة

**Spring MVC (Blocking):**
```java
@RestController
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        // يحجز Thread حتى يرجع من Database
        return userService.findById(id);
    }

    @GetMapping("/users")
    public List<User> getAllUsers() {
        // يحجز Thread
        return userService.findAll();
    }
}
```

**Spring WebFlux (Non-blocking):**
```java
@RestController
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping("/users/{id}")
    public Mono<User> getUser(@PathVariable Long id) {
        // لا يحجز Thread، يرجع فوراً
        return userService.findById(id);
    }

    @GetMapping("/users")
    public Flux<User> getAllUsers() {
        // لا يحجز Thread
        return userService.findAll();
    }
}
```

---

## المكونات الأساسية

### 1. Dependencies

```xml
<dependencies>
    <!-- Spring WebFlux -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>

    <!-- Reactive MongoDB -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-mongodb-reactive</artifactId>
    </dependency>

    <!-- R2DBC (Reactive SQL) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-r2dbc</artifactId>
    </dependency>

    <!-- R2DBC PostgreSQL Driver -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>r2dbc-postgresql</artifactId>
    </dependency>

    <!-- Reactor Test -->
    <dependency>
        <groupId>io.projectreactor</groupId>
        <artifactId>reactor-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### 2. Application Configuration

```yaml
# application.yml
server:
  port: 8080

spring:
  # Reactive MongoDB
  data:
    mongodb:
      uri: mongodb://localhost:27017/mydb

  # R2DBC PostgreSQL
  r2dbc:
    url: r2dbc:postgresql://localhost:5432/mydb
    username: user
    password: password

  # WebFlux
  webflux:
    base-path: /api
```

### 3. Server Configuration

```java
@Configuration
public class WebFluxConfig implements WebFluxConfigurer {

    // CORS Configuration
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
            .allowedOrigins("*")
            .allowedMethods("GET", "POST", "PUT", "DELETE")
            .allowedHeaders("*");
    }

    // Custom JSON configuration
    @Override
    public void configureHttpMessageCodecs(ServerCodecConfigurer configurer) {
        configurer.defaultCodecs().maxInMemorySize(16 * 1024 * 1024);
    }
}
```

---

## Reactive Controllers

### Basic Controller

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @Autowired
    private UserService userService;

    // GET /api/users/{id}
    @GetMapping("/{id}")
    public Mono<User> getUserById(@PathVariable String id) {
        return userService.findById(id);
    }

    // GET /api/users
    @GetMapping
    public Flux<User> getAllUsers() {
        return userService.findAll();
    }

    // POST /api/users
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Mono<User> createUser(@RequestBody User user) {
        return userService.save(user);
    }

    // PUT /api/users/{id}
    @PutMapping("/{id}")
    public Mono<User> updateUser(
        @PathVariable String id,
        @RequestBody User user
    ) {
        return userService.update(id, user);
    }

    // DELETE /api/users/{id}
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public Mono<Void> deleteUser(@PathVariable String id) {
        return userService.deleteById(id);
    }
}
```

### ResponseEntity

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping("/{id}")
    public Mono<ResponseEntity<User>> getUserById(@PathVariable String id) {
        return userService.findById(id)
            .map(user -> ResponseEntity.ok(user))
            .defaultIfEmpty(ResponseEntity.notFound().build());
    }

    @PostMapping
    public Mono<ResponseEntity<User>> createUser(@RequestBody User user) {
        return userService.save(user)
            .map(savedUser -> ResponseEntity
                .created(URI.create("/api/users/" + savedUser.getId()))
                .body(savedUser));
    }

    @PutMapping("/{id}")
    public Mono<ResponseEntity<User>> updateUser(
        @PathVariable String id,
        @RequestBody User user
    ) {
        return userService.update(id, user)
            .map(ResponseEntity::ok)
            .defaultIfEmpty(ResponseEntity.notFound().build());
    }
}
```

### Request Parameters & Headers

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    // Query Parameters
    @GetMapping("/search")
    public Flux<User> searchUsers(
        @RequestParam String name,
        @RequestParam(required = false, defaultValue = "0") int page,
        @RequestParam(required = false, defaultValue = "10") int size
    ) {
        return userService.searchByName(name, page, size);
    }

    // Headers
    @GetMapping("/{id}")
    public Mono<User> getUserById(
        @PathVariable String id,
        @RequestHeader("Authorization") String token
    ) {
        return userService.findById(id, token);
    }

    // Multiple Path Variables
    @GetMapping("/{userId}/posts/{postId}")
    public Mono<Post> getPostByUser(
        @PathVariable String userId,
        @PathVariable String postId
    ) {
        return userService.getPost(userId, postId);
    }
}
```

### ServerRequest & ServerResponse

```java
@RestController
public class UserController {

    @GetMapping("/users/stream")
    public Flux<ServerSentEvent<User>> streamUsers() {
        return userService.findAll()
            .map(user -> ServerSentEvent.<User>builder()
                .id(user.getId())
                .event("user-created")
                .data(user)
                .build())
            .delayElements(Duration.ofSeconds(1));
    }
}
```

---

## Functional Endpoints

### Router Configuration

```java
@Configuration
public class UserRouter {

    @Bean
    public RouterFunction<ServerResponse> userRoutes(UserHandler handler) {
        return RouterFunctions
            .route(GET("/api/users"), handler::getAllUsers)
            .andRoute(GET("/api/users/{id}"), handler::getUserById)
            .andRoute(POST("/api/users"), handler::createUser)
            .andRoute(PUT("/api/users/{id}"), handler::updateUser)
            .andRoute(DELETE("/api/users/{id}"), handler::deleteUser);
    }

    // Nested Routes
    @Bean
    public RouterFunction<ServerResponse> nestedRoutes(UserHandler handler) {
        return nest(path("/api/users"),
            route(GET(""), handler::getAllUsers)
                .andRoute(GET("/{id}"), handler::getUserById)
                .andRoute(POST(""), handler::createUser)
                .andRoute(PUT("/{id}"), handler::updateUser)
                .andRoute(DELETE("/{id}"), handler::deleteUser)
        );
    }

    // With Predicates
    @Bean
    public RouterFunction<ServerResponse> predicateRoutes(UserHandler handler) {
        return route()
            .GET("/api/users",
                 accept(MediaType.APPLICATION_JSON),
                 handler::getAllUsers)
            .GET("/api/users/{id}",
                 accept(MediaType.APPLICATION_JSON),
                 handler::getUserById)
            .POST("/api/users",
                  contentType(MediaType.APPLICATION_JSON),
                  handler::createUser)
            .build();
    }
}
```

### Handler Functions

```java
@Component
public class UserHandler {

    @Autowired
    private UserService userService;

    public Mono<ServerResponse> getAllUsers(ServerRequest request) {
        Flux<User> users = userService.findAll();
        return ServerResponse.ok()
            .contentType(MediaType.APPLICATION_JSON)
            .body(users, User.class);
    }

    public Mono<ServerResponse> getUserById(ServerRequest request) {
        String id = request.pathVariable("id");

        return userService.findById(id)
            .flatMap(user -> ServerResponse.ok()
                .contentType(MediaType.APPLICATION_JSON)
                .bodyValue(user))
            .switchIfEmpty(ServerResponse.notFound().build());
    }

    public Mono<ServerResponse> createUser(ServerRequest request) {
        Mono<User> userMono = request.bodyToMono(User.class);

        return userMono
            .flatMap(userService::save)
            .flatMap(user -> ServerResponse
                .created(URI.create("/api/users/" + user.getId()))
                .contentType(MediaType.APPLICATION_JSON)
                .bodyValue(user));
    }

    public Mono<ServerResponse> updateUser(ServerRequest request) {
        String id = request.pathVariable("id");
        Mono<User> userMono = request.bodyToMono(User.class);

        return userMono
            .flatMap(user -> userService.update(id, user))
            .flatMap(user -> ServerResponse.ok()
                .contentType(MediaType.APPLICATION_JSON)
                .bodyValue(user))
            .switchIfEmpty(ServerResponse.notFound().build());
    }

    public Mono<ServerResponse> deleteUser(ServerRequest request) {
        String id = request.pathVariable("id");

        return userService.deleteById(id)
            .then(ServerResponse.noContent().build());
    }

    // Query Parameters
    public Mono<ServerResponse> searchUsers(ServerRequest request) {
        String name = request.queryParam("name").orElse("");
        int page = request.queryParam("page")
            .map(Integer::parseInt)
            .orElse(0);
        int size = request.queryParam("size")
            .map(Integer::parseInt)
            .orElse(10);

        Flux<User> users = userService.searchByName(name, page, size);

        return ServerResponse.ok()
            .contentType(MediaType.APPLICATION_JSON)
            .body(users, User.class);
    }

    // Headers
    public Mono<ServerResponse> getUserWithAuth(ServerRequest request) {
        String id = request.pathVariable("id");
        String token = request.headers()
            .firstHeader("Authorization");

        return userService.findById(id, token)
            .flatMap(user -> ServerResponse.ok().bodyValue(user))
            .switchIfEmpty(ServerResponse.status(HttpStatus.UNAUTHORIZED).build());
    }
}
```

### Error Handling

```java
@Configuration
public class ErrorConfig {

    @Bean
    public RouterFunction<ServerResponse> errorRoutes() {
        return route()
            .onError(IllegalArgumentException.class, (e, request) ->
                ServerResponse.badRequest()
                    .bodyValue(Map.of("error", e.getMessage())))
            .onError(ResourceNotFoundException.class, (e, request) ->
                ServerResponse.notFound().build())
            .onError(Exception.class, (e, request) ->
                ServerResponse.status(HttpStatus.INTERNAL_SERVER_ERROR)
                    .bodyValue(Map.of("error", "Internal Server Error")))
            .build();
    }
}
```

---

## Reactive Data Access

### Reactive MongoDB

```java
// Entity
@Document(collection = "users")
public class User {
    @Id
    private String id;
    private String name;
    private String email;
    private Integer age;

    // getters, setters, constructors
}

// Repository
public interface UserRepository extends ReactiveMongoRepository<User, String> {

    Flux<User> findByName(String name);

    Flux<User> findByAgeGreaterThan(Integer age);

    Mono<User> findByEmail(String email);

    @Query("{ 'name': { $regex: ?0, $options: 'i' } }")
    Flux<User> searchByName(String name);

    @Query("{ 'age': { $gte: ?0, $lte: ?1 } }")
    Flux<User> findByAgeBetween(Integer minAge, Integer maxAge);
}

// Service
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    public Mono<User> findById(String id) {
        return userRepository.findById(id);
    }

    public Flux<User> findAll() {
        return userRepository.findAll();
    }

    public Mono<User> save(User user) {
        return userRepository.save(user);
    }

    public Mono<User> update(String id, User user) {
        return userRepository.findById(id)
            .flatMap(existingUser -> {
                existingUser.setName(user.getName());
                existingUser.setEmail(user.getEmail());
                existingUser.setAge(user.getAge());
                return userRepository.save(existingUser);
            });
    }

    public Mono<Void> deleteById(String id) {
        return userRepository.deleteById(id);
    }

    public Flux<User> searchByName(String name) {
        return userRepository.searchByName(name);
    }
}
```

### R2DBC (Reactive SQL)

```java
// Entity
@Table("users")
public class User {
    @Id
    private Long id;
    private String name;
    private String email;
    private Integer age;

    // getters, setters
}

// Repository
public interface UserRepository extends ReactiveCrudRepository<User, Long> {

    @Query("SELECT * FROM users WHERE name = :name")
    Flux<User> findByName(String name);

    @Query("SELECT * FROM users WHERE age > :age")
    Flux<User> findByAgeGreaterThan(Integer age);

    @Query("SELECT * FROM users WHERE email = :email")
    Mono<User> findByEmail(String email);

    @Modifying
    @Query("UPDATE users SET name = :name WHERE id = :id")
    Mono<Integer> updateName(Long id, String name);
}

// Configuration
@Configuration
@EnableR2dbcRepositories
public class R2DBCConfig extends AbstractR2dbcConfiguration {

    @Bean
    @Override
    public ConnectionFactory connectionFactory() {
        return ConnectionFactories.get(
            ConnectionFactoryOptions.builder()
                .option(DRIVER, "postgresql")
                .option(HOST, "localhost")
                .option(PORT, 5432)
                .option(USER, "user")
                .option(PASSWORD, "password")
                .option(DATABASE, "mydb")
                .build()
        );
    }
}
```

### Custom Reactive Repository

```java
@Repository
public class CustomUserRepository {

    @Autowired
    private ReactiveMongoTemplate mongoTemplate;

    public Flux<User> findByCustomQuery(String name, Integer minAge) {
        Query query = new Query();
        query.addCriteria(
            Criteria.where("name").regex(name, "i")
                .and("age").gte(minAge)
        );

        return mongoTemplate.find(query, User.class);
    }

    public Mono<Long> countByAge(Integer age) {
        Query query = Query.query(Criteria.where("age").is(age));
        return mongoTemplate.count(query, User.class);
    }

    public Flux<User> findWithPagination(int page, int size) {
        Query query = new Query()
            .skip(page * size)
            .limit(size);

        return mongoTemplate.find(query, User.class);
    }
}
```

---

## WebClient

**WebClient** هو HTTP Client reactive من Spring WebFlux.

### Basic Configuration

```java
@Configuration
public class WebClientConfig {

    @Bean
    public WebClient webClient() {
        return WebClient.builder()
            .baseUrl("https://api.example.com")
            .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
            .defaultHeader(HttpHeaders.USER_AGENT, "Spring WebFlux")
            .build();
    }

    // With Timeout
    @Bean
    public WebClient webClientWithTimeout() {
        HttpClient httpClient = HttpClient.create()
            .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 5000)
            .responseTimeout(Duration.ofSeconds(5));

        return WebClient.builder()
            .baseUrl("https://api.example.com")
            .clientConnector(new ReactorClientHttpConnector(httpClient))
            .build();
    }

    // With Custom Exchange Strategies
    @Bean
    public WebClient webClientWithCustomCodecs() {
        ExchangeStrategies strategies = ExchangeStrategies.builder()
            .codecs(configurer -> configurer
                .defaultCodecs()
                .maxInMemorySize(16 * 1024 * 1024))  // 16 MB
            .build();

        return WebClient.builder()
            .baseUrl("https://api.example.com")
            .exchangeStrategies(strategies)
            .build();
    }
}
```

### GET Requests

```java
@Service
public class UserClient {

    @Autowired
    private WebClient webClient;

    // Simple GET
    public Mono<User> getUserById(String id) {
        return webClient.get()
            .uri("/users/{id}", id)
            .retrieve()
            .bodyToMono(User.class);
    }

    // GET with Query Parameters
    public Flux<User> searchUsers(String name, int page, int size) {
        return webClient.get()
            .uri(uriBuilder -> uriBuilder
                .path("/users/search")
                .queryParam("name", name)
                .queryParam("page", page)
                .queryParam("size", size)
                .build())
            .retrieve()
            .bodyToFlux(User.class);
    }

    // GET with Headers
    public Mono<User> getUserWithAuth(String id, String token) {
        return webClient.get()
            .uri("/users/{id}", id)
            .header(HttpHeaders.AUTHORIZATION, "Bearer " + token)
            .retrieve()
            .bodyToMono(User.class);
    }

    // GET with Error Handling
    public Mono<User> getUserWithErrorHandling(String id) {
        return webClient.get()
            .uri("/users/{id}", id)
            .retrieve()
            .onStatus(HttpStatus::is4xxClientError, response ->
                Mono.error(new ResourceNotFoundException("User not found")))
            .onStatus(HttpStatus::is5xxServerError, response ->
                Mono.error(new RuntimeException("Server error")))
            .bodyToMono(User.class);
    }

    // GET List
    public Flux<User> getAllUsers() {
        return webClient.get()
            .uri("/users")
            .retrieve()
            .bodyToFlux(User.class);
    }
}
```

### POST Requests

```java
@Service
public class UserClient {

    @Autowired
    private WebClient webClient;

    // POST with Body
    public Mono<User> createUser(User user) {
        return webClient.post()
            .uri("/users")
            .contentType(MediaType.APPLICATION_JSON)
            .bodyValue(user)
            .retrieve()
            .bodyToMono(User.class);
    }

    // POST with Mono Body
    public Mono<User> createUserMono(Mono<User> userMono) {
        return webClient.post()
            .uri("/users")
            .contentType(MediaType.APPLICATION_JSON)
            .body(userMono, User.class)
            .retrieve()
            .bodyToMono(User.class);
    }

    // POST with Form Data
    public Mono<String> submitForm(String name, String email) {
        MultiValueMap<String, String> formData = new LinkedMultiValueMap<>();
        formData.add("name", name);
        formData.add("email", email);

        return webClient.post()
            .uri("/submit")
            .contentType(MediaType.APPLICATION_FORM_URLENCODED)
            .bodyValue(formData)
            .retrieve()
            .bodyToMono(String.class);
    }
}
```

### PUT & DELETE Requests

```java
@Service
public class UserClient {

    @Autowired
    private WebClient webClient;

    // PUT
    public Mono<User> updateUser(String id, User user) {
        return webClient.put()
            .uri("/users/{id}", id)
            .contentType(MediaType.APPLICATION_JSON)
            .bodyValue(user)
            .retrieve()
            .bodyToMono(User.class);
    }

    // DELETE
    public Mono<Void> deleteUser(String id) {
        return webClient.delete()
            .uri("/users/{id}", id)
            .retrieve()
            .bodyToMono(Void.class);
    }

    // PATCH
    public Mono<User> patchUser(String id, Map<String, Object> updates) {
        return webClient.patch()
            .uri("/users/{id}", id)
            .contentType(MediaType.APPLICATION_JSON)
            .bodyValue(updates)
            .retrieve()
            .bodyToMono(User.class);
    }
}
```

### Advanced Features

```java
@Service
public class UserClient {

    @Autowired
    private WebClient webClient;

    // Exchange (Full Response Access)
    public Mono<ResponseEntity<User>> getUserWithResponse(String id) {
        return webClient.get()
            .uri("/users/{id}", id)
            .exchangeToMono(response -> {
                if (response.statusCode().is2xxSuccessful()) {
                    return response.toEntity(User.class);
                } else {
                    return Mono.error(new RuntimeException("Error"));
                }
            });
    }

    // Retry
    public Mono<User> getUserWithRetry(String id) {
        return webClient.get()
            .uri("/users/{id}", id)
            .retrieve()
            .bodyToMono(User.class)
            .retryWhen(Retry.backoff(3, Duration.ofSeconds(1)));
    }

    // Timeout
    public Mono<User> getUserWithTimeout(String id) {
        return webClient.get()
            .uri("/users/{id}", id)
            .retrieve()
            .bodyToMono(User.class)
            .timeout(Duration.ofSeconds(5))
            .onErrorResume(TimeoutException.class, e ->
                Mono.error(new RuntimeException("Request timeout")));
    }

    // Parallel Requests
    public Mono<Map<String, User>> getUsersParallel(List<String> ids) {
        List<Mono<Tuple2<String, User>>> requests = ids.stream()
            .map(id -> webClient.get()
                .uri("/users/{id}", id)
                .retrieve()
                .bodyToMono(User.class)
                .map(user -> Tuples.of(id, user)))
            .collect(Collectors.toList());

        return Flux.merge(requests)
            .collectMap(Tuple2::getT1, Tuple2::getT2);
    }
}
```

---

## أمثلة عملية

### مثال 1: REST API كامل

```java
// Entity
@Document(collection = "posts")
public class Post {
    @Id
    private String id;
    private String title;
    private String content;
    private String authorId;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;

    // getters, setters, constructors
}

// Repository
public interface PostRepository extends ReactiveMongoRepository<Post, String> {
    Flux<Post> findByAuthorId(String authorId);
    Flux<Post> findByTitleContainingIgnoreCase(String title);
}

// Service
@Service
public class PostService {

    @Autowired
    private PostRepository postRepository;

    @Autowired
    private UserService userService;

    public Flux<Post> findAll() {
        return postRepository.findAll();
    }

    public Mono<Post> findById(String id) {
        return postRepository.findById(id);
    }

    public Mono<Post> create(Post post) {
        post.setCreatedAt(LocalDateTime.now());
        post.setUpdatedAt(LocalDateTime.now());
        return postRepository.save(post);
    }

    public Mono<Post> update(String id, Post post) {
        return postRepository.findById(id)
            .flatMap(existingPost -> {
                existingPost.setTitle(post.getTitle());
                existingPost.setContent(post.getContent());
                existingPost.setUpdatedAt(LocalDateTime.now());
                return postRepository.save(existingPost);
            });
    }

    public Mono<Void> deleteById(String id) {
        return postRepository.deleteById(id);
    }

    public Flux<Post> findByAuthorId(String authorId) {
        return postRepository.findByAuthorId(authorId);
    }

    // Advanced: Get Posts with Author Info
    public Flux<PostDTO> getPostsWithAuthor() {
        return postRepository.findAll()
            .flatMap(post -> userService.findById(post.getAuthorId())
                .map(author -> new PostDTO(
                    post.getId(),
                    post.getTitle(),
                    post.getContent(),
                    author.getName(),
                    post.getCreatedAt()
                ))
            );
    }
}

// Controller
@RestController
@RequestMapping("/api/posts")
public class PostController {

    @Autowired
    private PostService postService;

    @GetMapping
    public Flux<Post> getAllPosts() {
        return postService.findAll();
    }

    @GetMapping("/{id}")
    public Mono<ResponseEntity<Post>> getPostById(@PathVariable String id) {
        return postService.findById(id)
            .map(ResponseEntity::ok)
            .defaultIfEmpty(ResponseEntity.notFound().build());
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Mono<Post> createPost(@Valid @RequestBody Post post) {
        return postService.create(post);
    }

    @PutMapping("/{id}")
    public Mono<ResponseEntity<Post>> updatePost(
        @PathVariable String id,
        @Valid @RequestBody Post post
    ) {
        return postService.update(id, post)
            .map(ResponseEntity::ok)
            .defaultIfEmpty(ResponseEntity.notFound().build());
    }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public Mono<Void> deletePost(@PathVariable String id) {
        return postService.deleteById(id);
    }

    @GetMapping("/author/{authorId}")
    public Flux<Post> getPostsByAuthor(@PathVariable String authorId) {
        return postService.findByAuthorId(authorId);
    }

    @GetMapping("/with-author")
    public Flux<PostDTO> getPostsWithAuthor() {
        return postService.getPostsWithAuthor();
    }
}
```

### مثال 2: Server-Sent Events (SSE)

```java
@RestController
@RequestMapping("/api/stream")
public class StreamController {

    @Autowired
    private PostService postService;

    // Stream جميع Posts
    @GetMapping(value = "/posts", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<Post> streamPosts() {
        return postService.findAll()
            .delayElements(Duration.ofSeconds(1));
    }

    // Stream مع Server-Sent Events
    @GetMapping(value = "/posts-sse", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<ServerSentEvent<Post>> streamPostsSSE() {
        return postService.findAll()
            .map(post -> ServerSentEvent.<Post>builder()
                .id(post.getId())
                .event("post-created")
                .data(post)
                .retry(Duration.ofSeconds(3))
                .build())
            .delayElements(Duration.ofSeconds(1));
    }

    // Infinite Stream
    @GetMapping(value = "/time", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<String> streamTime() {
        return Flux.interval(Duration.ofSeconds(1))
            .map(sequence -> LocalDateTime.now().toString());
    }
}
```

### مثال 3: WebSocket

```java
// WebSocket Handler
@Component
public class ChatWebSocketHandler implements WebSocketHandler {

    private final FluxProcessor<ChatMessage, ChatMessage> processor;
    private final FluxSink<ChatMessage> sink;

    public ChatWebSocketHandler() {
        this.processor = DirectProcessor.<ChatMessage>create().serialize();
        this.sink = processor.sink();
    }

    @Override
    public Mono<Void> handle(WebSocketSession session) {
        // Receive messages from client
        Mono<Void> input = session.receive()
            .map(WebSocketMessage::getPayloadAsText)
            .map(text -> {
                try {
                    return new ObjectMapper().readValue(text, ChatMessage.class);
                } catch (Exception e) {
                    throw new RuntimeException(e);
                }
            })
            .doOnNext(sink::next)
            .then();

        // Send messages to client
        Flux<WebSocketMessage> output = processor
            .map(message -> {
                try {
                    return new ObjectMapper().writeValueAsString(message);
                } catch (Exception e) {
                    throw new RuntimeException(e);
                }
            })
            .map(session::textMessage);

        return session.send(output).and(input);
    }
}

// WebSocket Configuration
@Configuration
public class WebSocketConfig {

    @Autowired
    private ChatWebSocketHandler chatHandler;

    @Bean
    public HandlerMapping webSocketMapping() {
        Map<String, WebSocketHandler> map = new HashMap<>();
        map.put("/ws/chat", chatHandler);

        SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
        mapping.setUrlMap(map);
        mapping.setOrder(-1);
        return mapping;
    }

    @Bean
    public WebSocketHandlerAdapter handlerAdapter() {
        return new WebSocketHandlerAdapter();
    }
}
```

### مثال 4: Reactive Validation

```java
// Custom Validator
@Component
public class UserValidator {

    @Autowired
    private UserRepository userRepository;

    public Mono<Void> validate(User user) {
        return Mono.defer(() -> {
            List<String> errors = new ArrayList<>();

            if (user.getName() == null || user.getName().isEmpty()) {
                errors.add("Name is required");
            }

            if (user.getEmail() == null || !user.getEmail().contains("@")) {
                errors.add("Invalid email");
            }

            if (!errors.isEmpty()) {
                return Mono.error(new ValidationException(errors));
            }

            // Check if email already exists
            return userRepository.findByEmail(user.getEmail())
                .flatMap(existingUser ->
                    Mono.<Void>error(new ValidationException("Email already exists")))
                .then();
        });
    }
}

// Service with Validation
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private UserValidator userValidator;

    public Mono<User> create(User user) {
        return userValidator.validate(user)
            .then(userRepository.save(user));
    }
}

// Error Response
@Data
@AllArgsConstructor
public class ErrorResponse {
    private int status;
    private String message;
    private List<String> errors;
    private LocalDateTime timestamp;
}

// Global Error Handler
@RestControllerAdvice
public class GlobalErrorHandler {

    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidationException(ValidationException ex) {
        ErrorResponse error = new ErrorResponse(
            400,
            "Validation Failed",
            ex.getErrors(),
            LocalDateTime.now()
        );
        return ResponseEntity.badRequest().body(error);
    }

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFoundException(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            404,
            ex.getMessage(),
            Collections.emptyList(),
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
}
```

---

## Best Practices

### 1. Never Block في Reactive Chain

```java
// ❌ خطأ - Blocking
public Mono<User> getUserBlocking(String id) {
    return Mono.fromSupplier(() -> {
        User user = blockingRepository.findById(id);  // BLOCKING!
        return user;
    });
}

// ✅ صحيح - Non-blocking
public Mono<User> getUserNonBlocking(String id) {
    return reactiveRepository.findById(id);
}

// ✅ إذا كنت مضطراً للـ Blocking
public Mono<User> getUserWithBlocking(String id) {
    return Mono.fromCallable(() -> blockingRepository.findById(id))
        .subscribeOn(Schedulers.boundedElastic());  // Thread منفصل
}
```

### 2. استخدام Schedulers بحكمة

```java
// Schedulers أنواع:
// - Schedulers.immediate(): نفس Thread (افتراضي)
// - Schedulers.single(): Single thread للجميع
// - Schedulers.parallel(): Thread pool للـ CPU-bound
// - Schedulers.boundedElastic(): Thread pool للـ I/O blocking

// مثال
public Flux<User> processUsers() {
    return userRepository.findAll()
        .publishOn(Schedulers.parallel())  // معالجة على threads منفصلة
        .map(this::heavyComputation)
        .subscribeOn(Schedulers.boundedElastic());  // اشتراك على thread منفصل
}
```

### 3. Error Handling

```java
// ✅ استخدم onErrorResume للـ Fallback
public Mono<User> getUserWithFallback(String id) {
    return userRepository.findById(id)
        .onErrorResume(e -> {
            log.error("Error getting user", e);
            return Mono.just(getDefaultUser());
        });
}

// ✅ استخدم retry للإعادة
public Mono<User> getUserWithRetry(String id) {
    return userRepository.findById(id)
        .retryWhen(Retry.backoff(3, Duration.ofSeconds(1))
            .filter(throwable -> throwable instanceof TimeoutException));
}

// ✅ استخدم doOnError للـ Logging
public Mono<User> getUserWithLogging(String id) {
    return userRepository.findById(id)
        .doOnError(e -> log.error("Error getting user {}", id, e));
}
```

### 4. Backpressure

```java
// ✅ استخدم limitRate للتحكم
public Flux<User> getAllUsersWithBackpressure() {
    return userRepository.findAll()
        .limitRate(100);  // طلب 100 عنصر في كل مرة
}

// ✅ استخدم buffer للتجميع
public Flux<List<User>> getAllUsersBuffered() {
    return userRepository.findAll()
        .buffer(100);  // تجميع 100 عنصر
}

// ✅ استخدم window للتقسيم
public Flux<Flux<User>> getAllUsersWindowed() {
    return userRepository.findAll()
        .window(Duration.ofSeconds(1));  // نافذة كل ثانية
}
```

### 5. Testing

```java
@SpringBootTest
public class UserServiceTest {

    @Autowired
    private UserService userService;

    @Test
    public void testGetUser() {
        StepVerifier.create(userService.findById("1"))
            .expectNextMatches(user -> user.getName().equals("Ahmed"))
            .verifyComplete();
    }

    @Test
    public void testGetAllUsers() {
        StepVerifier.create(userService.findAll())
            .expectNextCount(10)
            .verifyComplete();
    }

    @Test
    public void testError() {
        StepVerifier.create(userService.findById("invalid"))
            .expectError(ResourceNotFoundException.class)
            .verify();
    }

    @Test
    public void testWithVirtualTime() {
        StepVerifier.withVirtualTime(() ->
            Flux.interval(Duration.ofSeconds(1))
                .take(3))
            .expectSubscription()
            .expectNoEvent(Duration.ofSeconds(1))
            .expectNext(0L)
            .thenAwait(Duration.ofSeconds(1))
            .expectNext(1L)
            .thenAwait(Duration.ofSeconds(1))
            .expectNext(2L)
            .verifyComplete();
    }
}
```

---

## أسئلة المقابلات

### أسئلة أساسية

**1. ما هو Spring WebFlux ولماذا نستخدمه؟**
- **الإجابة**: Spring WebFlux هو Reactive Web Framework يوفر:
  - Non-blocking I/O
  - Asynchronous programming
  - High scalability مع قليل من Threads
  - مبني على Project Reactor و Reactive Streams

**2. ما الفرق بين Mono و Flux؟**
- **الإجابة**:
  - **Mono**: Publisher ينشر 0 أو 1 عنصر
  - **Flux**: Publisher ينشر 0 إلى N عنصر

**3. ما الفرق بين WebFlux و Spring MVC؟**
- **الإجابة**:
  - **WebFlux**: Reactive، Non-blocking، Event Loop
  - **MVC**: Imperative، Blocking، Thread per Request

**4. ما هو Backpressure؟**
- **الإجابة**: Backpressure هو آلية للتحكم في معدل البيانات:
  - Subscriber يطلب عدد محدد من العناصر
  - Publisher يرسل فقط ما تم طلبه
  - يمنع Memory Overflow

**5. ما هي Reactive Streams؟**
- **الإجابة**: Reactive Streams هي Specification تحدد:
  - Publisher, Subscriber, Subscription, Processor
  - Asynchronous stream processing
  - Non-blocking backpressure

### أسئلة متقدمة

**6. كيف تعمل Schedulers في Reactor؟**
- **الإجابة**: Schedulers تحدد Thread/Thread Pool للتنفيذ:
  - `immediate()`: نفس Thread
  - `single()`: Single thread
  - `parallel()`: CPU-bound tasks
  - `boundedElastic()`: I/O blocking tasks

**7. ما الفرق بين map و flatMap؟**
- **الإجابة**:
  - **map**: تحويل 1-to-1 (Synchronous)
  - **flatMap**: تحويل 1-to-N مع دعم Asynchronous Publishers

**8. كيف تتعامل مع Blocking Code في WebFlux؟**
- **الإجابة**:
  - استخدام `subscribeOn(Schedulers.boundedElastic())`
  - عزل Blocking code في Thread منفصل
  - تجنب Blocking في Reactive chain قدر الإمكان

**9. ما الفرق بين publishOn و subscribeOn؟**
- **الإجابة**:
  - **publishOn**: يغير Scheduler للعمليات **بعده**
  - **subscribeOn**: يغير Scheduler **لكامل** السلسلة

**10. كيف تختبر Reactive Code؟**
- **الإجابة**: استخدام StepVerifier:
```java
StepVerifier.create(flux)
    .expectNext("A", "B", "C")
    .verifyComplete();
```

**11. ما هو Cold vs Hot Publisher؟**
- **الإجابة**:
  - **Cold**: يبدأ من البداية لكل Subscriber (مثل HTTP request)
  - **Hot**: يشارك نفس Stream (مثل Mouse clicks)

**12. كيف تنفذ Error Handling في WebFlux؟**
- **الإجابة**:
  - `onErrorReturn`: قيمة افتراضية
  - `onErrorResume`: Publisher بديل
  - `onErrorContinue`: استمرار رغم الأخطاء
  - `retry`: إعادة المحاولة

**13. ما هو WebClient وكيف يختلف عن RestTemplate؟**
- **الإجابة**:
  - **WebClient**: Reactive، Non-blocking، يدعم Streaming
  - **RestTemplate**: Blocking، Synchronous (deprecated)

**14. كيف تنفذ Pagination في WebFlux؟**
- **الإجابة**:
```java
public Flux<User> getUsers(int page, int size) {
    return repository.findAll()
        .skip(page * size)
        .take(size);
}
```

**15. ما هو Context في Reactor؟**
- **الإجابة**: Context هو Thread-local storage للـ Reactive chain:
  - لتمرير بيانات بين Operators
  - مفيد للـ Authentication, Tracing

### أسئلة Best Practices

**16. متى يجب استخدام WebFlux بدلاً من MVC؟**
- **الإجابة**: استخدم WebFlux عندما:
  - تحتاج High Concurrency
  - Microservices مع Communication كثيف
  - Real-time/Streaming applications
  - عمليات I/O كثيرة

**17. ما هي مشاكل استخدام Blocking Code في WebFlux؟**
- **الإجابة**:
  - يحجز Event Loop Threads
  - يقلل Performance
  - قد يسبب Thread Pool Saturation
  - يفقد فائدة Non-blocking

**18. كيف تحسن أداء WebFlux Applications؟**
- **الإجابة**:
  - استخدام Reactive Database Drivers (R2DBC)
  - تجنب Blocking operations
  - استخدام Caching
  - Backpressure handling
  - Connection pooling

**19. ما هي التحديات في Reactive Programming؟**
- **الإجابة**:
  - Learning curve عالي
  - Debugging صعب
  - Stack traces معقدة
  - يحتاج Reactive Libraries بالكامل
  - Testing معقد

**20. كيف تتعامل مع Transactions في WebFlux؟**
- **الإجابة**:
  - استخدام `@Transactional` مع Reactive repositories
  - استخدام `TransactionalOperator`
  - يجب استخدام Reactive Database Drivers

---

## الخلاصة

Spring WebFlux هو إطار قوي للتطبيقات Reactive:
- ✅ Non-blocking و Asynchronous
- ✅ High Scalability
- ✅ مبني على Project Reactor
- ✅ دعم Functional Programming
- ✅ مناسب للـ Microservices
- ✅ Real-time و Streaming

**متى تستخدم WebFlux:**
- High Concurrency applications
- Microservices architecture
- Real-time/Streaming data
- عمليات I/O كثيفة
- عندما تحتاج تقليل Resources

**متى تستخدم Spring MVC:**
- CRUD applications بسيطة
- فريق بدون خبرة Reactive
- Blocking Libraries موجودة
- تطبيقات Legacy

**الأدوات الأساسية:**
- **Project Reactor**: Reactive library
- **R2DBC**: Reactive SQL driver
- **WebClient**: Reactive HTTP client
- **Reactor Test**: Testing utilities
- **Spring Data Reactive**: Reactive repositories