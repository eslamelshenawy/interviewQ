# أسئلة المقابلات الشخصية عن Apache Kafka

## المحتويات
1. [أسئلة المستوى الأساسي (Beginner)](#أسئلة-المستوى-الأساسي-beginner)
2. [أسئلة المستوى المتوسط (Intermediate)](#أسئلة-المستوى-المتوسط-intermediate)
3. [أسئلة المستوى المتقدم (Advanced)](#أسئلة-المستوى-المتقدم-advanced)
4. [أسئلة عملية (Practical)](#أسئلة-عملية-practical)
5. [أسئلة التصميم (Design Questions)](#أسئلة-التصميم-design-questions)

---

## أسئلة المستوى الأساسي (Beginner)

### السؤال 1: ما هو Apache Kafka؟

**الإجابة:**
Apache Kafka هو **distributed streaming platform** يُستخدم لبناء تطبيقات real-time data pipelines و streaming applications.

**المميزات الرئيسية:**
- **High Throughput** - معالجة ملايين الرسائل في الثانية
- **Scalability** - قابل للتوسع أفقياً
- **Durability** - حفظ البيانات بشكل دائم على القرص
- **Fault Tolerance** - تحمل الأعطال عبر Replication
- **Low Latency** - زمن استجابة أقل من 10ms

**الاستخدامات:**
```
✅ Messaging System
✅ Activity Tracking
✅ Log Aggregation
✅ Stream Processing
✅ Event Sourcing
✅ Microservices Communication
```

---

### السؤال 2: اشرح المكونات الأساسية في Kafka

**الإجابة:**

#### 1. **Topic (الموضوع)**
```
فئة أو تصنيف للرسائل
مثال: orders, user-clicks, payments
```

#### 2. **Partition (التقسيم)**
```
Topic مقسم إلى partitions للتوازي

Topic: orders
├── Partition 0
├── Partition 1
└── Partition 2
```

#### 3. **Producer (المنتج)**
```java
// يرسل الرسائل إلى Topics
KafkaProducer<String, String> producer = new KafkaProducer<>(props);
producer.send(new ProducerRecord<>("topic", "key", "value"));
```

#### 4. **Consumer (المستهلك)**
```java
// يقرأ الرسائل من Topics
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(Arrays.asList("topic"));
```

#### 5. **Broker**
```
خادم Kafka - يخزن البيانات ويخدم الـ clients
```

#### 6. **Offset**
```
رقم تسلسلي فريد لكل رسالة داخل Partition
Offset 0, 1, 2, 3, ...
```

---

### السؤال 3: ما الفرق بين Kafka و Traditional Message Queue (RabbitMQ)؟

**الإجابة:**

| Feature | Kafka | RabbitMQ |
|---------|-------|----------|
| **النوع** | Distributed Log | Message Queue |
| **الاستهلاك** | Pull-based | Push-based |
| **التخزين** | على القرص (دائم) | في الذاكرة (مؤقت) |
| **الترتيب** | مضمون داخل Partition | مضمون في Queue |
| **المستهلكين** | Multiple consumers (Consumer Groups) | Single consumer per message |
| **Replay** | ✅ ممكن (من أي offset) | ❌ غير ممكن |
| **الأداء** | Very High (millions/sec) | Medium (thousands/sec) |
| **Use Case** | Streaming, Event Sourcing | Task Queues, RPC |

**متى تستخدم Kafka:**
- Real-time streaming
- Event sourcing
- Log aggregation
- High throughput

**متى تستخدم RabbitMQ:**
- Task queues
- Request-reply patterns
- Complex routing

---

### السؤال 4: ما هو Offset وكيف يعمل؟

**الإجابة:**

**Offset** هو رقم تسلسلي فريد لكل رسالة داخل Partition.

```
Partition 0:
┌─────────────────────────────────────┐
│ Offset 0 │ Offset 1 │ Offset 2 │ ... │
│ "msg1"   │ "msg2"   │ "msg3"   │     │
└─────────────────────────────────────┘
              ↑
         Consumer at offset 1
```

**أنواع Offsets:**

1. **Current Offset** - آخر رسالة قرأها Consumer
2. **Committed Offset** - آخر offset تم commit له
3. **Log-End Offset** - آخر رسالة في Partition

**Offset Management:**

```java
// Auto Commit (افتراضي)
props.put("enable.auto.commit", "true");
props.put("auto.commit.interval.ms", "5000");

// Manual Commit (أكثر أماناً)
props.put("enable.auto.commit", "false");
consumer.commitSync();  // أو commitAsync()
```

**Auto Offset Reset:**

```java
// earliest - من البداية
props.put("auto.offset.reset", "earliest");

// latest - من الآخر (الجديد فقط)
props.put("auto.offset.reset", "latest");

// none - error إذا لم يوجد offset
props.put("auto.offset.reset", "none");
```

---

### السؤال 5: اشرح Consumer Groups

**الإجابة:**

**Consumer Group** هو مجموعة من consumers يعملون معاً لقراءة رسائل من topic.

**المبدأ الأساسي:**
```
كل partition يُقرأ بواسطة consumer واحد فقط في نفس الـ group
```

**مثال:**

```
Topic: orders (3 partitions)

Consumer Group: "group-1"
├── Consumer A → Partition 0
├── Consumer B → Partition 1
└── Consumer C → Partition 2

Consumer Group: "group-2"
├── Consumer X → Partition 0, 1
└── Consumer Y → Partition 2
```

**القواعد:**

1. **إذا عدد Consumers = عدد Partitions:**
   ```
   كل consumer يقرأ من partition واحد
   ✅ أفضل أداء
   ```

2. **إذا عدد Consumers < عدد Partitions:**
   ```
   بعض consumers يقرأون من أكثر من partition
   ⚠️ حمل غير متوازن
   ```

3. **إذا عدد Consumers > عدد Partitions:**
   ```
   بعض consumers سيكون idle (بدون عمل)
   ❌ هدر موارد
   ```

**الفوائد:**
- **Scalability** - إضافة consumers للتوسع
- **Fault Tolerance** - إذا فشل consumer، partitions توزع على الباقيين
- **Load Balancing** - توزيع الحمل تلقائياً

**Code Example:**

```java
// Consumer 1
props.put("group.id", "my-group");
consumer.subscribe(Arrays.asList("orders"));

// Consumer 2 (نفس الـ group)
props.put("group.id", "my-group");
consumer.subscribe(Arrays.asList("orders"));

// كل consumer سيقرأ من partitions مختلفة
```

---

### السؤال 6: ما الفرق بين Partition و Topic؟

**الإجابة:**

**Topic:**
- فئة أو تصنيف منطقي للرسائل
- مثل: "orders", "payments", "logs"

**Partition:**
- تقسيم فعلي للـ Topic
- كل partition هو ordered log من الرسائل

**العلاقة:**

```
Topic: orders
│
├── Partition 0: [msg1, msg2, msg3]
├── Partition 1: [msg4, msg5, msg6]
└── Partition 2: [msg7, msg8, msg9]
```

**لماذا Partitions؟**

1. **Scalability:**
   ```
   توزيع البيانات على عدة brokers
   ```

2. **Parallelism:**
   ```
   قراءة/كتابة متوازية من عدة consumers/producers
   ```

3. **Ordering:**
   ```
   الترتيب مضمون داخل partition (ليس across partitions)
   ```

**Partitioning Strategy:**

```java
// 1. بناءً على Key
record = new ProducerRecord<>("topic", "key123", "value");
// نفس الـ key → نفس الـ partition

// 2. Round Robin (بدون key)
record = new ProducerRecord<>("topic", "value");

// 3. Custom Partitioner
props.put("partitioner.class", MyCustomPartitioner.class.getName());
```

---

### السؤال 7: ما هو Replication Factor؟

**الإجابة:**

**Replication Factor** هو عدد النسخ الاحتياطية لكل partition.

**مثال:**

```bash
# إنشاء topic مع replication factor = 3
kafka-topics.sh --create \
  --topic orders \
  --partitions 3 \
  --replication-factor 3
```

**التوزيع:**

```
Partition 0:
├── Leader: Broker 1 (يخدم القراءة والكتابة)
├── Replica: Broker 2 (نسخة احتياطية)
└── Replica: Broker 3 (نسخة احتياطية)

إذا فشل Broker 1:
→ Broker 2 أو 3 يصبح Leader الجديد
```

**ISR (In-Sync Replicas):**
```
Replicas التي تطابق Leader بالكامل
```

**Best Practices:**

```bash
# Production
replication-factor = 3
min.insync.replicas = 2

# Development/Testing
replication-factor = 1
```

**Trade-offs:**

| Replication Factor | Pros | Cons |
|-------------------|------|------|
| **1** | أسرع، أقل تخزين | ❌ لا fault tolerance |
| **2** | توازن معقول | ⚠️ يتحمل فشل broker واحد |
| **3** | ✅ fault tolerance عالية | أبطأ، 3x تخزين |

---

### السؤال 8: اشرح Kafka Architecture بشكل عام

**الإجابة:**

```
┌─────────────────────────────────────────────┐
│            Kafka Cluster                     │
│                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │ Broker 1 │  │ Broker 2 │  │ Broker 3 │  │
│  │          │  │          │  │          │  │
│  │ Leader   │  │ Follower │  │ Follower │  │
│  │ Part 0,2 │  │ Part 1   │  │ Replicas │  │
│  └──────────┘  └──────────┘  └──────────┘  │
│         ↑           ↑           ↑           │
└─────────┼───────────┼───────────┼───────────┘
          │           │           │
     ┌────┴──────┬────┴─────┬─────┴────┐
     │           │          │          │
┌────▼────┐ ┌───▼────┐ ┌───▼────┐ ┌───▼────┐
│Producer │ │Producer│ │Consumer│ │Consumer│
│    1    │ │    2   │ │   1    │ │   2    │
└─────────┘ └────────┘ └────────┘ └────────┘
```

**المكونات:**

1. **Producers:**
   - يرسلون الرسائل إلى Topics
   - يختارون Partition (بناءً على key أو round-robin)

2. **Brokers:**
   - Kafka servers
   - يخزنون البيانات
   - يخدمون الـ producers/consumers

3. **Consumers:**
   - يقرؤون الرسائل من Topics
   - ينتظمون في Consumer Groups

4. **ZooKeeper (قديم) / KRaft (جديد):**
   - إدارة Cluster
   - Leader election
   - Metadata management

---

## أسئلة المستوى المتوسط (Intermediate)

### السؤال 9: اشرح الفرق بين commitSync() و commitAsync()

**الإجابة:**

#### 1. **commitSync() - Synchronous Commit**

```java
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));

    for (ConsumerRecord<String, String> record : records) {
        processRecord(record);
    }

    // Blocking - ينتظر حتى ينتهي الـ commit
    consumer.commitSync();
}
```

**المميزات:**
- ✅ يضمن أن الـ commit نجح
- ✅ أكثر أماناً

**العيوب:**
- ❌ Blocking - يوقف التنفيذ
- ❌ أبطأ

#### 2. **commitAsync() - Asynchronous Commit**

```java
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));

    for (ConsumerRecord<String, String> record : records) {
        processRecord(record);
    }

    // Non-blocking - يستمر فوراً
    consumer.commitAsync((offsets, exception) -> {
        if (exception != null) {
            System.err.println("Commit failed: " + exception.getMessage());
        }
    });
}
```

**المميزات:**
- ✅ Non-blocking
- ✅ أسرع

**العيوب:**
- ❌ لا يضمن النجاح
- ❌ قد يحدث commit خارج الترتيب

#### 3. **Best Practice - الدمج بينهما**

```java
try {
    while (true) {
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));

        for (ConsumerRecord<String, String> record : records) {
            processRecord(record);
        }

        // Async commit للسرعة
        consumer.commitAsync();
    }
} finally {
    try {
        // Sync commit عند الإغلاق للتأكد
        consumer.commitSync();
    } finally {
        consumer.close();
    }
}
```

---

### السؤال 10: ما هو Producer Acknowledgment (acks) وما الفرق بين acks=0, 1, all؟

**الإجابة:**

**acks** يحدد عدد التأكيدات التي ينتظرها Producer قبل اعتبار الرسالة مُرسلة بنجاح.

#### 1. **acks = 0 (No Acknowledgment)**

```java
props.put("acks", "0");
```

**السلوك:**
- Producer لا ينتظر أي تأكيد من Broker
- يرسل الرسالة ويستمر فوراً

**المميزات:**
- ✅ أسرع throughput
- ✅ Lowest latency

**العيوب:**
- ❌ قد تُفقد الرسائل
- ❌ لا ضمان للتسليم

**Use Case:**
```
Metrics, logs (بيانات غير حساسة)
```

#### 2. **acks = 1 (Leader Acknowledgment)**

```java
props.put("acks", "1");
```

**السلوك:**
- Producer ينتظر تأكيد من Leader فقط
- Replicas قد لا تكون sync بعد

**المميزات:**
- ✅ توازن بين السرعة والأمان
- ✅ معظم الرسائل تُسلّم

**العيوب:**
- ❌ إذا فشل Leader قبل replication، تُفقد الرسالة

**Use Case:**
```
معظم التطبيقات (default)
```

#### 3. **acks = all / -1 (All In-Sync Replicas)**

```java
props.put("acks", "all");
props.put("min.insync.replicas", "2");  // في server.properties
```

**السلوك:**
- Producer ينتظر تأكيد من Leader وجميع ISR
- أقصى درجة أمان

**المميزات:**
- ✅ لا فقدان للبيانات
- ✅ Strongest durability guarantee

**العيوب:**
- ❌ أبطأ
- ❌ Higher latency

**Use Case:**
```
بيانات حساسة (payments, orders)
```

**المقارنة:**

| acks | Durability | Performance | Data Loss Risk |
|------|-----------|-------------|----------------|
| **0** | ❌ None | ⚡ Fastest | ⚠️ High |
| **1** | ⚠️ Medium | ✅ Fast | ⚠️ Medium |
| **all** | ✅ Strong | ⚠️ Slower | ✅ Very Low |

---

### السؤال 11: كيف تضمن Exactly-Once Semantics في Kafka؟

**الإجابة:**

**Exactly-Once Semantics** تضمن أن كل رسالة تُعالج مرة واحدة فقط (لا تكرار ولا فقدان).

#### 1. **Producer Idempotence**

```java
// تفعيل Idempotence
props.put("enable.idempotence", "true");

// يتطلب:
props.put("acks", "all");
props.put("retries", Integer.MAX_VALUE);
props.put("max.in.flight.requests.per.connection", "5");
```

**كيف يعمل:**
- Producer يضيف sequence number لكل رسالة
- Broker يرفض الرسائل المكررة

**مثال:**
```
Producer يرسل msg1 (seq=0)
Network error → Producer يعيد الإرسال
Broker يكتشف التكرار (seq=0 موجود) → يرفض
```

#### 2. **Transactional Producer**

```java
// إعداد Transactions
props.put("transactional.id", "my-transactional-id");
props.put("enable.idempotence", "true");

KafkaProducer<String, String> producer = new KafkaProducer<>(props);

// بدء Transaction
producer.initTransactions();

try {
    producer.beginTransaction();

    // إرسال رسائل
    producer.send(new ProducerRecord<>("topic1", "key1", "value1"));
    producer.send(new ProducerRecord<>("topic2", "key2", "value2"));

    // Commit Transaction
    producer.commitTransaction();
} catch (Exception e) {
    // Rollback
    producer.abortTransaction();
}
```

#### 3. **Exactly-Once Processing (Streams)**

```java
Properties props = new Properties();
props.put(StreamsConfig.PROCESSING_GUARANTEE_CONFIG,
    StreamsConfig.EXACTLY_ONCE_V2);

StreamsBuilder builder = new StreamsBuilder();
// ...
KafkaStreams streams = new KafkaStreams(builder.build(), props);
streams.start();
```

#### 4. **Consumer + External System (Manual)**

```java
// استخدام Database Transaction
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));

    for (ConsumerRecord<String, String> record : records) {
        // بدء transaction
        db.beginTransaction();

        try {
            // معالجة الرسالة
            processRecord(record);

            // حفظ الـ offset في نفس الـ transaction
            saveOffset(record.topic(), record.partition(), record.offset());

            // Commit
            db.commit();
        } catch (Exception e) {
            db.rollback();
        }
    }
}
```

**الخلاصة:**

| Mechanism | Use Case |
|-----------|----------|
| **Idempotence** | منع تكرار الرسائل في Producer |
| **Transactions** | عمليات atomic عبر عدة topics |
| **Streams Exactly-Once** | معالجة stream processing |
| **Manual (DB)** | تكامل مع أنظمة خارجية |

---

### السؤال 12: ما هو Consumer Rebalancing وكيف يعمل؟

**الإجابة:**

**Rebalancing** هو إعادة توزيع الـ partitions على الـ consumers عند تغيير في Consumer Group.

#### متى يحدث Rebalancing؟

1. **إضافة consumer جديد**
2. **إزالة/فشل consumer**
3. **تغيير في عدد partitions**
4. **Consumer لم يرسل heartbeat (session timeout)**

#### كيف يعمل؟

```
قبل Rebalancing:
Consumer Group (3 consumers, 6 partitions)
├── Consumer A → P0, P1
├── Consumer B → P2, P3
└── Consumer C → P4, P5

Consumer B يفشل →

بعد Rebalancing:
├── Consumer A → P0, P1, P2
└── Consumer C → P3, P4, P5
```

#### أنواع Rebalancing Strategies:

1. **Range (افتراضي)**
   ```
   توزيع متتالي
   C1: P0, P1
   C2: P2, P3
   ```

2. **RoundRobin**
   ```
   توزيع دائري
   C1: P0, P2, P4
   C2: P1, P3, P5
   ```

3. **Sticky**
   ```
   يحاول الحفاظ على الـ assignments القديمة
   ```

4. **CooperativeSticky (Kafka 2.4+)**
   ```
   Incremental rebalancing - أقل تعطيل
   ```

#### Handling Rebalancing:

```java
consumer.subscribe(
    Arrays.asList("orders"),
    new ConsumerRebalanceListener() {

        @Override
        public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
            System.out.println("Partitions revoked: " + partitions);

            // Commit offsets قبل فقدان الـ partitions
            consumer.commitSync();

            // تنظيف الموارد
            cleanup();
        }

        @Override
        public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
            System.out.println("Partitions assigned: " + partitions);

            // إعداد الموارد للـ partitions الجديدة
            initialize(partitions);
        }
    }
);
```

#### تقليل Rebalancing:

```java
// زيادة session timeout
props.put("session.timeout.ms", "30000");  // 30 ثانية

// زيادة max poll interval
props.put("max.poll.interval.ms", "600000");  // 10 دقائق

// تقليل heartbeat interval
props.put("heartbeat.interval.ms", "3000");  // 3 ثواني
```

#### المشاكل:

```
❌ Stop-the-world - جميع consumers تتوقف
❌ فقدان State (في بعض الحالات)
❌ تأخير في المعالجة
```

**Best Practice:**
```
استخدام CooperativeSticky في Kafka 2.4+
```

---

### السؤال 13: اشرح الفرق بين Stateless و Stateful Stream Processing

**الإجابة:**

#### 1. **Stateless Processing**

**التعريف:**
```
كل رسالة تُعالج بشكل مستقل
لا يحتاج لحفظ حالة سابقة
```

**أمثلة:**

```java
// Filter
stream.filter((key, value) -> value.length() > 10);

// Map
stream.map((key, value) -> KeyValue.pair(key, value.toUpperCase()));

// FlatMap
stream.flatMap((key, value) ->
    Arrays.stream(value.split(" "))
        .map(word -> KeyValue.pair(key, word))
        .collect(Collectors.toList())
);
```

**المميزات:**
- ✅ بسيط
- ✅ سريع
- ✅ لا يحتاج memory كبيرة
- ✅ سهل التوسع

#### 2. **Stateful Processing**

**التعريف:**
```
يحتاج لحفظ حالة (state) للمعالجة
مثل: aggregations, joins, windowing
```

**أمثلة:**

**A. Count Aggregation:**
```java
// عد الكلمات - يحتاج لحفظ العداد
KTable<String, Long> wordCounts = stream
    .flatMapValues(value -> Arrays.asList(value.split(" ")))
    .groupBy((key, word) -> word)
    .count();  // Stateful - يحفظ count لكل كلمة
```

**B. Windowing:**
```java
// عد الـ clicks كل 5 دقائق
KTable<Windowed<String>, Long> clicksPerWindow = stream
    .groupByKey()
    .windowedBy(TimeWindows.of(Duration.ofMinutes(5)))
    .count();  // Stateful - يحفظ count لكل نافذة
```

**C. Join:**
```java
// Join بين streams
KStream<String, String> joined = orders.join(
    payments,
    (order, payment) -> order + ":" + payment,
    JoinWindows.of(Duration.ofMinutes(5))
);  // Stateful - يحفظ البيانات للـ join
```

**D. Reduce:**
```java
// مجموع الأسعار
KTable<String, Double> totalPrice = stream
    .groupByKey()
    .reduce((value1, value2) -> value1 + value2);  // Stateful
```

#### State Stores:

```java
// Kafka يحفظ الـ state في State Stores
// - In-memory (سريع)
// - RocksDB (دائم)
// - Backed by changelog topic (للـ fault tolerance)

// مثال: Custom State Store
StoreBuilder<KeyValueStore<String, Long>> storeBuilder =
    Stores.keyValueStoreBuilder(
        Stores.persistentKeyValueStore("myStore"),
        Serdes.String(),
        Serdes.Long()
    );

builder.addStateStore(storeBuilder);
```

#### المقارنة:

| Feature | Stateless | Stateful |
|---------|-----------|----------|
| **State** | لا يحفظ | يحفظ state |
| **Memory** | قليل | كثير |
| **Complexity** | بسيط | معقد |
| **Performance** | أسرع | أبطأ |
| **Fault Tolerance** | سهل | يحتاج changelog |
| **Examples** | filter, map | count, join, windowing |

---

### السؤال 14: كيف تتعامل مع Large Messages في Kafka؟

**الإجابة:**

Kafka مصمم للرسائل الصغيرة (< 1MB). للرسائل الكبيرة، هناك عدة استراتيجيات:

#### 1. **زيادة Limits (غير مُفضل)**

```java
// Producer
props.put("max.request.size", "10485760");  // 10MB

// Broker (server.properties)
message.max.bytes=10485760
replica.fetch.max.bytes=10485760

// Consumer
props.put("max.partition.fetch.bytes", "10485760");
```

**المشاكل:**
- ❌ يبطئ الـ cluster
- ❌ يزيد الـ memory usage
- ❌ يزيد الـ network load

#### 2. **Chunking (التقسيم)**

```java
public class MessageChunker {
    private static final int CHUNK_SIZE = 900_000; // 900KB

    public void sendLargeMessage(String key, byte[] largeData) {
        String messageId = UUID.randomUUID().toString();
        int totalChunks = (int) Math.ceil((double) largeData.length / CHUNK_SIZE);

        for (int i = 0; i < totalChunks; i++) {
            int start = i * CHUNK_SIZE;
            int end = Math.min(start + CHUNK_SIZE, largeData.length);
            byte[] chunk = Arrays.copyOfRange(largeData, start, end);

            ChunkMetadata metadata = new ChunkMetadata(
                messageId, i, totalChunks, chunk
            );

            producer.send(new ProducerRecord<>(
                "chunked-messages", key, metadata
            ));
        }
    }
}

// Consumer - إعادة التجميع
public class MessageReassembler {
    private Map<String, List<ChunkMetadata>> chunksMap = new HashMap<>();

    public byte[] reassemble(ChunkMetadata chunk) {
        String messageId = chunk.getMessageId();

        chunksMap.putIfAbsent(messageId, new ArrayList<>());
        chunksMap.get(messageId).add(chunk);

        if (chunksMap.get(messageId).size() == chunk.getTotalChunks()) {
            // جميع الـ chunks وصلت
            byte[] fullMessage = combineChunks(chunksMap.get(messageId));
            chunksMap.remove(messageId);
            return fullMessage;
        }

        return null;  // انتظار باقي الـ chunks
    }
}
```

#### 3. **External Storage (الأفضل)**

```java
// تخزين البيانات الكبيرة خارجياً (S3, HDFS, etc.)
public class LargeMessageProducer {
    private AmazonS3 s3Client;

    public void sendLargeMessage(String key, byte[] largeData) {
        // رفع إلى S3
        String s3Key = "messages/" + UUID.randomUUID().toString();
        s3Client.putObject("my-bucket", s3Key, largeData);

        // إرسال reference فقط إلى Kafka
        MessageReference reference = new MessageReference(
            s3Key,
            largeData.length,
            "s3://my-bucket/" + s3Key
        );

        producer.send(new ProducerRecord<>(
            "message-references", key, reference
        ));
    }
}

// Consumer - تحميل من S3
public class LargeMessageConsumer {
    private AmazonS3 s3Client;

    public void processMessage(MessageReference reference) {
        // تحميل من S3
        S3Object object = s3Client.getObject(
            "my-bucket", reference.getS3Key()
        );

        byte[] data = IOUtils.toByteArray(object.getObjectContent());

        // معالجة البيانات
        process(data);

        // حذف من S3 (اختياري)
        s3Client.deleteObject("my-bucket", reference.getS3Key());
    }
}
```

#### 4. **Compression**

```java
// ضغط البيانات قبل الإرسال
props.put("compression.type", "lz4");  // أو snappy, gzip, zstd

// Custom compression
public byte[] compressData(byte[] data) {
    ByteArrayOutputStream baos = new ByteArrayOutputStream();
    try (GZIPOutputStream gzos = new GZIPOutputStream(baos)) {
        gzos.write(data);
    }
    return baos.toByteArray();
}
```

#### الخلاصة:

| Method | Pros | Cons | Use Case |
|--------|------|------|----------|
| **زيادة Limits** | سهل | ❌ يبطئ الـ cluster | Temporary only |
| **Chunking** | لا حدود للحجم | معقد | Medium files |
| **External Storage** | ✅ الأفضل | يحتاج infrastructure | Large files |
| **Compression** | تقليل الحجم | CPU overhead | Text/JSON data |

---

### السؤال 15: اشرح Kafka Streams Windowing Types

**الإجابة:**

**Windowing** يقسم الـ stream إلى نوافذ زمنية للمعالجة والتجميع.

#### 1. **Tumbling Window (نافذة ثابتة)**

```
النوافذ لا تتداخل، كل رسالة تنتمي لنافذة واحدة فقط

Timeline: ─────────────────────────────────→
Window 1: [════════]
Window 2:          [════════]
Window 3:                   [════════]
```

**Code:**
```java
KTable<Windowed<String>, Long> counts = stream
    .groupByKey()
    .windowedBy(TimeWindows.of(Duration.ofMinutes(5)))
    .count();

// كل 5 دقائق نافذة جديدة
// 00:00-00:05, 00:05-00:10, 00:10-00:15, ...
```

**Use Case:**
```
- عدد الـ orders كل ساعة
- إجمالي المبيعات كل يوم
```

#### 2. **Hopping Window (نافذة متداخلة)**

```
النوافذ تتداخل، الرسالة قد تنتمي لعدة نوافذ

Window size = 10min, Advance = 5min

Timeline: ─────────────────────────────────→
Window 1: [════════════]
Window 2:      [════════════]
Window 3:           [════════════]
```

**Code:**
```java
KTable<Windowed<String>, Long> counts = stream
    .groupByKey()
    .windowedBy(
        TimeWindows.of(Duration.ofMinutes(10))
            .advanceBy(Duration.ofMinutes(5))
    )
    .count();

// Windows:
// 00:00-00:10
// 00:05-00:15
// 00:10-00:20
```

**Use Case:**
```
- Moving average
- Trend analysis
```

#### 3. **Sliding Window**

```
نافذة تتحرك مع كل رسالة

Message 1 at 00:00 → Window: 00:00-00:05
Message 2 at 00:01 → Window: 00:01-00:06
```

**Code:**
```java
// Join مع sliding window
KStream<String, String> joined = orders.join(
    payments,
    (order, payment) -> order + ":" + payment,
    JoinWindows.of(Duration.ofMinutes(5))
);

// كل order يبحث عن payment خلال ±5 دقائق
```

**Use Case:**
```
- Stream joins
- Event correlation
```

#### 4. **Session Window (نافذة الجلسة)**

```
تعتمد على النشاط، تنتهي بعد فترة inactivity

User Activity:
Event 1 ───── Event 2 ── Event 3 ─────────────── Event 4
         2min        1min         10min gap

Session 1: [Event 1, Event 2, Event 3]
Session 2: [Event 4]
```

**Code:**
```java
KTable<Windowed<String>, Long> sessionCounts = stream
    .groupByKey()
    .windowedBy(SessionWindows.with(Duration.ofMinutes(5)))
    .count();

// إذا مر 5 دقائق بدون events، الجلسة تنتهي
```

**Use Case:**
```
- User sessions على website
- IoT device activity
```

#### مثال عملي - Website Analytics:

```java
// عد الـ page views
public class WebAnalytics {
    public static void main(String[] args) {
        StreamsBuilder builder = new StreamsBuilder();

        KStream<String, PageView> pageViews = builder.stream("page-views");

        // 1. Tumbling - عدد الـ views كل ساعة
        KTable<Windowed<String>, Long> hourlyViews = pageViews
            .groupBy((key, view) -> view.getPage())
            .windowedBy(TimeWindows.of(Duration.ofHours(1)))
            .count();

        // 2. Hopping - Moving average لآخر 10 دقائق، كل دقيقة
        KTable<Windowed<String>, Long> movingAvg = pageViews
            .groupBy((key, view) -> view.getPage())
            .windowedBy(
                TimeWindows.of(Duration.ofMinutes(10))
                    .advanceBy(Duration.ofMinutes(1))
            )
            .count();

        // 3. Session - جلسات المستخدم
        KTable<Windowed<String>, Long> userSessions = pageViews
            .groupBy((key, view) -> view.getUserId())
            .windowedBy(SessionWindows.with(Duration.ofMinutes(30)))
            .count();

        // Write results
        hourlyViews.toStream().to("hourly-views");
        movingAvg.toStream().to("moving-average");
        userSessions.toStream().to("user-sessions");
    }
}
```

#### Window Retention:

```java
// حفظ النوافذ لمدة معينة
props.put(
    StreamsConfig.WINDOW_STORE_CHANGE_LOG_ADDITIONAL_RETENTION_MS_CONFIG,
    Duration.ofDays(7).toMillis()
);
```

---

## أسئلة المستوى المتقدم (Advanced)

### السؤال 16: كيف تتعامل مع Consumer Lag وتحلها؟

**الإجابة:**

**Consumer Lag** هو الفرق بين آخر رسالة في partition (log-end-offset) وآخر رسالة قرأها consumer (current-offset).

#### قياس Consumer Lag:

```bash
# CLI
kafka-consumer-groups.sh \
  --bootstrap-server localhost:9092 \
  --group my-group \
  --describe

# Output:
# TOPIC    PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
# orders   0          1000            1500            500  ← Lag
# orders   1          2000            2100            100
```

**Monitoring:**
```java
// JMX Metrics
records-lag-max
records-lag-avg
records-lead-min

// Programmatic
Map<TopicPartition, Long> endOffsets = consumer.endOffsets(partitions);
Map<TopicPartition, Long> currentOffsets = consumer.position(partition);
long lag = endOffsets.get(partition) - currentOffsets.get(partition);
```

#### أسباب Consumer Lag:

1. **معالجة بطيئة**
   ```java
   // كل رسالة تأخذ وقت طويل
   for (ConsumerRecord record : records) {
       slowProcessing(record);  // 100ms لكل رسالة
   }
   ```

2. **عدد consumers قليل**
   ```
   3 partitions ولكن 1 consumer فقط
   ```

3. **Producer أسرع من Consumer**
   ```
   Producer: 10,000 msg/sec
   Consumer: 1,000 msg/sec
   ```

4. **Network/Infrastructure issues**

#### الحلول:

#### 1. **زيادة عدد Consumers**

```java
// من 1 consumer إلى 3
// Consumer 1
props.put("group.id", "my-group");
consumer.subscribe(Arrays.asList("orders"));

// Consumer 2 (نفس الـ group)
props.put("group.id", "my-group");
consumer.subscribe(Arrays.asList("orders"));

// Consumer 3
props.put("group.id", "my-group");
consumer.subscribe(Arrays.asList("orders"));

// الآن كل consumer يعالج partition واحد
```

#### 2. **زيادة عدد Partitions**

```bash
# من 3 إلى 9 partitions
kafka-topics.sh --alter \
  --topic orders \
  --partitions 9

# الآن يمكن إضافة 9 consumers
```

#### 3. **تحسين المعالجة (Batch Processing)**

```java
// Before (بطيء)
for (ConsumerRecord record : records) {
    database.insert(record);  // insert واحد لكل رسالة
}

// After (أسرع)
List<Record> batch = new ArrayList<>();
for (ConsumerRecord record : records) {
    batch.add(record);

    if (batch.size() >= 100) {
        database.batchInsert(batch);  // batch insert
        batch.clear();
    }
}
```

#### 4. **Parallel Processing داخل Consumer**

```java
ExecutorService executor = Executors.newFixedThreadPool(10);

while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));

    for (ConsumerRecord<String, String> record : records) {
        executor.submit(() -> {
            processRecord(record);
        });
    }

    // انتظار انتهاء المعالجة
    executor.shutdown();
    executor.awaitTermination(1, TimeUnit.MINUTES);

    consumer.commitSync();
}
```

#### 5. **تحسين Configurations**

```java
// زيادة حجم الـ batch
props.put("max.poll.records", "1000");  // من 500 إلى 1000

// تقليل polling interval
props.put("max.poll.interval.ms", "600000");  // 10 دقائق

// زيادة fetch size
props.put("fetch.min.bytes", "50000");
props.put("fetch.max.wait.ms", "100");
```

#### 6. **Auto-Scaling (Kubernetes/Cloud)**

```yaml
# Kubernetes HPA
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: kafka-consumer
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: kafka-consumer
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: External
    external:
      metric:
        name: kafka_consumer_lag
      target:
        type: Value
        value: "1000"  # Scale إذا lag > 1000
```

#### 7. **Dead Letter Queue (DLQ)**

```java
// إرسال الرسائل المعطوبة لـ DLQ
for (ConsumerRecord record : records) {
    try {
        processRecord(record);
    } catch (Exception e) {
        // إرسال لـ DLQ بدلاً من retry
        producer.send(new ProducerRecord<>("dlq-topic", record));
        logger.error("Failed to process, sent to DLQ", e);
    }
}

// المعالجة تستمر بدون توقف
```

#### Monitoring Dashboard:

```java
// Prometheus + Grafana
// Metrics:
- kafka_consumer_lag (الأهم)
- kafka_consumer_records_consumed_rate
- kafka_consumer_fetch_latency_avg
- processing_time_per_record

// Alerts:
- Lag > 10,000 → Warning
- Lag > 100,000 → Critical
```

---

### السؤال 17: اشرح Kafka Log Compaction وكيف يعمل؟

**الإجابة:**

**Log Compaction** هو آلية للاحتفاظ بآخر قيمة لكل key، وحذف القيم القديمة.

#### كيف يعمل؟

**Before Compaction:**
```
Offset: 0    1    2    3    4    5    6    7
Key:    A    B    A    C    B    A    D    C
Value:  1    2    3    4    5    6    7    8
```

**After Compaction:**
```
Offset: 5    6    7
Key:    A    D    C
Value:  6    7    8

- Key A: آخر قيمة = 6 (offset 5)
- Key B: حُذفت (آخر قيمة 5 في offset 4، لكن A بعدها)
- Key C: آخر قيمة = 8 (offset 7)
- Key D: آخر قيمة = 7 (offset 6)
```

#### Configuration:

```bash
# إنشاء topic مع compaction
kafka-topics.sh --create \
  --topic user-profiles \
  --partitions 3 \
  --replication-factor 2 \
  --config cleanup.policy=compact \
  --config min.cleanable.dirty.ratio=0.5 \
  --config segment.ms=100 \
  --config delete.retention.ms=86400000

# cleanup.policy options:
# - delete (افتراضي): حذف بناءً على time/size
# - compact: log compaction
# - compact,delete: كلاهما
```

#### Structure:

```
Partition:
├── Head (Clean) - بعد Compaction
│   ├── Segment 1 (compacted)
│   └── Segment 2 (compacted)
│
└── Tail (Dirty) - قبل Compaction
    ├── Segment 3 (active)
    └── Segment 4 (waiting for compaction)
```

#### Use Cases:

#### 1. **Database Change Capture (CDC)**

```java
// تخزين آخر حالة لكل record في DB
public class UserProfileProducer {
    public void updateUserProfile(String userId, UserProfile profile) {
        ProducerRecord<String, UserProfile> record =
            new ProducerRecord<>(
                "user-profiles",  // compacted topic
                userId,           // key
                profile           // value
            );

        producer.send(record);
    }
}

// Consumer يحصل على آخر حالة فقط
// مفيد لبناء local cache أو materialized view
```

#### 2. **Configuration Management**

```java
// تخزين configurations
// Key: config-name
// Value: config-value

producer.send(new ProducerRecord<>("app-config", "db.url", "localhost:3306"));
producer.send(new ProducerRecord<>("app-config", "db.url", "prod-db:3306"));

// بعد compaction، فقط "prod-db:3306" موجود
```

#### 3. **Event Sourcing (مع Snapshots)**

```java
// تخزين events + snapshots
// Key: aggregate-id
// Value: snapshot أو event

// Events
producer.send(new ProducerRecord<>("accounts", "ACC-123", "Created"));
producer.send(new ProducerRecord<>("accounts", "ACC-123", "Deposited 100"));
producer.send(new ProducerRecord<>("accounts", "ACC-123", "Withdrew 50"));

// Snapshot (يحل محل الـ events القديمة)
producer.send(new ProducerRecord<>("accounts", "ACC-123", "Snapshot: Balance=50"));

// بعد compaction، فقط الـ snapshot موجود
```

#### Tombstone Records (الحذف):

```java
// لحذف key نهائياً، نرسل null value
producer.send(new ProducerRecord<>("user-profiles", "user-123", null));

// بعد compaction + delete.retention.ms
// "user-123" يُحذف نهائياً
```

#### Compaction Process:

```
1. Kafka ينتظر حتى dirty ratio >= min.cleanable.dirty.ratio
2. Cleaner thread يبدأ compaction
3. يقرأ الـ dirty segments
4. يبني hash table للـ keys
5. ينسخ آخر قيمة لكل key إلى clean segment
6. يحذف الـ dirty segments القديمة
```

#### Configurations المهمة:

```properties
# server.properties

# تفعيل Compaction
log.cleaner.enable=true

# عدد الـ cleaner threads
log.cleaner.threads=2

# Minimum dirty ratio للبدء
log.cleaner.min.cleanable.ratio=0.5

# حذف tombstones بعد
log.cleaner.delete.retention.ms=86400000  # 1 day

# حجم الـ buffer
log.cleaner.dedupe.buffer.size=134217728  # 128MB
```

#### Monitoring:

```java
// JMX Metrics
kafka.log:type=LogCleaner,name=cleaner-recopy-percent
kafka.log:type=LogCleaner,name=max-dirty-percent
kafka.log:type=LogCleaner,name=time-since-last-run-ms
```

#### المقارنة:

| Feature | Delete | Compact |
|---------|--------|---------|
| **البيانات المحفوظة** | آخر N يوم/GB | آخر قيمة لكل key |
| **الحجم** | ينمو ثم يُحذف | ثابت تقريباً |
| **Use Case** | Logs, events | State, snapshots |
| **Performance** | أسرع | أبطأ (Compaction overhead) |

---

### السؤال 18: كيف تصمم Multi-Datacenter Kafka Setup؟

**الإجابة:**

#### الاستراتيجيات:

### 1. **Active-Passive (Disaster Recovery)**

```
Datacenter 1 (Active):
├── Kafka Cluster 1 (Primary)
│   ├── Broker 1, 2, 3
│   └── Producers/Consumers
│
Datacenter 2 (Passive):
└── Kafka Cluster 2 (Standby)
    ├── Broker 4, 5, 6
    └── Mirror Maker 2 (Replication)
```

**Configuration:**

```properties
# MirrorMaker 2 Config
clusters = primary, backup
primary.bootstrap.servers = dc1-broker1:9092,dc1-broker2:9092
backup.bootstrap.servers = dc2-broker1:9092,dc2-broker2:9092

primary->backup.enabled = true
primary->backup.topics = .*  # replicate all topics

# Replication settings
replication.factor = 3
offset.sync.enabled = true
```

**Failover Process:**

```bash
# 1. عند فشل DC1
# 2. تحويل traffic إلى DC2
# 3. DC2 يصبح active

# 4. Manual failover
kafka-reassign-partitions.sh \
  --bootstrap-server dc2-broker1:9092 \
  --reassignment-json-file failover.json \
  --execute
```

### 2. **Active-Active (Bi-directional Replication)**

```
Datacenter 1:                Datacenter 2:
├── Cluster 1                ├── Cluster 2
├── Producers/Consumers      ├── Producers/Consumers
└── MirrorMaker 2 →          └── MirrorMaker 2 →
         ↓                            ↓
         ←──── Replication ────────→
```

**Configuration:**

```properties
# Bi-directional MirrorMaker
clusters = dc1, dc2

dc1.bootstrap.servers = dc1-kafka:9092
dc2.bootstrap.servers = dc2-kafka:9092

# DC1 → DC2
dc1->dc2.enabled = true
dc1->dc2.topics = .*

# DC2 → DC1
dc2->dc1.enabled = true
dc2->dc1.topics = .*

# Prevent loops
replication.policy.separator = .
dc1->dc2.emit.heartbeats = true
```

**Handling Conflicts:**

```java
// Conflict Resolution Strategy
// 1. Last-Write-Wins (LWW)
record.timestamp()  // use timestamp to resolve

// 2. Application-level
if (record.topic().startsWith("dc1.")) {
    // من DC1
} else if (record.topic().startsWith("dc2.")) {
    // من DC2
}

// 3. Custom logic
resolvConflict(record1, record2);
```

### 3. **Stretched Cluster (NOT Recommended)**

```
⚠️ لا يُنصح به - High latency issues

Cluster spans multiple DCs:
├── DC1: Broker 1, 2
└── DC2: Broker 3, 4

المشاكل:
- High latency للـ replication
- Network partitions
- Split-brain scenarios
```

### 4. **Confluent Replicator (Commercial)**

```properties
# أفضل من MirrorMaker 2
# Features:
- Exactly-once delivery
- Auto topic creation
- Timestamp preservation
- Consumer offset translation

# Config
confluent.topic.replication.factor=3
src.kafka.bootstrap.servers=dc1:9092
dest.kafka.bootstrap.servers=dc2:9092
topic.whitelist=.*
```

#### Best Practices:

**1. Network:**
```
✅ Dedicated network link بين DCs
✅ Bandwidth >= expected throughput × replication factor
✅ Low latency (< 50ms)
```

**2. Topic Naming:**
```java
// Prefix topics بالـ datacenter
// DC1: dc1.orders, dc1.payments
// DC2: dc2.orders, dc2.payments

// Consumer يقرأ من both
consumer.subscribe(Pattern.compile("(dc1|dc2)\\.orders"));
```

**3. Monitoring:**
```java
// Metrics
- Replication lag
- Network throughput
- Failover time
- Data consistency

// Alerts
if (replicationLag > threshold) {
    alert("Replication lag high");
}
```

**4. Testing:**
```bash
# Chaos Engineering
# 1. Network partition test
# 2. Datacenter failure simulation
# 3. Failover drill
# 4. Data validation
```

#### مثال: Complete Setup

```yaml
# docker-compose.yml
version: '3'
services:
  # DC1 Cluster
  dc1-zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181

  dc1-kafka:
    image: confluentinc/cp-kafka:7.5.0
    depends_on:
      - dc1-zookeeper
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: dc1-zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://dc1-kafka:9092

  # DC2 Cluster
  dc2-zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2182

  dc2-kafka:
    image: confluentinc/cp-kafka:7.5.0
    depends_on:
      - dc2-zookeeper
    environment:
      KAFKA_BROKER_ID: 2
      KAFKA_ZOOKEEPER_CONNECT: dc2-zookeeper:2182
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://dc2-kafka:9093

  # MirrorMaker 2
  mirror-maker:
    image: confluentinc/cp-kafka:7.5.0
    depends_on:
      - dc1-kafka
      - dc2-kafka
    command: >
      connect-mirror-maker
      /etc/kafka/mm2.properties
    volumes:
      - ./mm2.properties:/etc/kafka/mm2.properties
```

---

## أسئلة عملية (Practical)

### السؤال 19: صمم نظام Order Processing باستخدام Kafka

**الإجابة:**

#### Architecture:

```
┌──────────┐
│ Frontend │
└────┬─────┘
     │ REST API
     ↓
┌────────────────┐
│ Order Service  │
└────┬───────────┘
     │ Produce
     ↓
┌─────────────────────────────────────────┐
│           Kafka Topics                   │
│                                          │
│  orders → order-validation → payments   │
│              ↓                    ↓      │
│         inventory-check      notifications│
└─────────────────────────────────────────┘
     ↓           ↓           ↓          ↓
┌─────────┐ ┌─────────┐ ┌────────┐ ┌──────────┐
│Validator│ │Inventory│ │Payment │ │Notifier  │
│ Service │ │ Service │ │Service │ │Service   │
└─────────┘ └─────────┘ └────────┘ └──────────┘
```

#### Implementation:

**1. Order Service (Producer):**

```java
@Service
public class OrderService {
    @Autowired
    private KafkaTemplate<String, Order> kafkaTemplate;

    public OrderResponse createOrder(OrderRequest request) {
        // إنشاء Order
        Order order = new Order(
            UUID.randomUUID().toString(),
            request.getCustomerId(),
            request.getItems(),
            "PENDING"
        );

        // إرسال إلى Kafka
        kafkaTemplate.send("orders", order.getId(), order)
            .addCallback(
                success -> log.info("Order sent: {}", order.getId()),
                failure -> log.error("Failed to send order", failure)
            );

        return new OrderResponse(order.getId(), "Order received");
    }
}
```

**2. Validation Service (Consumer → Producer):**

```java
@Service
public class OrderValidationService {
    @Autowired
    private KafkaTemplate<String, Order> kafkaTemplate;

    @KafkaListener(topics = "orders", groupId = "validation-service")
    public void validateOrder(Order order) {
        log.info("Validating order: {}", order.getId());

        // التحقق من البيانات
        if (isValid(order)) {
            order.setStatus("VALIDATED");
            kafkaTemplate.send("order-validation", order.getId(), order);
        } else {
            order.setStatus("INVALID");
            kafkaTemplate.send("order-failed", order.getId(), order);
        }
    }

    private boolean isValid(Order order) {
        // Business logic validation
        return order.getItems() != null && !order.getItems().isEmpty();
    }
}
```

**3. Inventory Service:**

```java
@Service
public class InventoryService {
    @Autowired
    private KafkaTemplate<String, Order> kafkaTemplate;

    @Autowired
    private InventoryRepository inventoryRepo;

    @KafkaListener(topics = "order-validation", groupId = "inventory-service")
    public void checkInventory(Order order) {
        log.info("Checking inventory for order: {}", order.getId());

        boolean available = true;

        for (OrderItem item : order.getItems()) {
            int stock = inventoryRepo.getStock(item.getProductId());

            if (stock < item.getQuantity()) {
                available = false;
                break;
            }
        }

        if (available) {
            // Reserve inventory
            reserveInventory(order);

            order.setStatus("INVENTORY_RESERVED");
            kafkaTemplate.send("inventory-check", order.getId(), order);
        } else {
            order.setStatus("OUT_OF_STOCK");
            kafkaTemplate.send("order-failed", order.getId(), order);
        }
    }

    @Transactional
    private void reserveInventory(Order order) {
        for (OrderItem item : order.getItems()) {
            inventoryRepo.decreaseStock(item.getProductId(), item.getQuantity());
        }
    }
}
```

**4. Payment Service:**

```java
@Service
public class PaymentService {
    @Autowired
    private KafkaTemplate<String, Order> kafkaTemplate;

    @Autowired
    private PaymentGateway paymentGateway;

    @KafkaListener(topics = "inventory-check", groupId = "payment-service")
    public void processPayment(Order order) {
        log.info("Processing payment for order: {}", order.getId());

        try {
            PaymentResult result = paymentGateway.charge(
                order.getCustomerId(),
                order.getTotalAmount()
            );

            if (result.isSuccess()) {
                order.setStatus("PAID");
                order.setPaymentId(result.getTransactionId());
                kafkaTemplate.send("payments", order.getId(), order);
            } else {
                order.setStatus("PAYMENT_FAILED");
                // Compensate - إرجاع الـ inventory
                kafkaTemplate.send("inventory-release", order.getId(), order);
            }
        } catch (Exception e) {
            log.error("Payment error for order: {}", order.getId(), e);
            order.setStatus("PAYMENT_ERROR");
            kafkaTemplate.send("order-failed", order.getId(), order);
        }
    }
}
```

**5. Notification Service:**

```java
@Service
public class NotificationService {
    @Autowired
    private EmailService emailService;

    @Autowired
    private SmsService smsService;

    @KafkaListener(topics = "payments", groupId = "notification-service")
    public void sendConfirmation(Order order) {
        log.info("Sending confirmation for order: {}", order.getId());

        // Email
        emailService.send(
            order.getCustomerEmail(),
            "Order Confirmation",
            "Your order " + order.getId() + " has been confirmed!"
        );

        // SMS
        smsService.send(
            order.getCustomerPhone(),
            "Order " + order.getId() + " confirmed. Total: $" + order.getTotalAmount()
        );
    }

    @KafkaListener(topics = "order-failed", groupId = "notification-service")
    public void sendFailureNotification(Order order) {
        log.info("Sending failure notification for order: {}", order.getId());

        emailService.send(
            order.getCustomerEmail(),
            "Order Failed",
            "Sorry, your order " + order.getId() + " failed: " + order.getStatus()
        );
    }
}
```

**6. Saga Pattern (Compensation):**

```java
@Service
public class InventoryCompensationService {
    @Autowired
    private InventoryRepository inventoryRepo;

    @KafkaListener(topics = "inventory-release", groupId = "compensation-service")
    @Transactional
    public void releaseInventory(Order order) {
        log.info("Releasing inventory for failed order: {}", order.getId());

        for (OrderItem item : order.getItems()) {
            inventoryRepo.increaseStock(item.getProductId(), item.getQuantity());
        }
    }
}
```

#### Topics Configuration:

```bash
# Create topics
kafka-topics.sh --create --topic orders --partitions 6 --replication-factor 3
kafka-topics.sh --create --topic order-validation --partitions 6 --replication-factor 3
kafka-topics.sh --create --topic inventory-check --partitions 6 --replication-factor 3
kafka-topics.sh --create --topic payments --partitions 6 --replication-factor 3
kafka-topics.sh --create --topic notifications --partitions 6 --replication-factor 3
kafka-topics.sh --create --topic order-failed --partitions 3 --replication-factor 3
kafka-topics.sh --create --topic inventory-release --partitions 3 --replication-factor 3
```

#### Error Handling:

```java
// Dead Letter Queue
@Bean
public ConcurrentKafkaListenerContainerFactory<String, Order> kafkaListenerContainerFactory() {
    ConcurrentKafkaListenerContainerFactory<String, Order> factory =
        new ConcurrentKafkaListenerContainerFactory<>();

    factory.setConsumerFactory(consumerFactory());

    // Error Handler with DLQ
    factory.setCommonErrorHandler(new DefaultErrorHandler(
        new DeadLetterPublishingRecoverer(kafkaTemplate(),
            (record, ex) -> new TopicPartition("order-dlq", record.partition())),
        new FixedBackOff(1000L, 3)  // 3 retries with 1 second interval
    ));

    return factory;
}
```

#### Monitoring:

```java
@Component
public class OrderMetrics {
    private final MeterRegistry meterRegistry;

    public OrderMetrics(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
    }

    public void recordOrderCreated() {
        meterRegistry.counter("orders.created").increment();
    }

    public void recordOrderCompleted(long processingTime) {
        meterRegistry.counter("orders.completed").increment();
        meterRegistry.timer("orders.processing.time").record(processingTime, TimeUnit.MILLISECONDS);
    }

    public void recordOrderFailed(String reason) {
        meterRegistry.counter("orders.failed", "reason", reason).increment();
    }
}
```

---

### السؤال 20: كيف تتعامل مع Message Ordering في Kafka؟

**الإجابة:**

**الترتيب في Kafka:**
- ✅ مضمون داخل **Partition الواحد**
- ❌ غير مضمون **across Partitions**

#### الاستراتيجيات:

### 1. **Single Partition (أبسط حل)**

```java
// جميع الرسائل في partition واحد
ProducerRecord<String, String> record =
    new ProducerRecord<>(
        "my-topic",
        0,  // Partition 0 دائماً
        "key",
        "value"
    );

// الترتيب مضمون، لكن:
// ❌ No parallelism
// ❌ Low throughput
// ❌ Single point of failure
```

### 2. **Keyed Messages (الأكثر استخداماً)**

```java
// الرسائل بنفس الـ key تذهب لنفس الـ partition
public class OrderProducer {
    public void sendOrder(Order order) {
        // استخدم customerId كـ key
        ProducerRecord<String, Order> record =
            new ProducerRecord<>(
                "orders",
                order.getCustomerId(),  // Key
                order
            );

        producer.send(record);
    }
}

// النتيجة:
// جميع orders للـ customer نفسه في نفس الـ partition
// الترتيب مضمون لكل customer
```

**Partitioning Logic:**
```java
partition = hash(key) % number_of_partitions

// مثال:
key = "customer-123"
hash("customer-123") = 456789
456789 % 3 = 0  → Partition 0

// جميع رسائل "customer-123" تذهب للـ Partition 0
```

### 3. **Custom Partitioner**

```java
public class OrderPartitioner implements Partitioner {
    @Override
    public int partition(String topic, Object key, byte[] keyBytes,
                        Object value, byte[] valueBytes, Cluster cluster) {

        Order order = (Order) value;
        int numPartitions = cluster.partitionCountForTopic(topic);

        // VIP customers → Partition 0 (للـ priority processing)
        if (order.isVip()) {
            return 0;
        }

        // Regional partitioning
        String region = order.getRegion();
        switch (region) {
            case "NORTH": return 1;
            case "SOUTH": return 2;
            case "EAST": return 3;
            case "WEST": return 4;
            default: return Math.abs(key.hashCode()) % numPartitions;
        }
    }

    @Override
    public void close() {}

    @Override
    public void configure(Map<String, ?> configs) {}
}

// Configuration
props.put("partitioner.class", OrderPartitioner.class.getName());
```

### 4. **Sequence Numbers**

```java
// إضافة sequence number للرسائل
public class SequencedMessage {
    private String key;
    private long sequenceNumber;
    private String payload;
    private long timestamp;

    // Constructor, Getters, Setters
}

// Producer
public class SequencedProducer {
    private AtomicLong sequenceCounter = new AtomicLong(0);

    public void send(String key, String payload) {
        SequencedMessage message = new SequencedMessage(
            key,
            sequenceCounter.incrementAndGet(),
            payload,
            System.currentTimeMillis()
        );

        producer.send(new ProducerRecord<>("topic", key, message));
    }
}

// Consumer
public class SequencedConsumer {
    private Map<String, Long> lastSequence = new HashMap<>();

    @KafkaListener(topics = "topic")
    public void consume(SequencedMessage message) {
        String key = message.getKey();
        long expectedSeq = lastSequence.getOrDefault(key, 0L) + 1;

        if (message.getSequenceNumber() == expectedSeq) {
            // الترتيب صحيح
            process(message);
            lastSequence.put(key, message.getSequenceNumber());
        } else if (message.getSequenceNumber() < expectedSeq) {
            // رسالة قديمة (duplicate)
            log.warn("Duplicate message: {}", message.getSequenceNumber());
        } else {
            // رسالة مفقودة
            log.error("Missing message. Expected: {}, Got: {}",
                expectedSeq, message.getSequenceNumber());
            // Handle: buffer, request replay, etc.
        }
    }
}
```

### 5. **Timestamp-based Ordering**

```java
// ترتيب بناءً على Timestamp
public class TimestampOrderingConsumer {
    private PriorityQueue<ConsumerRecord<String, Order>> buffer =
        new PriorityQueue<>(
            Comparator.comparing(ConsumerRecord::timestamp)
        );

    private long watermark = 0;

    @KafkaListener(topics = "orders")
    public void consume(ConsumerRecord<String, Order> record) {
        buffer.offer(record);

        // معالجة الرسائل القديمة (بناءً على watermark)
        while (!buffer.isEmpty() &&
               buffer.peek().timestamp() <= watermark) {
            ConsumerRecord<String, Order> orderedRecord = buffer.poll();
            process(orderedRecord);
        }

        // تحديث watermark
        watermark = Math.max(watermark, record.timestamp() - 5000);  // 5 sec buffer
    }
}
```

### 6. **Idempotent Producer (لمنع التكرار)**

```java
props.put("enable.idempotence", "true");
props.put("acks", "all");
props.put("retries", Integer.MAX_VALUE);
props.put("max.in.flight.requests.per.connection", "5");

// Kafka يضمن:
// 1. No duplicates
// 2. Ordering preserved (within partition)
// 3. Exactly-once semantics
```

### 7. **Consumer Configuration للترتيب**

```java
// Single consumer per partition
props.put("max.poll.records", "1");  // معالجة رسالة واحدة في المرة

// Manual commit للتحكم الكامل
props.put("enable.auto.commit", "false");

while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));

    for (ConsumerRecord<String, String> record : records) {
        // معالجة بالترتيب
        process(record);

        // Commit بعد كل رسالة
        consumer.commitSync();
    }
}
```

#### Best Practices:

| Scenario | Solution |
|----------|----------|
| **Global ordering** | Single partition (not scalable) |
| **Per-entity ordering** | Keyed messages (entity ID as key) |
| **Regional ordering** | Custom partitioner |
| **Exactly-once + ordering** | Idempotent producer + transactions |
| **Out-of-order detection** | Sequence numbers |
| **Late arrival handling** | Timestamp + watermark |

---

## أسئلة التصميم (Design Questions)

### السؤال 21: صمم Real-time Analytics Platform باستخدام Kafka

**الإجابة:**

#### Architecture:

```
┌──────────────────────────────────────────────────┐
│            Data Sources                           │
│  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐ │
│  │Web Apps│  │Mobile  │  │IoT     │  │Services│ │
│  └───┬────┘  └───┬────┘  └───┬────┘  └───┬────┘ │
└──────┼───────────┼───────────┼───────────┼──────┘
       │           │           │           │
       └───────────┴───────────┴───────────┘
                       ↓
         ┌─────────────────────────────┐
         │   Kafka Ingestion Layer     │
         │   Topics: raw-events        │
         └─────────────┬───────────────┘
                       ↓
         ┌─────────────────────────────┐
         │    Kafka Streams Layer      │
         │  ┌──────────┐  ┌──────────┐ │
         │  │Filtering │  │Transform │ │
         │  └──────────┘  └──────────┘ │
         │  ┌──────────┐  ┌──────────┐ │
         │  │Aggreg.   │  │Windowing │ │
         │  └──────────┘  └──────────┘ │
         └─────────────┬───────────────┘
                       ↓
         ┌─────────────────────────────┐
         │   Processed Topics          │
         │  - page-views               │
         │  - user-sessions            │
         │  - conversions              │
         │  - metrics                  │
         └─────────────┬───────────────┘
                       ↓
       ┌───────────────┴────────────────┐
       │                                 │
       ↓                                 ↓
┌──────────────┐              ┌──────────────────┐
│ Elasticsearch│              │  Real-time       │
│   (Search)   │              │  Dashboard       │
└──────────────┘              │  (WebSocket)     │
       ↓                       └──────────────────┘
┌──────────────┐
│   Kibana     │
│ (Visualization)
└──────────────┘
```

#### Implementation:

**1. Event Ingestion:**

```java
@RestController
@RequestMapping("/api/events")
public class EventIngestionController {
    @Autowired
    private KafkaTemplate<String, Event> kafkaTemplate;

    @PostMapping("/track")
    public ResponseEntity<?> trackEvent(@RequestBody Event event) {
        // إضافة metadata
        event.setTimestamp(System.currentTimeMillis());
        event.setEventId(UUID.randomUUID().toString());

        // إرسال إلى Kafka
        kafkaTemplate.send("raw-events", event.getUserId(), event);

        return ResponseEntity.ok("Event tracked");
    }
}

// Event Model
class Event {
    private String eventId;
    private String userId;
    private String sessionId;
    private String eventType;  // PAGE_VIEW, CLICK, PURCHASE, etc.
    private Map<String, Object> properties;
    private long timestamp;

    // Getters, Setters
}
```

**2. Stream Processing:**

```java
@Component
public class AnalyticsStreamProcessor {

    @Bean
    public KStream<String, Event> processEvents(StreamsBuilder builder) {
        KStream<String, Event> rawEvents = builder.stream("raw-events");

        // 1. Page Views Analytics
        processPageViews(rawEvents);

        // 2. User Sessions
        processUserSessions(rawEvents);

        // 3. Conversion Funnel
        processConversions(rawEvents);

        // 4. Real-time Metrics
        processMetrics(rawEvents);

        return rawEvents;
    }

    private void processPageViews(KStream<String, Event> events) {
        events
            .filter((key, event) -> "PAGE_VIEW".equals(event.getEventType()))
            .groupBy((key, event) ->
                (String) event.getProperties().get("page"))
            .windowedBy(TimeWindows.of(Duration.ofMinutes(5)))
            .count()
            .toStream()
            .map((windowedKey, count) -> KeyValue.pair(
                windowedKey.key() + "@" + windowedKey.window().start(),
                new PageViewMetric(
                    windowedKey.key(),
                    count,
                    windowedKey.window().start(),
                    windowedKey.window().end()
                )
            ))
            .to("page-view-metrics");
    }

    private void processUserSessions(KStream<String, Event> events) {
        events
            .groupBy((key, event) -> event.getUserId())
            .windowedBy(SessionWindows.with(Duration.ofMinutes(30)))
            .aggregate(
                UserSession::new,
                (key, event, session) -> {
                    session.addEvent(event);
                    return session;
                },
                (key, session1, session2) -> session1.merge(session2),
                Materialized.with(Serdes.String(), new UserSessionSerde())
            )
            .toStream()
            .map((windowedKey, session) -> KeyValue.pair(
                session.getSessionId(),
                session
            ))
            .to("user-sessions");
    }

    private void processConversions(KStream<String, Event> events) {
        // Funnel: PAGE_VIEW → ADD_TO_CART → CHECKOUT → PURCHASE

        KStream<String, Event> pageViews = events
            .filter((k, v) -> "PAGE_VIEW".equals(v.getEventType()));

        KStream<String, Event> addToCart = events
            .filter((k, v) -> "ADD_TO_CART".equals(v.getEventType()));

        KStream<String, Event> purchases = events
            .filter((k, v) -> "PURCHASE".equals(v.getEventType()));

        // Join streams
        JoinWindows window = JoinWindows.of(Duration.ofHours(1));

        pageViews
            .leftJoin(addToCart,
                (view, cart) -> new FunnelStep("view_to_cart", view, cart),
                window)
            .leftJoin(purchases,
                (funnel, purchase) -> funnel.addPurchase(purchase),
                window)
            .to("conversion-funnel");
    }

    private void processMetrics(KStream<String, Event> events) {
        // Real-time aggregations

        // 1. Events per second
        events
            .groupBy((key, event) -> event.getEventType())
            .windowedBy(TimeWindows.of(Duration.ofSeconds(10)))
            .count()
            .toStream()
            .to("events-per-second");

        // 2. Active users
        events
            .groupBy((key, event) -> event.getUserId())
            .windowedBy(TimeWindows.of(Duration.ofMinutes(5)))
            .count()
            .toStream()
            .map((key, count) -> KeyValue.pair(
                "active-users",
                new Metric("active_users", count, System.currentTimeMillis())
            ))
            .to("real-time-metrics");

        // 3. Top pages
        events
            .filter((k, v) -> "PAGE_VIEW".equals(v.getEventType()))
            .groupBy((key, event) ->
                (String) event.getProperties().get("page"))
            .windowedBy(TimeWindows.of(Duration.ofMinutes(15)))
            .count()
            .toStream()
            .groupBy((key, count) -> "all",
                Grouped.with(Serdes.String(), Serdes.Long()))
            .aggregate(
                TopPages::new,
                (key, count, topPages) -> topPages.add(count),
                Materialized.with(Serdes.String(), new TopPagesSerde())
            )
            .toStream()
            .to("top-pages");
    }
}
```

**3. Sink to Elasticsearch:**

```java
@Service
public class ElasticsearchSinkService {
    @Autowired
    private RestHighLevelClient elasticsearchClient;

    @KafkaListener(topics = "page-view-metrics", groupId = "es-sink")
    public void sinkPageViewMetrics(PageViewMetric metric) {
        IndexRequest request = new IndexRequest("page-views")
            .source(
                "page", metric.getPage(),
                "count", metric.getCount(),
                "window_start", metric.getWindowStart(),
                "window_end", metric.getWindowEnd(),
                "@timestamp", new Date(metric.getWindowEnd())
            );

        try {
            elasticsearchClient.index(request, RequestOptions.DEFAULT);
        } catch (IOException e) {
            log.error("Failed to index to Elasticsearch", e);
        }
    }

    @KafkaListener(topics = "user-sessions", groupId = "es-sink")
    public void sinkUserSessions(UserSession session) {
        IndexRequest request = new IndexRequest("user-sessions")
            .id(session.getSessionId())
            .source(
                "user_id", session.getUserId(),
                "session_id", session.getSessionId(),
                "duration", session.getDuration(),
                "page_views", session.getPageViews(),
                "events_count", session.getEventsCount(),
                "start_time", new Date(session.getStartTime()),
                "end_time", new Date(session.getEndTime())
            );

        try {
            elasticsearchClient.index(request, RequestOptions.DEFAULT);
        } catch (IOException e) {
            log.error("Failed to index session", e);
        }
    }
}
```

**4. Real-time Dashboard (WebSocket):**

```java
@Component
public class MetricsWebSocketService {
    private SimpMessagingTemplate messagingTemplate;

    public MetricsWebSocketService(SimpMessagingTemplate messagingTemplate) {
        this.messagingTemplate = messagingTemplate;
    }

    @KafkaListener(topics = "real-time-metrics", groupId = "websocket")
    public void streamMetrics(Metric metric) {
        // إرسال إلى WebSocket clients
        messagingTemplate.convertAndSend("/topic/metrics", metric);
    }

    @KafkaListener(topics = "events-per-second", groupId = "websocket")
    public void streamEventsPerSecond(
        @Payload ConsumerRecord<Windowed<String>, Long> record) {

        String eventType = record.key().key();
        long count = record.value();

        messagingTemplate.convertAndSend(
            "/topic/events-per-second",
            Map.of("eventType", eventType, "count", count)
        );
    }
}

// WebSocket Configuration
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.enableSimpleBroker("/topic");
        config.setApplicationDestinationPrefixes("/app");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws").setAllowedOrigins("*").withSockJS();
    }
}
```

**5. Frontend Dashboard:**

```javascript
// React Component
import React, { useState, useEffect } from 'react';
import SockJS from 'sockjs-client';
import { Stomp } from '@stomp/stompjs';

function RealTimeDashboard() {
    const [metrics, setMetrics] = useState({});
    const [eventsPerSecond, setEventsPerSecond] = useState([]);

    useEffect(() => {
        const socket = new SockJS('http://localhost:8080/ws');
        const stompClient = Stomp.over(socket);

        stompClient.connect({}, () => {
            // Subscribe to metrics
            stompClient.subscribe('/topic/metrics', (message) => {
                const metric = JSON.parse(message.body);
                setMetrics(prev => ({
                    ...prev,
                    [metric.name]: metric.value
                }));
            });

            // Subscribe to events per second
            stompClient.subscribe('/topic/events-per-second', (message) => {
                const data = JSON.parse(message.body);
                setEventsPerSecond(prev => [...prev, data].slice(-20));
            });
        });

        return () => stompClient.disconnect();
    }, []);

    return (
        <div>
            <h1>Real-Time Analytics</h1>

            <div className="metrics">
                <MetricCard
                    title="Active Users"
                    value={metrics.active_users}
                />
                <MetricCard
                    title="Page Views"
                    value={metrics.page_views}
                />
            </div>

            <LineChart data={eventsPerSecond} />
        </div>
    );
}
```

#### Scaling Considerations:

```yaml
# Kafka Configuration
topics:
  raw-events:
    partitions: 12
    replication-factor: 3
    retention: 7 days

  page-view-metrics:
    partitions: 6
    replication-factor: 3
    retention: 30 days

  user-sessions:
    partitions: 6
    replication-factor: 3
    retention: 90 days
    compaction: enabled  # Keep latest session state

# Streams Configuration
streams:
  num-stream-threads: 4
  cache-max-bytes-buffering: 10485760  # 10MB
  commit-interval-ms: 1000
```

---

## نصائح للمقابلة

### 1. الاستعداد الفني

```
✅ فهم عميق للـ Core Concepts
✅ الخبرة العملية (أمثلة من مشاريعك)
✅ معرفة الـ Best Practices
✅ Troubleshooting skills
```

### 2. أسئلة شائعة

```
- ما هو Kafka وما الفرق بينه وبين RabbitMQ؟
- اشرح Consumer Groups
- كيف تضمن Message Ordering؟
- كيف تتعامل مع Consumer Lag؟
- ما هو Exactly-Once Semantics؟
```

### 3. الإجابة الجيدة تحتوي على:

```
1. شرح المفهوم
2. أمثلة عملية (Code)
3. Trade-offs
4. Best Practices
5. خبرتك الشخصية
```

### 4. أسئلة لك للسؤال:

```
- ما حجم البيانات المتوقع؟
- ما هي متطلبات الـ latency؟
- هل تحتاجون Exactly-Once؟
- ما البنية التحتية الحالية؟
```

---

**بالتوفيق في مقابلتك! 🎯**

**تذكر: Kafka واسع جداً، ركز على:**
1. Core Concepts
2. Producer/Consumer APIs
3. Streams (إذا كان relevant)
4. Production Best Practices
5. خبرتك العملية

---

**تم إنشاء هذا الدليل كمرجع شامل لأسئلة المقابلات عن Apache Kafka**
