# Fehlerbehebung

Dieses Kapitel erklärt häufige Verbindungsfehler und was jeweils zu prüfen ist. Außerdem zeigt es, wo die Protokolle liegen und was in eine Supportanfrage gehört. Unter macOS öffnet das Menü „Hilfe“ dieses Kapitel über **Fehlerbehebung**.

Eine Fehlermeldung besteht aus einem kurzen Text und auf den meisten Bildschirmen aus dem Fehlercode in Klammern, zum Beispiel `TIMEOUT`. Klappen Sie **Details** auf, um die Originalmeldung des Kafka-Clients (librdkafka) zu sehen. Sie nennt oft die genaue Broker-Adresse und die Ursache.

<a id="cannot-connect"></a>

## Keine Verbindung möglich

Beginnen Sie mit **Verbindung testen** im Verbindungsformular, siehe [Verbindung testen](connections.md#test). Der Test hält beim ersten fehlgeschlagenen Schritt an, und der Schritt zeigt, wo Sie suchen müssen:

- **Konfiguration** — das Problem liegt im Formular selbst: eine Adresse ohne Port, eine fehlende oder nicht lesbare Zertifikatsdatei, ein falsches Zertifikatspasswort. Der Fehler nennt das Feld. Zu Zertifikatsdateien siehe [TLS-Fehler](troubleshooting.md#tls-errors).
- **DNS / TCP** — der Computer erreicht die Bootstrap-Server nicht. Siehe [DNS und TCP](troubleshooting.md#dns-tcp).
- **Kafka (TLS, SASL, Metadaten)** — das Netzwerk funktioniert, aber TLS-Handshake, SASL-Anmeldung oder Metadatenabfrage sind fehlgeschlagen. Siehe [TLS-Fehler](troubleshooting.md#tls-errors), [SASL-Fehler](troubleshooting.md#sasl-errors) und [Timeouts](troubleshooting.md#timeouts).

Die häufigste Ursache ist ein Protokoll, das nicht zum Port passt. Jeder Listener eines Brokers hat einen eigenen Port und ein eigenes Protokoll. Fragen Sie Ihre Kafka-Administratoren, welcher Port zu welchem Protokoll gehört, siehe [Sicherheitsprotokolle](connections.md#security).

Besteht der Test, aber Lesen oder Senden schlägt fehl, liegt die Ursache oft bei den Broker-Adressen. Der Client erhält die Liste der Broker vom Bootstrap-Server und verbindet sich dann mit jedem Broker unter der Adresse, die der Broker bekannt gibt (`advertised.listeners`). Sind diese Adressen intern, etwa Docker- oder Kubernetes-Hostnamen, sind sie von Ihrem Computer aus nicht erreichbar. Die Adressen stehen in der Broker-Tabelle der Cluster-Übersicht, siehe [Cluster-Übersicht](topics.md#cluster-overview). Bitten Sie Ihre Administratoren um einen von außen erreichbaren Listener.

<a id="dns-tcp"></a>

## DNS und TCP

**Keiner der Bootstrap-Server ist per TCP erreichbar** bedeutet, dass die App zu keiner Adresse unter **Bootstrap-Server** eine TCP-Verbindung öffnen konnte. Jede Adresse erhält bis zu 5 Sekunden. Die Details zeigen die zuletzt versuchte Adresse und den Grund. `DNS` in den Details bedeutet, dass der Hostname nicht aufgelöst werden konnte.

Prüfen Sie:

- Hostname und Port, ohne Protokollpräfix wie `https://`;
- ob ein VPN aktiv ist, wenn der Cluster in einem privaten Netz liegt;
- ob eine Firewall den Port blockiert;
- bei einem Broker in Docker, ob der Port für den Host freigegeben ist, zum Beispiel `-p 9092:9092`.

Um den Port außerhalb der App zu prüfen, führen Sie unter macOS oder Linux `nc -vz <host> <port>` aus oder unter Windows in PowerShell `Test-NetConnection <host> -Port <port>`.

**Verbindung zu den Brokern fehlgeschlagen** nach bestandenem TCP-Schritt bedeutet meist, dass ein Broker die Verbindung geschlossen hat oder später nicht erreichbar war. Prüfen Sie auch die Broker-Adressen, siehe [Keine Verbindung möglich](troubleshooting.md#cannot-connect).

<a id="tls-errors"></a>

## TLS-Fehler

Fehler im Schritt **Kafka (TLS, SASL, Metadaten)** während des TLS-Handshakes:

- **Der Broker akzeptiert unter dieser Adresse Verbindungen ohne TLS — Port prüfen oder ein Protokoll ohne SSL wählen**.
- **Das Broker-Zertifikat ist nicht von einer vertrauenswürdigen CA signiert — „Vertrauenswürdige CA“ prüfen**. Setzen Sie **Vertrauenswürdiges CA-Zertifikat** auf die Datei der CA, die die Broker-Zertifikate signiert hat. Diese Datei erhalten Sie von Ihren Administratoren. Bei leerem Feld wird nur den Stammzertifikaten des Systems vertraut.
- **Der Hostname des Brokers stimmt nicht mit seinem Zertifikat überein**. Verbinden Sie sich über den Namen, für den das Zertifikat ausgestellt wurde, nicht über eine IP-Adresse oder einen Alias. Nur in Testumgebungen können Sie **Hostname prüfen** deaktivieren.
- **Der Broker hat das Client-Zertifikat abgelehnt**. Der Broker verlangt ein Client-Zertifikat oder vertraut dem verwendeten nicht. Prüfen Sie, ob **Client-Zertifikat (mTLS)** aktiviert ist und das Zertifikat für diesen Cluster ausgestellt wurde. Siehe [Client-Zertifikat (mTLS)](connections.md#mtls).
- **Fehler bei der TLS-Verbindung** — ein anderes TLS-Problem. Klappen Sie **Details** auf, um die Ursache zu sehen.

Haben Sie ein Protokoll ohne SSL gewählt und sehen trotzdem einen TLS-Fehler, ist der Port wahrscheinlich ein TLS-Listener. Die Details lauten dann `connecting to a SSL listener?`. Wählen Sie `SSL` oder `SASL_SSL` oder verwenden Sie den Port eines Listeners ohne TLS.

Probleme mit Zertifikatsdateien werden im Schritt **Konfiguration** gemeldet, bevor die App eine Verbindung aufbaut. Die Meldung beginnt mit dem Feldnamen: vertrauenswürdige CA, Client-Zertifikat oder Client-Schlüssel. Häufige Fälle:

- Der Pfad ist nicht absolut, oder die Datei lässt sich nicht lesen. Geben Sie den vollständigen Pfad ein oder wählen Sie die Datei mit **Durchsuchen…**.
- **Client-Schlüssel: der Schlüssel ist verschlüsselt — Schlüsselpasswort angeben**, **Client-Schlüssel: falsches Schlüsselpasswort**, **Client-Zertifikat: falsches Passwort für den PKCS\#12-Keystore** oder **Falsches Passwort für den JKS-Keystore (oder die Datei ist beschädigt)** — geben Sie das richtige Passwort im Formular ein.
- **Der Client-Schlüssel passt nicht zum Client-Zertifikat** — die Schlüsseldatei gehört zu einem anderen Zertifikat.
- Das Client-Zertifikat ist abgelaufen oder noch nicht gültig — die Meldung nennt das Datum. Fordern Sie ein neues Zertifikat an.
- Der PKCS#12-Keystore verwendet einen veralteten Algorithmus, oder der Keystore liegt im JCEKS-Format vor. Konvertieren Sie die Datei wie unter [Zertifikatsformate](connections.md#certificate-formats) beschrieben.

<a id="sasl-errors"></a>

## SASL-Fehler

**Authentifizierung fehlgeschlagen** bedeutet, dass der Broker die SASL-Anmeldung nicht akzeptiert hat. Prüfen Sie:

- Benutzername und Passwort. Beim Bearbeiten einer gespeicherten Verbindung behält ein leeres Passwortfeld das gespeicherte Passwort bei. Geben Sie das Passwort erneut ein, wenn Sie nicht sicher sind, dass es stimmt;
- **SASL-Mechanismus**. Es muss der Mechanismus sein, für den Ihr Benutzer angelegt wurde. SCRAM-Zugangsdaten werden für `SCRAM-SHA-256` und `SCRAM-SHA-512` getrennt angelegt, und `PLAIN`-Benutzer werden wiederum separat konfiguriert;
- das Protokoll: `SASL_SSL` und `SASL_PLAINTEXT` verwenden meist unterschiedliche Ports.

**Das SASL-Passwort für die Verbindung ist nicht festgelegt** — für diese Verbindung ist kein Passwort gespeichert, etwa nach einem Import. Öffnen Sie die Verbindung, geben Sie das Passwort ein und speichern Sie. Siehe [Passwörter](connections.md#passwords).

**Kein Zugriff auf den Schlüsselbund des Systems** — die App kann das Passwort nicht aus dem Schlüsselbund des Systems lesen. Erlauben Sie unter macOS den Zugriff, wenn der Schlüsselbund danach fragt. Prüfen Sie unter Linux, ob GNOME Keyring, KWallet oder ein anderer Secret-Service-Anbieter in Ihrer Sitzung läuft.

**Keine Berechtigung für diesen Vorgang** ist etwas anderes: Die Anmeldung war erfolgreich, aber der Kafka-Benutzer hat keine Rechte (ACLs) für diesen Vorgang, etwa für das Lesen eines Topics oder einer Gruppe. Bitten Sie Ihre Administratoren um die Rechte. Beachten Sie, dass Kafka für Topics, die Sie nicht beschreiben dürfen, keinen Fehler meldet: Sie fehlen einfach in der Topic-Liste.

<a id="timeouts"></a>

## Timeouts

**Zeitüberschreitung beim Vorgang** bedeutet, dass der Broker nicht rechtzeitig geantwortet hat. Typische Ursachen sind ein langsames Netzwerk oder VPN, ein überlasteter Cluster oder nicht erreichbare Broker-Adressen, siehe [Keine Verbindung möglich](troubleshooting.md#cannot-connect).

Ein TLS- oder SASL-Problem kann wie ein Timeout aussehen. Die App versucht, die eigentliche Ursache zu finden und stattdessen anzuzeigen, aber das gelingt nicht immer. Klappen Sie **Details** auf: Meldungen wie `SSL handshake failed` oder `SASL authentication error` weisen auf die Ursache hin.

Ist das Netzwerk einfach langsam, erhöhen Sie **Anfrage-Timeout, ms** im Verbindungsformular. Standard ist `30000` ms. Dieser Wert gilt für Anfragen an die Broker, für das Senden von Nachrichten und für das Lesen. Der Verbindungstest verwendet eigene Grenzen (5 Sekunden pro TCP-Adresse und 10 Sekunden für den Kafka-Schritt), ein größerer Wert ändert den Test also nicht.

<a id="read-only-blocked"></a>

## Eine Aktion ist nicht verfügbar oder wird abgelehnt

Ist eine Schaltfläche ausgeblendet oder deaktiviert oder wird eine Aktion abgelehnt, prüfen Sie diese Ursachen:

- Die Verbindung ist schreibgeschützt. Ein Schloss-Symbol neben ihrem Namen kennzeichnet sie. Schaltflächen, die den Cluster verändern, sind ausgeblendet oder deaktiviert. Erreicht eine solche Anfrage trotzdem den Kern der App, etwa nachdem der Modus bei geöffnetem Bildschirm aktiviert wurde, meldet der Fehler, dass die Verbindung schreibgeschützt ist. Um Änderungen vorzunehmen, öffnen Sie die Verbindung, deaktivieren **Schreibgeschützt** und speichern. Siehe [Schreibgeschützt](connections.md#read-only).
- Es ist eine Prod-Verbindung. Änderungen verlangen die Eingabe des Verbindungsnamens. Solange der Name nicht übereinstimmt, wird die Aktion nicht ausgeführt. Siehe [Umgebungen und Prod](connections.md#environments).
- Die Consumer Group hat aktive Mitglieder. Offsets lassen sich nur zurücksetzen und eine Gruppe nur löschen, wenn sie keine Mitglieder hat. Siehe [Offsets zurücksetzen](groups.md#reset-offsets).
- Die Einstellung wird von Ihrer Organisation verwaltet. Eine Richtlinie kann die Sprache der Oberfläche und die Suche nach Updates festlegen; diese lassen sich in der App nicht ändern. Siehe [Von Ihrer Organisation festgelegte Einstellungen](getting-started.md#managed-settings).
- Kafka hat den Vorgang mit **Keine Berechtigung für diesen Vorgang** abgelehnt. Das ist eine Frage der Rechte im Cluster, nicht der Einstellungen von Glafira. Siehe [SASL-Fehler](troubleshooting.md#sasl-errors).

<a id="logs"></a>

## Protokolle

Die App protokolliert ihre Arbeit und die Fehler des Kafka-Clients. Der genaue Ordner steht auf dem Bildschirm **Über Glafira** unter **Protokolle:** Standardmäßig:

- macOS: `~/Library/Logs/io.github.vvnovg.glafira.kafka`
- Windows: `%LOCALAPPDATA%\io.github.vvnovg.glafira.kafka\logs`
- Linux: `~/.local/share/io.github.vvnovg.glafira.kafka/logs`

Die aktuelle Datei ist `app.log`. Erreicht sie 5 MB, wird sie in `app.1.log` umbenannt, und ältere Dateien rücken zu `app.2.log` usw. auf. Die App behält bis zu fünf Dateien und löscht die älteste.

Passwörter werden nicht protokolliert. Broker-Adressen und die Texte von Kafka-Fehlern werden protokolliert. Sie müssen die Dateien nicht von Hand sammeln: Das Diagnosearchiv enthält sie, siehe [Diagnosearchiv](updates.md#diagnostics).

<a id="support"></a>

## Support kontaktieren

Bevor Sie den Support kontaktieren, führen Sie **Verbindung testen** aus und notieren, welcher Schritt fehlschlägt. Senden Sie die Anfrage dann an die Stelle, die Glafira bei Ihnen betreut, etwa Ihre Administratoren. Fügen Sie Folgendes bei:

- das Diagnosearchiv, siehe [Diagnosearchiv](updates.md#diagnostics). Es enthält Version, Build, Protokolle und Verbindungsprofile ohne Passwörter;
- was Sie getan haben, was Sie erwartet haben und was passiert ist, Schritt für Schritt;
- den vollständigen Fehlertext zusammen mit dem Inhalt von **Details**. Sie können diesen Text markieren und kopieren;
- einen Screenshot, wenn das Problem in der Oberfläche liegt;
- den Zeitpunkt des Fehlers, um ihn in den Protokollen zu finden.

Können Sie das Archiv nicht erstellen, geben Sie die Werte **Version** und **Build** vom Bildschirm **Über Glafira** an und hängen Sie die Protokolldateien aus dem Protokollordner an.
