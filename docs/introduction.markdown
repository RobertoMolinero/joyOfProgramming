---
layout: page
title: Einführung
permalink: /introduction/
---

Auf dieser Seite geht es um das Thema Coding Dojo bzw. Coding Kata. 

Ein Coding Dojo bezeichnet eine Art das Programmieren zu erlernen und zu üben. Eine einzelne Aufgabe, eine sogenannte Coding Kata, wird eingeübt. Es wird versucht Schritt für Schritt eine Lösung und dann eine bessere Lösung zu finden.

Sinn und Zweck ist dabei das Üben selbst. Die Lösung ist in den meisten Fällen nur ein Nebenprodukt.

Folgende Prinzipien versuche ich dabei anzuwenden:

### 1. Schritt für Schritt.

Je nach Aufgabe ist es notwendig, die Aufgabe in mehrere Teilaufgaben zu unterteilen. Nach dem römischen Prinzip "Teile und herrsche" versucht man nicht, die gesamte Aufgabe sofort zu lösen. Stattdessen löst man die entstehenden Teilaufgaben.

Die Teilaufgaben müssen technisch überprüfbar sein. Es muss durch einen schriftlichen Test festgestellt werden können, ob die Anforderung erfüllt ist. Die Größe der erstellten Pakete sollte so gewählt werden, dass sie ohne Probleme innerhalb eines Tages erledigt werden können.

### 2. Testgetriebene Entwicklung

* Übersetzen der Anforderungen in einen Test
* Ausführen und Fehlschlagen des Tests
* Eine erste Implementierung
* Refactoring
* Alle Tests
* Dokumentation prüfen

### 3. Sicher voranschreiten

Ein einmal gemachter Fortschritt soll nicht wieder verloren gehen. Deshalb folgt auf jeden Schritt ein Sichern mit Hilfe eines Versionierungssystems. In meinem Fall ist das Git.

Ich schreibe das nicht extra auf. Aber der Leser sollte dies selbst tun. Das verhindert langes Suchen und Frustration.

### 4. Dokumentation

Dieser Teil ist für die meisten Menschen am nervigsten. Das Problem heißt "Vergesslichkeit". Spätestens nach ein paar Monaten weiß niemand mehr, was der Code macht. Und dann werden Änderungen zu einem Glücksspiel.

Die Dokumentation sollte hier aus mehreren Teilen bestehen. Der erste Teil ist der Test. Dieser sollte alle Anforderungen möglichst aussagekräftig auflisten und zeigen, mit welchen Parametern diese Anforderungen getestet wurden.

Der produktive Code wird zusätzlich an allen von außen zugänglichen Stellen mit Kommentaren versehen.
