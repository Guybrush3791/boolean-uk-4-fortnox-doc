# From JDBC to JPA and Hibernate

## Learning Objectives

- Describe the repeated work required by plain JDBC.
- Distinguish JPA, Hibernate and Spring Data JPA.
- Trace a request from the controller to PostgreSQL without hand-written SQL.

## The limit of the JDBC repository

Yesterday the repository opened a JDBC connection, prepared SQL, executed it and mapped each `ResultSet` row to a Java object. That gave us direct control over the database, but every CRUD operation needed more connection, statement and mapping code.

The MVC responsibilities were still useful:

```text file:"Day 12 request route"
request
    -> controller
    -> service
    -> JDBC repository
    -> prepared SQL statement
    -> PostgreSQL
```

Today we keep those responsibilities and replace only the persistence mechanism:

```text file:"Day 13 request route"
request
    -> controller
    -> service
    -> Spring Data repository
    -> Hibernate
    -> PostgreSQL
```

The controller still handles HTTP. The service remains the boundary for application behaviour. The repository still provides data operations. What changes is how the repository communicates with PostgreSQL.

## Three connected tools

**JPA** (Jakarta Persistence API) is a specification. It defines annotations and interfaces for mapping Java objects to relational data.

**Hibernate** is the JPA implementation used by Spring Boot. It reads the mapping metadata and generates SQL for PostgreSQL.

**Spring Data JPA** creates repository implementations from Java interfaces. Extending `JpaRepository` gives us common CRUD operations such as `findAll()`, `findById()`, `save()` and `delete()`.

An **ORM** (object-relational mapper) converts between objects and relational rows. It removes much of our repeated JDBC code, but it does not remove the database or SQL. Hibernate produces and executes the SQL for us.

## The trade-off

| Plain JDBC                     | Spring Data JPA with Hibernate                           |
| ------------------------------ | -------------------------------------------------------- |
| We write each SQL statement.   | Hibernate generates routine SQL                          |
| We map each result row.        | Entity metadata controls the mapping                     |
| SQL behaviour is explicit.     | Common CRUD needs less code                              |
| More control, more repetition. | More convenience, more framework behaviour to understand |

Use plain JDBC when direct SQL control is the goal. Use Spring Data JPA when the application mostly needs routine CRUD and object mapping. In this lesson we use one entity and one table. Relationships between entities belong to Part 2.

---

# Links
![[Lessons/2 - Java Back-end/Day 13/__blocks/Links]]
