# Kafka Streams and materialized values

## Project

Create a brand-new `JavaKafkaStreams` project for this exercise. Do not continue the Day 18 exercise, reuse `JavaKafkaSpring`, fork a starter, clone an exercise repository or copy an existing project. The previous lesson is a setup reference only.

## Learning Objectives

- Define a Kafka Streams topology with `@EnableKafkaStreams`, `StreamsBuilder` and `KStream`
- Complete an HTTP to Kafka to Streams to listener path
- Transform one input stream into several output topics
- Group records by a fixed key and by a value-derived key
- Build running counts as materialized values
- Reconstruct an in-memory key/value view after an application restart
- Use topic compaction to retain the latest value for every key

## Instructions

### 1. Generate a brand-new Spring Boot project

If `spring` is not available yet, use [[2 - Java SpringBoot GitHub Action Example#Prepare the smallest useful Devbox|the earlier tools-only Devbox and Spring Boot CLI setup]] to make the command available. Use only that tool environment. Do not continue its `JavaCiCd` project.

Move to the parent directory where you keep academy exercises. Confirm that the current directory is outside the Day 18 project and any existing Git repository, and that it does not already contain a `JavaKafkaStreams` directory. Then run:

```sh file:"Create a new JavaKafkaStreams project"
spring init \
  --type=gradle-project \
  --language=java \
  --boot-version=4.1.1 \
  --group-id=dev.wows.buk \
  --artifact-id=JavaKafkaStreams \
  --name=JavaKafkaStreams \
  --package-name=dev.wows.buk.JavaKafkaStreams \
  --java-version=21 \
  --dependencies=web,devtools,kafka,kafka-streams \
  --extract \
  JavaKafkaStreams
```

The command creates an independent project. Enter its root:

```sh file:"Enter the new project"
cd JavaKafkaStreams
```

Do not initialise this exercise by copying Java files, commits or configuration from the Day 18 project.

### 2. Add the Kafka development environment

Follow [[Lessons/2 - Java Back-end/Day 18/Arguments/1 - Devbox and project setup#Add the Kafka development environment|the previous lesson's Kafka development-environment setup]] to create a new `devbox.json` and `process-compose.yaml` inside this `JavaKafkaStreams` project. Use only the two complete configuration-file blocks from that section.

Names such as `JavaKafkaSpring` in the surrounding Day 18 prose refer to the previous lesson. For this exercise, both files belong in the new `JavaKafkaStreams` root. Do not run the previous lesson's project-generation command, follow its final instruction to add Day 18 code or use that project as a starting point.

Leave any Devbox shell that belongs to another project. From the new `JavaKafkaStreams` root, enter this project's shell and verify the generated application:

```sh file:"Verify the new project"
devbox shell
./gradlew test
```

Open this new directory in IntelliJ with Java 21 as the project SDK.

### 3. Configure the application

Replace `src/main/resources/application.properties` with the complete `src/main/resources/application.yaml`:

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
      group-id: support-message-view
      auto-offset-reset: earliest
    streams:
      application-id: "support-message-streams"
      state-store-cache-max-size: 0
      state-dir: .kafka-streams
      properties:
        dsl:
          store:
            suppliers:
              class: org.apache.kafka.streams.state.BuiltInDslStoreSuppliers$InMemoryDslStoreSuppliers
```

Create the exercise code below `dev.wows.buk.JavaKafkaStreams` so Spring discovers it. Use separate packages for the controller, Kafka sender and receivers, topic configuration and stream topology.

### 4. Start the services

From the project-root Devbox shell:

```sh file:"Start Kafka"
devbox services up
```

Leave that terminal open. Wait for `kafka-init` to finish successfully and for `kafka` to pass its readiness check.

In a second terminal, enter the same new project and prepare its shell:

```sh file:"Open the application shell"
devbox shell
```

Do not start the application yet. Implement the first complete path in the activity, then run it when instructed. Use Postman against `http://localhost:4000`. Check listener output after every request. A successful HTTP request proves that the sender ran, not that the complete stream was processed.

> [!warning] Optional destructive reset
> With the application and Kafka stopped, `devbox run kafka-reset` deletes the local topics, events and consumer offsets. Use it only when an instruction explicitly requires a clean broker.

## Activity

You are building a simplified **support-message analytics application** entirely inside the new project.

A support message is represented by a string:

```text
Kafka keeps events
Streams keep state
Kafka streams process events
```

Every HTTP request publishes one message to `support-messages`. Complete and verify the first uppercase path before adding the stateful counting paths.

### Core

#### 1. Complete the input and uppercase path

Declare these topics with one partition and one replica:

- `support-messages`
- `support-messages-uppercase`
- `support-message-count`
- `support-word-frequency`

Start `support-word-frequency` with the default cleanup policy. Compaction is added only during the recovery task.

Implement the input path:

- create a sender service that publishes string values to `support-messages` with `KafkaTemplate`
- create a controller with `GET /kafka/support`
- accept the support message in a `message` query parameter
- delegate from the controller to the sender

Enable Kafka Streams and create the source stream from `support-messages`. Configure string Serdes for the source and filter null or blank messages before producing any output.

Add the first output path:

- convert each valid message to uppercase
- write the transformed value to `support-messages-uppercase`
- use string Serdes for the output key and value
- add a listener that logs `UPPERCASE <- <message>`

Start the application from the second terminal:

```sh file:"Start the application"
./gradlew bootRun
```

In Postman, send a **GET** request to `http://localhost:4000/kafka/support` with `message` = `Kafka keeps events` in **Params**.

Close the first circle before continuing. Confirm that the HTTP request reaches the sender and that the listener prints:

```text
UPPERCASE <- KAFKA KEEPS EVENTS
```

#### 2. Build the running message count

Stop the application before changing its source. Keep Kafka running. Restart the application with `./gradlew bootRun` when this path is ready to verify.

Add a stateful output path for `support-message-count`:

1. group every valid message under the fixed key `"messages"`
2. use `count()` to maintain one running total
3. convert the aggregate back to a stream
4. convert the count to a string value
5. write each update to `support-message-count`
6. add a listener that logs `MESSAGE COUNT <- <count>`

Send three requests. The count listener should progress through:

```text
MESSAGE COUNT <- 1
MESSAGE COUNT <- 2
MESSAGE COUNT <- 3
```

Explain why this path produces one total rather than one count for each different message.

#### 3. Build the word-frequency view

Stop the application before adding this path. Keep Kafka running. Restart the application when the path is ready to verify.

Add a stateful output path for `support-word-frequency`:

1. split each valid message into words
2. remove empty words
3. convert every word to lowercase
4. use the normalized word as the record key
5. group equal word keys
6. count each group
7. convert each count to a string
8. write `<word, count>` updates to `support-word-frequency`

For these two messages:

```text
Kafka keeps events
Kafka streams process events
```

The output topic should receive updates equivalent to:

```text
kafka -> 1
keeps -> 1
events -> 1
kafka -> 2
streams -> 1
process -> 1
events -> 2
```

The order of independent output records can vary. Confirm that the materialized result maintains one latest count per normalized word rather than one total shared by all words.

#### 4. Rebuild the word-frequency view

Create a dedicated `WordFrequencyReceiver` component for `support-word-frequency`.

First, use a normal `@KafkaListener` that:

- receives `ConsumerRecord<String, String>` so it can read both the word key and count value
- stores the latest values in a `Map<String, Long>`
- logs the complete map after every update

Publish several messages and confirm that a later count replaces the earlier value for the same word while unrelated keys remain in the map.

Stop and restart only the Spring Boot application. The Java map starts empty, while the consumer group resumes from its committed offsets. Publish a message containing only some previously seen words and observe which keys are present in the rebuilt map.

Then extend `WordFrequencyReceiver` from `AbstractConsumerSeekAware`. Override `onPartitionsAssigned(...)` so it:

- clears the in-memory map
- seeks the assigned `support-word-frequency` partitions to the beginning

Restart the application and observe the retained output records being replayed into the map. Explain why a word cannot be reconstructed if its latest record is no longer retained by Kafka. Record that the missing key produces no error, then state whether silent expiry is desirable for this analytics view.

Finally:

1. add `.compact()` to the `support-word-frequency` topic declaration
2. stop the application and Kafka
3. run `devbox run kafka-reset`
4. remove the local Streams metadata with `rm -rf .kafka-streams`
5. start Kafka and the application again so Spring recreates the topics
6. inspect the topic configuration with:

```sh file:"Verify the compacted topic"
kafka-configs.sh \
  --bootstrap-server localhost:9092 \
  --entity-type topics \
  --entity-name support-word-frequency \
  --describe
```

Confirm that the description includes `cleanup.policy=compact`. This proves the topic configuration. Kafka compaction runs asynchronously, so it does not prove that older record versions have already been removed.

Continue the recovery check:

7. republish the sample messages
8. restart only the application

Confirm that replaying the compacted topic reconstructs the latest retained count for every word key, including words that receive no new update after the restart. Older updates may also be replayed until Kafka compacts the log, but later updates must replace them in the Java map. Record whether startup becomes slower while retained records are replayed.

When the exercise is complete, stop the application with Ctrl+C and stop Kafka from the project root with `devbox services stop`. Normal shutdown preserves Kafka events and consumer offsets. The configured materialized stores are in memory, so Kafka Streams reconstructs them from its Kafka changelog topics after a restart.

## Acceptance Criteria

- The work starts from the newly generated `JavaKafkaStreams` project, not the Day 18 exercise or another repository
- All input and output topics are declared with `NewTopic` and `TopicBuilder`
- The controller delegates publishing to a sender service
- The sender publishes to `support-messages` through `KafkaTemplate`
- Kafka Streams is enabled with `@EnableKafkaStreams`
- The source stream consumes string keys and values and filters null or blank messages
- The uppercase path writes one transformed output for each valid input message
- The message-count path groups under one fixed key and maintains one running total
- The word-frequency path groups by normalized word and maintains one count per word
- Stream keys and values use appropriate string Serdes when consumed, grouped and produced
- Output listeners make the uppercase, message-count and word-frequency results observable
- `WordFrequencyReceiver` uses `ConsumerRecord` to read both the word key and count value
- `onPartitionsAssigned(...)` clears the local map and seeks assigned partitions to the beginning
- The delete-policy replay limitation and whether silent key expiry is acceptable are explained
- `support-word-frequency` is recreated as a compacted topic for the final restart check
- The live topic configuration reports `cleanup.policy=compact`
- After the final restart, the listener map contains the latest retained value for every word key
- The effect of replay on startup is recorded
- The difference between Kafka Streams materialized state and the listener's Java map is explained in the README or code comments

## Extension

Choose one extension after completing the core activity:

1. **Total words across all messages**: group every extracted word under one fixed key and materialize one running total across the complete stream
2. **Count words per message**: produce one count independently for each input message without carrying state from earlier messages, then compare the output with the running total
3. **Ignore common words**: remove words such as `the`, `a` and `and` before grouping, and confirm that they never appear in the materialized frequency view
4. **A second analytics view**: add another independent output topic and listener without changing the existing three paths
