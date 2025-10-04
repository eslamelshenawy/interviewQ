# Apache ActiveMQ Documentation

## Table of Contents
- [Introduction](#introduction)
- [What is ActiveMQ?](#what-is-activemq)
- [Key Features](#key-features)
- [Architecture](#architecture)
- [Getting Started](#getting-started)
- [Installation](#installation)
- [Basic Concepts](#basic-concepts)
- [Use Cases](#use-cases)
- [Resources](#resources)

## Introduction

Apache ActiveMQ is an open-source, multi-protocol, Java-based message broker. It implements the Java Message Service (JMS) API and provides enterprise-level messaging capabilities for building distributed applications.

## What is ActiveMQ?

ActiveMQ is a message-oriented middleware (MOM) that enables applications to communicate asynchronously through message passing. It acts as an intermediary between message producers and consumers, ensuring reliable message delivery even when systems are temporarily unavailable.

### Key Characteristics

- **Open Source**: Apache Software Foundation project
- **JMS Compliant**: Fully implements JMS 1.1 and JMS 2.0 specifications
- **Multi-Protocol**: Supports AMQP, MQTT, STOMP, OpenWire, and more
- **Cross-Platform**: Written in Java, runs on any platform with JVM
- **High Performance**: Optimized for high throughput and low latency

## Key Features

### 1. **Message Patterns**
- **Point-to-Point (Queue)**: One-to-one message delivery
- **Publish-Subscribe (Topic)**: One-to-many message broadcasting

### 2. **Reliability**
- Message persistence to disk or database
- Transaction support
- Message acknowledgment mechanisms
- Dead Letter Queue (DLQ) for failed messages

### 3. **Clustering & High Availability**
- Master-Slave configuration
- Network of Brokers
- Load balancing
- Failover support

### 4. **Security**
- Authentication and authorization
- SSL/TLS encryption
- JAAS integration
- Role-based access control

### 5. **Monitoring & Management**
- Web console for administration
- JMX monitoring
- Advisory messages
- Statistics and metrics

## Architecture

### Core Components

```
┌─────────────────────────────────────────────────────┐
│                   ActiveMQ Broker                   │
├─────────────────────────────────────────────────────┤
│  ┌──────────┐  ┌──────────┐  ┌─────────────────┐  │
│  │ Queues   │  │ Topics   │  │ Durable Subs    │  │
│  └──────────┘  └──────────┘  └─────────────────┘  │
│  ┌──────────────────────────────────────────────┐  │
│  │         Message Persistence Layer            │  │
│  │  (KahaDB, JDBC, LevelDB, Memory)            │  │
│  └──────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────┐  │
│  │         Transport Connectors                 │  │
│  │  (TCP, NIO, SSL, HTTP, WebSocket)           │  │
│  └──────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
         ▲                              ▲
         │                              │
    ┌────┴────┐                   ┌────┴────┐
    │Producer │                   │Consumer │
    └─────────┘                   └─────────┘
```

### Message Flow

1. **Producer** creates and sends messages to the broker
2. **Broker** receives, stores, and manages messages
3. **Consumer** retrieves and processes messages from the broker
4. **Acknowledgment** confirms message processing
5. **DLQ** handles failed/expired messages

## Getting Started

### Installation

#### Download and Install

```bash
# Download ActiveMQ
wget https://archive.apache.org/dist/activemq/5.18.0/apache-activemq-5.18.0-bin.tar.gz

# Extract
tar -xzf apache-activemq-5.18.0-bin.tar.gz

# Navigate to directory
cd apache-activemq-5.18.0

# Start ActiveMQ
./bin/activemq start

# Stop ActiveMQ
./bin/activemq stop
```

#### Using Docker

```bash
# Pull and run ActiveMQ
docker run -d --name activemq \
  -p 61616:61616 \
  -p 8161:8161 \
  rmohr/activemq

# Access Web Console
# http://localhost:8161/admin
# Default credentials: admin/admin
```

### Maven Dependencies

```xml
<dependencies>
    <!-- ActiveMQ Core -->
    <dependency>
        <groupId>org.apache.activemq</groupId>
        <artifactId>activemq-all</artifactId>
        <version>5.18.0</version>
    </dependency>

    <!-- Spring Boot ActiveMQ Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-activemq</artifactId>
    </dependency>

    <!-- JMS API -->
    <dependency>
        <groupId>javax.jms</groupId>
        <artifactId>javax.jms-api</artifactId>
        <version>2.0.1</version>
    </dependency>
</dependencies>
```

### Configuration

#### application.properties (Spring Boot)

```properties
# ActiveMQ Configuration
spring.activemq.broker-url=tcp://localhost:61616
spring.activemq.user=admin
spring.activemq.password=admin
spring.activemq.packages.trust-all=false
spring.activemq.packages.trusted=com.example

# JMS Configuration
spring.jms.pub-sub-domain=false
spring.jms.template.default-destination=myQueue

# Connection Pool
spring.activemq.pool.enabled=true
spring.activemq.pool.max-connections=10
spring.activemq.pool.idle-timeout=30000
```

## Basic Concepts

### 1. Destinations

**Queue**: Point-to-point communication where each message is consumed by exactly one consumer.

```java
@Component
public class QueueExample {
    @JmsListener(destination = "orderQueue")
    public void receiveMessage(String message) {
        System.out.println("Received: " + message);
    }
}
```

**Topic**: Publish-subscribe pattern where messages are broadcast to all subscribers.

```java
@Component
public class TopicExample {
    @JmsListener(destination = "notificationTopic",
                 containerFactory = "topicListenerFactory")
    public void receiveMessage(String message) {
        System.out.println("Received: " + message);
    }
}
```

### 2. Message Types

- **TextMessage**: String content
- **ObjectMessage**: Serializable Java objects
- **MapMessage**: Key-value pairs
- **BytesMessage**: Binary data
- **StreamMessage**: Stream of Java primitives

### 3. Delivery Modes

- **Persistent**: Messages survive broker restart (stored on disk)
- **Non-Persistent**: Messages kept in memory only (faster but less reliable)

### 4. Acknowledgment Modes

- **AUTO_ACKNOWLEDGE**: Automatic acknowledgment
- **CLIENT_ACKNOWLEDGE**: Manual acknowledgment by client
- **DUPS_OK_ACKNOWLEDGE**: Lazy acknowledgment (duplicates possible)
- **SESSION_TRANSACTED**: Transactional acknowledgment

## Use Cases

### 1. **Decoupling Systems**
Enable independent development and deployment of microservices.

### 2. **Asynchronous Processing**
Handle time-consuming tasks without blocking the main application flow.

### 3. **Load Leveling**
Smooth out traffic spikes by queuing requests.

### 4. **Event-Driven Architecture**
Build reactive systems that respond to events.

### 5. **Integration**
Connect heterogeneous systems using standard messaging protocols.

### 6. **Reliable Communication**
Ensure message delivery even during network failures or system outages.

## ActiveMQ vs ActiveMQ Artemis

| Feature | ActiveMQ 5.x | ActiveMQ Artemis |
|---------|--------------|------------------|
| Architecture | Classic JMS broker | Next-gen, HornetQ-based |
| Performance | Good | Excellent |
| Protocol | JMS, STOMP, MQTT, AMQP | JMS, STOMP, MQTT, AMQP, Core |
| Persistence | KahaDB, JDBC | Journal-based |
| Clustering | Network of Brokers | Native clustering |
| Status | Maintenance mode | Active development |
| Use Case | Legacy systems | New deployments |

**Note**: ActiveMQ Artemis is the recommended choice for new projects.

## Monitoring and Management

### Web Console
Access at: `http://localhost:8161/admin`
- View queues and topics
- Browse messages
- Monitor connections
- Manage subscribers

### JMX Monitoring
```java
// Enable JMX in activemq.xml
<managementContext>
    <managementContext createConnector="true"/>
</managementContext>
```

### Advisory Messages
Special messages that provide broker events:
- Consumer start/stop
- Message delivery
- Queue creation/deletion

## Performance Tuning

### Producer Optimization
- Use async send for non-critical messages
- Batch messages when possible
- Use non-persistent delivery for non-critical data

### Consumer Optimization
- Use prefetch limits appropriately
- Implement concurrent consumers
- Use message selectors to filter at broker level

### Broker Optimization
- Configure appropriate memory limits
- Use KahaDB for better performance
- Enable producer flow control
- Optimize network connectors

## Security Best Practices

1. **Change default credentials** (admin/admin)
2. **Enable SSL/TLS** for transport
3. **Implement authentication** using JAAS
4. **Configure authorization** for destinations
5. **Use message-level encryption** for sensitive data
6. **Restrict network access** using firewalls
7. **Regular security updates** and patches

## Resources

### Official Documentation
- [Apache ActiveMQ Website](https://activemq.apache.org/)
- [ActiveMQ Documentation](https://activemq.apache.org/components/classic/documentation)
- [JMS API Documentation](https://docs.oracle.com/javaee/7/api/javax/jms/package-summary.html)

### Community
- [ActiveMQ Mailing Lists](https://activemq.apache.org/mailing-lists)
- [Stack Overflow - ActiveMQ Tag](https://stackoverflow.com/questions/tagged/activemq)
- [GitHub Repository](https://github.com/apache/activemq)

### Tutorials
- [Spring Boot ActiveMQ Integration](https://spring.io/guides/gs/messaging-jms/)
- [ActiveMQ Examples](https://github.com/apache/activemq/tree/main/assembly/src/release/examples)

## Next Steps

- Explore [ActiveMQ Interview Questions](ActiveMQ-Interview-Questions.md) for in-depth knowledge
- Practice with code examples
- Set up a local ActiveMQ instance
- Build a sample microservices application

---

**Last Updated**: 2025
**Version**: 5.18.x
**License**: Apache License 2.0
