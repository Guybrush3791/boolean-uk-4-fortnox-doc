# Integration testing

## Learning Objectives

- Build a small Spring Boot API with a persistent database path
- Identify valuable integration-test cases from application behaviour
- Load the complete Spring context with `@SpringBootTest`
- Send HTTP requests through `MockMvc` and `@AutoConfigureMockMvc`
- Create prerequisite test data through the application's endpoints
- Verify that controllers, services, repositories and PostgreSQL work together

## Instructions

1. Create a fresh project for this exercise. Do not copy or continue the shop application from the lesson
2. Complete the lesson's Prerequisites setup, changing the project identity for this exercise
3. Open the generated Spring Boot project in IntelliJ
4. Start PostgreSQL using Devbox
5. Implement and manually run the API described below before adding its integration tests
6. Keep the application deliberately small. Do not add authentication, users, pagination or unrelated features

## Activity

Build a simplified **bicycle-rental API**. First implement the application, then decide how to prove that its parts work together.

A bicycle has:

- a generated ID
- a model name
- an hourly price represented by `BigDecimal`

A rental has:

- a generated ID
- one bicycle
- a positive number of hours
- a total price calculated from the bicycle's hourly price and the rental duration

The client supplies the bicycle ID and number of hours. It must **not** supply the total price. The application derives that value.

### Core

#### 1. Build the bicycle API

Create the entity, repository, service, request DTO and controller code required to provide:

- `POST /bicycles` to create a bicycle
- `GET /bicycles` to retrieve all bicycles

A bicycle creation request has this shape:

```json
{
  "model": "City Bike",
  "hourlyPrice": 6.50
}
```

Store bicycles in PostgreSQL. A successful creation returns `201 Created` and the saved bicycle, including its generated ID.

#### 2. Build the rental API

Create the entity, repository, service, request DTO and controller code required to provide:

- `POST /rentals` to create a rental
- `GET /rentals` to retrieve all rentals

A rental creation request has this shape:

```json
{
  "bicycleId": 1,
  "hours": 4
}
```

The service loads the bicycle, creates the rental and derives its total price. For a bicycle priced at `6.50` per hour and a duration of `4` hours, the returned total is `26.00`.

Reject requests that cannot produce a valid rental with `400 Bad Request`. Keep this behaviour inside the normal controller, service and repository path rather than adding a second implementation for tests.

Run the application and use HTTP requests to confirm that bicycles and rentals can be created and retrieved before starting the test suite.

#### 3. Define the integration tests

Create `RentalIntegrationTest` under `src/test/java` and configure it with:

- `@SpringBootTest`
- `@AutoConfigureMockMvc`
- an injected `MockMvc`

Implement **between five and ten integration-test cases**. Choosing the behaviours, boundaries and failure conditions that deserve confidence is part of the exercise; no test-case list is provided.

Use HTTP endpoints as the entry point to the tests. When a test needs prerequisite bicycles or other setup data, create them through the application's endpoints rather than inserting them directly with a repository. Do not mock the controllers, services, repositories or database involved in the path being tested.

The suite must use PostgreSQL and must pass when run as a whole or when any test is run independently.

## Acceptance Criteria

- The API contains only the bicycle and rental behaviour required by the activity
- Controllers delegate application work to services
- Services use Spring Data repositories for persistence
- Bicycles and rentals are stored in PostgreSQL
- Rental totals are derived by the application and are never accepted from request JSON
- `RentalIntegrationTest` uses `@SpringBootTest`, `@AutoConfigureMockMvc` and `MockMvc`
- The suite contains between five and ten independently chosen integration-test cases
- Test requests enter through HTTP endpoints
- Prerequisite data is created through the application rather than inserted directly through repositories
- The integration path does not replace its controllers, services, repositories or PostgreSQL database with mocks
- `./gradlew test` completes successfully

## Extension

After completing the core activity, add one small business rule of your choice to the bicycle-rental API. Update the integration-test suite so that the new rule is covered without exceeding ten test cases. Document the rule briefly in the project README.
