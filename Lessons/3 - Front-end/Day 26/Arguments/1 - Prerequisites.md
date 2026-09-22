# Prerequisites

We will use these files to create one development environment for a full-stack project with a Spring Boot back end and a React front end. Devbox provides Java, the Spring Boot CLI, PostgreSQL, database and API clients, and Node.js.

Save both files in the project folder before starting the lesson.

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

The process definition waits for PostgreSQL to become healthy, then runs the database creation command using `DB_NAME`.

```yaml file:process-compose.yaml
version: "0.5"

processes:
  createdb:
    command: "createdb $DB_NAME || true"
    depends_on:
      postgresql:
        condition: process_healthy
```

---

# Links
![[Lessons/3 - Front-end/Day 26/__blocks/Links]]
