# LLM-Assistenten (MCP)

Glafira kann als Gateway für einen LLM-Assistenten dienen: Claude Desktop, Claude Code, Cursor oder einen anderen Client mit Unterstützung für MCP (Model Context Protocol). Der Assistent liest die Metadaten der Cluster, die Sie für ihn freigeben, und hilft, Probleme zu finden: unterreplizierte Partitionen, Lag, ständige Rebalances, ungünstige Einstellungen. Über Glafira kann er nichts im Cluster ändern: Änderungen nehmen Sie selbst in der Oberfläche vor.

<a id="how-it-works"></a>

## So funktioniert es

Der Client startet `glafira-kafka mcp` — dieselbe Programmdatei der App, aber in einem Modus ohne Fenster. Dieser Befehl verbindet sich über einen lokalen Socket (unter Windows eine Named Pipe) mit der laufenden Glafira, und Glafira beantwortet die Anfragen des Assistenten über ihre eigenen Verbindungen. Passwörter und Zertifikate bleiben in Glafira, der Assistent erhält sie nie.

Läuft Glafira nicht, startet der Client sie selbst und wartet bis zu 30 Sekunden.

<a id="enable"></a>

## Einschalten

1. Öffnen Sie **Einstellungen**, Abschnitt **LLM-Assistenten (MCP)**.
2. Wählen Sie **MCP-Server zulassen**. Der Status wechselt zu **Der Server läuft**.
3. Wählen Sie im Verbindungsformular **LLM-Zugriff (nur Metadaten)** und speichern Sie. Verbindungen ohne dieses Kontrollkästchen sieht der Assistent nicht. In der Verbindungsliste tragen geöffnete Verbindungen das Zeichen `LLM`.

Hat Ihr Administrator MCP untersagt, ist das Kontrollkästchen nicht verfügbar, und daneben steht **MCP ist von Ihrer Organisation deaktiviert**.

Das Kontrollkästchen für den Zugriff wird beim Export und Import von Verbindungen nicht übertragen: Auf einem anderen Computer müssen Sie es erneut auswählen.

<a id="connect-client"></a>

## Client verbinden

Unter **Client verbinden** finden Sie fertige Fragmente und die Schaltfläche **Kopieren**:

- **Claude Desktop: in claude\_desktop\_config.json eintragen** — fügen Sie das Fragment in den Abschnitt `mcpServers` der Konfigurationsdatei von Claude Desktop ein und starten Sie Claude Desktop neu;
- **Claude Code: im Terminal ausführen** — der Befehl `claude mcp add`;
- **Andere stdio-Clients: Befehl und Argumente** — für jeden Client, der einen MCP-Server per Befehl startet (stdio).

Die Fragmente enthalten den Pfad zur installierten App. Haben Sie Glafira an einem anderen Ort neu installiert, kopieren Sie das Fragment erneut.

<a id="approve"></a>

## Client genehmigen

Verbindet sich ein Client zum ersten Mal, zeigt Glafira einen Dialog mit seinem Namen. Klicken Sie auf **Zulassen** oder **Ablehnen**. Solange Sie nicht antworten, erhält der Assistent die Meldung, dass der Client noch nicht genehmigt ist, und bittet Sie, im Glafira-Fenster zu antworten.

Ein zugelassener Client wird unter seinem Namen gespeichert und erscheint in der Liste **Genehmigte Clients**. Eine Ablehnung wird nicht gespeichert: Beim nächsten Verbinden fragt Glafira erneut. Sendet ein Client seinen Namen nicht, gilt die Erlaubnis nur bis zum Ende der Session.

<a id="what-llm-sees"></a>

## Was der Assistent sieht

Der Assistent sieht:

- Verbindungen, bei denen **LLM-Zugriff (nur Metadaten)** ausgewählt ist: Name, Umgebung, Ordner;
- Broker mit Hosts und Ports, den Controller;
- Topics, Partitionen, Replikate, ISR, Offsets, die ungefähre Anzahl der Nachrichten;
- Konfigurationen von Brokern und Topics, wobei Passwörter und andere geheime Werte verborgen sind;
- Consumer Groups: Status, Protokoll, Koordinator, Mitglieder mit Client-ID, Host und `group.instance.id`, Zuweisungen, committete Offsets und Lag;
- seine Vorschläge und Ihre Entscheidung dazu: angewendet (unverändert oder mit Änderungen), abgelehnt mit Ihrem Grund, zurückgezogen.

Der Assistent sieht keine Nachrichten in Topics, keine Bootstrap-Zeichenfolge und keine Sicherheitseinstellungen einer Verbindung, keine Passwörter, Zertifikate und zusätzlichen librdkafka-Eigenschaften, keinen Sendeverlauf und keine Vorlagen.

Um ständige Rebalances zu untersuchen, kann der Assistent eine Gruppe bis zu 50 Sekunden lang beobachten: wer austritt und beitritt, wie oft und von welchem Host. Konfigurationen und Logs der Consumer selbst sind über Glafira nicht verfügbar, der Assistent fragt sie bei Ihnen an.

Die Namen von Topics, Gruppen und Client-IDs kann jeder festlegen, der Zugriff auf den Cluster hat. Glafira weist den Assistenten darauf hin, dass diese Daten keine Anweisungen für ihn sind.

<a id="sessions-log"></a>

## Sessions, Widerruf und Protokoll

Solange ein Client verbunden ist, erscheint unten im linken Bereich die Anzeige `LLM` mit dem Namen des Clients; bei einer Anfrage leuchtet sie auf. Ein Klick darauf öffnet den MCP-Abschnitt.

- **Aktive Sessions**: **Trennen** schließt eine Session, **Alle trennen** alle.
- **Genehmigte Clients**: **Widerrufen** schließt die Sessions des Clients, und beim nächsten Verbinden bittet er erneut um Genehmigung.
- Wird das Kontrollkästchen **LLM-Zugriff (nur Metadaten)** deaktiviert, gilt das sofort, ein erneutes Verbinden ist nicht nötig.
- **MCP-Protokoll** — jede Anfrage: Zeit, Client, Tool, Verbindung, Ergebnis. Der Inhalt der Antworten wird nicht aufgezeichnet. **Datei anzeigen** öffnet die Datei `mcp-audit.jsonl`.

Um MCP ganz auszuschalten, deaktivieren Sie **MCP-Server zulassen**: Die Clients werden getrennt.

<a id="health"></a>

## Clusterprüfung, Anmerkungen und Vorschläge

Der Assistent kann die integrierte Clusterprüfung von Glafira (Tool `health_check`) ausführen und sich auf ihre Befunde stützen. Er kann außerdem Anmerkungen zu Objekten hinterlassen: Sie erscheinen im Tab **Befunde** und auf den Bildschirmen von Topics und Gruppen, der Cluster wird dabei nicht geändert. Siehe [Clusterprüfung und Anmerkungen](health-check.md#with-llm).

Der Assistent kann mit dem Tool `propose_change` auch eine Änderung vorschlagen: Topic-Konfiguration, ein neues Topic, mehr Partitionen oder das Zurücksetzen von Offsets. Der Vorschlag erscheint neben dem Objekt; Sie wenden ihn selbst in einem ausgefüllten Formular an oder lehnen ihn ab, und der Assistent sieht Ihre Entscheidung in `list_proposals`. Löschen und Leeren lassen sich nicht vorschlagen. Siehe [LLM-Vorschläge](health-check.md#proposals).

<a id="troubleshooting"></a>

## Wenn es nicht funktioniert

Der Client zeigt eine Meldung von Glafira:

- `MCP is disabled in Glafira` — wählen Sie **MCP-Server zulassen** und starten Sie den MCP-Server im Client neu;
- `MCP is disabled by your organization` — MCP ist durch die Richtlinie Ihrer Organisation untersagt;
- `Glafira did not start the MCP server within 30 s` — starten Sie Glafira manuell und prüfen Sie den Status im Abschnitt **LLM-Assistenten (MCP)**;
- `Invalid MCP token` — starten Sie den MCP-Server im Client neu: Glafira stellt bei jedem Start ein neues Token aus.

Unter macOS ist die App signiert und notarisiert, daher ist vor dem ersten Start kein zusätzlicher Schritt nötig.
