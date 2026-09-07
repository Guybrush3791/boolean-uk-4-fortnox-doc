# Map employees to departments

## Learning Objectives

- Map a `Department` entity and repository.
- Use `@ManyToOne` and `@JoinColumn` on the foreign-key owner.
- Observe how Hibernate turns the object relationship into table structure.

## Map the department

Create `jpa-api/src/main/java/com/booleanuk/api/model/Department.java`:

```java file:Department.java
package com.booleanuk.api.model;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Table;
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import lombok.ToString;

@Entity
@Table(name = "departments")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@ToString
public class Department {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    private String name;
    private String location;
}
```

This uses the same JPA and Lombok pattern as `Employee`. `@Entity` makes `Department` persistent, and `@Table` maps it to `departments`.

Create `jpa-api/src/main/java/com/booleanuk/api/repo/DepartmentRepo.java`:

```java file:DepartmentRepo.java
package com.booleanuk.api.repo;

import org.springframework.data.jpa.repository.JpaRepository;

import com.booleanuk.api.model.Department;

public interface DepartmentRepo extends JpaRepository<Department, Integer> {
}
```

Spring Data supplies the repository implementation. The existing `EmployeeRepo` does not change.

## Put the foreign key on the owning side

Add the relationship imports and field to `Employee.java`:

```java file:Employee.java
import jakarta.persistence.JoinColumn;
import jakarta.persistence.ManyToOne;

// existing fields

@ManyToOne
@JoinColumn(name = "department_id", nullable = false)
@ToString.Exclude
private Department department;
```

Keep the existing class-level `@ToString`. `@ToString.Exclude` stops Lombok from following this relationship when it creates the employee's text representation.

`@ManyToOne` describes the Java relationship: many `Employee` objects can refer to one `Department` object.

`@JoinColumn` describes the database link. `department_id` is the column in `employees` that refers to `departments.id`. The employee is the **owning side** because its table stores the foreign key.

`nullable = false` makes a department required for every employee created in this lesson.

At this stage the relationship has one direction:

```text file:"One-way object relationship"
Employee -> Department
```

An employee response can include its department, but a department does not yet contain its employees. That inverse direction is the next limitation to solve.

Start the application and inspect PostgreSQL in DBeaver. Hibernate should create `departments` and add `department_id` to `employees`. There is no manual table creation or database wipe in this workflow.

Use `@ManyToOne` by itself when only the many side needs to navigate to the one side. Add the inverse mapping only when the one side also needs access to its related collection.

---

# Links
![[Lessons/2 - Java Back-end/Day 14/__blocks/Links]]
