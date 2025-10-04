# أسئلة المقابلات الشخصية - ActiveMQ

## جدول المحتويات
1. [أسئلة أساسية](#أسئلة-أساسية)
2. [JMS (Java Message Service)](#jms-java-message-service)
3. [Queue vs Topic](#queue-vs-topic)
4. [Message Types](#message-types)
5. [Transactions](#transactions)
6. [التكامل مع Spring Boot](#التكامل-مع-spring-boot)
7. [المقارنة](#المقارنة)
8. [الأداء والتحسين](#الأداء-والتحسين)

---

## أسئلة أساسية

### 1. ما هو ActiveMQ؟

**الإجابة:**
ActiveMQ هو Message Broker مفتوح المصدر من Apache يدعم بروتوكول JMS (Java Message Service).

**المميزات الرئيسية:**
- **JMS Compliant**: يلتزم بمعيار JMS
- **Multiple Protocols**: يدعم JMS, AMQP, STOMP, MQTT
- **Cross-Language**: يدعم Java, C++, Python, .NET
- **Reliable**: ضمان توصيل الرسائل
- **Scalable**: يدعم clustering

**المكونات:**
```
Producer → ActiveMQ Broker → Consumer
           (Queue/Topic)
```

**مثال بسيط:**
```java
// Producer
ConnectionFactory factory = new ActiveMQConnectionFactory("tcp://localhost:61616");
Connection connection = factory.createConnection();
connection.start();

Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
Destination destination = session.createQueue("TEST.QUEUE");
MessageProducer producer = session.createProducer(destination);

TextMessage message = session.createTextMessage("مرحباً ActiveMQ!");
producer.send(message);
System.out.println("تم إرسال: " + message.getText());

connection.close();
```

```java
// Consumer
ConnectionFactory factory = new ActiveMQConnectionFactory("tcp://localhost:61616");
Connection connection = factory.createConnection();
connection.start();

Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
Destination destination = session.createQueue("TEST.QUEUE");
MessageConsumer consumer = session.createConsumer(destination);

Message message = consumer.receive(1000);
if (message instanceof TextMessage) {
    TextMessage textMessage = (TextMessage) message;
    System.out.println("تم الاستقبال: " + textMessage.getText());
}

connection.close();
```

---

### 2. ما هي الفوائد الرئيسية لاستخدام ActiveMQ؟

**الإجابة:**

#### 1. Asynchronous Communication
```java
// بدون ActiveMQ - Synchronous
OrderService orderService = new OrderService();
orderService.processOrder(order);        // ينتظر
emailService.sendConfirmation(order);    // ينتظر
smsService.sendSMS(order);              // ينتظر

// مع ActiveMQ - Asynchronous
jmsTemplate.convertAndSend("orders", order);
// لا ينتظر - التنفيذ يكمل فوراً
```

#### 2. Loose Coupling
```java
// Producer لا يعرف من سيستهلك الرسالة
producer.send("order.created", order);

// Consumers متعددون يمكنهم الاستماع
// - Email Service
// - SMS Service
// - Inventory Service
// - Analytics Service
```

#### 3. Reliability
```java
// الرسائل تبقى حتى لو Consumer غير متصل
producer.send(queue, message);

// Persistent messages
Message message = session.createTextMessage("مهم");
producer.send(message, DeliveryMode.PERSISTENT, 4, 0);
```

#### 4. Load Balancing
```java
// عدة Consumers على نفس Queue - التوزيع تلقائي
// Consumer 1, 2, 3 على "payment.queue"
// ActiveMQ يوزع الرسائل بينهم
```

---

## JMS (Java Message Service)

### 3. ما هو JMS؟

**الإجابة:**
JMS (Java Message Service) هو Java API معياري للتعامل مع Message-Oriented Middleware.

**المكونات الرئيسية:**

#### 1. ConnectionFactory
```java
// إنشاء ConnectionFactory
ConnectionFactory factory = new ActiveMQConnectionFactory(
    "tcp://localhost:61616"
);
```

#### 2. Connection
```java
// إنشاء Connection
Connection connection = factory.createConnection();
connection.start();
```

#### 3. Session
```java
// إنشاء Session
Session session = connection.createSession(
    false,                      // transacted
    Session.AUTO_ACKNOWLEDGE    // acknowledgment mode
);
```

#### 4. Destination (Queue أو Topic)
```java
// Queue
Destination queue = session.createQueue("ORDER.QUEUE");

// Topic
Destination topic = session.createTopic("NEWS.TOPIC");
```

#### 5. MessageProducer
```java
MessageProducer producer = session.createProducer(destination);
TextMessage message = session.createTextMessage("Hello");
producer.send(message);
```

#### 6. MessageConsumer
```java
MessageConsumer consumer = session.createConsumer(destination);

// Synchronous receive
Message message = consumer.receive(1000);

// Asynchronous receive
consumer.setMessageListener(message -> {
    System.out.println("Received: " + message);
});
```

---

### 4. ما هي Acknowledgment Modes في JMS؟

**الإجابة:**

#### 1. AUTO_ACKNOWLEDGE
تأكيد تلقائي فور استقبال الرسالة.

```java
Session session = connection.createSession(
    false,
    Session.AUTO_ACKNOWLEDGE
);

MessageConsumer consumer = session.createConsumer(queue);
consumer.setMessageListener(message -> {
    // تم التأكيد تلقائياً
    process(message);
    // إذا فشلت المعالجة، الرسالة ضاعت!
});
```

**المشاكل:**
- ❌ إذا فشلت المعالجة، الرسالة تضيع

#### 2. CLIENT_ACKNOWLEDGE
تأكيد يدوي من Consumer.

```java
Session session = connection.createSession(
    false,
    Session.CLIENT_ACKNOWLEDGE
);

MessageConsumer consumer = session.createConsumer(queue);
consumer.setMessageListener(message -> {
    try {
        process(message);
        message.acknowledge(); // تأكيد يدوي
    } catch (Exception e) {
        // عدم التأكيد = إعادة التوصيل
    }
});
```

**المميزات:**
- ✅ ضمان عدم فقدان الرسائل
- ✅ إعادة المحاولة عند الفشل

#### 3. DUPS_OK_ACKNOWLEDGE
تأكيد كسول - قد تصل الرسالة مرتين.

```java
Session session = connection.createSession(
    false,
    Session.DUPS_OK_ACKNOWLEDGE
);
// أداء أفضل لكن احتمال تكرار الرسائل
```

**الاستخدام:**
- ✅ رسائل غير حرجة
- ✅ idempotent operations

#### 4. SESSION_TRANSACTED
مع Transactions.

```java
Session session = connection.createSession(
    true,  // transacted
    Session.SESSION_TRANSACTED
);

try {
    Message msg1 = consumer.receive();
    process(msg1);

    Message msg2 = consumer.receive();
    process(msg2);

    session.commit(); // تأكيد جميع الرسائل
} catch (Exception e) {
    session.rollback(); // إلغاء جميع الرسائل
}
```

---

## Queue vs Topic

### 5. ما الفرق بين Queue و Topic؟

**الإجابة:**

#### Queue (Point-to-Point)
رسالة واحدة تذهب لـ Consumer واحد فقط.

```
Producer → Queue → Consumer 1 ✓
                 → Consumer 2 (لن يستقبل نفس الرسالة)
                 → Consumer 3 (لن يستقبل نفس الرسالة)
```

**مثال:**
```java
// Producer
Destination queue = session.createQueue("ORDERS");
MessageProducer producer = session.createProducer(queue);
producer.send(session.createTextMessage("Order #123"));

// Consumers (واحد فقط سيستقبل)
// Consumer 1
MessageConsumer consumer1 = session.createConsumer(queue);
consumer1.setMessageListener(msg -> {
    System.out.println("Consumer 1 received: " + msg);
});

// Consumer 2
MessageConsumer consumer2 = session.createConsumer(queue);
consumer2.setMessageListener(msg -> {
    System.out.println("Consumer 2 received: " + msg);
});

// نتيجة: Consumer 1 أو Consumer 2 سيستقبل، ليس كلاهما
```

**الاستخدامات:**
- Task queues
- Load balancing
- Work distribution

---

#### Topic (Publish-Subscribe)
رسالة واحدة تذهب لجميع Subscribers.

```
Producer → Topic → Subscriber 1 ✓
                 → Subscriber 2 ✓
                 → Subscriber 3 ✓
```

**مثال:**
```java
// Publisher
Destination topic = session.createTopic("NEWS");
MessageProducer producer = session.createProducer(topic);
producer.send(session.createTextMessage("أخبار عاجلة!"));

// Subscribers (جميعهم سيستقبلون)
// Subscriber 1 (Email Service)
MessageConsumer subscriber1 = session.createConsumer(topic);
subscriber1.setMessageListener(msg -> {
    sendEmail(msg);
});

// Subscriber 2 (SMS Service)
MessageConsumer subscriber2 = session.createConsumer(topic);
subscriber2.setMessageListener(msg -> {
    sendSMS(msg);
});

// Subscriber 3 (Push Notification)
MessageConsumer subscriber3 = session.createConsumer(topic);
subscriber3.setMessageListener(msg -> {
    sendPushNotification(msg);
});

// نتيجة: جميع Subscribers سيستقبلون نفس الرسالة
```

**الاستخدامات:**
- Notifications
- Broadcasting
- Event propagation

---

#### مقارنة شاملة

| المقارنة | Queue | Topic |
|----------|-------|-------|
| **النموذج** | Point-to-Point | Publish-Subscribe |
| **عدد المستقبلين** | واحد فقط | الجميع |
| **التخزين** | حتى الاستهلاك | لا يخزن (إلا Durable) |
| **Load Balancing** | نعم | لا |
| **الاستخدام** | Task processing | Broadcasting |

---

### 6. ما هو Durable Subscription؟

**الإجابة:**
Durable Subscription يسمح لـ Subscriber باستقبال الرسائل حتى لو كان غير متصل وقت الإرسال.

#### Non-Durable (Default)
```java
// Non-durable subscriber
Topic topic = session.createTopic("NEWS");
MessageConsumer consumer = session.createConsumer(topic);
// إذا غير متصل، الرسائل تضيع
```

```
Publisher sends at 10:00 AM
                ↓
Subscriber offline → Message lost ❌
Subscriber online at 10:05 AM → Too late
```

#### Durable Subscription
```java
// Durable subscriber
Topic topic = session.createTopic("NEWS");
MessageConsumer consumer = session.createDurableSubscriber(
    topic,
    "NewsSubscriber1"  // Subscription name (فريد)
);
// الرسائل تخزن حتى لو Subscriber غير متصل
```

```
Publisher sends at 10:00 AM
                ↓
Subscriber offline → Message stored ✓
Subscriber online at 10:05 AM → Receives message
```

**مثال كامل:**
```java
// 1. Durable Subscriber
Connection connection = factory.createConnection();
connection.setClientID("Client1"); // مطلوب لـ Durable
connection.start();

Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
Topic topic = session.createTopic("STOCK.PRICES");

MessageConsumer consumer = session.createDurableSubscriber(
    topic,
    "StockPriceSubscriber"
);

consumer.setMessageListener(message -> {
    System.out.println("Received: " + message);
});

// 2. Unsubscribe (عند عدم الحاجة)
consumer.close();
session.unsubscribe("StockPriceSubscriber");
```

**متى تستخدمه:**
- ✅ رسائل مهمة لا يجب أن تضيع
- ✅ Subscribers قد يكونون offline أحياناً
- ❌ استهلاك مساحة تخزين

---

## Message Types

### 7. ما هي أنواع الرسائل في JMS؟

**الإجابة:**

#### 1. TextMessage
رسالة نصية بسيطة.

```java
TextMessage message = session.createTextMessage("مرحباً");
producer.send(message);

// Consumer
TextMessage received = (TextMessage) consumer.receive();
String text = received.getText();
```

#### 2. ObjectMessage
كائن Java قابل للتسلسل (Serializable).

```java
// إنشاء Object
Order order = new Order("123", "Customer1", 100.0);

// إرسال
ObjectMessage message = session.createObjectMessage(order);
producer.send(message);

// استقبال
ObjectMessage received = (ObjectMessage) consumer.receive();
Order receivedOrder = (Order) received.getObject();
```

**تحذير:** مشاكل أمنية محتملة مع Serialization.

#### 3. MapMessage
Key-Value pairs.

```java
// إرسال
MapMessage message = session.createMapMessage();
message.setString("name", "أحمد");
message.setInt("age", 25);
message.setDouble("salary", 5000.0);
producer.send(message);

// استقبال
MapMessage received = (MapMessage) consumer.receive();
String name = received.getString("name");
int age = received.getInt("age");
double salary = received.getDouble("salary");
```

#### 4. BytesMessage
بيانات byte[] خام.

```java
// إرسال
byte[] data = "مرحباً".getBytes("UTF-8");
BytesMessage message = session.createBytesMessage();
message.writeBytes(data);
producer.send(message);

// استقبال
BytesMessage received = (BytesMessage) consumer.receive();
byte[] receivedData = new byte[(int) received.getBodyLength()];
received.readBytes(receivedData);
String text = new String(receivedData, "UTF-8");
```

#### 5. StreamMessage
قراءة/كتابة تسلسلية.

```java
// إرسال
StreamMessage message = session.createStreamMessage();
message.writeString("أحمد");
message.writeInt(25);
message.writeDouble(5000.0);
producer.send(message);

// استقبال (يجب القراءة بنفس الترتيب)
StreamMessage received = (StreamMessage) consumer.receive();
String name = received.readString();
int age = received.readInt();
double salary = received.readDouble();
```

---

### 8. كيف تضيف Properties للرسائل؟

**الإجابة:**
Properties تستخدم للـ Metadata و Message Selection.

```java
// إنشاء رسالة مع Properties
TextMessage message = session.createTextMessage("Order #123");

// String property
message.setStringProperty("orderType", "premium");

// Int property
message.setIntProperty("priority", 5);

// Boolean property
message.setBooleanProperty("urgent", true);

// Standard JMS properties
message.setJMSPriority(9);
message.setJMSExpiration(System.currentTimeMillis() + 60000); // 1 minute

producer.send(message);
```

```java
// Consumer - قراءة Properties
Message received = consumer.receive();
String orderType = received.getStringProperty("orderType");
int priority = received.getIntProperty("priority");
boolean urgent = received.getBooleanProperty("urgent");

System.out.println("Type: " + orderType);
System.out.println("Priority: " + priority);
System.out.println("Urgent: " + urgent);
```

**Message Selectors:**
```java
// Consumer يستقبل رسائل محددة فقط
MessageConsumer consumer = session.createConsumer(
    queue,
    "priority > 5 AND orderType = 'premium'"
);

// سيستقبل فقط الرسائل التي تطابق الشرط
```

**أمثلة Selectors:**
```java
// Priority
"JMSPriority > 5"

// String matching
"orderType = 'premium'"

// Numeric comparison
"amount >= 1000"

// Boolean
"urgent = true"

// Combined
"priority > 5 AND orderType = 'premium' AND amount >= 1000"

// IN operator
"status IN ('pending', 'processing')"

// LIKE operator
"customerName LIKE 'A%'"

// IS NULL
"description IS NULL"
```

---

## Transactions

### 9. كيف تستخدم Transactions في ActiveMQ؟

**الإجابة:**

#### JMS Transactions
```java
// إنشاء Transacted Session
Session session = connection.createSession(
    true,  // transacted = true
    Session.SESSION_TRANSACTED
);

MessageProducer producer = session.createProducer(queue);
MessageConsumer consumer = session.createConsumer(queue);

try {
    // إرسال رسائل متعددة
    producer.send(session.createTextMessage("Message 1"));
    producer.send(session.createTextMessage("Message 2"));
    producer.send(session.createTextMessage("Message 3"));

    // استقبال ومعالجة
    Message msg = consumer.receive();
    process(msg);

    // Commit - تأكيد جميع العمليات
    session.commit();

} catch (Exception e) {
    // Rollback - إلغاء جميع العمليات
    session.rollback();
}
```

**مثال - نقل أموال:**
```java
Session session = connection.createSession(true, Session.SESSION_TRANSACTED);

try {
    // 1. سحب من حساب
    TextMessage withdrawMsg = session.createTextMessage(
        "WITHDRAW:1000:AccountA"
    );
    producer.send(withdrawQueue, withdrawMsg);

    // 2. إيداع في حساب آخر
    TextMessage depositMsg = session.createTextMessage(
        "DEPOSIT:1000:AccountB"
    );
    producer.send(depositQueue, depositMsg);

    // 3. Commit كلاهما معاً
    session.commit();
    System.out.println("تم النقل بنجاح");

} catch (Exception e) {
    // Rollback - إلغاء كلا العمليتين
    session.rollback();
    System.out.println("فشل النقل - تم الإلغاء");
}
```

---

### 10. ما الفرق بين Local و XA Transactions؟

**الإجابة:**

#### Local Transactions
Transaction ضمن JMS فقط.

```java
// Local transaction - JMS فقط
Session session = connection.createSession(true, Session.SESSION_TRANSACTED);

producer.send(message1);
producer.send(message2);
session.commit(); // Commit JMS فقط
```

#### XA Transactions (Distributed)
Transaction تشمل JMS + Database + موارد أخرى.

```java
// XA Transaction - JMS + Database
@Transactional
public void processOrder(Order order) {
    // 1. حفظ في Database
    orderRepository.save(order);

    // 2. إرسال JMS message
    jmsTemplate.convertAndSend("orders", order);

    // إذا فشل أي منهما، كلاهما يُلغى
}
```

**Spring Boot مع XA:**
```java
@Configuration
public class XAConfig {

    @Bean
    public JtaTransactionManager transactionManager() {
        return new JtaTransactionManager();
    }

    @Bean
    public ActiveMQXAConnectionFactory connectionFactory() {
        return new ActiveMQXAConnectionFactory("tcp://localhost:61616");
    }
}

@Service
public class OrderService {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Autowired
    private JmsTemplate jmsTemplate;

    @Transactional // XA transaction
    public void createOrder(Order order) {
        // Database operation
        jdbcTemplate.update(
            "INSERT INTO orders VALUES (?, ?)",
            order.getId(), order.getAmount()
        );

        // JMS operation
        jmsTemplate.convertAndSend("order.created", order);

        // كلاهما ينجح أو كلاهما يفشل
    }
}
```

---

## التكامل مع Spring Boot

### 11. كيف تستخدم ActiveMQ مع Spring Boot؟

**الإجابة:**

#### 1. Dependencies
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-activemq</artifactId>
</dependency>
```

#### 2. Configuration
```yaml
# application.yml
spring:
  activemq:
    broker-url: tcp://localhost:61616
    user: admin
    password: admin
    packages:
      trust-all: false
      trusted: com.example.model
    pool:
      enabled: true
      max-connections: 10
```

#### 3. Configuration Class
```java
@Configuration
@EnableJms
public class ActiveMQConfig {

    @Bean
    public JmsListenerContainerFactory<?> jmsListenerContainerFactory(
        ConnectionFactory connectionFactory
    ) {
        DefaultJmsListenerContainerFactory factory =
            new DefaultJmsListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory);
        factory.setConcurrency("3-10"); // 3 min, 10 max consumers
        factory.setSessionTransacted(true);
        return factory;
    }

    @Bean
    public MessageConverter jacksonJmsMessageConverter() {
        MappingJackson2MessageConverter converter =
            new MappingJackson2MessageConverter();
        converter.setTargetType(MessageType.TEXT);
        converter.setTypeIdPropertyName("_type");
        return converter;
    }
}
```

#### 4. Producer
```java
@Service
public class OrderProducer {

    @Autowired
    private JmsTemplate jmsTemplate;

    // إرسال String
    public void sendMessage(String message) {
        jmsTemplate.convertAndSend("TEST.QUEUE", message);
    }

    // إرسال Object
    public void sendOrder(Order order) {
        jmsTemplate.convertAndSend("ORDER.QUEUE", order);
    }

    // إرسال مع Priority
    public void sendUrgentOrder(Order order) {
        jmsTemplate.convertAndSend("ORDER.QUEUE", order, message -> {
            message.setJMSPriority(9);
            return message;
        });
    }

    // إرسال لـ Topic
    public void publishNews(String news) {
        jmsTemplate.setPubSubDomain(true); // Topic mode
        jmsTemplate.convertAndSend("NEWS.TOPIC", news);
        jmsTemplate.setPubSubDomain(false); // عودة لـ Queue mode
    }
}
```

#### 5. Consumer
```java
@Service
public class OrderConsumer {

    // استماع بسيط
    @JmsListener(destination = "ORDER.QUEUE")
    public void receiveOrder(Order order) {
        System.out.println("Received order: " + order.getId());
        processOrder(order);
    }

    // مع Message object
    @JmsListener(destination = "ORDER.QUEUE")
    public void receiveOrderWithMessage(Message message) throws JMSException {
        Order order = (Order) ((ObjectMessage) message).getObject();
        String priority = message.getStringProperty("priority");
        System.out.println("Order: " + order.getId() + ", Priority: " + priority);
    }

    // Message Selector
    @JmsListener(
        destination = "ORDER.QUEUE",
        selector = "priority > 5"
    )
    public void receiveHighPriorityOrders(Order order) {
        System.out.println("High priority order: " + order.getId());
    }

    // استماع من Topic
    @JmsListener(destination = "NEWS.TOPIC", containerFactory = "topicListenerFactory")
    public void receiveNews(String news) {
        System.out.println("News: " + news);
    }

    // مع Exception handling
    @JmsListener(destination = "ORDER.QUEUE")
    public void receiveOrderWithRetry(Order order) {
        try {
            processOrder(order);
        } catch (Exception e) {
            log.error("خطأ في معالجة الطلب", e);
            throw new RuntimeException(e); // سيعاد إرسال الرسالة
        }
    }
}
```

#### 6. مثال كامل - E-Commerce

```java
// Model
@Data
public class Order {
    private String id;
    private String customerId;
    private List<OrderItem> items;
    private BigDecimal total;
    private OrderStatus status;
}

// Producer
@Service
public class OrderService {

    @Autowired
    private JmsTemplate jmsTemplate;

    @Autowired
    private OrderRepository orderRepository;

    @Transactional
    public void createOrder(Order order) {
        // 1. حفظ في Database
        order.setStatus(OrderStatus.PENDING);
        orderRepository.save(order);

        // 2. إرسال لمعالجة الدفع
        jmsTemplate.convertAndSend("payment.queue", order);

        log.info("تم إنشاء طلب: {}", order.getId());
    }
}

// Payment Consumer
@Service
public class PaymentConsumer {

    @Autowired
    private PaymentGateway paymentGateway;

    @Autowired
    private JmsTemplate jmsTemplate;

    @JmsListener(destination = "payment.queue")
    @Transactional
    public void processPayment(Order order) {
        try {
            PaymentResult result = paymentGateway.charge(order);

            if (result.isSuccess()) {
                order.setStatus(OrderStatus.PAID);
                // إرسال لـ Shipping
                jmsTemplate.convertAndSend("shipping.queue", order);
                log.info("تم الدفع للطلب: {}", order.getId());
            } else {
                order.setStatus(OrderStatus.PAYMENT_FAILED);
                // إرسال لـ Failed queue
                jmsTemplate.convertAndSend("payment.failed", order);
            }

            orderRepository.save(order);

        } catch (Exception e) {
            log.error("خطأ في معالجة الدفع", e);
            throw new RuntimeException(e);
        }
    }
}

// Shipping Consumer
@Service
public class ShippingConsumer {

    @Autowired
    private JmsTemplate jmsTemplate;

    @JmsListener(destination = "shipping.queue")
    public void processShipping(Order order) {
        log.info("بدء الشحن للطلب: {}", order.getId());

        Shipment shipment = createShipment(order);
        order.setStatus(OrderStatus.SHIPPED);
        orderRepository.save(order);

        // إرسال إشعار للعميل
        Notification notification = new Notification(
            order.getCustomerId(),
            "تم شحن طلبك: " + order.getId()
        );
        jmsTemplate.convertAndSend("notifications.topic", notification);
    }
}

// Notification Consumer (Topic - multiple subscribers)
@Service
public class EmailNotificationConsumer {

    @JmsListener(destination = "notifications.topic", containerFactory = "topicListenerFactory")
    public void sendEmail(Notification notification) {
        emailService.send(notification);
        log.info("تم إرسال بريد إلكتروني");
    }
}

@Service
public class SmsNotificationConsumer {

    @JmsListener(destination = "notifications.topic", containerFactory = "topicListenerFactory")
    public void sendSms(Notification notification) {
        smsService.send(notification);
        log.info("تم إرسال SMS");
    }
}
```

---

## المقارنة

### 12. ActiveMQ vs RabbitMQ - أيهما أفضل؟

**الإجابة:**

| المقارنة | ActiveMQ | RabbitMQ |
|----------|----------|----------|
| **البروتوكول** | JMS (أساسي) | AMQP (أساسي) |
| **اللغة** | Java | Erlang |
| **الأداء** | جيد | ممتاز |
| **Routing** | بسيط | متقدم (4 Exchange types) |
| **التكامل** | قوي مع Java/Spring | متعدد اللغات |
| **Management UI** | بسيط | متقدم |
| **الشعبية** | متوسطة | عالية |
| **الاستخدام** | Java applications | Microservices |

**متى تستخدم ActiveMQ:**
- ✅ تطبيق Java بالكامل
- ✅ تحتاج JMS standard
- ✅ تكامل بسيط مع Spring
- ✅ بيئة enterprise

**متى تستخدم RabbitMQ:**
- ✅ Microservices متعددة اللغات
- ✅ Routing معقد
- ✅ أداء عالي
- ✅ مرونة في التوزيع

---

### 13. ActiveMQ vs Apache Kafka؟

**الإجابة:**

| المقارنة | ActiveMQ | Apache Kafka |
|----------|----------|--------------|
| **النوع** | Message Broker | Event Streaming Platform |
| **النموذج** | Push | Pull |
| **التخزين** | Queue/Topic | Log (persistent) |
| **الترتيب** | في Queue | في Partition |
| **الأداء** | آلاف/ثانية | ملايين/ثانية |
| **الاستخدام** | Messaging | Event Streaming |
| **Retention** | حتى الاستهلاك | مدة محددة (أيام) |

**متى تستخدم ActiveMQ:**
- ✅ Traditional messaging
- ✅ Request-Reply patterns
- ✅ Transactional messaging
- ✅ JMS compliance

**متى تستخدم Kafka:**
- ✅ Event streaming
- ✅ Log aggregation
- ✅ Big data processing
- ✅ Real-time analytics
- ✅ High throughput

---

## الأداء والتحسين

### 14. كيف تحسن أداء ActiveMQ؟

**الإجابة:**

#### 1. Connection Pooling
```java
@Bean
public PooledConnectionFactory pooledConnectionFactory() {
    PooledConnectionFactory factory = new PooledConnectionFactory();
    factory.setConnectionFactory(activeMQConnectionFactory());
    factory.setMaxConnections(10);
    return factory;
}
```

#### 2. Prefetch Limit
```java
// تحديد عدد الرسائل التي يحصل عليها Consumer مسبقاً
String url = "tcp://localhost:61616?jms.prefetchPolicy.queuePrefetch=10";
ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(url);
```

```yaml
# Spring Boot
spring:
  activemq:
    broker-url: tcp://localhost:61616?jms.prefetchPolicy.all=10
```

#### 3. Async Send
```java
// إرسال asynchronous (لا ينتظر التأكيد)
ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory();
factory.setUseAsyncSend(true);

// أو
jmsTemplate.setExplicitQosEnabled(true);
jmsTemplate.setDeliveryPersistent(false); // Non-persistent = أسرع
```

#### 4. Message Compression
```java
String url = "tcp://localhost:61616?jms.useCompression=true";
```

#### 5. Batch Processing
```java
@JmsListener(destination = "orders")
public void processBatch(List<Order> orders) {
    // معالجة دفعة واحدة بدلاً من رسالة واحدة
    orderService.processBatch(orders);
}
```

---

### 15. كيف تراقب ActiveMQ؟

**الإجابة:**

#### 1. Web Console
```
http://localhost:8161/admin
```

**المعلومات المتاحة:**
- عدد الرسائل في Queues/Topics
- عدد Consumers/Producers
- Memory usage
- Throughput

#### 2. JMX Monitoring
```java
@Bean
public MBeanServerConnection jmxConnection() throws Exception {
    String url = "service:jmx:rmi:///jndi/rmi://localhost:1099/jmxrmi";
    JMXServiceURL jmxUrl = new JMXServiceURL(url);
    JMXConnector connector = JMXConnectorFactory.connect(jmxUrl);
    return connector.getMBeanServerConnection();
}

// الحصول على Queue size
ObjectName queueName = new ObjectName(
    "org.apache.activemq:type=Broker,brokerName=localhost," +
    "destinationType=Queue,destinationName=orders"
);
Long queueSize = (Long) connection.getAttribute(queueName, "QueueSize");
```

#### 3. Spring Boot Actuator
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,metrics,activemq
```

```java
@Component
public class ActiveMQHealthIndicator implements HealthIndicator {

    @Autowired
    private ConnectionFactory connectionFactory;

    @Override
    public Health health() {
        try {
            Connection connection = connectionFactory.createConnection();
            connection.start();
            connection.close();
            return Health.up().build();
        } catch (Exception e) {
            return Health.down(e).build();
        }
    }
}
```

---

### 16. ما هي أفضل الممارسات لـ ActiveMQ؟

**الإجابة:**

#### 1. استخدم Message Selectors
```java
// بدلاً من استقبال جميع الرسائل وفلترتها
@JmsListener(destination = "orders")
public void processAll(Order order) {
    if (order.getPriority() > 5) {
        processHighPriority(order);
    }
}

// استخدم Selector
@JmsListener(
    destination = "orders",
    selector = "priority > 5"
)
public void processHighPriority(Order order) {
    // يستقبل فقط الرسائل ذات الأولوية العالية
}
```

#### 2. استخدم CLIENT_ACKNOWLEDGE
```java
@JmsListener(destination = "orders")
public void process(Order order, Session session, Message message) throws JMSException {
    try {
        processOrder(order);
        message.acknowledge();
    } catch (Exception e) {
        session.recover(); // إعادة الرسالة
    }
}
```

#### 3. استخدم Dead Letter Queue
```xml
<!-- activemq.xml -->
<policyEntry queue="orders">
    <deadLetterStrategy>
        <individualDeadLetterStrategy queuePrefix="DLQ." useQueueForQueueMessages="true"/>
    </deadLetterStrategy>
</policyEntry>
```

#### 4. Set Message TTL
```java
jmsTemplate.convertAndSend("orders", order, message -> {
    message.setJMSExpiration(System.currentTimeMillis() + 60000); // 1 minute
    return message;
});
```

#### 5. Monitor Performance
```java
@Scheduled(fixedRate = 60000)
public void monitorQueues() {
    long queueSize = getQueueSize("orders");
    if (queueSize > 10000) {
        alertAdmin("Queue 'orders' has " + queueSize + " messages!");
    }
}
```

---

## خاتمة

### أسئلة سريعة للمراجعة:

1. **ما الفرق بين Queue و Topic؟**
   - Queue: رسالة لـ consumer واحد
   - Topic: رسالة لجميع subscribers

2. **ما هي Acknowledgment Modes؟**
   - AUTO_ACKNOWLEDGE, CLIENT_ACKNOWLEDGE, DUPS_OK_ACKNOWLEDGE, SESSION_TRANSACTED

3. **متى تستخدم Durable Subscription؟**
   - عندما تريد استقبال الرسائل حتى لو كنت offline

4. **ما هو JMS؟**
   - Java API معياري للمراسلة

5. **ActiveMQ vs RabbitMQ؟**
   - ActiveMQ: قوي مع Java/JMS
   - RabbitMQ: أداء أعلى، routing أفضل

6. **كيف تحسن الأداء؟**
   - Connection pooling
   - Prefetch limit
   - Async send
   - Batch processing

---

**موارد إضافية:**
- [ActiveMQ Official Docs](https://activemq.apache.org/documentation)
- [JMS Specification](https://javaee.github.io/jms-spec/)
- [Spring JMS Documentation](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms)
