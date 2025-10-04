# أسئلة المقابلات الشخصية - RabbitMQ

## جدول المحتويات
1. [أسئلة أساسية](#أسئلة-أساسية)
2. [بروتوكول AMQP](#بروتوكول-amqp)
3. [أنواع Exchange](#أنواع-exchange)
4. [Queues والرسائل](#queues-والرسائل)
5. [Acknowledgment والتأكيد](#acknowledgment-والتأكيد)
6. [التكامل مع Spring Boot](#التكامل-مع-spring-boot)
7. [المقارنة مع Message Brokers الأخرى](#المقارنة-مع-message-brokers-الأخرى)
8. [الأداء والتحسين](#الأداء-والتحسين)

---

## أسئلة أساسية

### 1. ما هو RabbitMQ؟

**الإجابة:**
RabbitMQ هو Message Broker مفتوح المصدر يستخدم بروتوكول AMQP (Advanced Message Queuing Protocol). يعمل كوسيط بين المرسل والمستقبل للرسائل.

**المكونات الأساسية:**
- **Producer**: التطبيق الذي يرسل الرسائل
- **Exchange**: يستقبل الرسائل من Producer ويوجهها للQueues
- **Queue**: تخزن الرسائل حتى يستهلكها Consumer
- **Consumer**: التطبيق الذي يستقبل الرسائل
- **Binding**: ربط بين Exchange و Queue

**مثال بسيط:**
```java
// Producer
ConnectionFactory factory = new ConnectionFactory();
factory.setHost("localhost");
Connection connection = factory.newConnection();
Channel channel = connection.createChannel();

channel.queueDeclare("hello", false, false, false, null);
String message = "مرحباً RabbitMQ!";
channel.basicPublish("", "hello", null, message.getBytes());
System.out.println("تم إرسال: " + message);

channel.close();
connection.close();
```

```java
// Consumer
ConnectionFactory factory = new ConnectionFactory();
factory.setHost("localhost");
Connection connection = factory.newConnection();
Channel channel = connection.createChannel();

channel.queueDeclare("hello", false, false, false, null);
DeliverCallback deliverCallback = (consumerTag, delivery) -> {
    String message = new String(delivery.getBody(), "UTF-8");
    System.out.println("تم الاستقبال: " + message);
};
channel.basicConsume("hello", true, deliverCallback, consumerTag -> {});
```

---

### 2. ما الفرق بين RabbitMQ و ActiveMQ؟

**الإجابة:**

| المقارنة | RabbitMQ | ActiveMQ |
|----------|----------|----------|
| **البروتوكول** | AMQP (أساسي) | JMS (أساسي) |
| **اللغة** | Erlang | Java |
| **الأداء** | أسرع في معظم الحالات | جيد |
| **Exchange Types** | 4 أنواع متقدمة | بسيط |
| **Routing** | مرن جداً | محدود |
| **الشعبية** | عالية جداً | متوسطة |
| **التكامل** | متعدد اللغات | قوي مع Java |

**متى تستخدم RabbitMQ:**
- ✅ تحتاج routing معقد
- ✅ تطبيقات متعددة اللغات
- ✅ أداء عالي
- ✅ مرونة في توزيع الرسائل

**متى تستخدم ActiveMQ:**
- ✅ تطبيق Java بالكامل
- ✅ تحتاج JMS standard
- ✅ تكامل مع Spring بسيط

---

## بروتوكول AMQP

### 3. ما هو بروتوكول AMQP؟

**الإجابة:**
AMQP (Advanced Message Queuing Protocol) هو بروتوكول مفتوح المصدر للمراسلة.

**المميزات:**
- **Reliable**: ضمان وصول الرسائل
- **Interoperable**: يعمل مع لغات مختلفة
- **Feature-rich**: يدعم Routing, Queuing, Security
- **Vendor-neutral**: ليس خاص بشركة معينة

**الإصدارات:**
- AMQP 0-9-1 (الأكثر استخداماً في RabbitMQ)
- AMQP 1.0

---

### 4. اشرح دورة حياة الرسالة في RabbitMQ

**الإجابة:**

```
1. Producer → 2. Exchange → 3. Binding → 4. Queue → 5. Consumer

Producer sends message
    ↓
Exchange receives and routes based on routing key
    ↓
Binding rules determine which Queue(s)
    ↓
Queue stores message
    ↓
Consumer receives and processes
    ↓
Acknowledgment (optional)
```

**مثال كامل:**
```java
// 1. Producer يرسل رسالة مع routing key
channel.basicPublish(
    "logs",           // Exchange name
    "error.payment",  // Routing key
    null,
    message.getBytes()
);

// 2. Exchange يوجه الرسالة حسب النوع والروابط

// 3. Binding يحدد Queue المستهدف
channel.queueBind("error_queue", "logs", "error.*");

// 4. Queue تخزن الرسالة

// 5. Consumer يستقبل
channel.basicConsume("error_queue", false, (consumerTag, delivery) -> {
    String msg = new String(delivery.getBody(), "UTF-8");
    System.out.println("Received: " + msg);

    // 6. Acknowledgment
    channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
}, consumerTag -> {});
```

---

## أنواع Exchange

### 5. ما هي أنواع Exchange في RabbitMQ؟

**الإجابة:**
RabbitMQ يدعم 4 أنواع من Exchange:

#### 1. Direct Exchange
يوجه الرسالة للQueue التي لها نفس routing key بالضبط.

```java
// إنشاء Direct Exchange
channel.exchangeDeclare("direct_logs", "direct");

// Producer يرسل مع routing key محدد
channel.basicPublish("direct_logs", "error", null, message.getBytes());

// Binding - ربط Queue بـ routing key
channel.queueBind("error_queue", "direct_logs", "error");
channel.queueBind("all_logs", "direct_logs", "error");
channel.queueBind("all_logs", "direct_logs", "info");
```

**استخدام:**
- Log levels (error, info, warning)
- Task routing لعمال محددين

#### 2. Fanout Exchange
يرسل الرسالة لجميع Queues المربوطة (Broadcast).

```java
// إنشاء Fanout Exchange
channel.exchangeDeclare("notifications", "fanout");

// Producer يرسل بدون routing key
channel.basicPublish("notifications", "", null, message.getBytes());

// كل Queue مربوطة ستستقبل الرسالة
channel.queueBind("email_queue", "notifications", "");
channel.queueBind("sms_queue", "notifications", "");
channel.queueBind("push_queue", "notifications", "");
```

**استخدام:**
- Notifications لجميع المستخدمين
- Cache invalidation
- Real-time updates

#### 3. Topic Exchange
Routing مرن باستخدام patterns (wildcards).

```java
// إنشاء Topic Exchange
channel.exchangeDeclare("logs_topic", "topic");

// Producer يرسل مع routing key ذو أجزاء
channel.basicPublish("logs_topic", "error.payment.db", null, message.getBytes());

// Binding patterns
channel.queueBind("error_queue", "logs_topic", "error.*");
channel.queueBind("payment_queue", "logs_topic", "*.payment.*");
channel.queueBind("all_queue", "logs_topic", "#");
```

**Wildcards:**
- `*` : يطابق كلمة واحدة بالضبط
- `#` : يطابق صفر أو أكثر من الكلمات

**أمثلة:**
- `error.*` يطابق `error.payment`, `error.db`
- `*.payment.*` يطابق `error.payment.db`, `info.payment.api`
- `#` يطابق أي routing key

#### 4. Headers Exchange
Routing حسب headers بدلاً من routing key.

```java
// إنشاء Headers Exchange
channel.exchangeDeclare("headers_exchange", "headers");

// Producer يرسل مع headers
AMQP.BasicProperties props = new AMQP.BasicProperties.Builder()
    .headers(Map.of(
        "format", "pdf",
        "type", "report",
        "priority", "high"
    ))
    .build();
channel.basicPublish("headers_exchange", "", props, message.getBytes());

// Binding مع headers matching
Map<String, Object> bindingArgs = new HashMap<>();
bindingArgs.put("x-match", "all"); // أو "any"
bindingArgs.put("format", "pdf");
bindingArgs.put("type", "report");
channel.queueBind("pdf_reports", "headers_exchange", "", bindingArgs);
```

**x-match:**
- `all`: جميع headers يجب أن تطابق
- `any`: header واحد على الأقل يطابق

---

### 6. متى تستخدم كل نوع من Exchange؟

**الإجابة:**

| النوع | الاستخدام | مثال |
|------|----------|------|
| **Direct** | Routing محدد ومباشر | Task queues لعمال مختلفين |
| **Fanout** | Broadcast لجميع المستمعين | Notifications, Cache updates |
| **Topic** | Routing مرن بـ patterns | Logging system, Event routing |
| **Headers** | Routing معقد بـ metadata | Content-based routing |

**أمثلة من الواقع:**

```java
// 1. Direct - نظام المهام
channel.exchangeDeclare("tasks", "direct");
channel.basicPublish("tasks", "image.resize", null, task.getBytes());
channel.basicPublish("tasks", "video.encode", null, task.getBytes());

// 2. Fanout - إشعارات المستخدمين
channel.exchangeDeclare("user.notifications", "fanout");
channel.basicPublish("user.notifications", "", null, notification.getBytes());

// 3. Topic - نظام اللوقات
channel.exchangeDeclare("logs", "topic");
channel.basicPublish("logs", "error.payment.stripe", null, log.getBytes());
channel.basicPublish("logs", "info.user.login", null, log.getBytes());

// 4. Headers - Content routing
channel.exchangeDeclare("content", "headers");
// Routing حسب file type, size, priority
```

---

## Queues والرسائل

### 7. ما هي خصائص Queue في RabbitMQ؟

**الإجابة:**

```java
// إنشاء Queue مع جميع الخصائص
channel.queueDeclare(
    "my_queue",          // Queue name
    true,                // Durable - تبقى بعد إعادة تشغيل RabbitMQ
    false,               // Exclusive - فقط لهذا Connection
    false,               // Auto-delete - تحذف عند عدم الاستخدام
    Map.of(              // Arguments إضافية
        "x-message-ttl", 60000,              // TTL للرسائل (60 ثانية)
        "x-max-length", 1000,                // أقصى عدد رسائل
        "x-max-length-bytes", 10485760,      // أقصى حجم (10MB)
        "x-dead-letter-exchange", "dlx",     // Dead Letter Exchange
        "x-max-priority", 10                 // أقصى أولوية
    )
);
```

**الخصائص الرئيسية:**

1. **Durable**: Queue تبقى بعد restart
```java
channel.queueDeclare("durable_queue", true, false, false, null);
```

2. **Exclusive**: Queue خاصة بـ connection واحد
```java
channel.queueDeclare("temp_queue", false, true, false, null);
```

3. **Auto-delete**: تحذف تلقائياً عند عدم وجود consumers
```java
channel.queueDeclare("temp_queue", false, false, true, null);
```

---

### 8. ما هو Message TTL و Queue TTL؟

**الإجابة:**

#### Message TTL
تحديد عمر الرسالة قبل أن تُحذف.

```java
// 1. على مستوى Queue (جميع الرسائل)
Map<String, Object> args = new HashMap<>();
args.put("x-message-ttl", 60000); // 60 ثانية
channel.queueDeclare("my_queue", true, false, false, args);

// 2. على مستوى الرسالة الواحدة
AMQP.BasicProperties props = new AMQP.BasicProperties.Builder()
    .expiration("60000") // 60 ثانية
    .build();
channel.basicPublish("", "my_queue", props, message.getBytes());
```

#### Queue TTL
Queue نفسها تحذف بعد فترة عدم استخدام.

```java
Map<String, Object> args = new HashMap<>();
args.put("x-expires", 300000); // 5 دقائق بدون استخدام
channel.queueDeclare("temp_queue", false, false, false, args);
```

**استخدامات:**
- رسائل الـ OTP (صالحة لدقائق فقط)
- Temporary sessions
- Cache expiration

---

### 9. ما هو Dead Letter Exchange (DLX)؟

**الإجابة:**
Dead Letter Exchange يستقبل الرسائل التي فشلت في المعالجة.

**الحالات التي ترسل الرسالة إلى DLX:**
1. رسالة رُفضت بـ `basic.reject` أو `basic.nack` مع `requeue=false`
2. رسالة انتهت صلاحيتها (TTL)
3. Queue امتلأت (max-length)

**إعداد DLX:**
```java
// 1. إنشاء Dead Letter Exchange
channel.exchangeDeclare("dlx", "direct");
channel.queueDeclare("dead_letter_queue", true, false, false, null);
channel.queueBind("dead_letter_queue", "dlx", "failed");

// 2. Queue رئيسية مع DLX
Map<String, Object> args = new HashMap<>();
args.put("x-dead-letter-exchange", "dlx");
args.put("x-dead-letter-routing-key", "failed");
args.put("x-message-ttl", 60000); // الرسائل تنتهي بعد 60 ثانية
channel.queueDeclare("main_queue", true, false, false, args);

// 3. Consumer يرفض الرسالة
channel.basicConsume("main_queue", false, (consumerTag, delivery) -> {
    try {
        processMessage(delivery.getBody());
        channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
    } catch (Exception e) {
        // رفض الرسالة - سترسل إلى DLX
        channel.basicReject(delivery.getEnvelope().getDeliveryTag(), false);
    }
}, consumerTag -> {});
```

**مثال كامل - نظام معالجة Payments:**
```java
// Main Queue
Map<String, Object> mainArgs = new HashMap<>();
mainArgs.put("x-dead-letter-exchange", "payment.dlx");
mainArgs.put("x-dead-letter-routing-key", "payment.failed");
mainArgs.put("x-message-ttl", 300000); // 5 دقائق
channel.queueDeclare("payment.processing", true, false, false, mainArgs);

// Dead Letter Queue
channel.exchangeDeclare("payment.dlx", "direct");
channel.queueDeclare("payment.failed", true, false, false, null);
channel.queueBind("payment.failed", "payment.dlx", "payment.failed");

// Consumer
channel.basicConsume("payment.processing", false, (consumerTag, delivery) -> {
    PaymentRequest payment = deserialize(delivery.getBody());
    try {
        processPayment(payment);
        channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
    } catch (PaymentException e) {
        // فشل الدفع - إرسال إلى Failed Queue
        channel.basicReject(delivery.getEnvول().getDeliveryTag(), false);
        alertAdmin("Payment failed: " + payment.getId());
    }
}, consumerTag -> {});
```

---

## Acknowledgment والتأكيد

### 10. ما الفرق بين Auto-Ack و Manual Ack؟

**الإجابة:**

#### Auto-Acknowledgment
RabbitMQ يعتبر الرسالة مستهلكة فور إرسالها للConsumer.

```java
// Auto-Ack = true
channel.basicConsume("my_queue", true, (consumerTag, delivery) -> {
    String message = new String(delivery.getBody(), "UTF-8");
    processMessage(message); // إذا فشلت، الرسالة ضاعت!
}, consumerTag -> {});
```

**المشاكل:**
- ❌ إذا فشلت المعالجة، الرسالة تضيع
- ❌ إذا Consumer توقف، الرسائل تضيع
- ✅ أداء أعلى

#### Manual Acknowledgment
Consumer يؤكد استلام الرسالة يدوياً بعد المعالجة الناجحة.

```java
// Auto-Ack = false
channel.basicConsume("my_queue", false, (consumerTag, delivery) -> {
    try {
        String message = new String(delivery.getBody(), "UTF-8");
        processMessage(message);

        // تأكيد النجاح
        channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
    } catch (Exception e) {
        // رفض وإعادة للQueue
        channel.basicNack(delivery.getEnvelope().getDeliveryTag(), false, true);
    }
}, consumerTag -> {});
```

**المميزات:**
- ✅ ضمان عدم فقدان الرسائل
- ✅ إعادة المحاولة عند الفشل
- ❌ أداء أقل قليلاً

---

### 11. ما الفرق بين basicReject و basicNack؟

**الإجابة:**

```java
// basicReject - رفض رسالة واحدة
channel.basicReject(
    deliveryTag,
    false  // requeue: false = أرسل لـ DLX, true = أعد للQueue
);

// basicNack - رفض رسالة أو عدة رسائل
channel.basicNack(
    deliveryTag,
    true,   // multiple: true = رفض جميع الرسائل حتى هذا deliveryTag
    false   // requeue: false = DLX, true = Queue
);
```

**مثال - معالجة Batch:**
```java
channel.basicConsume("batch_queue", false, (consumerTag, delivery) -> {
    try {
        processBatch(delivery.getBody());
        // تأكيد جميع الرسائل في الـ Batch
        channel.basicAck(delivery.getEnvelope().getDeliveryTag(), true);
    } catch (Exception e) {
        // رفض جميع الرسائل في الـ Batch
        channel.basicNack(delivery.getEnvelope().getDeliveryTag(), true, false);
    }
}, consumerTag -> {});
```

---

### 12. ما هو Publisher Confirms؟

**الإجابة:**
Publisher Confirms يضمن أن RabbitMQ استقبل الرسالة بنجاح.

**بدون Confirms:**
```java
// Producer لا يعرف إذا وصلت الرسالة
channel.basicPublish("", "my_queue", null, message.getBytes());
// ماذا لو فشلت الشبكة؟
```

**مع Confirms:**
```java
// 1. تفعيل Publisher Confirms
channel.confirmSelect();

// 2. إرسال رسالة
channel.basicPublish("", "my_queue", null, message.getBytes());

// 3. انتظار التأكيد
if (channel.waitForConfirms(5000)) {
    System.out.println("تم التأكيد - الرسالة وصلت");
} else {
    System.out.println("فشل التأكيد - أعد المحاولة");
}
```

**Async Confirms (الأفضل للأداء):**
```java
channel.confirmSelect();

// Listener للتأكيدات
channel.addConfirmListener(new ConfirmListener() {
    @Override
    public void handleAck(long deliveryTag, boolean multiple) {
        System.out.println("تم التأكيد: " + deliveryTag);
    }

    @Override
    public void handleNack(long deliveryTag, boolean multiple) {
        System.out.println("فشل: " + deliveryTag + " - أعد الإرسال");
    }
});

// إرسال رسائل متعددة
for (int i = 0; i < 1000; i++) {
    channel.basicPublish("", "my_queue", null, ("Message " + i).getBytes());
}
```

---

## التكامل مع Spring Boot

### 13. كيف تستخدم RabbitMQ مع Spring Boot؟

**الإجابة:**

#### 1. إضافة Dependencies
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

#### 2. Configuration
```yaml
# application.yml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
    virtual-host: /
    listener:
      simple:
        acknowledge-mode: manual
        prefetch: 10
```

#### 3. RabbitMQ Configuration Class
```java
@Configuration
public class RabbitMQConfig {

    // Queue
    @Bean
    public Queue orderQueue() {
        return QueueBuilder.durable("orders")
            .withArgument("x-message-ttl", 60000)
            .withArgument("x-dead-letter-exchange", "dlx")
            .build();
    }

    // Direct Exchange
    @Bean
    public DirectExchange orderExchange() {
        return new DirectExchange("order.exchange");
    }

    // Binding
    @Bean
    public Binding orderBinding() {
        return BindingBuilder
            .bind(orderQueue())
            .to(orderExchange())
            .with("order.new");
    }

    // Topic Exchange مثال
    @Bean
    public TopicExchange logsExchange() {
        return new TopicExchange("logs");
    }

    @Bean
    public Queue errorQueue() {
        return new Queue("error.logs");
    }

    @Bean
    public Binding errorBinding() {
        return BindingBuilder
            .bind(errorQueue())
            .to(logsExchange())
            .with("error.*");
    }

    // Fanout Exchange مثال
    @Bean
    public FanoutExchange notificationExchange() {
        return new FanoutExchange("notifications");
    }

    @Bean
    public Queue emailQueue() {
        return new Queue("email.notifications");
    }

    @Bean
    public Queue smsQueue() {
        return new Queue("sms.notifications");
    }

    @Bean
    public Binding emailBinding() {
        return BindingBuilder.bind(emailQueue()).to(notificationExchange());
    }

    @Bean
    public Binding smsBinding() {
        return BindingBuilder.bind(smsQueue()).to(notificationExchange());
    }

    // Message Converter
    @Bean
    public Jackson2JsonMessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }
}
```

#### 4. Producer (Publisher)
```java
@Service
public class OrderProducer {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void sendOrder(Order order) {
        rabbitTemplate.convertAndSend(
            "order.exchange",
            "order.new",
            order
        );
        log.info("تم إرسال طلب: {}", order.getId());
    }

    // إرسال مع تأكيد
    public void sendOrderWithConfirm(Order order) {
        rabbitTemplate.setConfirmCallback((correlation, ack, reason) -> {
            if (ack) {
                log.info("تم التأكيد: {}", order.getId());
            } else {
                log.error("فشل الإرسال: {}, السبب: {}", order.getId(), reason);
            }
        });

        rabbitTemplate.convertAndSend("order.exchange", "order.new", order);
    }
}
```

#### 5. Consumer (Listener)
```java
@Service
public class OrderConsumer {

    // استماع بسيط
    @RabbitListener(queues = "orders")
    public void handleOrder(Order order) {
        log.info("تم استقبال طلب: {}", order.getId());
        processOrder(order);
    }

    // مع Manual Acknowledgment
    @RabbitListener(queues = "orders")
    public void handleOrderWithAck(
        Order order,
        Channel channel,
        @Header(AmqpHeaders.DELIVERY_TAG) long tag
    ) throws IOException {
        try {
            processOrder(order);
            channel.basicAck(tag, false);
            log.info("تمت معالجة الطلب: {}", order.getId());
        } catch (Exception e) {
            channel.basicNack(tag, false, false); // إرسال لـ DLX
            log.error("فشلت المعالجة: {}", order.getId(), e);
        }
    }

    // مع Retry
    @RabbitListener(queues = "orders")
    @RetryableTopic(
        attempts = "3",
        backoff = @Backoff(delay = 2000, multiplier = 2)
    )
    public void handleOrderWithRetry(Order order) {
        processOrder(order);
    }
}
```

#### 6. مثال كامل - نظام E-Commerce

```java
// Model
@Data
public class Order {
    private String id;
    private String customerId;
    private List<OrderItem> items;
    private BigDecimal total;
    private OrderStatus status;
    private LocalDateTime createdAt;
}

// Producer
@Service
public class OrderService {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void placeOrder(Order order) {
        order.setStatus(OrderStatus.PENDING);
        order.setCreatedAt(LocalDateTime.now());

        // حفظ في Database
        orderRepository.save(order);

        // إرسال إلى RabbitMQ
        rabbitTemplate.convertAndSend(
            "order.exchange",
            "order.created",
            order
        );

        log.info("تم إنشاء طلب: {}", order.getId());
    }
}

// Consumer - Payment Service
@Service
public class PaymentConsumer {

    @RabbitListener(queues = "payment.queue")
    public void processPayment(
        Order order,
        Channel channel,
        @Header(AmqpHeaders.DELIVERY_TAG) long tag
    ) throws IOException {
        try {
            // معالجة الدفع
            PaymentResult result = paymentGateway.process(order);

            if (result.isSuccess()) {
                order.setStatus(OrderStatus.PAID);
                // إرسال للـ Shipping
                rabbitTemplate.convertAndSend(
                    "order.exchange",
                    "order.paid",
                    order
                );
                channel.basicAck(tag, false);
            } else {
                order.setStatus(OrderStatus.PAYMENT_FAILED);
                channel.basicNack(tag, false, false);
            }
        } catch (Exception e) {
            log.error("خطأ في معالجة الدفع", e);
            channel.basicNack(tag, false, true); // إعادة المحاولة
        }
    }
}

// Consumer - Shipping Service
@Service
public class ShippingConsumer {

    @RabbitListener(queues = "shipping.queue")
    public void processShipping(Order order) {
        log.info("بدء الشحن للطلب: {}", order.getId());

        // إنشاء شحنة
        Shipment shipment = createShipment(order);

        // إرسال إشعار للعميل
        rabbitTemplate.convertAndSend(
            "notifications",  // Fanout exchange
            "",
            new Notification("تم شحن طلبك: " + order.getId())
        );
    }
}

// Consumer - Notifications (Email)
@Service
public class EmailConsumer {

    @RabbitListener(queues = "email.notifications")
    public void sendEmail(Notification notification) {
        emailService.send(notification);
        log.info("تم إرسال بريد إلكتروني");
    }
}

// Consumer - Notifications (SMS)
@Service
public class SmsConsumer {

    @RabbitListener(queues = "sms.notifications")
    public void sendSms(Notification notification) {
        smsService.send(notification);
        log.info("تم إرسال SMS");
    }
}
```

---

## المقارنة مع Message Brokers الأخرى

### 14. ما الفرق بين RabbitMQ و Apache Kafka؟

**الإجابة:**

| المقارنة | RabbitMQ | Apache Kafka |
|----------|----------|--------------|
| **النوع** | Message Broker | Event Streaming Platform |
| **النموذج** | Push-based | Pull-based |
| **التخزين** | Queue (تحذف بعد الاستهلاك) | Log (تبقى لمدة محددة) |
| **الترتيب** | في Queue واحدة | في Partition واحدة |
| **الأداء** | جيد (آلاف/ثانية) | ممتاز (ملايين/ثانية) |
| **الاستخدام** | Task queues, RPC | Event sourcing, Streaming |
| **Routing** | مرن جداً (Exchange types) | بسيط (Topics/Partitions) |
| **Consumer Groups** | محدود | متقدم |

**متى تستخدم RabbitMQ:**
- ✅ Task queues
- ✅ RPC patterns
- ✅ Routing معقد
- ✅ Guaranteed delivery
- ✅ Message priority

**متى تستخدم Kafka:**
- ✅ Event streaming
- ✅ Big data processing
- ✅ Log aggregation
- ✅ Real-time analytics
- ✅ High throughput

---

### 15. RabbitMQ vs Redis Pub/Sub؟

**الإجابة:**

| المقارنة | RabbitMQ | Redis Pub/Sub |
|----------|----------|---------------|
| **الضمانات** | Guaranteed delivery | Fire-and-forget |
| **التخزين** | Persistent queues | في الذاكرة فقط |
| **Acknowledgment** | يدعم | لا يدعم |
| **Routing** | متقدم | بسيط |
| **الأداء** | جيد | ممتاز |
| **استمرارية** | الرسائل تبقى | إذا لا يوجد مستمع، الرسالة تضيع |

**Redis Pub/Sub:**
```java
// Publisher
jedis.publish("notifications", "مرحباً");

// Subscriber
jedis.subscribe(new JedisPubSub() {
    @Override
    public void onMessage(String channel, String message) {
        System.out.println("Received: " + message);
    }
}, "notifications");
```

**الفرق الرئيسي:**
- Redis: إذا لا يوجد subscriber متصل، الرسالة تضيع
- RabbitMQ: الرسائل تخزن في Queue حتى يستهلكها consumer

---

## الأداء والتحسين

### 16. كيف تحسن أداء RabbitMQ؟

**الإجابة:**

#### 1. Prefetch Count
تحديد عدد الرسائل التي يستقبلها Consumer مرة واحدة.

```java
// بدون prefetch - Consumer يستقبل جميع الرسائل
channel.basicConsume("my_queue", false, callback);

// مع prefetch - Consumer يستقبل 10 رسائل فقط في المرة
channel.basicQos(10);
channel.basicConsume("my_queue", false, callback);
```

**فائدة:**
- توزيع متساوٍ للرسائل بين Consumers
- منع Consumer واحد من استهلاك جميع الرسائل

```java
// Spring Boot
spring:
  rabbitmq:
    listener:
      simple:
        prefetch: 10
```

#### 2. Lazy Queues
تخزين الرسائل على الـ Disk بدلاً من الـ RAM.

```java
Map<String, Object> args = new HashMap<>();
args.put("x-queue-mode", "lazy");
channel.queueDeclare("large_queue", true, false, false, args);
```

**متى تستخدمها:**
- ✅ Queues كبيرة جداً (ملايين الرسائل)
- ✅ تجنب استهلاك الذاكرة
- ❌ الأداء أقل قليلاً

#### 3. Publisher Confirms بشكل Batch

```java
channel.confirmSelect();
int batchSize = 100;
int outstandingConfirms = 0;

for (int i = 0; i < 10000; i++) {
    channel.basicPublish("", "queue", null, message.getBytes());
    outstandingConfirms++;

    if (outstandingConfirms == batchSize) {
        channel.waitForConfirmsOrDie(5000);
        outstandingConfirms = 0;
    }
}

if (outstandingConfirms > 0) {
    channel.waitForConfirmsOrDie(5000);
}
```

#### 4. Connection Pooling

```java
@Bean
public CachingConnectionFactory connectionFactory() {
    CachingConnectionFactory factory = new CachingConnectionFactory("localhost");
    factory.setChannelCacheSize(25);
    factory.setConnectionCacheSize(3);
    return factory;
}
```

#### 5. Message Batching

```java
// بدلاً من إرسال 1000 رسالة
for (int i = 0; i < 1000; i++) {
    channel.basicPublish("", "queue", null, ("Message " + i).getBytes());
}

// أرسل batch واحد
List<String> messages = IntStream.range(0, 1000)
    .mapToObj(i -> "Message " + i)
    .collect(Collectors.toList());
String batch = String.join(",", messages);
channel.basicPublish("", "queue", null, batch.getBytes());
```

---

### 17. كيف تراقب RabbitMQ؟

**الإجابة:**

#### 1. Management UI
RabbitMQ يوفر واجهة ويب على `http://localhost:15672`

```bash
# تفعيل Management Plugin
rabbitmq-plugins enable rabbitmq_management
```

**المعلومات المتاحة:**
- عدد الرسائل في كل Queue
- معدل الإرسال والاستقبال
- Connections و Channels
- Memory و Disk usage

#### 2. Prometheus + Grafana

```bash
# تفعيل Prometheus Plugin
rabbitmq-plugins enable rabbitmq_prometheus
```

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'rabbitmq'
    static_configs:
      - targets: ['localhost:15692']
```

#### 3. Spring Boot Actuator

```xml
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
        include: health,metrics,rabbitmq
```

```java
@Component
public class RabbitMQHealthIndicator implements HealthIndicator {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Override
    public Health health() {
        try {
            rabbitTemplate.execute(channel -> {
                channel.queueDeclarePassive("health-check");
                return true;
            });
            return Health.up().build();
        } catch (Exception e) {
            return Health.down(e).build();
        }
    }
}
```

#### 4. Custom Metrics

```java
@Service
public class MetricsService {

    private final MeterRegistry meterRegistry;

    public void recordMessageSent(String queue) {
        meterRegistry.counter("rabbitmq.messages.sent",
            "queue", queue
        ).increment();
    }

    public void recordMessageProcessed(String queue, long duration) {
        meterRegistry.timer("rabbitmq.message.processing.time",
            "queue", queue
        ).record(duration, TimeUnit.MILLISECONDS);
    }
}
```

---

### 18. ما هي أفضل الممارسات لـ RabbitMQ؟

**الإجابة:**

#### 1. استخدم Manual Acknowledgment
```java
// ❌ سيء
channel.basicConsume("queue", true, callback); // Auto-ack

// ✅ جيد
channel.basicConsume("queue", false, (tag, delivery) -> {
    try {
        process(delivery);
        channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
    } catch (Exception e) {
        channel.basicNack(delivery.getEnvelope().getDeliveryTag(), false, false);
    }
});
```

#### 2. استخدم Dead Letter Exchanges
```java
Map<String, Object> args = new HashMap<>();
args.put("x-dead-letter-exchange", "dlx");
args.put("x-message-ttl", 60000);
channel.queueDeclare("main_queue", true, false, false, args);
```

#### 3. حدد Message TTL
```java
args.put("x-message-ttl", 300000); // 5 دقائق
```

#### 4. استخدم Durable Queues و Persistent Messages
```java
// Durable queue
channel.queueDeclare("queue", true, false, false, null);

// Persistent message
AMQP.BasicProperties props = new AMQP.BasicProperties.Builder()
    .deliveryMode(2) // Persistent
    .build();
channel.basicPublish("", "queue", props, message.getBytes());
```

#### 5. استخدم Connection Pooling
```java
@Bean
public CachingConnectionFactory connectionFactory() {
    CachingConnectionFactory factory = new CachingConnectionFactory();
    factory.setChannelCacheSize(25);
    return factory;
}
```

#### 6. راقب الأداء
```java
@Scheduled(fixedRate = 60000)
public void monitorQueues() {
    Map<String, Object> queueInfo = rabbitAdmin.getQueueInfo("my_queue");
    Integer messageCount = (Integer) queueInfo.get("QUEUE_MESSAGE_COUNT");
    if (messageCount > 10000) {
        alertAdmin("Queue has " + messageCount + " messages!");
    }
}
```

---

## خاتمة

### أسئلة سريعة للمراجعة:

1. **ما الفرق بين Queue و Exchange؟**
   - Queue: تخزن الرسائل
   - Exchange: توجه الرسائل للQueues

2. **كم نوع Exchange في RabbitMQ؟**
   - 4 أنواع: Direct, Fanout, Topic, Headers

3. **متى تستخدم Topic Exchange؟**
   - عند الحاجة لـ routing مرن بـ patterns

4. **ما هو Dead Letter Exchange؟**
   - Exchange يستقبل الرسائل الفاشلة

5. **ما الفرق بين RabbitMQ و Kafka؟**
   - RabbitMQ: Message broker (task queues)
   - Kafka: Event streaming (logs/analytics)

6. **كيف تضمن عدم فقدان الرسائل؟**
   - Manual acknowledgment
   - Durable queues
   - Persistent messages
   - Publisher confirms

---

**موارد إضافية:**
- [RabbitMQ Official Docs](https://www.rabbitmq.com/documentation.html)
- [RabbitMQ Tutorials](https://www.rabbitmq.com/getstarted.html)
- [Spring AMQP Documentation](https://docs.spring.io/spring-amqp/reference/)
