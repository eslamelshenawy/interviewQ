# دليل Apache ActiveMQ الشامل

## جدول المحتويات
1. [ما هو ActiveMQ](#ما-هو-activemq)
2. [المميزات الرئيسية](#المميزات-الرئيسية)
3. [البنية المعمارية](#البنية-المعمارية)
4. [حالات الاستخدام](#حالات-الاستخدام)
5. [التثبيت والإعداد](#التثبيت-والإعداد)
6. [أمثلة عملية](#أمثلة-عملية)

---

## ما هو ActiveMQ

**Apache ActiveMQ** هو وسيط رسائل (Message Broker) مفتوح المصدر مكتوب بلغة Java، يدعم بروتوكولات متعددة ويوفر نظاماً موثوقاً لتبادل الرسائل بين التطبيقات الموزعة.

### المفاهيم الأساسية

#### Message Queue (طابور الرسائل)
طابور الرسائل هو آلية لتخزين الرسائل مؤقتاً حتى يتم معالجتها من قبل المستقبل. يعمل كوسيط بين المرسل (Producer) والمستقبل (Consumer).

#### Message Broker (وسيط الرسائل)
برنامج وسيط يستقبل الرسائل من المرسلين ويوزعها على المستقبلين، مما يفصل بين المكونات المختلفة للنظام.

#### JMS (Java Message Service)
واجهة برمجية قياسية في Java لإرسال واستقبال الرسائل بين التطبيقات. ActiveMQ يطبق معيار JMS بالكامل.

---

## المميزات الرئيسية

### 1. دعم البروتوكولات المتعددة
```
- AMQP (Advanced Message Queuing Protocol)
- MQTT (Message Queuing Telemetry Transport)
- STOMP (Simple Text Oriented Messaging Protocol)
- OpenWire (البروتوكول الأصلي لـ ActiveMQ)
- WebSocket
- REST API
```

### 2. أنماط المراسلة
- **Point-to-Point (نقطة لنقطة)**: Queue - رسالة واحدة لمستقبل واحد
- **Publish-Subscribe (نشر واشتراك)**: Topic - رسالة واحدة لعدة مستقبلين

### 3. الموثوقية والثبات
- **Message Persistence**: حفظ الرسائل على القرص
- **Guaranteed Delivery**: ضمان توصيل الرسائل
- **Transaction Support**: دعم المعاملات
- **Message Durability**: استمرارية الرسائل حتى بعد إعادة التشغيل

### 4. الأداء العالي
- معالجة آلاف الرسائل في الثانية
- دعم Clustering للتوزيع
- Load Balancing تلقائي
- Message Caching

### 5. إدارة ومراقبة سهلة
- Web Console لإدارة ActiveMQ
- JMX (Java Management Extensions) للمراقبة
- REST API للأتمتة
- دعم Hawtio للواجهة الرسومية المتقدمة

### 6. الأمان
- SSL/TLS للتشفير
- JAAS للمصادقة والترخيص
- Message-level security

---

## البنية المعمارية

### المكونات الأساسية

```
┌─────────────────────────────────────────────────────────┐
│                    ActiveMQ Broker                      │
│                                                         │
│  ┌──────────────┐      ┌──────────────┐               │
│  │   Queues     │      │    Topics    │               │
│  │              │      │              │               │
│  │  - Queue 1   │      │  - Topic 1   │               │
│  │  - Queue 2   │      │  - Topic 2   │               │
│  └──────────────┘      └──────────────┘               │
│                                                         │
│  ┌─────────────────────────────────────────────────┐  │
│  │        Persistent Storage (KahaDB)              │  │
│  └─────────────────────────────────────────────────┘  │
│                                                         │
│  ┌─────────────────────────────────────────────────┐  │
│  │     Transport Connectors (Protocols)            │  │
│  │  - OpenWire  - AMQP  - MQTT  - STOMP           │  │
│  └─────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
         ▲                           │
         │                           ▼
    ┌─────────┐              ┌─────────────┐
    │Producer │              │  Consumers  │
    │ (مرسل)  │              │ (مستقبلين) │
    └─────────┘              └─────────────┘
```

### شرح المكونات

#### 1. Producer (المنتج/المرسل)
التطبيق الذي يرسل الرسائل إلى ActiveMQ.

#### 2. Consumer (المستهلك/المستقبل)
التطبيق الذي يستقبل ويعالج الرسائل من ActiveMQ.

#### 3. Broker (الوسيط)
خادم ActiveMQ الذي يدير الرسائل ويوزعها.

#### 4. Destination (الوجهة)
المكان الذي توضع فيه الرسائل - إما Queue أو Topic.

#### 5. Message (الرسالة)
البيانات المرسلة، تتكون من:
- **Headers**: معلومات وصفية
- **Properties**: خصائص مخصصة
- **Body**: المحتوى الفعلي

---

## حالات الاستخدام

### 1. معالجة الطلبات غير المتزامنة
```java
// مثال: نظام معالجة الطلبات
Producer → ActiveMQ Queue → Consumer (Order Processing Service)
```
**الفائدة**: فصل استقبال الطلب عن معالجته، تحسين استجابة النظام.

### 2. إرسال الإشعارات
```java
// إرسال إشعارات متعددة
Notification Service → Topic → [Email Consumer, SMS Consumer, Push Consumer]
```
**الفائدة**: إرسال رسالة واحدة لعدة مستقبلين.

### 3. تكامل الأنظمة
```java
// ربط أنظمة مختلفة
System A → ActiveMQ → System B
```
**الفائدة**: فصل الأنظمة وتقليل الاعتماد المباشر.

### 4. معالجة البيانات الضخمة
```java
// توزيع معالجة البيانات
Data Source → Queue → [Worker 1, Worker 2, Worker 3, ...]
```
**الفائدة**: توزيع الحمل على عدة معالجات.

### 5. Event-Driven Architecture
```java
// بنية موجهة بالأحداث
Event Producer → Topic → Multiple Event Handlers
```

### 6. أنظمة الدفع والمعاملات المالية
- ضمان عدم فقدان المعاملات
- دعم Transactions
- Message Persistence

### 7. IoT (إنترنت الأشياء)
- دعم MQTT البروتوكول الخفيف
- معالجة آلاف الأجهزة المتصلة

---

## التثبيت والإعداد

### المتطلبات
- Java JDK 8 أو أحدث
- 512 MB RAM على الأقل (يفضل 1 GB+)
- 1 GB مساحة على القرص

### التثبيت

#### 1. تحميل ActiveMQ
```bash
# تحميل أحدث نسخة
wget https://archive.apache.org/dist/activemq/5.18.3/apache-activemq-5.18.3-bin.tar.gz

# فك الضغط
tar -xzf apache-activemq-5.18.3-bin.tar.gz
cd apache-activemq-5.18.3
```

#### 2. تشغيل ActiveMQ

**على Linux/Mac:**
```bash
./bin/activemq start
```

**على Windows:**
```cmd
bin\activemq.bat start
```

#### 3. التحقق من التشغيل
```bash
# Web Console متوفر على:
http://localhost:8161/admin

# بيانات الدخول الافتراضية:
Username: admin
Password: admin

# OpenWire بروتوكول على:
tcp://localhost:61616
```

### التثبيت عبر Docker

```bash
# تشغيل ActiveMQ في حاوية Docker
docker run -d \
  --name activemq \
  -p 61616:61616 \
  -p 8161:8161 \
  rmohr/activemq:latest

# التحقق
docker logs activemq
```

### الإعداد الأساسي

#### ملف activemq.xml
```xml
<broker xmlns="http://activemq.apache.org/schema/core"
        brokerName="localhost"
        dataDirectory="${activemq.data}">

    <!-- تكوين Persistence -->
    <persistenceAdapter>
        <kahaDB directory="${activemq.data}/kahadb"/>
    </persistenceAdapter>

    <!-- Transport Connectors -->
    <transportConnectors>
        <!-- OpenWire -->
        <transportConnector name="openwire"
            uri="tcp://0.0.0.0:61616?maximumConnections=1000"/>

        <!-- AMQP -->
        <transportConnector name="amqp"
            uri="amqp://0.0.0.0:5672"/>

        <!-- MQTT -->
        <transportConnector name="mqtt"
            uri="mqtt://0.0.0.0:1883"/>

        <!-- STOMP -->
        <transportConnector name="stomp"
            uri="stomp://0.0.0.0:61613"/>

        <!-- WebSocket -->
        <transportConnector name="ws"
            uri="ws://0.0.0.0:61614"/>
    </transportConnectors>

    <!-- إعدادات الأمان -->
    <plugins>
        <simpleAuthenticationPlugin>
            <users>
                <authenticationUser username="admin"
                    password="admin"
                    groups="admins,users"/>
                <authenticationUser username="user"
                    password="user"
                    groups="users"/>
            </users>
        </simpleAuthenticationPlugin>
    </plugins>

</broker>
```

---

## أمثلة عملية

### مثال 1: Producer و Consumer بسيط

#### Producer (المرسل)
```java
import org.apache.activemq.ActiveMQConnectionFactory;
import javax.jms.*;

public class SimpleProducer {
    private static final String BROKER_URL = "tcp://localhost:61616";
    private static final String QUEUE_NAME = "TEST.QUEUE";

    public static void main(String[] args) throws JMSException {
        // إنشاء Connection Factory
        ConnectionFactory connectionFactory =
            new ActiveMQConnectionFactory(BROKER_URL);

        // إنشاء Connection
        Connection connection = connectionFactory.createConnection();
        connection.start();

        // إنشاء Session
        Session session = connection.createSession(
            false,
            Session.AUTO_ACKNOWLEDGE
        );

        // إنشاء Destination (Queue)
        Destination destination = session.createQueue(QUEUE_NAME);

        // إنشاء Producer
        MessageProducer producer = session.createProducer(destination);

        // إنشاء وإرسال رسالة
        TextMessage message = session.createTextMessage("مرحباً من ActiveMQ!");
        producer.send(message);
        System.out.println("تم إرسال الرسالة: " + message.getText());

        // إغلاق الموارد
        producer.close();
        session.close();
        connection.close();
    }
}
```

#### Consumer (المستقبل)
```java
import org.apache.activemq.ActiveMQConnectionFactory;
import javax.jms.*;

public class SimpleConsumer {
    private static final String BROKER_URL = "tcp://localhost:61616";
    private static final String QUEUE_NAME = "TEST.QUEUE";

    public static void main(String[] args) throws JMSException {
        // إنشاء Connection Factory
        ConnectionFactory connectionFactory =
            new ActiveMQConnectionFactory(BROKER_URL);

        // إنشاء Connection
        Connection connection = connectionFactory.createConnection();
        connection.start();

        // إنشاء Session
        Session session = connection.createSession(
            false,
            Session.AUTO_ACKNOWLEDGE
        );

        // إنشاء Destination (Queue)
        Destination destination = session.createQueue(QUEUE_NAME);

        // إنشاء Consumer
        MessageConsumer consumer = session.createConsumer(destination);

        // استقبال الرسائل
        consumer.setMessageListener(message -> {
            if (message instanceof TextMessage) {
                try {
                    TextMessage textMessage = (TextMessage) message;
                    System.out.println("تم استقبال الرسالة: " +
                        textMessage.getText());
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        });

        // الانتظار (في تطبيق حقيقي، سيعمل باستمرار)
        System.out.println("في انتظار الرسائل...");
        try {
            Thread.sleep(60000); // انتظار دقيقة واحدة
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        // إغلاق الموارد
        consumer.close();
        session.close();
        connection.close();
    }
}
```

### مثال 2: استخدام Topic (Pub/Sub)

```java
public class TopicExample {

    // Publisher
    public static void publishMessage() throws JMSException {
        ConnectionFactory factory =
            new ActiveMQConnectionFactory("tcp://localhost:61616");
        Connection connection = factory.createConnection();
        connection.start();

        Session session = connection.createSession(
            false,
            Session.AUTO_ACKNOWLEDGE
        );

        // إنشاء Topic بدلاً من Queue
        Topic topic = session.createTopic("NEWS.TOPIC");
        MessageProducer publisher = session.createProducer(topic);

        TextMessage message = session.createTextMessage("خبر عاجل!");
        publisher.send(message);

        System.out.println("تم نشر الرسالة: " + message.getText());

        connection.close();
    }

    // Subscriber
    public static void subscribeToTopic() throws JMSException {
        ConnectionFactory factory =
            new ActiveMQConnectionFactory("tcp://localhost:61616");
        Connection connection = factory.createConnection();
        connection.start();

        Session session = connection.createSession(
            false,
            Session.AUTO_ACKNOWLEDGE
        );

        Topic topic = session.createTopic("NEWS.TOPIC");
        MessageConsumer subscriber = session.createConsumer(topic);

        subscriber.setMessageListener(message -> {
            try {
                TextMessage textMessage = (TextMessage) message;
                System.out.println("المشترك استلم: " +
                    textMessage.getText());
            } catch (JMSException e) {
                e.printStackTrace();
            }
        });

        System.out.println("في انتظار الأخبار...");
    }
}
```

---

## الإعدادات المتقدمة

### 1. تكوين Memory Limits

```xml
<systemUsage>
    <systemUsage>
        <memoryUsage>
            <memoryUsage percentOfJvmHeap="70" />
        </memoryUsage>
        <storeUsage>
            <storeUsage limit="10 gb"/>
        </storeUsage>
        <tempUsage>
            <tempUsage limit="5 gb"/>
        </tempUsage>
    </systemUsage>
</systemUsage>
```

### 2. تكوين Dead Letter Queue

```xml
<destinationPolicy>
    <policyMap>
        <policyEntries>
            <policyEntry queue=">">
                <deadLetterStrategy>
                    <individualDeadLetterStrategy
                        queuePrefix="DLQ."
                        useQueueForQueueMessages="true"/>
                </deadLetterStrategy>
            </policyEntry>
        </policyEntries>
    </policyMap>
</destinationPolicy>
```

### 3. Message Prefetch Settings

```java
// تحسين الأداء عبر Prefetch
String url = "tcp://localhost:61616?jms.prefetchPolicy.queuePrefetch=1000";
ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory(url);
```

---

## المراقبة والإدارة

### 1. Web Console
- الوصول: `http://localhost:8161/admin`
- عرض Queues و Topics
- إحصائيات الرسائل
- إدارة Connections

### 2. JMX Monitoring
```java
// استخدام JConsole أو VisualVM
// الاتصال بـ: service:jmx:rmi:///jndi/rmi://localhost:1099/jmxrmi
```

### 3. Metrics مهمة
- **Enqueue Count**: عدد الرسائل المرسلة
- **Dequeue Count**: عدد الرسائل المستلمة
- **Queue Size**: عدد الرسائل في الانتظار
- **Consumer Count**: عدد المستهلكين المتصلين
- **Memory Usage**: استخدام الذاكرة

---

## أفضل الممارسات

### 1. إدارة الاتصالات
```java
// استخدم Connection Pooling
PooledConnectionFactory pooledFactory =
    new PooledConnectionFactory(connectionFactory);
pooledFactory.setMaxConnections(10);
```

### 2. معالجة الأخطاء
```java
// استخدم ExceptionListener
connection.setExceptionListener(exception -> {
    System.err.println("خطأ في الاتصال: " + exception.getMessage());
    // إعادة الاتصال
});
```

### 3. استخدام Try-With-Resources
```java
try (ActiveMQConnection connection = (ActiveMQConnection)
        connectionFactory.createConnection()) {
    // عملك هنا
} catch (JMSException e) {
    e.printStackTrace();
}
```

### 4. تكوين Acknowledgment Mode المناسب
- **AUTO_ACKNOWLEDGE**: تلقائي - سهل لكن قد يفقد رسائل
- **CLIENT_ACKNOWLEDGE**: يدوي - مزيد من التحكم
- **DUPS_OK_ACKNOWLEDGE**: أداء أفضل لكن قد يكرر رسائل
- **SESSION_TRANSACTED**: للمعاملات

---

## الخلاصة

Apache ActiveMQ هو حل قوي وموثوق لإدارة الرسائل في الأنظمة الموزعة. يوفر:
- مرونة عالية في أنماط المراسلة
- أداء ممتاز وقابلية للتوسع
- موثوقية وضمان توصيل الرسائل
- سهولة التكامل مع التطبيقات المختلفة

### موارد إضافية
- الموقع الرسمي: https://activemq.apache.org/
- التوثيق: https://activemq.apache.org/documentation
- GitHub: https://github.com/apache/activemq

---

**ملاحظة**: للحصول على أسئلة المقابلات المفصلة والأمثلة المتقدمة، راجع ملف `ActiveMQ-Interview-Questions.md`.
