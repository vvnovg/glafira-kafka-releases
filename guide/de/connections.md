# Verbindungen

Eine Verbindung ist ein gespeichertes Profil eines Kafka-Clusters: Adressen, Sicherheitseinstellungen und Passwörter. Die Verbindungen stehen links im Bereich **Verbindungen**. Sie lassen sich in Ordner einsortieren und per Export und Import auf einen anderen Computer übertragen.

<a id="create"></a>

## Verbindung anlegen

1. Klicken Sie im Kopf des Bereichs **Verbindungen** auf `+` ( **Neue Verbindung** ). Um die Verbindung direkt in einem Ordner anzulegen, öffnen Sie das Ordnermenü und wählen **Neue Verbindung hier**.
2. Geben Sie unter **Name** einen Namen ein. Er erscheint im Bereich und in den Tab-Titeln.
3. Geben Sie unter **Bootstrap-Server** eine oder mehrere Adressen im Format `host:port` ein, getrennt durch Kommas, zum Beispiel `kafka1:9092,kafka2:9092`. Ein erreichbarer Broker genügt: Die übrigen findet der Client selbst.
4. Wählen Sie ein **Protokoll** und füllen Sie die zusätzlichen Felder aus. Siehe [Sicherheitsprotokolle](connections.md#security).
5. Optional wählen Sie **Ordner**, **Umgebung** und **Farbe** für den Punkt neben dem Namen.
6. Klicken Sie auf **Verbindung testen** und dann auf **Speichern**.

Nach dem Speichern schließt sich das Formular, und die Cluster-Übersicht öffnet sich.

Zwei weitere Felder sind für Sonderfälle gedacht. **Anfrage-Timeout, ms** legt fest, wie lange auf eine Antwort des Brokers gewartet wird. Standard ist `30000` ms, Minimum `1000`. Erhöhen Sie den Wert bei langsamen Netzen und VPNs. **Zusätzliche librdkafka-Eigenschaften** nimmt beliebige librdkafka-Einstellungen als `key=value` auf, eine pro Zeile. Zeilen, die mit `#` beginnen, werden ignoriert. Passwörter und Schlüssel sind dort nicht erlaubt: Eigenschaften, deren Name `password` oder `secret` enthält, sowie `sasl.jaas.config`, `ssl.key.pem` und `ssl.keystore.key` werden beim Speichern abgelehnt.

Um eine Verbindung später zu ändern, klicken Sie neben ihrem Namen auf `✎` ( **Verbindung bearbeiten** ). Zum Löschen öffnen Sie das Formular, klicken auf **Löschen** und bestätigen mit **Ja, löschen**.

<a id="test"></a>

## Verbindung testen

**Verbindung testen** prüft die Einstellungen im Formular Schritt für Schritt und hält beim ersten fehlgeschlagenen Schritt an:

1. **Konfiguration** — Adressen, Zertifikatsdateien, Passwörter und SASL-Benutzername.
2. **DNS / TCP** — die Hostnamen werden aufgelöst, und zu mindestens einem Bootstrap-Server lässt sich eine TCP-Verbindung öffnen.
3. **Kafka (TLS, SASL, Metadaten)** — TLS-Handshake und SASL-Anmeldung gelingen, und der Broker liefert Metadaten. Bei Erfolg zeigt der Schritt die Anzahl der Broker und Topics.

Der Test verwendet die aktuellen Werte im Formular, auch eingegebene, aber noch nicht gespeicherte Passwörter. Es wird nichts gespeichert und im Cluster nichts geändert. Schlägt ein Schritt fehl, erscheinen darunter Fehlertext und Details. Häufige Fehler sind unter [Keine Verbindung möglich](troubleshooting.md#cannot-connect) erklärt.

<a id="security"></a>

## Sicherheitsprotokolle

Das **Protokoll** muss zum Listener des Brokers passen, mit dem Sie sich verbinden. Welches das ist, wissen Ihre Kafka-Administratoren.

- `PLAINTEXT` — keine Verschlüsselung, keine Anmeldung. Typisch für lokale und Test-Cluster.
- `SSL` — der Datenverkehr wird per TLS verschlüsselt. Mit Client-Zertifikat ist das mTLS.
- `SASL_PLAINTEXT` — Anmeldung mit Benutzername und Passwort, ohne Verschlüsselung.
- `SASL_SSL` — Anmeldung mit Benutzername und Passwort über TLS.

Wenn Sie das Protokoll wechseln, zeigt das Formular nur die Felder, die dieses Protokoll braucht.

<a id="sasl"></a>

## SASL

Für `SASL_PLAINTEXT` und `SASL_SSL` füllen Sie drei Felder aus:

- **SASL-Mechanismus** — `PLAIN`, `SCRAM-SHA-256` oder `SCRAM-SHA-512`. Welcher funktioniert, bestimmt der Cluster.
- **Benutzername**.
- **Passwort**. Es wird im Schlüsselbund des Systems gespeichert, nicht im Profil. Siehe [Passwörter](connections.md#passwords).

SCRAM überträgt nie das Passwort selbst. `PLAIN` sendet es unverändert, daher geht das Passwort über `SASL_PLAINTEXT` im Klartext übers Netz. Verwenden Sie für Cluster außerhalb Ihres Computers nach Möglichkeit `SASL_SSL`.

<a id="tls"></a>

## SSL und vertrauenswürdige CA

Für `SSL` und `SASL_SSL` ergänzt das Formular die TLS-Einstellungen.

**Vertrauenswürdiges CA-Zertifikat** ist das Zertifikat der Zertifizierungsstelle (CA), die die Broker-Zertifikate signiert hat. Lassen Sie das Feld leer, wenn das Betriebssystem dieser CA bereits vertraut: Dann werden die Stammzertifikate des Systems verwendet. Andernfalls geben Sie den vollständigen Pfad zur Datei ein oder klicken auf **Durchsuchen…**. Nach der Auswahl einer Datei erscheint das Feld **CA-Format**. Für PKCS#12 und JKS gibt es zusätzlich das Feld **Truststore-Passwort**.

**Hostname prüfen** ist standardmäßig aktiviert. Es prüft, ob das Broker-Zertifikat für den Hostnamen ausgestellt wurde, mit dem Sie sich verbinden. Deaktivieren Sie die Option nur in Testumgebungen, etwa wenn Broker selbstsignierte Zertifikate mit anderen Namen verwenden.

<a id="mtls"></a>

## Client-Zertifikat (mTLS)

Authentifizieren die Broker Clients per Zertifikat, aktivieren Sie **Client-Zertifikat (mTLS)**. Zertifikat und Schlüssel stellen Ihre Kafka-Administratoren aus. Füllen Sie dann aus:

- **Client-Zertifikat** — Pfad zur Zertifikatsdatei.
- **Format des Client-Zertifikats** — PEM, PKCS#12 oder JKS.
- Für PEM: **Client-Schlüssel** — Pfad zur Datei mit dem privaten Schlüssel, sowie **Schlüsselpasswort (falls verschlüsselt)**, wenn der Schlüssel verschlüsselt ist.
- Für PKCS#12: **Keystore-Passwort**.
- Für JKS: **Keystore-Passwort**, **Schlüsselpasswort (falls abweichend)**, wenn der Schlüssel ein eigenes Passwort hat, und **Schlüssel-Alias**, wenn der Keystore mehrere Schlüssel enthält.

Pfade müssen absolut sein. Die Dateien werden bei jedem Verbindungsaufbau gelesen, nach einer Erneuerung des Zertifikats genügt es also, sich neu zu verbinden. Die App prüft, ob der Schlüssel zum Zertifikat passt und ob das Zertifikat noch gültig ist.

Ein häufiger Fall ist ein Cluster in Kubernetes, bei dem das Client-Secret `tls.crt`, `tls.key` und `ca.crt` enthält. Speichern Sie sie als Dateien, wählen Sie das Protokoll `SSL`, setzen Sie **Vertrauenswürdiges CA-Zertifikat** auf `ca.crt` und verwenden Sie `tls.crt` und `tls.key` als PEM-Client-Zertifikat und -Schlüssel mit leeren Passwörtern.

<a id="certificate-formats"></a>

## Zertifikatsformate

Glafira liest drei Formate. Das Format wird automatisch anhand der Dateiendung gewählt; prüfen Sie es im Formatfeld und ändern Sie es bei Bedarf.

- PEM — eine Textdatei, die mit `-----BEGIN` beginnt. Endungen `.pem`, `.crt`, `.cer`, `.key`. Bei einem Client-Zertifikat liegt der Schlüssel in einer separaten Datei. Die vertrauenswürdige CA kann auch ein binäres DER-Zertifikat sein.
- PKCS#12 — ein binärer Speicher mit Zertifikat und Schlüssel. Endungen `.p12`, `.pfx`.
- JKS — ein Java-Keystore. Endungen `.jks`, `.keystore`, `.truststore`.

Zwei Arten von Dateien müssen vorher konvertiert werden:

- JCEKS-Keystores werden nicht unterstützt. Konvertieren Sie sie nach PKCS#12:

```bash
keytool -importkeystore -srcstoretype JCEKS -srckeystore client.jceks -deststoretype PKCS12 -destkeystore client.p12
```

- PKCS#12-Dateien, die mit veralteten Algorithmen (RC2, 3DES) verschlüsselt sind, lassen sich möglicherweise nicht öffnen. Meldet die App einen veralteten Algorithmus, erstellen Sie die Datei mit AES neu:

```bash
openssl pkcs12 -export -keypbe AES-256-CBC -certpbe AES-256-CBC -in tls.crt -inkey tls.key -out client.p12
```

<a id="passwords"></a>

## Passwörter

Das SASL-Passwort sowie die Passwörter von Schlüsseln und Keystores werden im Schlüsselbund des Systems gespeichert: in der Schlüsselbundverwaltung unter macOS, in der Anmeldeinformationsverwaltung unter Windows, im Secret Service unter Linux. Sie werden nie in das Verbindungsprofil, in Exportdateien oder in das Diagnosearchiv geschrieben.

Wenn Sie eine gespeicherte Verbindung bearbeiten, sind die Passwortfelder leer und zeigen **leer lassen, um es beizubehalten**. Lassen Sie ein Feld leer, um das gespeicherte Passwort beizubehalten, oder geben Sie ein neues ein, um es zu ersetzen. Um ein gespeichertes Zertifikatspasswort zu löschen, wählen Sie neben dem Feld **Gespeichertes Passwort entfernen**. Beim Speichern werden Passwörter gelöscht, die die Verbindung nicht mehr verwendet, etwa nach einem Wechsel des Protokolls auf `PLAINTEXT`.

Kann die App nicht auf den Schlüsselbund zugreifen, erscheint beim Speichern ein Fehler. Prüfen Sie unter Linux, ob GNOME Keyring, KWallet oder ein anderer Secret-Service-Anbieter in Ihrer Sitzung läuft.

<a id="environments"></a>

## Umgebungen und Prod

**Umgebung** kennzeichnet die Verbindung: `local`, `dev`, `test`, `stage` oder `prod`. Die Umgebung erscheint als farbiges Badge neben dem Namen im Bereich, sodass Sie immer sehen, mit welchem Cluster Sie arbeiten.

`prod` bietet zusätzlichen Schutz. Jede Änderung über eine Prod-Verbindung, etwa das Erstellen eines Topics, das Senden einer Nachricht, das Zurücksetzen von Offsets oder das Starten einer Consumer-Session, verlangt die Eingabe des Verbindungsnamens. Löschen und Leeren verlangen bei jeder Verbindung den Namen des Objekts (beim Löschen mehrerer Topics deren Anzahl), bei Prod geben Sie beides ein. Diese Bestätigungen erscheinen im App-Fenster.

Eine neue Prod-Verbindung ist zunächst **Schreibgeschützt**. Deaktivieren Sie das Kontrollkästchen, wenn Sie Änderungen vornehmen müssen.

<a id="read-only"></a>

## Schreibgeschützt

**Schreibgeschützt** sperrt alles, was den Cluster verändert: Erstellen, Ändern, Leeren und Löschen von Topics, Senden von Nachrichten, Zurücksetzen von Offsets, Löschen von Gruppen und Starten von Consumer-Sessions. Die Schaltflächen für diese Aktionen sind ausgeblendet oder deaktiviert, und auch der Kern der App lehnt solche Vorgänge ab. Durchsuchen und Lesen bleiben möglich. Ein Schloss-Symbol neben dem Namen im Bereich kennzeichnet schreibgeschützte Verbindungen.

Verwenden Sie diesen Modus, wenn Sie Daten nur ansehen wollen, besonders auf gemeinsam genutzten Clustern. Der Schreibschutz ist ein Schutz innerhalb der App: Er ersetzt keine Zugriffsrechte (ACLs) in Kafka.

<a id="folders"></a>

## Ordner

Ordner helfen beim Sortieren von Verbindungen, etwa nach Projekt oder Umgebung. Ordner lassen sich verschachteln.

- Um einen Ordner auf oberster Ebene anzulegen, klicken Sie im Kopf des Bereichs auf `+📁` ( **Neuer Ordner** ). Für einen Unterordner öffnen Sie das Ordnermenü und wählen **Neuer Ordner**. Geben Sie den Namen ein und drücken Sie `Enter`, oder brechen Sie mit `Esc` ab.
- Um das Ordnermenü zu öffnen, klicken Sie in der Zeile des Ordners auf `⋯` ( **Weitere Aktionen** ) oder klicken mit der rechten Maustaste auf den Ordner. Das Menü enthält **Umbenennen**, **Löschen** sowie die Befehle für Export und Import.
- Nur ein leerer Ordner kann gelöscht werden. Verschieben oder löschen Sie zuerst seine Verbindungen und Unterordner.
- Ordnernamen müssen innerhalb einer Ebene eindeutig und höchstens 100 Zeichen lang sein.

Um eine Verbindung zu verschieben, ziehen Sie sie auf einen Ordner oder auf eine andere Verbindung, um sie davor einzuordnen. Um sie auf die oberste Ebene zu verschieben, legen Sie sie auf dem Kopf des Bereichs oder im leeren Platz unter der Liste ab. Sie können auch im Verbindungsformular einen **Ordner** wählen; **(Stammebene)** steht für die oberste Ebene. Ordner werden auf dieselbe Weise gezogen. Legen Sie einen Ordner am oberen Rand eines anderen Ordners ab, um ihn davor einzuordnen. Ein Ordner lässt sich nicht in einen eigenen Unterordner verschieben.

Ein Klick auf den Ordnernamen klappt den Ordner ein oder aus. Die App merkt sich, welche Ordner eingeklappt sind. Mit **Verbindungen filtern** über dem Baum finden Sie Verbindungen und Ordner nach Namen. Passende Ordner werden mit ihrem gesamten Inhalt angezeigt.

<a id="export"></a>

## Export

Der Export speichert Verbindungen und Ordner in einer Datei, etwa um sie mit Kollegen zu teilen oder auf einen anderen Computer zu übertragen.

1. Um alles zu exportieren, öffnen Sie das Menü `⋯` im Kopf des Bereichs und wählen **Alles exportieren…**. Um einen Ordner mit seinen Unterordnern zu exportieren, wählen Sie in seinem Menü **Ordner exportieren…**.
2. Wählen Sie den Speicherort. Vorgeschlagen wird `glafira-connections-<date>.glafira-connections.json` oder `<folder>.glafira-connections.json`.

Nach dem Speichern zeigt der Bereich an, wie viele Verbindungen und Ordner die Datei enthält.

Die Datei enthält die Verbindungseinstellungen und die Ordnerstruktur. Passwörter und geheime Eigenschaften enthält sie nicht. Auch Zertifikatsdateien sind nicht enthalten: Die Datei speichert nur deren Pfade. Hosts, Benutzernamen und Pfade stehen im Klartext darin, teilen Sie die Datei also nur mit Personen, die sie sehen dürfen.

<a id="import"></a>

## Import

1. Öffnen Sie das Menü `⋯` im Kopf des Bereichs und wählen Sie **Importieren…**. Um direkt in einen Ordner zu importieren, wählen Sie in seinem Menü **Hierher importieren…**.
2. Wählen Sie eine `.glafira-connections.json`-Datei. Der Dialog **Verbindungen importieren** öffnet sich.
3. Wählen Sie unter **Importieren nach**, wohin neue Ordner und Verbindungen kommen. **(Stammebene)** ist die oberste Ebene.
4. Lassen Sie **Vorhandene Verbindungen aktualisieren** aktiviert, um bereits vorhandene Verbindungen aus der Datei zu aktualisieren. Deaktivieren Sie die Option, um sie unverändert zu lassen.
5. Prüfen Sie die Vorschau und klicken Sie auf **Importieren** oder auf **Abbrechen**, um den Dialog ohne Änderungen zu schließen.

Die Vorschau wird nach jeder Änderung im Dialog aktualisiert. Sie zeigt die Summen und aufklappbare Listen: **Neu**, **Werden aktualisiert**, **Unverändert** und **Übersprungen** mit dem Grund für jede übersprungene Verbindung.

So ordnet die App die Datei Ihren Verbindungen zu:

- Verbindungen werden über eine interne ID zugeordnet, nicht über den Namen. Eine Verbindung gilt als vorhanden, wenn die Datei dieselbe Verbindung enthält: etwa weil sie von diesem Computer exportiert oder früher aus derselben Quelle importiert wurde.
- Eine aktualisierte Verbindung übernimmt die Einstellungen aus der Datei. Ihre Passwörter und ihr Platz im Baum bleiben erhalten.
- Vorhandene Ordner werden nicht umbenannt oder verschoben. Neue Verbindungen aus der Datei landen trotzdem darin.
- Hat eine neue Verbindung denselben Namen wie eine andere im selben Ordner, wird eine Nummer angehängt, zum Beispiel `kafka-dev (2)`.
- Verbindungen, die diese Version von Glafira nicht unterstützt oder die Geheimnisse in zusätzlichen Eigenschaften enthalten, werden übersprungen.

Die Vorschau warnt außerdem, wenn Zertifikatsdateien auf diesem Computer nicht gefunden werden. Passwörter sind nicht in der Datei, daher zeigt der Bereich nach dem Import an, wie viele Verbindungen Passwörter benötigen. Öffnen Sie jede dieser Verbindungen, geben Sie die Passwörter ein und speichern Sie.
