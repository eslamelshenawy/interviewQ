# دليل RabbitMQ الشامل

## جدول المحتويات
1. [ما هو RabbitMQ](#ما-هو-rabbitmq)
2. [بروتوكول AMQP](#بروتوكول-amqp)
3. [الميزات الرئيسية](#الميزات-الرئيسية)
4. [البنية المعمارية](#البنية-المعمارية)
5. [حالات الاستخدام](#حالات-الاستخدام)
6. [التثبيت والإعداد](#التثبيت-والإعداد)

## ما هو RabbitMQ

**RabbitMQ** هو وسيط رسائل (Message Broker) مفتوح المصدر يعمل كوسيط بين الأنظمة المختلفة لتبادل الرسائل بشكل موثوق وفعال. تم تطويره باستخدام لغة Erlang ويعتمد على بروتوكول AMQP (Advanced Message Queuing Protocol).

### الفكرة الأساسية
بدلاً من أن يتصل التطبيق A مباشرة بالتطبيق B لإرسال البيانات، يرسل التطبيق A الرسالة إلى RabbitMQ، ثم يقوم RabbitMQ بتوصيل الرسالة إلى التطبيق B. هذا يوفر:
- **فصل المكونات (Decoupling)**: التطبيقات لا تحتاج لمعرفة بعضها البعض
- **الموثوقية**: إذا كان التطبيق B غير متاح، تبقى الرسالة محفوظة
- **قابلية التوسع**: يمكن إضافة مستهلكين متعددين بسهولة

## بروتوكول AMQP

**AMQP (Advanced Message Queuing Protocol)** هو بروتوكول قياسي مفتوح لتبادل الرسائل بين الأنظمة.

### خصائص AMQP:

1. **موثوق (Reliable)**
   - يضمن وصول الرسائل بدون فقدان
   - يدعم آليات التأكيد (Acknowledgment)

2. **قابل للتشغيل البيني (Interoperable)**
   - يعمل عبر منصات ولغات برمجة مختلفة
   - معيار موحد يدعمه العديد من الوسطاء

3. **آمن (Secure)**
   - يدعم TLS/SSL للتشفير
   - آليات مصادقة متقدمة

4. **موجه للرسائل (Message-Oriented)**
   - يركز على نقل الرسائل بكفاءة
   - يدعم أنماط مختلفة من التوجيه

### مكونات بروتوكول AMQP:

```
┌─────────────┐                    ┌──────────────┐                    ┌─────────────┐
│  Publisher  │ ──── Message ───>  │   Exchange   │ ──── Routing ───>  │    Queue    │
└─────────────┘                    └──────────────┘                    └─────────────┘
                                           │                                    │
                                      Binding Key                               │
                                           │                                    │
                                           └────────────────────────────────────┘
                                                                                 │
                                                                                 v
                                                                         ┌─────────────┐
                                                                         │  Consumer   │
                                                                         └─────────────┘
```

## الميزات الرئيسية

### 1. المرونة في التوجيه (Flexible Routing)
- دعم أنواع مختلفة من Exchange (Direct, Fanout, Topic, Headers)
- قواعد توجيه معقدة باستخدام Routing Keys و Binding Keys

### 2. الموثوقية (Reliability)
- **Message Acknowledgment**: تأكيد استلام الرسائل
- **Publisher Confirms**: تأكيد نشر الرسائل
- **Persistence**: حفظ الرسائل على القرص
- **High Availability**: دعم التجميع والنسخ المتماثل

### 3. التجميع والتوزيع (Clustering & Federation)
- إنشاء مجموعات من خوادم RabbitMQ للتوسع الأفقي
- مشاركة الأحمال بين العقد المختلفة

### 4. إدارة متقدمة (Advanced Management)
- واجهة إدارة ويب شاملة
- HTTP API للإدارة البرمجية
- أدوات مراقبة وتسجيل متقدمة

### 5. البرمجة اللغوية المتعددة (Multi-Language Support)
- مكتبات عميلة لمعظم لغات البرمجة (Java, Python, .NET, PHP, Ruby, etc.)

### 6. المكونات الإضافية (Plugins)
- نظام مكونات إضافية غني
- دعم بروتوكولات إضافية (MQTT, STOMP)

## البنية المعمارية

### المكونات الأساسية

#### 1. المنتج (Producer)
التطبيق الذي يرسل الرسائل إلى RabbitMQ.

```java
// مثال بسيط على Producer
public class Producer {
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");

        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {

            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            String message = "Hello World!";
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
            System.out.println(" [x] Sent '" + message + "'");
        }
    }
}
```

#### 2. المستهلك (Consumer)
التطبيق الذي يستقبل الرسائل من RabbitMQ.

```java
// مثال بسيط على Consumer
public class Consumer {
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");

        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [x] Received '" + message + "'");
        };

        channel.basicConsume(QUEUE_NAME, true, deliverCallback, consumerTag -> { });
    }
}
```

#### 3. الطابور (Queue)
مخزن مؤقت يحتفظ بالرسائل حتى يتم استهلاكها.

**خصائص Queue:**
- **Name**: اسم فريد للطابور
- **Durable**: هل يبقى بعد إعادة تشغيل الخادم؟
- **Exclusive**: خاص بـ connection واحد فقط
- **Auto-delete**: يُحذف تلقائياً عند عدم الاستخدام
- **Arguments**: معاملات إضافية (TTL, Max Length, DLX, etc.)

```java
// إنشاء Queue مع خيارات متقدمة
Map<String, Object> args = new HashMap<>();
args.put("x-message-ttl", 60000); // TTL = 60 ثانية
args.put("x-max-length", 1000);   // أقصى 1000 رسالة
args.put("x-dead-letter-exchange", "dlx.exchange"); // DLX

channel.queueDeclare(
    "my.queue",        // name
    true,              // durable
    false,             // exclusive
    false,             // auto-delete
    args               // arguments
);
```

#### 4. التبادل (Exchange)
يستقبل الرسائل من المنتجين ويوجهها إلى الطوابير بناءً على قواعد محددة.

**أنواع Exchange:**

##### أ. Direct Exchange
يوجه الرسائل إلى الطوابير بناءً على تطابق دقيق للـ Routing Key.

```
Publisher → [routing_key="error"] → Direct Exchange
                                          │
                        ┌─────────────────┼─────────────────┐
                        │                 │                 │
                 [binding="error"]  [binding="info"]  [binding="warning"]
                        │                 │                 │
                        v                 v                 v
                   Error Queue        Info Queue      Warning Queue
```

```java
// Direct Exchange Example
channel.exchangeDeclare("logs.direct", "direct", true);

// ربط الطوابير بمفاتيح مختلفة
channel.queueBind("error.queue", "logs.direct", "error");
channel.queueBind("info.queue", "logs.direct", "info");
channel.queueBind("warning.queue", "logs.direct", "warning");

// إرسال رسالة
channel.basicPublish("logs.direct", "error", null, message.getBytes());
```

##### ب. Fanout Exchange
يوجه الرسائل إلى جميع الطوابير المربوطة (يتجاهل Routing Key).

```
Publisher → Fanout Exchange
                  │
    ┌─────────────┼─────────────┐
    │             │             │
    v             v             v
Queue 1       Queue 2       Queue 3
```

```java
// Fanout Exchange Example
channel.exchangeDeclare("logs.fanout", "fanout", true);

// ربط الطوابير (الـ routing key يُتجاهل)
channel.queueBind("queue1", "logs.fanout", "");
channel.queueBind("queue2", "logs.fanout", "");
channel.queueBind("queue3", "logs.fanout", "");

// جميع الطوابير ستستقبل الرسالة
channel.basicPublish("logs.fanout", "", null, message.getBytes());
```

##### ج. Topic Exchange
يوجه الرسائل بناءً على نمط (Pattern) في الـ Routing Key.

**الرموز الخاصة:**
- `*` (نجمة): تطابق كلمة واحدة بالضبط
- `#` (هاش): تطابق صفر أو أكثر من الكلمات

```
Routing Keys: "user.created.email"
              "user.updated.sms"
              "order.created.notification"

Binding Patterns:
  "user.*.*"        → يطابق جميع رسائل المستخدم
  "*.created.*"     → يطابق جميع رسائل الإنشاء
  "user.#"          → يطابق جميع ما يبدأ بـ user
  "#.email"         → يطابق جميع ما ينتهي بـ email
```

```java
// Topic Exchange Example
channel.exchangeDeclare("events.topic", "topic", true);

// ربط الطوابير بأنماط مختلفة
channel.queueBind("user.events", "events.topic", "user.#");
channel.queueBind("created.events", "events.topic", "*.created.*");
channel.queueBind("email.events", "events.topic", "#.email");

// إرسال رسالة
channel.basicPublish("events.topic", "user.created.email", null, message.getBytes());
```

##### د. Headers Exchange
يوجه الرسائل بناءً على Headers بدلاً من Routing Key.

```java
// Headers Exchange Example
channel.exchangeDeclare("events.headers", "headers", true);

// تعريف Headers للربط
Map<String, Object> bindingArgs = new HashMap<>();
bindingArgs.put("x-match", "all"); // "all" أو "any"
bindingArgs.put("format", "pdf");
bindingArgs.put("type", "report");

channel.queueBind("pdf.reports", "events.headers", "", bindingArgs);

// إرسال رسالة مع Headers
AMQP.BasicProperties props = new AMQP.BasicProperties.Builder()
    .headers(Map.of("format", "pdf", "type", "report"))
    .build();

channel.basicPublish("events.headers", "", props, message.getBytes());
```

#### 5. الربط (Binding)
العلاقة بين Exchange و Queue التي تحدد كيفية توجيه الرسائل.

```java
// Binding Syntax
channel.queueBind(
    String queue,          // اسم الـ Queue
    String exchange,       // اسم الـ Exchange
    String routingKey,     // مفتاح التوجيه
    Map<String, Object> arguments  // معاملات إضافية
);
```

### تدفق الرسائل الكامل

```
1. Producer ينشئ رسالة
   │
   v
2. يرسل الرسالة إلى Exchange مع Routing Key
   │
   v
3. Exchange يستخدم الـ Routing Key والـ Bindings لتحديد الـ Queues
   │
   v
4. الرسالة تُخزن في Queue/Queues
   │
   v
5. Consumer يستقبل الرسالة من Queue
   │
   v
6. Consumer يعالج الرسالة ويرسل ACK
   │
   v
7. RabbitMQ يحذف الرسالة من Queue
```

## حالات الاستخدام

### 1. معالجة غير متزامنة (Asynchronous Processing)
- معالجة الصور أو الفيديوهات
- إرسال البريد الإلكتروني
- توليد التقارير

```java
// مثال: معالجة الصور
@Service
public class ImageProcessor {

    @RabbitListener(queues = "image.processing")
    public void processImage(String imageUrl) {
        // معالجة الصورة في الخلفية
        BufferedImage image = downloadImage(imageUrl);
        BufferedImage thumbnail = createThumbnail(image);
        saveImage(thumbnail);

        // إرسال إشعار بالانتهاء
        notificationService.send("Image processed successfully");
    }
}
```

### 2. فصل الأنظمة (Decoupling Systems)
- Microservices Architecture
- Event-Driven Architecture
- SOA (Service-Oriented Architecture)

```java
// مثال: Order Service → Inventory Service
@Service
public class OrderService {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void createOrder(Order order) {
        // حفظ الطلب
        orderRepository.save(order);

        // إرسال حدث إلى Inventory Service
        OrderEvent event = new OrderEvent(order.getId(), order.getItems());
        rabbitTemplate.convertAndSend("orders.exchange", "order.created", event);
    }
}
```

### 3. موازنة الأحمال (Load Balancing)
- توزيع المهام على عدة Workers
- معالجة متوازية للمهام الثقيلة

```java
// عدة Workers يعملون على نفس الـ Queue
@Component
public class Worker1 {
    @RabbitListener(queues = "tasks.queue")
    public void processTask(Task task) {
        System.out.println("Worker 1 processing: " + task.getId());
        // معالجة المهمة
    }
}

@Component
public class Worker2 {
    @RabbitListener(queues = "tasks.queue")
    public void processTask(Task task) {
        System.out.println("Worker 2 processing: " + task.getId());
        // معالجة المهمة
    }
}
```

### 4. نظام الإشعارات (Notification System)
- إشعارات البريد الإلكتروني
- إشعارات SMS
- Push Notifications

```java
// مثال: نظام إشعارات متعدد القنوات
@Service
public class NotificationService {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void sendNotification(Notification notification) {
        // إرسال إلى Exchange من نوع Fanout
        rabbitTemplate.convertAndSend("notifications.fanout", "", notification);
    }
}

// كل قناة لها Consumer خاص
@Component
public class EmailConsumer {
    @RabbitListener(queues = "email.notifications")
    public void sendEmail(Notification notification) {
        emailService.send(notification);
    }
}

@Component
public class SmsConsumer {
    @RabbitListener(queues = "sms.notifications")
    public void sendSms(Notification notification) {
        smsService.send(notification);
    }
}
```

### 5. معالجة الطلبات المؤجلة (Delayed Jobs)
- جدولة المهام
- الرسائل المتأخرة

```java
// استخدام TTL و DLX لتأخير الرسائل
@Configuration
public class DelayedQueueConfig {

    @Bean
    public Queue delayQueue() {
        Map<String, Object> args = new HashMap<>();
        args.put("x-message-ttl", 30000); // 30 ثانية
        args.put("x-dead-letter-exchange", "processing.exchange");
        args.put("x-dead-letter-routing-key", "process");

        return new Queue("delay.queue", true, false, false, args);
    }

    @Bean
    public Queue processingQueue() {
        return new Queue("processing.queue", true);
    }
}
```

### 6. نظام السجلات (Logging System)
- جمع السجلات المركزية
- تصفية السجلات حسب المستوى

```java
// مثال: نظام سجلات بمستويات مختلفة
@Service
public class LogService {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void log(String level, String message) {
        LogMessage logMsg = new LogMessage(level, message, LocalDateTime.now());
        rabbitTemplate.convertAndSend("logs.topic", "log." + level, logMsg);
    }
}

// Consumers لمستويات مختلفة
@Component
public class ErrorLogConsumer {
    @RabbitListener(queues = "error.logs")
    public void handleError(LogMessage log) {
        // معالجة خاصة للأخطاء
        alertService.sendAlert(log);
        saveToDatabase(log);
    }
}
```

## التثبيت والإعداد

### 1. تثبيت RabbitMQ

#### على Windows:

1. تحميل وتثبيت Erlang:
```bash
# تحميل من: https://www.erlang.org/downloads
# تثبيت Erlang/OTP
```

2. تحميل وتثبيت RabbitMQ:
```bash
# تحميل من: https://www.rabbitmq.com/download.html
# تثبيت RabbitMQ Server
```

3. تفعيل Management Plugin:
```bash
cd "C:\Program Files\RabbitMQ Server\rabbitmq_server-3.x.x\sbin"
rabbitmq-plugins enable rabbitmq_management
```

#### على Linux (Ubuntu/Debian):

```bash
# تحديث النظام
sudo apt-get update

# تثبيت Erlang
sudo apt-get install erlang

# إضافة مستودع RabbitMQ
wget -O- https://www.rabbitmq.com/rabbitmq-release-signing-key.asc | sudo apt-key add -
echo "deb https://dl.bintray.com/rabbitmq/debian $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/rabbitmq.list

# تثبيت RabbitMQ
sudo apt-get update
sudo apt-get install rabbitmq-server

# تشغيل الخدمة
sudo systemctl start rabbitmq-server
sudo systemctl enable rabbitmq-server

# تفعيل Management Plugin
sudo rabbitmq-plugins enable rabbitmq_management
```

#### باستخدام Docker:

```bash
# تشغيل RabbitMQ مع Management UI
docker run -d --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  -e RABBITMQ_DEFAULT_USER=admin \
  -e RABBITMQ_DEFAULT_PASS=admin123 \
  rabbitmq:3-management

# الوصول إلى Management UI:
# http://localhost:15672
# Username: admin
# Password: admin123
```

### 2. إعداد Spring Boot مع RabbitMQ

#### إضافة Dependencies في pom.xml:

```xml
<dependencies>
    <!-- Spring Boot Starter for AMQP -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>

    <!-- Spring Boot Starter Web (اختياري) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Lombok (اختياري) -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

#### إعداد application.yml:

```yaml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: admin
    password: admin123
    virtual-host: /

    # إعدادات Publisher Confirms
    publisher-confirm-type: correlated
    publisher-returns: true

    # إعدادات Connection
    connection-timeout: 60000

    # إعدادات Listener
    listener:
      simple:
        acknowledge-mode: manual  # أو auto
        prefetch: 10
        retry:
          enabled: true
          initial-interval: 3000
          max-attempts: 3
          multiplier: 2
        default-requeue-rejected: false

    # إعدادات Template
    template:
      mandatory: true
      retry:
        enabled: true
        initial-interval: 1000
        max-attempts: 3
        multiplier: 2
```

#### إعداد application.properties (بديل):

```properties
# RabbitMQ Configuration
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=admin
spring.rabbitmq.password=admin123
spring.rabbitmq.virtual-host=/

# Publisher Confirms
spring.rabbitmq.publisher-confirm-type=correlated
spring.rabbitmq.publisher-returns=true

# Connection
spring.rabbitmq.connection-timeout=60000

# Listener
spring.rabbitmq.listener.simple.acknowledge-mode=manual
spring.rabbitmq.listener.simple.prefetch=10
spring.rabbitmq.listener.simple.retry.enabled=true
spring.rabbitmq.listener.simple.retry.initial-interval=3000
spring.rabbitmq.listener.simple.retry.max-attempts=3
spring.rabbitmq.listener.simple.default-requeue-rejected=false

# Template
spring.rabbitmq.template.mandatory=true
spring.rabbitmq.template.retry.enabled=true
spring.rabbitmq.template.retry.initial-interval=1000
spring.rabbitmq.template.retry.max-attempts=3
```

### 3. مثال تطبيق Spring Boot كامل

```java
// Application.java
@SpringBootApplication
public class RabbitMQApplication {
    public static void main(String[] args) {
        SpringApplication.run(RabbitMQApplication.class, args);
    }
}

// RabbitMQConfig.java
@Configuration
public class RabbitMQConfig {

    // Queue Declarations
    @Bean
    public Queue userQueue() {
        return new Queue("user.queue", true);
    }

    @Bean
    public Queue orderQueue() {
        return new Queue("order.queue", true);
    }

    // Exchange Declarations
    @Bean
    public DirectExchange directExchange() {
        return new DirectExchange("app.direct", true, false);
    }

    @Bean
    public TopicExchange topicExchange() {
        return new TopicExchange("app.topic", true, false);
    }

    @Bean
    public FanoutExchange fanoutExchange() {
        return new FanoutExchange("app.fanout", true, false);
    }

    // Bindings
    @Bean
    public Binding userBinding() {
        return BindingBuilder
            .bind(userQueue())
            .to(directExchange())
            .with("user");
    }

    @Bean
    public Binding orderBinding() {
        return BindingBuilder
            .bind(orderQueue())
            .to(topicExchange())
            .with("order.#");
    }

    // Message Converter (JSON)
    @Bean
    public MessageConverter jsonMessageConverter() {
        return new Jackson2JsonMessageConverter();
    }

    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
        RabbitTemplate template = new RabbitTemplate(connectionFactory);
        template.setMessageConverter(jsonMessageConverter());
        return template;
    }
}

// Producer Service
@Service
@Slf4j
public class MessageProducer {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void sendUserMessage(User user) {
        try {
            rabbitTemplate.convertAndSend("app.direct", "user", user);
            log.info("User message sent: {}", user);
        } catch (Exception e) {
            log.error("Error sending user message", e);
        }
    }

    public void sendOrderMessage(Order order) {
        try {
            rabbitTemplate.convertAndSend("app.topic", "order.created", order);
            log.info("Order message sent: {}", order);
        } catch (Exception e) {
            log.error("Error sending order message", e);
        }
    }
}

// Consumer Service
@Component
@Slf4j
public class MessageConsumer {

    @RabbitListener(queues = "user.queue")
    public void consumeUserMessage(User user) {
        log.info("Received user message: {}", user);
        // معالجة الرسالة
    }

    @RabbitListener(queues = "order.queue")
    public void consumeOrderMessage(Order order,
                                    @Header(AmqpHeaders.DELIVERY_TAG) long tag,
                                    Channel channel) throws IOException {
        try {
            log.info("Received order message: {}", order);
            // معالجة الرسالة
            processOrder(order);

            // تأكيد استلام الرسالة
            channel.basicAck(tag, false);
        } catch (Exception e) {
            log.error("Error processing order", e);
            // رفض الرسالة وإعادتها للطابور
            channel.basicNack(tag, false, true);
        }
    }

    private void processOrder(Order order) {
        // منطق معالجة الطلب
    }
}

// Models
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Long id;
    private String name;
    private String email;
}

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Order {
    private Long id;
    private String productName;
    private Double price;
    private Integer quantity;
}
```

### 4. إدارة RabbitMQ

#### استخدام Management UI:
- الوصول: `http://localhost:15672`
- المميزات:
  - عرض Queues, Exchanges, Bindings
  - مراقبة معدل الرسائل
  - إنشاء وحذف المكونات
  - عرض الاتصالات النشطة
  - إدارة المستخدمين والصلاحيات

#### استخدام Command Line:

```bash
# عرض قائمة Queues
rabbitmqctl list_queues name messages consumers

# عرض قائمة Exchanges
rabbitmqctl list_exchanges name type

# عرض قائمة Bindings
rabbitmqctl list_bindings

# إضافة مستخدم جديد
rabbitmqctl add_user username password

# تعيين صلاحيات
rabbitmqctl set_permissions -p / username ".*" ".*" ".*"

# تعيين دور المستخدم
rabbitmqctl set_user_tags username administrator

# حذف مستخدم
rabbitmqctl delete_user username

# حالة الخادم
rabbitmqctl status

# إعادة تعيين RabbitMQ
rabbitmqctl reset
```

## الخلاصة

RabbitMQ هو أداة قوية لبناء أنظمة موزعة وقابلة للتوسع. يوفر:
- **الموثوقية**: ضمان عدم فقدان الرسائل
- **المرونة**: أنماط توجيه متعددة
- **قابلية التوسع**: دعم التجميع والتوزيع
- **سهولة الاستخدام**: مكتبات عميلة غنية ودعم متعدد اللغات

من خلال فهم المكونات الأساسية وأنماط الاستخدام، يمكنك بناء أنظمة قوية ومتينة باستخدام RabbitMQ.
