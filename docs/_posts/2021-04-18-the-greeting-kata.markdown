---
layout: post 
title:  "The Greeting Kata"
date:   2021-04-18 13:51:59 +0200 
categories: jekyll update
---

Diese Kata habe ich aus dem Github-Account
von [Test Double](https://github.com/testdouble/contributing-tests/wiki/Greeting-Kata) entnommen.

Es ist ein klassisches Beispiel für testgetriebene Entwicklung und klassische Vorgehensweise (oder "Detroid" oder "
Bottom-Up"). Das heißt mit jeder neuen Anforderung wird eine zusätzliche Funktionalität zur vorherigen hinzugefügt.

Nur eine Sache habe ich geändert. Ich habe das sogenannte Oxford Komma weggelassen. Bei diesem speziellen Komma wird in
einer Auflistung nach dem letzten Objekt einfach noch ein Komma gesetzt. Ich finde das sieht schrecklich aus. Außerdem
macht es die Aufgabe viel zu einfach wenn man ohne nachzudenken überall mal ein Komma rankleben kann. Meine
Deutschlehrerin wäre davon auf jeden Fall nicht begeistert.

**1. Anforderung: Schreiben Sie eine Methode, die eine einfache Begrüßung ausgibt. Die Eingabe ist ein Name in Form einer Zeichenkette, die Ausgabe ist eine Begrüßung an diesen. Wenn der übergebene Name z. B. "Bob" ist, soll die Methode die Zeichenkette "Hello, Bob." zurückgeben.**

Der Test beginnt mit einem Titel. Danach folgt in einem Ausdruck eine Tabelle mit allen zu testenden Konstellationen. Mit 'headers()' werden die einzelnen
Spalten benannt. Diese Namen werden dann bei Aufruf und Test verwendet.

Anschliessend folgt in dem Abschnitt 'forAll{}' Aufruf und Test aller Konstellationen.

```kotlin
test("Requirement 1: The method greet(name) interpolates 'name' in a simple greeting.") {
    table(
        headers("name", "result"),
        row("Bob", "Hello, Bob."),
        row("Jill", "Hello, Jill."),
        row("Jane", "Hello, Jane."),
        row("Amy", "Hello, Amy."),
        row("Brian", "Hello, Brian."),
        row("Charlotte", "Hello, Charlotte.")
    ).forAll { name, result ->
        greet(name) shouldBe result
    }
}
```

Der erste Lösungsentwurf ist nicht sonderlich schwer.

```kotlin
fun greet(name: String): String {
    return "Hello, " + name + "."
}
```

Wir verbinden die Begrüßung mit dem Namen und einem Satzzeichen und schon läuft der Test durch.

Was könnte man daran noch verbessern?

Das Zusammensetzen von Zeichenketten mit '+' sieht man in der Praxis sehr häufig. Es ist aber nicht gut lesbar. Besteht
der entstehende String aus mehr als zwei Teilen wird die Zusammensetzung schnell unübersichtlich.

Fast jede Sprache bietet Funktionen die das Zusammensetzen übernehmen. Es lohnt sich diese zu suchen und von Anfang an
zu nutzen. In Kotlin kann man die Variable einfach mit vorangehenden '$' in die Zeichenkette einfügen.

```kotlin
fun greet(name: String): String {
    return "Hello, $name."
}
```

Das zweite Problem ist der feststehende Texte an sich. Es ist kein guter Stil feststehende Texte und Zahlen mit
Programmlogik zu mischen. Solche Werte sollte man in eine Konstante auslagern. Der Name der Konstante sollte sprechend
gewählt sein. In einem Kommentar an der Konstante können zusätzliche Informationen angegeben werden.

```kotlin
private const val greetingFormula = "Hello, %s."

fun greet(name: String): String {
    return String.format(greetingFormula, name)
}
```

**2. Anforderung: Behandeln Sie fehlende Werte, indem Sie einen Standardgruß einführen. Wenn der übergebene Parameter
den Wert _null_ hat, dann soll die Methode die Zeichenkette "Hello, my Friend." zurückgeben.**

Die zweite Anforderung ist nur eine kleine Ergänzung. Sollte der übergebene Wert _null_ sein, soll eine Art Standardgruß
ausgegeben werden.

Der Test fällt sehr einfach aus da es nur einen zu testenden Parameter gibt.

```kotlin
test("Requirement 2: When name is 'null', then the method should return the string \"Hello, my friend.\"") {
    greet(null) shouldBe "Hello, my friend."
}
```

Allerdings muss ich jetzt die Implementierung anpassen. In Kotlin darf man nicht einfach ein _null_ übergeben wenn ein
_String_ erwartet wird. Man muss die Möglichkeit explizit einräumen indem man den Parameter von _String_ auf _String?_
ändert.

Die einfachste Umsetzung ist eine Überprüfung am Anfang der Methode.

```kotlin
fun greet(name: String?): String {
    if (name == null) {
        return String.format(greetingFormula, "my friend")
    }
    return String.format(greetingFormula, name)
}
```

In Kotlin gibt es noch eine bessere Möglichkeit. Man kann eine Variable mit einem Standardwert belegen.

```kotlin
fun greet(name: String = "my friend"): String {
    return String.format(greetingFormula, name)
}
```

Und auch hier kann man den Text wieder in eine Konstante auslagern.

```kotlin
private const val greetingFormula = "Hello, %s."
private const val unknownName = "my friend"

fun greet(name: String = unknownName): String {
    return String.format(greetingFormula, name)
}
```

**3. Anforderung: Lerne schreien. Wenn der Name in Großbuchstaben geschrieben wird, soll die Methode den Benutzer
anschreien. Wenn der Name z. B. "JERRY" lautet, soll die Methode die Zeichenfolge "HELLO JERRY!" zurückgeben.**

Wir sollen den Nutzer anschreien. Gut, wenn er das so möchte. Der Aufbau des Tests hat sich nicht geändert. Nur der
Titel und die Testdaten werden für die neue Anforderung entsprechend angelegt.

```kotlin
test("Requirement 3: When name is all uppercase, then the method should shout back to the user.") {
    table(
        headers("name", "result"),
        row("JERRY", "HELLO JERRY!"),
        row("JILL", "HELLO JILL!"),
        row("JANE", "HELLO JANE!"),
        row("AMY", "HELLO AMY!")
    ).forAll { name, result ->
        greet(name) shouldBe result
    }
}
```

Wieder lässt sich das Problem am einfachsten lösen in dem man mit einem _if_ die Großschreibung des Parameters prüft.
Anschliessend schreit man wie jetzt gefordert oder man grüßt so wie es in der ersten Anforderung gefordert ist.

```kotlin
private const val shoutingFormula = "HELLO %s!"

fun greet(name: String = unknownName): String {
    if (name.toUpperCase() == name) {
        return String.format(shoutingFormula, name)
    }
    return String.format(greetingFormula, name)
}
```

Noch ist die Programmlogik überschaubar. Sollten aber noch mehr Abzweigungen hinzukommen müssen wir uns etwas anderes
überlegen.

**4. Anforderung: Behandeln Sie zwei übergebene Namen. Wenn der übergebene Parameter eine Liste mit zwei Namen ist, dann
sollen beide Namen ausgegeben werden. Wenn der Parameter z. B. ["Jill", "Jane"] ist, dann soll die Methode die
Zeichenkette "Hello, Jill and Jane." zurückgeben.**

Jetzt wird es langsam interessant. Wir sollen mehrere Personen gleichzeitig grüßen. Dafür wird nun eine Liste von Namen
übergeben.

```kotlin
test("Requirement 4: Handle two names of input.") {
    table(
        headers("list", "result"),
        row(listOf("Jill", "Jane"), "Hello, Jill and Jane."),
        row(listOf("Amy", "Brian"), "Hello, Amy and Brian."),
        row(listOf("Bob", "Charlotte"), "Hello, Bob and Charlotte.")
    ).forAll { list, result ->
        greet(list) shouldBe result
    }
}
```

Wir überschreiben die vorhandene Methode 'greeting()' mit einer Methode die eine Liste als Parameter nimmt.
Anschliessend geben wir wie schon mehrfach gesehen mit einer Konstante und einem Formatierungsbefehl den gewünschten
Wert zurück.

```kotlin
private const val greetingFormulaForTwo = "Hello, %s and %s."

fun greet(list: List<String>): String {
    return String.format(greetingFormulaForTwo, list[0], list[1])
}
```

**5. Anforderung: Verarbeiten Sie eine beliebige Anzahl von Namen als Eingabe. Wenn eine Liste mehr als zwei Namen
beinhaltet, trennen Sie diese mit Kommas und schließen Sie mit einem "and" ab. Bei der Eingabe
von ["Amy", "Brian", "Charlotte"] soll die Zeichenkette "Hallo, Amy, Brian und Charlotte." zurückgeben werden.**

Jetzt kommt noch etwas Komplexität hinzu. Eine beliebig lange Liste von Namen soll gegrüßt werden. In unserem Test bin ich
mal bis zur Anzahl von sechs Namen gegangen. Das sollte reichen.

```kotlin
test("Requirement 5: Handle an arbitrary number of names as input for greeting.") {
    table(
        headers("list", "result"),
        row(listOf(), "Hello, my friend."),
        row(listOf("Amy"), "Hello, Amy."),
        row(listOf("Amy", "Brian"), "Hello, Amy and Brian."),
        row(listOf("Amy", "Brian", "Charlotte"), "Hello, Amy, Brian and Charlotte."),
        row(listOf("Amy", "Brian", "Charlotte", "Jill"), "Hello, Amy, Brian, Charlotte and Jill."),
        row(
            listOf("Amy", "Brian", "Charlotte", "Jill", "Jane"),
            "Hello, Amy, Brian, Charlotte, Jill and Jane."
        ),
        row(
            listOf("Amy", "Brian", "Charlotte", "Jill", "Jane", "Bob"),
            "Hello, Amy, Brian, Charlotte, Jill, Jane and Bob."
        )
    ).forAll { list, result ->
        greet(list) shouldBe result
    }
}
```

Die Sonderfälle 'leere Liste' und 'Liste mit einem Element' habe ich gleich am Anfang behandelt. In beiden Fällen rufe
ich einfach die bereits bestehende Methode mit dem Parameter 'String' auf.

Jetzt zum eigentlichen Problem. Das Problem ist das Satzende. Man hängt alle Werte mit Kommas getrennt aneinander und
schliesst mit Punkt ab. Nur zwischen dem vorletzten und dem letzten Element kommt ein abschließendes 'and'. Wie erkennt
man das in einer einfachen vorwärts gerichteten Schleife rechtzeitig?

Eine einfache Lösung besteht darin das Problem gleich zu Beginn auszuschalten. Man setzt die letzten beiden Elemente der
Liste zu einem Objekt 'Satzende' zusammen und entfernt sie aus der Liste. Anschliessend kann man in einer Schleife die
übrigen Elemente mit einem Komma getrennt an den Anfang stellen. Ganz zum Schluss hängt man vorn die Grußformel an.

```kotlin
private const val greetingStart = "Hello, %s"
private const val greetingEnd = "%s and %s."
private const val separator = ", "

fun greet(list: List<String>): String {
    if (list.isEmpty()) return greet()
    if (list.size == 1) return greet(list[0])

    val takeLast = list.takeLast(2)
    val endOfSentence = String.format(greetingEnd, takeLast[0], takeLast[1])

    val beginning = list.dropLast(2)
    val reversed = beginning.asReversed()

    val stringBuilder = StringBuilder(endOfSentence)

    for (s in reversed) {
        stringBuilder.insert(0, s + separator)
    }

    return String.format(greetingStart, stringBuilder.toString())
}
```

Da das Grüßen einer beliebigen Anzahl von Nutzern eine Verallgemeinerung des Grüßens von zwei Personen ist kann die
dafür verwendete Logik überschrieben und die Konstante 'greetingFormulaForTwo' gelöscht werden.

**6. Anforderung: Erlauben Sie das Mischen von normalen und geschrienen Namen, indem Sie die Antwort in zwei Begrüßungen
trennen. Wenn der übergebene Parameter z. B. ["Amy", "BRIAN", "Charlotte"] lautet, soll die Methode die Zeichenfolge "
Hello, Amy and Charlotte. AND HALLO BRIAN!" zurückgeben.**

Es soll eine gemischte Liste übergeben werden mit Personen die gegrüßt und angeschrien werden sollen. Und diese Liste
ist ungeordnet.

```kotlin
test("Requirement 6: Allow mixing of normal and shouted names by separating the response into two greetings.") {
    table(
        headers("list", "result"),
        row(listOf("Amy", "BRIAN", "Charlotte"), "Hello, Amy and Charlotte. AND HELLO BRIAN!"),
        row(
            listOf("Amy", "BOB", "BRIAN", "Charlotte"),
            "Hello, Amy and Charlotte. AND HELLO BOB! AND HELLO BRIAN!"
        ),
        row(listOf("Amy", "Brian", "CHARLOTTE"), "Hello, Amy and Brian. AND HELLO CHARLOTTE!"),
        row(
            listOf("Amy", "BOB", "Brian", "CHARLOTTE"),
            "Hello, Amy and Brian. AND HELLO BOB! AND HELLO CHARLOTTE!"
        ),
        row(listOf("Amy", "Brian"), "Hello, Amy and Brian."),
        row(listOf("BOB", "CHARLOTTE"), "HELLO BOB! AND HELLO CHARLOTTE!"),
        row(listOf("Amy", "CHARLOTTE"), "Hello, Amy. AND HELLO CHARLOTTE!"),
    ).forAll { list, result ->
        greet(list) shouldBe result
    }
}
```

Zwei Schritte sind hier zu implementieren. Zuerst muss die Liste in zwei Listen aufgeteilt werden. Eine Liste soll die
groß- und eine die kleingeschriebenen Wörter enthalten. Anschliessend kann man mit den beiden Listen die Methoden zum
Grüßen bzw. Anschreien ansteuern.

Für den ersten Teil gibt es viele Möglichkeiten. Die meisten Programmieranfänger werden das Problem lösen in dem sie mit
einer Schleife über die Liste iterieren, die Werte einzeln testen und dann einer Ergebnisliste hinzufügen. Das
funktioniert, allerdings geht es einfacher und schneller mit Streams. Ähnlich wie in SQL werden hier Methoden zum
Verarbeiten ganzer Tabellen bzw. Listen angeboten. Das 'Wie' muss dabei nicht implementiert werden. Nur das 'Was'.

```kotlin
fun splitListForGreetAndShout(mixedList: List<String>): Pair<List<String>, List<String>> {
    val partition = mixedList.partition { it.toUpperCase() != it }
    return Pair(partition.first, partition.second)
}
```

Die Liste wird partitioniert nach der Funktion '{ it.toUpperCase() != it }', also ob eine Zeichenkette nach Umwandlung
in Großbuchstaben sich verändert hat.

Um diese Teilfunktionalität zu testen habe ich einen Test extra geschrieben.

```kotlin
test("Requirement 6a: A mixed list is correctly split into upper and lower case") {
    table(
        headers("list", "result"),
        row(listOf("Amy", "BRIAN", "Charlotte"), Pair(listOf("Amy", "Charlotte"), listOf("BRIAN"))),
        row(
            listOf("Amy", "BOB", "BRIAN", "Charlotte"),
            Pair(listOf("Amy", "Charlotte"), listOf("BOB", "BRIAN"))
        ),
        row(listOf("Amy", "Brian", "CHARLOTTE"), Pair(listOf("Amy", "Brian"), listOf("CHARLOTTE"))),
        row(
            listOf("Amy", "BOB", "Brian", "CHARLOTTE"),
            Pair(listOf("Amy", "Brian"), listOf("BOB", "CHARLOTTE"))
        ),
        row(listOf("Amy", "Brian"), Pair(listOf("Amy", "Brian"), listOf())),
        row(listOf("BOB", "CHARLOTTE"), Pair(listOf(), listOf("BOB", "CHARLOTTE"))),
        row(listOf("Amy", "CHARLOTTE"), Pair(listOf("Amy"), listOf("CHARLOTTE")))
    ).forAll { list, result ->
        splitListForGreetAndShout(list) shouldBe result
    }
}
```

Danach wird je eine Methode zum grüßen und schreien aufgerufen. Die Ergebnisse werden im letzten Schritt zusammengefügt.

```kotlin
fun greet(list: List<String>): String {
    val (lowercase, uppercase) = splitListForGreetAndShout(list)

    val greetingResult = greetingListOfPeople(lowercase)
    val shoutingResult = shoutingToListOfPeople(uppercase, lowercase.isEmpty())

    return "$greetingResult $shoutingResult".trim()
}
```

Die Methode zum Grüßen hat sich fast gar nicht verändert.

```kotlin
fun greetingListOfPeople(list: List<String>): String {
    if (list.isEmpty()) return ""
    if (list.size == 1) return greet(list[0])

    val takeLast = list.takeLast(2)
    val endOfSentence = String.format(greetingEnd, takeLast[0], takeLast[1])

    val beginning = list.dropLast(2)
    val reversed = beginning.asReversed()

    val stringBuilder = StringBuilder(endOfSentence)

    for (s in reversed) {
        stringBuilder.insert(0, s + separator)
    }

    return String.format(greetingStart, stringBuilder.toString())
}
```

Die Methode zum Schreien ist neu, aber nicht sonderlich schwer. Es muss hier kein Satz mit Kommas zusammengesetzt
werden. Jeder wird in einem neuen Satz von neuem angeschrien. Die einzige Besonderheit ist das 'AND' am Anfang. Dafür
muss ein Boolean-Wert übergeben werden der angibt ob es sich um den Anfang handelt. Ist das der Fall wird das erste
Element herausgenommen und einzeln behandelt. Die restlichen Elemente werden anschliessend in einer Schleife in Sätze
mit 'AND' zusammengefügt.

```kotlin
private const val shoutingFormula = "HELLO %s!"
private const val shoutingFormulaWithAnd = " AND HELLO %s!"

fun shoutingToListOfPeople(list: List<String>, firstElement: Boolean): String {
    if (list.isEmpty()) return ""

    val stringBuilder = StringBuilder()

    if (firstElement)
        stringBuilder.append(String.format(shoutingFormula, list[0]))

    val tail = if (firstElement)
        list.drop(1)
    else
        list

    for (name in tail)
        stringBuilder.append(String.format(shoutingFormulaWithAnd, name))

    return stringBuilder.toString().trim()
}
```

Auch für das Schreien habe ich noch einen Test ergänzt.

```kotlin
test("Requirement 6b: Handle an arbitrary number of names as input for shouting.") {
    table(
        headers("list", "firstElement", "result"),
        row(listOf(), false, ""),
        row(listOf(), true, ""),
        row(listOf("JILL"), false, "AND HELLO JILL!"),
        row(listOf("JILL"), true, "HELLO JILL!"),
        row(listOf("JILL", "JANE"), false, "AND HELLO JILL! AND HELLO JANE!"),
        row(listOf("JILL", "JANE"), true, "HELLO JILL! AND HELLO JANE!"),
        row(listOf("BOB", "JILL", "JANE"), false, "AND HELLO BOB! AND HELLO JILL! AND HELLO JANE!"),
        row(listOf("BOB", "JILL", "JANE"), true, "HELLO BOB! AND HELLO JILL! AND HELLO JANE!"),
        row(
            listOf("AMY", "BOB", "JILL", "JANE"),
            false,
            "AND HELLO AMY! AND HELLO BOB! AND HELLO JILL! AND HELLO JANE!"
        ),
        row(
            listOf("AMY", "BOB", "JILL", "JANE"),
            true,
            "HELLO AMY! AND HELLO BOB! AND HELLO JILL! AND HELLO JANE!"
        ),
        row(
            listOf("BRIAN", "AMY", "BOB", "JILL", "JANE"),
            false,
            "AND HELLO BRIAN! AND HELLO AMY! AND HELLO BOB! AND HELLO JILL! AND HELLO JANE!"
        ),
        row(
            listOf("BRIAN", "AMY", "BOB", "JILL", "JANE"),
            true,
            "HELLO BRIAN! AND HELLO AMY! AND HELLO BOB! AND HELLO JILL! AND HELLO JANE!"
        ),
        row(
            listOf("CHARLOTTE", "BRIAN", "AMY", "BOB", "JILL", "JANE"),
            false,
            "AND HELLO CHARLOTTE! AND HELLO BRIAN! AND HELLO AMY! AND HELLO BOB! AND HELLO JILL! AND HELLO JANE!"
        ),
        row(
            listOf("CHARLOTTE", "BRIAN", "AMY", "BOB", "JILL", "JANE"),
            true,
            "HELLO CHARLOTTE! AND HELLO BRIAN! AND HELLO AMY! AND HELLO BOB! AND HELLO JILL! AND HELLO JANE!"
        )
    ).forAll { list, firstElement, result ->
        shoutingToListOfPeople(list, firstElement) shouldBe result
    }
}
```

**Anforderung 7: Wenn einer der Einträge ein Komma enthält, teilen Sie diese als eigene Eingabe auf. Für die
Eingabe ["Bob", "Charlie, Dianne"] soll die Methode die Zeichenfolge "Hallo, Bob, Charlie und Dianne." zurückgeben.**

Wieder ist eine Vorbehandlung dem eigentlichen Grüßen vorzuschalten. Wir benötigen eine Funktion die alle Elemente einer
Liste nach einem Komma durchsucht und ggf. die gefundene Auflistung aufspaltet. Der Test sieht wie folgt aus.

```kotlin
test("Requirement 7a: If any entries in name are a string containing a comma, the list will be expanded.") {
    table(
        headers("list", "result"),
        row(listOf("Bob", "Charlie, Dianne"), listOf("Bob", "Charlie", "Dianne")),
        row(listOf("Bob", "Charlie, Dianne, Amy"), listOf("Bob", "Charlie", "Dianne", "Amy")),
        row(listOf("Bob, Charlotte", "Charlie, Dianne"), listOf("Bob", "Charlotte", "Charlie", "Dianne")),
        row(listOf("Bob, Charlotte", "Dianne"), listOf("Bob", "Charlotte", "Dianne")),
        row(listOf("BOB, CHARLOTTE"), listOf("BOB", "CHARLOTTE")),
        row(listOf("Amy, BOB", "Brian, CHARLOTTE"), listOf("Amy", "BOB", "Brian", "CHARLOTTE"))
    ).forAll { list, result ->
        expandList(list) shouldBe result
    }
}
```

Die Methode selbst ist nicht sehr groß. Die Liste wird in einer Schleife durchlaufen. Beinhaltet das Element ein Komma
wird es mit 'split' aufgeteilt. Ansonsten wird das Element ohne weitere Verarbeitung an die Liste angefügt.

```kotlin
fun expandList(list: List<String>): List<String> {
    val expandedListOfNames = mutableListOf<String>()

    for (name in list)
        if (name.contains(separator))
            expandedListOfNames.addAll(name.split(",").map { c -> c.trim() })
        else
            expandedListOfNames.add(name)

    return expandedListOfNames
}
```

Anschliessend wird ein Aufruf der Methode an der ersten Stelle der Methode eingebaut.

```kotlin
fun greet(list: List<String>): String {
    val expandList = expandList(list)
    val (lowercase, uppercase) = splitListForGreetAndShout(expandList)

    val greetingResult = greetingListOfPeople(lowercase)
    val shoutingResult = shoutingToListOfPeople(uppercase, lowercase.isEmpty())

    return "$greetingResult $shoutingResult".trim()
}
```

Es ist nicht explizit gefordert aber das Schreien sollte doch auch mit Kommatrennung funktionieren. Das habe ich mit
einem meiner Testfälle auch überrpüft.

**Anforderung 8: Erlauben Sie der Eingabe, die durch Anforderung 7 eingeführten absichtlichen Kommas zu umgehen.
Diese können auf die gleiche Weise wie bei CSV mit doppelten Anführungszeichen um den Eintrag herum entkommen. Wenn Name
z. B. ["Bob", "\"Charlie, Dianne\"] lautet, sollte die Methode die Zeichenfolge "Hallo, Bob und Charlie, Dianne."
zurückgeben.**

Das ist nur noch eine kleine Ergänzung zu Aufgabe 7. Trotzdem ist sie nicht in jedem Fall trivial. Es ist eine dieser
ganz kleinen Sonderwünsche die der Kunde kurz vor oder nach Release noch hat.

Wenn man bis zu diesem Punkt sauber und mit viel Tests gearbeitet hat stellt sich hier kein großes Problem. Hat man aber
alles auf die Schnelle irgendwie zusammengetippt wird es schwer bis unmöglich solche Kleinigkeiten im Nachhinein
einzufügen.

*Test*

```kotlin
test("Requirement 8: Allow the input to escape intentional commas introduced by Requirement 7.") {
    table(
        headers("list", "result"),
        row(listOf("Bob", "\"Charlie, Dianne\""), "Hello, Bob and Charlie, Dianne."),
        row(listOf("\"Bob, Charlotte\"", "\"Charlie, Dianne\""), "Hello, Bob, Charlotte and Charlie, Dianne."),
        row(listOf("\"Bob, Charlotte\"", "Charlie"), "Hello, Bob, Charlotte and Charlie."),
        row(listOf("\"BOB, CHARLOTTE\""), "HELLO BOB, CHARLOTTE!"),
        row(listOf("\"Bob, Charlotte\""), "Hello, Bob, Charlotte."),
        row(listOf("\"BOB, Charlotte\""), "Hello, BOB, Charlotte."),
        row(listOf("\"Bob, CHARLOTTE\""), "Hello, Bob, CHARLOTTE.")
    ).forAll { list, result ->
        greet(list) shouldBe result
    }
}
```

Eine kleine Hilfsfunktion schaut ob ein Element mit Anführungszeichen beginnt und endet.

```kotlin
fun isEscaped(name: String): Boolean {
    return name.startsWith("\"") && name.endsWith("\"")
}
```

Und diese Funktion kann dann in der ursrpünglichen Funktion zum Aufspalten von Listen eingebaut werden.

```kotlin
fun expandList(list: List<String>): List<String> {
    val expandedListOfNames = mutableListOf<String>()

    for (name in list)
        if (name.contains(separator) && !isEscaped(name))
            expandedListOfNames.addAll(name.split(",").map { c -> c.trim() })
        else expandedListOfNames.add(name.trim('\"'))

    return expandedListOfNames
}
```

Die Aufspaltung der Liste erfolgt nur dann wenn das Element nicht mit Hochkommas umfasst ist. Ansonsten wird das Element ohne Hochkommas zurückgegeben.

**Schlussbetrachtungen**

Die Aufgabe begann einfach und baute langsam aber sicher immer mehr Komlexität auf.

Ich denke das die Aufgabe komplex genug ist um eine perfekte Lösung auszuschliessen. Aber genau deswegen ist sie auch interessant.

Warum sollte man noch Schach spielen wenn es einen, nicht widerlegbaren Weg zum Matt gäbe? 

Ich selbst bin ganz zufrieden mit meinem Ergebnis. Alle Anforderungen laufen, auch nach dem ich sie so präzise wie möglich genommen habe. Die Lösung enthält wenig Redundanzen und ist auch insgesamt recht kompakt. Die Lesbarkeit empfinde ich selbst als gut.
