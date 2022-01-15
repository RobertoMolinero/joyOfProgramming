---
layout: post
title:  "Euler #349: Langton's Ant"
date:   2022-01-15 09:56:42 +0200
categories: java junit5 mathematics
---

Dieses Problem habe ich zufällig gefunden. Im deutschen [Wikipedia-Artikel zum Project Euler](https://de.wikipedia.org/wiki/Project_Euler) wurde dieses Problem als Beispiel für ein Problem genannt, das vordergründig durch das Implementieren einer Rechenvorschrift zu lösen ist.

Die Originalaufgabe:

```
An ant moves on a regular grid of squares that are coloured either black or white.
The ant is always oriented in one of the cardinal directions (left, right, up or down) and moves from square to adjacent square according to the following rules:
- if it is on a black square, it flips the colour of the square to white, rotates 90 degrees counterclockwise and moves forward one square.
- if it is on a white square, it flips the colour of the square to black, rotates 90 degrees clockwise and moves forward one square.

Starting with a grid that is entirely white, how many squares are black after 10^18 moves of the ant?
```

Die Übersetzung:

```
Eine Ameise bewegt sich auf einem regelmäßigen Gitter aus Quadraten, die entweder schwarz oder weiß gefärbt sind.
Die Ameise ist immer in eine der Kardinalrichtungen (links, rechts, oben oder unten) ausgerichtet und bewegt sich von Feld zu Feld nach den folgenden Regeln:
- Befindet sich die Ameise auf einem schwarzen Feld, wechselt sie die Farbe des Feldes zu weiß, dreht sich um 90 Grad gegen den Uhrzeigersinn und zieht ein Feld weiter.
- Befindet er sich auf einem weißen Feld, wechselt er die Farbe des Feldes auf schwarz, dreht sich um 90 Grad im Uhrzeigersinn und rückt ein Feld vor.

Wie viele Quadrate sind nach 10^18 Zügen der Ameise schwarz, wenn man von einem komplett weißen Gitter ausgeht?
```

Mich sprach gleich an das es sich mal wieder um einen Automaten handelt, ganz ähnlich meiner Implementierung vom 'Game of Life'. Außderdem reizte mich die Frage. Es wird nach dem Zustand nach sage und schreibe 10^18 Schritten gefragt. Also nach 1.000.000.000.000.000.000 Schritten. Wird die Ameise nach kurzer Zeit die Wege wiederholen oder wird das Ergebnis wirklich gigantisch groß? Wenn es wirklich groß wird, kann ich es überhaupt noch speichern? Und wenn sich die Wege nicht bald wiederholen, muss ich dann wirklich 10^18 Schritte durchrechnen lassen? Und wenn ja, wie lange wird das wohl dauern?

**1. Die Welt und ihre Objekte**

Wir haben also eine Ameise. Diese Ameise befindet sich an einer Position. Und um zu wissen welches Feld unsere Ameise als nächstes betritt hat sie zusätzlich noch eine Richtung.

Die Position ist eine Klasse mit einem x- und einem y-Wert die zusammen genommen die Koordinaten der Position ergeben.

```java
public record Position(int x, int y) {
}
```

Die Richtung ist ein Enum mit der Aufzählung aller Richtungen.

```java
public enum Direction {

    TOP(0, -1),
    RIGHT(1, 0),
    BOTTOM(0, 1),
    LEFT(-1, 0);

    int horizontal;

    int vertical;

    Direction(int horizontal, int vertical) {
        this.horizontal = horizontal;
        this.vertical = vertical;
    }

    public int getHorizontal() {
        return horizontal;
    }

    public int getVertical() {
        return vertical;
    }
}
```

Beide Klassen werden nun in der Klasse Ameise als Attribute eingetragen.

```java
public record Ant(Position position, Direction direction) {
}
```

Jetzt brauchen wir noch eine Darstellung der Welt. Die Welt besteht aus einem zweidimensionalen, unendlich großen Feld das in Zellen aufgeteilt ist. Jede Zelle kann entweder schwarz oder weiß sein.

Die Zelle ist ein einfaches Objekt in das ich nur die Position als Attribut aufnehme. Die Farbe lasse ich bewusst weg.

```java
public record Cell(Position position) {
}
```

In dem Objekt das meine Welt darstellt nehme ich nun die Ameise und eine Liste von Zellen auf. Da die Welt theoretisch unendlich groß ist und am Anfang alle Zellen weiß sein sollen, speichere ich in meiner Liste nur die schwarzen Zellen. Das ist der Grund warum das Attribut Farbe in der Zelle weggelassen habe. Ist eine Zelle an einer bestimmten Position nicht in meiner Liste vorhanden dann existiert diese Zelle trotzdem. Nur ist sie dann eben eine weiße Zelle.

```java
public record World(Ant ant, List<Cell> blackCells) {
}
```

Die letzte und äußerste Klasse des Modells ist der Zustand der Welt zu einem bestimmten Zeitpunkt t0. Diese Klasse nenne ich Generation.

```java
public record Generation(World world, int time) {
}
```

**2. Der Algorithmus**

Der Algorithmus besteht aus 4 einfachen Vorschriften die in jedem Schritt nacheinander behandelt werden.

```java
public class LangtonAlgorithm {

    public static Generation calculateNextGeneration(Generation generation) {

        int actualTime = generation.time();
        List<Cell> actualBlackCells = generation.world().blackCells();
        Direction actualDirection = generation.world().ant().direction();
        Position actualPosition = generation.world().ant().position();

        int newTime = 0;
        List<Cell> newBlackCells = new ArrayList<>();
        Direction newDirection = Direction.TOP;
        Position newPosition = new Position(0, 0);

        return new Generation(new World(new Ant(newPosition, newDirection), newBlackCells), newTime);
    }
}
```

Ich bekomme eine Generation übergeben und liefere am Ende das Ergebnis als ein neues Objekt vom Typ Generation zurück.

Ich habe hier die folgenden 4 Schritte vorbereitet. Zuerst habe die zu ändernden Werte ausgelesen und in Variablen 'actual...' abgespeichert. Darunter stehen jetzt 4 Zeilen in denen die neuen Werte ermittelt und in Variablen 'new...' abgespeichert werden sollen. So lange die richtigen Methoden noch nicht vorhanden sind befülle ich die Variablen einfach noch ein mal mit den Startwerten.

In der letzten Zeile baue ich mit den neuen Werten das Ergebnis vom Typ Generation zusammen und gebe es zurück.

Würde man jetzt die Simulation starten gäbe es keinen Fehler, es würde sich nur erst mal nichts bewegen.

**2.1. Das Attribut Zeit durch inkrementieren ermitteln.**

Nun kommen wir zur schrittweisen Umsetzung. Die einfachste Änderung ist sicherlich die der Zeit.

```java
public class LangtonAlgorithm {

    private static int determineNewTime(int actualTime) {
        return actualTime + 1;
    }
}
```

Die neue Zeit ergibt sich einfach aus der alten Zeit plus 1. In der aufrufenden Methode muss ich nur den Standardwert 0 durch den Aufruf dieser Methode ersetzen.

Mit einem kleinen Test kann man das überprüfen.

```java
class LangtonAlgorithmTest {

    @ParameterizedTest
    @MethodSource
    void determineNewTime(Generation generation, int expectedNewTime) {
        Generation nextGeneration = LangtonAlgorithm.calculateNextGeneration(generation);
        assertEquals(expectedNewTime, nextGeneration.time());
    }

    static Stream<Arguments> determineNewTime() {
        Position position = new Position(0, 0);
        Ant ant = new Ant(position, Direction.TOP);
        List<Cell> noCell = new ArrayList<>();

        return Stream.of(
                arguments(new Generation(new World(ant, noCell), 0), 1),
                arguments(new Generation(new World(ant, noCell), 1), 2),
                arguments(new Generation(new World(ant, noCell), 2), 3)
        );
    }
}
```

**2.2. Wechsle die Farbe des Feldes (weiß nach schwarz oder schwarz nach weiß).**

Das Färben des aktuellen Felds ist der zweite Schritt. Zur Erinnerung: Die Welt ist unendlich groß und am Anfang komplett weiß. Aus Platzgründen speichern wir also nur die schwarzen Felder.

Ist das Feld in meiner Liste vorhanden befinde ich mich auf einem schwarzen Feld. Das Umfärben realisiere ich also in dem ich das Feld aus der Liste lösche.

Befinde ich mich auf einem weißen Feld merke ich das dadurch das es in der Liste der schwarzen Felder fehlt. Um es schwarz zu färben nehme ich es einfach in die Liste auf.

Man könnte das Ganze jetzt implementieren in dem man mit 'contains()' das Vorhandensein prüft und dann entsprechend mit 'add()' und 'remove()' handeln. Es geht aber etwas kürzer. Die Methode 'remove()' gibt zurück ob etwas gefunden und gelöscht wurde. Ich rufe also direkt 'remove()' auf. Und wenn es nichts zu löschen gab füge ich das Element hinzu.

```java
public class LangtonAlgorithm {

    private static List<Cell> colorCell(Position actualPosition, List<Cell> actualBlackCells) {
        List<Cell> newBlackCells = new ArrayList<>(actualBlackCells);

        if (!newBlackCells.remove(new Cell(actualPosition)))
            newBlackCells.add(new Cell(actualPosition));

        return newBlackCells;
    }
}
```

Der Aufruf aus der Hauptmethode und eine kleine Testmethode dazu und die Sache ist erledigt.

```java
class LangtonAlgorithmTest {

    @ParameterizedTest
    @MethodSource
    void colorCell(Generation generation, int expectedBlackCells) {
        Generation nextGeneration = LangtonAlgorithm.calculateNextGeneration(generation);
        assertEquals(expectedBlackCells, nextGeneration.world().blackCells().size());
    }

    static Stream<Arguments> colorCell() {
        Position position = new Position(0, 0);
        Ant ant = new Ant(position, Direction.TOP);
        List<Cell> noCell = new ArrayList<>();
        List<Cell> oneCell = List.of(new Cell(new Position(0, 0)));

        return Stream.of(
                arguments(new Generation(new World(ant, noCell), 0), 1),
                arguments(new Generation(new World(ant, oneCell), 0), 0)
        );
    }
}
```

**2.3. Auf einem weißen Feld drehe 90 Grad nach rechts; auf einem schwarzen Feld drehe 90 Grad nach links.**

Jetzt muss sich die Maus noch drehen und ein Feld fortbewegen. Die Maus soll sich entweder mit oder gegen den Uhrzeigersinn um 90 Grad drehen. Eine Logik bei der abhängig von der übergebenen Richtung eine andere Richtung ermittelt wird.

Ich habe mich dazu entschieden die Logik direkt im Enum unterzubringen. In Java hat jedes Enum intern eine Nummerierung, beginnend bei 0. Diese Nummerierung kann mit der Methode 'ordinal()' abgefragt werden. Wenn ich jetzt das nachfolgende Enum haben will frage ich einfach nach dem Enum-Eintrag dessen 'ordinal()' um 1 höher ist als das aktuelle.

Für den Fall das ich bereits beim letzten Enum angelangt bin wende ich auf das Ergebnis das Modulo von der Gesamtlänge an. Das Ergebnis ist dann 0 bzw. wieder das erste Objekt.

```java
public enum Direction {

    public Direction turnClockwise() {
        return Direction.values()[(this.ordinal() + 1) % 4];
    }

    public Direction turnCounterclockwise() {
        return Direction.values()[(this.ordinal() + 3) % 4];
    }
}
```

Für die Gegenrichtung könnte man nun einfach minus 1 rechnen. Allerdings wird es dann etwas schwieriger beim ersten Objekt. Denn -1 modulo 4 ist in Java -1. Das finde ich persönlich seltsam. Der Taschenrechner unter Windows ist nämlich der Meinung das -1 modulo 4 gleich 3 ist.

Ich habe das Problem gelöst in dem ich einfach addiere statt zu subtrahieren. Ich verschiebe den Index einfach um plus 4 nach rechts. Das kann ich machen da ich mich nach modulo 4 ja wieder in der normalen Region befinde. Und nun ziehe ich 1 ab. Ich addiere am Ende, Anzahl der Einträge minus 1, also 3 dazu.

Eine Methode die abhängig der vorgehenden Färbung dreht sieht dann wie folgt aus. 

```java
class DirectionTest {
 
    private static Direction determineNewDirection(Direction actualDirection, boolean elementRemoved) {
        if (!elementRemoved)
            return actualDirection.turnClockwise();

        return actualDirection.turnCounterclockwise();
    }
}
```

Im dazugehörigen Test definiere ich jede Richtung und rufe von da aus eine der beiden Drehen-Operationen auf. Das zweite Argument ist die neue Richtung die ich als Ergebnis erwarte.

```java
class DirectionTest {

    @ParameterizedTest
    @MethodSource
    void turnClockwise(Direction direction, Direction nextDirection) {
        Assert.assertEquals(nextDirection, direction.turnClockwise());
    }

    static Stream<Arguments> turnClockwise() {
        return Stream.of(
                arguments(Direction.TOP, Direction.RIGHT),
                arguments(Direction.RIGHT, Direction.BOTTOM),
                arguments(Direction.BOTTOM, Direction.LEFT),
                arguments(Direction.LEFT, Direction.TOP)
        );
    }

    @ParameterizedTest
    @MethodSource
    void turnCounterclockwise(Direction direction, Direction nextDirection) {
        Assert.assertEquals(nextDirection, direction.turnCounterclockwise());
    }

    static Stream<Arguments> turnCounterclockwise() {
        return Stream.of(
                arguments(Direction.TOP, Direction.LEFT),
                arguments(Direction.LEFT, Direction.BOTTOM),
                arguments(Direction.BOTTOM, Direction.RIGHT),
                arguments(Direction.RIGHT, Direction.TOP)
        );
    }
}
```

**2.4. Gehe zum nächsten Feld in der aktuellen Blickrichtung.**

Die letzte Operation ist wieder genau so einfach wie das Inkrementieren der Zeit.

Ich addiere zu x- und y-Wert der aktuellen Position x- und y-Wert der vorher ermittelten Richtung hinzu.

```java
public class LangtonAlgorithm {

    private static Position determineNewPosition(Position actualPosition, Direction newDirection) {
        return new Position(actualPosition.x() + newDirection.getHorizontal(), actualPosition.y() + newDirection.getVertical());
    }
}
```

Auch dazu noch ein kleiner Test.

```java
class LangtonAlgorithmTest {

    @ParameterizedTest
    @MethodSource
    void determineNewPosition(Generation generation, Position expectedNewPosition) {
        Generation nextGeneration = LangtonAlgorithm.calculateNextGeneration(generation);
        assertEquals(expectedNewPosition, nextGeneration.world().ant().position());
    }

    static Stream<Arguments> determineNewPosition() {
        Position position = new Position(0, 0);
        Ant ant = new Ant(position, Direction.TOP);
        List<Cell> noCell = new ArrayList<>();
        List<Cell> oneCell = List.of(new Cell(new Position(0, 0)));

        return Stream.of(
                arguments(new Generation(new World(ant, noCell), 0), new Position(1, 0)),
                arguments(new Generation(new World(ant, oneCell), 0), new Position(-1, 0))
        );
    }
}
```

**3. Die GUI und was mit ihr so erkennen kann.**

Ich habe, ähnlich wie bei meinem 'Game of Life' wieder eine kleine GUI gebastelt.

Folge Keys sind angebunden.

Key | Price
---|---
Space | One Step
K (Kilo) | 1000 Step
C (Cycle) | 104 Step
R (Reset) | Reset to start position (Generation 0)
Esc (Escape) | Exit
+ / - | Zoom in / Zoom out

Man kann sehr schön experementieren. Entweder die Simulation schrittweise verfolgen oder mit 'C' und 'K' viele Schritte auf ein mal ablaufen lassen. Die Implementierung ist performant genug um 1000 Schritte ohne Verzögerung sofort anzuzeigen.

**4. Die Lösung.**

Jetzt können wir uns also die Auswirkungen dieses simplen Algorithmus in Ruhe anschauen. Obwohl die Vorschriften sehr einfach sind wirkt das Muster ca. 10.000 Schritte lang völlig chaotisch.

![Langton's Ameise nach 3000 Schritten](https://raw.githubusercontent.com/RobertoMolinero/joyOfProgramming/main/docs/_posts/img/langton_3000.png)

Und dann wird es mit einem Schlag geordnet. Es entsteht eine richtige Ameisenstraße die nach unten links verläuft.

![Langton's Ameise nach 12000 Schritten](https://raw.githubusercontent.com/RobertoMolinero/joyOfProgramming/main/docs/_posts/img/langton_12000.png)

Dabei entsteht ein Zyklus bei dem alle 104 Schritte die Ameise sich um 2 Felder nach links und unten bewegt. Die Anzahl der hinzukommenden schwarzen Felder beträgt jeweils 12.

![Langton's Ameise nach 12104 Schritten](https://raw.githubusercontent.com/RobertoMolinero/joyOfProgramming/main/docs/_posts/img/langton_12104.png)

Die Lösung:

Ich berechne die ersten 12000 Schritte und schreibe die Anzahl der schwarzen Felder auf. Es sind 952.

Anschliessend ziehe ich die 12.000 Felder von der Gesamtmenge 10^18 ab. Das Ergebnis teile ich durch die ermittelte Länge des Zyklus 104. Das Ergebnis ist tatsächlich eine glatte Zahl ohne Kommastellen. Dieses Teilergebnis lautet.

```
(10^18 - 12.000) / 104 = 9.615.384.615.384.500
```

Die Gesamtmenge der schwarzen Felder ergibt sich also wie folgt:

```
Felder nach 12.000 Schritten + ((10^18-12.000)/104) * 12
952 + (9.615.384.615.384.500 * 12) = 115.384.615.384.614.952
```

Ich habe mich bei Euler eingeloggt und das Ergebnis überprüft. Es stimmt.

**5. Mögliche Erweiterungen.**

Ich habe nur den Grundalgorithmus implementiert. In dem [Artikel bei Wikipedia](https://de.wikipedia.org/wiki/Ameise_(Turingmaschine)) wird eine sehr interessante Verallgemeinerung des Algorithmus beschrieben. Hier werden die Richtungen in die sich die Ameise jeweils bewegt als Zeichenfolge bestehend aus 'L' und 'R' angegeben. Und das können beliebig viele sein. So ergeben sich unendlich viele neue Muster.

Und natürlich kann man auch alle anderen Parameter der Welt ändern. Beispielsweise können die Zellen von Vierecken zu Drei-, Fünf- oder Sechsecken werden.

Und die Welt muss am Anfang ja auch nicht leer sein. Ein zufällig erzeugtes Muster ergibt auch immer wieder eine andere Lösung. 
