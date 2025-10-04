# ActiveMQ Interview Questions

## Table of Contents
1. [Basic Questions](#basic-questions)
2. [JMS (Java Message Service)](#jms-java-message-service)
3. [Queue vs Topic](#queue-vs-topic)
4. [Message Types](#message-types)
5. [Transactions](#transactions)
6. [Spring Boot Integration](#spring-boot-integration)
7. [Comparison](#comparison)
8. [Performance and Optimization](#performance-and-optimization)

---

## Basic Questions

### 1. What is ActiveMQ?

**Answer:**
ActiveMQ is an open-source message broker from Apache that implements the JMS (Java Message Service) protocol.

**Key Features:**
- **JMS Compliant**: Adheres to JMS standard
- **Multiple Protocols**: Supports JMS, AMQP, STOMP, MQTT
- **Cross-Language**: Supports Java, C++, Python, .NET
- **Reliable**: Guaranteed message delivery
- **Scalable**: Supports clustering

**Components:**
```
Producer → ActiveMQ Broker → Consumer
           (Queue/Topic)
```

**Simple Example:**
```java
// Producer
ConnectionFactory factory = new ActiveMQConnectionFactory("tcp://localhost:61616");
Connection connection = factory.createConnection();
connection.start();

Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
Destination destination = session.createQueue("TEST.QUEUE");
MessageProducer producer = session.createProducer(destination);

TextMessage message = session.createTextMessage("Hello ActiveMQ!");
producer.send(message);
System.out.println("Sent: " + message.getText());

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
    System.out.println("Received: " + textMessage.getText());
}

connection.close();
```

---

### 2. What are the main benefits of using ActiveMQ?

**Answer:**

#### 1. Asynchronous Communication
```java
// Without ActiveMQ - Synchronous
OrderService orderService = new OrderService();
orderService.processOrder(order);        // waits
emailService.sendConfirmation(order);    // waits
smsService.sendSMS(order);              // waits

// With ActiveMQ - Asynchronous
jmsTemplate.convertAndSend("orders", order);
// doesn't wait - execution continues immediately
```

#### 2. Loose Coupling
```java
// Producer doesn't know who will consume the message
producer.send("order.created", order);

// Multiple Consumers can listen
// - Email Service
// - SMS Service
// - Inventory Service
// - Analytics Service
```

#### 3. Reliability
```java
// Messages persist even if Consumer is offline
producer.send(queue, message);

// Persistent messages
Message message = session.createTextMessage("Important");
producer.send(message, DeliveryMode.PERSISTENT, 4, 0);
```

#### 4. Load Balancing
```java
// Multiple Consumers on same Queue - automatic distribution
// Consumer 1, 2, 3 on "payment.queue"
// ActiveMQ distributes messages among them
```

---

## JMS (Java Message Service)

### 3. What is JMS?

**Answer:**
JMS (Java Message Service) is a standard Java API for working with Message-Oriented Middleware.

**Core Components:**

#### 1. ConnectionFactory
```java
// Create ConnectionFactory
ConnectionFactory factory = new ActiveMQConnectionFactory(
    "tcp://localhost:61616"
);
```

#### 2. Connection
```java
// Create Connection
Connection connection = factory.createConnection();
connection.start();
```

#### 3. Session
```java
// Create Session
Session session = connection.createSession(
    false,                      // transacted
    Session.AUTO_ACKNOWLEDGE    // acknowledgment mode
);
```

#### 4. Destination (Queue or Topic)
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

### 4. What are the Acknowledgment Modes in JMS?

**Answer:**

#### 1. AUTO_ACKNOWLEDGE
Automatic acknowledgment upon message receipt.

```java
Session session = connection.createSession(
    false,
    Session.AUTO_ACKNOWLEDGE
);

MessageConsumer consumer = session.createConsumer(queue);
consumer.setMessageListener(message -> {
    // Acknowledged automatically
    process(message);
    // If processing fails, message is lost!
});
```

**Issues:**
- ❌ If processing fails, message is lost

#### 2. CLIENT_ACKNOWLEDGE
Manual acknowledgment from Consumer.

```java
Session session = connection.createSession(
    false,
    Session.CLIENT_ACKNOWLEDGE
);

MessageConsumer consumer = session.createConsumer(queue);
consumer.setMessageListener(message -> {
    try {
        process(message);
        message.acknowledge(); // Manual acknowledgment
    } catch (Exception e) {
        // No acknowledgment = redelivery
    }
});
```

**Benefits:**
- ✅ Guarantee no message loss
- ✅ Retry on failure

#### 3. DUPS_OK_ACKNOWLEDGE
Lazy acknowledgment - message may arrive twice.

```java
Session session = connection.createSession(
    false,
    Session.DUPS_OK_ACKNOWLEDGE
);
// Better performance but possible duplicate messages
```

**Use Case:**
- ✅ Non-critical messages
- ✅ Idempotent operations

#### 4. SESSION_TRANSACTED
With Transactions.

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

    session.commit(); // Acknowledge all messages
} catch (Exception e) {
    session.rollback(); // Reject all messages
}
```

---

## Queue vs Topic

### 5. What's the difference between Queue and Topic?

**Answer:**

#### Queue (Point-to-Point)
One message goes to only one Consumer.

```
Producer → Queue → Consumer 1 ✓
                 → Consumer 2 (won't receive same message)
                 → Consumer 3 (won't receive same message)
```

**Example:**
```java
// Producer
Destination queue = session.createQueue("ORDERS");
MessageProducer producer = session.createProducer(queue);
producer.send(session.createTextMessage("Order #123"));

// Consumers (only one will receive)
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

// Result: Either Consumer 1 OR Consumer 2 receives, not both
```

**Use Cases:**
- Task queues
- Load balancing
- Work distribution

---

#### Topic (Publish-Subscribe)
One message goes to all Subscribers.

```
Producer → Topic → Subscriber 1 ✓
                 → Subscriber 2 ✓
                 → Subscriber 3 ✓
```

**Example:**
```java
// Publisher
Destination topic = session.createTopic("NEWS");
MessageProducer producer = session.createProducer(topic);
producer.send(session.createTextMessage("Breaking news!"));

// Subscribers (all will receive)
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

// Result: All Subscribers receive the same message
```

**Use Cases:**
- Notifications
- Broadcasting
- Event propagation

---

#### Comprehensive Comparison

| Comparison | Queue | Topic |
|------------|-------|-------|
| **Model** | Point-to-Point | Publish-Subscribe |
| **# of Receivers** | One only | All |
| **Storage** | Until consumption | Not stored (except Durable) |
| **Load Balancing** | Yes | No |
| **Use Case** | Task processing | Broadcasting |

---

### 6. What is Durable Subscription?

**Answer:**
Durable Subscription allows Subscriber to receive messages even if offline when published.

#### Non-Durable (Default)
```java
// Non-durable subscriber
Topic topic = session.createTopic("NEWS");
MessageConsumer consumer = session.createConsumer(topic);
// If offline, messages are lost
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
    "NewsSubscriber1"  // Subscription name (unique)
);
// Messages stored even if Subscriber offline
```

```
Publisher sends at 10:00 AM
                ↓
Subscriber offline → Message stored ✓
Subscriber online at 10:05 AM → Receives message
```

**Complete Example:**
```java
// 1. Durable Subscriber
Connection connection = factory.createConnection();
connection.setClientID("Client1"); // Required for Durable
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

// 2. Unsubscribe (when no longer needed)
consumer.close();
session.unsubscribe("StockPriceSubscriber");
```

**When to use:**
- ✅ Important messages that shouldn't be lost
- ✅ Subscribers may be offline sometimes
- ❌ Consumes storage space

---

## Message Types

### 7. What are the message types in JMS?

**Answer:**

#### 1. TextMessage
Simple text message.

```java
TextMessage message = session.createTextMessage("Hello");
producer.send(message);

// Consumer
TextMessage received = (TextMessage) consumer.receive();
String text = received.getText();
```

#### 2. ObjectMessage
Serializable Java object.

```java
// Create Object
Order order = new Order("123", "Customer1", 100.0);

// Send
ObjectMessage message = session.createObjectMessage(order);
producer.send(message);

// Receive
ObjectMessage received = (ObjectMessage) consumer.receive();
Order receivedOrder = (Order) received.getObject();
```

**Warning:** Potential security issues with Serialization.

#### 3. MapMessage
Key-Value pairs.

```java
// Send
MapMessage message = session.createMapMessage();
message.setString("name", "John");
message.setInt("age", 25);
message.setDouble("salary", 5000.0);
producer.send(message);

// Receive
MapMessage received = (MapMessage) consumer.receive();
String name = received.getString("name");
int age = received.getInt("age");
double salary = received.getDouble("salary");
```

#### 4. BytesMessage
Raw byte[] data.

```java
// Send
byte[] data = "Hello".getBytes("UTF-8");
BytesMessage message = session.createBytesMessage();
message.writeBytes(data);
producer.send(message);

// Receive
BytesMessage received = (BytesMessage) consumer.receive();
byte[] receivedData = new byte[(int) received.getBodyLength()];
received.readBytes(receivedData);
String text = new String(receivedData, "UTF-8");
```

#### 5. StreamMessage
Sequential read/write.

```java
// Send
StreamMessage message = session.createStreamMessage();
message.writeString("John");
message.writeInt(25);
message.writeDouble(5000.0);
producer.send(message);

// Receive (must read in same order)
StreamMessage received = (StreamMessage) consumer.receive();
String name = received.readString();
int age = received.readInt();
double salary = received.readDouble();
```

---

### 8. How to add Properties to messages?

**Answer:**
Properties are used for Metadata and Message Selection.

```java
// Create message with Properties
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
// Consumer - read Properties
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
// Consumer receives only specific messages
MessageConsumer consumer = session.createConsumer(
    queue,
    "priority > 5 AND orderType = 'premium'"
);

// Will receive only messages matching the condition
```

**Selector Examples:**
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

### 9. How to use Transactions in ActiveMQ?

**Answer:**

#### JMS Transactions
```java
// Create Transacted Session
Session session = connection.createSession(
    true,  // transacted = true
    Session.SESSION_TRANSACTED
);

MessageProducer producer = session.createProducer(queue);
MessageConsumer consumer = session.createConsumer(queue);

try {
    // Send multiple messages
    producer.send(session.createTextMessage("Message 1"));
    producer.send(session.createTextMessage("Message 2"));
    producer.send(session.createTextMessage("Message 3"));

    // Receive and process
    Message msg = consumer.receive();
    process(msg);

    // Commit - confirm all operations
    session.commit();

} catch (Exception e) {
    // Rollback - cancel all operations
    session.rollback();
}
```

**Example - Money Transfer:**
```java
Session session = connection.createSession(true, Session.SESSION_TRANSACTED);

try {
    // 1. Withdraw from account
    TextMessage withdrawMsg = session.createTextMessage(
        "WITHDRAW:1000:AccountA"
    );
    producer.send(withdrawQueue, withdrawMsg);

    // 2. Deposit to another account
    TextMessage depositMsg = session.createTextMessage(
        "DEPOSIT:1000:AccountB"
    );
    producer.send(depositQueue, depositMsg);

    // 3. Commit both together
    session.commit();
    System.out.println("Transfer successful");

} catch (Exception e) {
    // Rollback - cancel both operations
    session.rollback();
    System.out.println("Transfer failed - rolled back");
}
```

---

### 10. What's the difference between Local and XA Transactions?

**Answer:**

#### Local Transactions
Transaction within JMS only.

```java
// Local transaction - JMS only
Session session = connection.createSession(true, Session.SESSION_TRANSACTED);

producer.send(message1);
producer.send(message2);
session.commit(); // Commit JMS only
```

#### XA Transactions (Distributed)
Transaction includes JMS + Database + other resources.

```java
// XA Transaction - JMS + Database
@Transactional
public void processOrder(Order order) {
    // 1. Save to Database
    orderRepository.save(order);

    // 2. Send JMS message
    jmsTemplate.convertAndSend("orders", order);

    // If either fails, both are rolled back
}
```

**Spring Boot with XA:**
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

        // Both succeed or both fail
    }
}
```

---

## Spring Boot Integration

### 11. How to use ActiveMQ with Spring Boot?

**Answer:**

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

    // Send String
    public void sendMessage(String message) {
        jmsTemplate.convertAndSend("TEST.QUEUE", message);
    }

    // Send Object
    public void sendOrder(Order order) {
        jmsTemplate.convertAndSend("ORDER.QUEUE", order);
    }

    // Send with Priority
    public void sendUrgentOrder(Order order) {
        jmsTemplate.convertAndSend("ORDER.QUEUE", order, message -> {
            message.setJMSPriority(9);
            return message;
        });
    }

    // Send to Topic
    public void publishNews(String news) {
        jmsTemplate.setPubSubDomain(true); // Topic mode
        jmsTemplate.convertAndSend("NEWS.TOPIC", news);
        jmsTemplate.setPubSubDomain(false); // Back to Queue mode
    }
}
```

#### 5. Consumer
```java
@Service
public class OrderConsumer {

    // Simple listener
    @JmsListener(destination = "ORDER.QUEUE")
    public void receiveOrder(Order order) {
        System.out.println("Received order: " + order.getId());
        processOrder(order);
    }

    // With Message object
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

    // Listen from Topic
    @JmsListener(destination = "NEWS.TOPIC", containerFactory = "topicListenerFactory")
    public void receiveNews(String news) {
        System.out.println("News: " + news);
    }

    // With Exception handling
    @JmsListener(destination = "ORDER.QUEUE")
    public void receiveOrderWithRetry(Order order) {
        try {
            processOrder(order);
        } catch (Exception e) {
            log.error("Error processing order", e);
            throw new RuntimeException(e); // Message will be redelivered
        }
    }
}
```

#### 6. Complete Example - E-Commerce

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
        // 1. Save to Database
        order.setStatus(OrderStatus.PENDING);
        orderRepository.save(order);

        // 2. Send for payment processing
        jmsTemplate.convertAndSend("payment.queue", order);

        log.info("Created order: {}", order.getId());
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
                // Send to Shipping
                jmsTemplate.convertAndSend("shipping.queue", order);
                log.info("Payment successful for order: {}", order.getId());
            } else {
                order.setStatus(OrderStatus.PAYMENT_FAILED);
                // Send to Failed queue
                jmsTemplate.convertAndSend("payment.failed", order);
            }

            orderRepository.save(order);

        } catch (Exception e) {
            log.error("Payment processing error", e);
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
        log.info("Starting shipment for order: {}", order.getId());

        Shipment shipment = createShipment(order);
        order.setStatus(OrderStatus.SHIPPED);
        orderRepository.save(order);

        // Send notification to customer
        Notification notification = new Notification(
            order.getCustomerId(),
            "Your order has shipped: " + order.getId()
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
        log.info("Email sent");
    }
}

@Service
public class SmsNotificationConsumer {

    @JmsListener(destination = "notifications.topic", containerFactory = "topicListenerFactory")
    public void sendSms(Notification notification) {
        smsService.send(notification);
        log.info("SMS sent");
    }
}
```

---

## Comparison

### 12. ActiveMQ vs RabbitMQ - Which is better?

**Answer:**

| Comparison | ActiveMQ | RabbitMQ |
|------------|----------|----------|
| **Protocol** | JMS (primary) | AMQP (primary) |
| **Language** | Java | Erlang |
| **Performance** | Good | Excellent |
| **Routing** | Simple | Advanced (4 Exchange types) |
| **Integration** | Strong with Java/Spring | Multi-language |
| **Management UI** | Simple | Advanced |
| **Popularity** | Medium | High |
| **Use Case** | Java applications | Microservices |

**When to use ActiveMQ:**
- ✅ Pure Java application
- ✅ Need JMS standard
- ✅ Simple Spring integration
- ✅ Enterprise environment

**When to use RabbitMQ:**
- ✅ Multi-language Microservices
- ✅ Complex routing
- ✅ High performance
- ✅ Flexible distribution

---

### 13. ActiveMQ vs Apache Kafka?

**Answer:**

| Comparison | ActiveMQ | Apache Kafka |
|------------|----------|--------------|
| **Type** | Message Broker | Event Streaming Platform |
| **Model** | Push | Pull |
| **Storage** | Queue/Topic | Log (persistent) |
| **Ordering** | In Queue | In Partition |
| **Performance** | Thousands/sec | Millions/sec |
| **Use Case** | Messaging | Event Streaming |
| **Retention** | Until consumption | Fixed period (days) |

**When to use ActiveMQ:**
- ✅ Traditional messaging
- ✅ Request-Reply patterns
- ✅ Transactional messaging
- ✅ JMS compliance

**When to use Kafka:**
- ✅ Event streaming
- ✅ Log aggregation
- ✅ Big data processing
- ✅ Real-time analytics
- ✅ High throughput

---

## Performance and Optimization

### 14. How to improve ActiveMQ performance?

**Answer:**

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
// Set number of messages Consumer prefetches
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
// Asynchronous send (doesn't wait for confirmation)
ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory();
factory.setUseAsyncSend(true);

// Or
jmsTemplate.setExplicitQosEnabled(true);
jmsTemplate.setDeliveryPersistent(false); // Non-persistent = faster
```

#### 4. Message Compression
```java
String url = "tcp://localhost:61616?jms.useCompression=true";
```

#### 5. Batch Processing
```java
@JmsListener(destination = "orders")
public void processBatch(List<Order> orders) {
    // Process batch instead of single message
    orderService.processBatch(orders);
}
```

---

### 15. How to monitor ActiveMQ?

**Answer:**

#### 1. Web Console
```
http://localhost:8161/admin
```

**Available Information:**
- Number of messages in Queues/Topics
- Number of Consumers/Producers
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

// Get Queue size
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

### 16. What are the best practices for ActiveMQ?

**Answer:**

#### 1. Use Message Selectors
```java
// Instead of receiving all messages and filtering
@JmsListener(destination = "orders")
public void processAll(Order order) {
    if (order.getPriority() > 5) {
        processHighPriority(order);
    }
}

// Use Selector
@JmsListener(
    destination = "orders",
    selector = "priority > 5"
)
public void processHighPriority(Order order) {
    // Receives only high priority messages
}
```

#### 2. Use CLIENT_ACKNOWLEDGE
```java
@JmsListener(destination = "orders")
public void process(Order order, Session session, Message message) throws JMSException {
    try {
        processOrder(order);
        message.acknowledge();
    } catch (Exception e) {
        session.recover(); // Requeue message
    }
}
```

#### 3. Use Dead Letter Queue
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

## Conclusion

### Quick Review Questions:

1. **What's the difference between Queue and Topic?**
   - Queue: message to one consumer
   - Topic: message to all subscribers

2. **What are Acknowledgment Modes?**
   - AUTO_ACKNOWLEDGE, CLIENT_ACKNOWLEDGE, DUPS_OK_ACKNOWLEDGE, SESSION_TRANSACTED

3. **When to use Durable Subscription?**
   - When you want to receive messages even if offline

4. **What is JMS?**
   - Standard Java API for messaging

5. **ActiveMQ vs RabbitMQ?**
   - ActiveMQ: Strong with Java/JMS
   - RabbitMQ: Better performance, routing

6. **How to improve performance?**
   - Connection pooling
   - Prefetch limit
   - Async send
   - Batch processing

---

**Additional Resources:**
- [ActiveMQ Official Docs](https://activemq.apache.org/documentation)
- [JMS Specification](https://javaee.github.io/jms-spec/)
- [Spring JMS Documentation](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jms)
