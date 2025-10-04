# RabbitMQ Interview Questions

## Table of Contents
1. [Basic Questions](#basic-questions)
2. [AMQP Protocol](#amqp-protocol)
3. [Exchange Types](#exchange-types)
4. [Queues and Messages](#queues-and-messages)
5. [Acknowledgment](#acknowledgment)
6. [Spring Boot Integration](#spring-boot-integration)
7. [Comparison with Other Message Brokers](#comparison-with-other-message-brokers)
8. [Performance and Optimization](#performance-and-optimization)

---

## Basic Questions

### 1. What is RabbitMQ?

**Answer:**
RabbitMQ is an open-source message broker that implements the AMQP (Advanced Message Queuing Protocol). It acts as a mediator between message senders and receivers.

**Core Components:**
- **Producer**: Application that sends messages
- **Exchange**: Receives messages from Producer and routes them to Queues
- **Queue**: Stores messages until consumed by Consumer
- **Consumer**: Application that receives messages
- **Binding**: Link between Exchange and Queue

**Simple Example:**
```java
// Producer
ConnectionFactory factory = new ConnectionFactory();
factory.setHost("localhost");
Connection connection = factory.newConnection();
Channel channel = connection.createChannel();

channel.queueDeclare("hello", false, false, false, null);
String message = "Hello RabbitMQ!";
channel.basicPublish("", "hello", null, message.getBytes());
System.out.println("Sent: " + message);

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
    System.out.println("Received: " + message);
};
channel.basicConsume("hello", true, deliverCallback, consumerTag -> {});
```

---

### 2. What's the difference between RabbitMQ and ActiveMQ?

**Answer:**

| Comparison | RabbitMQ | ActiveMQ |
|------------|----------|----------|
| **Protocol** | AMQP (primary) | JMS (primary) |
| **Language** | Erlang | Java |
| **Performance** | Faster in most cases | Good |
| **Exchange Types** | 4 advanced types | Simple |
| **Routing** | Very flexible | Limited |
| **Popularity** | Very high | Medium |
| **Integration** | Multi-language | Strong with Java |

**When to use RabbitMQ:**
- ✅ Need complex routing
- ✅ Multi-language applications
- ✅ High performance
- ✅ Flexible message distribution

**When to use ActiveMQ:**
- ✅ Pure Java application
- ✅ Need JMS standard
- ✅ Simple Spring integration

---

## AMQP Protocol

### 3. What is the AMQP protocol?

**Answer:**
AMQP (Advanced Message Queuing Protocol) is an open standard application layer protocol for message-oriented middleware.

**Features:**
- **Reliable**: Guaranteed message delivery
- **Interoperable**: Works with different languages
- **Feature-rich**: Supports Routing, Queuing, Security
- **Vendor-neutral**: Not tied to any company

**Versions:**
- AMQP 0-9-1 (most used in RabbitMQ)
- AMQP 1.0

---

### 4. Explain the message lifecycle in RabbitMQ

**Answer:**

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

**Complete Example:**
```java
// 1. Producer sends message with routing key
channel.basicPublish(
    "logs",           // Exchange name
    "error.payment",  // Routing key
    null,
    message.getBytes()
);

// 2. Exchange routes message based on type and bindings

// 3. Binding determines target Queue
channel.queueBind("error_queue", "logs", "error.*");

// 4. Queue stores message

// 5. Consumer receives
channel.basicConsume("error_queue", false, (consumerTag, delivery) -> {
    String msg = new String(delivery.getBody(), "UTF-8");
    System.out.println("Received: " + msg);

    // 6. Acknowledgment
    channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
}, consumerTag -> {});
```

---

## Exchange Types

### 5. What are the Exchange types in RabbitMQ?

**Answer:**
RabbitMQ supports 4 types of Exchanges:

#### 1. Direct Exchange
Routes message to Queue with exact matching routing key.

```java
// Create Direct Exchange
channel.exchangeDeclare("direct_logs", "direct");

// Producer sends with specific routing key
channel.basicPublish("direct_logs", "error", null, message.getBytes());

// Binding - bind Queue to routing key
channel.queueBind("error_queue", "direct_logs", "error");
channel.queueBind("all_logs", "direct_logs", "error");
channel.queueBind("all_logs", "direct_logs", "info");
```

**Use Cases:**
- Log levels (error, info, warning)
- Task routing to specific workers

#### 2. Fanout Exchange
Broadcasts message to all bound Queues.

```java
// Create Fanout Exchange
channel.exchangeDeclare("notifications", "fanout");

// Producer sends without routing key
channel.basicPublish("notifications", "", null, message.getBytes());

// All bound Queues receive the message
channel.queueBind("email_queue", "notifications", "");
channel.queueBind("sms_queue", "notifications", "");
channel.queueBind("push_queue", "notifications", "");
```

**Use Cases:**
- Notifications to all users
- Cache invalidation
- Real-time updates

#### 3. Topic Exchange
Flexible routing using patterns (wildcards).

```java
// Create Topic Exchange
channel.exchangeDeclare("logs_topic", "topic");

// Producer sends with multi-part routing key
channel.basicPublish("logs_topic", "error.payment.db", null, message.getBytes());

// Binding patterns
channel.queueBind("error_queue", "logs_topic", "error.*");
channel.queueBind("payment_queue", "logs_topic", "*.payment.*");
channel.queueBind("all_queue", "logs_topic", "#");
```

**Wildcards:**
- `*`: matches exactly one word
- `#`: matches zero or more words

**Examples:**
- `error.*` matches `error.payment`, `error.db`
- `*.payment.*` matches `error.payment.db`, `info.payment.api`
- `#` matches any routing key

#### 4. Headers Exchange
Routes based on headers instead of routing key.

```java
// Create Headers Exchange
channel.exchangeDeclare("headers_exchange", "headers");

// Producer sends with headers
AMQP.BasicProperties props = new AMQP.BasicProperties.Builder()
    .headers(Map.of(
        "format", "pdf",
        "type", "report",
        "priority", "high"
    ))
    .build();
channel.basicPublish("headers_exchange", "", props, message.getBytes());

// Binding with headers matching
Map<String, Object> bindingArgs = new HashMap<>();
bindingArgs.put("x-match", "all"); // or "any"
bindingArgs.put("format", "pdf");
bindingArgs.put("type", "report");
channel.queueBind("pdf_reports", "headers_exchange", "", bindingArgs);
```

**x-match:**
- `all`: all headers must match
- `any`: at least one header must match

---

### 6. When to use each Exchange type?

**Answer:**

| Type | Use Case | Example |
|------|----------|---------|
| **Direct** | Specific, direct routing | Task queues for different workers |
| **Fanout** | Broadcast to all listeners | Notifications, Cache updates |
| **Topic** | Flexible routing with patterns | Logging system, Event routing |
| **Headers** | Complex routing with metadata | Content-based routing |

**Real-World Examples:**

```java
// 1. Direct - Task System
channel.exchangeDeclare("tasks", "direct");
channel.basicPublish("tasks", "image.resize", null, task.getBytes());
channel.basicPublish("tasks", "video.encode", null, task.getBytes());

// 2. Fanout - User Notifications
channel.exchangeDeclare("user.notifications", "fanout");
channel.basicPublish("user.notifications", "", null, notification.getBytes());

// 3. Topic - Logging System
channel.exchangeDeclare("logs", "topic");
channel.basicPublish("logs", "error.payment.stripe", null, log.getBytes());
channel.basicPublish("logs", "info.user.login", null, log.getBytes());

// 4. Headers - Content Routing
channel.exchangeDeclare("content", "headers");
// Routing based on file type, size, priority
```

---

## Queues and Messages

### 7. What are Queue properties in RabbitMQ?

**Answer:**

```java
// Create Queue with all properties
channel.queueDeclare(
    "my_queue",          // Queue name
    true,                // Durable - survives broker restart
    false,               // Exclusive - only for this Connection
    false,               // Auto-delete - deleted when unused
    Map.of(              // Additional arguments
        "x-message-ttl", 60000,              // Message TTL (60 seconds)
        "x-max-length", 1000,                // Max messages
        "x-max-length-bytes", 10485760,      // Max size (10MB)
        "x-dead-letter-exchange", "dlx",     // Dead Letter Exchange
        "x-max-priority", 10                 // Max priority
    )
);
```

**Key Properties:**

1. **Durable**: Queue survives restart
```java
channel.queueDeclare("durable_queue", true, false, false, null);
```

2. **Exclusive**: Queue private to one connection
```java
channel.queueDeclare("temp_queue", false, true, false, null);
```

3. **Auto-delete**: Deleted automatically when no consumers
```java
channel.queueDeclare("temp_queue", false, false, true, null);
```

---

### 8. What is Message TTL and Queue TTL?

**Answer:**

#### Message TTL
Set message expiration before deletion.

```java
// 1. Queue level (all messages)
Map<String, Object> args = new HashMap<>();
args.put("x-message-ttl", 60000); // 60 seconds
channel.queueDeclare("my_queue", true, false, false, args);

// 2. Per-message level
AMQP.BasicProperties props = new AMQP.BasicProperties.Builder()
    .expiration("60000") // 60 seconds
    .build();
channel.basicPublish("", "my_queue", props, message.getBytes());
```

#### Queue TTL
Queue itself deleted after period of inactivity.

```java
Map<String, Object> args = new HashMap<>();
args.put("x-expires", 300000); // 5 minutes of inactivity
channel.queueDeclare("temp_queue", false, false, false, args);
```

**Use Cases:**
- OTP messages (valid for minutes only)
- Temporary sessions
- Cache expiration

---

### 9. What is Dead Letter Exchange (DLX)?

**Answer:**
Dead Letter Exchange receives messages that failed processing.

**Cases that send message to DLX:**
1. Message rejected with `basic.reject` or `basic.nack` with `requeue=false`
2. Message expired (TTL)
3. Queue full (max-length)

**DLX Setup:**
```java
// 1. Create Dead Letter Exchange
channel.exchangeDeclare("dlx", "direct");
channel.queueDeclare("dead_letter_queue", true, false, false, null);
channel.queueBind("dead_letter_queue", "dlx", "failed");

// 2. Main Queue with DLX
Map<String, Object> args = new HashMap<>();
args.put("x-dead-letter-exchange", "dlx");
args.put("x-dead-letter-routing-key", "failed");
args.put("x-message-ttl", 60000); // Messages expire after 60 seconds
channel.queueDeclare("main_queue", true, false, false, args);

// 3. Consumer rejects message
channel.basicConsume("main_queue", false, (consumerTag, delivery) -> {
    try {
        processMessage(delivery.getBody());
        channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
    } catch (Exception e) {
        // Reject message - will be sent to DLX
        channel.basicReject(delivery.getEnvelope().getDeliveryTag(), false);
    }
}, consumerTag -> {});
```

**Complete Example - Payment Processing:**
```java
// Main Queue
Map<String, Object> mainArgs = new HashMap<>();
mainArgs.put("x-dead-letter-exchange", "payment.dlx");
mainArgs.put("x-dead-letter-routing-key", "payment.failed");
mainArgs.put("x-message-ttl", 300000); // 5 minutes
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
        // Payment failed - send to Failed Queue
        channel.basicReject(delivery.getEnvelope().getDeliveryTag(), false);
        alertAdmin("Payment failed: " + payment.getId());
    }
}, consumerTag -> {});
```

---

## Acknowledgment

### 10. What's the difference between Auto-Ack and Manual Ack?

**Answer:**

#### Auto-Acknowledgment
RabbitMQ considers message consumed as soon as sent to Consumer.

```java
// Auto-Ack = true
channel.basicConsume("my_queue", true, (consumerTag, delivery) -> {
    String message = new String(delivery.getBody(), "UTF-8");
    processMessage(message); // If this fails, message is lost!
}, consumerTag -> {});
```

**Issues:**
- ❌ If processing fails, message is lost
- ❌ If Consumer crashes, messages are lost
- ✅ Higher performance

#### Manual Acknowledgment
Consumer manually confirms message after successful processing.

```java
// Auto-Ack = false
channel.basicConsume("my_queue", false, (consumerTag, delivery) -> {
    try {
        String message = new String(delivery.getBody(), "UTF-8");
        processMessage(message);

        // Confirm success
        channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
    } catch (Exception e) {
        // Reject and requeue
        channel.basicNack(delivery.getEnvelope().getDeliveryTag(), false, true);
    }
}, consumerTag -> {});
```

**Benefits:**
- ✅ Guarantee no message loss
- ✅ Retry on failure
- ❌ Slightly lower performance

---

### 11. What's the difference between basicReject and basicNack?

**Answer:**

```java
// basicReject - reject single message
channel.basicReject(
    deliveryTag,
    false  // requeue: false = send to DLX, true = requeue
);

// basicNack - reject single or multiple messages
channel.basicNack(
    deliveryTag,
    true,   // multiple: true = reject all up to this deliveryTag
    false   // requeue: false = DLX, true = Queue
);
```

**Example - Batch Processing:**
```java
channel.basicConsume("batch_queue", false, (consumerTag, delivery) -> {
    try {
        processBatch(delivery.getBody());
        // Acknowledge all messages in batch
        channel.basicAck(delivery.getEnvelope().getDeliveryTag(), true);
    } catch (Exception e) {
        // Reject all messages in batch
        channel.basicNack(delivery.getEnvelope().getDeliveryTag(), true, false);
    }
}, consumerTag -> {});
```

---

### 12. What is Publisher Confirms?

**Answer:**
Publisher Confirms ensures RabbitMQ successfully received the message.

**Without Confirms:**
```java
// Producer doesn't know if message arrived
channel.basicPublish("", "my_queue", null, message.getBytes());
// What if network failed?
```

**With Confirms:**
```java
// 1. Enable Publisher Confirms
channel.confirmSelect();

// 2. Send message
channel.basicPublish("", "my_queue", null, message.getBytes());

// 3. Wait for confirmation
if (channel.waitForConfirms(5000)) {
    System.out.println("Confirmed - message arrived");
} else {
    System.out.println("Failed confirmation - retry");
}
```

**Async Confirms (Better performance):**
```java
channel.confirmSelect();

// Listener for confirmations
channel.addConfirmListener(new ConfirmListener() {
    @Override
    public void handleAck(long deliveryTag, boolean multiple) {
        System.out.println("Confirmed: " + deliveryTag);
    }

    @Override
    public void handleNack(long deliveryTag, boolean multiple) {
        System.out.println("Failed: " + deliveryTag + " - resend");
    }
});

// Send multiple messages
for (int i = 0; i < 1000; i++) {
    channel.basicPublish("", "my_queue", null, ("Message " + i).getBytes());
}
```

---

## Spring Boot Integration

### 13. How to use RabbitMQ with Spring Boot?

**Answer:**

#### 1. Add Dependencies
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

    // Topic Exchange example
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

    // Fanout Exchange example
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
        log.info("Sent order: {}", order.getId());
    }

    // Send with confirmation
    public void sendOrderWithConfirm(Order order) {
        rabbitTemplate.setConfirmCallback((correlation, ack, reason) -> {
            if (ack) {
                log.info("Confirmed: {}", order.getId());
            } else {
                log.error("Failed: {}, reason: {}", order.getId(), reason);
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

    // Simple listener
    @RabbitListener(queues = "orders")
    public void handleOrder(Order order) {
        log.info("Received order: {}", order.getId());
        processOrder(order);
    }

    // With Manual Acknowledgment
    @RabbitListener(queues = "orders")
    public void handleOrderWithAck(
        Order order,
        Channel channel,
        @Header(AmqpHeaders.DELIVERY_TAG) long tag
    ) throws IOException {
        try {
            processOrder(order);
            channel.basicAck(tag, false);
            log.info("Processed order: {}", order.getId());
        } catch (Exception e) {
            channel.basicNack(tag, false, false); // Send to DLX
            log.error("Failed processing: {}", order.getId(), e);
        }
    }

    // With Retry
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

#### 6. Complete Example - E-Commerce System

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

        // Save to Database
        orderRepository.save(order);

        // Send to RabbitMQ
        rabbitTemplate.convertAndSend(
            "order.exchange",
            "order.created",
            order
        );

        log.info("Created order: {}", order.getId());
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
            // Process payment
            PaymentResult result = paymentGateway.process(order);

            if (result.isSuccess()) {
                order.setStatus(OrderStatus.PAID);
                // Send to Shipping
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
            log.error("Payment processing error", e);
            channel.basicNack(tag, false, true); // Retry
        }
    }
}

// Consumer - Shipping Service
@Service
public class ShippingConsumer {

    @RabbitListener(queues = "shipping.queue")
    public void processShipping(Order order) {
        log.info("Starting shipment for order: {}", order.getId());

        // Create shipment
        Shipment shipment = createShipment(order);

        // Send customer notification
        rabbitTemplate.convertAndSend(
            "notifications",  // Fanout exchange
            "",
            new Notification("Your order has shipped: " + order.getId())
        );
    }
}

// Consumer - Notifications (Email)
@Service
public class EmailConsumer {

    @RabbitListener(queues = "email.notifications")
    public void sendEmail(Notification notification) {
        emailService.send(notification);
        log.info("Email sent");
    }
}

// Consumer - Notifications (SMS)
@Service
public class SmsConsumer {

    @RabbitListener(queues = "sms.notifications")
    public void sendSms(Notification notification) {
        smsService.send(notification);
        log.info("SMS sent");
    }
}
```

---

## Comparison with Other Message Brokers

### 14. What's the difference between RabbitMQ and Apache Kafka?

**Answer:**

| Comparison | RabbitMQ | Apache Kafka |
|------------|----------|--------------|
| **Type** | Message Broker | Event Streaming Platform |
| **Model** | Push-based | Pull-based |
| **Storage** | Queue (deleted after consumption) | Log (retained for period) |
| **Ordering** | Within single Queue | Within single Partition |
| **Performance** | Good (thousands/sec) | Excellent (millions/sec) |
| **Use Case** | Task queues, RPC | Event sourcing, Streaming |
| **Routing** | Very flexible (Exchange types) | Simple (Topics/Partitions) |
| **Consumer Groups** | Limited | Advanced |

**When to use RabbitMQ:**
- ✅ Task queues
- ✅ RPC patterns
- ✅ Complex routing
- ✅ Guaranteed delivery
- ✅ Message priority

**When to use Kafka:**
- ✅ Event streaming
- ✅ Big data processing
- ✅ Log aggregation
- ✅ Real-time analytics
- ✅ High throughput

---

### 15. RabbitMQ vs Redis Pub/Sub?

**Answer:**

| Comparison | RabbitMQ | Redis Pub/Sub |
|------------|----------|---------------|
| **Guarantees** | Guaranteed delivery | Fire-and-forget |
| **Storage** | Persistent queues | In-memory only |
| **Acknowledgment** | Supported | Not supported |
| **Routing** | Advanced | Simple |
| **Performance** | Good | Excellent |
| **Persistence** | Messages persist | If no listener, message lost |

**Redis Pub/Sub:**
```java
// Publisher
jedis.publish("notifications", "Hello");

// Subscriber
jedis.subscribe(new JedisPubSub() {
    @Override
    public void onMessage(String channel, String message) {
        System.out.println("Received: " + message);
    }
}, "notifications");
```

**Key Difference:**
- Redis: If no subscriber connected, message is lost
- RabbitMQ: Messages stored in Queue until consumed

---

## Performance and Optimization

### 16. How to improve RabbitMQ performance?

**Answer:**

#### 1. Prefetch Count
Limit number of messages Consumer receives at once.

```java
// Without prefetch - Consumer receives all messages
channel.basicConsume("my_queue", false, callback);

// With prefetch - Consumer receives only 10 messages at a time
channel.basicQos(10);
channel.basicConsume("my_queue", false, callback);
```

**Benefit:**
- Even distribution of messages among Consumers
- Prevent one Consumer from consuming all messages

```java
// Spring Boot
spring:
  rabbitmq:
    listener:
      simple:
        prefetch: 10
```

#### 2. Lazy Queues
Store messages on Disk instead of RAM.

```java
Map<String, Object> args = new HashMap<>();
args.put("x-queue-mode", "lazy");
channel.queueDeclare("large_queue", true, false, false, args);
```

**When to use:**
- ✅ Very large Queues (millions of messages)
- ✅ Avoid memory consumption
- ❌ Slightly lower performance

#### 3. Publisher Confirms in Batch

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
// Instead of sending 1000 messages
for (int i = 0; i < 1000; i++) {
    channel.basicPublish("", "queue", null, ("Message " + i).getBytes());
}

// Send one batch
List<String> messages = IntStream.range(0, 1000)
    .mapToObj(i -> "Message " + i)
    .collect(Collectors.toList());
String batch = String.join(",", messages);
channel.basicPublish("", "queue", null, batch.getBytes());
```

---

### 17. How to monitor RabbitMQ?

**Answer:**

#### 1. Management UI
RabbitMQ provides web interface at `http://localhost:15672`

```bash
# Enable Management Plugin
rabbitmq-plugins enable rabbitmq_management
```

**Available Information:**
- Number of messages in each Queue
- Send/receive rate
- Connections and Channels
- Memory and Disk usage

#### 2. Prometheus + Grafana

```bash
# Enable Prometheus Plugin
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

### 18. What are the best practices for RabbitMQ?

**Answer:**

#### 1. Use Manual Acknowledgment
```java
// ❌ Bad
channel.basicConsume("queue", true, callback); // Auto-ack

// ✅ Good
channel.basicConsume("queue", false, (tag, delivery) -> {
    try {
        process(delivery);
        channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
    } catch (Exception e) {
        channel.basicNack(delivery.getEnvelope().getDeliveryTag(), false, false);
    }
});
```

#### 2. Use Dead Letter Exchanges
```java
Map<String, Object> args = new HashMap<>();
args.put("x-dead-letter-exchange", "dlx");
args.put("x-message-ttl", 60000);
channel.queueDeclare("main_queue", true, false, false, args);
```

#### 3. Set Message TTL
```java
args.put("x-message-ttl", 300000); // 5 minutes
```

#### 4. Use Durable Queues and Persistent Messages
```java
// Durable queue
channel.queueDeclare("queue", true, false, false, null);

// Persistent message
AMQP.BasicProperties props = new AMQP.BasicProperties.Builder()
    .deliveryMode(2) // Persistent
    .build();
channel.basicPublish("", "queue", props, message.getBytes());
```

#### 5. Use Connection Pooling
```java
@Bean
public CachingConnectionFactory connectionFactory() {
    CachingConnectionFactory factory = new CachingConnectionFactory();
    factory.setChannelCacheSize(25);
    return factory;
}
```

#### 6. Monitor Performance
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

## Conclusion

### Quick Review Questions:

1. **What's the difference between Queue and Exchange?**
   - Queue: stores messages
   - Exchange: routes messages to Queues

2. **How many Exchange types in RabbitMQ?**
   - 4 types: Direct, Fanout, Topic, Headers

3. **When to use Topic Exchange?**
   - When you need flexible routing with patterns

4. **What is Dead Letter Exchange?**
   - Exchange that receives failed messages

5. **What's the difference between RabbitMQ and Kafka?**
   - RabbitMQ: Message broker (task queues)
   - Kafka: Event streaming (logs/analytics)

6. **How to ensure no message loss?**
   - Manual acknowledgment
   - Durable queues
   - Persistent messages
   - Publisher confirms

---

**Additional Resources:**
- [RabbitMQ Official Docs](https://www.rabbitmq.com/documentation.html)
- [RabbitMQ Tutorials](https://www.rabbitmq.com/getstarted.html)
- [Spring AMQP Documentation](https://docs.spring.io/spring-amqp/reference/)
