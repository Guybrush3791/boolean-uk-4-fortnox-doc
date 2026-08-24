# Depend on a behaviour

## 1. The next dependency problem

The `Scrabble` starter code creates an `Alphabet` and reads its score table:

```java file:Scrabble.java hlt:2-3
public Scrabble() {
    Alphabet alphabet = new Alphabet();
    this.letterScores = alphabet.getLetterScores();
}
```

This combines two responsibilities:

1. `Scrabble` calculates a word score
2. `Scrabble` chooses the English scoring data it will use

The scoring algorithm only needs one behaviour: **give me the score for each letter**. It does not need to know whether that information comes from English, Greek or Russian rules.

## 2. Describe the behaviour with an interface

An **interface** is a Java type that describes operations a value promises to provide. It is an abstraction: it states *what* a collaborator can do without choosing *how* it does it.

For this example, define the scoring behaviour:

```java file:Alphabet.java
public interface Alphabet {
    Map<Character, Integer> getLetterScores();
}
```

This interface does not contain the English table. It is not an implementation and it does not inject anything. It is a contract that an implementation can fulfil.

Each implementation provides the same operation, but with its own scoring data. These four-letter samples use the scores supplied with the exercise; a complete implementation contains the remaining letters too.

```java file:EnglishAlphabet.java hlt:1
public class EnglishAlphabet implements Alphabet {
    @Override
    public Map<Character, Integer> getLetterScores() {
        return Map.of(
            'a', 1,
            'b', 3,
            'c', 3,
            'd', 2
            // ...
        );
    }
}
```

```java file:RussianAlphabet.java hlt:1
public class RussianAlphabet implements Alphabet {
    @Override
    public Map<Character, Integer> getLetterScores() {
        return Map.of(
            'а', 1,
            'е', 2,
            'й', 3,
            'о', 4
            // ...
        );
    }
}
```

```java file:GreekAlphabet.java hlt:1
public class GreekAlphabet implements Alphabet {
    @Override
    public Map<Character, Integer> getLetterScores() {
        return Map.of(
            'α', 1,
            'β', 2,
            'γ', 3,
            'ζ', 4
            // ...
        );
    }
}
```

`implements Alphabet` means that each class fulfils the `Alphabet` contract. It is not a subclass extension of `Scrabble`, and `Scrabble` does not need to change merely because another implementation is added.

## 3. Inject the abstraction

`Scrabble` can now receive an `Alphabet` through its constructor:

```java file:Scrabble.java hlt:4-6
public class Scrabble {
    private final Map<Character, Integer> letterScores;

    public Scrabble(Alphabet alphabet) {
        this.letterScores = alphabet.getLetterScores();
    }

    public int score(String word) {
        int total = 0;

        for (char character : word.toCharArray()) {
            if (this.letterScores.containsKey(character)) {
                total += this.letterScores.get(character);
            }
        }

        return total;
    }
}
```

This is still **constructor dependency injection**. The difference is the type of the injected dependency:

- `Scrabble` depends on the `Alphabet` **interface**
- `EnglishAlphabet`, `RussianAlphabet` and `GreekAlphabet` are concrete **implementations** of that interface
- the program chooses one implementation and supplies it to `Scrabble`

```java file:Program.java hlt:1,4
Alphabet alphabet = new RussianAlphabet();
Scrabble scrabble = new Scrabble(alphabet);

int score = scrabble.score("дврфъ");
```

The `score` method is unchanged. It still adds the score of each character it recognises. Only the source of those scores has changed.

## 4. Separate the four ideas

These words describe related but different parts of the design:

| Term                     | In this example                                                                        |
| ------------------------ | -------------------------------------------------------------------------------------- |
| **Dependency**           | `Scrabble` needs an object that can provide letter scores                              |
| **Dependency injection** | The caller passes that object to `new Scrabble(alphabet)`                              |
| **Abstraction**          | The `Alphabet` interface describes the required `getLetterScores()` behaviour          |
| **Implementation**       | `EnglishAlphabet`, `RussianAlphabet` and `GreekAlphabet` provide the actual score maps |
| **Extension**            | Adding `GreekAlphabet` adds a new supported rule set without editing `Scrabble`        |

The interface gives `Scrabble` a stable dependency type. Dependency injection gives the program a place to choose the implementation.

> [!note] Adding a new implementation is extension of the available behaviour, it is not a change to the injection technique

## 5. Why this is useful in tests

Interfaces also make it possible to supply a small test implementation that contains only the scores needed by one test:

```java file:ScrabbleTest.java
Alphabet tinyAlphabet = () -> Map.of('a', 1, 'b', 3);
Scrabble scrabble = new Scrabble(tinyAlphabet);

Assertions.assertEquals(4, scrabble.score("ab"));
```

The lambda works because `Alphabet` has one abstract method. The test is not using a special version of dependency injection. It is using the same constructor and choosing a deliberately small implementation. If you want fixed declaration you can annotate the `interface` as `@FunctionalInterface`.

> [!important]
> Do not create an interface simply because a class has one consumer today. Start with constructor injection when the caller should supply a collaborator. Introduce an interface when the consumer should depend on a behaviour and more than one implementation, configuration or test substitute is genuinely useful.

---

# Links
![[Lessons/1 - Java Core/Day 4/__blocks/Links]]
