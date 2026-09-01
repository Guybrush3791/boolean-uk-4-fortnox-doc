# Java Migrations Workshop

## Learning Objectives

- Understand that database migrations are a way to control incremental changes to a database schema
- Understand that database migrations offer protection against destructive operations
- Understand that you can version control them as well as keep a log of the migrations so that it is possible to see a complete history of all of the various versions of your database
- Understand that it is also possible to integrate database migrations directly into your Java code so that it runs from there rather than as a separate application

## Introduction

In the previous lesson, you configured PostgreSQL and DBeaver in Devbox. You then created tables and ran SQL directly against the local `booleandb` database.

That works while we are exploring SQL, but it leaves an important problem: the database remembers the final structure, not the ordered steps that produced it. Another developer cannot reliably reproduce those changes just by cloning the project.

A **database migration** is a versioned file that describes one change to the database schema. Flyway reads the migration files in version order, applies the ones that have not run yet, and records the result in a `flyway_schema_history` table.

The mental model is:

1. Write one schema change in a migration file.
2. Give the file a versioned name.
3. Run `flyway migrate`.
4. Flyway applies each pending version and records it.

Today we will use Flyway from the command line. It can also be integrated into Java so migrations run from the application, but that is outside this workshop.

## Before the first migration

Complete the [[1 - Flyway Installation|Flyway installation]] prerequisite before continuing. Open the folder containing `devbox.json`, `process-compose.yaml` and `flyway.toml` in VSCode so its terminal starts at the root of the workshop environment.

Start the same local PostgreSQL service used on Day 10:

```sh file:"Start PostgreSQL"
devbox services up
```

Keep that terminal running. The database should accept connections with the following values:

| Setting | Value |
| --- | --- |
| Host | `localhost` |
| Port | `5432` |
| Database | `booleandb` |
| Username | `postgres` |
| Password | Leave blank |

Use DBeaver to inspect the database while Flyway makes changes. DBeaver is the client that lets you see and query the schema; Flyway is the tool that applies and records the ordered schema changes.

Run every Flyway command from this root folder so it can find the configuration and migration directory prepared in the prerequisite.

## Create the initial Books table

Start with one monolithic `Books` table:

**Books**

**id**, title, author, publisher, year, genre, score, author_email, publisher_location

The initial query is:

```sql file:V1_0_0__initial_table_creation.sql
CREATE TABLE IF NOT EXISTS Books(
	id SERIAL PRIMARY KEY,
	title TEXT NOT NULL,
	author TEXT,
	publisher TEXT,
	year INT,
	genre TEXT,
	score INT,
	author_email TEXT,
	publisher_location TEXT
);
```

Create `db/migration/V1_0_0__initial_table_creation.sql` and copy the query into it.

The filename tells Flyway how to handle the file:

- `V` marks it as a versioned migration.
- `1_0_0` is the migration version.
- `__` is the required double-underscore separator.
- `initial_table_creation` describes the change for people reading the history.
- `.sql` tells Flyway and the editor that the file contains SQL.

`V` is Flyway's default prefix and can be changed in its configuration. Keep the default throughout this workshop so every migration follows the same convention.

Run the migration from the project root:

```sh file:"Apply pending migrations"
flyway migrate
```

Flyway should apply version `1.0.0`. Refresh the connection in DBeaver and check the observable result:

- `Books` has the columns defined in the migration.
- `flyway_schema_history` records that version `1.0.0` was applied.

The history table is how Flyway knows not to run the same migration again.

## Separate Authors from Books

The monolithic table is enough to store book data, but it repeats author details for every book. We will migrate toward the normalised structure discussed in the relationships lesson.

The complete change has three ordered steps:

1. Create the `Authors` table.
2. Remove the duplicated author columns from `Books`.
3. Add an `author_id` foreign key from `Books` to `Authors`.

A migration file can contain more than one SQL statement. We are using separate versioned files here so that each stage is visible in the history and Flyway can apply the stages in a clear order.

### Version 2.0.0: create Authors

Create `db/migration/V2_0_0__create_authors_table.sql` with this query:

```sql file:V2_0_0__create_authors_table.sql
 CREATE TABLE IF NOT EXISTS Authors (
    id SERIAL PRIMARY KEY,
    name TEXT,
    email TEXT
);
```

This migration creates the table that will own author names and email addresses.

### Version 2.1.0: remove duplicated author details

Create `db/migration/V2_1_0__remove_author_details_from_books.sql` with this query:

```sql file:V2_1_0__remove_author_details_from_books.sql
ALTER TABLE Books
DROP COLUMN author,
DROP COLUMN author_email;
```

This migration removes the author details that no longer belong in `Books`.

> **Data-loss warning:** If `Books` already contains data, dropping these columns destroys the author values stored in them. In a development database with dummy data, you can recreate the data. To preserve production data, you would first create the new table and foreign-key column, copy each author into `Authors`, connect each book to the new author ID, verify the result, and only then remove the old columns. With millions of records, that can be a non-trivial migration.

### Version 2.2.0: connect Books to Authors

Create `db/migration/V2_2_0__add_foreign_key_for_authors_to_books.sql` with this query:

```sql file:V2_2_0__add_foreign_key_for_authors_to_books.sql
ALTER TABLE Books
ADD COLUMN author_id INT;

ALTER TABLE Books
ADD CONSTRAINT fk_author_id FOREIGN KEY (author_id) REFERENCES Authors (id);
```

The first statement gives `Books` an integer `author_id` column. The second names a foreign-key constraint and requires values in `Books.author_id` to reference `Authors.id`.

Apply all three pending versions:

```sh file:"Apply the author migrations"
flyway migrate
```

Flyway checks the current schema history and runs versions `2.0.0`, `2.1.0`, and `2.2.0` in order. Refresh DBeaver and verify the result:

- `Authors` exists.
- `Books.author` and `Books.author_email` no longer exist.
- `Books.author_id` exists and has the foreign-key constraint.

## Inspect the migration history

Ask Flyway for the current state:

```sh file:"Show migration information"
flyway info
```

The output should list the four versioned files applied so far:

- `1.0.0`: initial table creation
- `2.0.0`: create authors table
- `2.1.0`: remove author details from books
- `2.2.0`: add foreign key for authors to books

The output also records when each migration was installed. This is the reproducible history that was missing when we changed the schema directly in DBeaver.

## Clean and rebuild the local schema

If you make a mistake and want to rebuild this local practice schema from the beginning, Flyway can remove the objects from the configured schema:

```sh file:"Clean the local schema"
flyway -cleanDisabled=false clean
```

`clean` is disabled by default because it is destructive. The `-cleanDisabled=false` option enables it for this command. Flyway also provides a persistent configuration setting, but keep the safe default and enable cleaning only for the specific local command. Use it only against the local classroom database after checking the connection in `flyway.toml`.

Run the migrations again to rebuild the schema:

```sh file:"Rebuild the schema"
flyway migrate
```

Flyway recreates the schema by replaying the versioned files in order.

## Rebuild to a target version

Some paid Flyway editions provide `flyway undo`, but that command is not available in the free edition used in this workshop. For this local database, you can achieve a similar learning result by cleaning the schema and migrating only to a selected version.

First clean the local schema:

```sh file:"Clean before targeting a version"
flyway -cleanDisabled=false clean
```

Then migrate through version `2.1`:

```sh file:"Migrate to version 2.1"
flyway -target=2.1 migrate
```

Flyway applies `V1_0_0`, `V2_0_0`, and `V2_1_0`, but leaves `V2_2_0` pending. Confirm that state:

```sh file:"Inspect the pending migration"
flyway info
```

Run `flyway migrate` again to apply the final pending migration.

## Guided task: separate Publishers from Books

Now repeat the same process for publisher data:

1. Create a `Publishers` table containing the publisher details.
2. Remove `publisher` and `publisher_location` from `Books`.
3. Add a `publisher_id` foreign key from `Books` to `Publishers`.

Create migration files with versions `V3_0_0`, `V3_1_0`, and `V3_2_0`. Choose descriptions that explain each change, then use `flyway migrate`, `flyway info`, and DBeaver to verify the result before comparing your work with the examples below.

## Activity

Before starting the activity, keep the Books migrations separate from the new migration sequence. Either change the `locations` value in `flyway.toml`, or move the existing migration files out of `db/migration` and place the activity files there.

Imagine you have a similar data structure for bands and their albums. Start by creating an `Albums` table with logical names and data types for these columns:

- id
- name
- year of release
- highest chart position
- artist/band name
- number of members in the act
- artist/band year founded
- record company name
- record company location
- record company year founded

Create an initial migration file that creates one monolithic table containing all of these columns.

Then create ordered migration files that separate the structure into more logical, normalised tables. Use `flyway migrate`, `flyway info`, and DBeaver to check that each migration produces the structure you intended.

## Assessment

For the afternoon task, follow the same route: begin with the monolithic table described in the exercise, then add versioned migration files until the database has a well-normalised structure.

Full details can be found in the repository. Remember to fork and clone it.

[Afternoon Activity](https://github.com/boolean-uk/java-api-migrations)

## When to use each tool

Use DBeaver when you need to inspect the schema, view data, or test a query while learning. Use Flyway migration files when a schema change must be ordered, repeatable, version controlled, and visible to everyone working on the project.

---

# Links
![[Lessons/2 - Java Back-end/Day 11/__blocks/Links]]
