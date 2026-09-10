# Kafka in SpringBoot

## Project

Create a new project using the Devbox files and `spring init` instructions below. No starter repository or fork is required.

## Learning Objectives

- Explain the roles of Kafka topics, partitions, offsets and consumer groups.
- Publish string events with `KafkaTemplate`.
- Consume events with `@KafkaListener`.
- Distribute work between consumers in one group.
- Fan out one event to several independent consumer groups.
- Use a record key when related events must remain ordered.

## Instructions

### 1. Prepare a fresh environment

Use your existing Devbox installation. On Windows, work inside WSL2. In a new, empty working folder, save these two files together:

```json file:devbox.json
{
  "$schema": "https://raw.githubusercontent.com/jetify-com/devbox/0.16.0/.schema/devbox.schema.json",
  "name": "JavaKafka",
  "description": "Spring Boot + Kafka demo. devbox installs everything, so this works the same on Linux, macOS and Windows (WSL2).",

  "packages": [
    // java 21
    "temurin-bin@21",

    // spring cli
    "spring-boot-cli",

    // kafka server
    "apacheKafka"
  ],

  "env": {
    "JAVA_HOME": "$DEVBOX_PACKAGES_DIR"
  },

  "shell": {
    "scripts": {
      "kafka-reset": [
        "set -e",
        "cd \"$DEVBOX_PROJECT_ROOT\"",
        "rm -rf .kafka",
        "echo 'Kafka data wiped. Start clean with: devbox services up'"
      ]
    }
  }
}
```

```yaml file:process-compose.yaml
version: "0.5"

processes:

  kafka-init:
    command: |
      set -e
      mkdir -p "$DEVBOX_PROJECT_ROOT/.kafka"

      cat > "$DEVBOX_PROJECT_ROOT/.kafka/server.properties" <<EOF
      process.roles=broker,controller
      node.id=1
      controller.quorum.voters=1@localhost:9093

      listeners=PLAINTEXT://:9092,CONTROLLER://:9093
      advertised.listeners=PLAINTEXT://localhost:9092
      controller.listener.names=CONTROLLER
      inter.broker.listener.name=PLAINTEXT
      listener.security.protocol.map=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT

      num.partitions=1
      default.replication.factor=1
      offsets.topic.replication.factor=1
      transaction.state.log.replication.factor=1
      transaction.state.log.min.isr=1
      group.initial.rebalance.delay.ms=0

      log.dirs=$DEVBOX_PROJECT_ROOT/.kafka/data
      EOF

      kafka-storage.sh format \
        --cluster-id "HypWXQWlmzQCLD9iGXLrQQ" \
        --config "$DEVBOX_PROJECT_ROOT/.kafka/server.properties" \
        --ignore-formatted

  kafka:
    command: |
      export KAFKA_HEAP_OPTS="-Xmx512M -Xms256M"
      exec kafka-server-start.sh "$DEVBOX_PROJECT_ROOT/.kafka/server.properties"
    depends_on:
      kafka-init:
        condition: process_completed_successfully
    availability:
      restart: on_failure
      max_restarts: 3
    readiness_probe:
      exec:
        command: "kafka-broker-api-versions.sh --bootstrap-server localhost:9092"
      initial_delay_seconds: 5
      period_seconds: 5
      timeout_seconds: 10
      failure_threshold: 30
```

Devbox installs Java 21, the Spring Boot CLI and Kafka. Process Compose initializes a local single-node Kafka broker, keeps its data in `.kafka`, and checks readiness before you test message delivery. This setup is for local development, not production.

### 2. Generate a new project

Enter the environment:

```sh file:"Enter the Devbox shell"
devbox shell
```

Run the following command in that shell. Use a fresh folder rather than modifying your earlier CI/CD project:

```sh file:"Create the Kafka project"
spring init \
  --type=gradle-project \
  --language=java \
  --boot-version=4.1.1 \
  --group-id=dev.wows.buk \
  --artifact-id=JavaKafka \
  --name=JavaKafka \
  --package-name=dev.wows.buk.JavaKafka \
  --java-version=21 \
  --dependencies=web,devtools,kafka \
  --extract \
  JavaKafka
```

`web` supplies Spring MVC, `devtools` supports local development, and `kafka` supplies Spring for Apache Kafka. The generated project includes the Gradle Wrapper and test dependencies. The Devbox environment and application folder are named `JavaKafka`; the base Java package is `dev.wows.buk.JavaKafka`.

Leave this shell, move the environment files into the generated project, and re-enter Devbox from the project root before starting Kafka:

```sh file:"Prepare the project root"
exit
mv devbox.json devbox.lock process-compose.yaml JavaKafka/
cd JavaKafka
devbox shell
./gradlew test
```

Open this `JavaKafka` folder in IntelliJ with Java 21 as the project SDK. Keep `devbox.lock` in version control to record resolved package versions. Add `.devbox/` and `.kafka/` to the generated `.gitignore`; keep the Gradle Wrapper tracked.

### 3. Configure the application

Replace `src/main/resources/application.properties` with `src/main/resources/application.yaml`:

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

The application serves HTTP on port `4000` and connects to Kafka on port `9092`. Listeners without an explicit group use `demo`. `earliest` applies when a group has no committed offset; it does not rewind an existing group.

### 4. Start the services and implement the activity

From the project-root Devbox shell:

```sh file:"Start Kafka"
devbox services up
```

Leave that terminal open. Wait for `kafka-init` to finish successfully and `kafka` to pass its readiness check. In a second terminal, open the same project root and enter its Devbox shell with `devbox shell`.

Create your strategy packages below `dev.wows.buk.JavaKafka` so Spring discovers their classes. After implementing each strategy, start the application in the second terminal with `./gradlew bootRun`. Use Postman against `http://localhost:4000`, with the endpoint paths and request parameters you define. Check listener output as well as the HTTP response: starting a send does not prove consumption.

Stop the application with Ctrl+C and stop Kafka from the project root with `devbox services stop`. Normal shutdown preserves events and offsets.

> [!warning] Optional destructive reset
> Only with the application and Kafka stopped, use `devbox run kafka-reset` to delete all local topics, events and consumer offsets in `.kafka`. Start again with `devbox services up`. Do not reset data as part of normal startup.

## Activity

You are working on a simplified **order-event application**. Use the new project and local Kafka broker you prepared above to implement the three strategies.

An order event is represented by a string during this exercise:

```text
ORDER-123:CREATED
ORDER-123:PAID
ORDER-456:CREATED
```

Create separate packages for the **plain**, **parallel** and **fan-out** strategies. Each package should contain only the configuration, producer, consumer and controller code needed by that strategy.

### Core

#### 1. Plain order audit

Create a topic with:

- the name `order-events-plain`
- one partition
- one replica

Implement:

- an HTTP endpoint that accepts an event string
- a producer service that publishes the event with `KafkaTemplate`
- one listener that receives the event and logs `AUDIT <- <event>`

Confirm that the endpoint can publish several events and that the listener receives them in order.

#### 2. Parallel order processing

Create a topic with at least **two partitions** and a consumer group named `order-processing-group`.

Implement:
- an endpoint and producer for the parallel topic
- two listeners in `order-processing-group`
- output that includes the listener name, record partition and event value

Both listeners must share the work: each event is processed by one listener in the group, not by both.

Publish several events and observe which partition and listener process each one. Do not assume that unkeyed records alternate strictly between listeners.

Then change the producer to use the order ID as the record key. For example, both `ORDER-123:CREATED` and `ORDER-123:PAID` should use `ORDER-123` as their key.

Confirm that:

- events for one order are assigned to the same partition
- their order is preserved inside that partition
- events for different orders can be handled in parallel

#### 3. Fan-out order notifications

Create a fan-out topic and two independent consumer groups:

- `customer-notification-group`
- `analytics-group`

Implement one listener for each group:

- the notification listener logs `NOTIFICATION <- <event>`
- the analytics listener logs `ANALYTICS <- <event>`

Publish an event once and confirm that **both groups** receive it. The groups must not share the event as competing workers.

## Acceptance Criteria

- All topics are declared from Spring configuration with `NewTopic` and `TopicBuilder`
- Producers publish through `KafkaTemplate` rather than calling listeners directly
- Consumers use `@KafkaListener`
- Parallel listeners use the same group ID
- Fan-out listeners use different group IDs
- Parallel output identifies the partition that handled each event
- Events with the same order ID key remain on the same partition
- Controllers delegate Kafka work to producer services

## Extension

Choose one extension after completing the core activity:
1. **Structured events**: replace the string value with an `OrderEvent` object and configure JSON serialization/deserialization
2. **A third parallel consumer**: add a third consumer to a two-partition topic, observe its assignment, then increase the partition count and explain the change
3. **Independent parallel groups**: give both fan-out groups several consumers and dig into how fan-out between groups combines with parallelism inside each group
4. **Producer result handling**: inspect the future returned by `KafkaTemplate.send(...)` and report whether the broker acknowledged the record