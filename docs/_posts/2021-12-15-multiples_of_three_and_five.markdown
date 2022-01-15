---
layout: post
title:  "Euler #001: Multiples of 3 or 5"
date:   2021-12-15 09:24:59 +0200
categories: java junit5 mathematics
---

In diesem Post möchte ich damit beginnen einige ausgewählte Probleme der Seite [Project Euler](https://de.wikipedia.org/wiki/Project_Euler) vorzustellen.

Es handelt sich bei dieser Seite um eine Sammlung von mathematischen Rätseln die ich jeweils mit einem Programm lösen möchte.

Es gibt aktuell (Stand: 30.12.2021) über 750 beschriebene Probleme. Der Schwierigkeitsgrad reicht von einfacher Schulmathematik bis hin zu Aufgaben bei denen sehr weitreichende Mathematikkenntnisse erforderlich sind.

Der Rechenaufwand ist ebenfalls sehr unterschiedlich. Einige Aufgaben sind selbst primitiv implementiert so schnell das man die Zeit kaum messen kann. Bei vielen Aufgaben muss man aber etwas genauer über die Effizienz der eingesetzten Lösung nachdenken. Das Optimieren der Lösung auf eine bestimmte Zeit ist ein interessanter Aspekt des Ganzen.

Über eine Million Menschen haben mindestens eine der gestellten Aufgaben gelöst. Und natürlich haben einige davon ihre Lösungen im Internet geteilt. Aber meistens nur den Code. Eine Erklärung der mathematischen Hintergründe wird eher selten gegeben. Eine Seite mit solchen Erklärungen ist die von [Kristian](https://www.mathblog.dk/project-euler-solutions). Hier schaue ich immer nach wenn ich den Verdacht habe das es noch einen mathematischen Trick gibt den ich nicht sehe.

Ich werde für mich selbst versuchen so viele Aufgaben wie möglich zu lösen. Ich werde aber aus Zeitgründen nicht zu jeder gefundenen Lösung einen Artikel schreiben können. In meinem [github-Account](https://github.com/RobertoMolinero/kataCollectionJavaAndJUnit5/tree/main/src/main/java/euler) lege ich die Lösungen dennoch alle ab. 

### 1. Problem 001: Multiples of 3 or 5

Original: If we list all the natural numbers below 10 that are multiples of 3 or 5, we get 3, 5, 6 and 9. The sum of these multiples is 23. Find the sum of all the multiples of 3 or 5 below 1000.

Übersetzung: Wenn wir alle natürlichen Zahlen unter 10 auflisten, die Vielfache von 3 oder 5 sind, erhalten wir 3, 5, 6 und 9. Die Summe dieser Vielfachen ist 23. Finde die Summe aller Vielfachen von 3 oder 5 unter 1000.

### 2. Die einfache Lösung

Die erste Aufgabe befasst sich mit Schulmathematik. Die einfachste Lösung ist alle Zahlen bis zum Limit durchzugehen. Jede Zahl wird auf Teilbarkeit von 3 und 5 geprüft und ggf. auf eine Variable aufsummiert.

```java
public class MultiplesOf3Or5 {
 
    public static int calculateSimple(int limit) {
        int sum = 0;

        for (int i = 1; i < limit; i++) {
            if (i % 3 == 0 || i % 5 == 0) {
                sum += i;
            }
        }

        return sum;
    }
}
```

Das Ganze kann man auch mit Streams schreiben. Es macht im Hintergrund das gleiche, ist aber in vielen Fällen lesbarer.

```java
public class MultiplesOf3Or5 {

    public static int calculateSimpleWithStream(int limit) {
        return IntStream
                .range(1, limit)
                .filter(n -> n % 3 == 0 || n % 5 == 0)
                .sum();
    }
}
```

Ein kleiner Test bestätigt das richtige Ergebnis von beiden Varianten.

```java
public class MultiplesOf3Or5 {

    @ParameterizedTest
    @MethodSource("exampleCalculations")
    void calculateSimple(int limit, int expectedSum) {
        Assert.assertEquals(expectedSum, MultiplesOf3Or5.calculateSimple(limit));
    }

    @ParameterizedTest
    @MethodSource("exampleCalculations")
    void calculateSimpleWithStream(int limit, int expectedSum) {
        Assert.assertEquals(expectedSum, MultiplesOf3Or5.calculateSimpleWithStream(limit));
    }

    static Stream<Arguments> exampleCalculations() {
        return Stream.of(
                arguments(10, 23),
                arguments(100, 2318),
                arguments(1000, 233168),
                arguments(10000, 23331668)
        );
    }
}
```

### 3. Die Zeitmessung

Jetzt möchte ich noch die Zeit messen. Aus dem [GitHub Account des JUnit Teams](https://github.com/junit-team/junit5/blob/main/documentation/src/test/java/example/timing/TimingExtension.java) habe ich mir dafür eine Klasse kopiert.

```java
public class TimingExtension implements BeforeTestExecutionCallback, AfterTestExecutionCallback {

    private static final Logger logger = Logger.getLogger(TimingExtension.class.getName());

    private static final String START_TIME = "start time";

    @Override
    public void beforeTestExecution(ExtensionContext context) throws Exception {
        getStore(context).put(START_TIME, System.currentTimeMillis());
    }

    @Override
    public void afterTestExecution(ExtensionContext context) throws Exception {
        Method testMethod = context.getRequiredTestMethod();
        long startTime = getStore(context).remove(START_TIME, long.class);
        long duration = System.currentTimeMillis() - startTime;

        logger.info(() ->
                String.format("Method [%s] took %s ms.", testMethod.getName(), duration));
    }

    private Store getStore(ExtensionContext context) {
        return context.getStore(Namespace.create(getClass(), context.getRequiredTestMethod()));
    }
}
```

Es werden zwei Methoden überschrieben die vor und nach jedem Test ausgeführt werden. Vor jedem Test (beforeTestExecution(...)) wird die Systemzeit ('System.currentTimeMillis()') in Millisekunden genommen und in einer Variable 'START_TIME' im Kontext des Tests abgelegt. Nach jedem Test (afterTestExecution(...)) wird mit Hilfe dieser Variable und der neuen aktuellen Systemzeit die Dauer des Tests bestimmt und ausgegeben.

Um diese Erweiterung für die Tests anzubinden wird die Klasse mit der Annotation '@ExtendedWith' markiert.

```java
@ExtendWith(TimingExtension.class)
class MultiplesOf3Or5Test {
}
```

Die Ausgabe im Terminal:

```text
Dez. 30, 2021 2:26:36 NACHM. euler.TimingExtension afterTestExecution
INFORMATION: Method [calculateSimpleWithStream] took 2 ms.
Dez. 30, 2021 2:26:36 NACHM. euler.TimingExtension afterTestExecution
INFORMATION: Method [calculateSimpleWithStream] took 1 ms.
Dez. 30, 2021 2:26:36 NACHM. euler.TimingExtension afterTestExecution
INFORMATION: Method [calculateSimpleWithStream] took 1 ms.
Dez. 30, 2021 2:26:36 NACHM. euler.TimingExtension afterTestExecution
INFORMATION: Method [calculateSimpleWithStream] took 1 ms.
Dez. 30, 2021 2:26:36 NACHM. euler.TimingExtension afterTestExecution
INFORMATION: Method [calculateSimple] took 0 ms.
Dez. 30, 2021 2:26:36 NACHM. euler.TimingExtension afterTestExecution
INFORMATION: Method [calculateSimple] took 0 ms.
Dez. 30, 2021 2:26:36 NACHM. euler.TimingExtension afterTestExecution
INFORMATION: Method [calculateSimple] took 0 ms.
Dez. 30, 2021 2:26:36 NACHM. euler.TimingExtension afterTestExecution
INFORMATION: Method [calculateSimple] took 1 ms.
```

Jetzt weiß ich welche Testmethode in welcher Zeit abgeschlossen wurde. Ich würde aber gern noch wissen welche Testparameter jeweils genommen wurden. Deswegen habe ich die Ausgabe noch ein wenig ergänzt.

```java
public class TimingExtension implements BeforeTestExecutionCallback, AfterTestExecutionCallback {

    @Override
    public void afterTestExecution(ExtensionContext context) throws Exception {
        Method testMethod = context.getRequiredTestMethod();
        String displayName = context.getDisplayName();
        long startTime = getStore(context).remove(START_TIME, long.class);
        long duration = System.currentTimeMillis() - startTime;

        logger.info(() ->
                String.format("Method [%s] with parameter settings (%s) took %s ms.", testMethod.getName(), displayName, duration));
    }
}
```

Ich hole mir zusätzlich aus dem Kontext den 'DisplayName' und gebe ihn mit aus.

```
Dez. 30, 2021 4:09:14 NACHM. euler.TimingExtension afterTestExecution
INFORMATION: Method [calculateSimpleWithStream] with parameter settings ([1] 10, 23) took 2 ms.
Dez. 30, 2021 4:09:14 NACHM. euler.TimingExtension afterTestExecution
INFORMATION: Method [calculateSimpleWithStream] with parameter settings ([2] 100, 2318) took 1 ms.
Dez. 30, 2021 4:09:14 NACHM. euler.TimingExtension afterTestExecution
INFORMATION: Method [calculateSimpleWithStream] with parameter settings ([3] 1000, 233168) took 2 ms.
Dez. 30, 2021 4:09:14 NACHM. euler.TimingExtension afterTestExecution
INFORMATION: Method [calculateSimpleWithStream] with parameter settings ([4] 10000, 23331668) took 2 ms.
Dez. 30, 2021 4:09:14 NACHM. euler.TimingExtension afterTestExecution
INFORMATION: Method [calculateSimple] with parameter settings ([1] 10, 23) took 1 ms.
Dez. 30, 2021 4:09:14 NACHM. euler.TimingExtension afterTestExecution
INFORMATION: Method [calculateSimple] with parameter settings ([2] 100, 2318) took 0 ms.
Dez. 30, 2021 4:09:14 NACHM. euler.TimingExtension afterTestExecution
INFORMATION: Method [calculateSimple] with parameter settings ([3] 1000, 233168) took 1 ms.
Dez. 30, 2021 4:09:14 NACHM. euler.TimingExtension afterTestExecution
INFORMATION: Method [calculateSimple] with parameter settings ([4] 10000, 23331668) took 1 ms.
Dez. 30, 2021 4:09:14 NACHM. euler.TimingExtension afterTestExecution
```

### 3. Die Gaußsche Summenformel

Eigentlich könnte man an dieser Stelle stoppen. Die Laufzeit ist fast nicht messbar. Man müsste in der Geschichte der Rechentechnik schon einige Schritte rückwärts machen um hier eine Bearbeitungszeit im Bereich von Sekunden zu erreichen.

Es gibt aber eine Verbesserung, die man später vielleicht noch mal gebrauchen könnte. Es handelt sich um die Gaußsche Summenformel.

```
1 ... 100 = (1+100) + (2+99) + (3+98) + ... + (50+51) 
```

Wenn man eine Reihe von Zahlen aufsummieren möchte, addiert man immer die erste und die letzte Zahl miteinander. Da das paarweise geschieht gibt es halb so viele Paare wie Elemente der Reihe. Die Summe aller Zahlen von 1 bis 100 lautet also: 101 * 50 Paare = 5050.

Die Summenformel lautet für N Objekte:

```
(N+1) * (N/2)
```

Die Implementierung dieser Formel.

```java
public class MultiplesOf3Or5 {

    public static int calculateSumOfGauss(int limit) {
        double limitAsDouble = limit;
        return (int) ((limitAsDouble / 2) * (limitAsDouble + 1));
    }
}
```

Ein kleiner Test dazu.

```java
@ExtendWith(TimingExtension.class)
class MultiplesOf3Or5Test {
 
    @ParameterizedTest
    @MethodSource
    void calculateSumOfGauss(int limit, int expectedSum) {
        Assert.assertEquals(expectedSum, MultiplesOf3Or5.calculateSumOfGauss(limit));
    }

    static Stream<Arguments> calculateSumOfGauss() {
        return Stream.of(
                arguments(10, 55),
                arguments(100, 5050),
                arguments(1000, 500500),
                arguments(10000, 50005000)
        );
    }
}
```

Jetzt brauchen wir aber das Vielfache bis zu einer gewissen Grenze. Das sieht für die Vielfachen 3 und 5 wie folgt aus.

```
3 + 6 + 9 + ... + 999 = 3 * (1 + 2 + 3 + ... + 333)
5 + 10 + 15 + ... + 995 = 5 * (1 + 2 + 3 + ... + 199) 
```

Ich muss also nur das neue Limit berechnen (hier 333 bzw. 199) und kann dann die bereits geschriebene Methode mit der gaußschen Summenformel nutzen.

```java
public class MultiplesOf3Or5 {
 
    public static int calculateSumOfGaussWithMultiple(int limit, int multiple) {
        int adjustedLimit = limit / multiple;
        int sumOfGauss = calculateSumOfGauss(adjustedLimit);
        return multiple * sumOfGauss;
    }
}
```

Im Test frage ich die Teilschritte für das Endergebnis ab.

```java
@ExtendWith(TimingExtension.class)
class MultiplesOf3Or5Test {
 
    @ParameterizedTest
    @MethodSource
    void calculateSumOfGaussWithMultiple(int limit, int multiple, int expectedSum) {
        Assert.assertEquals(expectedSum, MultiplesOf3Or5.calculateSumOfGaussWithMultiple(limit, multiple));
    }

    static Stream<Arguments> calculateSumOfGaussWithMultiple() {
        return Stream.of(
                arguments(999, 3, 166833),
                arguments(999, 5, 99500),
                arguments(999, 15, 33165)
        );
    }
}
```

Zum Schluss kann ich diese Methode nutzen um wieder auf das Endergebnis zu kommen.

Da es Zahlen gibt die sowohl durch 3 als auch durch 5 teilbar sind habe ich einige Elemente doppelt gazählt. Um das zu korrigieren bestimme ich die Elemente des kleinsten gemeinsamen Vielfachen und ziehe anschliessend diese Summe wieder ab.

```java
public class MultiplesOf3Or5 {

    public static int calculateMultiplesOf3And5(int limit) {
        int withMultiple3 = calculateSumOfGaussWithMultiple(limit - 1, 3);
        int withMultiple5 = calculateSumOfGaussWithMultiple(limit - 1, 5);
        int withMultiple15 = calculateSumOfGaussWithMultiple(limit - 1, 15);
        return withMultiple3 + withMultiple5 - withMultiple15;
    }
}
```

Das Problem ist gelöst. Wer will kann sich auf der Seite einen Account anlegen und die Lösung überprüfen.

Man kann das Problem natürlich auch noch ausführlicher lösen. Zum Beispiel könnte man eine Methode schreiben die die Summe für beliebige Zahlenpaare löst. Oder für eine bebliebig lange Liste von Zahlen. Dadurch wird die Lösung natürlich noch mal ein ganzes Stück komplexer.

Ich selbst werde es erst mal an dieser Stelle belassen und mir später noch ein paar andere Rätsel dieser Seite vornehmen.
