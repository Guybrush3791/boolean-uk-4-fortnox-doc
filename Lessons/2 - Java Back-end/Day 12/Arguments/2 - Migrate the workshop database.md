# Migrate the workshop database

## Learning Objectives

- Apply ordered Flyway migrations to the local PostgreSQL database.
- Create and populate the `customers` and `stock` tables.
- Verify the resulting schema and migration history.

## Configure Flyway for the project

Keep the migrations with the Spring Boot resources so their relationship to the application is visible. Create `flyway.toml` in the project root:

```toml file:flyway.toml
[environments.local]
url = "jdbc:postgresql://localhost:5432/booleandb"
user = "postgres"
password = ""

[flyway]
environment = "local"
locations = ["filesystem:src/main/resources/db/migration"]
```

This configuration points Flyway at the local PostgreSQL instance from the previous day and tells it where to find the migration files.

Create this directory:

```text file:"Migration directory"
src/main/resources/db/migration/
```

Flyway applies versioned files in order. The filename begins with `V`, includes a version number, uses two underscores as a separator, and ends with a description.

## Create the customers table

Create the first migration:

```sql file:V1_0_0__create_customers_table.sql
CREATE TABLE customers (
    id      SERIAL PRIMARY KEY,
    name    TEXT NOT NULL,
    address TEXT NOT NULL,
    email   TEXT NOT NULL,
    phone   TEXT
);
```

`SERIAL` asks PostgreSQL to generate each new `id`. The Java code will therefore supply the customer details but not the identifier.

Populate the table in the next migration:

```sql file:V1_1_0__populate_customers_table.sql
INSERT INTO customers (name, address, email, phone) VALUES
    ('Ada Lovelace', 'Church of St Mary Magdalene, Hucknall, Nottingham, UK', 'ada@lovelace.com', '012345675'),
    ('Charles Babbage', 'Kensal Green, London, UK', 'charles@differenceengine.com', '012345674'),
    ('Grace Hopper', 'Arlington County, Virginia, USA', 'grace@bugsrus.com', '012345673'),
    ('Alan Turing', '43 Adlington Road, Wilmslow, Cheshire, UK', 'alan@bletchleypark.org.uk', '012345672'),
    ('Katherine Johnson', 'Newport News, Virginia, USA', 'katherine@nasa.org', '012345671');
```

The insert omits `id`, so PostgreSQL generates the values.

## Create the stock table

The same migration sequence prepares the data used for the stock implementation later in the workshop.

Create the table:

```sql file:V2_0_0__create_stocks_table.sql
CREATE TABLE stock (
    id          SERIAL PRIMARY KEY,
    name        TEXT NOT NULL,
    category    TEXT NOT NULL,
    description TEXT NOT NULL
);
```

Populate it:

```sql file:V2_1_0__populate_stocks_table.sql
INSERT INTO stock (name, category, description) VALUES
    ('Analytical Engine', 'Hardware', 'Brass, steam driven, mostly imaginary'),
    ('Punch Card Reader', 'Hardware', 'Reads 80 column cards, one at a time'),
    ('COBOL Compiler', 'Software', 'Turns English looking text into machine code'),
    ('Bombe', 'Hardware', 'Electromechanical device for breaking Enigma traffic'),
    ('Slide Rule', 'Accessories', 'Batteries most definitely not included');
```

These are the complete four migration files used by the workshop code trace.

## Apply and inspect the migrations

If the local `booleandb` still contains the previous workshop schema, clean it before applying this new sequence:

```sh file:"Clean the local workshop database"
flyway -cleanDisabled=false clean
```

> [!warning] Local database only
> `clean` removes database objects. Check the URL in `flyway.toml` before running it, and use it only against the disposable local classroom database.

Apply every pending migration:

```sh file:"Apply the migrations"
flyway migrate
```

Inspect Flyway's recorded state:

```sh file:"Inspect migration history"
flyway info
```

Refresh the connection in DBeaver and verify the observable result:

- `customers` contains five rows.
- `stock` contains five rows.
- `flyway_schema_history` records versions `1.0.0`, `1.1.0`, `2.0.0` and `2.1.0` as successful.

If these results are present, the database is ready for the Java connection.

---

# Links
![[Lessons/2 - Java Back-end/Day 12/__blocks/Links]]
