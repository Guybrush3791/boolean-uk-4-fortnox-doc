# Kafka in SpringBoot

[Spring for Apache Kafka](https://docs.spring.io/spring-kafka/reference/) provides Spring-friendly producers, consumers and configuration around the Apache Kafka client. Spring Boot supplies the default beans from application properties, while the application describes its topics and business behaviour.

This lesson implements three delivery strategies:

- **plain** for one processing path;
- **parallel** for sharing work across consumers;
- **fan-out** for sending the same event to independent groups.

The examples use `String` keys and values so that the focus remains on topic, partition and consumer-group behaviour. Production systems commonly use JSON, Avro or another structured event format.

Complete **Devbox and project setup** from the lesson navigation below before continuing. The following steps use the generated project and its Devbox shell.

## Configure the application with YAML

Replace the generated `src/main/resources/application.properties` with `src/main/resources/application.yaml`. Keep only the YAML configuration, rather than defining the same settings in both files:

```yaml file:application.yaml
server:
  port: 4000
spring:
  web:
    error:
      include-message: always
      include-binding-errors: always
      include-stacktrace: never
      include-exception: false
  kafka:
    bootstrap-servers: "localhost:9092"
    consumer:
      group-id: demo
      auto-offset-reset: earliest
```

- `server.port` makes the HTTP endpoints available at `http://localhost:4000`, separately from Kafka's port `9092`
- YAML indentation groups related settings. `spring.web.error` controls which details appear in Spring Boot's default error responses: messages and binding errors are included, stack traces and exception class names are not
- `bootstrap-servers` is the initial broker address used to discover the Kafka cluster
- `group-id` becomes the default consumer group for listeners that do not declare their own group
- `auto-offset-reset: earliest` starts a new group at the oldest retained record when it has no committed offset. It does not rewind a group that already has an offset

## Start Kafka and the application

In the project-root Devbox shell, start the services:

```sh file:"Start the local Kafka broker"
devbox services up
```

Leave this terminal open. Wait for `kafka-init` to complete successfully and the `kafka` readiness check to pass. In a second terminal, open the same project root.

Add the application classes described below under `dev.wows.buk.JavaKafka`, using separate subpackages for the three strategies. Keep them below the main application's package so Spring can discover them. Then run the application in this second terminal:

```sh file:"Run the application"
devbox shell
./gradlew bootRun
```

Use Postman to verify each strategy after adding it. For the plain example, send **GET** to `http://localhost:4000/send` and add `message` = `hello` in **Params**. The response should be `sent: hello`, and the application listener should print `hello`. The listener output, not the HTTP response alone, demonstrates consumption.

### Stop or reset the local environment

Stop `bootRun` with Ctrl+C and stop the services from the project root with `devbox services stop`. Normal shutdown keeps Kafka data for the next run.

> [!warning] Optional destructive reset
> Only after stopping the application and Kafka, run `devbox run kafka-reset` if you deliberately want to delete all local topics, events and consumer offsets stored in `.kafka`. Then run `devbox services up` to initialize a clean broker. Resetting is not required for normal startup.

## The common Spring Kafka flow

Each strategy uses the same Spring components:

1. A configuration class declares a `NewTopic` bean.
2. A service publishes a value with `KafkaTemplate`.
3. A method annotated with `@KafkaListener` receives records.
4. A controller exposes an HTTP endpoint that calls the producer service.

`KafkaTemplate.send(...)` is asynchronous. Returning `"sent"` immediately means the send was started; it does not by itself prove that the broker acknowledged the record. Applications that must report the result should inspect the returned future and handle failures.

# Plain strategy

## Purpose

The plain strategy is the smallest useful Kafka flow: one producer writes to one topic and one consumer handles the records in order.

Use it when:

- the workload does not need parallel consumers;
- one application owns the processing step;
- preserving a simple, ordered flow is more important than throughput;
- introducing Kafka concepts before partitioning and groups.

The topic has one partition and one replica. One partition gives a single ordered log; one replica is suitable for local development but does not provide broker-failure redundancy.

```java file:KafkaMessageConfig.java hlt:4-6
@Configuration
public class KafkaMessageConfig {

    public static final String TOPIC = "topic-test-1";

    @Bean
    public NewTopic getMessageTopic() {
        return TopicBuilder
                .name(TOPIC)
                .partitions(1) // [!note] One ordered log and one active consumer per group
                .replicas(1)   // [!note] Local single-broker project
                .build();
    }
}
```

`KafkaTemplate<String, String>` is injected into the service. Spring Boot creates and configures the template from the Kafka properties.

```java file:KafkaSender.java hlt:10
@Service
public class KafkaSender {

    private final KafkaTemplate<String, String> kafkaTemplate;

    public KafkaSender(KafkaTemplate<String, String> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    public String sendMessage(String message) {
        kafkaTemplate.send(KafkaMessageConfig.TOPIC, message); // [!note] Publish a value without an explicit key
        return "sent: " + message;
    }
}
```

A listener subscribes to the same topic. Because no `groupId` is present on the annotation, the listener uses the default `demo` group from `application.yaml`.

```java file:KafkaReceiver.java hlt:4
@Component
public class KafkaReceiver {

    @KafkaListener(topics = KafkaMessageConfig.TOPIC)
    public void readMessage(String message) {
        System.err.println(message); // [!note] Replace logging with business processing in a real application
    }
}
```

The controller is only an HTTP adapter. Kafka-specific work stays in the sender service.

```java file:KafkaController.java
@RestController
public class KafkaController {

    private final KafkaSender kafkaSender;

    public KafkaController(KafkaSender kafkaSender) {
        this.kafkaSender = kafkaSender;
    }

    @GetMapping("/send")
    public String sendMessage(@RequestParam String message) {
        return kafkaSender.sendMessage(message);
    }
}
```

Calling `/send?message=hello` publishes `hello`. The listener in the `demo` group consumes it from the topic's only partition.

# Parallel strategy

## Purpose

Parallel consumption increases throughput by splitting a topic across partitions and assigning those partitions to several consumers in the **same consumer group**.

Use it when:

- every event should be handled once by the application group;
- one consumer cannot process the workload quickly enough;
- independent keys can be processed concurrently;
- the application can scale consumers up to the topic's partition count.

The sample creates two partitions and names one shared group:

```java file:KafkaParallelConfig.java hlt:5,10
@Configuration
public class KafkaParallelConfig {

    public static final String TOPIC = "topic-test-parallel";
    public static final String GROUP = "parallel-group";

    @Bean
    public NewTopic getParallelTopic() {
        return TopicBuilder
                .name(TOPIC)
                .partitions(2) // [!note] Allows two consumers in the group to work at the same time
                .replicas(1)
                .build();
    }
}
```

Both listener methods use the same topic and group. Kafka assigns each partition to one consumer in the group, so each record is handled by **A or B**, never both during normal processing.

```java file:KafkaParallelReceiver.java hlt:5-6,13-14
@Component
public class KafkaParallelReceiver {

    @KafkaListener(
        topics = KafkaParallelConfig.TOPIC,
        groupId = KafkaParallelConfig.GROUP
    )
    public void readMessageA(ConsumerRecord<String, String> record) {
        print("A", record);
    }

    @KafkaListener(
        topics = KafkaParallelConfig.TOPIC,
        groupId = KafkaParallelConfig.GROUP
    )
    public void readMessageB(ConsumerRecord<String, String> record) {
        print("B", record);
    }

    private void print(String name, ConsumerRecord<String, String> record) {
        System.err.println(
            name + " <- partition " + record.partition() + ": " + record.value()
        ); // [!note] ConsumerRecord exposes partition and other Kafka metadata
    }
}
```

### Choosing a partition

The producer can let Kafka choose a partition, provide a key, or select a partition explicitly.

```java file:KafkaParallelSender.java
public String sendMessage(String message) {

    kafkaTemplate.send(KafkaParallelConfig.TOPIC, message);
    // No key: the producer chooses partitions for efficient batching.
    // Do not assume strict A/B round-robin delivery.

    // kafkaTemplate.send(KafkaParallelConfig.TOPIC, message, message);
    // A stable key keeps equal keys on the same partition.

    // kafkaTemplate.send(KafkaParallelConfig.TOPIC, UUID.randomUUID().toString(), message);
    // A random key spreads records, but removes useful business ordering.

    // kafkaTemplate.send(KafkaParallelConfig.TOPIC, 0, message, message);
    // An explicit partition gives direct control but couples code to topic layout.

    return "sent: " + message;
}
```

A business identifier such as `orderId` is normally the most useful key. It allows different orders to be processed concurrently while preserving the order of events for the same order.

```java file:KafkaParallelController.java
@GetMapping("/send-parallel")
public String sendMessage(@RequestParam String message) {
    return kafkaParallelSender.sendMessage(message);
}
```

> [!warning]
> More listeners do not automatically create more parallelism. With two partitions, only two members of `parallel-group` can have active assignments. A third member remains idle until a rebalance gives it a partition.

# Fan-out strategy

## Purpose

Fan-out allows several independent applications to react to the same event. Kafka achieves this with **different consumer groups**: every group maintains its own offsets and reads the topic independently.

Use it when one event should trigger several concerns, for example:

- an order event updates inventory;
- the same event sends a customer notification;
- another group records analytics or audit data.

Only one partition is needed to demonstrate fan-out. The groups do not divide that partition between each other; each group reads it separately.

```java file:KafkaFanoutConfig.java hlt:5-7,13
@Configuration
public class KafkaFanoutConfig {

    public static final String TOPIC = "topic-test-fanout";
    public static final String GROUP_ONE = "fanout-group-1";
    public static final String GROUP_TWO = "fanout-group-2";

    @Bean
    public NewTopic getFanoutTopic() {
        return TopicBuilder
                .name(TOPIC)
                .partitions(1)
                .replicas(1)
                .build();
    }
}
```

The two methods subscribe to the same topic with different group IDs. Consequently, **both** receive every record.

```java file:KafkaFanoutReceiver.java hlt:4,10
@Component
public class KafkaFanoutReceiver {

    @KafkaListener(
        topics = KafkaFanoutConfig.TOPIC,
        groupId = KafkaFanoutConfig.GROUP_ONE
    )
    public void readMessageOne(String message) {
        System.err.println("group-1 <- " + message);
    }

    @KafkaListener(
        topics = KafkaFanoutConfig.TOPIC,
        groupId = KafkaFanoutConfig.GROUP_TWO
    )
    public void readMessageTwo(String message) {
        System.err.println("group-2 <- " + message);
    }
}
```

The producer is unchanged apart from the topic name:

```java file:KafkaFanoutSender.java hlt:3
public String sendMessage(String message) {
    kafkaTemplate.send(KafkaFanoutConfig.TOPIC, message);
    return "sent: " + message;
}
```

```java file:KafkaFanoutController.java
@GetMapping("/send-fanout")
public String sendMessage(@RequestParam String message) {
    return kafkaFanoutSender.sendMessage(message);
}
```

Calling `/send-fanout?message=hello` produces output from `fanout-group-1` **and** `fanout-group-2`. Kafka does not need two physical copies of the record in the topic; each group simply tracks and advances its own offset.

## Comparing the three strategies

| Strategy | Topic partitions | Consumer groups | Result for one event |
| --- | ---: | ---: | --- |
| **Plain** | 1 | 1 | One consumer processes the event. |
| **Parallel** | 2 | 1 shared group | A or B processes the event; work is distributed. |
| **Fan-out** | 1 | 2 independent groups | Both groups process the event. |

These patterns can be combined. A production topic might have many partitions, while several independent groups each run several consumers. Kafka then provides fan-out **between groups** and parallelism **within each group**.

---

# Links
![[Lessons/2 - Java Back-end/Day 18/__blocks/Links]]
