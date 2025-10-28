# 📝 دليل استخدام Logging Aspect في Spring

## 🚀 الخطوات الكاملة لتطبيق Logging Aspect

### الخطوة 1️⃣: إضافة Dependencies

```xml
<!-- pom.xml for Maven -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

### الخطوة 2️⃣: إنشاء Logging Aspect

```java
package com.example.aspects;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import java.util.Arrays;

@Aspect
@Component
public class LoggingAspect {

    private static final Logger logger = LoggerFactory.getLogger(LoggingAspect.class);

    // ====== طريقة 1: استخدام @Before و @After ======

    /**
     * Log قبل تنفيذ أي method في الـ Service layer
     */
    @Before("execution(* com.example.service.*.*(..))")
    public void logBefore(JoinPoint joinPoint) {
        String methodName = joinPoint.getSignature().getName();
        String className = joinPoint.getTarget().getClass().getSimpleName();
        Object[] args = joinPoint.getArgs();

        logger.info("🚀 Starting {}.{} with arguments: {}",
            className, methodName, Arrays.toString(args));
    }

    /**
     * Log بعد نجاح تنفيذ الـ method
     */
    @AfterReturning(
        pointcut = "execution(* com.example.service.*.*(..))",
        returning = "result"
    )
    public void logAfterReturning(JoinPoint joinPoint, Object result) {
        String methodName = joinPoint.getSignature().getName();
        String className = joinPoint.getTarget().getClass().getSimpleName();

        logger.info("✅ Successfully executed {}.{} - Result: {}",
            className, methodName, result);
    }

    /**
     * Log في حالة حدوث Exception
     */
    @AfterThrowing(
        pointcut = "execution(* com.example.service.*.*(..))",
        throwing = "error"
    )
    public void logAfterThrowing(JoinPoint joinPoint, Throwable error) {
        String methodName = joinPoint.getSignature().getName();
        String className = joinPoint.getTarget().getClass().getSimpleName();

        logger.error("❌ Error in {}.{}: {}",
            className, methodName, error.getMessage(), error);
    }

    // ====== طريقة 2: استخدام @Around (أقوى وأشمل) ======

    /**
     * Log شامل مع قياس الوقت
     */
    @Around("@annotation(com.example.annotations.LogExecution)")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long startTime = System.currentTimeMillis();

        String methodName = joinPoint.getSignature().getName();
        String className = joinPoint.getTarget().getClass().getSimpleName();
        Object[] args = joinPoint.getArgs();

        logger.info("⏱️ Starting {}.{} with args: {}",
            className, methodName, Arrays.toString(args));

        try {
            // تنفيذ الـ method الأصلية
            Object result = joinPoint.proceed();

            long endTime = System.currentTimeMillis();
            long executionTime = endTime - startTime;

            logger.info("✅ {}.{} executed successfully in {} ms. Result: {}",
                className, methodName, executionTime, result);

            // تحذير لو الـ method بطيئة
            if (executionTime > 1000) {
                logger.warn("⚠️ Slow method detected: {}.{} took {} ms",
                    className, methodName, executionTime);
            }

            return result;

        } catch (Exception e) {
            long endTime = System.currentTimeMillis();
            long executionTime = endTime - startTime;

            logger.error("❌ {}.{} failed after {} ms. Error: {}",
                className, methodName, executionTime, e.getMessage(), e);

            throw e;
        }
    }
}
```

### الخطوة 3️⃣: إنشاء Custom Annotation (اختياري)

```java
package com.example.annotations;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogExecution {
    String value() default "";
}
```

### الخطوة 4️⃣: استخدام الـ Logging في Service

```java
package com.example.service;

import com.example.annotations.LogExecution;
import com.example.model.User;
import com.example.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    // الـ method دي هيتم logging ليها تلقائي عشان هي في service package
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }

    // الـ method دي هيتم logging ليها بالتفصيل عشان عليها @LogExecution
    @LogExecution
    public User getUserById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new RuntimeException("User not found"));
    }

    @LogExecution
    public User createUser(String name, String email) {
        User user = new User();
        user.setName(name);
        user.setEmail(email);
        return userRepository.save(user);
    }

    @LogExecution
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
}
```

### الخطوة 5️⃣: النتيجة في الـ Console

```log
2024-01-15 10:30:25.123  INFO  --- [main] LoggingAspect : ⏱️ Starting UserService.createUser with args: [John Doe, john@example.com]
2024-01-15 10:30:25.456  INFO  --- [main] LoggingAspect : ✅ UserService.createUser executed successfully in 333 ms. Result: User{id=1, name='John Doe', email='john@example.com'}

2024-01-15 10:30:30.789  INFO  --- [main] LoggingAspect : ⏱️ Starting UserService.getUserById with args: [999]
2024-01-15 10:30:30.890  ERROR --- [main] LoggingAspect : ❌ UserService.getUserById failed after 101 ms. Error: User not found

2024-01-15 10:30:35.111  INFO  --- [main] LoggingAspect : ⏱️ Starting UserService.getAllUsers with args: []
2024-01-15 10:30:36.222  WARN  --- [main] LoggingAspect : ⚠️ Slow method detected: UserService.getAllUsers took 1111 ms
2024-01-15 10:30:36.223  INFO  --- [main] LoggingAspect : ✅ UserService.getAllUsers executed successfully in 1111 ms. Result: [User{...}, User{...}]
```

## 🎯 أنواع مختلفة من Logging Aspects

### 1️⃣ Controller Logging Aspect

```java
@Aspect
@Component
public class ControllerLoggingAspect {

    private static final Logger logger = LoggerFactory.getLogger(ControllerLoggingAspect.class);

    @Around("@within(org.springframework.web.bind.annotation.RestController)")
    public Object logRestController(ProceedingJoinPoint joinPoint) throws Throwable {
        HttpServletRequest request = ((ServletRequestAttributes)
            RequestContextHolder.currentRequestAttributes()).getRequest();

        String httpMethod = request.getMethod();
        String url = request.getRequestURL().toString();
        String clientIP = request.getRemoteAddr();

        logger.info("📥 {} Request to {} from IP: {}", httpMethod, url, clientIP);

        long startTime = System.currentTimeMillis();

        try {
            Object result = joinPoint.proceed();

            long responseTime = System.currentTimeMillis() - startTime;
            logger.info("📤 Response sent in {} ms", responseTime);

            return result;

        } catch (Exception e) {
            logger.error("❌ Request failed: {}", e.getMessage());
            throw e;
        }
    }
}
```

### 2️⃣ Database Operations Logging

```java
@Aspect
@Component
public class RepositoryLoggingAspect {

    private static final Logger logger = LoggerFactory.getLogger(RepositoryLoggingAspect.class);

    @Before("@within(org.springframework.stereotype.Repository)")
    public void logDatabaseOperation(JoinPoint joinPoint) {
        String operation = joinPoint.getSignature().getName();
        String entity = joinPoint.getTarget().getClass().getSimpleName();

        logger.debug("🗄️ Database operation: {} on {}", operation, entity);
    }

    @AfterReturning(
        pointcut = "@within(org.springframework.stereotype.Repository)",
        returning = "result"
    )
    public void logDatabaseResult(JoinPoint joinPoint, Object result) {
        if (result instanceof List) {
            logger.debug("📊 Returned {} records", ((List<?>) result).size());
        }
    }
}
```

### 3️⃣ Logging مع معلومات User

```java
@Aspect
@Component
public class SecurityLoggingAspect {

    private static final Logger logger = LoggerFactory.getLogger(SecurityLoggingAspect.class);

    @Before("@annotation(com.example.annotations.SecureOperation)")
    public void logSecureOperation(JoinPoint joinPoint) {
        // جيب معلومات الـ user الحالي
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        String username = auth != null ? auth.getName() : "anonymous";

        String operation = joinPoint.getSignature().getName();

        logger.info("🔒 User [{}] is performing secure operation: {}",
            username, operation);
    }
}
```

## 🎨 Logging Levels وإمتى تستخدم كل واحد

```java
@Aspect
@Component
public class SmartLoggingAspect {

    private static final Logger logger = LoggerFactory.getLogger(SmartLoggingAspect.class);

    @Around("execution(* com.example..*(..))")
    public Object smartLog(ProceedingJoinPoint joinPoint) throws Throwable {
        String methodName = joinPoint.getSignature().getName();

        // TRACE - معلومات تفصيلية جداً للـ debugging
        logger.trace("Entering method: {}", methodName);

        // DEBUG - معلومات مفيدة للـ development
        logger.debug("Method arguments: {}", Arrays.toString(joinPoint.getArgs()));

        try {
            Object result = joinPoint.proceed();

            // INFO - معلومات عامة عن سير العمل
            logger.info("Method {} executed successfully", methodName);

            return result;

        } catch (IllegalArgumentException e) {
            // WARN - مشاكل ممكن نتعامل معاها
            logger.warn("Invalid argument in {}: {}", methodName, e.getMessage());
            throw e;

        } catch (Exception e) {
            // ERROR - أخطاء حرجة
            logger.error("Critical error in {}: {}", methodName, e.getMessage(), e);
            throw e;
        }
    }
}
```

## ⚙️ إعدادات الـ Logging في application.properties

```properties
# تحديد مستوى الـ logging
logging.level.root=INFO
logging.level.com.example=DEBUG
logging.level.com.example.aspects=TRACE
logging.level.org.springframework.web=WARN

# تحديد صيغة الـ log
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} - %msg%n
logging.pattern.file=%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n

# حفظ الـ logs في ملف
logging.file.name=application.log
logging.file.path=/var/log/myapp

# حجم الملف والـ rotation
logging.logback.rollingpolicy.max-file-size=10MB
logging.logback.rollingpolicy.max-history=30
```

## 🔧 إعدادات متقدمة باستخدام logback.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <!-- Console Appender بألوان -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>
                %d{HH:mm:ss.SSS} %highlight(%-5level) %cyan(%logger{36}) - %msg%n
            </pattern>
        </encoder>
    </appender>

    <!-- File Appender للـ errors فقط -->
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/errors.log</file>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/errors-%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- Logger للـ Aspects -->
    <logger name="com.example.aspects" level="DEBUG"/>

    <!-- Root Logger -->
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="ERROR_FILE"/>
    </root>

</configuration>
```

## 💡 نصائح مهمة

### ✅ افعل

1. **استخدم المستوى المناسب** من الـ logging (DEBUG للتطوير، INFO للإنتاج)
2. **احمي المعلومات الحساسة** - متسجلش passwords أو tokens
3. **استخدم placeholders** بدل من concatenation للأداء
4. **أضف context مفيد** في الـ logs

### ❌ لا تفعل

```java
// ❌ خطأ - معلومات حساسة
logger.info("User login: username={}, password={}", username, password);

// ✅ صح
logger.info("User login attempt: username={}", username);

// ❌ خطأ - concatenation مكلف
logger.debug("Processing " + items.size() + " items");

// ✅ صح - placeholder أفضل
logger.debug("Processing {} items", items.size());

// ❌ خطأ - logging كتير جداً
for (Item item : items) {
    logger.info("Processing item: {}", item);
}

// ✅ صح - log summary
logger.info("Processing {} items", items.size());
```

## 📊 مثال كامل: تطبيق حقيقي

```java
@SpringBootApplication
@EnableAspectJAutoProxy
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

// Controller
@RestController
@RequestMapping("/api/users")
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        return ResponseEntity.ok(userService.getUserById(id));
    }

    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody UserDTO userDto) {
        return ResponseEntity.ok(userService.createUser(userDto));
    }
}

// Service with logging
@Service
public class UserService {

    @LogExecution
    public User getUserById(Long id) {
        // Business logic
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
    }

    @LogExecution
    public User createUser(UserDTO userDto) {
        // Business logic
        User user = new User(userDto);
        return userRepository.save(user);
    }
}

// The Aspect handles all logging automatically!
```

## 🎉 الخلاصة

مع الـ Logging Aspect:
- **كود نظيف** - مفيش logging code في الـ business logic
- **Centralized** - كل الـ logging في مكان واحد
- **Consistent** - نفس الـ format في كل مكان
- **Configurable** - سهل تغيير مستوى أو طريقة الـ logging
- **Performance** - ممكن تشغيله أو توقفه حسب البيئة

بدل ما تكتب logging في كل method، خلي الـ Aspect يعملها تلقائي! 🚀