---
layout: single
toc: false
title: "CLI Architecture"
date: 2023-06-02 01:19:10 +0200
categories: Software-Design
comments: true
tagline: Obwohl oft als komplex betrachtet, erweisen sich eigenentwickelte Kommandozeilenwerkzeuge für versierte Computernutzer als unverzichtbar. Dieser Artikel stellt eine übersichtliche Sw-Architektur für diese Tools vor, um deren Entwicklung zu strukturieren.
header:
  overlay_image: assets/img/frontmatter/handshake.jpg
  caption:  "Image generated using midjourney"
  
---


> Automate the boring stuff - Al Sweigart

Bei Software- sowie Heimprojekten besteht oft Bedarf an Kommandozeilenwerkzeugen, die spezifische Aufgaben, wie etwa Code-Generierung, Auswertungen, Paketierung usw.., automatisieren. Allzu häufig werden diese Tools ad hoc und unstrukturiert entwickelt, implementiert, rasch fertiggestellt  und danach vergessen. Das führt leider dazu, dass sie später schwer zu verstehen und wartungsintensiv sind. Eine gute Software-Dokumentation kann dabei helfen, dieses Problem zu umgehen. Dieser Artikel schlägt eine zusätzliche Maßnahme vor: Eine einfache Architektur für Kommandozeilenwerkzeuge, die das Verstehen und die Weiterentwicklung derselbigen erleichtern soll.

# Software-Architektur

Das folgende Bild stellt die vorgeschlagene Software-Architektur eines Kommandozeilenwerkzeugs dar.

![cli-architecture](/assets/img/cli-architecture/cli-architecture.png)

Das Bild zeigt ein UML-Klassendiagramm. Lass Dich jedoch nicht von dieser Bezeichnung irritieren - hier werden lediglich Namespaces und ihre Beziehungen zueinander dargestellt.

{: .notice--info} 
In Python definiere ich einen Namespace einfach als Modul inklusive Untermodulen.

Die Pfeile im Diagramm symbolisieren Abhängigkeiten, welche am besten durch den Satz "A benutzt B" erklärt werden. Bezogen auf das Diagramm bedeutet dies zum Beispiel "`cli` benutzt `use_cases`". Beachte dabei, dass die Beziehungen gerichtet sind: es ist `use_cases` verboten, `cli` zu "benutzen".


Für weiterführende Informationen über die verschiedenen Arten von Abhängigkeiten in UML verweise ich auf meinen eigenen Artikel zu diesem Thema. [link]({% post_url 2023-05-20-uml-to-cpp-bridge %})


## Schichten

Die Architektur des Kommandozeilenwerkzeugs ist in vier Schichten gegliedert: External, Use-Cases, Operations und Models. Die wichtigste Regel ist, dass eine Schicht nur Abhängigkeiten zur nächst tieferen Schicht aufweisen darf. Aus schmerzhafter Erfahrung empfehle ich, sich strikt an diese Regel zu halten, um eine ungeordnete und unverständliche Architektur zu vermeiden. 

Jede Schicht hat spezifische Aufgaben und Funktionen, die nachfolgend erläutert werden.

__External__: Diese Schicht verbindet das Programm mit der externen Welt. Hier werden die Kommandozeilenoptionen verarbeitet. Idealerweise sollte hier eine Bibliothek verwendet werden, die diese Aufgabe bewältigt. 

{: .notice--info} 
Bei Python bevorzuge ich [Click](https://click.palletsprojects.com/en/) für diese Aufgabe.

__Use-Cases__: In dieser Schicht liegen die  Anwendungsfälle, hier Use-Case, des Tools. Ein Use-Case beschreibt eine auszuführenden Aufgabe, etwa "Konvertiere Datei von Format A nach Format B". Er sollte entsprechend benannt und dokumentiert werden. 

__Operations__: In dieser Schicht liegen die für die Durchführung der Aufgabe notwendigen Klassen. Im vorliegenden Fall sind das Klassen zum Lesen des Datenmodells  (`read`), Klassen zum umwandeln eines Datenmodells in eine anderes (`transform`) und schließlich Klassen zum anzeigen eines Datenmodells (`view`).

__Models__: Schließlich gibt es die Models-Schicht. Diese enthält alle Datenmodelle. Gemäß [Wikipedia](https://de.wikipedia.org/wiki/Datenmodell) definieren wir hier ein Datenmodell als ein "Modell der zu verarbeitenden Daten eines Anwendungsbereichs und ihrer Beziehungen zueinander". Diese Schicht sollte möglichst keine Logik enthalten.

{: .notice--info} 
Bei Python empfehle ich den Einsatz von [dataclasses](https://docs.python.org/3/library/dataclasses.html) und/oder [named tuples](https://docs.python.org/3/library/collections.html#collections.namedtuple) um das Datenmodel abzubilden.


# Detailierte Erklärung
Die Details unserer Architektur lassen sich am besten anhand eines Beispiels erklären. Wir nehmen an, dass ein Benutzer unser Kommandozeilenwerkzeug folgendermaßen verwendet:

```
my_tool file1.xyz file2.xyz
```

Nach diesem Aufruf geschieht Folgendes: Der Code im Namespace `external` wertet die Kommandozeilenargumente aus und führt anschließend einen Use Case aus dem Namespace `use_cases` aus. Ich sehe den Use Case als eine eigenständige Aufgabe des Kommandozeilenwerkzeugs.

Nehmen wir beispielsweise an, dass `my_tool` die Aufgabe hat, alle Zeichen in einer Datei in Großbuchstaben zu konvertieren. In diesem Fall könnte der entsprechende Use Case folgendermaßen aussehen:

```python
uc_convert_all_chars_to_uppercase(…)
```

Innerhalb des Use Case wird die eigentliche Aufgabe ausgeführt. Da ein Kommandozeilenwerkzeug in der Regel keine weiteren Eingaben während der Ausführung erfordert, kannst du es immer nach dem [EVA-Prinzip](https://de.wikipedia.org/wiki/EVA-Prinzip) strukturieren. Dies bedeutet, dass du die Aufgabe in einen Eingabe-, einen Verarbeitungs- und einen Ausgabeschritt aufteilst. Der Use Case führt diese Schritte stets in dieser Reihenfolge aus, ohne Ausnahmen!

{: .notice--warning}
Es ist verführerisch, Dateien während der gesamten Programmlaufzeit zu lesen. Mein Rat: Vermeide das! Du hast ausreichend Speicherplatz. Lade alles, was du benötigst, in den Speicher und verarbeite es dort. Das macht den Prozess viel klarer.

Das EVA-Prinzip wird in unserer Architektur wie folgt abgebildet:

__Eingabe__ - entspricht dem `read` Namespace  
__Verarbeitung__ - entspricht dem `transform` Namespace  
__Ausgabe__ - entspricht dem `view` Namespace  

Unsere Architektur bietet also jedem dieser Schritte einen eigenen Namespace und damit einen Raum für seine Implementierung.

Im `read` Namespace werden Dateien eingelesen und in ein Datenmodell konvertiert, beispielsweise das Lesen von `file1.xyz`. Das resultierende Datenmodell wird dem Use Case zurückgegeben.

Mit diesem Modell als Argument führt der Use Case eine oder mehrere Operationen im `transform` Namespace aus, um die Daten in das gewünschte Format zu bringen. Im Beispiel würden wir die im Speicher vorhandenen Zeichenketten in Großbuchstaben umwandeln und sie dann an den Use Case zurückgeben.

Schließlich wird das transformierte Modell durch Operationen im `view` Namespace zur Anzeige gebracht. Im Beispiel bedeutet das, wir speichern die Ergebnisse in `file2.xyz`.

Der Namespace `model_ops`, wie im UML-Diagramm dargestellt, nimmt eine Sonderstellung ein. Er enthält allgemeine Modelllogik, die auch von den Views verwendet werden kann und sollte. Oft handelt es sich dabei um Iteratoren, die ein Modell auf bestimmte Weise darstellen. Es wäre paradox, diese Logik in jedem Namespace separat zu implementieren, deshalb gibt es den `model_ops` Namespace.

{: .notice--warning}
__Ein Wort der Warnung:__ Der Namespace `models` ist nicht dazu gedacht, Geschäftslogik zu beinhalten, die über den Anwendungsbereich eines Modells hinausgeht. Für diesen Zweck kann der Namespace `model_ops` ebenfalls genutzt werden. Wo genau die Grenze gezogen wird, hängt von deiner persönlichen Einschätzung ab.

# Projektstruktur
Zum Abschluss steht die Frage im Raum, wie sich die Architektur in einem Projekt umsetzen lässt. Momentan verwende ich den Ansatz, für jeden Namespace ein Verzeichnis zu erstellen. Dabei bevorzuge ich eine flache Abbildung der Architektur auf die Verzeichnisse, sprich, alle Namespaces befinden sich unmittelbar unter dem Hauptverzeichnis.

Das bedeutet allerdings nicht, dass die Schichtenarchitektur missachtet werden sollte. Mit Blick auf diese Aspekte schlage ich folgende Projektstruktur vor:

```
.\external
.\model_ops
.\models
.\readers
.\transforms
.\use_cases
.\views
```

{: .notice--info}
In Python finden sich unter den Verzeichnissen die Module, in denen die jeweiligen Objekte gespeichert sind.

Für sehr einfache Projekte lässt sich die vorgeschlagene Struktur auch in einer einzigen Datei umsetzen. Es liegt in deinem Ermessen, die Struktur so zu gestalten, dass sie stets nützlich und praktikabel bleibt.

# Warum so konkret? Das ist schlecht testbar!

Wenn du die Architektur betrachtest, stellst du fest, dass es ausschließlich direkte "use"-Beziehungen gibt. Natürlich kannst du auch alles als Objekte innerhalb der Namespaces ausführen und diese Objekte gegen Interfaces implementieren. Diese Interface fungieren dann als Schnittstellen zwischen den Schichten. Dieses als [Dependency Inversion](https://de.wikipedia.org/wiki/Dependency-Inversion-Prinzip) bekannte Prinzip ist natürlich nicht ausgeschlossen. Insbesondere bei statisch typisierten Programmiersprachen (z.B. C++) würde ich bei komplexen Aufgaben diesen Weg wählen. Es ermöglicht eine besser testbare Applikation, macht das Ganze aber meiner Meinung nach deutlich weniger übersichtlich. Für einfache Projekte rate ich daher davon ab.

Für dynamisch typisierte Programmiersprachen (z.B. Python) besteht zudem die Möglichkeit, Aufrufe für Tests zu "patchen". Ein Beispiel hierfür ist das unter Python bekannte [Monkey Patching](https://en.wikipedia.org/wiki/Monkey_patch#:~:text=Monkey%20patching%20is%20a%20technique,Python%2C%20Groovy%2C%20etc.). Es ermöglicht die Modifikation von Code während der Laufzeit. Dies erlaubt das Einfügen von Mocks, wie beim Dependency Inversion Prinzip, aber ohne den zusätzlichen Schritt, Interfaces einzufügen. Dies stellt häufig eine gute Alternative dar, sollten Tests erforderlich sein.

# Zusammenfassung

Der Artikel präsentiert eine einfache, schichtenbasierte Architektur für die Entwicklung von Kommandozeilenwerkzeugen. Die Architektur besteht aus vier Hauptbestandteilen: External, Use-Cases, Operations und Models. Dabei dient die External-Schicht als Brücke zur Außenwelt, die Use-Cases-Schicht definiert die Aufgaben des Tools, die Operations-Schicht stellt Funktionen zur Aufgabendurchführung bereit und die Models-Schicht beherbergt die Datenmodelle. Diese Struktur erleichtert das Verständnis und die Entwicklung solcher Werkzeuge und kann in der Projektstruktur widergespiegelt werden.


# Bibliografie

[1] ...
