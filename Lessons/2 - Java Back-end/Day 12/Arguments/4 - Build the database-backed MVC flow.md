# Build the database-backed MVC flow

## Learning Objectives

- Keep the service between the controller and repository.
- Connect existing MVC endpoints to database-backed CRUD operations.
- Verify success, validation and not-found responses through HTTP.

## Preserve the service boundary

The database changes where data is stored, but it does not remove the service. `CustomerService` receives the repository through constructor injection and gives the controller a data-source-independent API.

Create `service/CustomerService.java`:

```java file:CustomerService.java
package dev.wows.buk.JavaDB.service;

import dev.wows.buk.JavaDB.model.Customer;
import dev.wows.buk.JavaDB.repository.CustomerRepository;
import org.springframework.stereotype.Service;

import java.sql.SQLException;
import java.util.List;

@Service
public class CustomerService {
    private final CustomerRepository customerRepository;

    public CustomerService(CustomerRepository customerRepository) {
        this.customerRepository = customerRepository;
    }

    public List<Customer> getAll() throws SQLException {
        return this.customerRepository.getAll();
    }

    public Customer getById(long id) throws SQLException {
        return this.customerRepository.get(id);
    }

    public Customer create(Customer customer) throws SQLException {
        return this.customerRepository.add(customer);
    }

    public Customer update(long id, Customer customer) throws SQLException {
        return this.customerRepository.update(id, customer);
    }

    public Customer delete(long id) throws SQLException {
        return this.customerRepository.delete(id);
    }
}
```

This service currently delegates each operation. Keeping the layer in place prevents the controller from depending directly on database code and gives later business rules one clear home.

## Connect the controller to the service

Create `controller/CustomerController.java`:

```java file:CustomerController.java
package dev.wows.buk.JavaDB.controller;

import dev.wows.buk.JavaDB.model.Customer;
import dev.wows.buk.JavaDB.service.CustomerService;
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

import java.sql.SQLException;
import java.util.List;

@RestController
@RequestMapping("customers")
public class CustomerController {
    private final CustomerService customerService;

    public CustomerController(CustomerService customerService) {
        this.customerService = customerService;
    }

    @GetMapping
    public ResponseEntity<List<Customer>> getAll() throws SQLException {
        return ResponseEntity.ok(this.customerService.getAll());
    }

    @GetMapping("/{id}")
    public ResponseEntity<Customer> getOne(@PathVariable long id) throws SQLException {
        Customer customer = this.customerService.getById(id);
        if (customer == null) {
            throw new ResponseStatusException(HttpStatus.NOT_FOUND, "No customer with that id was found");
        }
        return ResponseEntity.ok(customer);
    }

    @PostMapping
    public ResponseEntity<Customer> create(@RequestBody Customer customer) throws SQLException {
        this.checkRequiredFields(customer);
        return new ResponseEntity<>(this.customerService.create(customer), HttpStatus.CREATED);
    }

    @PutMapping("/{id}")
    public ResponseEntity<Customer> update(@PathVariable long id, @RequestBody Customer customer) throws SQLException {
        this.checkRequiredFields(customer);
        Customer updated = this.customerService.update(id, customer);
        if (updated == null) {
            throw new ResponseStatusException(HttpStatus.NOT_FOUND, "No customer with that id was found");
        }
        return new ResponseEntity<>(updated, HttpStatus.CREATED);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Customer> delete(@PathVariable long id) throws SQLException {
        Customer deleted = this.customerService.delete(id);
        if (deleted == null) {
            throw new ResponseStatusException(HttpStatus.NOT_FOUND, "No customer with that id was found");
        }
        return ResponseEntity.ok(deleted);
    }

    private void checkRequiredFields(Customer customer) {
        if (customer.getName() == null || customer.getAddress() == null
                || customer.getEmail() == null || customer.getPhoneNumber() == null) {
            throw new ResponseStatusException(HttpStatus.BAD_REQUEST,
                    "Could not create or update the customer, please check all required fields are correct");
        }
    }
}
```

The HTTP layer now has the same responsibilities as the in-memory version:

| Request | Service call | Success response | Missing or invalid data |
| --- | --- | --- | --- |
| `GET /customers` | `getAll()` | `200 OK` with a list | Not applicable |
| `GET /customers/{id}` | `getById(id)` | `200 OK` with one customer | `404 Not Found` |
| `POST /customers` | `create(customer)` | `201 Created` | `400 Bad Request` |
| `PUT /customers/{id}` | `update(id, customer)` | `201 Created` | `400 Bad Request` or `404 Not Found` |
| `DELETE /customers/{id}` | `delete(id)` | `200 OK` with the deleted customer | `404 Not Found` |

> [!note] Current workshop contract
> The database migration allows `phone` to be `NULL`, but the completed controller rejects a request whose `phoneNumber` is missing. Follow the controller behaviour when testing this workshop implementation.

## Verify the complete path

Run the Spring Boot application and open Postman. Send a `GET` request to `http://localhost:4000/customers` to reach the collection of customers, or add a customer ID to the end of the URL to reach one row. The collection response should contain the five customers inserted by `V1_1_0__populate_customers_table.sql`.

To create a customer, select `POST` and use the collection URL. In **Body**, select **raw** and **JSON**, then provide values for `name`, `address`, `email` and `phoneNumber`. Sending those values takes them through the same application route:

```text file:"Observed request route"
request body
    -> CustomerController
    -> CustomerService
    -> CustomerRepository
    -> prepared SQL statement
    -> PostgreSQL row
    -> Customer response body
```

Also request an ID that does not exist and confirm that the controller returns `404 Not Found`.

## Repeat the pattern for stock

The stock migrations are already applied. Use the same package strategy to add:

- `model/Stock.java`
- `repository/StockRepository.java`
- `service/StockService.java`
- `controller/StockController.java`

The `stock` table contains `id`, `name`, `category` and `description`. Implement the same five repository operations and expose them through `/stock` and `/stock/{id}`. Keep the controller dependent on `StockService`, not directly on `StockRepository`.

The completed Code Trace uses this same controller to service to repository flow for both resources.

---

# Links
![[Lessons/2 - Java Back-end/Day 12/__blocks/Links]]
