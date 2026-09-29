# Bob's Bagels - AI-assisted development

![[Repository/Day 3/Ex/1 - java tdd oop bobs bagels/assets/bagels.jpg]]

## Exercise

[[Repository/Day 3/Ex/1 - java tdd oop bobs bagels/README|Bob's Bagels exercise]]

Re-create Bob's Bagels from scratch in a new Git repository using one AI coding tool. The linked exercise defines the required behaviour. Do not clone the starter or reuse an existing solution. Your role is to direct the tool and verify the result.

## Build the project incrementally

Ask the AI to create the minimal Java project, then implement one small piece of behaviour at a time. For each change, inspect a focused failing test, ask for the smallest implementation that makes it pass, run the tests, review the diff and create a Git checkpoint.

## Do not craft code manually

The AI tool must create and change all source code, tests, build files and configuration. You may write prompts, inspect files, run commands and manage Git. When you find a problem, give the evidence to the AI and ask it to make the correction.

## Apply the updated course techniques

Build the final design using dependency injection, polymorphism and inheritance where they express meaningful relationships. Apply these techniques from the beginning rather than creating a basic solution and adding them through separate refactoring phases.

## Approve and explain the result

Treat every generated change as unreviewed code. Inspect it, verify it against the exercise and reject anything you cannot justify. Finish only when the complete test suite passes, you have approved the final code and you can explain the design and implementation without AI help.

## Possible extensions

### Persist data in PostgreSQL

Replace the in-memory storage with PostgreSQL so products, stock and completed orders survive an application restart. Keep the domain behaviour unchanged and prove the persistence path with tests.

### Add a React front end

Create an order page that displays the available bagels and fillings with their prices. A customer should be able to add or remove items, see stock or capacity messages, view the running basket total and submit the order. Show a confirmation after the saved order is returned by the back end.
