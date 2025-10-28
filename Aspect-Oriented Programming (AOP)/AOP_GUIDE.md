# 🎯 Aspect-Oriented Programming (AOP) - دليل شامل

## 📋 جدول المحتويات
- [ما هو AOP؟](#ما-هو-aop)
- [المشكلة التي يحلها AOP](#المشكلة-التي-يحلها-aop)
- [المفاهيم الأساسية](#المفاهيم-الأساسية)
- [أنواع Advice](#أنواع-advice)
- [Pointcut Expressions](#pointcut-expressions)
- [أمثلة عملية](#أمثلة-عملية)
- [AOP في مشروعنا](#aop-في-مشروعنا)
- [أفضل الممارسات](#أفضل-الممارسات)

---

## ما هو AOP؟

**Aspect-Oriented Programming (AOP)** هو نمط برمجة يهدف إلى فصل **Cross-cutting Concerns** عن **Business Logic** الأساسي.

### Cross-cutting Concerns:
هي الوظائف التي تتداخل مع أجزاء متعددة من التطبيق مثل:
- 🔐 **Security/Authorization**
- 📝 **Logging**
- 🔄 **Transaction Management**
- ⚡ **Caching**
- 📊 **Performance Monitoring**
- ✅ **Validation**
- 🛡️ **Error Handling**

---

## المشكلة التي يحلها AOP

### ❌ بدون AOP - كود مكرر ومعقد:

```java
@RestController
public class UserController {

    public ResponseEntity<User> getUser(Long id) {
        // 1. Authorization check
        if (!securityService.hasPermission("READ_USER")) {
            return ResponseEntity.status(403).build();
        }

        // 2. Logging
        log.info("Getting user with id: {}", id);

        // 3. Validation
        if (id == null || id <= 0) {
            return ResponseEntity.badRequest().build();
        }

        // 4. Performance monitoring
        long startTime = System.currentTimeMillis();

        try {
            // 5. Business Logic (أخيراً!)
            User user = userService.findById(id);

            // 6. More logging
            log.info("User found: {}", user.getName());

            // 7. Performance logging
            long duration = System.currentTimeMillis() - startTime;
            log.info("Method executed in {} ms", duration);

            return ResponseEntity.ok(user);

        } catch (Exception e) {
            // 8. Error logging
            log.error("Error getting user: {}", e.getMessage());
            return ResponseEntity.status(500).build();
        }
    }

    public ResponseEntity<User> updateUser(Long id, User user) {
        // نفس الكود المكرر مرة أخرى! 😩
        if (!securityService.hasPermission("UPDATE_USER")) {
            return ResponseEntity.status(403).build();
        }

        log.info("Updating user with id: {}", id);
        // ... وهكذا
    }
}
```

### ✅ مع AOP - كود نظيف ومركز:

```java
@RestController
public class UserController {

    @Authorized("READ_USER")        // 🔐 Security
    @LogExecutionTime              // 📊 Performance
    @ValidateParams                // ✅ Validation
    public ResponseEntity<User> getUser(@Valid @Positive Long id) {
        // فقط Business Logic! 🎯
        User user = userService.findById(id);
        return ResponseEntity.ok(user);
    }

    @Authorized("UPDATE_USER")      // 🔐 Security
    @LogExecutionTime              // 📊 Performance
    @Transactional                 // 🔄 Transaction
    public ResponseEntity<User> updateUser(@Valid Long id, @Valid User user) {
        // فقط Business Logic! 🎯
        User updated = userService.update(id, user);
        return ResponseEntity.ok(updated);
    }
}
```

---

## المفاهيم الأساسية

### 1. **Aspect** 🎭
الكلاس الذي يحتوي على Cross-cutting Logic:

```java
@Aspect
@Component
public class SecurityAspect {
    // Authorization logic هنا
}
```

### 2. **Join Point** 📍
النقطة في التطبيق حيث يمكن تطبيق الـ Aspect (عادة method execution):

```java
// هذا method هو Join Point
public void saveUser(User user) { }
```

### 3. **Pointcut** 🎯
Expression يحدد أي Join Points سيتم تطبيق الـ Aspect عليها:

```java
@Pointcut("@annotation(Authorized)")
public void authorizedMethods() {}
```

### 4. **Advice** ⚡
الكود الذي يتم تنفيذه في Join Point:

```java
@Around("authorizedMethods()")
public Object checkAuthorization(ProceedingJoinPoint point) {
    // الكود هنا
}
```

### 5. **Weaving** 🧵
عملية ربط الـ Aspects مع الـ Application code.

---

## أنواع Advice

### 1. **@Before** - قبل تنفيذ الـ Method
```java
@Before("@annotation(LogBefore)")
public void logMethodEntry(JoinPoint point) {
    log.info("🚀 Starting method: {}", point.getSignature().getName());
}
```

### 2. **@After** - بعد تنفيذ الـ Method (success أو failure)
```java
@After("@annotation(LogAfter)")
public void logMethodExit(JoinPoint point) {
    log.info("🏁 Finished method: {}", point.getSignature().getName());
}
```

### 3. **@AfterReturning** - بعد النجاح فقط
```java
@AfterReturning(value = "@annotation(LogSuccess)", returning = "result")
public void logSuccess(JoinPoint point, Object result) {
    log.info("✅ Method {} returned: {}", point.getSignature().getName(), result);
}
```

### 4. **@AfterThrowing** - بعد حدوث Exception فقط
```java
@AfterThrowing(value = "@annotation(LogError)", throwing = "exception")
public void logError(JoinPoint point, Exception exception) {
    log.error("❌ Method {} threw exception: {}", point.getSignature().getName(), exception.getMessage());
}
```

### 5. **@Around** - يحيط بالـ Method (الأقوى والأكثر مرونة)
```java
@Around("@annotation(Authorized)")
public Object checkAuthorization(ProceedingJoinPoint point, Authorized authorized) throws Throwable {
    // Before logic
    String permission = authorized.value();
    if (!hasPermission(permission)) {
        throw new AccessDeniedException("Access denied");
    }

    try {
        // تنفيذ الـ Method الأصلي
        Object result = point.proceed();

        // After logic (في حالة النجاح)
        log.info("✅ Authorized method executed successfully");
        return result;

    } catch (Exception e) {
        // After logic (في حالة الفشل)
        log.error("❌ Authorized method failed: {}", e.getMessage());
        throw e;
    }
}
```

---

## Pointcut Expressions

### 1. **بناءً على Annotation**
```java
@Pointcut("@annotation(org.springframework.web.bind.annotation.PostMapping)")
public void postMappingMethods() {}

@Pointcut("@annotation(Authorized)")
public void authorizedMethods() {}
```

### 2. **بناءً على Package**
```java
@Pointcut("execution(* com.dgcash.tanweel.bankintegration.controller.*.*(..))")
public void controllerMethods() {}
```

### 3. **بناءً على Method Name**
```java
@Pointcut("execution(* save*(..))")
public void saveMethods() {}

@Pointcut("execution(* find*(..))")
public void findMethods() {}
```

### 4. **بناءً على Return Type**
```java
@Pointcut("execution(ResponseEntity *(..))")
public void responseEntityMethods() {}
```

### 5. **تركيب Pointcuts**
```java
@Pointcut("controllerMethods() && authorizedMethods()")
public void authorizedControllerMethods() {}

@Pointcut("saveMethods() || updateMethods()")
public void modifyingMethods() {}
```

---

## أمثلة عملية

### 1. **Security Aspect** 🔐

```java
@Aspect
@Component
@Slf4j
public class SecurityAspect {

    @Around("@annotation(authorized)")
    public Object checkAuthorization(ProceedingJoinPoint point, Authorized authorized) throws Throwable {
        String requiredPermission = authorized.value();

        // Get current user
        DigitalCashUserDto currentUser = getCurrentUser();
        if (currentUser == null) {
            throw new UnauthorizedException("User not authenticated");
        }

        // Check permission
        if (!hasPermission(currentUser, requiredPermission)) {
            log.warn("🚫 User {} denied access to method {} - missing permission: {}",
                     currentUser.getUsername(), point.getSignature().getName(), requiredPermission);
            throw new AccessDeniedException("Insufficient permissions");
        }

        log.info("✅ User {} authorized for method {} with permission: {}",
                 currentUser.getUsername(), point.getSignature().getName(), requiredPermission);

        return point.proceed();
    }

    private boolean hasPermission(DigitalCashUserDto user, String permission) {
        return user.getPermissions().contains(permission);
    }
}
```

### 2. **Logging Aspect** 📝

```java
@Aspect
@Component
@Slf4j
public class LoggingAspect {

    @Around("@annotation(LogExecutionTime)")
    public Object logExecutionTime(ProceedingJoinPoint point) throws Throwable {
        String methodName = point.getSignature().getName();
        String className = point.getSignature().getDeclaringTypeName();

        log.info("🚀 Starting method: {}.{}", className, methodName);
        long startTime = System.currentTimeMillis();

        try {
            Object result = point.proceed();
            long duration = System.currentTimeMillis() - startTime;

            log.info("✅ Method {}.{} completed successfully in {} ms",
                     className, methodName, duration);
            return result;

        } catch (Exception e) {
            long duration = System.currentTimeMillis() - startTime;
            log.error("❌ Method {}.{} failed after {} ms: {}",
                      className, methodName, duration, e.getMessage());
            throw e;
        }
    }

    @Before("@annotation(LogParams)")
    public void logParameters(JoinPoint point) {
        String methodName = point.getSignature().getName();
        Object[] args = point.getArgs();

        log.info("📥 Method {} called with parameters: {}", methodName, Arrays.toString(args));
    }
}
```

### 3. **Validation Aspect** ✅

```java
@Aspect
@Component
@Slf4j
public class ValidationAspect {

    @Before("@annotation(ValidateNotNull)")
    public void validateNotNull(JoinPoint point, ValidateNotNull validateNotNull) {
        Object[] args = point.getArgs();
        String[] paramNames = validateNotNull.params();

        for (int i = 0; i < args.length && i < paramNames.length; i++) {
            if (args[i] == null) {
                throw new IllegalArgumentException("Parameter '" + paramNames[i] + "' cannot be null");
            }
        }

        log.info("✅ Validation passed for method: {}", point.getSignature().getName());
    }
}
```

### 4. **Caching Aspect** ⚡

```java
@Aspect
@Component
@Slf4j
public class CachingAspect {

    private final Map<String, Object> cache = new ConcurrentHashMap<>();

    @Around("@annotation(Cacheable)")
    public Object cache(ProceedingJoinPoint point, Cacheable cacheable) throws Throwable {
        String cacheKey = generateCacheKey(point, cacheable.key());

        // Check cache first
        if (cache.containsKey(cacheKey)) {
            log.info("🎯 Cache hit for key: {}", cacheKey);
            return cache.get(cacheKey);
        }

        // Execute method and cache result
        log.info("💾 Cache miss for key: {} - executing method", cacheKey);
        Object result = point.proceed();
        cache.put(cacheKey, result);

        return result;
    }

    private String generateCacheKey(ProceedingJoinPoint point, String keyPrefix) {
        String methodName = point.getSignature().getName();
        String args = Arrays.toString(point.getArgs());
        return keyPrefix + ":" + methodName + ":" + args.hashCode();
    }
}
```

---

## AOP في مشروعنا

### التكوين الحالي:

```java
// 1. تفعيل AOP في Main Class
@SpringBootApplication
@EnableAspectJAutoProxy  // ← يفعل AOP
public class BankIntegrationApplication {
    // ...
}
```

```java
// 2. استخدام @Authorized في Controller
@RestController
@RequestMapping("/api/v1/leantech")
public class LeantechController {

    @PostMapping("/token")
    @Authorized("CAN_CREATE_APPLICATION")  // ← AOP في العمل
    public ResponseEntity<TokenResponse> getToken() {
        // Business logic فقط
        return ResponseEntity.ok(leantechService.getAccessToken());
    }
}
```

```java
// 3. Security Aspect في dgcash-authorization library
@Aspect
@Component
public class UserAuthenticator {

    @Around("@annotation(com.dgcash.common.authorization.data.dto.Authorized)")
    public Object authenticate(ProceedingJoinPoint point) throws Throwable {
        // يفحص الصلاحيات قبل تنفيذ الـ method
    }
}
```

### Dependencies المطلوبة:

```xml
<!-- في pom.xml -->
<dependency>
    <groupId>com.dgcash.common.core</groupId>
    <artifactId>dgcash-authorization</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>

<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.22.1</version>
</dependency>
```

---

## أفضل الممارسات

### 1. **اجعل الـ Aspects محددة** 🎯
```java
// ❌ عام جداً
@Around("execution(* *(..))")

// ✅ محدد ودقيق
@Around("@annotation(Authorized)")
```

### 2. **استخدم Annotations للوضوح** 🏷️
```java
// ✅ واضح ومفهوم
@Authorized("CAN_CREATE_APPLICATION")
@LogExecutionTime
@Transactional
public void createApplication() { }
```

### 3. **اجعل الـ Advice سريع** ⚡
```java
@Around("@annotation(Authorized)")
public Object checkAuth(ProceedingJoinPoint point, Authorized authorized) throws Throwable {
    // ✅ فحص سريع
    if (!hasPermission(authorized.value())) {
        throw new AccessDeniedException("Access denied");
    }

    // ❌ لا تضع عمليات بطيئة هنا
    // heavyDatabaseOperation();

    return point.proceed();
}
```

### 4. **استخدم التسمية الواضحة** 📛
```java
// ✅ أسماء واضحة
@Authorized("CAN_CREATE_APPLICATION")
@LogExecutionTime
@ValidateNotNull(params = {"userId", "applicationData"})

// ❌ أسماء غامضة
@Auth("PERM1")
@Log
@Valid
```

### 5. **اختبر الـ Aspects** 🧪
```java
@Test
public void testAuthorizationAspect() {
    // Given
    when(securityService.hasPermission("CAN_CREATE_APPLICATION")).thenReturn(false);

    // When & Then
    assertThrows(AccessDeniedException.class, () -> {
        controller.getToken();
    });
}
```

### 6. **تجنب Circular Dependencies** 🔄
```java
// ❌ لا تحقن services في aspects قد تستخدم نفس الـ aspect
@Aspect
public class LoggingAspect {
    @Autowired
    private UserService userService; // قد يسبب circular dependency
}

// ✅ استخدم static utilities أو separate services
@Aspect
public class LoggingAspect {
    private static final Logger log = LoggerFactory.getLogger(LoggingAspect.class);
}
```

---

## الخلاصة 🎊

**AOP يساعدك على:**
- ✅ **فصل Concerns** - Business logic منفصل عن Security/Logging
- ✅ **تقليل التكرار** - كتابة الكود مرة واحدة واستخدامه في أماكن متعددة
- ✅ **تحسين القراءة** - الكود أصبح أنظف وأكثر وضوحاً
- ✅ **سهولة الصيانة** - تغيير الـ authorization logic في مكان واحد
- ✅ **إمكانية الاختبار** - يمكن اختبار Business logic منفصلاً عن Cross-cutting concerns

**AOP = كود أنظف + أقل تكراراً + أسهل صيانة! 🚀**