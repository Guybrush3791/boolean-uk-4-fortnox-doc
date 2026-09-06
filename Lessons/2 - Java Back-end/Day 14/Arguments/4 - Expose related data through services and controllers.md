# Expose related data through services and controllers

## Learning Objectives

- Keep each Spring Data repository behind a service.
- Return related entities through `ResponseEntity<T>` controllers.
- Verify both directions of the relationship through HTTP and PostgreSQL.

## Add the department service

Create `jpa-api/src/main/java/com/booleanuk/api/service/DepartmentService.java`:

```java file:DepartmentService.java
package com.booleanuk.api.service;

import java.util.List;
import java.util.Optional;

import org.springframework.stereotype.Service;

import com.booleanuk.api.model.Department;
import com.booleanuk.api.repo.DepartmentRepo;

@Service
public class DepartmentService {

    private final DepartmentRepo departmentRepo;

    public DepartmentService(DepartmentRepo departmentRepo) {
        this.departmentRepo = departmentRepo;
    }

    public List<Department> getAll() {
        return departmentRepo.findAll();
    }

    public Optional<Department> getOne(int id) {
        return departmentRepo.findById(id);
    }

    public Department create(Department department) {
        return departmentRepo.save(department);
    }
}
```

The service owns the persistence calls. The controller will depend on this behaviour instead of importing `DepartmentRepo` directly.

The existing `EmployeeService` remains between `EmployeeController` and `EmployeeRepo`. Because `department` is now part of an employee, include it with the existing field assignments in `update()`:

```java file:EmployeeService.java
employee.setFirstName(replacement.getFirstName());
employee.setLastname(replacement.getLastname());
employee.setLocation(replacement.getLocation());
employee.setEmail(replacement.getEmail());
employee.setDepartment(replacement.getDepartment());
```

Without the last assignment, a `PUT` request could update the simple fields but leave the old relationship unchanged.

## Add the department controller

Create `jpa-api/src/main/java/com/booleanuk/api/controller/DepartmentController.java`:

```java file:DepartmentController.java
package com.booleanuk.api.controller;

import java.util.List;
import java.util.Optional;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import com.booleanuk.api.model.Department;
import com.booleanuk.api.service.DepartmentService;

@RestController
@RequestMapping("departments")
public class DepartmentController {

    private final DepartmentService departmentService;

    public DepartmentController(DepartmentService departmentService) {
        this.departmentService = departmentService;
    }

    @GetMapping
    public ResponseEntity<List<Department>> getAllDepartments() {
        return ResponseEntity.ok(departmentService.getAll());
    }

    @GetMapping("{id}")
    public ResponseEntity<Department> getDepartmentById(@PathVariable int id) {
        Optional<Department> optDepartment = departmentService.getOne(id);

        if (optDepartment.isEmpty())
            return ResponseEntity.notFound().build();

        return ResponseEntity.ok(optDepartment.get());
    }

    @PostMapping
    public ResponseEntity<Department> createDepartment(@RequestBody Department department) {
        Department newDepartment = departmentService.create(department);
        return new ResponseEntity<>(newDepartment, HttpStatus.CREATED);
    }
}
```

This is the same MVC boundary used for employees:

```text file:"Department request route"
HTTP request
    -> DepartmentController
    -> DepartmentService
    -> DepartmentRepo
    -> Hibernate-generated SQL
    -> PostgreSQL
```

## Verify the relationship

From the repository root, start PostgreSQL:

```sh file:"Start PostgreSQL"
devbox services up
```

Keep that process running. In a second terminal, enter the Devbox shell and start the application:

```sh file:"Run the application"
devbox shell
cd jpa-api
./gradlew bootRun
```

Use Postman to create the one side first:

1. Send `POST http://localhost:4000/departments`.
2. Under **Body**, select **raw** and **JSON**.
3. Send:

```json file:"Department request body"
{
  "name": "Engineering",
  "location": "Stockholm"
}
```

Record the generated department `id`. Then create an employee that refers to it:

1. Send `POST http://localhost:4000/employees`.
2. Under **Body**, select **raw** and **JSON**.
3. Replace `1` with the department `id` and send:

```json file:"Employee relationship request body"
{
  "firstName": "Ada",
  "lastname": "Lovelace",
  "location": "London",
  "email": "ada@example.com",
  "department": {
    "id": 1
  }
}
```

Verify these observable results:

1. `GET http://localhost:4000/employees/{id}` returns `200 OK` with one nested department and no nested `employees` list.
2. `GET http://localhost:4000/departments/{id}` returns `200 OK` with an `employees` list whose entries do not repeat `department`.
3. A missing employee or department ID returns `404 Not Found`.
4. DBeaver shows the same department row and the employee row's `department_id` foreign key.
5. Restarting the application preserves both rows.

The lesson adds a relationship without changing the MVC responsibilities. Use repositories for persistence operations, services for application behaviour and controllers for HTTP responses. Completing every `PUT` and `DELETE` route for the related resources belongs to the exercise.

---

# Links
![[Lessons/2 - Java Back-end/Day 14/__blocks/Links]]
