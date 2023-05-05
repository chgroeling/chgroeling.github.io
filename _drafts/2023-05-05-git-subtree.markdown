---
layout: single
toc: false
title:  "Git Subtree"
date:   2023-05-05 06:00:10 +0200
categories: Git
comments: true
---

# Motivation

Das Internet ist voll von Artikeln darüber, warum man keine Git-Submodule verwenden sollte. Doch welche Alternativen hat man? Eine Möglichkeit ist git subtree, die ich in diesem Artikel gerne erkunden möchte.

Bei Git Subtree handelt es sich um ein Shell Skript das mit git mitgeliefert wird, also direkt verfügbar ist, dass im Grunde … kurze Erklärung

# Warum kann die Verwendung von Git Submodulen eine schlechte Idee sein?

Zuvor aber nochmal die Gründe warum die Verwendung von git submodules bei einigen Entwicklern verpönt ist:

1. Git clone klont keine Submodule. Man muss `git submodule update` oder `git clone --recursive` verwenden. 
2. Git-Submodule werden nicht automatisch synchronisiert, man muss `git submodule update` durchführen.
3. Git diff gibt auch nach einem Commit etwas zurück, wenn das Submodul eine Änderung hat. Man muss in das Submodul ebenfalls committen / pushen oder die Änderung rückgängig machen. Im Grunde genommen muss man jede Operation, die man im Git-Hauptmodul durchführe, auch im Submodul durchführen, wenn Dateien in beiden geändert wurden.
4. Das Beheben von Merge-Konflikten in einem Repo ist schon schwer genug. Wenn man Änderungen am Haupt-Repo und einem Sub-Modul Repo hat wird es zur Qual.
5. Der Vergleich von Datei-Revisionen wird “fasst” unmöglich wenn man Dateien/Verzeichnisse vom Hauptprojekt in das Sub-Repo verschiebt.

Zusammengefasst braucht man einiges an Wissen im gesamten Team wie man mit Sub Modulen umgeht. Dies erzeugt “Reibung” während der Entwicklung die von der eigentlichen Entwicklungstätigkeit ablenkt.

# Welche Vor- und Nachteile hat die Verwendung von Git Subtree

Kommen wir nun zu Betrachung von Git Subtree. Was genau gewinnt oder verliert man durch Verwendung dieses Weges?

#### Vorteile

- Der Code des Unterprojekts ist sofort verfügbar, nachdem der `clone` des Superprojekts fertig ist.
- `subtree` verlangt von den Benutzern Ihres Repositorys nichts Neues zu lernen, sie können die Tatsache ignorieren, dass Sie `subtree` zur Verwaltung von Abhängigkeiten verwenden.
- `subtree` fügt keine neuen Metadaten-Dateien hinzu, wie es `submodules` tut (d.h. `.gitmodule`).
- Der Inhalt des Moduls kann geändert werden, ohne dass eine separate Kopie der Abhängigkeit irgendwo anders im Repository liegt.

#### Nachteile

- Man muss sich mit einer neuen Merge-Strategie ( `subtree`) vertraut machen. Dies brauchen aber nur diejenigen zu tun die mit dem Remote des subtree arbeiten wollen
- Es ist etwas komplizierter, den Code für die Unterprojekte wieder "upstream" zu bringen.
- Die Verantwortung, den Code von Super- und Unterprojekten in den Commits nicht zu vermischen, liegt bei Ihnen.
 




# Versuch macht klug

Am besten lässt sich die Verwendung von Git-Subtree am Beispiel erklären. Hierzu möchte ich ein paar typische Use-Cases durchgehen die man beim arbeiten
mit Git-Subtree zu tun hat.

## Setup: Erstelle zwei lokale Test-Repos

Als erstes benötigen wir ein Repository die als subtree in einem Projekt eingebunden werden soll. Zu Testzwecken erstelle ich im folgenden eine lokale Git Repo mit drei Commits.

```bash
mkdir repo1
cd repo1
git init
touch file1.txt
git add --all
git commit --message "Initial commit"

echo "Change 1" >> file1.txt
git add --all
git commit --message "First change"

echo "Change 2" >> file1.txt
cat file1.txt
git add --all
git commit --message "Second change"

cd ..
```

Als zweites benötige ich eine git repo in der die andere Repo als Subtree eingebunden werden soll. 

```bash
mkdir subtree_test
cd subtree_test
git init
touch README.md
git add --all
git commit --message "Initial commit"
cd ..
```

## Erste Aufgabe: Subtree in einem Projekt hinzufügen

Nun füge ich die Repo “repo1” meiner Repo “subtree_test” als subtree hinzu.

```bash
cd subtree_test
git subtree add --prefix my_repo1 ../repo1 master
```

Die Kommandozeilenoption “—prefix” gibt das Verzeichnis an in dem der subtree erzeugt werden soll. In unserem Fall ist dies “my_repo1” . Danach folgt die Upstream Referenz. Im Beispiel ist das unsere lokale Repository. Im Allgemein Fall ist dies meist ein Link zu einem Git Server.

Nach Ausführung des Befehls wird die “repo1” im Verzeichnis “my_repo1” angelegt.

Der git commit graph (git log --graph) sieht danach wie folgt aus:

{% include figure image_path="/assets/img/subtree/git_log_first_task.png" alt="git log --graph" %}

Folgendes fällt auf:

- Es gibt nun zwei Root Commits. Einen für die Repo “subtree_test”. Einen anderen für die Repo “repo1”
- Die gesamte History von “repo1” wurde in die Repo “subtree_test” importiert.

## Zweite Aufgabe: Subtree hinzufügen aber die history “squashen”

Setzen wir den master wieder auf den “Initial commit” zurück.

```bash
git reset --hard 72c68ec
```

Und verwenden nun die Kommandozeilenoption “—squash”

```bash
git subtree add --prefix my_repo1 ../repo1 master --squash
```

Der git commit graph sieht nun wie folgt aus.

{% include figure image_path="/assets/img/subtree/git_log_second_task.png" alt="git log --graph" %}

Die “repo1” ist immer noch Teil der Repo “subtree_test”. Nun wurde aber die History der “repo1” zuvor zusammengefasst. Im Git Jargon, es wurde ein “squash” durchgeführt. Das ist unter anderem dann nützlich wenn man die History der “repo1” nicht in seinem Hauptprojekt benötigt. 

## Dritte Aufgabe: Eine Änderung aus “repo1” in “subtree_test” übernehmen

Wechseln wir nun in “repo1” und fügen eine weitere Änderung hinzu:

```bash
cd ..
cd repo1 
echo "Change 3" >> file1.txt
git commit --message "Third change"
cd ..
```

Nun versuchen wir diese Änderung in unser Hauptrepo “subtree_test” zu übernehmen:

```bash
cd subtree_test
git subtree pull --prefix my_repo1 ../repo1 master --squash
```

Hier fällt eine der Schwächen von Git Subtree auf. Es wird kein Link zu “repo1” in dem Git Repo gespeichert. Wenn wir mit einem Remote arbeiten wollen müssen wir iesen Link zu eben diesem Remote immer wieder angeben. 

{: .notice--warning} 
Es gibt einen Bug der ein “subtree pull” ohne “—squash” verhindert wenn es zuvor verwendet wurde. Dieser Bug wurde meines Erachtens in git 2.40.x gefixt.

Der git commit graph sieht nun wie folgt aus.

{% include figure image_path="/assets/img/subtree/git_log_third_task.png" alt="git log --graph" %}

## Vierte Aufgabe: Eine Änderung aus “subtree_test” in “repo1” übernehmen

Da bei git per default nicht auf einen ausgecheckten branch gepusht werden darf muss unser “repo1” erst einmal auf einen anderen branch gesetzt werden. Dies wird wie folgt getan:

```bash
cd ..
cd repo1
git checkout -b test_branch
cd ..
```

Dieser Schritt ist nicht notwendig wenn mit richtigen “remotes” gearbeitet wird.

Nun kann man seine Änderungen wie folgt in “repo1” übernehmen.

```bash
cd my_repo1
echo "Change 4" >> file1.txt
cd ..
git commit --message "Change in subtree_test"
git subtree push --prefix my_repo1 ../repo1 master
```

Das Kommando wird nun einen “split” durchführen. Hierbei werden die Änderungen 

# Zusammenfassung

# References

- [git-subtree/git-subtree.txt at master · apenwarr/git-subtree (github.com)](https://github.com/apenwarr/git-subtree/blob/master/git-subtree.txt)