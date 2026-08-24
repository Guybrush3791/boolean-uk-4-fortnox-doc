# Bob's Bagels - Dependency Injection

![[Repository/Day 3/Ex/1 - java tdd oop bobs bagels/assets/bagels.jpg]]

## Learning Objectives

- Use constructor injection to make a class's required collaborators explicit
- Explain the drawback of creating a dependency inside the class that uses it
- Use an interface to depend on the behaviour a class needs
- Improve the distribution and reuse of an existing object-oriented domain model

## Set up instructions

- Continue with your Bob's Bagels codebase from the previous day. Do not start a new project.
- Work from your completed **core** solution for the ten Bob's Bagels user stories.
- Open the project in IntelliJ and run its existing tests before you begin.
- Do not work on discounts, receipts, SMS, or any other extension work in this exercise.

## Exercise Requirements

- Keep the existing responsibilities in your domain model:
  - `Inventory` knows which products Bob stocks, their details, and their prices.
  - `Basket` holds items, enforces its capacity, and calculates its total.
  - `Bagel` owns its fillings and includes them in its own price.
- Preserve the behaviour already covered by the original core user stories. Your refactor must not break adding, removing, capacity checks, prices, fillings, or stock validation.
- Use the Red Green Refactor approach. Write a failing test before each behaviour change, make it pass, then improve the design without changing behaviour.
- Commit your test and source-code changes separately to show that workflow.
- Update your domain model and class diagram to show the new dependency relationships.

## User Stories

```
11.
As the code that assembles Bob's Bagels,
So that a basket does not choose its own catalogue,
I'd like to provide an inventory when I create a basket.
```

Refactor `Basket` so that its constructor receives an `Inventory`. A basket must not create an inventory for itself with `new Inventory()`.

A basket needs an inventory because it validates whether an item is stocked. Dependency injection does not remove that dependency; it makes the caller responsible for choosing and supplying it.

```
12.
As a Bob's Bagels manager,
So that every basket uses the catalogue I chose,
I'd like a basket to reject items that are not stocked by its supplied inventory.
```

Keep the existing stock-validation behaviour. A normal `Inventory` must still reject an item whose SKU Bob does not stock.

```
13.
As a developer,
So that Basket only depends on the behaviour it needs,
I'd like to describe stock checking with an interface.
```

Create an interface named `Stock` with one method:

```java
boolean hasItem(String sku);
```

Make `Inventory` implement `Stock`. Refactor `Basket` to receive and store a `Stock` rather than an `Inventory`.

`Basket` should use only `hasItem(...)` to validate an item. It should not gain responsibility for prices, product details, or creating items; those remain the responsibility of `Inventory`.

```
14.
As a developer,
So that I can test a basket without building Bob's full catalogue,
I'd like to provide a small test implementation of Stock.
```

In a test, create a small implementation of `Stock` that accepts a SKU you choose. Inject it into a basket, then prove that the basket accepts an item with that SKU.

Keep a separate test showing that Bob's real `Inventory` still rejects an unstocked SKU. The different outcomes come from the supplied `Stock` implementation, not from changing `Basket`.

## Definition of done

- `Basket` has constructor(s) that receive its required stock dependency.
- `Basket` does not create an `Inventory` internally.
- `Inventory` implements `Stock`.
- `Basket` depends on `Stock` and uses only `hasItem(...)` for stock validation.
- Existing core behaviour and tests still pass.
- A focused test uses a small `Stock` implementation instead of the full inventory.
- Your updated class diagram shows `Basket` depending on `Stock`, and `Inventory` implementing `Stock`.

## Test Output

When a test fails, read the first useful part of the stack trace. Look for the assertion message, the test method name, and the line in one of your own files where the failure began.

Run the complete test suite after each refactor. A green test suite is evidence that the dependency design changed without breaking the Bob's Bagels behaviour you built on the previous day.
