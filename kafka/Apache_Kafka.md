# دليل شامل لـ Apache Kafka

## المحتويات
1. [مقدمة عن Kafka](#مقدمة-عن-kafka)
2. [المفاهيم الأساسية](#المفاهيم-الأساسية)
3. [بنية Kafka Architecture](#بنية-kafka-architecture)
4. [التثبيت والإعداد](#التثبيت-والإعداد)
5. [Producer API](#producer-api)
6. [Consumer API](#consumer-api)
7. [Kafka Streams](#kafka-streams)
8. [Kafka Connect](#kafka-connect)
9. [إدارة Topics](#إدارة-topics)
10. [أمثلة عملية](#أمثلة-عملية)

---

## مقدمة عن Kafka

### ما هو Apache Kafka؟
**Apache Kafka** هو نظام **distributed streaming platform** يُستخدم لبناء تطبيقات real-time data pipelines و streaming applications.

### الاستخدامات الرئيسية

```
📊 Messaging System       - نظام رسائل بين الأنظمة
📈 Activity Tracking      - تتبع نشاط المستخدمين
📁 Log Aggregation        - تجميع السجلات
🔄 Stream Processing      - معالجة البيانات المتدفقة
💾 Event Sourcing         - حفظ الأحداث
📡 Commit Log             - سجل التغييرات
```

### مميزات Kafka

✅ **High Throughput** - معالجة ملايين الرسائل في الثانية
✅ **Scalability** - قابل للتوسع أفقياً
✅ **Durability** - حفظ البيانات بشكل دائم
✅ **Fault Tolerance** - تحمل الأعطال
✅ **Low Latency** - زمن استجابة قليل (milliseconds)
✅ **Distributed** - موزع على عدة servers

---

## المفاهيم الأساسية

### 1. Topic (الموضوع)

```
Topic = فئة أو تصنيف للرسائل

مثال:
- orders           (طلبات الشراء)
- user-clicks      (نقرات المستخدمين)
- system-logs      (سجلات النظام)
- payments         (المدفوعات)
```

### 2. Partition (التقسيم)

```
كل Topic مقسم إلى Partitions

Topic: orders
├── Partition 0: [msg1, msg2, msg3]
├── Partition 1: [msg4, msg5, msg6]
└── Partition 2: [msg7, msg8, msg9]

الفوائد:
✅ Parallelism - معالجة متوازية
✅ Scalability - قابلية التوسع
✅ Ordering - ترتيب الرسائل داخل Partition
```

### 3. Offset (الإزاحة)

```
رقم تسلسلي فريد لكل رسالة داخل Partition

Partition 0:
Offset 0: "Order #1"
Offset 1: "Order #2"
Offset 2: "Order #3"
Offset 3: "Order #4"
       ↑
   المستهلك هنا (يقرأ من offset 3)
```

### 4. Producer (المُنتج)

```java
// يرسل الرسائل إلى Topics
Producer → Topic → Partitions
```

### 5. Consumer (المُستهلك)

```java
// يقرأ الرسائل من Topics
Consumer ← Topic ← Partitions
```

### 6. Consumer Group

```
مجموعة من Consumers يعملون معاً

Topic: orders (3 partitions)
├── Consumer Group: "order-processors"
    ├── Consumer 1 → reads Partition 0
    ├── Consumer 2 → reads Partition 1
    └── Consumer 3 → reads Partition 2

القواعد:
✅ كل partition يُقرأ بواسطة consumer واحد فقط في نفس الـ group
✅ إذا كان عدد consumers أكبر من partitions، بعضهم سيكون idle
✅ إذا فشل consumer، partitions توزع على الباقيين
```

### 7. Broker

```
Kafka Server - الخادم الذي يخزن البيانات

Kafka Cluster:
├── Broker 1 (Leader for Partition 0, 2)
├── Broker 2 (Leader for Partition 1)
└── Broker 3 (Replicas)
```

### 8. Replication (النسخ المتماثل)

```
نسخ احتياطية للبيانات

Topic: orders (Replication Factor = 3)

Partition 0:
├── Leader: Broker 1
├── Replica 1: Broker 2
└── Replica 2: Broker 3

إذا فشل Broker 1:
→ Broker 2 يصبح Leader جديد
```

---

## بنية Kafka Architecture

### الهيكل العام

```
┌─────────────────────────────────────────────┐
│            Kafka Cluster                     │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐     │
│  │ Broker 1│  │ Broker 2│  │ Broker 3│     │
│  └─────────┘  └─────────┘  └─────────┘     │
│       ↑            ↑            ↑           │
└───────┼────────────┼────────────┼───────────┘
        │            │            │
    ┌───┴───────┬────┴────────┬───┴────┐
    │           │             │        │
┌───▼───┐  ┌───▼───┐    ┌───▼───┐  ┌──▼──┐
│Producer│ │Producer│   │Consumer│ │Consumer│
└────────┘ └────────┘   └────────┘ └──────┘
```

### ZooKeeper (قديم) vs KRaft (جديد)

```
# ZooKeeper Mode (قبل Kafka 2.8)
ZooKeeper
    ↓
يدير Metadata للـ Brokers

# KRaft Mode (Kafka 3.0+)
بدون ZooKeeper
Kafka يدير نفسه
```

---

## التثبيت والإعداد

### التثبيت على Windows

```bash
# 1. تحميل Kafka
# من https://kafka.apache.org/downloads

# 2. فك الضغط
cd C:\kafka

# 3. تشغيل ZooKeeper (إذا كنت تستخدم ZooKeeper mode)
bin\windows\zookeeper-server-start.bat config\zookeeper.properties

# 4. تشغيل Kafka Broker
bin\windows\kafka-server-start.bat config\server.properties
```

### التثبيت على Linux/Mac

```bash
# 1. تحميل وفك الضغط
wget https://downloads.apache.org/kafka/3.6.0/kafka_2.13-3.6.0.tgz
tar -xzf kafka_2.13-3.6.0.tgz
cd kafka_2.13-3.6.0

# 2. تشغيل ZooKeeper
bin/zookeeper-server-start.sh config/zookeeper.properties

# 3. تشغيل Kafka
bin/kafka-server-start.sh config/server.properties
```

### التثبيت باستخدام Docker

```yaml
# docker-compose.yml
version: '3'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - "2181:2181"

  kafka:
    image: confluentinc/cp-kafka:7.5.0
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
```

```bash
# تشغيل
docker-compose up -d

# إيقاف
docker-compose down
```

---

## Producer API

### Maven Dependencies

```xml
<!-- pom.xml -->
<dependencies>
    <dependency>
        <groupId>org.apache.kafka</groupId>
        <artifactId>kafka-clients</artifactId>
        <version>3.6.0</version>
    </dependency>

    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-simple</artifactId>
        <version>2.0.9</version>
    </dependency>
</dependencies>
```

### Producer بسيط

```java
import org.apache.kafka.clients.producer.*;
import org.apache.kafka.common.serialization.StringSerializer;
import java.util.Properties;

public class SimpleProducer {
    public static void main(String[] args) {
        // 1. إعداد Properties
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
            StringSerializer.class.getName());
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,
            StringSerializer.class.getName());

        // 2. إنشاء Producer
        KafkaProducer<String, String> producer =
            new KafkaProducer<>(props);

        // 3. إنشاء Record
        ProducerRecord<String, String> record =
            new ProducerRecord<>("my-topic", "key1", "Hello Kafka!");

        // 4. إرسال الرسالة
        try {
            producer.send(record);
            System.out.println("Message sent successfully");
        } finally {
            producer.close();
        }
    }
}
```

### Producer مع Callback

```java
public class ProducerWithCallback {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("key.serializer", StringSerializer.class.getName());
        props.put("value.serializer", StringSerializer.class.getName());

        KafkaProducer<String, String> producer = new KafkaProducer<>(props);

        for (int i = 0; i < 10; i++) {
            ProducerRecord<String, String> record =
                new ProducerRecord<>("my-topic", "key-" + i, "message-" + i);

            // إرسال مع Callback
            producer.send(record, new Callback() {
                @Override
                public void onCompletion(RecordMetadata metadata, Exception exception) {
                    if (exception == null) {
                        System.out.println("Message sent successfully:");
                        System.out.println("Topic: " + metadata.topic());
                        System.out.println("Partition: " + metadata.partition());
                        System.out.println("Offset: " + metadata.offset());
                        System.out.println("Timestamp: " + metadata.timestamp());
                    } else {
                        System.err.println("Error sending message: " +
                            exception.getMessage());
                    }
                }
            });
        }

        producer.close();
    }
}
```

### Producer مع Custom Partitioner

```java
// Custom Partitioner
import org.apache.kafka.clients.producer.Partitioner;
import org.apache.kafka.common.Cluster;

public class CustomPartitioner implements Partitioner {
    @Override
    public int partition(String topic, Object key, byte[] keyBytes,
                        Object value, byte[] valueBytes, Cluster cluster) {
        // الرسائل التي تحتوي على "VIP" تذهب للـ Partition 0
        if (value.toString().contains("VIP")) {
            return 0;
        }

        // باقي الرسائل توزع بشكل round-robin
        int numPartitions = cluster.partitionCountForTopic(topic);
        return Math.abs(key.hashCode()) % (numPartitions - 1) + 1;
    }

    @Override
    public void close() {}

    @Override
    public void configure(Map<String, ?> configs) {}
}

// استخدام Custom Partitioner
public class ProducerWithCustomPartitioner {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("key.serializer", StringSerializer.class.getName());
        props.put("value.serializer", StringSerializer.class.getName());
        props.put("partitioner.class", CustomPartitioner.class.getName());

        KafkaProducer<String, String> producer = new KafkaProducer<>(props);

        producer.send(new ProducerRecord<>("my-topic", "user1", "VIP Customer"));
        producer.send(new ProducerRecord<>("my-topic", "user2", "Regular Customer"));

        producer.close();
    }
}
```

### Producer مع JSON

```java
// POJO Class
class Order {
    private String orderId;
    private String customerId;
    private double amount;

    // Constructor, Getters, Setters
    public Order(String orderId, String customerId, double amount) {
        this.orderId = orderId;
        this.customerId = customerId;
        this.amount = amount;
    }

    public String getOrderId() { return orderId; }
    public String getCustomerId() { return customerId; }
    public double getAmount() { return amount; }
}

// JSON Serializer
import com.fasterxml.jackson.databind.ObjectMapper;
import org.apache.kafka.common.serialization.Serializer;

public class JsonSerializer<T> implements Serializer<T> {
    private final ObjectMapper objectMapper = new ObjectMapper();

    @Override
    public byte[] serialize(String topic, T data) {
        try {
            return objectMapper.writeValueAsBytes(data);
        } catch (Exception e) {
            throw new RuntimeException("Error serializing JSON", e);
        }
    }
}

// Producer
public class JsonProducer {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("key.serializer", StringSerializer.class.getName());
        props.put("value.serializer", JsonSerializer.class.getName());

        KafkaProducer<String, Order> producer = new KafkaProducer<>(props);

        Order order = new Order("ORD-001", "CUST-123", 1500.50);
        ProducerRecord<String, Order> record =
            new ProducerRecord<>("orders", order.getOrderId(), order);

        producer.send(record, (metadata, exception) -> {
            if (exception == null) {
                System.out.println("Order sent: " + order.getOrderId());
            } else {
                exception.printStackTrace();
            }
        });

        producer.close();
    }
}
```

### Producer Configurations المهمة

```java
Properties props = new Properties();

// ✅ Required
props.put("bootstrap.servers", "localhost:9092");
props.put("key.serializer", StringSerializer.class.getName());
props.put("value.serializer", StringSerializer.class.getName());

// ⚙️ Performance
props.put("batch.size", 16384);           // حجم الـ batch
props.put("linger.ms", 10);               // الانتظار قبل الإرسال
props.put("buffer.memory", 33554432);     // حجم الذاكرة للـ buffer
props.put("compression.type", "snappy");  // الضغط (snappy, gzip, lz4, zstd)

// 🔒 Reliability
props.put("acks", "all");                 // 0, 1, all/-1
props.put("retries", 3);                  // عدد المحاولات
props.put("max.in.flight.requests.per.connection", 5);
props.put("enable.idempotence", true);    // منع التكرار

// 🔑 Partitioning
props.put("partitioner.class", "org.apache.kafka.clients.producer.RoundRobinPartitioner");
```

### Acknowledgment Modes (acks)

```java
// acks = 0 (أسرع، غير آمن)
// Producer لا ينتظر تأكيد من Broker
props.put("acks", "0");

// acks = 1 (متوسط)
// Producer ينتظر تأكيد من Leader فقط
props.put("acks", "1");

// acks = all/-1 (أبطأ، أكثر أماناً)
// Producer ينتظر تأكيد من Leader وجميع Replicas
props.put("acks", "all");
props.put("min.insync.replicas", 2);  // في server.properties
```

---

## Consumer API

### Consumer بسيط

```java
import org.apache.kafka.clients.consumer.*;
import org.apache.kafka.common.serialization.StringDeserializer;
import java.time.Duration;
import java.util.*;

public class SimpleConsumer {
    public static void main(String[] args) {
        // 1. إعداد Properties
        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "my-consumer-group");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,
            StringDeserializer.class.getName());
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,
            StringDeserializer.class.getName());
        props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");

        // 2. إنشاء Consumer
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);

        // 3. Subscribe to Topic
        consumer.subscribe(Collections.singletonList("my-topic"));

        // 4. Poll for messages
        try {
            while (true) {
                ConsumerRecords<String, String> records =
                    consumer.poll(Duration.ofMillis(100));

                for (ConsumerRecord<String, String> record : records) {
                    System.out.printf("Topic: %s, Partition: %d, Offset: %d, " +
                        "Key: %s, Value: %s%n",
                        record.topic(), record.partition(), record.offset(),
                        record.key(), record.value());
                }
            }
        } finally {
            consumer.close();
        }
    }
}
```

### Consumer مع Manual Commit

```java
public class ManualCommitConsumer {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("group.id", "my-group");
        props.put("key.deserializer", StringDeserializer.class.getName());
        props.put("value.deserializer", StringDeserializer.class.getName());
        props.put("enable.auto.commit", "false");  // Manual commit

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        consumer.subscribe(Arrays.asList("my-topic"));

        try {
            while (true) {
                ConsumerRecords<String, String> records =
                    consumer.poll(Duration.ofMillis(100));

                for (ConsumerRecord<String, String> record : records) {
                    System.out.println("Processing: " + record.value());

                    // معالجة الرسالة...
                    processRecord(record);
                }

                // Commit بعد معالجة جميع الرسائل
                consumer.commitSync();
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            consumer.close();
        }
    }

    private static void processRecord(ConsumerRecord<String, String> record) {
        // معالجة الرسالة
        System.out.println("Processed: " + record.value());
    }
}
```

### Consumer مع Async Commit

```java
public class AsyncCommitConsumer {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("group.id", "async-group");
        props.put("key.deserializer", StringDeserializer.class.getName());
        props.put("value.deserializer", StringDeserializer.class.getName());
        props.put("enable.auto.commit", "false");

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        consumer.subscribe(Arrays.asList("my-topic"));

        try {
            while (true) {
                ConsumerRecords<String, String> records =
                    consumer.poll(Duration.ofMillis(100));

                for (ConsumerRecord<String, String> record : records) {
                    System.out.println("Value: " + record.value());
                }

                // Async commit مع callback
                consumer.commitAsync(new OffsetCommitCallback() {
                    @Override
                    public void onComplete(Map<TopicPartition, OffsetAndMetadata> offsets,
                                          Exception exception) {
                        if (exception != null) {
                            System.err.println("Commit failed: " +
                                exception.getMessage());
                        } else {
                            System.out.println("Commit successful");
                        }
                    }
                });
            }
        } finally {
            // Sync commit عند الإغلاق
            try {
                consumer.commitSync();
            } finally {
                consumer.close();
            }
        }
    }
}
```

### Consumer مع Specific Partition

```java
public class PartitionConsumer {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("group.id", "partition-group");
        props.put("key.deserializer", StringDeserializer.class.getName());
        props.put("value.deserializer", StringDeserializer.class.getName());

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);

        // Assign specific partitions
        TopicPartition partition0 = new TopicPartition("my-topic", 0);
        TopicPartition partition1 = new TopicPartition("my-topic", 1);
        consumer.assign(Arrays.asList(partition0, partition1));

        // Seek to specific offset
        consumer.seek(partition0, 10);  // ابدأ من offset 10

        while (true) {
            ConsumerRecords<String, String> records =
                consumer.poll(Duration.ofMillis(100));

            for (ConsumerRecord<String, String> record : records) {
                System.out.printf("Partition: %d, Offset: %d, Value: %s%n",
                    record.partition(), record.offset(), record.value());
            }
        }
    }
}
```

### Consumer Rebalancing

```java
import org.apache.kafka.clients.consumer.ConsumerRebalanceListener;
import org.apache.kafka.common.TopicPartition;

public class RebalanceConsumer {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("group.id", "rebalance-group");
        props.put("key.deserializer", StringDeserializer.class.getName());
        props.put("value.deserializer", StringDeserializer.class.getName());
        props.put("enable.auto.commit", "false");

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);

        // Subscribe مع RebalanceListener
        consumer.subscribe(Arrays.asList("my-topic"),
            new ConsumerRebalanceListener() {
                @Override
                public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
                    System.out.println("Partitions revoked: " + partitions);
                    // Commit offsets قبل فقدان الـ partitions
                    consumer.commitSync();
                }

                @Override
                public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
                    System.out.println("Partitions assigned: " + partitions);
                }
            });

        try {
            while (true) {
                ConsumerRecords<String, String> records =
                    consumer.poll(Duration.ofMillis(100));

                for (ConsumerRecord<String, String> record : records) {
                    System.out.println(record.value());
                }

                consumer.commitAsync();
            }
        } finally {
            consumer.close();
        }
    }
}
```

### Consumer Configurations المهمة

```java
Properties props = new Properties();

// ✅ Required
props.put("bootstrap.servers", "localhost:9092");
props.put("group.id", "my-consumer-group");
props.put("key.deserializer", StringDeserializer.class.getName());
props.put("value.deserializer", StringDeserializer.class.getName());

// 📖 Offset Management
props.put("auto.offset.reset", "earliest");  // earliest, latest, none
props.put("enable.auto.commit", "true");
props.put("auto.commit.interval.ms", "5000");

// ⚙️ Performance
props.put("fetch.min.bytes", "1");           // الحد الأدنى للبيانات
props.put("fetch.max.wait.ms", "500");       // أقصى وقت انتظار
props.put("max.partition.fetch.bytes", "1048576");  // 1MB
props.put("max.poll.records", "500");        // عدد الرسائل لكل poll

// 🔄 Rebalancing
props.put("session.timeout.ms", "10000");
props.put("heartbeat.interval.ms", "3000");
props.put("max.poll.interval.ms", "300000");
```

---

## Kafka Streams

### مثال بسيط - Word Count

```java
import org.apache.kafka.streams.*;
import org.apache.kafka.streams.kstream.*;
import java.util.Properties;
import java.util.Arrays;

public class WordCountStream {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, "word-count-app");
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG,
            Serdes.String().getClass());
        props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG,
            Serdes.String().getClass());

        StreamsBuilder builder = new StreamsBuilder();

        // 1. قراءة من topic
        KStream<String, String> textLines =
            builder.stream("input-topic");

        // 2. معالجة البيانات
        KTable<String, Long> wordCounts = textLines
            // تقسيم إلى كلمات
            .flatMapValues(value -> Arrays.asList(value.toLowerCase().split("\\W+")))
            // تجميع حسب الكلمة
            .groupBy((key, word) -> word)
            // عد التكرارات
            .count();

        // 3. كتابة النتيجة
        wordCounts.toStream()
            .to("output-topic", Produced.with(Serdes.String(), Serdes.Long()));

        // 4. تشغيل التطبيق
        KafkaStreams streams = new KafkaStreams(builder.build(), props);
        streams.start();

        // Shutdown hook
        Runtime.getRuntime().addShutdownHook(new Thread(streams::close));
    }
}
```

### Stream Transformations

```java
public class StreamTransformations {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, "transformations-app");
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");

        StreamsBuilder builder = new StreamsBuilder();
        KStream<String, String> source = builder.stream("input-topic");

        // 1. filter
        KStream<String, String> filtered = source
            .filter((key, value) -> value.length() > 10);

        // 2. map
        KStream<String, String> mapped = source
            .map((key, value) -> KeyValue.pair(key.toUpperCase(), value.toUpperCase()));

        // 3. flatMap
        KStream<String, String> flatMapped = source
            .flatMap((key, value) -> {
                List<KeyValue<String, String>> result = new ArrayList<>();
                for (String word : value.split(" ")) {
                    result.add(KeyValue.pair(key, word));
                }
                return result;
            });

        // 4. branch - تقسيم Stream
        KStream<String, String>[] branches = source.branch(
            (key, value) -> value.contains("ERROR"),
            (key, value) -> value.contains("WARNING"),
            (key, value) -> true  // باقي الرسائل
        );

        KStream<String, String> errors = branches[0];
        KStream<String, String> warnings = branches[1];
        KStream<String, String> others = branches[2];

        // 5. merge - دمج Streams
        KStream<String, String> merged = errors.merge(warnings);

        // 6. peek - للـ debugging
        source.peek((key, value) ->
            System.out.println("Key: " + key + ", Value: " + value));
    }
}
```

### Joining Streams

```java
public class StreamJoins {
    public static void main(String[] args) {
        StreamsBuilder builder = new StreamsBuilder();

        // Stream 1: Orders
        KStream<String, String> orders = builder.stream("orders");

        // Stream 2: Payments
        KStream<String, String> payments = builder.stream("payments");

        // Join Window - 5 دقائق
        JoinWindows joinWindow = JoinWindows.of(Duration.ofMinutes(5));

        // Inner Join
        KStream<String, String> joined = orders.join(
            payments,
            (orderValue, paymentValue) ->
                "Order: " + orderValue + ", Payment: " + paymentValue,
            joinWindow
        );

        // Left Join
        KStream<String, String> leftJoined = orders.leftJoin(
            payments,
            (orderValue, paymentValue) ->
                "Order: " + orderValue +
                ", Payment: " + (paymentValue != null ? paymentValue : "PENDING"),
            joinWindow
        );

        // Outer Join
        KStream<String, String> outerJoined = orders.outerJoin(
            payments,
            (orderValue, paymentValue) ->
                "Order: " + (orderValue != null ? orderValue : "NONE") +
                ", Payment: " + (paymentValue != null ? paymentValue : "NONE"),
            joinWindow
        );

        joined.to("joined-topic");
    }
}
```

### Windowing

```java
public class WindowingExample {
    public static void main(String[] args) {
        StreamsBuilder builder = new StreamsBuilder();
        KStream<String, Long> clicks = builder.stream("user-clicks");

        // 1. Tumbling Window (نافذة ثابتة)
        KTable<Windowed<String>, Long> tumblingCounts = clicks
            .groupByKey()
            .windowedBy(TimeWindows.of(Duration.ofMinutes(5)))
            .count();

        // 2. Hopping Window (نافذة متداخلة)
        KTable<Windowed<String>, Long> hoppingCounts = clicks
            .groupByKey()
            .windowedBy(TimeWindows.of(Duration.ofMinutes(5))
                .advanceBy(Duration.ofMinutes(1)))
            .count();

        // 3. Session Window (نافذة جلسة)
        KTable<Windowed<String>, Long> sessionCounts = clicks
            .groupByKey()
            .windowedBy(SessionWindows.with(Duration.ofMinutes(10)))
            .count();

        // كتابة النتيجة
        tumblingCounts.toStream()
            .map((key, value) -> KeyValue.pair(
                key.key() + "@" + key.window().start() + "->" + key.window().end(),
                value
            ))
            .to("windowed-counts");
    }
}
```

---

## Kafka Connect

### Source Connector - قراءة من Database

```json
{
  "name": "mysql-source-connector",
  "config": {
    "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
    "tasks.max": "1",
    "connection.url": "jdbc:mysql://localhost:3306/mydb",
    "connection.user": "user",
    "connection.password": "password",
    "table.whitelist": "orders,customers",
    "mode": "incrementing",
    "incrementing.column.name": "id",
    "topic.prefix": "mysql-"
  }
}
```

### Sink Connector - كتابة إلى Elasticsearch

```json
{
  "name": "elasticsearch-sink-connector",
  "config": {
    "connector.class": "io.confluent.connect.elasticsearch.ElasticsearchSinkConnector",
    "tasks.max": "1",
    "topics": "orders,customers",
    "connection.url": "http://localhost:9200",
    "type.name": "_doc",
    "key.ignore": "true",
    "schema.ignore": "true"
  }
}
```

### إدارة Connectors

```bash
# عرض جميع Connectors
curl http://localhost:8083/connectors

# إنشاء Connector
curl -X POST http://localhost:8083/connectors \
  -H "Content-Type: application/json" \
  -d @connector-config.json

# حذف Connector
curl -X DELETE http://localhost:8083/connectors/mysql-source-connector

# إيقاف مؤقت
curl -X PUT http://localhost:8083/connectors/mysql-source-connector/pause

# استئناف
curl -X PUT http://localhost:8083/connectors/mysql-source-connector/resume

# حالة Connector
curl http://localhost:8083/connectors/mysql-source-connector/status
```

---

## إدارة Topics

### CLI Commands

```bash
# إنشاء Topic
kafka-topics.sh --create \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --partitions 3 \
  --replication-factor 2

# عرض جميع Topics
kafka-topics.sh --list \
  --bootstrap-server localhost:9092

# وصف Topic
kafka-topics.sh --describe \
  --bootstrap-server localhost:9092 \
  --topic my-topic

# حذف Topic
kafka-topics.sh --delete \
  --bootstrap-server localhost:9092 \
  --topic my-topic

# زيادة عدد Partitions
kafka-topics.sh --alter \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --partitions 5

# تغيير Configurations
kafka-configs.sh --alter \
  --bootstrap-server localhost:9092 \
  --entity-type topics \
  --entity-name my-topic \
  --add-config retention.ms=86400000
```

### Producer من CLI

```bash
# إرسال رسائل
kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic

# مع key
kafka-console-producer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --property "parse.key=true" \
  --property "key.separator=:"
```

### Consumer من CLI

```bash
# قراءة رسائل
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic

# من البداية
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --from-beginning

# مع key
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --property print.key=true \
  --property key.separator=":"

# مع consumer group
kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic my-topic \
  --group my-group
```

### Consumer Groups Management

```bash
# عرض جميع Consumer Groups
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --list

# وصف Consumer Group
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group my-group \
  --describe

# Reset Offsets
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group my-group \
  --reset-offsets \
  --to-earliest \
  --topic my-topic \
  --execute

# حذف Consumer Group
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group my-group \
  --delete
```

---

## أمثلة عملية

### مثال 1: Real-time Order Processing System

```java
// Order POJO
class Order {
    private String orderId;
    private String customerId;
    private String product;
    private int quantity;
    private double price;
    private String status;

    // Constructor, Getters, Setters
}

// Order Producer
public class OrderProducer {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("key.serializer", StringSerializer.class.getName());
        props.put("value.serializer", JsonSerializer.class.getName());

        KafkaProducer<String, Order> producer = new KafkaProducer<>(props);

        // محاكاة طلبات
        for (int i = 1; i <= 100; i++) {
            Order order = new Order(
                "ORD-" + i,
                "CUST-" + (i % 10),
                "Product-" + (i % 5),
                (int) (Math.random() * 10 + 1),
                Math.random() * 1000,
                "PENDING"
            );

            ProducerRecord<String, Order> record =
                new ProducerRecord<>("orders", order.getOrderId(), order);

            producer.send(record, (metadata, exception) -> {
                if (exception == null) {
                    System.out.println("Order sent: " + order.getOrderId() +
                        " to partition " + metadata.partition());
                }
            });

            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        producer.close();
    }
}

// Order Processor (Consumer)
public class OrderProcessor {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("group.id", "order-processors");
        props.put("key.deserializer", StringDeserializer.class.getName());
        props.put("value.deserializer", JsonDeserializer.class.getName());
        props.put("enable.auto.commit", "false");

        KafkaConsumer<String, Order> consumer = new KafkaConsumer<>(props);
        consumer.subscribe(Arrays.asList("orders"));

        KafkaProducer<String, Order> producer = createProducer();

        try {
            while (true) {
                ConsumerRecords<String, Order> records =
                    consumer.poll(Duration.ofMillis(100));

                for (ConsumerRecord<String, Order> record : records) {
                    Order order = record.value();

                    // معالجة الطلب
                    System.out.println("Processing order: " + order.getOrderId());

                    // تحديث الحالة
                    order.setStatus("PROCESSING");

                    // التحقق من المخزون
                    if (checkInventory(order)) {
                        order.setStatus("CONFIRMED");
                        // إرسال لـ topic آخر
                        producer.send(new ProducerRecord<>(
                            "confirmed-orders", order.getOrderId(), order));
                    } else {
                        order.setStatus("OUT_OF_STOCK");
                        producer.send(new ProducerRecord<>(
                            "failed-orders", order.getOrderId(), order));
                    }
                }

                consumer.commitSync();
            }
        } finally {
            consumer.close();
            producer.close();
        }
    }

    private static KafkaProducer<String, Order> createProducer() {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("key.serializer", StringSerializer.class.getName());
        props.put("value.serializer", JsonSerializer.class.getName());
        return new KafkaProducer<>(props);
    }

    private static boolean checkInventory(Order order) {
        // محاكاة فحص المخزون
        return Math.random() > 0.2; // 80% نجاح
    }
}
```

### مثال 2: Log Aggregation System

```java
// Log Entry
class LogEntry {
    private String timestamp;
    private String level;
    private String service;
    private String message;

    // Constructor, Getters, Setters
}

// Log Producer (من خدمات مختلفة)
public class ServiceLogProducer {
    private static final KafkaProducer<String, LogEntry> producer;

    static {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("key.serializer", StringSerializer.class.getName());
        props.put("value.serializer", JsonSerializer.class.getName());
        producer = new KafkaProducer<>(props);
    }

    public static void log(String service, String level, String message) {
        LogEntry entry = new LogEntry(
            LocalDateTime.now().toString(),
            level,
            service,
            message
        );

        ProducerRecord<String, LogEntry> record =
            new ProducerRecord<>("application-logs", service, entry);

        producer.send(record);
    }

    public static void main(String[] args) {
        // محاكاة logs من خدمات مختلفة
        String[] services = {"auth-service", "payment-service", "order-service"};
        String[] levels = {"INFO", "WARNING", "ERROR"};

        for (int i = 0; i < 1000; i++) {
            String service = services[(int) (Math.random() * services.length)];
            String level = levels[(int) (Math.random() * levels.length)];

            log(service, level, "Log message " + i);

            try {
                Thread.sleep(50);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        producer.close();
    }
}

// Log Analyzer (Kafka Streams)
public class LogAnalyzer {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, "log-analyzer");
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");

        StreamsBuilder builder = new StreamsBuilder();
        KStream<String, LogEntry> logs = builder.stream("application-logs");

        // 1. عد الـ logs حسب المستوى
        KTable<String, Long> logCounts = logs
            .groupBy((key, value) -> value.getLevel())
            .count();

        logCounts.toStream().to("log-counts");

        // 2. فلترة الأخطاء فقط
        logs.filter((key, value) -> value.getLevel().equals("ERROR"))
            .to("error-logs");

        // 3. عد الـ logs حسب الخدمة
        KTable<String, Long> serviceCounts = logs
            .groupBy((key, value) -> value.getService())
            .count();

        serviceCounts.toStream().to("service-log-counts");

        // 4. نافذة زمنية - عدد الأخطاء كل دقيقة
        KTable<Windowed<String>, Long> errorCountsPerMinute = logs
            .filter((key, value) -> value.getLevel().equals("ERROR"))
            .groupBy((key, value) -> value.getService())
            .windowedBy(TimeWindows.of(Duration.ofMinutes(1)))
            .count();

        errorCountsPerMinute.toStream()
            .map((key, value) -> KeyValue.pair(
                key.key() + "@" + key.window().start(),
                value
            ))
            .to("error-counts-per-minute");

        KafkaStreams streams = new KafkaStreams(builder.build(), props);
        streams.start();

        Runtime.getRuntime().addShutdownHook(new Thread(streams::close));
    }
}
```

### مثال 3: Real-time Analytics Dashboard

```java
// User Event
class UserEvent {
    private String userId;
    private String eventType;  // CLICK, VIEW, PURCHASE
    private String page;
    private long timestamp;

    // Constructor, Getters, Setters
}

// Event Producer
public class UserEventProducer {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("key.serializer", StringSerializer.class.getName());
        props.put("value.serializer", JsonSerializer.class.getName());

        KafkaProducer<String, UserEvent> producer = new KafkaProducer<>(props);

        String[] eventTypes = {"CLICK", "VIEW", "PURCHASE"};
        String[] pages = {"home", "products", "cart", "checkout"};

        for (int i = 0; i < 10000; i++) {
            UserEvent event = new UserEvent(
                "user-" + (int) (Math.random() * 100),
                eventTypes[(int) (Math.random() * eventTypes.length)],
                pages[(int) (Math.random() * pages.length)],
                System.currentTimeMillis()
            );

            producer.send(new ProducerRecord<>(
                "user-events", event.getUserId(), event));

            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        producer.close();
    }
}

// Analytics Processor
public class AnalyticsProcessor {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, "analytics-app");
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");

        StreamsBuilder builder = new StreamsBuilder();
        KStream<String, UserEvent> events = builder.stream("user-events");

        // 1. عدد الأحداث حسب النوع
        events.groupBy((key, value) -> value.getEventType())
            .count()
            .toStream()
            .to("event-type-counts");

        // 2. عدد الأحداث حسب الصفحة
        events.groupBy((key, value) -> value.getPage())
            .count()
            .toStream()
            .to("page-view-counts");

        // 3. عدد المشتريات لكل مستخدم
        events.filter((key, value) -> value.getEventType().equals("PURCHASE"))
            .groupByKey()
            .count()
            .toStream()
            .to("user-purchase-counts");

        // 4. معدل التحويل (Conversion Rate)
        KTable<String, Long> views = events
            .filter((key, value) -> value.getEventType().equals("VIEW"))
            .groupBy((key, value) -> value.getPage())
            .count();

        KTable<String, Long> purchases = events
            .filter((key, value) -> value.getEventType().equals("PURCHASE"))
            .groupBy((key, value) -> value.getPage())
            .count();

        // Join views and purchases
        views.join(purchases,
            (viewCount, purchaseCount) -> {
                double conversionRate = (double) purchaseCount / viewCount * 100;
                return String.format("%.2f%%", conversionRate);
            })
            .toStream()
            .to("conversion-rates");

        // 5. نافذة زمنية - الأحداث كل 30 ثانية
        events.groupBy((key, value) -> value.getEventType())
            .windowedBy(TimeWindows.of(Duration.ofSeconds(30)))
            .count()
            .toStream()
            .map((key, value) -> KeyValue.pair(
                key.key() + "@" + key.window().start(),
                value
            ))
            .to("events-per-30-seconds");

        KafkaStreams streams = new KafkaStreams(builder.build(), props);
        streams.start();

        Runtime.getRuntime().addShutdownHook(new Thread(streams::close));
    }
}
```

---

## Best Practices

### 1. Producer Best Practices

```java
// ✅ استخدم idempotence
props.put("enable.idempotence", true);

// ✅ استخدم compression
props.put("compression.type", "snappy");

// ✅ استخدم batching
props.put("batch.size", 16384);
props.put("linger.ms", 10);

// ✅ handle errors properly
producer.send(record, (metadata, exception) -> {
    if (exception != null) {
        // log error and retry
        logger.error("Failed to send message", exception);
    }
});

// ✅ close producer properly
Runtime.getRuntime().addShutdownHook(new Thread(() -> {
    producer.close(Duration.ofSeconds(10));
}));
```

### 2. Consumer Best Practices

```java
// ✅ استخدم consumer group
props.put("group.id", "my-consumer-group");

// ✅ handle rebalancing
consumer.subscribe(topics, new ConsumerRebalanceListener() {
    // ...
});

// ✅ commit offsets properly
consumer.commitAsync((offsets, exception) -> {
    if (exception != null) {
        consumer.commitSync();  // retry with sync
    }
});

// ✅ handle errors and retry
try {
    processRecord(record);
} catch (Exception e) {
    // send to DLQ (Dead Letter Queue)
    sendToDeadLetterQueue(record);
}
```

### 3. Topic Design

```bash
# ✅ عدد مناسب من Partitions
# Rule of thumb: throughput / partition throughput
# مثال: 1GB/s ÷ 50MB/s = 20 partitions

# ✅ Replication factor >= 2
--replication-factor 3

# ✅ Retention policy
--config retention.ms=604800000  # 7 days
--config retention.bytes=1073741824  # 1GB

# ✅ Cleanup policy
--config cleanup.policy=delete  # or compact
```

### 4. Performance Tuning

```java
// Producer
props.put("acks", "1");  // balance between speed and reliability
props.put("compression.type", "lz4");  // أسرع من gzip
props.put("batch.size", 32768);
props.put("buffer.memory", 67108864);  // 64MB

// Consumer
props.put("fetch.min.bytes", "1024");
props.put("fetch.max.wait.ms", "500");
props.put("max.poll.records", "500");
```

---

## Monitoring و Troubleshooting

### Metrics المهمة

```java
// Producer Metrics
- record-send-rate
- record-error-rate
- request-latency-avg
- batch-size-avg
- compression-rate-avg

// Consumer Metrics
- records-consumed-rate
- fetch-latency-avg
- records-lag-max
- commit-latency-avg

// Broker Metrics
- MessagesInPerSec
- BytesInPerSec
- BytesOutPerSec
- UnderReplicatedPartitions
- ISRShrinkRate
```

### Common Issues

```bash
# 1. Consumer Lag
# الحل: زيادة عدد consumers أو partitions

# 2. Rebalancing كثير
# الحل: زيادة session.timeout.ms و max.poll.interval.ms

# 3. Out of Memory
# الحل: تقليل batch.size و buffer.memory

# 4. Data Loss
# الحل: استخدام acks=all و min.insync.replicas=2
```

---

## الخلاصة

### متى تستخدم Kafka؟

✅ **Real-time data streaming** - بث البيانات الحية
✅ **Event-driven architecture** - بنية قائمة على الأحداث
✅ **Log aggregation** - تجميع السجلات
✅ **Microservices communication** - التواصل بين الخدمات
✅ **Activity tracking** - تتبع النشاط
✅ **Metrics collection** - جمع المقاييس

### المفاهيم الرئيسية

🎯 **Topic** - فئة الرسائل
🎯 **Partition** - تقسيم للتوازي
🎯 **Offset** - رقم تسلسلي
🎯 **Producer** - يرسل الرسائل
🎯 **Consumer** - يقرأ الرسائل
🎯 **Consumer Group** - مجموعة مستهلكين
🎯 **Broker** - خادم Kafka

### الأدوات

🔧 **Producer API** - إرسال الرسائل
🔧 **Consumer API** - قراءة الرسائل
🔧 **Streams API** - معالجة البيانات
🔧 **Connect API** - التكامل مع أنظمة خارجية

---

**تم إنشاء هذا الدليل كمرجع شامل لـ Apache Kafka**
