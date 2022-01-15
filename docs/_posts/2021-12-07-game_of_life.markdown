---
layout: post
title:  "Game of Life"
date:   2021-12-07 15:11:32 +0200
categories: java junit5 mathematics
---

[My Solution on GitHub](https://github.com/RobertoMolinero/kataCollectionJavaAndJUnit5/tree/main/src/main/java/gameOfLife)

[My Tests on GitHub](https://github.com/RobertoMolinero/kataCollectionJavaAndJUnit5/tree/main/src/test/java/gameOfLife)

Das Spiel des Lebens (englisch: 'Conway’s Game of Life') ist ein vom Mathematiker John Horton Conway 1970 entworfenes Spiel, basierend auf einem zweidimensionalen zellulären Automaten. Es ist eine einfache und bis heute populäre Umsetzung der Automaten-Theorie von Stanisław Marcin Ulam.

[Wikipedia](https://de.wikipedia.org/wiki/Conways_Spiel_des_Lebens)

Die Mathematik dahinter ist recht überschaubar und die Implementierung nicht sehr schwer. Trotzdem ist dieser zelluläre Automat ein sehr mächtiges Werkzeug zum Simulieren von verschiedensten wissenschaftlichen Modellen. In einem späteren Beitrag möchte ich das ein mal an Hand eines Beispiels aus diesem Buch demonstrieren.

[Pixelspiele: Modellieren und Simulieren mit zellulären Automaten (Springer-Lehrbuch)](https://www.amazon.de/Pixelspiele-Modellieren-Simulieren-zellul%C3%A4ren-Springer-Lehrbuch/dp/3642451306/ref=pd_sbs_1/262-8206082-4401203?pd_rd_w=HmBz8&pf_rd_p=840402ae-0fda-4c49-9fb0-db7f87dd6eac&pf_rd_r=MQFGVFR0QPNH0DD3YRZ8&pd_rd_r=e36c8594-38b6-49ba-9850-cd0b92faa153&pd_rd_wg=BI2ek&pd_rd_i=3642451306&psc=1)

Auch der Standardalgorithmus der hier behandelt wird ist sehr interessant. Und nach Fertigstellung lädt er immer wieder zu unendlich vielen Modifizierungen und Erweiterungen ein.

Das Spielfeld des 'Game of Life' ist zweidimensional und in Reihen und Spalten aufgeteilt. Jedes Quadrat im Gitter ist eine Zelle, die den Zustand 'tot' oder 'lebendig' haben kann.

Ein Spielfeld bildet eine Generation ab. Ausgehend von einer Generation wird nun mit Hilfe von einfachen Regeln die Folgegeneration errechnet.

```
1. Regel: Jede lebende Zelle mit 0 oder 1 lebenden Nachbarn stirbt, wie bei einer Unterbevölkerung.
2. Regel: Jede lebende Zelle mit zwei oder drei lebenden Nachbarn lebt in der nächsten Generation weiter.
3. Regel: Jede lebende Zelle mit mehr als drei lebenden Nachbarn stirbt wie bei einer Überbevölkerung.
4. Regel: Jede tote Zelle, die genau drei lebende Nachbarn hat, wird zu einer lebenden Zelle, sozusagen durch Reproduktion.
```

**1. Die Datenstruktur und deren Darstellung**

Ähnlich wie bei 'TicTacToe' würde ich hier zuerst die Datenstruktur entwerfen. Danach teste ich die Datenstruktur in dem ich ein Beispiel instanziere und ausgebe.

Unsere Welt besteht aus einer Ansammlung von Zellen. Eine Generation ist diese Welt zum Zeitpunkt t0. Unser Programm soll für eine vorgegebene Generation t die Folgegeneration t+1 berechnen.

Beginnen wir also zuerst mit der Zelle.

```java
public class Cell {

    private int x;

    private int y;

    public Cell(int x, int y) { /* ... */ }

    public int getX() { return x; }
    public int getY() { return y; }
}
```

Eine einfache Datenklasse mit den Attributen 'x' und 'y'. Dazu ein Konstruktor und get-Methoden zum Abfragen der Attribute.

```java
public class World {

    private List<Cell> livingCells;

    public World(List<Cell> livingCells) { /* ... */ }

    public List<Cell> getLivingCells() { return livingCells; }
}
```

Die Klasse 'World' enthält nun eine Liste mit den lebenden Zellen.

```java
public class Generation {

    private World world;

    private int time;

    public Generation(World world, int time) {
        this.world = world;
        this.time = time;
    }

    public World getWorld() {
        return world;
    }

    public int getTime() {
        return time;
    }
}
```

Die Klasse 'Generation' wiederum enthält die Welt zu einem bestimmten Zeitpunkt t0.

Und wie testet man das Ganze jetzt? Meine Idee: Über die korrekte Ausgabe. Ich lege also eine Methode an, die das ganze Objekt als Zeichenkette ausgibt. Der Test dazu sieht wie folgt aus:

```java
class GenerationTest {

    @Test
    void getTextRepresentation() {
        List<Cell> cells = new ArrayList<>();
        cells.add(new Cell(0, 0));
        cells.add(new Cell(1, 0));
        cells.add(new Cell(2, 0));
        cells.add(new Cell(0, 1));
        cells.add(new Cell(0, 2));
        cells.add(new Cell(2, 1));
        cells.add(new Cell(2, 2));
        cells.add(new Cell(0, 4));
        cells.add(new Cell(0, 5));
        cells.add(new Cell(2, 4));
        cells.add(new Cell(2, 5));
        cells.add(new Cell(0, 6));
        cells.add(new Cell(1, 6));
        cells.add(new Cell(2, 6));

        World world = new World(cells);
        Generation generation = new Generation(world, 0);

        assertEquals("""
                Generation: 0 [X min = 0 max = 2] [Y min = 0 max = 6]

                ■■■
                ■□■
                ■□■
                □□□
                ■□■
                ■□■
                ■■■
                """, generation.getTextRepresentation());
    }
}
```

Ich ermittle zuerst die Minimum- und Maximumwerte für 'x' und 'y'. Danach gehen ich in einer Schleife von y_min zu y_max und in einer innereren Schleife von x_min zu x_max. Ist eine Zelle mit den aktuellen Koordinaten in der Liste der lebenden Zellen vorhanden gebe ich das Zeichen für eine lebende Zelle aus. Ansonsten das für eine tote Zelle.

```java
public class Generation {

    public static final String LIVING_CELL = "\u25a0";
    public static final String DEAD_CELL = "\u25a1";
    
    public String getTextRepresentation() {

        int minX = world.getLivingCells().stream().map(Cell::getX).min(Integer::compare).orElse(0);
        int minY = world.getLivingCells().stream().map(Cell::getY).min(Integer::compare).orElse(0);
        int maxX = world.getLivingCells().stream().map(Cell::getX).max(Integer::compare).orElse(0);
        int maxY = world.getLivingCells().stream().map(Cell::getY).max(Integer::compare).orElse(0);

        StringBuilder stringBuilder = new StringBuilder();

        stringBuilder.append("Generation: " + time + " ");
        stringBuilder.append("[X min = " + minX + " max = " + maxX + "] ");
        stringBuilder.append("[Y min = " + minY + " max = " + maxY + "]");
        stringBuilder.append("\n\n");

        for (int y = minY; y <= maxY; y++) {
            for (int x = minX; x <= maxX; x++) {
                if (world.getLivingCells().contains(new Cell(x, y))) {
                    stringBuilder.append(LIVING_CELL);
                } else {
                    stringBuilder.append(DEAD_CELL);
                }
            }
            stringBuilder.append("\n");
        }

        return stringBuilder.toString();
    }
}
```

Das sieht schön aus, funktioniert so aber leider erst mal nicht. Das Ergebnis ist ein Feld mit ausschliesslich toten Zellen. Die Frage ob eine Zelle in unserer Liste vorhanden ist wird immer negativ beantwortet. Warum?

Die Frage ob ein Objekt gleich ist mit einem Objekt in einer Liste wird über die Methode 'equals()' des Objekts gelöst. Ist die Methode nicht überschrieben wird die der Oberklasse 'Object' genommen. Und diese vergleicht die Speicheradressen der Objekte, da sie keine näheren Kenntnisse über die Art des eigentlichen Objekts hat.

Wir wollen das zwei Zellen als gleich gelten wenn ihre Attribute 'x' und 'y' den gleichen Wert besitzen.

```java
public class Cell {

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Cell cell = (Cell) o;
        return x == cell.x && y == cell.y;
    }

    @Override
    public int hashCode() {
        return Objects.hash(x, y);
    }
}
```

Und nun läuft auch der Test.

Ich persönlich generiere diese Methoden immer. Ich finde es nicht sinnvoll sie selbst zu schreiben. Es geht, aber es gibt ein paar Fallstricke zu beachten. Und wenn es hier einen Fehler gibt ist der in der Praxis manchmal nur schwer finden.

**2. Die Regeln für lebende Zellen (Vorbereitung)**

Es gibt insgesamt 4 Regeln. Die ersten 3 Regeln beschäftigen sich mit den lebenden Zellen.

Gibt es zu wenig Nachbarn ist es eine Unterpopulation. Sind es zu viele Nachbarn ist es ein Überpopulation. Und dazwischen gibt es einen Korridor in dem die Zelle weiterleben kann.

Der erste Schritt ist also das Zählen der Nachbarn. Ausgehend von einer bestimmten Zellen geht man in alle Richtungen und schaut ob es da eine lebende Zelle gibt. Die primitivste Lösung wäre in einer for-Schleife die Liste der lebenden Zellen durchzugehen und dann immer für 'x' und 'y' mal 1 zu addieren bzw. zu subtrahieren. Der so entstehende Code ist aber nicht besonders schön. Deswegen habe ich mich dafür entschieden alle Richtungen in einem Enum aufzuzählen und über dieses Enum dann in einer Schleife zu iterieren.

Das Enum sieht wie folgt aus.

```java
public enum Direction {

    TOP_LEFT(-1, -1),
    TOP_CENTER(0, -1),
    TOP_RIGHT(1, -1),
    LEFT(-1, 0),
    RIGHT(1, 0),
    BOTTOM_LEFT(-1, 1),
    BOTTOM_CENTER(0, 1),
    BOTTOM_RIGHT(1, 1);

    int horizontal;

    int vertical;

    Direction(int horizontal, int vertical) {
        this.horizontal = horizontal;
        this.vertical = vertical;
    }
}
```

Und so sieht das Iterieren nun aus. Innnerhalb der Schleife wird mit jeden Eintrag des Enums eine neues Objekt vom Typ Zelle erstellt. Ist diese Zelle gleich mit einer Zelle in unserer Liste erhöhen wir den Zähler 'count' um 1.   

```java
public class World {
 
    public int countLivingNeighbors(Cell referencePoint) {
        int x = referencePoint.getX();
        int y = referencePoint.getY();

        int count = 0;

        for (Direction value : Direction.values()) {
            if (livingCells.contains(new Cell(x + value.horizontal, y + value.vertical))) {
                count += 1;
            }
        }

        return count;
    }
}
```

Mit einem kleinen Test stelle ich sicher das alles funktioniert.

```java
class WorldTest {

    @ParameterizedTest
    @MethodSource
    void countLivingNeighbors(World world, int livingNeighbors) {
        assertEquals(livingNeighbors, world.countLivingNeighbors(new Cell(1, 1)));
    }

    static Stream<Arguments> countLivingNeighbors() {
        return Stream.of(
                arguments(allDirections, 8),
                arguments(block, 3),
                arguments(circle, 4)
        );
    }
}
```

**3. Die Regeln für lebende Zellen**

Jetzt können wir die Regeln umsetzen.

Ob man jetzt auf tot oder lebendig prüft ist im Prinzip egal. Ich glaube aber das einfachste ist auf das Weiterleben zu prüfen da hier nur eine Regel existiert. Gibt es 2 oder 3 Nachbarn lebt die Zelle weiter. In diesem Fall übertragen wir diese Zelle in die nächste Generation.

Der Test sieht wie folgt aus.

```java
class GameOfLifeTest {

    @ParameterizedTest
    @MethodSource
    void willALivingCellStayAlive(World world, Cell cell, boolean isStayingAlive) {
        GameOfLife gameOfLife = new GameOfLife();
        assertEquals(isStayingAlive, gameOfLife.willALivingCellStayAlive(world, cell, world.countLivingNeighbors(cell)));
    }

    static Stream<Arguments> willALivingCellStayAlive() {
        return Stream.of(
                // underpopulation
                arguments(new World(Arrays.asList(CENTER)), CENTER, false),
                arguments(new World(Arrays.asList(CENTER, TOP_LEFT)), CENTER, false),
                arguments(new World(Arrays.asList(CENTER, BOTTOM_CENTER)), CENTER, false),
                arguments(new World(Arrays.asList(CENTER, BOTTOM_RIGHT)), CENTER, false),

                // ideal environment
                arguments(new World(Arrays.asList(CENTER, TOP_LEFT, BOTTOM_RIGHT)), CENTER, true),
                arguments(new World(Arrays.asList(CENTER, LEFT, RIGHT)), CENTER, true),
                arguments(new World(Arrays.asList(CENTER, LEFT, RIGHT)), CENTER, true),
                arguments(block, CENTER, true),
                arguments(new World(Arrays.asList(CENTER, TOP_CENTER, TOP_RIGHT, BOTTOM_CENTER)), CENTER, true),
                arguments(new World(Arrays.asList(CENTER, LEFT, TOP_RIGHT, BOTTOM_CENTER)), CENTER, true),

                // overpopulation
                arguments(full, CENTER, false)
        );
    }
}
```

Die Methode prüft ob die Zelle lebt und 2 oder 3 Nachbarn hat. Das Ergebnis 'true' oder 'false' wird direkt zurückgegeben.

```java
public class GameOfLife {
    
    public boolean willALivingCellStayAlive(World world, Cell cell, int livingNeighbors) {
        return world.getLivingCells().contains(cell) && Arrays.asList(2, 3).contains(livingNeighbors);
    }
}
```

Der zweite Teil des Ausdrucks kann auch einfach als 'livingNeighbors >= 2 && livingNeighbors <= 3' geschrieben werden. Das funktioniert genau so. Ich persönlich finde diesen Ausdruck aber ein bisschen lesbarer. Außerdem ist es schwieriger einen Fehler einzubauen. Ein '<' und ein '>' verdrehen geht in der Praxis sehr schnell. Der Methodenaufruf scheint mir sicherer.

**4. Die Regel für tote Zellen**

Die letzte Regel befasst sich mit den toten Zellen. Diese sollen zum Leben erwachen wenn sie genau 3 lebende Nachbarn haben.

```java
class GameOfLifeTest {
 
    @ParameterizedTest
    @MethodSource
    void willADeadCellBeginToLive(World world, Cell cell, boolean willBeginToLive) {
        GameOfLife gameOfLife = new GameOfLife();
        assertEquals(willBeginToLive, gameOfLife.willADeadCellBeginToLive(world, cell, world.countLivingNeighbors(cell)));
    }

    static Stream<Arguments> willADeadCellBeginToLive() {
        return Stream.of(
                // underpopulation
                arguments(new World(Arrays.asList(TOP_LEFT, BOTTOM_RIGHT)), CENTER, false),
                arguments(new World(Arrays.asList(TOP_LEFT, RIGHT)), CENTER, false),
                arguments(new World(Arrays.asList(LEFT, BOTTOM_LEFT)), CENTER, false),

                // ideal environment
                arguments(new World(Arrays.asList(TOP_LEFT, RIGHT, BOTTOM_RIGHT)), CENTER, true),
                arguments(new World(Arrays.asList(TOP_LEFT, LEFT, RIGHT)), CENTER, true),
                arguments(new World(Arrays.asList(LEFT, RIGHT, BOTTOM_LEFT)), CENTER, true),

                // overpopulation
                arguments(circle, CENTER, false),
                arguments(allDirections, CENTER, false)
        );
    }
}
```

Auch diese letzte Regel lässt sich relativ einfach umsetzen. Sie ist auch sehr ähnlich zur letzten Regel.

```java
public class GameOfLife {

    public boolean willADeadCellBeginToLive(World world, Cell cell, int livingNeighbors) {
        return !world.getLivingCells().contains(cell) && livingNeighbors == 3;
    }
}
```

**5. Die Vorbereitung für die Berechnung der nächsten Generation**

Jetzt soll eine komplette neue Generation erzeugt werden. Aber dafür ist noch mal ein kleiner Zwischenschritt notwendig.

Die Frage ist welche Zellen bei der Berechnung betrachtet werden müssen. Natürlich müssen alle lebenden Zellen betrachtet werden. Diese haben wir in einer Liste stehen.

Aber auch ein paar tote Zellen müssen betrachtet. Nämlich mindestens alle toten Zellen die 3 lebende Nachbarn haben. Ich habe das wie folgt gelöst. Ich betrachte alle Zellen die in einer beliebigen Richtung ein Nachbar einer lebenden Zelle sind.

In einer doppelten Schleife iteriere ich durch die lebenden Zellen und das bereits bestehende Enum 'Direction'. Anstatt wie vorhin zu addieren füge ich alle entdeckten Zellen, tot und lebendig, einer Liste hinzu. Um dabei doppelte Einträge zu vermeiden trage ich die Zellen in ein Set statt in eine Liste ein. Ein Set ist eine Liste die keine doppelten Einträge zulässt. Das funktioniert weil wir die Methode 'equals()' von Cell bereits überschreiben haben. Und diese Methode wird von Set genutzt um zu erkennen ob 2 Zellen gleich sind.

```java
public class World {

    public List<Cell> getAllCellsToCalculate() {
        Set<Cell> neighbors = new HashSet<>();

        for (Cell cell : livingCells) {
            for (Direction value : Direction.values()) {
                neighbors.add(new Cell(cell.getX() + value.horizontal, cell.getY() + value.vertical));
            }
        }

        return List.copyOf(neighbors);
    }
}
```

Und natürlich schreibe ich auch für diese Methode einen Test.

```java
class WorldTest {

    @ParameterizedTest
    @MethodSource
    void getAllCellsToCalculate(World world, int cellsToCalculate) {
        assertEquals(cellsToCalculate, world.getAllCellsToCalculate().size());
    }

    static Stream<Arguments> getAllCellsToCalculate() {
        return Stream.of(
                arguments(allDirections, 25),
                arguments(block, 16),
                arguments(circle, 21),
                arguments(cross, 25),
                arguments(new World(Arrays.asList(TOP_RIGHT, CENTER)), 14),
                arguments(new World(Arrays.asList(TOP_RIGHT, CENTER, BOTTOM_LEFT)), 19),
                arguments(new World(Arrays.asList(TOP_RIGHT, CENTER, BOTTOM_RIGHT)), 18)
        );
    }
}
```

**6. Die Berechnung der nächsten Generation**

Jetzt kommt der große Schritt auf den wir bis hier hin zugearbeitet haben. Die Berechnung der nächsten Generation.

Zuerst lassen wir uns alle Zellen geben die zu betrachten sind. Dann gehen wir diese Liste durch, zählen die Nachbarn und wenden unsere beiden Regeln an. Alle lebenden Zellen der nächsten Generation werden in einem neuen Objekt vom Typ 'World' gesammelt. Am Ende wird mit dieser 'World' eine neue Generation mit einer um 1 inkrementierten Zeiteinheit erzeugt und zurückgegeben. 

```java
public class GameOfLife {

    public Generation calculateNextGeneration(Generation generation) {
        List<Cell> allCellsToCalculate = generation.getWorld().getAllCellsToCalculate();

        World newWorld = new World(new ArrayList<>());

        for (Cell cell : allCellsToCalculate) {
            int livingNeighbors = generation.getWorld().countLivingNeighbors(cell);
            if (willALivingCellStayAlive(generation.getWorld(), cell, livingNeighbors)) {
                newWorld.getLivingCells().add(cell);
            }
            if (willADeadCellBeginToLive(generation.getWorld(), cell, livingNeighbors)) {
                newWorld.getLivingCells().add(cell);
            }
        }

        return new Generation(newWorld, generation.getTime() + 1);
    }
}
```

Der Test dazu kann beliebig lang werden. Testobjekte können zum Beispiel dem verlinkten Wikipedia-Artikel entnommen werden.

```java
class GameOfLifeTest {
 
    @ParameterizedTest
    @MethodSource
    void calculateNextGeneration(Generation generation, Generation nextGeneration) {
        GameOfLife gameOfLife = new GameOfLife();
        Generation result = gameOfLife.calculateNextGeneration(generation);
        assertTrue(nextGeneration.getWorld().getLivingCells().containsAll(result.getWorld().getLivingCells()));
        assertTrue(result.getWorld().getLivingCells().containsAll(nextGeneration.getWorld().getLivingCells()));
    }

    static Stream<Arguments> calculateNextGeneration() {
        return Stream.of(
                arguments(new Generation(blinker_horizontal, 0), new Generation(blinker_vertical, 1)),
                arguments(new Generation(blinker_vertical, 1), new Generation(blinker_horizontal, 2))
        );
    }
}
```

**7. Eine Explosion in der Konsole**

Für eine kleine Demo in der Konsole habe ich mir ein Muster heraussortiert das scheinbar explodiert sich am Ende komplett auflöst. Ich lasse die Berechnung so lange laufen bis eine leere Generation entsteht.

```java
public class Explosion {

    public static void main(String[] args) {
        List<Cell> cells = new ArrayList<>();
        cells.add(new Cell(0, 0));
        cells.add(new Cell(1, 0));
        cells.add(new Cell(2, 0));
        cells.add(new Cell(0, 1));
        cells.add(new Cell(0, 2));
        cells.add(new Cell(2, 1));
        cells.add(new Cell(2, 2));
        cells.add(new Cell(0, 4));
        cells.add(new Cell(0, 5));
        cells.add(new Cell(2, 4));
        cells.add(new Cell(2, 5));
        cells.add(new Cell(0, 6));
        cells.add(new Cell(1, 6));
        cells.add(new Cell(2, 6));

        World world = new World(cells);
        Generation generation = new Generation(world, 0);
        System.out.println(generation.getTextRepresentation());

        GameOfLife gameOfLife = new GameOfLife();

        do {
            Generation nextGeneration = gameOfLife.calculateNextGeneration(generation);
            System.out.println(nextGeneration.getTextRepresentation());
            generation = nextGeneration;
        } while (!generation.getWorld().isExtinct());
    }
}
```

Dafür habe ich noch eine Methode 'isExtinct()' in der Klasse 'World' ergänzt. Ist die Liste der lebenden Zellen leer so gilt diese Welt als ausgestorben.

```java
public class World {

    public boolean isExtinct() {
        return livingCells.isEmpty();
    }
}
```

Der Vollständigkeit halber mit einem kleinen Test.

```java
class WorldTest {

    @ParameterizedTest
    @MethodSource
    void isExtinct(World world, boolean isExtinct) {
        assertEquals(isExtinct, world.isExtinct());
    }

    static Stream<Arguments> isExtinct() {
        return Stream.of(
                arguments(empty, true),
                arguments(allDirections, false),
                arguments(block, false),
                arguments(circle, false),
                arguments(cross, false)
        );
    }
}
```

Für eine schönere Ausgabe habe ich für diesen Fall noch eine Kleinigkeit in der Methode 'getTextRepresentation()' ergänzt.
        
```java
public class Generation {

    public String getTextRepresentation() {

        // ...

        if (world.isExtinct()) {
            stringBuilder.append("The world is extinct!");
            return stringBuilder.toString();
        }

        for (int y = minY; y <= maxY; y++) {
            for (int x = minX; x <= maxX; x++) {
                if (world.getLivingCells().contains(new Cell(x, y))) {
                    stringBuilder.append(LIVING_CELL);
                } else {
                    stringBuilder.append(DEAD_CELL);
                }
            }
            stringBuilder.append("\n");
        }

        return stringBuilder.toString();
    }
}
```

Das Ergebnis sind alle Generationen bis zum Aussterben. Hier habe ich mal als Beispiel die 10. Generation herausgenommen.

```
Generation: 10 [X min = -3 max = 5] [Y min = -4 max = 10]

□□□■■■□□□
□□■□□□■□□
□■□□□□□■□
■□□□■□□□■
■□□□□□□□■
■□■□□□■□■
□■■□□□■■□
□□□□□□□□□
□■■□□□■■□
■□■□□□■□■
■□□□□□□□■
■□□□■□□□■
□■□□□□□■□
□□■□□□■□□
□□□■■■□□□
```

**8. Eine Explosion in der GUI (Swing)**

Ich habe einen relativ einfach Dialog angelegt. Mit der Leertaste kann man zur nächsten Generation springen. Mit Escape schliesst man den Dialog.

Die Ausgabe ist leider etwas komplizierter geworden. Aber so ist das Muster immer schön zentriert.

```java
public class Dialog extends JDialog {
    private void createUIComponents() {

        canvas = new JPanel() {
            @Override
            protected void paintComponent(Graphics g) {
                super.paintComponent(g);
                Graphics2D g2d = (Graphics2D) g;

                int centerX = (int) (this.getSize().getWidth() / 2);
                int centerY = (int) (this.getSize().getHeight() / 2);

                int minX = world.getLivingCells().stream().map(Cell::getX).min(Integer::compare).orElse(0);
                int minY = world.getLivingCells().stream().map(Cell::getY).min(Integer::compare).orElse(0);
                int maxX = world.getLivingCells().stream().map(Cell::getX).max(Integer::compare).orElse(0);
                int maxY = world.getLivingCells().stream().map(Cell::getY).max(Integer::compare).orElse(0);

                int diffX = maxX - minX;
                int diffY = maxY - minY;

                g2d.setColor(Color.BLACK);

                int size = 10;

                for (Cell cell : generation.getWorld().getLivingCells()) {
                    g2d.fillRect(centerX + (cell.getX() * size) - ((diffX * size) / 2), centerY + (cell.getY() * size) - ((diffY * size) / 2), size, size);
                }
            }
        };
    }
}
```

![Ein Screenshot der 29. Generation](https://raw.githubusercontent.com/RobertoMolinero/joyOfProgramming/main/docs/_posts/img/gui_with_explosion_generation_29.png)