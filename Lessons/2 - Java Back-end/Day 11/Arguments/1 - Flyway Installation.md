# Flyway Installation

The Day 10 Devbox setup gave us a consistent local PostgreSQL server and DBeaver client. We will now extend that same environment with Flyway so database schema changes can be applied from versioned files.

Before continuing, place the `devbox.json` and `process-compose.yaml` files from Day 10 in the root folder where you will work through the migrations lesson.

## What is new in this configuration

The existing Java, PostgreSQL and DBeaver entries stay in place. The new Flyway-specific components are:

- `flyway` installs the Flyway command-line tool inside the Devbox environment.
- `flyway.toml` tells Flyway how to connect to the local `booleandb` database and where to find migration files.

Together, these additions let everyone apply the same ordered schema changes against the same local database setup.

## `devbox.json`

Add `flyway` after the existing database packages in the `packages` array:

```json file:devbox.json hlt:3
    "postgresql@17",
    "dbeaver-bin",
    "flyway"
```

This is the only change required in `devbox.json`.

## Enter the updated environment

Enter or restart the Devbox shell so the Flyway command becomes available:

```sh file:"Enter the Devbox shell"
devbox shell
```

Check the installed version:

```sh file:"Check the Flyway installation"
flyway --version
```

The exact version can change when Devbox resolves the package. The important result is that the command runs inside the project environment.

Run Flyway without a subcommand to display the available command-line options:

```sh file:"Show the Flyway options"
flyway
```

## `flyway.toml`

Create `flyway.toml` in the same root folder as `devbox.json`:

```toml file:flyway.toml
[environments.local]
url = "jdbc:postgresql://localhost:5432/booleandb"
user = "postgres"
password = ""

[flyway]
environment = "local"
locations = ["filesystem:db/migration"]
```

The configuration has two responsibilities:

- `[environments.local]` tells Flyway how to connect to the local PostgreSQL database configured on Day 10.
- `[flyway]` selects that environment and tells Flyway to read migration files from `db/migration`.

> [!attention] Password-free local setup
> The empty password matches the local classroom database. Use this only for local development. A shared or production database must use authentication, and real credentials must not be committed to Git.

Run Flyway commands from this root folder so it can find both `flyway.toml` and `db/migration`.

---

# Links
![[Lessons/2 - Java Back-end/Day 11/__blocks/Links]]
