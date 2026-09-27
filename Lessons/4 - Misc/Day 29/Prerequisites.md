# Prerequisites

Prepare the shared development environment before building the example application or its integration test. This setup provides Java, Spring Boot, PostgreSQL and the generated Gradle test dependencies used throughout the lesson.

# 1. Prepare the development environment

Create a short temporary setup folder and save the following two files inside it. The Spring Boot project will be generated as a child folder and the setup files will then be moved into that project.

```json file:devbox.json
{
  "$schema": "https://raw.githubusercontent.com/jetify-com/devbox/0.16.0/.schema/devbox.schema.json",
  "name": "java-devbox-test1",
  "description": "Java develpment on any os. devbox installs everything, so this works the same on Linux, macOS and Windows (WSL2).",

  "packages": [
    // Java 21 (Eclipse Temurin)
    "temurin-bin@21",
    // Spring Boot CLI
    "spring-boot-cli",
    // Clients
    "curl",
    "postman",

    // database
    "postgresql@17",
    "dbeaver-bin",

    // nodejs
    "nodejs@22"
  ],

  "env": {
    // Points JAVA_HOME at the JDK devbox just installed
    "JAVA_HOME": "$DEVBOX_PACKAGES_DIR",

    "PGUSER": "postgres",
    "PGPORT": "5432",
    "PGDATABASE": "booleandb"
  },
  "shell": {
    "init_hook": [
      "test -f \"$PGDATA/PG_VERSION\" || initdb --username=$PGUSER --auth=trust --encoding=UTF8 --locale=C",

      "java --version"
    ],
    "scripts": {
      "db-shell": "psql",
      "db-reset": "test -n \"$PGDATA\" && rm -rf \"$PGDATA\" && echo 'Database deleted. Run: devbox services up'"
    }
  }
}
```

`devbox.json` supplies Java 21, the Spring Boot CLI and PostgreSQL. The database variables match the application configuration used later in the lesson.

```yaml file:process-compose.yaml
version: "0.5"

processes:
  createdb:
    command: "createdb $DB_NAME || true"
    depends_on:
      postgresql:
        condition: process_healthy
```

The `createdb` process waits for PostgreSQL to become healthy. Because `DB_NAME` is not set, `createdb` uses `PGDATABASE` and creates `booleandb` when it does not already exist.

Enter the temporary setup folder and start the Devbox shell:

```sh file:"Enter the Devbox shell"
devbox shell
```

# 2. Generate the Spring Boot project

Run the following command from inside that Devbox shell:

```sh file:"Create java-int-test"
spring init \
  --type=gradle-project \
  --language=java \
  --boot-version=4.1.1 \
  --group-id=com.booleanuk \
  --artifact-id=java-int-test \
  --name=java-int-test \
  --package-name=com.booleanuk \
  --java-version=21 \
  --dependencies=web,devtools,data-jpa,postgresql,lombok \
  --extract \
  java-int-test
```

Spring Initializr creates a nested `java-int-test` folder. Leave the current shell, move the Devbox files into the generated project root, then enter a new shell from that root:

```sh file:"Move the environment into the generated project"
exit
mv devbox.json devbox.lock process-compose.yaml java-int-test/
cd java-int-test
devbox shell
```

Re-entering the shell matters because `DEVBOX_PROJECT_ROOT` must point at the generated Spring Boot project rather than the temporary parent folder.

The generated `build.gradle` contains both application and test dependencies:

```gradle file:build.gradle
plugins {
    id 'java'
    id 'org.springframework.boot' version '4.1.1'
    id 'io.spring.dependency-management' version '1.1.7'
}

group = 'com.booleanuk'
version = '0.0.1-SNAPSHOT'

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-webmvc'
    compileOnly 'org.projectlombok:lombok'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    runtimeOnly 'org.postgresql:postgresql'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-data-jpa-test'
    testImplementation 'org.springframework.boot:spring-boot-starter-webmvc-test'
    testCompileOnly 'org.projectlombok:lombok'
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
    testAnnotationProcessor 'org.projectlombok:lombok'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

Spring Boot 4 adds test modules that match the selected application starters. `spring-boot-starter-webmvc-test` provides the MVC testing support used by `MockMvc`, while `spring-boot-starter-data-jpa-test` provides testing support for the JPA path. The JUnit Platform launcher allows Gradle to discover and run the tests.

# 3. Connect the application to PostgreSQL

Create the following configuration:

```yaml file:src/main/resources/application.yaml
server:
  port: 4000
spring:
  web:
    error:
      include-message: always
      include-binding-errors: always
      include-stacktrace: never
      include-exception: false
  datasource:
    url: jdbc:postgresql://localhost:5432/booleandb
    username: postgres
    password: ""
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true
```

Start PostgreSQL and keep it running in a separate terminal:

```sh file:"Start PostgreSQL"
devbox services up
```

The tests use this real PostgreSQL service. They do not replace it with an in-memory database or Testcontainers.

---

# Links
![[Lessons/4 - Misc/Day 29/__blocks/Links]]
