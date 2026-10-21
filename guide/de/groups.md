# Consumer Groups

Eine Consumer Group ist eine Menge von Consumern, die sich das Lesen von Topics teilen. Kafka weist jede Partition einem Mitglied der Gruppe zu und speichert für jede Partition den Offset, den die Gruppe committet hat. Anwendungen lesen nach einem Neustart ab diesem Offset weiter.

Öffnen Sie **Consumer Groups** unter der Verbindung im Bereich **Verbindungen**. Die Gruppen eines einzelnen Topics finden Sie auch im Tab **Consumer Groups** des Topics. Siehe [Topic-Bildschirm](topics.md#topic-details).

<a id="group-list"></a>

## Gruppenliste

Die Liste zeigt alle Consumer Groups des Clusters. Die Spalten:

- **Gruppe** — die Gruppen-ID (`group.id`).
- **Status** — der von Kafka gemeldete Status, zum Beispiel `Stable` (Mitglieder lesen), `Empty` (keine Mitglieder), `PreparingRebalance` oder `CompletingRebalance` (Partitionen werden neu verteilt) und `Dead` (die Gruppe existiert nicht mehr).
- **Mitglieder** — die Anzahl der aktiven Mitglieder.
- `Assignor` — die Strategie, nach der Partitionen verteilt werden, zum Beispiel `range`.
- **Lag** — der gesamte Lag der Gruppe. Siehe [Lag](groups.md#lag).
- **Topics** — die Topics, für die die Gruppe Offsets committet hat.

Geben Sie unter **Gruppen suchen** Text ein, um die Gruppen anzuzeigen, deren ID diesen Text enthält; Groß- und Kleinschreibung spielen keine Rolle. Die Statusliste daneben zeigt nur Gruppen in einem bestimmten Status; **Alle Status** hebt diesen Filter auf. Der Zähler zeigt, wie viele Gruppen angezeigt werden und wie viele es insgesamt gibt.

Mitglieder, Assignor und Lag werden beim Scrollen nur für die sichtbaren Zeilen geladen. Bis sie da sind, zeigt die Zelle `…`. Lassen sie sich nicht laden, zeigt die Zelle `—`; zeigen Sie mit der Maus darauf, um den Grund zu sehen.

**Aktualisieren** lädt die Liste und ihre Daten neu. Haben einige Broker nicht geantwortet, weist eine Warnung darauf hin, dass die Liste unvollständig sein kann.

<a id="lag"></a>

## Lag

Der Lag gibt an, wie viele Nachrichten die Gruppe noch nicht gelesen hat. Für jede Partition ist er das Ende des Logs minus den committeten Offset. Der Lag wird nie unter null angezeigt.

Der Lag hängt nur von committeten Offsets ab. Ein aktiver Consumer hat möglicherweise schon weiter gelesen, als er committet hat, sein tatsächlicher Lag kann also kleiner sein. Sobald die Gruppe erneut committet, sinkt der Lag.

Eine Partition ohne committeten Offset zeigt **kein Commit** und zählt nicht zur Summe. Der gesamte Lag einer Gruppe zeigt `—`, wenn die Gruppe überhaupt keine Commits hat (**keine Commits**) oder wenn die Offsets einer Partition nicht gelesen werden konnten (**Lag unbekannt: Fehler bei den Partitions-Offsets**).

Der Lag zählt Offsets, nicht Nachrichten. In Topics mit Compaction oder Transaktionen enthalten manche Offsets keine sichtbaren Nachrichten, der Gruppe bleiben also möglicherweise weniger Nachrichten zu lesen, als der Lag anzeigt.

Die Werte sind eine Momentaufnahme zum Zeitpunkt des Ladens. Um den Lag zu verfolgen, öffnen Sie die Gruppe und aktivieren die automatische Aktualisierung.

<a id="group-details"></a>

## Gruppendetails

Klicken Sie in der Liste auf eine Gruppe, um ihren Tab zu öffnen. Oben zeigt eine Zusammenfassung Status, Anzahl der Mitglieder und Assignor.

- **Auto-Aktualisierung** — **Auto-Aktualisierung: aus**, **alle 5 s** oder **alle 10 s**.
- **Aktualisieren** — sofort neu laden.
- **Als Mitglied lesen** — eine [Consumer-Session](consumer-sessions.md) in dieser Gruppe starten. Das Session-Formular übernimmt die Gruppen-ID und die Topics, für die die Gruppe Commits hat. Bei Verbindungen mit aktivierter Option **Schreibgeschützt** ist die Schaltfläche deaktiviert; zeigen Sie mit der Maus darauf, um den Grund zu sehen.

**Partitionen** listet die Partitionen auf, für die die Gruppe Commits hat, sowie die ihren Mitgliedern zugewiesenen Partitionen. Der gesamte Lag der Gruppe steht über der Tabelle. Die Spalten sind **Topic**, **Partition**, `Committed` (der committete Offset), `Log end` (der End-Offset der Partition), **Lag** und **Mitglied** (die Client-ID des Mitglieds, das die Partition liest).

**Mitglieder** listet die aktiven Mitglieder auf: `Client ID`, **Host**, `Member ID` und **Zuweisung** (die Partitionen des Mitglieds als `topic:partition`). Eine Gruppe ohne Mitglieder zeigt **Keine aktiven Mitglieder.**

Die zweite Zeile enthält **Offsets zurücksetzen** und **Gruppe löschen**. Bei schreibgeschützten Verbindungen ist sie ausgeblendet.

<a id="reset-offsets"></a>

## Offsets zurücksetzen

Beim Zurücksetzen werden die committeten Offsets einer Gruppe verschoben. Die Gruppe liest dann Nachrichten erneut oder überspringt sie. Offsets lassen sich nur für eine Gruppe ohne aktive Mitglieder zurücksetzen (Status `Empty` oder `Dead`). Andernfalls würden die Mitglieder die neuen Offsets mit ihren eigenen Commits überschreiben. Beenden Sie zuerst die Anwendungen der Gruppe.

1. Klicken Sie im Tab der Gruppe auf **Offsets zurücksetzen**. Solange die Gruppe Mitglieder hat, ist die Schaltfläche deaktiviert; zeigen Sie mit der Maus darauf, um den Grund zu sehen.
2. Wählen Sie die **Strategie**. Siehe [Strategien zum Zurücksetzen](groups.md#reset-strategies).
3. Wählen Sie den **Umfang**. Siehe [Umfang des Zurücksetzens](groups.md#reset-scope).
4. Klicken Sie auf **Vorschau**.
5. Prüfen Sie die Tabelle und klicken Sie auf die Schaltfläche zum Anwenden. Sie zeigt die Anzahl der Partitionen, die sich ändern.

Die Vorschau ändert nichts. Für jede Partition zeigt sie Topic, Partition, **Aktuell** (den committeten Offset), **Neu** (den Ziel-Offset) und `Low / High` (den ersten und den End-Offset der Partition). Eine Partition, für die sich kein Ziel berechnen lässt, ist mit **Kein Ziel-Offset — wird nicht angewendet** und dem Grund gekennzeichnet. Solche Partitionen werden übersprungen.

Angewendet wird nur der in der Vorschau gezeigte Plan. Jede Änderung im Formular blendet die Vorschau aus, und Sie müssen erneut auf **Vorschau** klicken. Bei einer Prod-Verbindung verlangt das Anwenden den Verbindungsnamen; bei anderen Verbindungen wird sofort angewendet.

Vor dem Anwenden prüft die App erneut, ob die Gruppe keine Mitglieder hat. Nach dem Anwenden listet **Ergebnis** jede Partition mit ihrem neuen Offset oder einem Fehler auf.

Sie können ein Zurücksetzen auch von einer Nachricht aus vorbereiten: siehe [Gruppe auf eine Nachricht zurücksetzen](browse.md#reset-group-here). **Schließen** schließt den Bereich zum Zurücksetzen.

<a id="reset-strategies"></a>

## Strategien zum Zurücksetzen

**Strategie** legt den Ziel-Offset für jede Partition im Umfang fest:

- **Vom Anfang (earliest)** — der erste noch gespeicherte Offset. Die Gruppe liest alles erneut, was das Topic vorhält.
- **Vom Ende (latest)** — das Ende der Partition. Die Gruppe überspringt alles Angesammelte und liest nur neue Nachrichten.
- **Nach Zeit** — die erste Nachricht, die zum Zeitpunkt unter **Datum und Uhrzeit (lokal)** oder danach geschrieben wurde. In Partitionen ohne neuere Nachrichten ist das Ziel das Ende.
- **Bestimmter Offset** — der angegebene **Offset**, eine ganze Zahl ab 0, in jeder Partition. Hat eine Partition diesen Offset nicht, ist das Ziel ihr erster bzw. ihr End-Offset.
- **Um ±N verschieben** — verschiebt den committeten Offset um den Wert unter **Verschiebung (±N)**. Eine negative Zahl liest die letzten N Nachrichten erneut, eine positive überspringt N Nachrichten. Das Ergebnis bleibt zwischen erstem und End-Offset. Partitionen ohne committeten Offset lassen sich nicht verschieben und werden übersprungen.

Die Zeit ist Ihre Ortszeit, mit Sekunden. Kafka ermittelt den Offset anhand der Zeitstempel der Nachrichten.

<a id="reset-scope"></a>

## Umfang des Zurücksetzens

**Umfang** legt fest, welche Partitionen zurückgesetzt werden. Die übrigen Partitionen behalten ihre Offsets.

- **Ganze Gruppe** — alle Partitionen, für die die Gruppe Offsets committet hat. Hat die Gruppe noch keine Commits, wählen Sie stattdessen Topics.
- **Topics** — alle Partitionen der aufgeführten Topics, auch Partitionen ohne Commits. Trennen Sie die Namen durch Kommas, Leerzeichen oder Zeilenumbrüche. So können Sie eine Gruppe ein neues Topic ab einer gewählten Stelle lesen lassen.
- **Partitionen** — nur die aufgeführten Partitionen. Geben Sie sie unter **Partitionen (Topic:Partition)** als `topic:partition` ein, getrennt durch Kommas oder Leerzeichen, zum Beispiel `orders:0, orders:3`.

<a id="delete-group"></a>

## Gruppe löschen

Das Löschen einer Gruppe entfernt alle ihre committeten Offsets. Nur eine Gruppe ohne Mitglieder im Status `Empty` lässt sich löschen.

1. Klicken Sie im Tab der Gruppe auf **Gruppe löschen**. Solange die Gruppe Mitglieder hat, ist die Schaltfläche deaktiviert; zeigen Sie mit der Maus darauf, um den Grund zu sehen.
2. Geben Sie zur Bestätigung die Gruppen-ID ein. Bei einer Prod-Verbindung geben Sie zusätzlich den Verbindungsnamen ein.
3. Klicken Sie auf **Löschen**.

Nach dem Löschen schließt sich der Tab der Gruppe. Startet eine Anwendung mit dieser `group.id` erneut, legt Kafka die Gruppe neu an. Ohne committete Offsets beginnt die Anwendung dort, wo ihre Einstellung `auto.offset.reset` es vorgibt: am Anfang oder am Ende.

Bei Verbindungen mit aktivierter Option **Schreibgeschützt** ist die Schaltfläche ausgeblendet.
