---
layout: post
title:  "TicTacToe"
date:   2021-11-30 09:24:59 +0200
categories: game java spock
---

[My Solution on GitHub](https://github.com/RobertoMolinero/kataCollectionJavaAndSpock/tree/main/src/main/java/ticTacToe)

[My Tests on GitHub](https://github.com/RobertoMolinero/kataCollectionJavaAndSpock/tree/main/src/test/groovy/ticTacToe)

Diese Kata ist gerade für Anfänger sehr interessant. Zum einen ist die Problemstellung übersichtlich und enthält nicht
zu viele Komponenten auf ein mal. Sollte man während der Implementierung doch noch ein mal etwas ändern wollen ist dafür
nicht zu viel Arbeit notwendig.

Zum anderen ergibt sich am Ende ein sicht- bzw. spielbares Ergebnis das auch zu weiteren Experimenten einlädt.

Der entscheidende Punkt besteht darin einen Plan zu finden. Das heißt man zerlegt die Gesamtaufgabe in Aufgabenteile und
ordnet diese Aufgabenteile anschliessend in einer Liste so an das man sie von oben nach unten abarbeiten kann. Das
Ergebnis jeder Teilaufgabe muss ein funktionierender, testbarer Stand sein.

Meine Vorgehensweise besteht aus 3 Teilen. Ich lege die Datenstruktur fest und schreibe eine Methode die die Daten
darstellt. Die Darstellung kann ich dann testen.

+ Die Datenstruktur und deren Darstellung

Bereits nach diesem ersten Schritt kann man in einer Extraklasse mit main-Methode das bisher geschriebene schon ein mal
starten und manuell prüfen. Das sollte aber natürlich nicht dazu verleiten auf manuelle Tests umzuschwenken. Die
Hauptarbeit des Testens sollte weiterhin bei den automatisierten Tests bleiben.

Als zweites schaffe ich die Methode zur Eingabe. Dabei behandel ich schrittweise erst die normale Eingabe und danach
alle möglichen Sonderfälle.

+ Eingabe der Züge
    + Ein korrekter Zug wird eingetragen.
    + Ein Zug mit fehlerhaften Koordinaten wird nicht eingetragen.
    + Ein Zug auf einem bereits gesetzten Feld wird nicht eingetragen.

Eine kleine Methode zum wechseln des aktiven Spielers schliesst den Eingabeteil ab.

+ Nach einem Zug wird der aktuelle Spieler gewechselt.

Als letztes folgt eine Methode mit der nach jedem Zug geprüft wird ob das Spiel entschieden worden ist. Dabei gehe ich
auch wieder jeden einzelnen Fall nacheinander ab.

+ Validierung der Züge
    + Das Spielbrett wird auf vollständige Reihen untersucht.
    + Das Spielbrett wird auf vollständige Spalten untersucht.
    + Das Spielbrett wird auf vollständige Diagonalen untersucht.
    + Ist das Spielbrett voll wird das Spiel unentschieden gewertet.

Um das Spiel nun wirklich spielen zu können werde ich versuchen zwei Oberflächen zu schaffen. Eine text- und eine
grafikbasierte.

+ Ein Spiel mit Konsole
+ Ein Spiel mit einer GUI (Swing)

Anmerkung: Bevor ich mit der Implementierung der einfachen Variante beginne möchte ich an dieser Stelle noch schnell ein Geständnis ablegen. Bei meinem ersten Versuch hatte sich ein Fehler eingeschlichen den ich bis zur abschliessenden Implementierung der Konsole nicht bemerkt habe. Ich hatte die ganze Zeit 'x' und 'y' im zweidimensionalen Array verdreht. Da ich es im Code und im Test genau gleich verdreht habe ist das nicht aufgefallen. Ich habe es natürlich gleich korrigiert, trotzdem wurmte mich diese Fehlerquelle weiterhin. Diese dämlichen primitiven Datentypen. Und deswegen habe ich eine weitere Version, ohne primitive Datentypen, neben die ursprüngliche Lösung gestellt. Der Zugriff ist ein wenig komplexer, aber ich finde den Code am Ende übersichtlicher und lesbarer. Aber urteilt gern selbst.

[My Solution (Without Primitives) on GitHub](https://github.com/RobertoMolinero/kataCollectionJavaAndSpock/tree/main/src/main/java/ticTacToeWithoutPrimitives)

[My Tests (Without Primitives) on GitHub](https://github.com/RobertoMolinero/kataCollectionJavaAndSpock/tree/main/src/test/groovy/ticTacToeWithoutPrimitives)

**1. Die Datenstruktur und deren Darstellung**

Das Spiel besteht aus einem zweidimensionalen Feld. Das kann man in Java mit verschiedenen Datentypen darstellen. Die
einfachste Möglichkeit ist ein zweidimensionales Array.

Als Datentyp habe ich Character gewählt. Mehr ist nicht notwendig. Man könnte auch String nehmen, aber das schien mir an
dieser Stelle zu viel.

```java
public class Board {

    private static final char EMPTY = 'o';

    char[][] fields;

    public Board() {
        this.fields = new char[][]{
                {EMPTY, EMPTY, EMPTY},
                {EMPTY, EMPTY, EMPTY},
                {EMPTY, EMPTY, EMPTY}
        };
    }
}
```

Neben dem Feld muss man sich noch merken welcher Spieler gerade an Reihe ist. Die einfachste Lösung wäre wahrscheinlich
ein boolean (bspw. isNextPlayerOne). Ich habe das mit einem Enum gelöst. Und bei dieser Gelegenheit habe ich die
Attribute die zum Spieler gehören als Eigenschaften in das Enum eingetragen.

```java
public enum ActivePlayer {
    PLAYER_ONE("Bud", 'x'),
    PLAYER_TWO("Terrence", '+');

    private String name;

    private char piece;

    ActivePlayer(String name, char piece) {
        this.name = name;
        this.piece = piece;
    }

    public char getPiece() {
        return piece;
    }

    public String getName() {
        return name;
    }
}
```

In der Klasse TicTacToe habe ich die beiden Klassen als Attribute eingefügt.

```java
public class TicTacToe {

    ActivePlayer activePlayer;

    Board board;

    public TicTacToe() {
        this.activePlayer = ActivePlayer.PLAYER_ONE;
        this.board = new Board();
    }
}
```

Doch wie testet man das Ganze jetzt? Meine Idee: Über die korrekte Ausgabe.

Ich lege also eine Methode an die das ganze Objekt als Zeichenkette ausgibt. Der Test dazu sieht wie folgt aus:

```groovy
def "Der initiale Zustand wird korrekt angezeigt."() {
    given:
    Game sut = new Game()

    when:
    def output = sut.getOutput()

    then:
    output == """Active Player: Bud
                    |ooo
                    |ooo
                    |ooo
                    |""".stripMargin()
}
```

Die Methode _getOutput()_ selbst lege ich in der Klasse _TicTacToe_ an.

```java
public class TicTacToe {
    public String getOutput() {
        final StringBuilder sb = new StringBuilder();
        sb.append("Active Player: ").append(activePlayer.getName()).append("\n");

        for (char[] y : board.getFields()) {
            for (char x : y) {
                sb.append(x);
            }
            sb.append("\n");
        }

        return sb.toString();
    }
}
```

Anmerkung: Ich hätte an dieser Stelle natürlich auch einfach die Methode _Object.toString()_ überschreiben können. Aber ich finde das nicht ganz korrekt. Die Methode _Object.toString()_ ist eine von Java gelieferte Methode für rein technische Zwecke. Fachliche Aufgaben wie eine bestimmte Darstellung auf einem Bildschirm möchte ich da nicht reinschreiben.

**2.1. Eingabe der Züge: Ein korrekter Zug wird eingetragen.**

Ich lege als erstes eine Methode _enterMove()_ in der Klasse _TicTacToe_ an.

```java
public class TicTacToe {
    public void enterMove(int x, int y) {
    }
}
```

Als Parameter übergebe ich die Koordinaten des anzukreuzenden Kästchen. Da sie direkt auf den Eigenschaften der
umgebenden Klasse operiert muss sie nichts zurückgeben.

Der Testfall sieht nun wie folgt aus.

```groovy
@Unroll
def "Die valide Eingabe (x=#x, y=#y) fuer Spieler #player wird korrekt eingetragen."() {
    given:
    TicTacToe sut = new TicTacToe()
    sut.activePlayer = player
    sut.board.fields = fieldsInput as char[][]

    when:
    sut.enterMove(x, y)

    then:
    sut.board.fields == fieldsOutput as char[][]

    where:
    player                  | fieldsInput                                         | x | y || fieldsOutput
    ActivePlayer.PLAYER_ONE | [['o', 'o', 'o'], ['o', 'o', 'o'], ['o', 'o', 'o']] | 0 | 0 || [['x', 'o', 'o'], ['o', 'o', 'o'], ['o', 'o', 'o']]
    ActivePlayer.PLAYER_TWO | [['x', 'o', 'o'], ['o', 'o', 'o'], ['o', 'o', 'o']] | 1 | 1 || [['x', 'o', 'o'], ['o', '+', 'o'], ['o', 'o', 'o']]
    ActivePlayer.PLAYER_ONE | [['x', 'o', 'o'], ['o', '+', 'o'], ['o', 'o', 'o']] | 0 | 2 || [['x', 'o', 'o'], ['o', '+', 'o'], ['x', 'o', 'o']]
    ActivePlayer.PLAYER_TWO | [['x', 'o', 'o'], ['o', '+', 'o'], ['x', 'o', 'o']] | 2 | 0 || [['x', 'o', '+'], ['o', '+', 'o'], ['x', 'o', 'o']]
    ActivePlayer.PLAYER_ONE | [['x', 'o', '+'], ['o', '+', 'o'], ['x', 'o', 'o']] | 2 | 2 || [['x', 'o', '+'], ['o', '+', 'o'], ['x', 'o', 'x']]
}
```

Ich konstruiere ein Spiel mit einer bestimmten Spielstellung und gebe an wer als nächstes einen Zug macht. Anschliessend
rufe ich die neue Methode auf und teste ob alle Ergebnisse passen.

In der Klasse _Board_ trage ich eine einfache Methode zum setzen des Werts ein.

```java
public class Board {
    public void setCell(ActivePlayer activePlayer, int x, int y) {
        fields[y][x] = activePlayer.getPiece();
    }
}
```

Zum Abschlusse rufe ich die Methode aus meiner Methode _enterMove()_ auf.

```java
public class TicTacToe {
    public void enterMove(int x, int y) {
        board.setCell(activePlayer, x, y);
    }
}
```

**2.2. Eingabe der Züge: Ein Zug mit fehlerhaften Koordinaten wird nicht eingetragen.**

Jetzt möchte ich sicherstellen, das ungültige Koordinatenangaben nicht berücksichtigt werden. Gerade im Zusammenspiel
mit primitiven Datentypen muss ich Vorsicht walten lassen. Denn der Zugriff auf ein Element außerhalb der eigentlichen
Länge würde zu einer Exception und zum Abbruch des Programms führen.

Diesen Fehlerfall stelle ich in einem Test nach.

```groovy
@Unroll
def "Die nicht valide Eingabe (x=#x, y=#y) fuer Spieler #player wird nicht eingetragen."() {
    given:
    TicTacToe sut = new TicTacToe()
    sut.activePlayer = player
    sut.board.fields = fields

    when:
    sut.enterMove(x, y)

    then:
    sut.board.fields == fields

    where:
    player                  | fields                                              | x  | y
    ActivePlayer.PLAYER_ONE | [['o', 'o', 'o'], ['o', '+', 'o'], ['o', 'o', 'o']] | -1 | 0
    ActivePlayer.PLAYER_TWO | [['+', 'x', 'o'], ['o', '+', 'x'], ['+', 'o', 'x']] | 0  | -1
    ActivePlayer.PLAYER_ONE | [['+', 'x', 'o'], ['+', 'o', 'x'], ['o', '+', 'x']] | -1 | -1
    ActivePlayer.PLAYER_TWO | [['o', 'o', 'o'], ['o', '+', 'o'], ['o', 'o', 'o']] | -3 | -5
    ActivePlayer.PLAYER_ONE | [['+', 'x', 'o'], ['o', '+', 'x'], ['+', 'o', 'x']] | -5 | -7
}
```

Wenn ich den Test aufrufe kommt tatsächlich genau das was ich verhindern wollte.

```text
java.lang.ArrayIndexOutOfBoundsException: Index -1 out of bounds for length 3

	at ticTacToe.entity.Board.setCell(Board.java:22)
	at ticTacToe.TicTacToe.enterMove(TicTacToe.java:34)
	at ticTacToe.TicTacToeTest.Die nicht valide Eingabe (x=#x, y=#y) für Spieler #player wird nicht eingetragen. Der naechste Spieler ist #nextPlayer(TicTacToeTest.groovy:55)
```

Zuerst schreibe ich in die Klasse _Board_ eine Methode die prüft ob ein übergebener Punkt auf dem Spielbrett existiert.

```java
public class Board {
    public boolean isFieldCoordinateValid(int x, int y) {
        return x >= 0 && x <= 2 && y >= 0 && y <= 2;
    }
}
```

Diese Methode kann ich jetzt zum prüfen nutzen. Ich rufe die Methode zum setzen nur noch auf wenn die Koordinaten
korrekt sind.

```java
public class TicTacToe {
    public void enterMove(int x, int y) {
        if (board.isFieldCoordinateValid(x, y)) {
            board.setCell(activePlayer, x, y);
        }
    }
}
```

**2.3. Eingabe der Züge: Ein Zug auf einem bereits gesetzten Feld wird nicht eingetragen.**

Das ist ein Anforderung die man beim ersten Umsetzen schnell mal übersehen kann. Ein bereits markiertes Feld darf
natürlich nicht noch mal gewählt werden.

```groovy
@Unroll
def "Das bereits besetzte Feld (x=#x, y=#y) fuer Spieler #player wird nicht eingetragen."() {
    given:
    TicTacToe sut = new TicTacToe()
    sut.activePlayer = player
    sut.board.fields = fields as char[][]

    when:
    sut.enterMove(x, y)

    then:
    sut.board.fields == fields as char[][]

    where:
    player                  | fields                                              | x | y
    ActivePlayer.PLAYER_ONE | [['o', 'o', 'o'], ['o', '+', 'o'], ['o', 'o', 'o']] | 1 | 1
    ActivePlayer.PLAYER_TWO | [['+', 'x', '+'], ['o', '+', 'x'], ['x', 'o', 'x']] | 2 | 0
    ActivePlayer.PLAYER_TWO | [['+', 'x', 'o'], ['+', 'o', 'x'], ['o', '+', 'x']] | 1 | 2
}
```

Wieder wird eine recht simple Methode in die Klasse _Board_ zum prüfen eingesetzt.

```java
public class Board {
    public boolean isCellEmpty(int x, int y) {
        return fields[y][x] == EMPTY;
    }
}
```

Danach wird die Bedingung beim setzen ergänzt.

```java
public class TicTacToe {
    public void enterMove(int x, int y) {
        if (board.isFieldCoordinateValid(x, y) && board.isCellEmpty(x, y)) {
            board.setCell(activePlayer, x, y);
        }
    }
}
```

**2.4. Eingabe der Züge: Nach einem Zug wird der aktuelle Spieler gewechselt.**

```groovy
@Unroll
def "Nach dem aktuellen Spieler #player ist als naechster Spieler #nextPlayer am Zug."() {
    given:
    TicTacToe sut = new TicTacToe()
    sut.activePlayer = player

    when:
    sut.switchActivePlayer()

    then:
    sut.activePlayer == nextPlayer

    where:
    player                  || nextPlayer
    ActivePlayer.PLAYER_ONE || ActivePlayer.PLAYER_TWO
    ActivePlayer.PLAYER_TWO || ActivePlayer.PLAYER_ONE
}
```

In die Klasse _TicTacToe_ schreibe ich ein Methode zum wechseln des aktuellen Spielers.

```java
public class TicTacToe {
    public void switchActivePlayer() {
        if (activePlayer.equals(ActivePlayer.PLAYER_ONE))
            activePlayer = ActivePlayer.PLAYER_TWO;
        else
            activePlayer = ActivePlayer.PLAYER_ONE;
    }
}
```

**3.1. Validierung der Züge: Das Spielbrett wird auf vollständige Reihen untersucht.**

Das Spiel kann gespielt werden. Nur bisher ohne Ergebnis. Man kann das Brett vollschreiben und hängt anschliessend fest.

Das Spiel soll aber an einem bestimmten Punkt von einem Zustand 'noch nicht entschieden' zu einem Zustand 'entschieden'
kommen. Um die verschiedenen Zustände abzubilden habe ich als Erstes wieder ein Enum angelegt.

```java
public enum GameState {
    PLAYER_ONE_WIN("Spieler 1 gewinnt."),
    PLAYER_TWO_WIN("Spieler 2 gewinnt."),
    UNDECIDED("Weiterkämpfen!"),
    DRAW("Draw.");

    String result;

    GameState(String result) {
        this.result = result;
    }
}
```

Anschliessend habe ich in der Klasse _TicTacToe_ eine neue Methode zum auswerten geschrieben die vorerst ohne weiteren
Test den Zustand 'UNDECIDED' zurückgibt.

```java
public class TicTacToe {
    public GameState evaluateGame() {
        return GameState.UNDECIDED;
    }
}
```

Jetzt werten wir das Brett aus. Wir beginnen mit den Reihen.

```groovy
@Unroll
def "Der aktuelle Spielstand (Row) #fields mit aktuellen Spieler #player wird korrekt bewertet."() {
    given:
    TicTacToe sut = new TicTacToe()
    sut.board.fields = fields
    sut.activePlayer = player

    when:
    def gameState = sut.evaluateGame()

    then:
    gameState == result

    where:
    player                  | fields                                              || result
    ActivePlayer.PLAYER_ONE | [['o', 'o', 'o'], ['o', 'o', 'o'], ['o', 'o', 'o']] || GameState.UNDECIDED
    ActivePlayer.PLAYER_TWO | [['o', 'x', 'o'], ['x', 'o', 'x'], ['o', 'x', 'o']] || GameState.UNDECIDED
    ActivePlayer.PLAYER_ONE | [['o', 'o', 'o'], ['x', 'x', 'x'], ['o', 'o', 'o']] || GameState.PLAYER_ONE_WIN
    ActivePlayer.PLAYER_TWO | [['o', 'o', 'o'], ['+', '+', '+'], ['o', 'o', 'o']] || GameState.PLAYER_TWO_WIN
    ActivePlayer.PLAYER_ONE | [['x', 'x', 'x'], ['o', 'o', 'o'], ['o', 'o', 'o']] || GameState.PLAYER_ONE_WIN
    ActivePlayer.PLAYER_TWO | [['o', 'o', 'o'], ['o', 'o', 'o'], ['+', '+', '+']] || GameState.PLAYER_TWO_WIN
}
```

Eine kleine Methode zum Testen der Reihe in der Klasse _Board_. In einer Schleife gehe ich die Reihen von 0 bis 2 durch
und prüfe für den aktiven Spieler.

```java
public class Board {
    private boolean isRowComplete(ActivePlayer activePlayer, int y) {
        return fields[0][y] == activePlayer.getPiece() && fields[1][y] == activePlayer.getPiece() && fields[2][y] == activePlayer.getPiece();
    }
}
```

Und der Aufruf in unserer Methode.

```java
public class TicTacToe {
    public GameState evaluateGame() {
        if (board.isRowComplete(activePlayer)) {
            if (ActivePlayer.PLAYER_ONE.equals(activePlayer)) {
                return GameState.PLAYER_ONE_WIN;
            } else {
                return GameState.PLAYER_TWO_WIN;
            }
        }
        return GameState.UNDECIDED;
    }
}
```

**3.2. Validierung der Züge: Das Spielbrett wird auf vollständige Spalten untersucht.**

Für die Spalten passiert jetzt nicht viel anders.

```groovy
@Unroll
def "Der aktuelle Spielstand (Column) #fields mit aktuellen Spieler #player wird korrekt bewertet."() {
    given:
    TicTacToe sut = new TicTacToe()
    sut.board.fields = fields
    sut.activePlayer = player

    when:
    def gameState = sut.evaluateGame()

    then:
    gameState == result

    where:
    player                  | fields                                              || result
    ActivePlayer.PLAYER_ONE | [['o', 'o', 'o'], ['o', 'o', 'o'], ['o', 'o', 'o']] || GameState.UNDECIDED
    ActivePlayer.PLAYER_ONE | [['o', '+', 'o'], ['+', 'o', '+'], ['o', '+', 'o']] || GameState.UNDECIDED
    ActivePlayer.PLAYER_ONE | [['o', 'x', 'o'], ['o', 'x', 'o'], ['o', 'x', 'o']] || GameState.PLAYER_ONE_WIN
    ActivePlayer.PLAYER_TWO | [['o', '+', 'o'], ['o', '+', 'o'], ['o', '+', 'o']] || GameState.PLAYER_TWO_WIN
    ActivePlayer.PLAYER_ONE | [['x', 'o', 'o'], ['x', 'o', 'o'], ['x', 'o', 'o']] || GameState.PLAYER_ONE_WIN
    ActivePlayer.PLAYER_TWO | [['o', 'o', '+'], ['o', 'o', '+'], ['o', 'o', '+']] || GameState.PLAYER_TWO_WIN
}
```

Die Methode ist fast identisch mit der letzten.

```java
public class Board {
    private boolean isColumnComplete(ActivePlayer activePlayer, int x) {
        return fields[x][0] == activePlayer.getPiece() && fields[x][1] == activePlayer.getPiece() && fields[x][2] == activePlayer.getPiece();
    }
}
```

Und auch die Einbindung ist nur ein kleine Ergänzung.

```java
public class TicTacToe {
    public GameState evaluateGame() {
        if (board.isRowComplete(activePlayer) || board.isColumnComplete(activePlayer)) {
            if (ActivePlayer.PLAYER_ONE.equals(activePlayer)) {
                return GameState.PLAYER_ONE_WIN;
            } else {
                return GameState.PLAYER_TWO_WIN;
            }
        }
        return GameState.UNDECIDED;
    }
}
```

Hier würde ich jetzt noch ein kleines Refactoring einstreuen. Die beiden Prüfmethoden zur Vollständigkeit von Reihen und
Spalten sind sich einfach zu ähnlich.

```java
public class Board {
    public boolean isActivePlayerWinning(ActivePlayer activePlayer) {
        for (int i = 0; i < 3; i++) {
            if (isRowComplete(activePlayer, i) || isColumnComplete(activePlayer, i))
                return true;
        }

        return isDiagonalComplete(activePlayer);
    }

    private boolean isRowComplete(ActivePlayer activePlayer, int y) {
        return fields[0][y] == activePlayer.getPiece() && fields[1][y] == activePlayer.getPiece() && fields[2][y] == activePlayer.getPiece();
    }

    private boolean isColumnComplete(ActivePlayer activePlayer, int x) {
        return fields[x][0] == activePlayer.getPiece() && fields[x][1] == activePlayer.getPiece() && fields[x][2] == activePlayer.getPiece();
    }
}
```

Ich schalte eine Methode _isActivePlayerWinning()_ dazwischen und lagere die beiden for-Schleifen in diese Methode aus.
Da die einzelnen Prüfmethoden nicht mehr direkt aufgerufen werden kann ich sie auf _private_ umstellen.

**3.3. Validierung der Züge: Das Spielbrett wird auf vollständige Diagonalen untersucht.**

Zuerst der Test.

```groovy
@Unroll
def "Der aktuelle Spielstand (Diagonal) #fields mit aktuellen Spieler #player wird korrekt bewertet."() {
    given:
    TicTacToe sut = new TicTacToe()
    sut.board.fields = fields
    sut.activePlayer = player

    when:
    def gameState = sut.evaluateGame()

    then:
    gameState == result

    where:
    player                  | fields                                              || result
    ActivePlayer.PLAYER_ONE | [['o', 'o', 'o'], ['o', 'o', 'o'], ['o', 'o', 'o']] || GameState.UNDECIDED
    ActivePlayer.PLAYER_TWO | [['x', 'o', 'x'], ['o', 'o', 'o'], ['x', 'o', 'x']] || GameState.UNDECIDED
    ActivePlayer.PLAYER_ONE | [['x', 'o', 'o'], ['o', 'x', 'o'], ['o', 'o', 'x']] || GameState.PLAYER_ONE_WIN
    ActivePlayer.PLAYER_TWO | [['o', 'o', '+'], ['o', '+', 'o'], ['+', 'o', 'o']] || GameState.PLAYER_TWO_WIN
}
```

In der Klasse _Board_ wird noch ein dritte private Prüfmethode geschrieben und in die nach außen sichtbare Methode _
isActivePlayerWinning()_ eingebaut.

```java
public class Board {
    public boolean isActivePlayerWinning(ActivePlayer activePlayer) {
        for (int i = 0; i < 3; i++) {
            if (isRowComplete(activePlayer, i) || isColumnComplete(activePlayer, i))
                return true;
        }

        return isDiagonalComplete(activePlayer);
    }

    private boolean isDiagonalComplete(ActivePlayer activePlayer) {
        return (fields[0][0] == activePlayer.getPiece() && fields[1][1] == activePlayer.getPiece() && fields[2][2] == activePlayer.getPiece())
                || (fields[2][0] == activePlayer.getPiece() && fields[1][1] == activePlayer.getPiece() && fields[0][2] == activePlayer.getPiece());
    }
}
```

**3.4. Validierung der Züge: Ist das Spielbrett voll wird das Spiel unentschieden gewertet.**

Zum Schluss kommt der Test ob das Spiel wegen vollen Brett abgebrochen werden muss.

```groovy
@Unroll
def "Der aktuelle Spielstand (Full) #board wird korrekt bewertet."() {
    given:
    TicTacToe sut = new TicTacToe()
    sut.board.fields = [['+', 'x', '+'], ['+', 'x', '+'], ['x', '+', 'x']]

    when:
    def gameState = sut.evaluateGame()

    then:
    gameState == GameState.DRAW
}
```

Die Methode zum testen ist recht simpel. Ich gehe einfach alle Felder durch und breche ab sobald ich ein leeres Feld
finde. Finde ich im gesamten Durchlauf kein einziges leeres Feld ist das Spiel als 'Unentschieden' zu werten.

```java
public class Board {
    public boolean isBoardFull() {
        for (char[] y : fields) {
            for (char x : y) {
                if (x == 'o') {
                    return false;
                }
            }
        }
        return true;
    }
}
```

Der Aufruf der Methode erfolgt nach der Prüfung der Gewinnbedingungen.

```java
public class TicTacToe {
    public GameState evaluateGame() {
        if (board.isActivePlayerWinning(activePlayer)) {
            if (ActivePlayer.PLAYER_ONE.equals(activePlayer)) {
                return GameState.PLAYER_ONE_WIN;
            } else {
                return GameState.PLAYER_TWO_WIN;
            }
        }
        if (board.isBoardFull()) return GameState.DRAW;
        return GameState.UNDECIDED;
    }
}
```

**4. Ein Spiel mit Konsole**

Diese kleine Klasse soll das Spiel über die Konsole steuerbar machen.

```java
public class Console {

    public static void main(String[] args) {
        TicTacToe t = new TicTacToe();
        t.switchActivePlayer();

        do {
            int x, y;

            t.switchActivePlayer();
            System.out.println(t.getOutput());

            Scanner input = new Scanner(System.in);
            System.out.print("x: ");
            x = input.nextInt();
            System.out.print("y: ");
            y = input.nextInt();

            t.enterMove(x, y);
        } while (t.evaluateGame() == GameState.UNDECIDED);

        System.out.println(t.getOutput());
        System.out.println(t.evaluateGame().getResult());
    }
}
```

In einer Schleife wird so lange gespielt bis das Ergebnis nicht mehr 'Unentschieden' ist.

Zuerst werden die Koordinaten entgegengenommen und eingtragen. Danach werden die Siegbedingungen geprüft und schließlich
der Spieler gewechselt.

**5. Ein Spiel mit einer GUI (Swing)**

Ich habe auch noch eine kleine GUI mit Swing gebastelt. Die könnt ihr euch direkt in meinem Repository anschauen. Von
der Programmsteuerung her ist sie nahezu identisch mit der Konsole.
