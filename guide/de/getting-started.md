# Erste Schritte

Glafira for Apache Kafka ist ein Desktop-Client für Kafka-Cluster. Damit können Sie Topics und Consumer Groups durchsuchen, Nachrichten lesen und senden sowie Offsets zurücksetzen. Dieses Kapitel behandelt die Installation, den ersten Start, ein lokales Kafka zum Üben und die Sprache der Oberfläche.

<a id="install-macos"></a>

## Installation unter macOS

Glafira läuft unter macOS 11 oder neuer. Das Installationsprogramm finden Sie auf der Release-Seite: https://github.com/vvnovg/glafira-kafka-releases/releases

Wählen Sie die passende Datei für Ihren Mac: `aarch64` für Apple Silicon (M1 und neuer), `x64` für Intel.

1. Öffnen Sie die `.dmg`-Datei.
2. Ziehen Sie Glafira for Apache Kafka in den Ordner „Programme“.
3. Starten Sie die App aus „Programme“ oder dem Launchpad.

Alternativ können Sie das `.pkg`-Installationsprogramm verwenden. Es installiert die App für alle Benutzer in `/Applications`.

Die Builds sind von Apple signiert und notarisiert, daher öffnet sich die App sofort und ohne Sicherheitswarnungen.

Meldet macOS trotzdem, die App sei beschädigt oder könne nicht überprüft werden, ist der Download fehlerhaft: Laden Sie die `.dmg`- oder `.pkg`-Datei erneut von der Release-Seite herunter.

<a id="install-windows"></a>

## Installation unter Windows

Laden Sie das `.msi`-Installationsprogramm (`x64`) von der Release-Seite herunter: https://github.com/vvnovg/glafira-kafka-releases/releases

1. Führen Sie die `.msi`-Datei aus.
2. Bestätigen Sie die Installation in der Windows-Abfrage. Die App wird für alle Benutzer des Computers installiert, daher sind Administratorrechte erforderlich.
3. Starten Sie Glafira for Apache Kafka über das Startmenü.

Beta-Builds sind noch nicht signiert. Blockiert SmartScreen das Installationsprogramm, klicken Sie auf „Weitere Informationen“ und dann auf „Trotzdem ausführen“. Sie können die Datei auch vorab in PowerShell entsperren:

```powershell
Unblock-File -Path "$env:USERPROFILE\Downloads\Glafira_for_Apache_Kafka_*_x64_en-US.msi"
```

<a id="install-linux"></a>

## Installation unter Linux

Builds für `x86_64` gibt es in drei Formaten. Laden Sie das für Ihre Distribution passende von der Release-Seite herunter: https://github.com/vvnovg/glafira-kafka-releases/releases

- Debian, Ubuntu und Derivate: `sudo apt install ./<file>.deb`
- Fedora, RHEL und Derivate: `sudo dnf install ./<file>.rpm`
- Beliebige Distribution: Machen Sie das AppImage mit `chmod +x <file>.AppImage` ausführbar und starten Sie es.

Starten Sie die App nach der Installation eines Pakets über das Anwendungsmenü oder mit `glafira-kafka` im Terminal.

Glafira speichert Passwörter im Secret Service (zum Beispiel GNOME Keyring oder KWallet). Stellen Sie sicher, dass einer davon in Ihrer Sitzung läuft. Andernfalls kann die App keine Passwörter für Verbindungen speichern.

<a id="first-run"></a>

## Erster Start

Das Fenster besteht aus zwei Bereichen. Links befindet sich der Bereich **Verbindungen**. Rechts liegen Tabs: Jeder geöffnete Bildschirm, etwa eine Topic-Liste oder der Producer, erhält einen eigenen Tab.

Solange es keine Verbindungen gibt, zeigt der Bereich einen kurzen Hinweis, und der Tab-Bereich schlägt vor, eine Verbindung hinzuzufügen. So beginnen Sie:

1. Klicken Sie im Kopf des Bereichs **Verbindungen** auf `+` ( **Neue Verbindung** ).
2. Füllen Sie **Name** und **Bootstrap-Server** aus, zum Beispiel `localhost:9092`.
3. Klicken Sie auf **Verbindung testen**, um die Einstellungen zu prüfen, und dann auf **Speichern**.

Nach dem Speichern öffnet sich die Cluster-Übersicht. Unter jeder Verbindung im Bereich gibt es Links zu **Cluster-Übersicht**, **Topics**, **Consumer Groups**, **Producer** und **Consumer-Session**. Alle Verbindungseinstellungen sind unter [Verbindungen](connections.md#create) beschrieben.

Unten im Bereich befinden sich Links zu **Einstellungen**, **Projekt unterstützen**, **Hilfe** und **Über Glafira**. Das Zeichen `?` neben einem Feld zeigt einen kurzen Hinweis dazu. Mit `F1` öffnen Sie an jeder Stelle des Fensters das Kapitel der Anleitung zum aktuellen Bildschirm. Unter macOS ist die Anleitung auch im Menü „Hilfe“ zu finden.

Wenn Sie Kafka Explorer (den früheren Namen der App) verwendet haben, übernimmt Glafira beim ersten Start dessen Verbindungen, Vorlagen und Sendeverlauf. Auch die Passwörter werden aus dem Schlüsselbund des Systems kopiert. Bereits Vorhandenes wird nicht überschrieben, und die alten Daten werden nicht gelöscht. macOS fragt möglicherweise einmal nach Zugriff auf die alten Schlüsselbundeinträge.

<a id="local-cluster"></a>

## Lokales Kafka zum Üben

Um die App ohne gemeinsamen Cluster auszuprobieren, starten Sie Kafka in Docker auf Ihrem Computer. Ein Broker genügt:

```bash
docker run -d --name kafka -p 9092:9092 apache/kafka:3.9.0
```

Legen Sie dann eine Verbindung mit dem Bootstrap-Server `localhost:9092`, dem Protokoll `PLAINTEXT` und der Umgebung `local` an. Wenn Sie den Broker nicht mehr brauchen, entfernen Sie ihn mit `docker rm -f kafka`.

Der Hinweis im leeren Bereich **Verbindungen** erwähnt `docker compose -f dev/docker-compose.yml up -d`. Diese Datei gehört zum Quellcode von Glafira. Wenn Sie den Quellcode haben, startet dieser Befehl einen Broker mit drei Listenern:

- `localhost:9092` — `PLAINTEXT`.
- `localhost:9094` — `SASL_PLAINTEXT` mit `PLAIN` (Benutzer `tester` / `tester-secret` und `admin` / `admin-secret`) und SCRAM (`scram-user` / `scram-secret` für `SCRAM-SHA-256` und `SCRAM-SHA-512`).
- `localhost:9095` — `SSL` mit Client-Zertifikaten (mTLS). Die Testzertifikate liegen in `dev/certs/out`.

Diese Benutzer und Zertifikate sind nur für lokale Tests gedacht.

<a id="language"></a>

## Sprache der Oberfläche

Die Oberfläche ist auf Englisch, Russisch, Deutsch, Französisch, Spanisch, Italienisch, Polnisch, Portugiesisch (Brasilien), Chinesisch (vereinfacht und traditionell), Japanisch und Koreanisch verfügbar. Alle Sprachen sind kostenlos.

So ändern Sie die Sprache:

1. Klicken Sie unten im linken Bereich auf **Einstellungen**.
2. Wählen Sie unter **Sprache der Oberfläche** eine Sprache.

Die Änderung gilt sofort, auch für diese Anleitung und das macOS-Menü. **Systemsprache** (die Standardeinstellung) folgt der Sprache Ihres Betriebssystems. Ist diese Sprache nicht in der Liste, erscheint die Oberfläche auf Englisch.

<a id="managed-settings"></a>

## Von Ihrer Organisation festgelegte Einstellungen

In Unternehmen können Administratoren einige Optionen per Richtlinie für alle Benutzer festlegen. Diese Optionen lassen sich in der App nicht ändern. Eine Richtlinie kann:

- die Sprache der Oberfläche festlegen — die Sprachliste unter **Einstellungen** ist dann nicht verfügbar, mit dem Vermerk, dass die Sprache von Ihrer Organisation festgelegt ist;
- die Suche nach Updates abschalten oder auf einen internen Mirror umleiten.

Ob eine Richtlinie gilt, sehen Sie unter **Über Glafira** in der Zeile **Organisationsrichtlinie**. Enthält die Richtlinie einen Fehler, wird er dort angezeigt. Bei Fragen zur Richtlinie wenden Sie sich an Ihre Administratoren. Updates sind unter [Updates und Diagnose](updates.md) beschrieben.
