# Nachrichten durchsuchen

Nachrichten durchsuchen Sie im Tab **Nachrichten** eines Topics. Dabei tritt die App keiner Consumer Group bei und committet nichts, der Cluster bleibt also unverändert. Das funktioniert auch bei Verbindungen mit aktivierter Option **Schreibgeschützt**. Um als Mitglied einer Gruppe zu lesen, verwenden Sie eine [Consumer-Session](consumer-sessions.md).

<a id="start-reading"></a>

## Lesen starten

1. Öffnen Sie ein Topic aus der Liste **Topics**. Zuerst öffnet sich der Tab **Nachrichten**.
2. Wählen Sie unter **Ab**, wo das Lesen beginnen soll. Siehe [Startposition](browse.md#start-position).
3. Legen Sie optional Formate und Filter fest.
4. Klicken Sie auf **Lesen**.

Die Nachrichten erscheinen in der Tabelle, sobald sie eintreffen. Die Zeile neben den Schaltflächen zeigt, wie viele Nachrichten durchsucht wurden, wie viele zu den Filtern passen und wie viele in der Tabelle stehen. Außerdem zeigt sie den Status: **wird gelesen…**, **Limit erreicht**, **Ende des Topics erreicht** oder **gestoppt**.

Ohne `Live tail` endet das Lesen am Ende des Topics, wie es beim Klick auf **Lesen** war, oder früher, wenn das **Limit** erreicht ist. Das Limit zählt Nachrichten, die zu den Filtern passen. Es kann zwischen 1 und 100000 liegen; große Werte verlangsamen das Lesen.

Mit aktiviertem `Live tail` endet das Lesen nicht am Ende: Neue Nachrichten werden hinzugefügt, sobald sie geschrieben werden. Sind alle alten Nachrichten gelesen, zeigt der Status **Warten auf neue Nachrichten…**. Die Tabelle behält die letzten Nachrichten bis zum Limit und verwirft die ältesten.

**Stoppen** beendet das Lesen und behält die Tabelle. Während gelesen wird, wird aus der Schaltfläche **Lesen** die Schaltfläche **Neu starten**: Sie beginnt mit den aktuellen Einstellungen von vorn und leert die Tabelle. Bricht die Verbindung zum Broker ab, verbindet sich die App selbst neu und zeigt das in der Statuszeile an.

Nachrichten aus verschiedenen Partitionen stehen in der Tabelle gemischt. Kafka garantiert die Reihenfolge nur innerhalb einer Partition. Die Spalte `P` zeigt die Partition, `Offset` die Position darin. Wählen Sie `UTC`, um Zeiten in UTC statt in Ortszeit anzuzeigen.

<a id="start-position"></a>

## Startposition

**Startposition** legt fest, wo das Lesen in jeder Partition beginnt:

- **Neueste N** — die letzten N Nachrichten. N wird gleichmäßig auf die Partitionen verteilt: Bei 100 Nachrichten und 4 Partitionen nimmt die App bis zu 25 aus jeder.
- **Vom Anfang** — ab der ältesten noch gespeicherten Nachricht.
- **Ab Offset** — ab demselben Offset in jeder Partition. Hat eine Partition keinen solchen Offset, beginnt das Lesen an ihrem Anfang oder Ende. Dieser Modus ist vor allem bei einer einzigen Partition nützlich.
- **Ab Zeitpunkt** — ab der ersten Nachricht mit einem Zeitstempel zur angegebenen Ortszeit oder danach. In Partitionen ohne neuere Nachrichten beginnt das Lesen am Ende.
- **Letzte X Minuten** — dasselbe, ab jetzt zurückgerechnet.

Das Feld mit **Partitionen: alle** beschränkt das Lesen auf bestimmte Partitionen. Geben Sie die Partitionsnummern durch Kommas getrennt ein, zum Beispiel `0,2`. Lassen Sie das Feld leer, um alle Partitionen zu lesen.

<a id="filters"></a>

## Filter

Filter zeigen nur passende Nachrichten. Die App liest trotzdem alle Nachrichten ab der Startposition und prüft jede, Filter beschleunigen das Lesen also nicht. Alle ausgefüllten Felder müssen zutreffen.

- **Schlüssel enthält** und **Wert enthält** — Text im Schlüssel oder im Wert.
- `regex` — diese Felder und den Header-Wert als reguläre Ausdrücke behandeln.
- `Aa` — Groß- und Kleinschreibung beachten. Standardmäßig spielt sie keine Rolle.
- **Header** — die Nachricht hat einen Header mit genau diesem Namen. **Header-Wert** prüft zusätzlich seinen Wert.
- `JSONPath` — eine Bedingung für einen JSON-Wert, zum Beispiel `$.order.status == "FAILED"`. Ohne `==` oder `!=` muss der Pfad nur existieren, zum Beispiel `$.error`. Nachrichten, deren Wert kein gültiges JSON ist, passen nicht.
- **Zeit von** und **Zeit bis** — der Zeitstempel der Nachricht liegt in diesem Bereich, in Ortszeit. Beide Grenzen sind eingeschlossen.

Filter prüfen den in der Tabelle angezeigten Text nach dem Dekodieren. Ist das Wertformat `hex`, durchsucht der Wertfilter den Hex-Text. Filter gelten beim Klick auf **Lesen**; lesen Sie nach einer Änderung erneut.

Ein Filter verschiebt die Startposition nicht. **Neueste N** mit Filter findet Treffer beispielsweise nur unter den letzten N Nachrichten. Um das ganze Topic zu durchsuchen, beginnen Sie mit **Vom Anfang**.

<a id="formats"></a>

## Formate

**Format** legt fest, wie Schlüssel und Werte dekodiert werden, getrennt für **Schlüssel** und **Wert**. Nur die Anzeige ändert sich, nicht die Daten.

- `auto` — der Standard. Gültige JSON-Objekte und -Arrays werden als JSON angezeigt, anderer UTF-8-Text als Zeichenkette, Binärdaten als Hex.
- `string` — Text in UTF-8. Bytes, die kein gültiges UTF-8 sind, werden durch `�` ersetzt.
- `json` — JSON mit Einrückungen in den Nachrichtendetails. Ist der Wert kein gültiges JSON, zeigen die Details den Fehler.
- `hex` — jedes Byte als zwei Hex-Ziffern.
- `base64` — die Bytes in Base64.

Die Details zeigen, welches Format tatsächlich verwendet wurde, zum Beispiel `json`, wenn `auto` JSON erkannt hat. Header-Werte werden als Text angezeigt; binäre Header-Werte als `0x`, gefolgt von Hex. Ein fehlender Schlüssel oder Wert wird als `null` angezeigt.

Sehr große Schlüssel und Werte werden nur teilweise angezeigt, etwa das erste 1 MB Text. Das gilt auch für `hex` und `base64`. Die Nachrichtendetails zeigen dann eine Warnung.

<a id="message-details"></a>

## Nachrichtendetails

Klicken Sie auf eine Zeile, um die Details der Nachricht neben der Tabelle zu öffnen. Oben stehen Partition, Offset und Zeit. Darunter:

- **Wert** und **Schlüssel** — der vollständige Text mit Format und Größe in Bytes. JSON wird mit Einrückungen angezeigt. **Kopieren** kopiert den Text.
- **Header** — eine Tabelle mit Namen und Werten der Header.

Die Tabellenspalten sind `P`, `Offset`, **Zeit**, **Schlüssel**, **Wert**, **Header** (ihre Anzahl) und **Größe** (Größe von Schlüssel und Wert in Bytes). Die Zeit ist der Zeitstempel der Nachricht. Je nach Topic-Einstellung `message.timestamp.type` setzt ihn der Producer oder der Broker.

<a id="export-messages"></a>

## Nachrichten kopieren

Glafira speichert durchsuchte Nachrichten nicht in einer Datei. Um eine Nachricht aus der App zu übernehmen, kopieren Sie sie:

- **Alles als JSON kopieren** kopiert die ganze Nachricht als JSON: Topic, Partition, Offset, Zeitstempel, Schlüssel, Wert, Header und Größe. Schlüssel und Wert werden dekodiert und mit ihrem Format übernommen.
- **Kopieren** neben **Wert** oder **Schlüssel** kopiert nur diesen Text.

Der kopierte Text verwendet das zum Lesen gewählte Format. Um Binärdaten exakt zu kopieren, lesen Sie sie als `hex` oder `base64`. Sehr große Schlüssel und Werte werden wie angezeigt nur teilweise kopiert, auch in `hex` und `base64`. Siehe [Formate](browse.md#formats).

<a id="open-in-producer"></a>

## Im Producer öffnen

**Erneut senden** öffnet die Nachricht im **Producer** derselben Verbindung. Das Formular übernimmt Topic, Schlüssel, Wert, Header und Formate der Nachricht. Die Partition wählt Kafka. Platzhalter im Nachrichtentext werden nicht ersetzt: Die Kopie wird so gesendet, wie sie gelesen wurde.

Es wird nicht sofort gesendet. Prüfen Sie das Formular, ändern Sie es bei Bedarf und senden Sie es aus dem Producer. Siehe [Producer](producer.md).

Die Schaltfläche ist in zwei Fällen deaktiviert, der Grund wird angezeigt:

- der Schlüssel oder Wert wurde nur teilweise angezeigt, weil er zu groß ist;
- der Text wurde verlustbehaftet dekodiert, weil er kein gültiges UTF-8 ist. Lesen Sie ihn als `hex` oder `base64`, um eine exakte Kopie zu senden.

Bei Verbindungen mit aktivierter Option **Schreibgeschützt** ist die Schaltfläche ausgeblendet.

<a id="reset-group-here"></a>

## Gruppe auf eine Nachricht zurücksetzen

**Gruppe hierher zurücksetzen…** bereitet ein Zurücksetzen der Offsets vor, sodass eine Consumer Group ab dieser Nachricht liest.

1. Klicken Sie in den Nachrichtendetails auf **Gruppe hierher zurücksetzen…**.
2. Wählen Sie eine Gruppe aus der Liste. Es werden nur vorhandene Gruppen aufgeführt.
3. Klicken Sie auf **Zum Zurücksetzen**.

Der Tab der Gruppe öffnet sich mit ausgefülltem Formular zum Zurücksetzen. Als **Strategie** ist **Bestimmter Offset** mit dem Offset der Nachricht gesetzt, als **Umfang** **Partitionen** mit Topic und Partition der Nachricht. Noch ändert sich nichts: Klicken Sie auf **Vorschau**, um die neuen Offsets zu sehen, und wenden Sie das Zurücksetzen dort an. Die Gruppe darf keine aktiven Mitglieder haben. Siehe [Consumer Groups](groups.md).

Bei Verbindungen mit aktivierter Option **Schreibgeschützt** ist die Schaltfläche ausgeblendet.
