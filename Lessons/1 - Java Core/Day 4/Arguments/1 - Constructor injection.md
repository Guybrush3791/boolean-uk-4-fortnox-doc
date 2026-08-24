# Constructor injection

## 1. A dependency is a collaborator

A `Computer` cannot turn on without a `PowerSupply`. The power supply is therefore a dependency of the computer.

The starter code creates that dependency inside `turnOn()`:

```java file:Computer.java hlt:5
public class Computer {
    public ArrayList<Game> installedGames = new ArrayList<>();

    public void turnOn() {
        PowerSupply psu = new PowerSupply();
        psu.turnOn();
    }
}
```

This code works in isolation, but it hides an important decision inside `Computer`: every call creates a new `PowerSupply`. The caller cannot choose a particular supply, inspect the one that was used, or share it with another object.

The problem is not that `Computer` has a dependency. A computer is supposed to depend on a power supply. The problem is that `Computer` is also taking responsibility for *creating* its collaborator.

## 2. Supply the dependency through the constructor

With **constructor injection**, the caller creates the power supply and passes it to the computer:

```java file:Computer.java hlt:5-6
public class Computer {
    public ArrayList<Game> installedGames = new ArrayList<>();
    private final PowerSupply psu;

    public Computer(PowerSupply psu) {
        this.psu = psu;
    }

    public void turnOn() {
        this.psu.turnOn();
    }
}
```

The program that assembles the objects now makes the choice explicit:

```java file:Program.java hlt:2-3
PowerSupply psu = new PowerSupply();
Computer computer = new Computer(psu);

computer.turnOn();
```

`Computer` uses the `PowerSupply`; it does not construct one. This is dependency injection because the dependency arrives from outside the class.

The dependency is still a **concrete class**. `Computer` knows it requires `PowerSupply`, and that is acceptable when this is the only meaningful kind of collaborator. Constructor injection has already improved the design: the object graph is visible at construction time and the same `PowerSupply` instance can be observed in a test.

```java file:ComputerTest.java hlt:1-6
PowerSupply myPsu = new PowerSupply();
Computer myPc = new Computer(myPsu);

myPc.turnOn();

Assertions.assertTrue(myPsu.isOn);
```

The assertion checks the exact object that was supplied to `Computer`. It would not be possible if `Computer` silently created a different power supply for itself.

## 3. Inject the game data too

The starter `Computer` also creates a hard-coded game inside `installGame()`:

```java file:Computer.java hlt:2
public void installGame() {
    Game game = new Game("Morrowind");
    this.installedGames.add(game);
}
```

This method cannot install the game its caller chose. The caller should provide the information needed to create a `Game`:

```java file:Computer.java hlt:1,6
public void installGame(String name) {
    Game game = new Game(name);
    this.installedGames.add(game);
}

computer.installGame("Final Fantasy XI");
```

This is **not** constructor dependency injection: `name` is input for one operation, and `Game` is created as part of that operation. It is still an improvement because the method no longer fixes the choice of game in its own implementation.

A caller can also provide an existing game collection when it constructs the computer:

```java file:Computer.java hlt:1-3
public Computer(PowerSupply psu, ArrayList<Game> installedGames) {
    this.psu = psu;
    this.installedGames = installedGames;
}
```

Here, the collection is injected through the constructor. It allows a computer to begin with preinstalled games without the constructor inventing that starting state.

> [!important]
> Dependency injection answers **who supplies an object?** It does not mean every value must be injected, and it does not forbid a class from creating short-lived objects while performing its own work. Use it for collaborators and state that the class needs but should not own the choice of.

## 4. The boundary of this first step

Injecting a `PowerSupply` improves object ownership and testability, but `Computer` still depends directly on the `PowerSupply` class. If the program later needs several kinds of power supply, replacing that concrete dependency with an interface can be useful. That is the next lesson.

Do not confuse the two changes:

| Change                               | What it achieves                                                              |
| ------------------------------------ | ----------------------------------------------------------------------------- |
| Pass a `PowerSupply` into `Computer` | Dependency injection: the caller supplies the collaborator                    |
| Type the dependency as an interface  | Abstraction: `Computer` depends on a behaviour rather than one implementation |

The techniques often appear together, but each solves a different problem.

---

# Links
![[Lessons/1 - Java Core/Day 4/__blocks/Links]]
