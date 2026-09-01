# PostgreSQL Server Installation

The previous Devbox setup gave us a consistent Java development environment. We will now extend that same environment with a database server and a graphical database client.

PostgreSQL runs as a separate service. Our application and database clients connect to that service instead of keeping data only in application memory.

Before continuing, make sure Devbox is installed by following [[1 - Jetify DevBox Installation|Devbox installation]]. Place both configuration files below in the root folder of your project.

## What is new in this configuration

The existing Java, Spring Boot, API client and OpenAPI entries stay in place. The new database-specific components are:

- `postgresql@17` installs the PostgreSQL server and command-line tools. Its Devbox plugin makes the server available as the `postgresql` service.
- `dbeaver-bin` installs DBeaver, a graphical client for connecting to the server, viewing tables and running SQL.
- `PGUSER`, `PGPORT` and `PGDATABASE` define the default user, port and database used by PostgreSQL client commands.
- The `init_hook` runs `initdb` only when the PostgreSQL data directory has not been initialised yet.
- `db-shell` opens the `psql` command-line client.
- `db-reset` deletes the local PostgreSQL data directory so it can be created again.

Together, the Devbox service and DBeaver give us a complete local database environment: PostgreSQL stores and serves the data, while DBeaver gives us a visual client for working with it.

> [!attention] Password-free local setup
> We are not setting a PostgreSQL password in this lesson. The `--auth=trust` option allows local connections without a password, which keeps the classroom setup simple. Use this only for the local development environment. A shared or production database must use authentication.

## `devbox.json`

Use the complete updated configuration below. Focus on the database-specific additions described above rather than relearning the entries from the earlier setup.

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

    // OpenAPI
    "redocly",

    // database
    "postgresql@17",
    "dbeaver-bin"
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
      "script-test": "echo 'Hello, World!'",

      "db-shell": "psql",
      "db-reset": "test -n \"$PGDATA\" && rm -rf \"$PGDATA\" && echo 'Database deleted. Run: devbox services up'"
    }
  }
}
```

## `process-compose.yaml`

Devbox already provides the `postgresql` service when PostgreSQL is installed. This file adds a `createdb` process that waits until the server is healthy before trying to create the lesson database. `|| true` lets the process finish when the database already exists, but it also hides other `createdb` errors, so check the service log if the database is missing.

```yaml file:process-compose.yaml
version: "0.5"
processes:
  createdb:
    command: "createdb $DB_NAME || true"
    depends_on:
      postgresql:
        condition: process_healthy
```

> [!note] Database name
> `DB_NAME` is not defined in this configuration. When it expands to an empty value, the command runs as `createdb`, which uses `PGDATABASE` and creates `booleandb`.

## Start the server

With both files in the project folder, run:

```sh
devbox services up
```

The first run can take longer while Devbox downloads the packages and initialises the data directory. When the `postgresql` log reports `database system is ready to accept connections`, the server is ready on port `5432`.

![[devbox-postgres-up.png]]

Keep this terminal running while you use the database. Open a separate terminal for the application or command-line client.

## Connect with DBeaver

Create a PostgreSQL connection with these values:

| Setting | Value |
| --- | --- |
| Host | `localhost` |
| Port | `5432` |
| Database | `booleandb` |
| Username | `postgres` |
| Password | Leave blank |

This proves both sides of the environment are available: the PostgreSQL service accepts the connection and DBeaver can act as its client.

## If PostgreSQL does not start

A deeply nested project path can make the Unix-domain socket path exceed its `107` byte limit:

```text
LOG: Unix-domain socket path "/home/***/.devbox/virtenv/postgresql/.s.PGSQL.5432" is too long (maximum 107 bytes)
```

Move the project to a shorter path, for example:

- Windows: `C:\Java\<short-project-name>`
- Linux or macOS: `/home/<username>/Java/<short-project-name>`

---

# Links
![[Lessons/2 - Java Back-end/Day 10/__blocks/Links]]
