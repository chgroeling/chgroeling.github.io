---
layout: single
toc: false
title:  "Git Subtree in Aktion: Praktische Szenarien und Anleitungen für Entwickler"
date:   2023-05-05 06:00:10 +0200
categories: Git
comments: true
---

# Motivation
Das Internet ist voll von Artikeln, die erklären, warum man keine Git-Submodule verwenden sollte. Aber welche Alternativen gibt es? Eine Möglichkeit ist Git-Subtree, das ich in diesem Artikel gerne näher betrachten möchte.

Git-Subtree bindet den Inhalt eines externen Repositories direkt in das Hauptrepository ein, ähnlich wie einen einfachen Ordner. Dabei bleibt die Historie des externen Repositories erhalten und es ermöglicht den Austausch von Änderungen zwischen beiden Repositories. Auf diese Weise können Entwickler Änderungen im externen Repository vornehmen und sie problemlos in das Hauptrepository integrieren oder umgekehrt.

# Warum kann die Verwendung von Git Submodulen eine schlechte Idee sein?
Bevor wir uns Git-Subtree genauer anschauen, hier noch einmal die Gründe, warum die Verwendung von Git-Submodulen bei einigen Entwicklern unbeliebt ist:

* Git klont keine Submodule. Man muss entweder `git submodule update` oder `git clone --recursive` verwenden.
* Git-Submodule werden nicht automatisch synchronisiert, man muss `git submodule update` durchführen.
* Git diff gibt auch nach einem Commit etwas zurück, wenn das Submodul eine Änderung hat. Man muss im Submodul ebenfalls commiten/pushen oder die Änderung rückgängig machen. Im Grunde muss man jede Operation, die man im Hauptmodul durchführt, auch im Submodul durchführen, wenn Dateien in beiden geändert wurden.
* Das Beheben von Merge-Konflikten in einem Repo ist schon schwer genug. Wenn man Änderungen am Haupt-Repo und einem Submodul-Repo hat, wird es zur Qual.
* Der Vergleich von Dateirevisionen wird fast unmöglich, wenn man Dateien/Verzeichnisse vom Hauptprojekt in das Submodul-Repo verschiebt.

Zusammenfassend benötigt das gesamte Team einiges an Wissen, um mit Submodulen umzugehen. Dies erzeugt Reibung während der Entwicklung, die von der eigentlichen Entwicklungsarbeit ablenkt.

# Welche Vor- und Nachteile hat die Verwendung von Git-Subtree

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

## Vorbereitung: Erstelle zwei lokale Test-Repos

Zuerst benötigen wir ein Repository, das als Subtree in einem Projekt eingebunden werden soll. Zu Testzwecken erstelle ich im Folgenden ein lokales Git-Repository mit drei Commits.

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

Als Nächstes benötige ich ein Git-Repository, in das das andere Repository als Subtree eingebunden werden soll.

```bash
mkdir subtree_test
cd subtree_test
git init
touch README.md
git add --all
git commit --message "Initial commit"
cd ..
```

Nun, da wir beide Repositories erstellt haben, können wir fortfahren und das Subtree-Konzept in der Praxis anwenden. 

## Anwendungsfall 1: Subtree in einem Projekt hinzufügen

Zunächst füge ich das Repository "repo1" als Subtree zu meinem Repository "subtree_test" hinzu.

```bash
cd subtree_test
git subtree add --prefix my_repo1 ../repo1 master
```

Die Kommandozeilenoption "--prefix" gibt das Verzeichnis an, in dem der Subtree erstellt werden soll. In unserem Fall ist dies "my_repo1". Danach folgt die Upstream-Referenz, im Beispiel unsere lokale Repository. Im allgemeinen Fall handelt es sich dabei meist um einen Link zu einem Git-Server.

Nach Ausführung des Befehls wird der Inhalt der Repository "repo1" im Verzeichnis "./my_repo1" angelegt.

Der git commit graph (``git log --graph --oneline``) der Repository "subtree_test" sieht danach wie folgt aus:


```
*   0745df0 (HEAD -> master) Add 'my_repo1/' from commit '4499ee8ec47f747f2beb30512db9202d6a76f650'
|\
| * 4499ee8 Second change
| * 99564a0 First change
| * 33c5799 Initial commit
* 5beb502 Initial commit
```


Beachte Folgendes: Es gibt nun zwei Root-Commits – einen für das Repository "subtree_test" und einen weiteren für "repo1". Zudem wird durch das Hinzufügen eines Subtrees die gesamte Historie des hinzugefügten Repositories in das Hauptrepository integriert, was eine saubere und nachvollziehbare Verbindung zwischen beiden Repositories ermöglicht. Die Metadaten, wie zum Beispiel der Ursprung des hinzugefügten Subtrees und die beteiligten Commits, werden in der Commit-Nachricht gespeichert, um einen besseren Überblick und eine leichtere Nachverfolgung der Änderungen zu gewährleisten.

## Anwendungsfall 2: Subtree hinzufügen, aber die history “squashen”

Setzen wir den master wieder auf den “Initial commit” zurück.

```bash
git reset --hard HEAD~1
```

Und verwenden nun die Kommandozeilenoption ``--squash``:

```bash
git subtree add --prefix my_repo1 ../repo1 master --squash
```


Der Git-Commit-Graph (``git log --graph --oneline``) der Repository "subtree_test" sieht nun wie folgt aus:

```
*   cdea121 (HEAD -> master) Merge commit '7d5bc4a1dcafca9be8bcd161fa0f038655001695' as 'my_repo1'
|\
| * 7d5bc4a Squashed 'my_repo1/' content from commit 4499ee8
* 5beb502 Initial commit
```

Die "repo1" ist immer noch Teil des Repos "subtree_test". Nun wurde jedoch die Historie von "repo1" zuvor zusammengefasst. Im Git-Jargon heißt das, ein "Squash" wurde durchgeführt. Das ist unter anderem dann nützlich, wenn man die Historie von "repo1" nicht in seinem Hauptprojekt benötigt.

## Anwendungsfall 3: Eine Änderung aus “repo1” in “subtree_test” übernehmen (pull)

Wechseln wir nun in "repo1" und fügen eine weitere Änderung hinzu:

```bash
cd ..
cd repo1 
echo "Change 3" >> file1.txt
git add --all
git commit --message "Third change"
cd ..
```

Nun versuchen wir, diese Änderung in unser Hauptrepo "subtree_test" zu übernehmen:

```bash
cd subtree_test
git subtree pull --prefix my_repo1 ../repo1 master --squash
```

Hier zeigt sich eine Schwäche von Git Subtree: Es wird kein Link zu "repo1" im Git-Repository gespeichert. Wenn wir mit einem Remote arbeiten wollen, müssen wir den Link zu diesem Remote immer wieder angeben.

{: .notice--warning} 
Es gibt einen Bug, der ein "subtree pull" ohne "--squash" verhindert, wenn „—-squash““ zuvor verwendet wurde. Dieser Bug wurde meines Erachtens in Git 2.40.x behoben. Meist ist dies aber irrelevant da ohnehin in der Regel mit „—-squash“ gearbeitet wird.

Der Git-Commit-Graph  (``git log --graph --oneline``) der Repository "subtree_test" sieht nun wie folgt aus.

```
*   19d5c22 (HEAD -> master) Merge commit '702d51b28ba5daa1b1d99431a071ea5d4df81b6b'
|\
| * 702d51b Squashed 'my_repo1/' changes from 4499ee8..50d8d46
* | cdea121 Merge commit '7d5bc4a1dcafca9be8bcd161fa0f038655001695' as 'my_repo1'
|\|
| * 7d5bc4a Squashed 'my_repo1/' content from commit 4499ee8
* 5beb502 Initial commit
```

Wie man sehen kann, wurden die Änderungen erneut in einem eigenen "Squash"-Commit zusammengefasst. Anschließend wurden sie in einem "Merge"-Commit in den Master-Branch integriert. 

## Anwendungsfall 4: Eine Änderung aus “subtree_test” in “repo1” übernehmen (push)

Da bei Git standardmäßig nicht auf einen ausgecheckten Branch gepusht werden darf, muss unser "repo1" zunächst auf einen anderen Branch gesetzt werden. Dies erfolgt wie folgt:

```bash
cd ..
cd repo1
git checkout -b test_branch
cd ..
```

Dieser Schritt ist nicht notwendig, wenn mit lokalen "remotes" gearbeitet wird.

Nun kann man seine Änderungen wie folgt in "repo1" übernehmen:

```bash
cd subtree_test
cd my_repo1
echo "Change 4" >> file1.txt
git add --all
git commit --message "Change in subtree_test"
cd..

git subtree push --prefix my_repo1 ../repo1 master
```

Das Kommando führt nun einen "Split" durch. Dabei werden die Änderungen, die spezifisch für das Subtree sind, von den Änderungen im Hauptrepository getrennt. Anschließend werden die Änderungen aus dem Subtree in das "repo1" Repository gepusht. Dieser Prozess stellt sicher, dass nur die relevanten Änderungen im Subtree in das externe Repository übertragen werden und die Historie beider Projekte sauber bleibt.

Schaut man nun auf den Commit-Graph (git log --graph --oneline master) von "repo1", erhält man folgendes Ergebnis:
```
8313d6a (master) Change in subtree_test
* 50d8d46 (HEAD -> test_branch) Third change
* 4499ee8 Second change
* 99564a0 First change
* 33c5799 Initial commit
```

Die in "subtree_test" vorgenommene Änderung wurden isoliert und danach in "repo1" übertragen.

# Fazit
Git Subtree stellt eine leistungsstarke Alternative zu Git-Submodulen dar, die es Entwicklern ermöglicht, externe Repositories direkt in das Hauptrepository einzubinden und auf diese Weise Änderungen problemlos zwischen beiden Repositories auszutauschen. Im Vergleich zu Git-Submodulen bietet Git Subtree einige Vorteile wie eine einfachere Handhabung und weniger zusätzliche Metadaten. Es gibt jedoch auch Nachteile, wie die Notwendigkeit, sich mit einer neuen Merge-Strategie vertraut zu machen und die Verantwortung, den Code von Haupt- und Unterprojekten in den Commits nicht zu vermischen. Die im Artikel vorgestellten Anwendungsfälle dienen als praktische Anleitungen für den Umgang mit Git Subtree und zeigen, wie man das Konzept effizient in realen Entwicklungsprojekten einsetzen kann.

# Bibliografie

- [git-subtree/git-subtree.txt at master · apenwarr/git-subtree (github.com)](https://github.com/apenwarr/git-subtree/blob/master/git-subtree.txt)
-  Paolucci, Nicola. [Git subtree: the alternative to Git submodule ](https://www.atlassian.com/git/tutorials/git-subtree)
- Paolucci, Nicola. [The power of Git subtree](https://blog.developer.atlassian.com/the-power-of-git-subtree/?_ga=2-71978451-1385799339-1568044055-1068396449-1567112770)
- [032 Introduction to Git Subtrees](https://www.youtube.com/watch?v=sC1sfoCo5qY)
