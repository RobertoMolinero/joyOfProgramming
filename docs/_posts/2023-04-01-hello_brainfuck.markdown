---
layout: post
title:  "Hello Brainfuck!"
date:   2021-10-18 13:45:17 +0200
categories: java junit5 code languages
---

### Die Sprache 'Brainfuck'

Die Programmiersprache Brainfuck ist eine sogenannte esoterische Sprache, mit deren Hilfe man einen einfachen Automaten programmieren kann.

Der Aufbau ähnelt einer Turing-Maschine. Auf der einen Seite gibt es ein Band bestehend aus Zellen. Jede Zelle hat einen eigenen Wert. Dazu gibt es einen Zeiger der anzeigt wo aktuell ein Wert gelesen bzw. geschrieben werden kann. 

Und auf der anderen Seite gibt es ein Programm in Form einer Zeichenkette bestehend aus Zeichen des Brainfuck-Befehlssatz. Dazu gibt es einen Zeiger der angibt welcher Befehl aus der Zeichenkette der aktuell auszuführende ist.

Der Befehlssatz besteht aus lediglich 8 Anweisungen, wobei jeder Befehl aus einem einzelnen Zeichen besteht. 

| Befehl | Semantik                                                                                         |
|--------|--------------------------------------------------------------------------------------------------|
| \>     | inkrementiert den Zeiger (Band)                                                                  |
| <      | dekrementiert den Zeiger (Band)                                                                  |
| +      | inkrementiert den aktuellen Zellenwert                                                           |
| −      | dekrementiert den aktuellen Zellenwert                                                           |
| .      | Gibt den aktuellen Zellenwert als ASCII-Zeichen auf der Standardausgabe aus                      |
| ,      | Liest ein Zeichen von der Standardeingabe und speichert dessen ASCII-Wert in der aktuellen Zelle |
| [      | Springt nach vorne, hinter den passenden ']'-Befehl, wenn der aktuelle Zellenwert 0 ist          |
| ]      | Springt zurück, hinter den passenden '['-Befehl, wenn der aktuelle Zellenwert nicht 0 ist        |

Obwohl der Aufbau von Maschine und Befehlssatz sehr rudimentär ist wurde für mehrere Brainfuck-Umgebungen die Turing-Vollständigkeit bewiesen. Das heißt jeder nur erdenkliche Algorithmus kann theoretisch in Brainfuck implementiert werden.

In der Praxis ist das aber kaum umsetzbar. Das Schreiben eines Programmes in Brainfuck ist sehr komplex und fordert eine ganz eigene Art zu denken. 

Aus dem beschränkten Befehlssatz ergibt sich zwangsläufig auch eine beschränkte Aussagekraft des Geschriebenen. Die Lesbarkeit selbst einfacher Programme ist einfach nicht gegeben. Selbst das typische Einführungsbeispiel 'Hello World!' sieht bereits so aus:

```
++++++++[>++++[>++>+++>+++>+<<<<-]>+>+>->>+[<]<-]>>.>---.+++++++..+++.>>.<-.<.+++.------.--------.>>+.>++.
```

Wer hier ein 'Hello World!' erkennt kann wirklich stolz auf sich sein.

### Die Kata

Die Einfachheit des Systems und der Sprache ergibt nun eine schöne Kata, zumindest meiner Meinung nach.

Es soll ein Interpreter für ein Stück Quellcode in der Sprache Brainfuck geschrieben werden. Das Problem soll als gelöst gelten wenn das oben aufgeführte 'Hello World!'  korrekt interpretiert wird.

### 0. Die Maschine (Speicher und Speicherzeiger) und ihre Initialisierung

Die Maschine besteht aus einem Speicher und einem Zeiger der angibt, wo aktuell im Speicher gelesen bzw. geschrieben wird.

Zur Initialisierung habe ich 2 Konstruktoren angelegt. In einem kann man den Zustand des Speichers selbst angeben. Das ist gerade für die Tests sehr praktisch, man kann sich den Speicher so legen wie man ihn für einen bestimmten Test braucht und anschließend den zu testenden Befehl übergeben. 

Im anderen Konstruktor wird ein Speicher mit einer Standardgröße mit Standardwerten (0) angelegt. Der Pointer behält seinen Default-Wert 0.

```java
public class Brainfuck {

    private final byte[] memory;
    private int memoryPointer;

    private static final int DEFAULT_SIZE = 32;

    public Brainfuck() {
        memory = new byte[DEFAULT_MEMORY_SIZE];
    }

    public Brainfuck(int memoryPointer, byte[] memory) {
        if (memory.length == 0) {
            throw new IllegalArgumentException("Memory is empty.");
        }

        if (memoryPointer < 0 || memoryPointer >= memory.length) {
            throw new IllegalArgumentException("Pointer position out of range.");
        }

        this.memoryPointer = memoryPointer;
        this.memory = memory;
    }
}
```

Ich habe Tests für die Exceptions des zweiten Konstruktors geschrieben. Wenn der Speicher leer ist oder wenn der Zeiger außerhalb des angegebenen Speichers liegt soll mit einer vordefinierten Exception reagiert werden. 

```java
class BrainfuckTest {
    private static final byte[] eightBytes = new byte[]{0, 1, 2, 3, 4, 5, 6, 7};
    private static final byte[] sixteenBytes = new byte[]{0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15};
    
    @Test
    public void whenMemoryIsEmpty() {
        Exception exception = assertThrows(IllegalArgumentException.class, () -> new Brainfuck(0, new byte[]{}));
        Assertions.assertEquals("Memory is empty.", exception.getMessage());
    }

    @ParameterizedTest
    @MethodSource
    public void whenPointerIsOutOfRange(int pointer, byte[] memory) {
        Exception exception = assertThrows(IllegalArgumentException.class, () -> new Brainfuck(pointer, memory));
        Assertions.assertEquals("Pointer position out of range.", exception.getMessage());
    }

    static Stream<Arguments> whenPointerIsOutOfRange() {
        return Stream.of(
                arguments(-1, new byte[]{0}),
                arguments(1, new byte[]{0}),
                arguments(2, new byte[]{0}),
                arguments(8, eightBytes),
                arguments(15, eightBytes),
                arguments(16, sixteenBytes)
        );
    }
}
```

### 1. Die Zellenbefehle '+' und '-'

Jetzt wo die Maschine steht können die einzelnen Befehle implementiert werden.

Die Reihenfolge spielt keine Rolle, ich fange einfach mal mit den Speicheroperationen '+' und '-' an. Es sollten die einfachsten Befehle sein. In einem Array wird an einer vorgegebenen Stelle der Wert inkrementiert bzw. dekrementiert.

```java
public class Brainfuck {
    void decrementMemoryValue() {
        memory[memoryPointer]--;
    }

    void incrementMemoryValue() {
        memory[memoryPointer]++;
    }
}
```

Die Tests sind ebenfalls recht einfach. Ich stelle mit dem parametrisierten Konstruktor einen bestimmten Zustand her. Anschliessend lasse ich die Operation laufen und schaue ob der sich ergebende Zustand wie erwartet aussieht.

```java
class BrainfuckTest {
    @ParameterizedTest
    @MethodSource
    void decrementMemoryValue(int pointer, byte[] memory, byte[] expectedNewMemory) {
        // arrange
        Brainfuck sut = new Brainfuck(pointer, memory);

        // act
        sut.decrementMemoryValue();

        // assert
        Assertions.assertArrayEquals(expectedNewMemory, sut.getMemory());
    }

    static Stream<Arguments> decrementMemoryValue() {
        return Stream.of(
                arguments(0, new byte[]{6, 2, 9, -4}, new byte[]{5, 2, 9, -4}),
                arguments(1, new byte[]{7, 5, -8, 1}, new byte[]{7, 4, -8, 1}),
                arguments(2, new byte[]{0, 8, -7, 3}, new byte[]{0, 8, -8, 3}),
                arguments(3, new byte[]{-2, 6, 0, 4}, new byte[]{-2, 6, 0, 3})
        );
    }
    
    @ParameterizedTest
    @MethodSource
    void incrementMemoryValue(int pointer, byte[] memory, byte[] expectedNewMemory) {
        // arrange
        Brainfuck sut = new Brainfuck(pointer, memory);

        // act
        sut.incrementMemoryValue();

        // assert
        Assertions.assertArrayEquals(expectedNewMemory, sut.getMemory());
    }

    static Stream<Arguments> incrementMemoryValue() {
        return Stream.of(
                arguments(0, new byte[]{-8, 5, 9, 4}, new byte[]{-7, 5, 9, 4}),
                arguments(1, new byte[]{7, 6, 7, 7}, new byte[]{7, 7, 7, 7}),
                arguments(2, new byte[]{0, 1, 2, 3}, new byte[]{0, 1, 3, 3}),
                arguments(3, new byte[]{5, 6, 4, 3}, new byte[]{5, 6, 4, 4})
        );
    }
}
```

### 2. Die Zeigerbefehle '>' und '<'

Der Zeiger, welcher eine Zahl ist, wird einfach um 1 erhöht oder erniedrigt. Die einzigen Besonderheiten die beachtet werden müssen sind die Ränder des Speichers. Ich habe einen endlich großen Speicher angelegt. Verlässt der Zeiger an einem der Ränder den gültigen Bereich soll er auf der anderen Seite wieder auftauchen.

```java
public class Brainfuck {
    void decrementMemoryPointer() {
        if (memoryPointer == 0) {
            memoryPointer = memory.length - 1;
        } else {
            memoryPointer--;
        }
    }

    void incrementMemoryPointer() {
        if (memoryPointer == memory.length - 1) {
            memoryPointer = 0;
        } else {
            memoryPointer++;
        }
    }
}
```

Das ist natürlich nur eine Möglichkeit. Man könnte beim Verlassen auch eine Exception werfen und das Programm abbrechen mit dem Hinweis das die Speichergröße falsch gewählt wurde.

Oder aber man wählt einen dynamischen Speicher der theoretisch unendlich groß wird und so beliebige Zeigerpositionen erlaubt. Ich belasse es mal bei dieser einfachen Lösung. Wer mag kann hier gern weiter experimentieren.

Ein paar Tests zum überprüfen der Funktion, mit und ohne Übergang.

```java
class BrainfuckTest {
    @ParameterizedTest
    @MethodSource
    void decrementMemoryPointer(int pointer, byte[] memory, int expectedPointerPosition) {
        // arrange
        Brainfuck sut = new Brainfuck(pointer, memory);

        // act
        sut.decrementMemoryPointer();

        // assert
        Assertions.assertEquals(expectedPointerPosition, sut.getMemoryPointer());
    }

    static Stream<Arguments> decrementMemoryPointer() {
        return Stream.of(
                arguments(5, eightBytes, 4),
                arguments(3, eightBytes, 2),
                arguments(0, eightBytes, 7),
                arguments(15, sixteenBytes, 14),
                arguments(8, sixteenBytes, 7),
                arguments(0, sixteenBytes, 15)
        );
    }

    @ParameterizedTest
    @MethodSource
    void incrementMemoryPointer(int pointer, byte[] memory, int expectedPointerPosition) {
        // arrange
        Brainfuck sut = new Brainfuck(pointer, memory);

        // act
        sut.incrementMemoryPointer();

        // assert
        Assertions.assertEquals(expectedPointerPosition, sut.getMemoryPointer());
    }

    static Stream<Arguments> incrementMemoryPointer() {
        return Stream.of(
                arguments(5, eightBytes, 6),
                arguments(3, eightBytes, 4),
                arguments(0, eightBytes, 1),
                arguments(7, eightBytes, 0),
                arguments(15, sixteenBytes, 0),
                arguments(8, sixteenBytes, 9),
                arguments(0, sixteenBytes, 1)
        );
    }
}
```

### 3. Aus- und Eingabe mit '.' und ','

Die Ausgabe möchte ich nicht direkt auf der Konsole sehen. Ich sammel stattdessen alles in einem StringBuilder und gebe es am Ende vollständig aus.

```java
public class Brainfuck {

    private final StringBuilder outputBuffer = new StringBuilder();

    void output() {
        outputBuffer.append(Character.toString(memory[memoryPointer]));
    }
}
```

Die gesammelte Ausgabe kann ich nun in einem Test auswerten.

```java
class BrainfuckTest {
    @ParameterizedTest
    @MethodSource
    void output(int pointer, byte[] memory, String expectedOutput) {
        // arrange
        Brainfuck sut = new Brainfuck(pointer, memory);

        // act
        sut.output();

        // assert
        Assertions.assertEquals(sut.getOutputBuffer().toString(), expectedOutput);
    }

    static Stream<Arguments> output() {
        return Stream.of(
                arguments(0, new byte[]{87, 2, 9, -4}, "W"),
                arguments(1, new byte[]{7, 111, -8, 1}, "o"),
                arguments(2, new byte[]{0, 8, 114, 3}, "r"),
                arguments(3, new byte[]{-2, 6, 0, 108}, "l"),
                arguments(0, new byte[]{100, 6, 0, 4}, "d")
        );
    }
}
```

Für den Input nutze ich die Klasse Scanner aus Java.

```java
public class Brainfuck {

    private final static Scanner scanner = new Scanner(System.in);

    void input() {
        memory[memoryPointer] = (byte) (scanner.next().charAt(0));
    }
}
```

Hier ist es schwierig einen Test zu definieren. Im oben erwähnten 'Hello World!' kommt aber keine Eingabe vor.

Deswegen werde ich den Test erst ein mal verschieben und später manuell testen.

### 4. Die Schleifenbefehle '[' und ']'

Die Schleifenbefehle '[' und ']' sind die einzigen Befehle die etwas komplizierter in der Implementierung sind. Von der öffnenden Klammer soll zur schließenden Klammer gesprungen werden wenn an der Stelle im Speicher '0' steht. Man kann sich das wie bei einer normalen for-Schleife in Java vorstellen. Eine Laufvariable wird gezählt und sobald sie einen bestimmten Wert hat, in diesem Fall '0', wird die Schleife verlassen.

Ist der Wert '0' so läuft man die Befehlskette ab und sucht nach dem passenden ']'.

Die Schwierigkeit liegt in dem Wort 'passend'. Wenn auf dem Weg eine öffnende Klammer auftaucht dann muss man die übernächste schließende Klammer ansteuern. Ich habe das mit der Variable 'level' versucht abzubilden. Für jede gefundene öffnende Klammer erhöhe ich den Wert von 'level' und für jeden schließende erniedrige ich den Wert wieder. Ist eine schließende Klammer gefunden und der Wert von 'level' gleich 0 dann haben wir die dazugehörige Klammer entdeckt.

```java
public class Brainfuck {
    int startLoop(int commandPointer, String command) {
        int level = 0;

        if (memory[memoryPointer] == 0) {
            for (int i = commandPointer + 1; i <= command.length(); i++) {
                if (command.charAt(i) == '[') {
                    level++;
                }

                if (command.charAt(i) == ']') {
                    if (level == 0) {
                        return i;
                    } else {
                        level--;
                    }
                }
            }
        }

        return commandPointer;
    }
}
```

Mit Hilfe von ein paar Beispielprogrammen aus dem Internet prüfe ich ob die richtigen Positionen angesprungen werden. 

```java
class BrainfuckTest {
    private static final String helloWorld = "++++++++[>++++[>++>+++>+++>+<<<<-]>+>+>->>+[<]<-]>>.>---.+++++++..+++.>>.<-.<.+++.------.--------.>>+.>++.";
    private static final String loop = "[-]";
    private static final String plus = "+++>[-]<[>+<-]";
    
    @ParameterizedTest
    @MethodSource
    void startLoop(int pointer, byte[] memory, int commandPointer, String command, int expectedNewCommandPointer) {
        // arrange
        Brainfuck sut = new Brainfuck(pointer, memory);
        sut.validate(command);

        // act
        int newCommandPointer = sut.startLoop(commandPointer);

        // assert
        Assertions.assertEquals(expectedNewCommandPointer, newCommandPointer);
    }

    static Stream<Arguments> startLoop() {
        return Stream.of(
                arguments(0, new byte[]{0}, 0, loop, 2),
                arguments(1, new byte[]{0, 1, 2, 3}, 0, loop, 0),
                arguments(0, new byte[]{0}, 1, plus, 3),
                arguments(0, new byte[]{0}, 5, plus, 10),
                arguments(0, new byte[]{1}, 5, plus, 5),
                arguments(2, new byte[]{5, -7, 9, -3}, 5, plus, 5),
                arguments(3, new byte[]{0, 0, 0, 0}, 14, helloWorld, 33),
                arguments(3, new byte[]{0, 0, 0, 0}, 43, helloWorld, 45),
                arguments(1, new byte[]{0, 0}, 8, helloWorld, 48)
        );
    }
}
```

Die Methode für die schließende Klammer ist nahezu gleich. Ich laufe lediglich nach links statt nach rechts und halte Ausschau nach der passenden öffnenden Klammer.

```java
public class Brainfuck {
    int endLoop(int commandPointer, String command) {
        int level = 0;

        if (memory[memoryPointer] != 0) {
            for (int i = commandPointer - 1; i >= 0; i--) {
                if (command.charAt(i) == ']') {
                    level++;
                }

                if (command.charAt(i) == '[') {
                    if (level == 0) {
                        return i;
                    } else {
                        level--;
                    }
                }
            }
        }

        return commandPointer;
    }
}
```

Auch der Test ist recht ähnlich. Ich prüfe das Gleiche, nur in die umgedrehte Richtung.

```java
class BrainfuckTest {
    @ParameterizedTest
    @MethodSource
    void endLoop(int pointer, byte[] memory, int commandPointer, String command, int expectedNewCommandPointer) {
        // arrange
        Brainfuck sut = new Brainfuck(pointer, memory);
        sut.validate(command);

        // act
        int newCommandPointer = sut.endLoop(commandPointer);

        // assert
        Assertions.assertEquals(expectedNewCommandPointer, newCommandPointer);
    }

    static Stream<Arguments> endLoop() {
        return Stream.of(
                arguments(0, new byte[]{1}, 2, loop, 0),
                arguments(1, new byte[]{0, 0, 0, 0}, 2, loop, 2),
                arguments(0, new byte[]{1}, 3, plus, 1),
                arguments(0, new byte[]{1}, 10, plus, 5),
                arguments(0, new byte[]{0}, 10, plus, 10),
                arguments(2, new byte[]{0, 0, 0, 0}, 10, plus, 10),
                arguments(3, new byte[]{0, 1, 2, 3}, 33, helloWorld, 14),
                arguments(3, new byte[]{0, 1, 2, 3}, 45, helloWorld, 43),
                arguments(3, new byte[]{0, 1, 2, 3}, 48, helloWorld, 8)
        );
    }
}
```

### 5. All together now: Interpretieren einer Befehlskette

Jetzt wo alle Teilbefehle implementiert und getestet sind versuche ich das Ganze mal zusammenzusetzen.

```java
public class Brainfuck {
    String interpret(String command) {

        for (int i = 0; i < command.length(); i++) {
            char charAt = command.charAt(i);
            switch (charAt) {
                case '>':
                    incrementMemoryPointer();
                    break;
                case '<':
                    decrementMemoryPointer();
                    break;
                case '+':
                    incrementMemoryValue();
                    break;
                case '-':
                    decrementMemoryValue();
                    break;
                case '.':
                    output();
                    break;
                case ',':
                    input();
                    break;
                case '[':
                    i = startLoop(i);
                    break;
                case ']':
                    i = endLoop(i);
                    break;
                default:
                    break;
            }
        }

        return outputBuffer.toString();
    }
}
```

Der abschliessende Test mit 2 Versionen von 'Hello World!'. Es funktioniert.

```java
class BrainfuckTest {
    private static final String helloWorld = "++++++++[>++++[>++>+++>+++>+<<<<-]>+>+>->>+[<]<-]>>.>---.+++++++..+++.>>.<-.<.+++.------.--------.>>+.>++.";
    private static final String helloWorldRecord = "+[-->-[>>+>-----<<]<--<---]>-.>>>+.>>..+++[.>]<<<<.+++.------.<<-.>>>>+.";
    
    @ParameterizedTest
    @MethodSource
    void interpret(int pointer, byte[] memory, String command, String expectedOutput) {
        // arrange
        Brainfuck sut = new Brainfuck(pointer, memory);

        // act
        String output = sut.interpret(command);

        // assert
        Assertions.assertEquals(expectedOutput, output);
    }

    static Stream<Arguments> interpret() {
        return Stream.of(
                arguments(0, new byte[]{0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0}, helloWorld, "Hello World!\n"),
                arguments(0, new byte[]{0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0}, helloWorldRecord, "Hello, World!")
        );
    }
}
```

Es läuft. Wer einen funktionierenden Interpreter benötigt kann an dieser Stelle aufhören zu lesen. Die Lösungen im Internet sehen dieser auch recht ähnlich. Nur das die meisten Lösungen in Sprachen wie C oder C++ geschrieben sind.

Meiner Meinung nach gibt es nun aber noch an mehreren Stellen Dinge die man vereinfachen und verbessern kann. Auch die Struktur des Programmes ist maximal flach. Hier kann man Dinge vielleicht noch etwas besser zuordnen. 

### Zusatz 1. Vorprüfung des Codes mit einem Stack

Als erstes möchte ich vor der Ausführung prüfen ob das Programm überhaupt eine korrekte Klammerung hat. 

Dazu nutze ich eine Datenstruktur namens 'Stack'. Es handelt sich um einen einfachen Stapel auf den man Objekte ablegen und wieder herunternehmen kann.

Der Algorithmus läuft wie folgt. Man geht von links nach rechts durch das Programm durch. Jede öffnende Klammer wird einfach auf den Stack draufgelegt. Bei einer schließenden Klammer wird geschaut ob es etwas auf dem Stack gibt. Wenn ja, nimmt man das Objekt herunter und matched damit das Klammernpaar. Ist der Stack leer gibt es wohl zu viele schließende (oder zu wenig öffnende) Klammern.

Nach dem Durchlauf durch das komplette Programm muss der Stack leer sein. Sonst gibt es zu wenig schließende (oder zu viele öffnende) Klammern.

```java
public class Brainfuck {
    boolean validate(String command) {
        Deque<Integer> stack = new ArrayDeque<>();

        for (int i = 0; i < command.length(); i++) {
            char charAt = command.charAt(i);
            if (charAt == '[') {
                stack.push(i);
            } else if (charAt == ']') {
                Integer poll = stack.poll();
                if (poll == null) {
                    return false;
                }
            }
        }

        return stack.isEmpty();
    }
}
```

Ich habe ein wenig mit den bereits im Test vorhandenen Beispielprogrammen herumgespielt und versucht für jeden Fehlerfall etwas zu finden.

```java
class BrainfuckTest {
    private static final String broken = ">]-";
    private static final String brokenTooManyClosingBrackets = ">[-]<[>+<-]]";
    private static final String brokenTooManyOpeningBrackets = ">[-<[>+<-]";

    @ParameterizedTest
    @MethodSource
    public void validate(String command) {
        // arrange + act
        Throwable exception = Assertions.assertThrows(IllegalArgumentException.class, () -> new Program(command));

        // assert
        Assertions.assertEquals("The brackets within the statement are not balanced.", exception.getMessage());
    }

    static Stream<Arguments> validate() {
        return Stream.of(
                arguments(broken),
                arguments(brokenTooManyClosingBrackets),
                arguments(brokenTooManyOpeningBrackets)
        );
    }
}
```

### Zusatz 2. Vereinfachung der Schleifenbefehle

Und jetzt ergibt sich eine schöne Optimierung. Die oben implementierten Befehle sind alle sehr simpel, außer die für die öffnende und schließende Klammer. Hier muss jeweils durch den Befehl iteriert und gesucht werden.

Das kann man jetzt abkürzen. Wenn ich die Position beim Prüfen eh in der Hand halte kann ich sie auch abspeichern. Beim erfolgreichen Match eines Klammerpaares schreibe ich einfach einen Eintrag in eine Map.

```java
public class Brainfuck {
    private final Map<Integer, Integer> openCloseAssignment = new HashMap<>();
    private final Map<Integer, Integer> closeOpenAssignment = new HashMap<>();
    
    boolean validate(String command) {
        Deque<Integer> stack = new ArrayDeque<>();

        for (int i = 0; i < command.length(); i++) {
            char charAt = command.charAt(i);

            if (charAt == '[') {
                stack.push(i);
            } else if (charAt == ']') {
                Integer poll = stack.poll();
                if (poll == null) {
                    return false;
                }
                openCloseAssignment.put(poll, i);
                closeOpenAssignment.put(i, poll);
            }
        }

        return stack.isEmpty();
    }
}
```

Die Implementierung der beiden Klammerbefehle fällt dadurch auf wenige Zeilen zusammen.

```java
public class Brainfuck {
    int startLoop(int commandPointer) {
        if (memory[memoryPointer] == 0) {
            return openCloseAssignment.get(commandPointer);
        }
        return commandPointer;
    }

    int endLoop(int commandPointer) {
        if (memory[memoryPointer] != 0) {
            return closeOpenAssignment.get(commandPointer);
        }
        return commandPointer;
    }
}
```

Mit dem Ausführen aller Tests stelle ich sicher das nach wie vor alles funktioniert.

### Zusatz 3. Entfernen der Kommentare

Jetzt gibt es noch einen Punkt in Sachen Performance. Momentan prüfe ich jedes Zeichen vom Programm ob es Teil vom Brainfuck Befehlssatz ist.

```java
public class Brainfuck {
    String interpret(String command) {

        for (int i = 0; i < command.length(); i++) {
            char charAt = command.charAt(i);
            switch (charAt) {
                case '>':
                    incrementMemoryPointer();
                    break;
                    
                // ...die restlichen Befehle...
                
                default:
                    break;
            }
        }

        return outputBuffer.toString();
    }
}
```

Das ist so weit nicht falsch, kann aber in einem größeren Programm mit vielen Kommentaren zu einer erhöhten Laufzeit führen.

Deswegen möchte ich vor der Ausführung einfach alles herauslöschen was ich nicht brauche. Dieses Verhalten entspricht auch der Spezifikation von Brainfuck die besagt das alle Zeichen die nicht zum Befehlssatz gehören ignoriert werden.

Es gibt verschiedene Wege das Filtern zu bewerkstelligen. Ich persönlich filtere am liebsten mit Streams. Man kann sich aber auch eine Regular Expression bauen.

```java
public class Brainfuck {
    private String reduceToInstructionSet(String completeCommand) {
        List<Integer> instructionSet = List.of(
                (int) '+', (int) '-',
                (int) '.', (int) ',',
                (int) '<', (int) '>',
                (int) '[', (int) ']'
        );

        StringBuilder command = new StringBuilder();
        completeCommand
                .chars()
                .filter(instructionSet::contains)
                .forEach(c -> command.append(Character.toString(c)));

        return command.toString();
    }
}
```

Und natürlich habe ich auch das am Ende noch einmal mit einem Test abgeprüft.

```java
class ProgramTest {

    private static final String helloWorldWithComments = "++++++++++" +
            "[" +
            ">+++++++>++++++++++>+++>+<<<<-" +
            "]                       Schleife zur Vorbereitung der Textausgabe" +
            ">++.                    Ausgabe von 'H'" +
            ">+.                     Ausgabe von 'e'" +
            "+++++++.                'l'" +
            ".                       'l'" +
            "+++.                    'o'" +
            ">++.                    Leerzeichen" +
            "<<+++++++++++++++.      'W'" +
            ">.                      'o'" +
            "+++.                    'r'" +
            "------.                 'l'" +
            "--------.               'd'" +
            ">+.                     '!'" +
            ">.                      Zeilenvorschub" +
            "+++.                    Wagenrücklauf";

    private static final String helloWorldWithOutComments = "++++++++++[>+++++++>++++++++++>+++>+<<<<-]>++.>+.+++++++..+++.>++.<<+++++++++++++++.>.+++.------.--------.>+.>.+++.";

    @Test
    public void reduceToInstructionSet() {
        // arrange + act
        Program sut = new Program(helloWorldWithComments);

        // assert
        Assertions.assertEquals(helloWorldWithOutComments, sut.getCommand());
    }
}
```

### Zusatz 4. Das Command Pattern

Jetzt sind die Implementierungen sämtlicher Befehle trivial und das Programm läuft stabil und schnell. Der einzige etwas komplexere Teil ist die Methode, die alles zusammensetzt.

Die Methode 'interpret()' ist ein gigantisches switch-Statement mit sehr vielen verschiedenen Fällen.

Es gibt verschiedene Möglichkeiten um das umzubauen. Ich habe mich für das Design Pattern "Command Pattern" entschieden.

Da das ganze Programm jetzt in die Welt der Objektorientierung geholt werden soll habe ich als erstes alle Attribute die das Speicherband betreffen in eine eigene Klasse namens 'Machine' ausgelagert. Alle Attribute die das Programm betreffen habe ich in eine Klasse namens 'Program' verschoben. Die Test habe ich entsprechend in eigene Testklassen umgezogen.

Die Klasse Brainfuck stellt nun nur noch einen Interpreter dar. Dementsprechend habe ich sie umbenannt und mit den beiden neuen Klassen als Attribute ausgestattet.  

```java
public class Interpreter {
    
    private final Machine machine;
    private final Program program;    
    // ...
}
```

Und jetzt zum Command Pattern. Man erstellt ein Interface und definiert darin wie die 8 Befehle aussehen sollen.

```java
public interface Command {
    void execute(Machine machine, Program program);
}
```

Danach definiert man im Interpreter eine Map die als Schlüssel das Zeichen für den Befehl und als Wert den Befehl selbst hat.

```java
public class Interpreter {
    private static final Map<String, Command> COMMAND_MAP;

    static {
        COMMAND_MAP = Map.of(
                "<", (machine, program) -> {
                    if (machine.getMemoryPointer() == 0) {
                        machine.setMemoryPointer(machine.getMemory().length - 1);
                    } else {
                        machine.decrementMemoryPointer();
                    }
                },
                ">", (machine, program) -> {
                    if (machine.getMemoryPointer() == machine.getMemory().length - 1) {
                        machine.setMemoryPointer(0);
                    } else {
                        machine.incrementMemoryPointer();
                    }
                },
                "-", (machine, program) -> machine.decrementMemoryAt(machine.getMemoryPointer()),
                "+", (machine, program) -> machine.incrementMemoryAt(machine.getMemoryPointer()),
                ".", (machine, program) -> machine.appendToOutputBuffer(Character.toString(machine.getMemory()[machine.getMemoryPointer()])),
                ",", (machine, program) -> machine.setMemoryAt(machine.getMemoryPointer(), (byte) (scanner.next().charAt(0))),
                "[", (machine, program) -> {
                    if (machine.getMemory()[machine.getMemoryPointer()] == 0) {
                        program.setCommandPointer(program.getOpenCloseAssignment().get(program.getCommandPointer()));
                    }
                },
                "]", (machine, program) -> {
                    if (machine.getMemory()[machine.getMemoryPointer()] != 0) {
                        program.setCommandPointer(program.getCloseOpenAssignment().get(program.getCommandPointer()));
                    }
                });
    }
}
```

Und die Methode zum Ausführen eines einzelnen Befehls sieht nun wie folgt aus.

```java
public class Interpreter {
    public void executeCommand(String commandType) {
        Command command = COMMAND_MAP.get(commandType);
        command.execute(this.machine, this.program);
    }
}
```

In die Klasse InterpreterTest habe ich nun Schritt für Schritt die Tests der einzelnen Befehle umgezogen. 

Das ist alles was mir zum Thema eingefallen ist. Ich bin mit der Lösung zufrieden. 

### Und jetzt?

Und was könnte man jetzt noch machen? Es gibt viele Ideen. Hier ein paar Anregungen. 

- Befehlssatz erweitern: Der Interpreter kann nun recht einfach durch eigene Befehle erweitert werden. Man könnte zum Beispiel Zahlen (1-9) hineinbringen, welche entsprechend viele Befehle vom Typ '+' nacheinander ausführen. Das würde das 'Hello World!'-Beispiel stark kürzen. 
- Datenstruktur erweitern: Genau so wie die Sprache kann man die verwendete Datenstruktur anpassen oder sogar durch eine zusätzliche ergänzen. Man könnte bspw. einen Stack hinzufügen zum zwischenspeichern von Werten.
- Editor: Ein Editor oder eine Weboberfläche mit der man Programme schreiben und ausführen kann. Vielleicht sogar schrittweise um Programme besser analysieren zu können.
- Andere Sprachen: Es gibt natürlich noch viel mehr Sprachen die im Aufbau einfach und zur vorgestellten recht ähnlich sind. Wer sich speziell für obskure Sprachen interessiert kann sich bspw. dieses Buch ein mal ansehen:

[Strange Code: Esoteric Languages That Make Programming Fun Again](https://www.amazon.de/Strange-Code-Esoteric-Languages-Programming/dp/1718502400/ref=asc_df_1718502400/?tag=googshopde-21&linkCode=df0&hvadid=579471575035&hvpos=&hvnetw=g&hvrand=5655257395350108549&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=9042613&hvtargid=pla-1650950316430&psc=1&th=1&psc=1)

...und natürlich besteht immer noch die Möglichkeit sich eine komplett eigene Sprache ausdenken. 
