# Producer

Der **Producer** sendet Nachrichten an Kafka: einzeln aus einem Formular oder viele auf einmal. Öffnen Sie ihn über die Verbindung im Bereich **Verbindungen** oder mit **Nachricht senden** im Tab eines Topics; dann ist das Topic bereits eingetragen. Jede Verbindung hat einen eigenen Producer-Tab.

Bei Verbindungen mit aktivierter Option **Schreibgeschützt** öffnet sich der Producer mit der Warnung **Die Verbindung ist schreibgeschützt — Senden nicht verfügbar.** Die Schaltflächen zum Senden sind ausgeblendet, Vorlagen und Verlauf lassen sich aber weiterhin ansehen.

<a id="send-message"></a>

## Nachricht senden

1. Geben Sie das **Topic** ein. Das Feld schlägt die Topics des Clusters vor, ohne interne Topics.
2. Geben Sie optional eine Nummer unter **Partition** ein. Zeigt das Feld **auto**, wird die Partition anhand des Schlüssels gewählt: Nachrichten mit demselben Schlüssel landen in derselben Partition, wie beim Java-Client. Nachrichten ohne Schlüssel landen in einer zufälligen Partition.
3. Füllen Sie **Schlüssel** und **Wert** aus und wählen Sie deren Formate. Siehe [Formate für Schlüssel und Wert](producer.md#key-value-formats).
4. Fügen Sie optional Header hinzu.
5. Klicken Sie auf **Senden**.

Nach erfolgreichem Senden zeigt eine Zeile unter dem Formular Partition, Offset und Zeit der Nachricht. Ist `acks` auf `0` gesetzt, liefert der Broker keinen Offset, und stattdessen wird `—` angezeigt.

Ist die Partitionsnummer zu groß oder bestätigt der Broker die Nachricht nicht innerhalb des Verbindungs-Timeouts, zeigt die App einen Fehler. Auch ein nicht vorhandenes Topic führt zu einem Fehler, es sei denn, der Broker legt Topics automatisch an (`auto.create.topics.enable`). Ist das automatische Anlegen deaktiviert, kann es bis zu etwa 30 s (dem Verbindungs-Timeout) dauern, bis dieser Fehler erscheint. Fehler im Formular, etwa ungültiges JSON oder ein unbekannter Platzhalter, werden angezeigt, bevor etwas gesendet wird.

Bei einer Prod-Verbindung verlangt das Senden die Eingabe des Verbindungsnamens und eine Bestätigung mit **An Prod senden**. Siehe [Umgebungen und Prod](connections.md#environments).

**Formular leeren** leert das Formular und setzt die Producer-Einstellungen auf die Standardwerte zurück. Das Topic bleibt erhalten.

<a id="key-value-formats"></a>

## Formate für Schlüssel und Wert

**Schlüsselformat** und **Wertformat** legen fest, wie der Text im Feld in Bytes umgewandelt wird:

- `string` — der Text unverändert, in UTF-8.
- `json` — der Text wird auf gültiges JSON geprüft und so gesendet, wie er eingegeben wurde, ohne Neuformatierung.
- `hex` — zwei Hex-Ziffern pro Byte, zum Beispiel `48656c6c6f`. Leerzeichen und Zeilenumbrüche werden ignoriert.
- `base64` — Bytes in Standard-Base64.
- `null` — überhaupt kein Schlüssel bzw. kein Wert. Das Feld ist leer, und eine Texteingabe schaltet das Format auf `string` um.

Standard ist `null` für den Schlüssel und `json` für den Wert. Um eine Nachricht mit Schlüssel zu senden, geben Sie ihn einfach ein: Das Format wechselt zu `string`. Ein leerer String ist auch ein Schlüssel, kein fehlender: Wenn Sie `string` wählen und das Feld leer lassen, erhalten alle solchen Nachrichten denselben Hash und landen in einer Partition.

Eine Nachricht mit dem Wert `null` ist ein Tombstone. In einem Topic mit `cleanup.policy=compact` weist sie Kafka an, ältere Nachrichten mit demselben Schlüssel zu entfernen.

<a id="headers"></a>

## Header

Header sind Paare aus Name und Wert, die mit der Nachricht gesendet werden. Klicken Sie auf **Header hinzufügen**, um eine Zeile hinzuzufügen, und füllen Sie dann **Name**, **Wert** und **Format** aus. Die Schaltfläche `×` entfernt die Zeile.

Das Format eines Header-Werts ist `string` (Text in UTF-8) oder `hex` (Bytes als Hex-Ziffern). Zeilen mit leerem Namen werden nicht gesendet. Kafka erlaubt mehrere Header mit demselben Namen, und alle werden gesendet.

Platzhalter funktionieren in Header-Werten, aber nicht in Header-Namen.

<a id="placeholders"></a>

## Platzhalter

Platzhalter werden bei jedem Senden durch Werte ersetzt. Sie funktionieren im Schlüssel, im Wert und in Header-Werten. Die Liste steht unter den Headern:

- `{{uuid}}` — eine neue zufällige UUID, zum Beispiel `0b6f6f2c-3c1e-4a8e-9a54-2f0d6f1d7c21`.
- `{{now}}` — die aktuelle Zeit in UTC im Format ISO 8601 mit Millisekunden, zum Beispiel `2026-10-15T12:30:00.000Z`.
- `{{nowMillis}}` — die aktuelle Zeit in Millisekunden seit 1970-01-01 UTC.
- `{{seq}}` — die Nummer der Kopie in einem Stapel, beginnend bei 0. Beim einzelnen Senden ist sie `0`.
- `{{randomInt 1 100}}` — eine zufällige ganze Zahl zwischen den beiden Zahlen, beide eingeschlossen.
- `{{randomString 8}}` — eine zufällige Zeichenkette aus lateinischen Buchstaben und Ziffern der angegebenen Länge, bis zu 100000.
- `{{env NAME}}` — der Wert der Umgebungsvariablen `NAME`. Ist sie nicht gesetzt, wird nichts gesendet und ein Fehler angezeigt.

Leerzeichen innerhalb der Klammern sind erlaubt. Ein unbekannter Platzhalter oder ein fehlendes `}}` bricht das Senden mit einem Fehler ab. Um `{{` als Text zu senden, schreiben Sie `\{{`.

Beim Format `json` wird das JSON nach dem Ersetzen der Platzhalter geprüft. Platzhalter, die Text erzeugen, brauchen Anführungszeichen: `{"id": "{{uuid}}"}`. Zahlen nicht: `{"n": {{seq}}}`.

`{{env NAME}}` liest die Umgebung des Glafira-Prozesses. Starten Sie Glafira über das Dock oder das Startmenü, fehlen möglicherweise Variablen, die nur in Ihrem Terminal-Profil gesetzt sind.

Kommt eine Nachricht aus [Im Producer öffnen](browse.md#open-in-producer) oder aus dem Verlauf, wird jedes `{{` darin als `\{{` geschrieben. Die Nachricht wird dann genau so gesendet, wie sie war, ohne dass etwas ersetzt wird.

<a id="templates"></a>

## Vorlagen

Eine Vorlage speichert das ganze Formular: Topic, Partition, Schlüssel, Wert, Header, Formate und Producer-Einstellungen. Platzhalter werden so gespeichert, wie sie geschrieben sind, und bei jedem Senden ersetzt.

Um eine Vorlage zu speichern, klicken Sie auf **Als Vorlage speichern**, geben unter **Vorlagenname** einen Namen ein und klicken auf **Speichern**. Das Topic darf leer sein. Standardmäßig gehört die Vorlage zu dieser Verbindung. Wählen Sie **Für alle Verbindungen**, um sie bei jeder Verbindung anzuzeigen; solche Vorlagen sind in der Liste mit **(alle Verbindungen)** gekennzeichnet.

Um eine Vorlage zu verwenden, wählen Sie sie in der Liste **Vorlage**. Das Formular wird durch die Vorlage ersetzt. Sie können das Formular vor dem Senden bearbeiten; die Vorlage ändert sich dabei nicht.

Beim Speichern entsteht immer eine neue Vorlage. Um eine Vorlage zu ändern, laden Sie sie, bearbeiten das Formular, speichern es unter einem neuen Namen und löschen die alte. Um die ausgewählte Vorlage zu löschen, klicken Sie auf **Vorlage löschen** und bestätigen mit **Löschen**.

Die Vorlagen einer Verbindung werden zusammen mit der Verbindung gelöscht.

<a id="history"></a>

## Verlauf

**Verlauf (letzte 100)** listet die letzten 100 Einzelsendungen dieser Verbindung auf, die neuesten zuerst. Nachrichten aus Stapelsendungen werden nicht aufgenommen. Jede Zeile zeigt Zeit, Topic, Partition und Offset sowie den Anfang des Schlüssels.

Der Verlauf speichert die Nachricht so, wie sie gesendet wurde, mit bereits ersetzten Platzhaltern. **Erneut senden** lädt sie zusammen mit ihren Producer-Einstellungen ins Formular. Gesendet wird erst, wenn Sie auf **Senden** klicken. Die Kopie wird genau so gesendet, wie sie gespeichert ist: Beispielsweise wird dieselbe UUID erneut gesendet.

Ein Schlüssel, Wert oder Header-Wert über 64 KB wird nur teilweise gespeichert. Für solche Zeilen ist **Erneut senden** deaktiviert.

Der Verlauf wird auf diesem Computer im Klartext gespeichert, einschließlich der Werte von `{{env NAME}}`. Senden Sie keine Geheimnisse über den Producer, wenn das eine Rolle spielt. Der Verlauf einer Verbindung wird zusammen mit der Verbindung gelöscht.

<a id="batch-copies"></a>

## N Kopien des Formulars

**Stapelversand** unter dem Formular sendet viele Nachrichten in einem Durchlauf. Im Modus **N Kopien des Formulars** wird das Formular so oft wie angegeben gesendet.

1. Füllen Sie das Formular wie für ein einzelnes Senden aus.
2. Wählen Sie **N Kopien des Formulars** und geben Sie unter **Kopien** die Anzahl ein, von 1 bis 1000000.
3. Legen Sie optional die **Senderate** fest. Siehe [Senderate](producer.md#batch-rate).
4. Klicken Sie auf **Starten**.

Platzhalter werden für jede Kopie separat ersetzt. `{{seq}}` ist die Nummer der Kopie, von 0 bis N−1, sodass jede Nachricht einen eigenen Schlüssel erhalten kann, zum Beispiel `order-{{seq}}`. Fehler in Platzhaltern werden vor dem Start an der ersten Kopie geprüft.

Während des Durchlaufs zeigt der Bereich, wie viele Nachrichten gesendet wurden und wie viele fehlgeschlagen sind. Am Ende zeigt er **fertig**. Fehler werden mit der Nummer der Kopie aufgeführt, gezählt ab 1. Die Liste zeigt die ersten 20 Fehler; gibt es mehr, nennt ein Hinweis darunter die Gesamtzahl.

**Abbrechen** beendet den Durchlauf. Nachrichten, die die App noch nicht verlassen haben, werden nicht gesendet, und der Bereich zeigt ihre Anzahl. Die App wartet bis zu 2 Sekunden, bis der Broker bereits gesendete Nachrichten bestätigt. Danach noch unbestätigte Nachrichten zählen als Fehler, da unbekannt ist, ob sie geschrieben wurden. Auch das Schließen des Producer-Tabs beendet den Durchlauf.

Bei einer Prod-Verbindung verlangt der Durchlauf den Verbindungsnamen und wird mit **Auf Prod starten** bestätigt.

<a id="batch-file"></a>

## Aus einer Datei senden

Im Modus **aus Datei** wird jede Zeile bzw. jeder Datensatz einer Datei zu einer Nachricht. Unterstützt werden JSON Lines (`.jsonl`, `.ndjson`) und CSV (`.csv`).

1. Geben Sie im Formular oben das **Topic** ein. Alle Nachrichten gehen an dieses Topic.
2. Wählen Sie **aus Datei** und klicken Sie auf **Datei auswählen…**.
3. Klicken Sie auf **Starten**.

Schlüssel, Wert, Header und Partition aus dem Formular werden nicht verwendet, und Platzhalter werden nicht ersetzt. Die Producer-Einstellungen des Formulars gelten.

In JSON Lines ist jede Zeile ein JSON-Objekt, zum Beispiel `{"key": "k1", "value": {"id": 1}, "headers": {"source": "test"}, "partition": 0}`. Alle Felder sind optional:

- `key` und `value` — eine JSON-Zeichenkette wird als ihr Text gesendet, ohne Anführungszeichen. Jeder andere JSON-Wert, etwa ein Objekt oder eine Zahl, wird als JSON-Text gesendet. `null` oder ein fehlendes Feld bedeutet kein Schlüssel bzw. kein Wert.
- `headers` — ein Objekt mit Zeichenketten als Werten.
- `partition` — eine Partitionsnummer. Fehlt sie, wird die Partition anhand des Schlüssels gewählt.

Leere Zeilen werden übersprungen. Die Datei muss in UTF-8 vorliegen.

In CSV enthält die erste Zeile die Spaltennamen. Die Spalte `value` ist erforderlich; die Spalten `key`, `partition` und `header.NAME` sind optional, zum Beispiel `header.source`. Eine leere `key`-Zelle bedeutet kein Schlüssel, eine leere Header-Zelle bedeutet, dass dieser Header fehlt. Eine leere `value`-Zelle sendet einen leeren Wert, nicht `null`; aus CSV lässt sich also kein Tombstone senden.

Die Datei wird als Stream gelesen, daher ist die Gesamtzahl vorab nicht bekannt und wird als `—` angezeigt. Eine Zeile oder ein Datensatz über 16 MB, ungültiges JSON oder eine negative Partition zählt als Fehler für diese Zeile, und der Durchlauf fährt mit der nächsten Zeile fort. Fehler werden mit der Zeilennummer angezeigt.

<a id="batch-rate"></a>

## Senderate

**Senderate** begrenzt einen Stapel auf die angegebene Anzahl Nachrichten pro Sekunde. Geben Sie eine ganze Zahl ab 1 ein. Ein leeres Feld bedeutet **unbegrenzt**.

Die Nachrichten werden gleichmäßig über die Zeit verteilt: Bei 10 Nachrichten pro Sekunde geht etwa alle 100 ms eine Nachricht hinaus. Nutzen Sie die Begrenzung, um einen gemeinsam genutzten Cluster nicht zu überlasten oder eine gleichmäßige Last nachzubilden.

Ohne Begrenzung sendet die App so schnell, wie der Broker annimmt. Die Senderate gilt für beide Stapelmodi.

<a id="producer-settings"></a>

## Producer-Einstellungen

**Producer-Einstellungen** gelten für Einzelsendungen und Stapel. Sie werden in Vorlagen und im Verlauf gespeichert.

`acks` legt fest, wie viele Replikate einen Schreibvorgang bestätigen müssen:

- `all` — der Standard. Der Leader wartet auf alle synchronen Replikate. Das ist die sicherste Wahl. Sind weniger Replikate synchron, als die Topic-Einstellung `min.insync.replicas` verlangt, schlägt der Schreibvorgang fehl.
- `1` — nur der Leader bestätigt. Schneller, aber eine Nachricht kann verloren gehen, wenn der Leader ausfällt, bevor die Follower sie kopiert haben.
- `0` — der Producer wartet auf keine Bestätigung. Nachrichten können verloren gehen, und es wird kein Offset zurückgegeben.

**Komprimierung** komprimiert Nachrichten-Batches: `none` (Standard), `gzip`, `snappy`, `lz4` oder `zstd`. Komprimierung spart Netzwerkverkehr und Speicherplatz auf Kosten von etwas CPU-Leistung. `zstd` und `lz4` sind gängige Optionen. Bei einer einzelnen kleinen Nachricht ist der Gewinn gering.

**Idempotenz** setzt `enable.idempotence`. Sie verhindert Duplikate, wenn der Client ein Senden nach einem Netzwerkfehler wiederholt. Sie erfordert `acks` mit dem Wert `all`; bei anderen Werten zeigt die App einen Fehler und sendet nichts.
