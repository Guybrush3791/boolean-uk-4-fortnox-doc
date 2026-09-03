# Create and connect the JPA project

## Learning Objectives

- Reuse the existing Devbox toolchain and PostgreSQL service.
- Generate the Spring Boot project with the Spring Boot CLI.
- Configure Spring Boot to connect to the local `booleandb` database.

## Keep the Devbox environment

There is no new system dependency for this lesson. Continue with the Devbox environment used on the previous database lessons.

| Managed by Devbox | Managed by the generated Gradle project |
| --- | --- |
| Java 21 and the Spring Boot CLI | Spring Web |
| PostgreSQL 17 and its service | Spring Data JPA and Hibernate |
| DBeaver and the command-line clients | PostgreSQL JDBC driver |
| The local `booleandb` database | Lombok and Spring Boot DevTools |

Start PostgreSQL from the folder containing the existing Devbox configuration:

```sh file:"Start PostgreSQL"
devbox services up
```

Keep that process running. Open a second terminal in the same folder and enter the development shell:

```sh file:"Enter the development shell"
devbox shell
```

The local connection remains:

| Setting | Value |
| --- | --- |
| Host | `localhost` |
| Port | `5432` |
| Database | `booleandb` |
| Username | `postgres` |
| Password | Leave blank |

## Generate the project from the terminal

Use the Spring Boot CLI instead of the browser form. Run this inside the Devbox shell:

```sh file:"Create the JPA project"
spring init \
  --type=gradle-project \
  --language=java \
  --boot-version=4.1.1 \
  --group-id=com.booleanuk \
  --artifact-id=api \
  --name=api \
  --package-name=com.booleanuk.api \
  --java-version=21 \
  --dependencies=web,devtools,data-jpa,postgresql,lombok \
  --extract \
  jpa-api
```

The command creates `jpa-api`, extracts the project and includes the Gradle Wrapper. The dependency list gives the project its HTTP layer, JPA implementation, PostgreSQL driver and Lombok annotation processor without manual dependency editing.

Move into the generated project:

```sh file:"Open the generated project"
cd jpa-api
```

All Java classes in this lesson belong under the generated `com.booleanuk.api` package or one of its subpackages. Spring Boot scans those subpackages when the application starts.

## Connect Spring Boot to Devbox PostgreSQL

Create `src/main/resources/application.yaml`:

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

`spring.datasource` now uses the exact PostgreSQL host, port, database and trusted local user created by the Devbox setup. The empty string is the intentionally blank password for this local classroom database.

`ddl-auto: update` tells Hibernate to compare entity mappings with the local schema and add the missing table structure. It also keeps the rows when the application restarts. Use this convenience only for the local workshop database.

`show-sql` and `format_sql` make Hibernate's generated SQL visible while we learn what the ORM is doing. The old `max-active` and `max-idle` entries are not needed for this connection and are not generic `spring.datasource` properties.

Start the empty project once:

```sh file:"Run the application"
./gradlew bootRun
```

A successful start on port `4000` without a PostgreSQL connection error proves that Spring Boot can reach the Devbox database. Stop it with `Ctrl+C` before adding the first entity.

---

# Links
![[Lessons/2 - Java Back-end/Day 13/__blocks/Links]]
