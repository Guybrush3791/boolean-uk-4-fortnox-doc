# Add the inverse relationship without recursive JSON

## Learning Objectives

- Map the inverse side with `@OneToMany` and `mappedBy`.
- Explain why a bidirectional object graph can produce recursive JSON.
- Control the nested response with `@JsonIgnoreProperties`.

## Add the inverse side

A department response still has no employee information. Add a collection to `Department.java`:

```java file:Department.java
import java.util.ArrayList;
import java.util.List;

import jakarta.persistence.OneToMany;

// existing fields

@OneToMany(mappedBy = "department")
@ToString.Exclude
private List<Employee> employees = new ArrayList<>();
```

`@OneToMany` describes the inverse direction: one department can be related to many employees.

`mappedBy = "department"` refers to the Java field named `department` in `Employee`. It tells Hibernate that `Employee.department` already owns the `department_id` foreign key. It does not create a second join column.

The collection is initialised so a new department has an empty list rather than `null`. `@ToString.Exclude` prevents Lombok from repeatedly following the same object graph.

The Java model is now bidirectional:

```text file:"Two-way object relationship"
Department -> employees[0] -> department -> employees[0] -> ...
```

## The JSON limitation

Spring uses Jackson to convert the returned entity into JSON. Without a response boundary, Jackson can follow the two directions forever:

```text file:"Recursive JSON route"
department
    -> employees
        -> department
            -> employees
                -> ...
```

The database query may have succeeded even though the HTTP response fails. The problem is no longer the foreign key. It is the conversion of a circular object graph into a tree-shaped JSON document.

> [!question] Pair investigation
> Before applying the fix, request both `/employees` and `/departments`. Read the application error, draw the repeated object route, then research a Jackson annotation that can stop each response at its repeated edge while keeping both relationship directions available.

## Stop at the repeated edge

Import `com.fasterxml.jackson.annotation.JsonIgnoreProperties` in both entities.

On `Employee.department`, ignore the department's `employees` property when the department is nested inside an employee:

```java file:Employee.java
@ManyToOne
@JoinColumn(name = "department_id", nullable = false)
@JsonIgnoreProperties("employees", allowSetters = true)
@ToString.Exclude
private Department department;
```

On `Department.employees`, ignore each employee's `department` property when serializing a response, but allow Jackson to use its setter when reading request JSON:

```java file:Department.java
@OneToMany(mappedBy = "department")
@JsonIgnoreProperties(value = "department", allowSetters = true)
@ToString.Exclude
private List<Employee> employees = new ArrayList<>();
```

`allowSetters = true` makes `department` write-only at this nested JSON edge. Jackson leaves it out of the response to stop recursion, but can still call the Lombok-generated setter when creating an `Employee` from request data. The annotation does not remove either Java relationship and it does not change the PostgreSQL foreign key.

The two useful response shapes are now finite:

```text file:"Finite relationship responses"
employee response
    -> employee fields
    -> department fields
    -> stop before department.employees

department response
    -> department fields
    -> employee fields
    -> stop before employee.department
```

`@ToString.Exclude` and `@JsonIgnoreProperties` solve different traversals. The first controls Lombok's generated `toString()`. The second controls Jackson's JSON conversion.

Use a bidirectional JPA mapping when both entities need to navigate the relationship. Then define where the JSON representation stops rather than exposing an unlimited object graph.

---

# Links
![[Lessons/2 - Java Back-end/Day 14/__blocks/Links]]
