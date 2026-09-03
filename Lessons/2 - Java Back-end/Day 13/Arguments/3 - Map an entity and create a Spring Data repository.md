# Map an entity and create a Spring Data repository

## Learning Objectives

- Map an `Employee` class to the `employees` table.
- Use Lombok for the accessors and no-argument constructor required by the mapping.
- Create a Spring Data repository for routine CRUD operations.

## Map one Java class to one table

Create `src/main/java/com/booleanuk/api/model/Employee.java`:

```java file:Employee.java
package com.booleanuk.api.model;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Table;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

@Entity
@Table(name = "employees")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Column(name = "first_name")
    private String firstName;

    @Column(name = "last_name")
    private String lastName;

    @Column(name = "location")
    private String location;

    @Column(name = "email_address")
    private String email;
}
```

`@Entity` makes the class part of the JPA model. `@Table` selects the PostgreSQL table. Each `@Column` connects a Java field name to its database column name.

`@Id` marks the primary key. `GenerationType.IDENTITY` lets PostgreSQL generate that value when a row is inserted.

Lombok generates `get...()` and `set...()` methods from `@Getter` and `@Setter`. `@NoArgsConstructor` and `@AllArgsConstructor` generates the empty constructor that Hibernate and Spring's JSON mapping need when they create an object.

> [!note] We use these focused annotations rather than `@Data`
> A JPA entity has a generated identity and mutable fields, so automatically generating `equals()`, `hashCode()` and `toString()` for every field can produce behaviour we have not designed.

## Replace handwritten SQL with a repository interface

Create `src/main/java/com/booleanuk/api/repository/EmployeeRepository.java`:

```java file:EmployeeRepository.java
package com.booleanuk.api.repository;

import com.booleanuk.api.model.Employee;
import org.springframework.data.jpa.repository.JpaRepository;

public interface EmployeeRepository extends JpaRepository<Employee, Integer> {
}
```

The two generic types tell Spring Data what this repository stores:

- `Employee` is the entity type.
- `Integer` is the type of `Employee.id`.

There is no implementation class and no `@Repository` annotation to add here. Spring Data detects the interface and creates a repository object at runtime. Its inherited operations include:

| Operation | Purpose |
| --- | --- |
| `findAll()` | Return every employee. |
| `findById(id)` | Return one employee as an `Optional<Employee>`. |
| `save(employee)` | Insert a new row or update a managed row. |
| `delete(employee)` | Delete the matching row. |

`Optional<Employee>` represents either one employee or no employee. The controller will use that difference to return `200 OK` or `404 Not Found`.

Run the application again. With `ddl-auto: update`, Hibernate reads the entity and creates the missing `employees` table in `booleandb`. Refresh the database in DBeaver and verify that the table contains the mapped columns before adding the HTTP flow.

---

# Links
![[Lessons/2 - Java Back-end/Day 13/__blocks/Links]]
