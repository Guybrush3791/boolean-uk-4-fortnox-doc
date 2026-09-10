# Event Stream in Kafka - part 1

[Apache Kafka](https://kafka.apache.org/) is a **distributed event-streaming platform**. Applications publish events to Kafka, Kafka stores them durably, and one or more applications read and process those events independently.

An event is a record of something that happened: an order was created, a payment was accepted, a sensor changed value, or a user updated a profile. Instead of requiring every application to call every other application directly, Kafka becomes the shared event backbone between them.

## Why event streaming?

In a synchronous system, the caller must know which service should handle a request and usually waits for its response. This creates direct dependencies between services. If a downstream service is slow or unavailable, the caller is affected too.

With event streaming, the producer publishes an event and continues its work. Consumers process that event when they are ready. Producers and consumers can therefore be developed, deployed and scaled separately.

Kafka is useful when a system needs:

- **asynchronous communication** between applications
- **high-throughput event processing**
- **durable event history** that can be replayed
- **horizontal scaling** across several consumers
- **multiple independent reactions** to the same event
- **ordering** for related events within the same partition

![[Sync Event VS Event Driven|1600]]

## How Kafka works

A Kafka cluster contains one or more **brokers**. Producers write records to named **topics**, and each topic is split into one or more **partitions**. A partition is an ordered, append-only log: new records are added at the end and receive an increasing **offset**.

Consumers read records from partitions and keep track of their position. Reading a record does not remove it. Kafka retains records according to the topic's retention policy, so a consumer can resume from its last committed offset or deliberately replay older events.

A simplified event flow is:

1. A **producer** creates an event with a value and, optionally, a key
2. Kafka assigns the event to a **partition** in its topic
3. The broker appends the event to that partition's log
4. A **consumer** reads the event and performs its work
5. The consumer's **group** records an offset representing its progress
6. Other consumer groups can read the same event independently

![[Kafka Message Broker|1600]]

## Core components

| Component | Purpose |
| --- | --- |
| **Event / record** | The data published to Kafka. A record can contain a key, value, timestamp and headers. |
| **Producer** | Publishes records to a topic. |
| **Topic** | A named stream of related records, such as `order-created`. |
| **Partition** | A shard of a topic and the unit of ordering and parallelism. |
| **Offset** | A record's position inside one partition. |
| **Broker** | A Kafka server that stores partitions and serves producers and consumers. |
| **Consumer** | Reads and processes records from one or more partitions. |
| **Consumer group** | A set of consumers cooperating on the same workload. Within a group, one partition is assigned to at most one consumer at a time. |
| **Replication** | Copies partitions across brokers so data can remain available when a broker fails. |

## Ordering, keys and partitions

Kafka guarantees record order **inside one partition**, not across an entire multi-partition topic. A producer key is therefore more than metadata: records with the same key are normally routed to the same partition.

For example, using an `orderId` as the key keeps all events for one order in order, while different orders can be processed in parallel on different partitions. Without a key, the producer chooses partitions to distribute batches efficiently, but the application must not rely on strict round-robin delivery.

Partitions also define the maximum useful parallelism of a consumer group. A topic with two partitions can actively supply at most two consumers in the same group; additional consumers remain idle until an assignment changes.

## Kafka and other message brokers

Kafka can act as a message broker, but its durable log and replay model make it different from a traditional queue-first broker such as RabbitMQ or ActiveMQ.

| Concern | Kafka | Traditional queue broker |
| --- | --- | --- |
| **Primary model** | Distributed event log and stream | Queues with message routing |
| **After consumption** | Records remain until retention removes them | Acknowledged messages are normally removed from the queue |
| **Replay** | Consumers can move their offsets and read events again | Usually requires republishing or a separate dead-letter/archive design |
| **Scaling** | Topics are partitioned; consumers in a group share partitions | Workers compete for messages from a queue |
| **Fan-out** | Different consumer groups read the same topic independently | Exchanges/topics route copies to multiple queues or subscribers |
| **Ordering** | Guaranteed per partition | Usually guaranteed per queue, subject to concurrency and redelivery |
| **Routing** | Topic, key, partition and headers | Often supports rich exchanges, routing keys, selectors and priorities |
| **Best fit** | Event history, analytics, integration streams, high-volume pipelines | Task queues, commands, request/reply and complex routing |

Neither model is universally better. Choose Kafka when event retention, replay, independent consumers and partition-based scale are central requirements. Choose a queue-oriented broker when short-lived commands, per-message routing, priorities or request/reply patterns are more important.

## Kafka in Spring Boot

Start a new project with the supplied Devbox and Process Compose files. Devbox installs Java 21, the Spring Boot CLI and Kafka; `spring init` generates the application, and `application.yaml` connects it to the local broker. The dedicated setup note covers Devbox and project generation; the Kafka lesson then configures the application and teaches the three messaging strategies.

Spring Boot integrates with Kafka through **Spring for Apache Kafka**. The main application-facing components are:

- `KafkaTemplate<K, V>` to publish records;
- `@KafkaListener` to declare consumer methods;
- `NewTopic` and `TopicBuilder` to describe topics;
- application properties for broker addresses and consumer defaults;
- `ConsumerRecord<K, V>` when a listener needs metadata such as the key, partition and offset.

The practical lesson implements three common strategies:

1. **Plain consumption**: one topic, one partition and one consumer.
2. **Parallel consumption**: several partitions shared by consumers in the same group.
3. **Fan-out consumption**: independent consumer groups each receive every event.

## Lesson

[[1 - Devbox and project setup|Devbox and project setup]]

[[2 - Kafka in SpringBoot|Kafka in SpringBoot]]

## Exercise

[[Repository/Day 18/Ex/1 - Kafka in SpringBoot/README|Kafka in SpringBoot]]

---

# Links
![[Lessons/2 - Java Back-end/Day 18/__blocks/Links]]
