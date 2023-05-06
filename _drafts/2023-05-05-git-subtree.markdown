---
layout: single
toc: false
title:  "Git Subtree"
date:   2023-05-05 06:00:10 +0200
categories: Git
comments: true
---

# Motivation

Das Internet ist voll von Artikeln, die erklären, warum man keine Git-Submodule verwenden sollte. Aber welche Alternativen gibt es? Eine Möglichkeit ist git subtree, das ich in diesem Artikel gerne näher betrachten möchte.

Git Subtree bindet den Inhalt eines externen Repositories direkt in das Hauptrepository ein, ähnlich wie einen einfachen Ordner. Dabei bleibt die Historie des externen Repositories erhalten und es ermöglicht den Austausch von Änderungen zwischen beiden Repositories. Auf diese Weise können Entwickler Änderungen im externen Repository vornehmen und sie problemlos in das Hauptrepository integrieren oder umgekehrt.

# Warum kann die Verwendung von Git Submodulen eine schlechte Idee sein?

Bevor wir uns git subtree genauer anschauen, hier noch einmal die Gründe, warum die Verwendung von Git-Submodulen bei einigen Entwicklern unbeliebt ist:

* Git klont keine Submodule. Man muss entweder `git submodule update` oder `git clone --recursive` verwenden.
* Git-Submodule werden nicht automatisch synchronisiert, man muss `git submodule update` durchführen.
* Git diff gibt auch nach einem Commit etwas zurück, wenn das Submodul eine Änderung hat. Man muss im Submodul ebenfalls commiten/pushen oder die Änderung rückgängig machen. Im Grunde genommen muss man jede Operation, die man im Hauptmodul durchführt, auch im Submodul durchführen, wenn Dateien in beiden geändert wurden.
* Das Beheben von Merge-Konflikten in einem Repo ist schon schwer genug. Wenn man Änderungen am Haupt-Repo und einem Submodul-Repo hat, wird es zur Qual.
* Der Vergleich von Dateirevisionen wird fast unmöglich, wenn man Dateien/Verzeichnisse vom Hauptprojekt in das Submodul-Repo verschiebt.

Zusammenfassend benötigt das gesamte Team einiges an Wissen, um mit Submodulen umzugehen. Dies erzeugt Reibung während der Entwicklung, die von der eigentlichen Entwicklungsarbeit ablenkt.

# Welche Vor- und Nachteile hat die Verwendung von Git Subtree

Nun wollen wir Git Subtree genauer betrachten. Was genau gewinnt oder verliert man durch die Verwendung dieses Ansatzes?

#### Vorteile:

* Der Code des Unterprojekts ist sofort verfügbar, nachdem das Klonen des Hauptprojekts abgeschlossen ist.
* Subtree verlangt von den Benutzern Ihres Repositorys nichts Neues zu lernen; sie können die Tatsache ignorieren, dass Sie Subtree zur Verwaltung von Abhängigkeiten verwenden.
* Subtree fügt keine neuen Metadatendateien hinzu, wie es bei Submodulen der Fall ist (z.B. `.gitmodule).
* Der Inhalt des Moduls kann geändert werden, ohne dass eine separate Kopie der Abhängigkeit irgendwo anders im Repository liegt.

#### Nachteile:

* Man muss sich mit einer neuen Merge-Strategie (Subtree) vertraut machen. Dies ist jedoch nur für diejenigen erforderlich, die mit dem Remote des Subtree arbeiten möchten.
* Es ist etwas komplizierter, den Code für die Unterprojekte wieder "upstream" zu bringen.
* Die Verantwortung, den Code von Haupt- und Unterprojekten in den Commits nicht zu vermischen, liegt bei Ihnen.


# Übung macht den Meister

Die Verwendung von Git Subtree lässt sich am besten anhand von Beispielen erklären. Dazu möchte ich einige typische Anwendungsfälle durchgehen, denen man bei der Arbeit mit Git Subtree begegnet.

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

## Anwendungsfall 1: Subtree in einem Projekt hinzufügen

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

## Anwendungsfall 2: Subtree hinzufügen aber die history “squashen”

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

## Anwendungsfall 3: Eine Änderung aus “repo1” in “subtree_test” übernehmen

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

## Anwendungsfall 4: Eine Änderung aus “subtree_test” in “repo1” übernehmen

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