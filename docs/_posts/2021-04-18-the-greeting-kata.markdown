---
layout: post
title:  "The Greeting Kata"
date:   2021-04-18 13:51:59 +0200
categories: jekyll update
---

I took this kata from the github account of "Test Double".

You can find the task here: [Greeting Kata](https://github.com/testdouble/contributing-tests/wiki/Greeting-Kata)

It is a classic example of test-driven development and classical approach (or "Detroid" or "Bottom-Up"). That is, with each new requirement, additional functionality is added to the previous one.

Only one thing I changed. I have omitted the so-called Oxford comma. This particular comma simply adds another comma after the last object in a listing. I think that looks terrible. And on the other hand it makes the task much too easy if you can put a comma everywhere without thinking. My German teacher would definitely not be thrilled with this.

--- 
Requirement 1: Write a method that outputs a simple greeting.

**Test**

The input is a name in the form of a string, the output is a greeting to it. For example, if the name passed is "Bob," the method should return the string "Hello, Bob."

The test consists of a title, a setup, the execution and testing of the method, and a table listing all constellations to be tested.

{% highlight groovy %}
@Unroll
def "Requirement 1: The method greet(name) interpolates \"name\" in a simple greeting. For the input of \"#input\" we get the solution \"#output\""() {
given:
Greeting sut = new Greeting()

    when:
    def result = sut.greeting(input)

    then:
    result == output

    where:
    input       | output
    "Bob"       | "Hello, Bob."
    "Jill"      | "Hello, Jill."
    "Jane"      | "Hello, Jane."
    "Amy"       | "Hello, Amy."
    "Brian"     | "Hello, Brian."
    "Charlotte" | "Hello, Charlotte."
}
{% endhighlight %}

**Development**

The first draft solution is not difficult.

{% highlight java %}
package greeting;

public class Greeting {
    public String greeting(String name) {
        return "Hello, " + name + ".";
    }
}
{% endhighlight %}

**Refactoring 1/2**

Feststehende Texte und Zahlen mit Programmlogik zu mischen ist kein guter Stil. Solche Werte sollte man in eine Konstante auslagern und kommentieren. Die Konstante kann dann auch an mehreren Stellen wiederverwendet werden.

{% highlight java %}
private static final String GREETING_FORMULA = "Hello, ";
private static final String GREETING_FORMULA_END = ".";

public String greeting(String name) {
    return GREETING_FORMULA + name + GREETING_FORMULA_END;
}
{% endhighlight %}

**Refactoring 2/2**

Auch das ist aber noch nicht so richtig schön. Das Zusammenaddieren von Texten sieht man häufig. Es hat aber mehrere Nachteile. Selbst mit gut benannten Konstanten wird die Zusammensetzung schnell lang. Und wenn die einzusetzende Variable in der Mitte steht muss man mehrere Konstanten anlegen.

Deswegen ist es besser von Anfang an auf Methoden zurückzugreifen die ein Einfügen in bestehende Textmuster erlauben. Solche Methoden sind in ganz ähnlicher Form in vielen Programmiersprachen vorhanden.

{% highlight java %}
private static final String GREETING_FORMULA = "Hello, %s.";

public String greeting(String name) {
    return String.format(GREETING_FORMULA, name);
}
{% endhighlight %}

---
Anforderung 2: Behandeln Sie fehlende Werte, indem Sie einen Standardgruß einführen. Wenn der übergebene Parameter den Wert _null_ hat, dann soll die Methode die Zeichenkette "Hello, my Friend." zurückgeben.

To be continued...