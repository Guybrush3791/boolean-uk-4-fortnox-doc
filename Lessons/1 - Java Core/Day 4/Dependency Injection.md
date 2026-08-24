# Dependency Injection

Programs are made from objects that collaborate. A `Computer` needs a `PowerSupply`; a `Scrabble` game needs a way to look up letter scores. When one object needs another object to do its work, it has a **dependency**.

A class can create its own dependencies, but then it decides both *what* it needs and *which exact object* it will use. That makes the class harder to configure, reuse and test. **Dependency injection** changes that responsibility: code outside the class creates the collaborator and supplies it when the class is constructed.

![[Without Dependency Injection|600]]

Dependency injection does not remove a dependency. `Computer` still needs a power supply. It makes that requirement explicit and leaves object creation to the part of the program that assembles the objects.

![[With Dependency Injection|600]]

In the first lesson, `Computer` receives a concrete collaborator, `PowerSupply`, and can also receive a collection of `Game` objects. In the second, `Scrabble` receives an **interface** representing the behaviour it needs. The interface is not dependency injection itself: it is an abstraction that allows several scoring implementations to be supplied through the same constructor.

## Learning outcomes

By the end of this lesson, you can:

- identify a class dependency and spot when it is created in the wrong place;
- use constructor injection to give an object the concrete collaborators it requires;
- explain why an interface can be a better dependency type when several implementations are valid;
- distinguish dependency injection, abstraction, implementation and extension; and
- choose an implementation outside the class that uses it.

## LC
### Repository
https://github.com/WOWS-Inc/java-dependency-injection-live-coding.git

## Lesson

[[1 - Constructor injection|Constructor injection]]

[[2 - Depend on a behaviour|Depend on a behaviour]]

## Exercise

[[Bob's Bagels - Dependency Injection]]

---

# Links
![[Lessons/1 - Java Core/Day 4/__blocks/Links]]
