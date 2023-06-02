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

Die Pfeile im Diagramm symbolisieren Abhängigkeiten, welche am besten durch den Satz "A benutzt B" erklärt werden. Bezogen auf das Diagramm bedeutet dies zum Beispiel "``cli`` benutzt ``use_cases``". Beachte dabei, dass die Beziehungen gerichtet sind: es ist ``use_cases`` verboten, ``cli`` zu "benutzen".


Für weiterführende Informationen über die verschiedenen Arten von Abhängigkeiten in UML verweise ich auf meinen eigenen Artikel zu diesem Thema. [link]({% post_url 2023-05-20-uml-to-cpp-bridge %})


## Schichten

Die Architektur des Kommandozeilenwerkzeugs ist in vier Schichten gegliedert: External, Use-Cases, Operations und Models. Die wichtigste Regel ist, dass eine Schicht nur Abhängigkeiten zur nächst tieferen Schicht aufweisen darf. Aus schmerzhafter Erfahrung empfehle ich, sich strikt an diese Regel zu halten, um eine ungeordnete und unverständliche Architektur zu vermeiden. 

Jede Schicht hat spezifische Aufgaben und Funktionen, die nachfolgend erläutert werden.

__External__: Diese Schicht verbindet das Programm mit der externen Welt. Hier werden die Kommandozeilenoptionen verarbeitet. Idealerweise sollte hier eine Bibliothek verwendet werden, die diese Aufgabe bewältigt. 

{: .notice--info} 
Bei Python bevorzuge ich [Click](https://click.palletsprojects.com/en/) für diese Aufgabe.

__Use-Cases__: In dieser Schicht liegen die  Anwendungsfälle, hier Use-Case, des Tools. Ein Use-Case beschreibt eine auszuführenden Aufgabe, etwa "Konvertiere Datei von Format A nach Format B". Er sollte entsprechend benannt und dokumentiert werden. 

__Operations__: In dieser Schicht liegen die für die Durchführung der Aufgabe notwendigen Klassen. Im vorliegenden Fall sind das Klassen zum Lesen des Datenmodells  (``read``), Klassen zum umwandeln eines Datenmodells in eine anderes (``transform``) und schließlich Klassen zum anzeigen eines Datenmodells (``view``).

__Models__: Schließlich gibt es die Models-Schicht. Diese enthält alle Datenmodelle. Gemäß [Wikipedia](https://de.wikipedia.org/wiki/Datenmodell) definieren wir hier ein Datenmodell als ein "Modell der zu verarbeitenden Daten eines Anwendungsbereichs und ihrer Beziehungen zueinander". Diese Schicht sollte möglichst keine Logik enthalten.

{: .notice--info} 
Bei Python empfehle ich den Einsatz von [dataclasses](https://docs.python.org/3/library/dataclasses.html) und/oder [named tuples](https://docs.python.org/3/library/collections.html#collections.namedtuple) um das Datenmodel abzubilden.


# Detailierte Erklärung
Am besten kann man die Details der Architektur am Beispiel erklären. Hierzu gehen wir davon aus das ein Benutzer unser Kommandozeilenwerkzeug wie folgt aufruft:

```
my_tool file1.xyz file2.xyz
```

Es folgt folgender Ablauf. Durch Code im Namespace ``external`` werden die Kommandozeilenargumente ausgewertet und anschließend ein Use Case, aus dem Namespace ``use_cases``, ausgeführt. Ich betrachte einen Use Case als eine eigenständige Aufgabe des Kommandozeilenwerkzeugs.

Gehen wir beispielsweise davon aus, dass die Aufgabe von `my_tool` ist, alle Zeichen in einer Datei in Großbuchstaben umwandeln. Hier könnte der zugehörige Use-Case folgendermaßen lauten:

```python
uc_convert_all_chars_to_uppercase(…)
```

Innerhalb des Use Cases wird die tatsächliche Aufgabe ausgeführt. Da ein Kommandozeilenwerkzeug in der Regel keine weiteren Eingaben unterstützt, kann man es stets gemäß dem [EVA Prinzips](https://de.wikipedia.org/wiki/EVA-Prinzip) organisieren. Das bedeutet wir teilen die Aufgabe in einen Eingabe-, einen Verarbeitungs- sowie einen Ausgabeschritt auf. Der Use Case führt diese Schritte immer in dieser Reihenfolge aus, ohne Abweichung!

{: .notice--warning} 
Beachte, dass es verführerisch sein kann, Dateien während der gesamten Programmlaufzeit zu lesen. Mein Ratschlag lautet: Vermeide das! Du hast ausreichend Speicherplatz. Lade alles, was du benötigst, in den Speicher und arbeite dann damit. Das macht den Vorgang viel übersichtlicher.

Das EVA Prinzip wird wie folgt auf die Architektur abgebildet:

**Eingabe** -  entspricht dem ``read`` Namespace    
**Verarbeitung** - entspricht dem ``transform`` Namespace  
**Ausgabe** - entspricht dem ``view`` Namespace  

Unsere Architektur bietet also jedem der genannten Vorgänge einen eigenen Namespace und somit einen Raum für Implementierungen.

Im ``read`` Namespace werden Dateien eingelesen und in ein Datenmodell konvertiert. Im vorliegenden Beispiel wäre dies das lesen der Datei `file1.xyz`. Das resultierende Datenmodell wird dem Use Case zurückgegeben.

Mit diesem Modell als Argument führt der Use Case eine oder mehrere Operationen im ``transform`` Namespace aus, um die Daten in das gewünschte Format zu bringen. Wiederum auf das Beispiel bezogen wandeln wir die im Speicher liegenden Zeichenketten in eine Großbuchstabendarstellung um. Anschließend folgt die Rückgabe an den Use-Case.

Abschließend wird das umgewandelte Modell durch Operationen aus dem ``view`` Namespace übergeben und angezeigt. Für das Beispiel beudetet das, wir speichern die Ergebnisse in `file2.xyz`.

Der Namespace ``model_ops``, wie im UML-Diagramm dargestellt, hat eine Sonderstellung. Er enthält Modelllogik, die so allgemein ist, dass sie auch von Views verwendet werden kann und sollte. Häufig handelt es sich dabei um Iteratoren, die ein Modell auf eine bestimmte Weise darstellen. Es wäre natürlich paradox derartige Logik in jedem Namespace separat auszuführen, da für gibt es eben ``model_ops``. 

{: .notice--warning}
Ein Wort der Warnung: Der Namespace ``models`` dient nicht dazu Business Logic zu integrieren die über den Scope eines Models hinausgeht. Hierür kann der Namespace ``model_ops`` ebenfalls verwenden werden. Wo genau die Grenze gezogen wird ist abhängig von der persönlichen Einschätzung.

# Projektstruktur
Abschließend bleibt die Frage, wie die Architektur auf ein Projekt abgebildet werden kann. Momentan verfolge ich den Ansatz, für jeden Namespace ein Verzeichnis zu erstellen. Dabei bevorzuge ich eine flache Abbildung der Architektur auf die Verzeichnisse, d.h. alle Namespaces befinden sich direkt unter dem Hauptverzeichnis.

Das bedeutet jedoch nicht, dass wir die Schichtenarchitektur nicht befolgen müssen. Aus dieser Perspektive schlage ich folgende Projektstruktur vor:

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
Bei Python sind unter der Verzeichnissen die Module zu finden in denen die jeweiligen Objekte abliegen.

Für sehr einfache Projekte kann man die vorgeschlagene Struktur auch in einer einzigen Datei umsetzen. Es liegt im Ermessen des Anwenders, die Struktur so zu gestalten, dass sie hilfreich ist und bleibt.

# Warum so konkret? Das ist schlecht testbar!

Wenn man die Architektur betrachtet stellt man fest das alles direkte "use" Beziehungen sind. Wir können hier natürlich auch alles als Objekte in den Namespaces ausführen und zwischen den Objekten Interfaces zur Kommunikation verwenden. Dies als [Dependency Inversion](https://de.wikipedia.org/wiki/Dependency-Inversion-Prinzip) bekannte Prinzip ist natürlich nicht ausgeschlossen. Insbesondere bei statischen typisierten Programmiersprachen würde ich bei komplexen Aufgaben selber diesen Weg gehen. Dies erlaubt eine besser Testbare Applikation, macht das Ganze aber meiner Meinung nach deutlich weniger übersichtlich. Für einfache Projekte rate ich daher davon ab. 

Für dynamisch typisierte Programmiersprachen gibt es noch den Weg Aufrufe für Tests zu "patchen". Als Beispiel sei hier das unter Python bekannte [monkey patching](https://en.wikipedia.org/wiki/Monkey_patch#:~:text=Monkey%20patching%20is%20a%20technique,Python%2C%20Groovy%2C%20etc.) gennant. Es erlaubt Code während der Laufzeit zu modifizieren.. Auch dies erlaubt das einfügen von Mocks wie beim Dependency Inversion Prinzip, aber ohne den zusätzlichen Schritt interfaces einzufügen. Dies ist häufig eine gute Alternative sollten Tests benötigt werden.

# Bibliografie

[1] ...
