# Devbox and project setup

## Prepare the Devbox environment

The Spring application needs a running Kafka broker as well as its Java dependencies. Devbox supplies Java 21, the Spring Boot CLI and Kafka; Process Compose, used by `devbox services`, starts the local broker in the right order.

Use the Devbox installation from the environment-setup lesson. On Windows, run these commands inside WSL2. Start in a new, empty working folder, not inside the earlier CI/CD project, and save these two files together.

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

### Read the environment files

- `temurin-bin@21` supplies the Java runtime and compiler; `spring-boot-cli` supplies `spring init`; `apacheKafka` supplies the broker and Kafka command-line tools
- Devbox accepts the comments shown in `devbox.json`. `JAVA_HOME` points to its package directory, and `DEVBOX_PROJECT_ROOT` identifies the folder containing the active `devbox.json`
- `shell.scripts.kafka-reset` defines the optional `devbox run kafka-reset` command. It does not run automatically
- `kafka-init` writes the broker configuration and formats its local storage. The `<<EOF` block writes several lines to one file; `$DEVBOX_PROJECT_ROOT` expands to the project path
- This single process is both a **broker**, which stores events, and a **controller**, which manages cluster metadata. This is Kafka's KRaft mode, so no ZooKeeper service is needed
- Port `9092` accepts application connections; port `9093` is used by the controller. The fixed cluster ID identifies this local development cluster
- Replication settings use one copy because only one broker is running. The `.kafka` folder holds its configuration and persistent event data
- `--ignore-formatted` allows startup to reuse already formatted storage; it does not clear existing events or offsets
- `depends_on` starts Kafka only after initialization succeeds. The readiness probe checks that the broker answers requests; the restart policy retries a failed broker process up to three times

This is a local, single-broker setup without authentication or encryption, not a production configuration. Keep the generated `devbox.lock` with the project to record resolved package versions; the unversioned package names alone do not pin Kafka or the Spring CLI.

## Generate the Spring Boot project

Enter the environment:

```sh file:"Enter the Devbox shell"
devbox shell
```

Create a new project named `JavaKafka` using the familiar generator command, now with the `kafka` dependency:

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

`web` adds Spring MVC, `devtools` adds local development support, and `kafka` adds Spring for Apache Kafka. The generated `build.gradle` also includes the corresponding test dependencies. No manual dependency block or separate Kafka Streams dependency is needed for these examples. The project includes the Gradle Wrapper, so Devbox does not need a Gradle package.

The Devbox environment and generated application folder are named `JavaKafka`; the base Java package is `dev.wows.buk.JavaKafka`.

Leave the temporary Devbox shell, move the environment files into the generated project, then enter a fresh shell there. Do this before starting services so `DEVBOX_PROJECT_ROOT` points to the application root:

```sh file:"Place Devbox at the project root"
exit
mv devbox.json devbox.lock process-compose.yaml JavaKafka/
cd JavaKafka
devbox shell
./gradlew test
```

At this point the generated context test should pass. It checks the starting Spring application, not message delivery. Open this `JavaKafka` folder in IntelliJ and select Java 21 as the project SDK.

Keep `devbox.json`, `devbox.lock`, `process-compose.yaml` and the Gradle Wrapper in version control. Add `.devbox/` and `.kafka/` to the generated `.gitignore` so installed packages and local Kafka data are not committed.

The environment and generated project are now ready. Continue with **Kafka in SpringBoot** in the lesson navigation below to configure `application.yaml`, start Kafka and implement the messaging strategies.

---

# Links
![[Lessons/2 - Java Back-end/Day 18/__blocks/Links]]
