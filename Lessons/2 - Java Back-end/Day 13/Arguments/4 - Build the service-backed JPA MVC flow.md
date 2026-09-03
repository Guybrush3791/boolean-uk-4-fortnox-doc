# Build the service-backed JPA MVC flow

## Learning Objectives

- Keep the service between the controller and repository.
- Use Spring Data repository operations through the service.
- Verify CRUD behaviour through HTTP and PostgreSQL.

## Preserve the service boundary

Spring Data reduces the repository code, but it does not remove the service layer. The controller should depend on application behaviour, not directly on a persistence interface.

Create `src/main/java/com/booleanuk/api/service/EmployeeService.java`:

```java file:EmployeeService.java
package com.booleanuk.api.service;

import com.booleanuk.api.model.Employee;
import com.booleanuk.api.repository.EmployeeRepository;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class EmployeeService {
    private final EmployeeRepository employeeRepository;

    public EmployeeService(EmployeeRepository employeeRepository) {
        this.employeeRepository = employeeRepository;
    }

    public List<Employee> getAll() {
        return this.employeeRepository.findAll();
    }

    public Optional<Employee> getById(Integer id) {
        return this.employeeRepository.findById(id);
    }

    public Employee create(Employee employee) {
        return this.employeeRepository.save(employee);
    }

    public Optional<Employee> update(Integer id, Employee replacement) {
        return this.employeeRepository.findById(id)
                .map(employee -> {
                    employee.setFirstName(replacement.getFirstName());
                    employee.setLastName(replacement.getLastName());
                    employee.setLocation(replacement.getLocation());
                    employee.setEmail(replacement.getEmail());
                    return this.employeeRepository.save(employee);
                });
    }

    public Optional<Employee> delete(Integer id) {
        Optional<Employee> employee = this.employeeRepository.findById(id);
        if (employee.isPresent()) {
            this.employeeRepository.delete(employee.get());
        }
        return employee;
    }
}
```

`@Service` lets Spring create this class as an application component. Constructor injection supplies its repository. The controller will only call these service methods.

`update()` first loads the managed entity, changes its fields and saves it. This preserves the existing identifier instead of replacing it with an identifier from the request body.

## Connect HTTP to the service

Create `src/main/java/com/booleanuk/api/controller/EmployeeController.java`:

```java file:EmployeeController.java
package com.booleanuk.api.controller;

import com.booleanuk.api.model.Employee;
import com.booleanuk.api.service.EmployeeService;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.server.ResponseStatusException;

import java.util.List;

@RestController
@RequestMapping("employees")
public class EmployeeController {
    private final EmployeeService employeeService;

    public EmployeeController(EmployeeService employeeService) {
        this.employeeService = employeeService;
    }

    @GetMapping
    public ResponseEntity<List<Employee>> getAllEmployees() {
        return ResponseEntity.ok(this.employeeService.getAll());
    }

    @GetMapping("/{id}")
    public ResponseEntity<Employee> getEmployeeById(@PathVariable Integer id) {
        Employee employee = this.employeeService.getById(id)
                .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, "Not found"));
        return ResponseEntity.ok(employee);
    }

    @PostMapping
    public ResponseEntity<Employee> createEmployee(@RequestBody Employee employee) {
        Employee created = this.employeeService.create(employee);
        return new ResponseEntity<>(created, HttpStatus.CREATED);
    }

    @PutMapping("/{id}")
    public ResponseEntity<Employee> updateEmployee(
            @PathVariable Integer id,
            @RequestBody Employee employee) {
        Employee updated = this.employeeService.update(id, employee)
                .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, "Not found"));
        return new ResponseEntity<>(updated, HttpStatus.CREATED);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Employee> deleteEmployee(@PathVariable Integer id) {
        Employee deleted = this.employeeService.delete(id)
                .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND, "Not found"));
        return ResponseEntity.ok(deleted);
    }
}
```

The controller imports `EmployeeService`, not `EmployeeRepository`. It translates HTTP requests and missing values into response codes while the service coordinates persistence.

| Request | Service call | Observable response |
| --- | --- | --- |
| `GET /employees` | `getAll()` | `200 OK` with a JSON list. |
| `GET /employees/{id}` | `getById(id)` | `200 OK`, or `404 Not Found`. |
| `POST /employees` | `create(employee)` | `201 Created` with the generated `id`. |
| `PUT /employees/{id}` | `update(id, employee)` | `201 Created`, or `404 Not Found`. |
| `DELETE /employees/{id}` | `delete(id)` | `200 OK`, or `404 Not Found`. |

## Verify the complete route

Start PostgreSQL with `devbox services up`, then run the application with `./gradlew bootRun` in another Devbox shell.

In Postman, send `POST http://localhost:4000/employees`. Under **Body**, select **raw** and **JSON**, then send:

```json file:"Employee request body"
{
  "firstName": "Ada",
  "lastName": "Lovelace",
  "location": "London",
  "email": "ada@example.com"
}
```

Verify these observable results:

1. The response is `201 Created` and contains a generated `id`.
2. `GET http://localhost:4000/employees/{id}` returns the same employee with `200 OK`.
3. DBeaver shows the same row in `booleandb.public.employees`.
4. A missing ID returns `404 Not Found`.
5. After restarting the application, the row still exists because the local configuration uses `ddl-auto: update`.

The final request path is now:

```text file:"Observed JPA route"
request body
    -> EmployeeController
    -> EmployeeService
    -> EmployeeRepository
    -> Hibernate-generated SQL
    -> PostgreSQL row
    -> Employee response body
```

For a single-table CRUD API, this is the complete Day 13 boundary. Entity relationships and foreign keys through JPA belong to Part 2.

---

# Links
![[Lessons/2 - Java Back-end/Day 13/__blocks/Links]]
