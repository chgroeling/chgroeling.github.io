---
title: "Aktienanalyse leicht gemacht: Google Sheets und die GOOGLEFINANCE-Funktion"
date: 2023-06-17 08:00:10 +0200
categories: Finanzen
---

Seit einiger Zeit interessiere ich mich für das Thema der Aktienanlage. Dabei folge ich weitestgehend Gerd Kommers Buch "Souverän investieren für Einsteiger: Wie Sie mit ETFs ein Vermögen bilden". Im Buch präsentiert der Autor eine sogenannte passive Anlagestrategie, die nur wenig Eingriffe des Anlegers erfordert. Mein Geld lege ich gemäß dieser Strategie an und habe damit bis jetzt eine gute Rendite erzielt.

Trotz dieser gemütlichen Herangehensweise schielt ein Teil von mir immer wieder neugierig auf Internetseiten und Blogs, die "aktive" Anlagestrategien verfolgen und hiermit vermeidlich erfolgreich sind. Daher habe ich mich entschlossen, im Rahmen kleiner Experimente zu starten, um mir selbst das Thema aktive Anlage vertraut zu machen.

Als Ingenieur bin ich ein Freund von Fakten und Zahlen. Deshalb habe ich auf den bekannten Internet-Portalen nach Aktien gesucht, um mich dort rational und nüchtern zu informieren. Was mir dabei sofort ins Auge stach, war die schier endlose Auswahl von Aktienwerten, die sehr bunt präsentiert und auf Basis eher undurchsichtiger Metriken sortiert wurden. Mir ist klar, welchen Einfluss die Psychologie auf eine aktive Anlagestrategie hat. Alles schreit "Kauf mich!" Als Anfänger ist es schwer, dort einen klaren Kopf zu bewahren. Weiterhin bieten die meisten Portale kaum Unterstützung für eigene Berechnungen, etwa, welche Verluste man akzeptieren kann. Man sitzt dann schnell mit dem Taschenrechner vor dem Bildschirm - ziemlich umständlich.

Aus diesem Grund habe ich nach einer Lösung gesucht, die es mir erlaubt, meine Entscheidungen unabhängig von den psychologisch hoch optimierten Internetseiten zu treffen, aber gleichzeitig aktuelle Daten zu einem Aktienwert liefert.

Zu meiner großen Überraschung war die einfachste Lösung für mein Problem die  bekannte Online-Tabellenkalkulation [Google Sheets](https://www.google.com/sheets).

# Kurze Einführung: Google Finance 

Google Sheets verfügt über die Funktion `GOOGLEFINANCE`, mit der man völlig kostenfrei Informationen von [www.google.com/finance](https://www.google.com/finance/) abfragen kann. Das folgende Bild zeigt beispielhaft, wie das so funktioniert:
 
![Google Finance Example](/img/google_finance/Google_Finance_Simple.png)

Der Clou, es handelt sich um Echtzeitdaten (mit ca. 15 Minuten Verzögerung). Das heißt, die Tabelle wird jeweils mit den aktuellsten Werten aktualisiert.

Allgemein lautet der Aufruf wie folgt:

```
GOOGLEFINANCE(ticker; [attribute]; [start_date]; [end_date|num_days]; [interval])`
```

Es gibt einige Quellen im Internet, wie diese Funktion zu verwenden ist. Am besten schaut ihr hierfür direkt auf die offizielle Hilfeseite von Google zu dem Thema. Diese findet ihr [hier](https://support.google.com/docs/answer/3093281?hl=de&sjid=6919203607782479755-EU). Weiterhin gibt es einen guten Artikel auf techjunkies.blog, geschrieben von Patrik Woesner, der die verschiedenen Funktionen genauer beleuchtet. Dieser ist [hier](https://techjunkies.blog/code/googlefinance/) zu finden.

# Ein eigenes Aktiendashboard mit Google Sheets erstellen

Ausgerüstet mit diesen Kenntnissen habe ich beispielhaft die nachstehenden Tabelle erstellt, die als Grundlage für mein persönliches Aktiendashboard dienen soll. Die in der Tabelle genannten Unternehmen sind lediglich Beispiele, halte sie also nicht für einen Anlagetip:

![Google Finance Example](/img/google_finance/Google_Sheets_Finance.png)

In der Tabelle enthalten sind alle verfügbaren Informationen, die die `GOOGLEFINANCE`-Funktion für einen Aktienwert liefert. Man kann sofort wichtige Kennzahlen erkennen und entsprechend reagieren. Besonders hervorzuheben sind folgende Werte, die gerade am Anfang sehr hilfreich sein können:

| __Attribut:__ | __Erklärung__:
| :---- | :----- 
|__price__ | Der aktuelle Preis der Aktie. | 
|__changepct__ | Die prozentuale Veränderung des Aktienpreises im Vergleich zum vorherigen Handelstag. 
| __high__ | Der höchste Preis, den die Aktie im Laufe des aktuellen Handelstages erreicht hat. 
| __low__ | Der niedrigste Preis, den die Aktie im Laufe des aktuellen Handelstages erreicht hat. | 


__Warnung__: Nicht alle Informationen sind für jeden Aktienwert an jeder Börse verfügbar. Zum Beispiel liefert die Abfrage des Beta-Wertes für alle oben gezeigten Aktienwerte kein Ergebnis.

Es ist schon toll, welche Möglichkeiten sich durch diese Funktion bieten. Ich verbinde damit folgende Vorteile:
- Der Zugriff auf die Aktieninformationen ist völlig kostenlos.
- Die Verzögerung ist, je nach Börse, in einem akzeptablen Rahmen.
- Man hat freie Hand bei der Darstellung.
- Man kann mit allen Werten eigene Berechnungen anstellen.

Es gibt allerdings auch Nachteile:
- Die Lösung ist auf Google Sheets beschränkt. Es gibt keine Möglichkeit, die Daten aus Google Sheets zu exportieren oder abzufragen. Google begrenzt hier den Zugriff ganz bewusst.
- Im deutschen Anlageraum sind nur Aktienwerte verfügbar, die an der Frankfurter Börse sowie Xetra gehandelt werden.

Die vorgestellten Möglichkeiten bieten einen hervorragenden Überblick über die aktuelle Lage der Aktienkurse. Es gibt jedoch noch einen weiteren, oft übersehenen Aspekt, der für eine tiefere Analyse von Aktien unerlässlich ist: historische Kursdaten. Diese Daten repräsentieren die Entwicklung der Aktienpreise über einen bestimmten Zeitraum hinweg und liefern wertvolle Informationen für Anlageentscheidungen.

# Historische Kursdaten

Historische Kursdaten helfen, Muster zu erkennen, Strategien zu testen, Risiken zu bewerten, Prognosen zu treffen und den wahren Wert von Finanzinstrumenten zu bestimmen.  

Auch hier  lässt `GOOGLEFINANCE` einem nicht im Stich. Durch die Angabe eines Zeitraums
von `[start_date]` bis  `[end_date]` sowie dem  `[interval]` im oben dargestellten Funktionsaufruf können Kursdaten für einen beachtlichen langen Zeitraum heruntergeladen werden.

Die Möglichkeit, die dies bietet, habe ich im folgenden Bild exemplarisch dargestellt:

![Google Finance Example](/img/googlefinance_hist.png)

Wie abgebildet ist der Kursverlauf der "Mercedes-Benz-Group" über mehr als ein Jahrzehnt verfügbar. Gleiches gilt für viele andere Aktienwerte.

# Ausblick

> "Man versteht nur wirklich, was man erschaffen kann" - Giambattista Vico

Ich werde nun für meinen Teil ein Dashboard in Google Sheets erstellen, das nach Branchen aufgeteilt ist, geeignete Aktien auswählen und damit als Erstes meine Anlagekompetenz testen.

Dazu versuche ich, den Aktienmarkt mittels Kurshistorie in vergangenen Zeiträumen zu betrachten und meine Entscheidungsfähigkeit auf Basis dieser Daten zu prüfen. So erstelle ich gewissermaßen ein Entscheidungsspiel für mich.

Außerdem kann ich mir auf diese Weise gleich ein paar der viel gerühmten Metriken anschauen und testen, denen auf Aktienportalen so viel Wert beigemessen wird.

Mal sehen, wie erfolgreich ich bin oder ob ich das aktive Anlegen lieber gleich wieder sein lassen sollte 🙂

Was würdest du mit einer solchen Möglichkeit anfangen? Schreib mir ruhig.

# Zusammenfassung

In meinem Bestreben, aktives Investieren zu erforschen, fand ich Google Sheets mit seiner `GOOGLEFINANCE`-Funktion äußerst hilfreich. Mit diesem Tool kann ich auf Echtzeit- und historische Aktieninformationen zugreifen und diese nach meinen Vorstellungen darstellen und analysieren. Trotz einiger Einschränkungen, wie der Exklusivität auf Google Sheets und der Begrenzung auf bestimmte Aktienwerte, bietet `GOOGLEFINANCE` ein praktisches, kostenloses Instrument für meine experimentelle Reise in das aktive Investieren.
