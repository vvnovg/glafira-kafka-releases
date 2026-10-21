# Updates und Diagnose

Dieses Kapitel beschreibt, wie Sie die App aktualisieren, den Bildschirm **Über Glafira**, das Diagnosearchiv und wie Sie das Projekt unterstützen können.

<a id="updates"></a>

## Updates

Beim Start prüft die App einmal, ob eine neue Version verfügbar ist. Gibt es eine, nennt ein Hinweis über den Tabs ihre Version. **Details** öffnet den Bildschirm **Über Glafira**. Schlägt die Prüfung fehl, etwa ohne Internetzugang, erscheint keine Meldung.

So suchen Sie manuell:

1. Öffnen Sie unten im linken Bereich **Über Glafira**.
2. Klicken Sie unter **Updates** auf **Nach Updates suchen**.
3. Ist eine neue Version verfügbar, zeigt der Bildschirm ihre Nummer und die Versionshinweise. Klicken Sie auf **Installieren und neu starten**.

Die App lädt das Update herunter, prüft seine Signatur, installiert es und startet neu. Ein durch ein Netzwerkproblem unterbrochener Download wird automatisch wiederholt, insgesamt bis zu drei Versuche. Schlagen alle drei Versuche fehl, meldet die App **Der Download des Updates wurde unterbrochen — Netzwerk prüfen und erneut versuchen**. Schlägt die Signaturprüfung fehl, meldet die App **Die Signatur des Updates konnte nicht verifiziert werden — das Update wurde nicht installiert**.

Die Installation kann ein Administratorpasswort verlangen, weil die App für alle Benutzer des Computers installiert ist: über das `.pkg`-Installationsprogramm unter macOS, das `.msi`-Installationsprogramm unter Windows oder ein `.deb`- bzw. `.rpm`-Paket unter Linux. Benutzer ohne Administratorrechte können eine solche Installation nicht aktualisieren.

Werden gerade Nachrichten gelesen, läuft eine Consumer-Session oder wird ein Stapel gesendet, fragt die App zuerst nach und nennt die Anzahl der aktiven Sessions. Klicken Sie auf **Sessions beenden und aktualisieren**, um sie zu beenden und fortzufahren, oder auf **Abbrechen**. Verbindungen, Einstellungen und andere Daten bleiben nach einem Update erhalten.

Updates werden von der Release-Seite auf GitHub heruntergeladen. Schlägt die Prüfung mit **Suche nach Updates fehlgeschlagen** fehl, prüfen Sie Ihr Netzwerk oder Ihren Proxy. **Updates sind in diesem Build nicht konfiguriert** bedeutet, dass dieser Build ohne den Updater-Schlüssel erstellt wurde, etwa ein Entwicklungs- oder Test-Build. Laden Sie neue Versionen in diesem Fall von der Release-Seite herunter: https://github.com/vvnovg/glafira-kafka-releases/releases

Eine Organisation kann die Suche nach Updates per Richtlinie abschalten oder auf einen internen Mirror umleiten. Ist die Suche abgeschaltet, zeigt der Abschnitt **Updates** nach einem Klick auf **Nach Updates suchen** die Meldung **Updates werden von Ihrer Organisation verwaltet**. Wie **Updates sind in diesem Build nicht konfiguriert** erscheint diese Meldung erst nach dem Klick; die Prüfung beim Start zeigt in diesen Fällen nichts an.

<a id="about"></a>

## Über Glafira

Öffnen Sie unten im linken Bereich **Über Glafira**. Unter macOS finden Sie den Bildschirm auch im Menü „Hilfe“ als **Über Glafira**.

Der Bildschirm zeigt eine kurze Beschreibung der App sowie:

- **Version** — die Versionsnummer, und **Build** — die Kennung des Quellcodes, aus dem die App gebaut wurde. Geben Sie beides in einer Supportanfrage an.
- **System** — Betriebssystem und Prozessorarchitektur.
- **Lizenz** — Apache 2.0; der Link öffnet den Lizenztext im Browser.
- **Organisationsrichtlinie** — **festgelegt** oder **keine**. Fehler in der Richtlinie werden darunter aufgeführt.
- **Daten:** — der Ordner mit Verbindungsprofilen, Vorlagen, Verlauf und Einstellungen.
- **Protokolle:** — der Ordner mit den Protokolldateien. Siehe [Protokolle](troubleshooting.md#logs).

Auch die Abschnitte **Updates** und **Diagnose** befinden sich auf diesem Bildschirm.

<a id="diagnostics"></a>

## Diagnosearchiv

Das Diagnosearchiv sammelt, was der Support braucht, um ein Problem zu verstehen. So erstellen Sie es:

1. Öffnen Sie **Über Glafira**.
2. Klicken Sie unter **Diagnose** auf **Diagnose erstellen**. Unter macOS können Sie auch im Menü „Hilfe“ **Diagnose erstellen…** wählen.
3. Wählen Sie den Speicherort. Vorgeschlagen wird `diagnostics-<date>.zip`.

Nach dem Speichern zeigt der Bildschirm den Pfad des Archivs. Die ZIP-Datei enthält:

- `info.json` — Version und Build der App, Betriebssystem, Pfade des Daten- und Protokollordners sowie Status der Richtlinie;
- `connections.json` — Verbindungsprofile: Namen, Umgebungen, Bootstrap-Server, Protokolle, SASL-Benutzernamen, Pfade der Zertifikatsdateien, zusätzliche Eigenschaften und Timeouts;
- `logs/` — die Protokolldateien der App.

Das Archiv enthält keine Passwörter: Sie liegen im Schlüsselbund des Systems, und zusätzliche Eigenschaften mit Passwörtern und Geheimnissen werden beim Speichern einer Verbindung abgelehnt. Auch Zertifikatsdateien sind nicht enthalten. Hostnamen, Benutzernamen und Dateipfade stehen im Klartext darin, und die Protokolle enthalten Broker-Adressen aus Kafka-Fehlermeldungen. Sehen Sie das Archiv durch, bevor Sie es außerhalb Ihres Unternehmens versenden.

Die App sendet nichts von sich aus. Hängen Sie das Archiv selbst an Ihre Supportanfrage an, siehe [Support kontaktieren](troubleshooting.md#support).

<a id="support"></a>

## Projekt unterstützen

Glafira ist kostenlos und quelloffen unter der Apache License 2.0. Spenden halten das Projekt am Leben. **Projekt unterstützen** unten im linken Bereich, auf dem Bildschirm **Über Glafira** und unter macOS im Menü „Hilfe“ öffnet die Seite zur Unterstützung im Browser: https://github.com/vvnovg/glafira-kafka-releases/blob/main/SUPPORT.md
