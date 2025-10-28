# 🎯 شرح مصطلحات AOP: Advice, Pointcut, Target Object

## 🎬 تخيل السيناريو ده

تخيل إن عندك مطعم 🍕 وعايز تراقب كل العمليات اللي بتحصل فيه:
- **Target Object** = المطعم (المكان اللي هتراقبه)
- **Pointcut** = الأوقات اللي هتراقب فيها (لما حد يطلب، لما الطلب يجهز، لما يتم التوصيل)
- **Advice** = إيه اللي هتعمله في الأوقات دي (تسجيل، حساب الوقت، التأكد من الجودة)

---

## 1️⃣ Target Object 🎯

### التعريف البسيط:
**الـ Target Object هو الكلاس أو الـ Object اللي انت عايز تضيف عليه سلوك إضافي**

### مثال توضيحي:

```java
// ❌ قبل استخدام AOP - الـ Target Object فيه كود مختلط
@Service
public class OrderService {  // <-- ده الـ Target Object

    public Order createOrder(String item, int quantity) {
        // كود logging (مش من وظيفة الـ service)
        System.out.println("Creating order for: " + item);

        // الكود الأساسي (ده الشغل الحقيقي)
        Order order = new Order(item, quantity);
        orderRepository.save(order);

        // كود logging تاني
        System.out.println("Order created successfully");

        return order;
    }
}
```

```java
// ✅ بعد استخدام AOP - الـ Target Object نظيف
@Service
public class OrderService {  // <-- ده برضه الـ Target Object

    public Order createOrder(String item, int quantity) {
        // بس الكود الأساسي!
        Order order = new Order(item, quantity);
        return orderRepository.save(order);
    }
}

// الـ Aspect هيضيف الـ logging على الـ Target Object ده
@Aspect
@Component
public class LoggingAspect {
    // هنا هنضيف الـ logging على OrderService
}
```

**الخلاصة:** الـ Target Object هو الكلاس العادي بتاعك اللي الـ Aspect هيشتغل عليه

---

## 2️⃣ Pointcut 📍

### التعريف البسيط:
**الـ Pointcut هو "الفلتر" أو "الشرط" اللي بيحدد فين بالظبط الـ Aspect هيشتغل**

### تشبيه سهل:
تخيل إن الـ Pointcut زي **كاميرات المراقبة** في المطعم:
- ممكن تحطها عند **الباب** (عند دخول أي method)
- ممكن تحطها في **المطبخ** (عند methods معينة)
- ممكن تحطها عند **الكاشير** (عند methods الدفع)

### أمثلة عملية:

```java
@Aspect
@Component
public class SecurityAspect {

    // Pointcut 1: كل الـ methods في OrderService
    @Pointcut("execution(* com.example.OrderService.*(..))")
    public void allOrderServiceMethods() {}

    // Pointcut 2: كل الـ methods اللي اسمها يبدأ بـ create
    @Pointcut("execution(* create*(..))")
    public void allCreateMethods() {}

    // Pointcut 3: كل الـ methods اللي عليها annotation معين
    @Pointcut("@annotation(com.example.RequiresAuth)")
    public void methodsWithAuthAnnotation() {}

    // Pointcut 4: كل الـ methods في package معين
    @Pointcut("within(com.example.service..*)")
    public void allServicePackageMethods() {}

    // Pointcut 5: methods اللي بتاخد String parameter
    @Pointcut("execution(* *(String))")
    public void methodsWithStringParam() {}
}
```

### شرح execution expression:

```java
execution(modifiers-pattern? return-type-pattern declaring-type-pattern?
          method-name-pattern(param-pattern) throws-pattern?)

// أمثلة:
execution(public * *(..))           // كل public methods
execution(* set*(..))                // كل methods تبدأ بـ set
execution(* com.example.OrderService.*(..))  // كل methods في OrderService
execution(* com.example.*.*(..))    // كل methods في package معين
execution(* save(Order))             // methods اسمها save وتأخذ Order
```

### مثال حقيقي بالتفصيل:

```java
// الـ Target Classes
@Service
public class OrderService {
    public Order createOrder(String item) { /* ... */ }
    public void cancelOrder(Long id) { /* ... */ }
    public Order findOrder(Long id) { /* ... */ }
}

@Service
public class UserService {
    public User createUser(String name) { /* ... */ }
    public void deleteUser(Long id) { /* ... */ }
}

// الـ Aspect مع Pointcuts مختلفة
@Aspect
@Component
public class LoggingAspect {

    // Pointcut يستهدف كل create methods
    @Before("execution(* create*(..))")
    public void logAllCreations(JoinPoint jp) {
        // ده هيشتغل على:
        // ✅ OrderService.createOrder()
        // ✅ UserService.createUser()
        // ❌ OrderService.cancelOrder() - مش بتبدأ بـ create
    }

    // Pointcut يستهدف OrderService فقط
    @Before("execution(* com.example.OrderService.*(..))")
    public void logOrderOperations(JoinPoint jp) {
        // ده هيشتغل على:
        // ✅ OrderService.createOrder()
        // ✅ OrderService.cancelOrder()
        // ✅ OrderService.findOrder()
        // ❌ UserService.createUser() - مش في OrderService
    }
}
```

---

## 3️⃣ Advice 💡

### التعريف البسيط:
**الـ Advice هو "الأكشن" أو "الكود" اللي هيتنفذ عند الـ Pointcut**

### تشبيه سهل:
لو الـ Pointcut هو **الكاميرا** اللي بتراقب مكان معين،
الـ Advice هو **الحارس** اللي بيتصرف لما الكاميرا تلاحظ حاجة

### أنواع الـ Advice:

```java
@Aspect
@Component
public class ComprehensiveAspect {

    // 1️⃣ @Before - يتنفذ قبل الـ method
    @Before("execution(* com.example.OrderService.createOrder(..))")
    public void checkPermissionBeforeCreate() {
        System.out.println("🔍 التحقق من الصلاحيات قبل إنشاء الطلب");
        // لو مفيش صلاحية، ممكن نرمي Exception
    }

    // 2️⃣ @After - يتنفذ بعد الـ method (سواء نجح أو فشل)
    @After("execution(* com.example.OrderService.createOrder(..))")
    public void cleanupAfterCreate() {
        System.out.println("🧹 تنظيف الموارد بعد محاولة إنشاء الطلب");
        // يتنفذ دايماً حتى لو حصل Exception
    }

    // 3️⃣ @AfterReturning - يتنفذ بعد نجاح الـ method
    @AfterReturning(
        pointcut = "execution(* com.example.OrderService.createOrder(..))",
        returning = "order"
    )
    public void celebrateSuccess(Order order) {
        System.out.println("🎉 تم إنشاء الطلب بنجاح: " + order.getId());
        // يتنفذ فقط لو الـ method نجحت
    }

    // 4️⃣ @AfterThrowing - يتنفذ لو حصل Exception
    @AfterThrowing(
        pointcut = "execution(* com.example.OrderService.createOrder(..))",
        throwing = "error"
    )
    public void handleError(Exception error) {
        System.out.println("❌ خطأ في إنشاء الطلب: " + error.getMessage());
        // يتنفذ فقط لو حصل Exception
    }

    // 5️⃣ @Around - الأقوى! يلف حول الـ method بالكامل
    @Around("execution(* com.example.OrderService.createOrder(..))")
    public Object measureTime(ProceedingJoinPoint joinPoint) throws Throwable {
        // قبل تنفيذ الـ method
        System.out.println("⏱️ بدء قياس الوقت");
        long startTime = System.currentTimeMillis();

        try {
            // تنفيذ الـ method الأصلية
            Object result = joinPoint.proceed();

            // بعد نجاح الـ method
            long endTime = System.currentTimeMillis();
            System.out.println("✅ الوقت المستغرق: " + (endTime - startTime) + " ms");

            return result;

        } catch (Exception e) {
            // في حالة الخطأ
            System.out.println("❌ فشلت العملية");
            throw e;
        }
    }
}
```

---

## 🎯 مثال شامل يجمع الثلاثة

### السيناريو: نظام بنكي 🏦

```java
// 1️⃣ الـ TARGET OBJECT - الكلاس اللي هنضيف عليه سلوك
@Service
public class BankService {

    public void transferMoney(String from, String to, double amount) {
        // عملية التحويل
        System.out.println("تحويل " + amount + " من " + from + " إلى " + to);
    }

    public double checkBalance(String accountNumber) {
        // الاستعلام عن الرصيد
        return 1000.0; // مثال
    }

    public void withdrawCash(String accountNumber, double amount) {
        // سحب نقدي
        System.out.println("سحب " + amount + " من " + accountNumber);
    }
}

// 2️⃣ الـ ASPECT مع POINTCUTS و ADVICES
@Aspect
@Component
public class BankingAspect {

    // POINTCUT - تحديد العمليات المالية الحساسة
    @Pointcut("execution(* com.example.BankService.transfer*(..)) || " +
              "execution(* com.example.BankService.withdraw*(..))")
    public void sensitiveOperations() {}

    // ADVICE 1 - التحقق من الأمان قبل العمليات الحساسة
    @Before("sensitiveOperations()")
    public void checkSecurity(JoinPoint joinPoint) {
        System.out.println("🔒 التحقق من هوية المستخدم قبل: " +
                          joinPoint.getSignature().getName());
    }

    // ADVICE 2 - تسجيل العمليات المالية
    @AfterReturning("sensitiveOperations()")
    public void logTransaction(JoinPoint joinPoint) {
        System.out.println("📝 تسجيل العملية: " +
                          joinPoint.getSignature().getName() +
                          " مع المعاملات: " +
                          Arrays.toString(joinPoint.getArgs()));
    }

    // ADVICE 3 - إرسال تنبيه في حالة الخطأ
    @AfterThrowing(
        pointcut = "sensitiveOperations()",
        throwing = "error"
    )
    public void alertOnError(JoinPoint joinPoint, Exception error) {
        System.out.println("🚨 تنبيه أمني! فشلت العملية: " +
                          joinPoint.getSignature().getName() +
                          " - السبب: " + error.getMessage());
    }

    // ADVICE 4 - مراقبة الأداء للعمليات
    @Around("execution(* com.example.BankService.*(..))")
    public Object monitorPerformance(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();

        Object result = joinPoint.proceed();

        long duration = System.currentTimeMillis() - start;
        if (duration > 100) {
            System.out.println("⚠️ العملية " +
                              joinPoint.getSignature().getName() +
                              " بطيئة: " + duration + " ms");
        }

        return result;
    }
}

// 3️⃣ استخدام الـ Service
@Component
public class BankingApp {

    @Autowired
    private BankService bankService;

    public void performOperations() {
        // عند استدعاء أي method، الـ Aspects تشتغل تلقائياً!

        bankService.transferMoney("ACC001", "ACC002", 500);
        // Output:
        // 🔒 التحقق من هوية المستخدم قبل: transferMoney
        // تحويل 500.0 من ACC001 إلى ACC002
        // 📝 تسجيل العملية: transferMoney مع المعاملات: [ACC001, ACC002, 500.0]

        bankService.checkBalance("ACC001");
        // Output:
        // (فقط مراقبة الأداء تعمل هنا لأنها ليست عملية حساسة)

        bankService.withdrawCash("ACC001", 1000);
        // Output:
        // 🔒 التحقق من هوية المستخدم قبل: withdrawCash
        // سحب 1000.0 من ACC001
        // 📝 تسجيل العملية: withdrawCash مع المعاملات: [ACC001, 1000.0]
    }
}
```

---

## 📊 جدول مقارنة سريع

| المصطلح | الوصف | المثال |
|---------|-------|--------|
| **Target Object** | الكلاس اللي عايز تضيف عليه سلوك | `BankService`, `OrderService` |
| **Pointcut** | الشرط/المكان اللي الـ Advice هيشتغل فيه | `execution(* transfer*(..))` |
| **Advice** | الكود/السلوك اللي هيتنفذ | `@Before`, `@After`, `@Around` |

---

## 💡 تشبيه نهائي للفهم

تخيل إنك بتلعب كرة قدم ⚽:

- **Target Object** = اللاعب (محمد صلاح مثلاً)
- **Pointcut** = اللحظات المهمة (لما يجيب جول، لما ياخد كارت، لما يخرج من الملعب)
- **Advice** = رد الفعل (الاحتفال بالجول، الاحتجاج على الكارت، التصفيق عند الخروج)

```java
@Aspect
public class FootballAspect {

    // Pointcut: لما اللاعب يجيب جول
    @AfterReturning("execution(* Player.scoreGoal(..))")
    public void celebrate() {
        System.out.println("🎉 جووووول! الجمهور يحتفل!");
    }

    // Pointcut: لما اللاعب ياخد كارت أصفر
    @After("execution(* Player.receiveYellowCard(..))")
    public void protest() {
        System.out.println("😤 الجمهور يحتج على الحكم!");
    }
}
```

---

## 🎓 الخلاصة

- **Target Object**: الكلاس العادي بتاعك اللي عايز تضيف عليه features
- **Pointcut**: الفلتر اللي بيحدد "فين" و "إمتى" تضيف الـ features
- **Advice**: الكود اللي هيتنفذ (الـ feature نفسه)

الثلاثة يشتغلوا مع بعض:
1. الـ **Pointcut** بيشاور على مكان في الـ **Target Object**
2. الـ **Advice** بيتنفذ في المكان ده
3. النتيجة: **Target Object** بقى عنده قدرات إضافية من غير ما نغير كوده! 🚀