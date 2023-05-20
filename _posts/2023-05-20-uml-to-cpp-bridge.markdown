---
layout: single
toc: false
title: "Die Brücke zwischen UML und C++: Beziehungen in Klassendiagrammen und ihre Implementierung"
date: 2023-05-20 06:00:10 +0200
categories: Software-Design
comments: true
---

Ein UML-Klassendiagramm visualisiert die unterschiedlichen Klassen eines Systems, ihre Attribute, Operationen und Beziehungen zueinander. Bei korrekter Anwendung veranschaulicht UML genau, wie Klassendiagramme in Code umgesetzt werden können. Das Kernproblem - und oft der schwierigste Teil - besteht darin, die vielfältigen Klassenbeziehungen korrekt zu interpretieren.

Im Folgenden erläutere ich die wichtigsten Beziehungstypen in UML-Klassendiagrammen. Jede Beziehung wird kurz erklärt, ein Alltagsbeispiel illustriert sie und ein Abschnitt mit einem C++-Codebeispiel schließt den Abschnitt ab.

Ziel dieses Artikels ist es, auch codeorientierten Menschen - wie mir selbst - klarzumachen, wie ein UML-Klassendiagramm in Code umgesetzt wird. Der Artikel richtet sich an Entwickler mit bereits vorhandener UML-Erfahrung, kann jedoch auch Anfängern helfen, die ihre ersten Schritte mit UML machen.

{:refdef: style="text-align: center;padding: 20px"}

# Assoziation 

Eine Assoziation ist eine "kennt-ein"-Beziehung; keines der Objekte ist Teil oder Mitglied des anderen. Es ist die schwächste Beziehung, die Besitz anzeigt.

Das folgende Bild illustriert die gerichtete Assoziation "``Foo`` kennt ein ``Bar``".


![Assoziation](/assets/img/uml/uml_elements-Assoziation.drawio.png){:width="60%"}
{:refdef}

Wenn der Pfeil weggelassen wird, bedeutet dies, dass beide Klassen sich gegenseitig kennen. Aus Gründen der Einfachheit konzentriere ich mich auf das Beispiel der gerichteten Assoziation.

**Alltagsbeispiel:** Arzt und Patient - Eine Klasse "Arzt" und eine Klasse "Patient" können eine gerichtete Assoziation haben. Ein Arzt kümmert sich um seine Patienten und kennt deren medizinische Geschichte, aber ein Patient kennt normalerweise nicht alle Details über seinen Arzt. In diesem Fall ist die Assoziation vom Arzt zum Patienten gerichtet.

## Beispiel-Implementierung

```c++
class Bar {
  …
}

class Foo {
  Foo(Bar *bar) : bar_ptr(bar) {}
 
  void SetBar(Bar *bar) { bar_ptr = bar; }
 
  void Func() { bar_ptr->FuncA(); }

  …

  Bar *bar_ptr; // pointer
};
```


# Aggregation

Die Aggregation ist eine spezielle Form der Assoziation und stellt eine "hat-ein"-Beziehung dar. Es bedeutet, dass ich ein Objekt habe, das ich mir geliehen habe. Ich kann weiterleben, auch wenn dieses Objekt nicht mehr existiert.

Das folgende Bild stellt die Aggregation "``Foo`` hat ein ``Bar``" dar.

![Aggregation](/assets/img/uml/uml_elements-Aggregation.drawio.png){:width="60%"}
{:refdef}


Eine Aggregation kann auftreten, wenn eine Klasse eine Sammlung oder einen Container für andere Klassen ist, die enthaltenen Klassen aber nicht stark vom Container abhängig sind. Das heißt, wenn der Container zerstört wird, bleibt sein Inhalt unberührt.

Man könnte Aggregation und Assoziation verwechseln, da der Unterschied lediglich logisch ist: Es hängt davon ab, ob das Objekt Teil des anderen ist oder nicht.

**Alltagsbeispiel:** Fußballteam und Spieler - Ein Fußballteam besteht aus vielen Spielern. In diesem Fall ist das Fußballteam das Ganze und die Spieler sind die Teile. Ein Fußballteam "hat" Spieler.

## Beispiel-Implementierung

Wie zuvor beschrieben unterscheiden sich die Implementierungen von Assoziation und Aggregation nicht. 

```c++
class Bar {
  …
}

class Foo {
public:
  Foo(Bar *bar) : bar_ptr(bar) {}
 
  void SetBar(Bar *bar) { bar_ptr = bar; }
 
  void Func() { bar_ptr->FuncA(); }

  …

  Bar *bar_ptr; // pointer
};
```



# Komposition

Eine Komposition ist eine spezielle Form der Aggregation und stellt eine "besitzt-eine" oder "gehört-zu"-Beziehung dar. Ich besitze ein Objekt und bin für seine Lebensdauer verantwortlich. Wenn ich nicht mehr existierte, dann existiere das Objekt auch nicht mehr.

Das folgende Bild stellt die Komposition "``Foo`` besitzt ein ``Bar``" dar.

![Komposition](/assets/img/uml/uml_elements-Komposition.drawio.png){:width="60%"}
{:refdef}


Dies ist eine stärkere Form der "hat-ein"-Beziehung, die impliziert, dass die Lebensdauer des Teil-Objekts (der "Besitz") an die des Ganzen gebunden ist.

**Alltagsbeispiel:** Mensch und Herz - Ein Mensch besitzt ein Herz. Ein Mensch kann ohne sein Herz nicht existieren. Wenn das Herz aufhört zu existieren, stirbt auch der Mensch. Hierbei ist die Komposition von der Klasse "Herz" zur Klasse "Mensch".

## Beispiel-Implementierung

```c++
// Example 1
class Foo {
public:
  Foo() : bar_ptr(nullptr) { bar_ptr = new Bar()}
  ~Foo() { delete bar_ptr; }
  …
private:
  …
  Bar *bar_ptr; // pointer
};

// Example 2
class Foo {
   …
   Bar bar;
}
```

# Generalisierung

Generalisierung ist ein anderer Ausdruck für Vererbung. Es handelt sich hierbei um "ist-ein"-Beziehungen. Damit weisen wir einem Objekt einen Oberbegriff (Kategorie) zu.

Das folgende Bild stellt die Generalisierung "``Foo`` ist ein ``Bar``" dar.

![Generalisierung](/assets/img/uml/uml_elements-Generalisierung.drawio.png){:width="60%"}
{:refdef}

**Alltagsbeispiel:** Tier und Hund - Ein Hund ist ein Tier. In diesem Fall wäre "Tier" die Basisklasse und "Hund" wäre eine abgeleitete Klasse.


## Beispiel-Implementierung

```c++
class Bar {
public:
  ~virtual Bar() {};

  virtual void FuncA() { … /* Bar::FuncA implementation */ }
  virtual void FuncB() { … /* Bar::FuncB implementation */ }
  …
};

// Generalization of Bar
class Foo : public Bar {
public:
  virtual void FuncA() { Bar::FuncA(); … /* Foo::FuncA implementation */}
  virtual void FuncB() { Bar::FuncB(); … /* Foo::FuncB implementation */}
}

```

# Abhängigkeit

Abhängigkeiten repräsentieren eine "benutzt-ein"-Beziehung zwischen zwei Klassen. Hierbei kann eine Änderung in einer Klasse Änderungen in der abhängigen Klasse erfordern.

Das folgende Bild stellt die Abhängigkeit "``Foo`` benutzt-ein ``Bar``" dar.

![Abhängigkeit](/assets/img/uml/uml_elements-Abhängigkeit.drawio.png){:width="60%"}
{:refdef}

**Alltagsbeispiel:** Koch und Rezept - Ein Koch ist abhängig von einem Rezept, um ein Gericht zuzubereiten. Wenn das Rezept geändert wird (etwa die Zutaten oder die Zubereitungsanweisungen), muss der Koch seine Methode zum Zubereiten des Gerichts entsprechend ändern.

## Beispiel-Implementierung
```c++
class Foo {
  ...
  void F1(Bar y) {…; y.FuncA(); }
  void F2(Bar *y) {…; y->FuncB(); }
  void F3(Bar &y) {…; y.FuncC(); }
  void F4() { Bar y; y.FuncD(); …}
  void F5() {…; Y::StaticFunc(); }
  ...
};
```

# Realisierung

Eine Realisierung ist eine Beziehung zwischen zwei Klassen, in der eine Klasse das Verhalten, das durch eine andere Klasse oder häufig durch eine Schnittstelle definiert ist, umsetzt oder "realisiert". Man könnte sagen, dass es sich um eine "erfüllt-die"-Beziehung handelt.

Das folgende Bild stellt die Realisierung "``Foo`` erfüllt das ``IBar`` Interface" dar.

![Realisierung](/assets/img/uml/uml_elements-Realisierung.drawio.png){:width="60%"}
{:refdef}


**Alltagsbeispiel:**  Es ist schwierig, ein Alltagsbeispiel zu finden, da dieses Konzept in der realen Welt kaum greifbar ist. Grob gesagt könnte eine Klasse eine Schnittstelle "Laufbar" implementieren, die eine Methode "laufen" definiert. Diese Klasse könnte ein "Hund", ein "Mensch" oder ein "Roboter" sein - alles, was "laufen" kann.

## Beispiel-Implementierung

```c++
// Abstract interface
class IBar {
  ~virtual IBar() {};

  …
  virtual void FuncA() = 0;
  virtual void FuncB() = 0;
};

// Realization of interface IBar
class Foo : public IBar{
  …
  virtual void FuncA() { … /* FuncA implementation */}
  virtual void FuncB() { … /* FuncB implementation */}
}

```

# Zusammenfassung

Dieser Artikel beleuchtet die Beziehungsarten in UML-Klassendiagrammen und ihre Umsetzung in C++ Code. Die wichtigsten Beziehungsarten sind Assoziation, Aggregation, Komposition, Generalisierung, Abhängigkeit und Realisierung. Durch den Vergleich mit Alltagsbeispielen und konkreten C++-Implementierungen werden diese Konzepte anschaulich und verständlich dargestellt.



# Bibliografie

[1] Sadique, Ali. [UML Class Diagram Explained With C++ samples](https://cppcodetips.wordpress.com/2013/12/23/uml-class-diagram-explained-with-c-samples/)  
[2] [UML 2 Tutorial - Class Diagram](https://sparxsystems.com/resources/tutorials/uml2/class-diagram.html)  
[3] [C++ Mapping to UML](https://docs.nomagic.com/pages/viewpage.action?pageId=38044261)  
[4] [Wikipedia - Class diagram](https://en.wikipedia.org/wiki/Class_diagram)
