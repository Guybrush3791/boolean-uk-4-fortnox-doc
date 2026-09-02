# From in-memory data to Postgres

## Learning Objectives

- Explain why an in-memory repository cannot preserve data between application runs.
- Trace a request through the controller, service and repository to PostgreSQL.
- Prepare the Spring Boot project to use the PostgreSQL JDBC driver.

## Reuse the existing local database stack

There is no new Devbox dependency for this lesson. Continue with the Java, PostgreSQL, DBeaver and Flyway environment used on the previous day.

Start the local PostgreSQL service before running migrations or the Spring Boot application:

```sh file:"Start PostgreSQL"
devbox services up
```

The application and Flyway use the same local connection:

| Setting | Value |
| --- | --- |
| Host | `localhost` |
| Port | `5432` |
| Database | `booleandb` |
| Username | `postgres` |
| Password | Leave blank |

DBeaver lets you inspect the schema and data. Flyway applies schema changes. The Java application connects to the same server through JDBC.

## Add the PostgreSQL driver

Create the Spring Boot project from the folder that should contain it:

```sh file:"Create the Spring Boot project"
spring init \
  --type=gradle-project \
  --language=java \
  --boot-version=4.1.1 \
  --group-id=dev.wows.buk \
  --artifact-id=JavaDB \
  --name=JavaDB \
  --package-name=dev.wows.buk.JavaDB \
  --java-version=21 \
  --dependencies=web,devtools,postgresql \
  --extract \
  JavaDB2
```

The generated `build.gradle` declares the PostgreSQL driver with `runtimeOnly`. Change that one dependency declaration to `implementation`:

```groovy file:build.gradle
implementation 'org.postgresql:postgresql'
```

JDBC (Java Database Connectivity) is the Java API used to communicate with a relational database. This project imports `PGSimpleDataSource` directly, so the PostgreSQL driver must be available while the Java source is compiled rather than only while the application runs.

## Keep the existing HTTP configuration

The application continues to listen on port `4000` and includes useful request error messages without returning stack traces:

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
```

The next step is to create the database tables before Java tries to query them.

---

# Links
![[Lessons/2 - Java Back-end/Day 12/__blocks/Links]]
