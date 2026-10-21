# Clusterprüfung und Anmerkungen

Glafira prüft einen Cluster mit integrierten Regeln und zeigt die gefundenen Probleme direkt bei den Objekten: im Verbindungsbaum, in den Listen der Topics und Consumer Groups und auf den Bildschirmen von Topics und Gruppen. Ein LLM-Assistent kann diese Befunde um eigene Anmerkungen ergänzen. Die Prüfung liest den Cluster nur und ändert nichts daran.

<a id="run"></a>

## Prüfung starten

Öffnen Sie bei einer Verbindung **Cluster-Übersicht** oder **Befunde** und klicken Sie auf **Cluster prüfen**. Eine Prüfung dauert bis zu 45 Sekunden: Glafira liest Broker, Partitionen, Topic-Konfigurationen (bis zu 2000 Topics) und Consumer Groups (bis zu 500). Gibt es mehr Topics oder Gruppen, lässt sich etwas nicht lesen oder sind die 45 Sekunden abgelaufen, wird das Ergebnis als **teilweise geprüft** markiert, und der Befund **Unvollständige Prüfung** zeigt, was nicht gelesen wurde.

Glafira fragt den Cluster nicht von selbst ab: Eine Prüfung läuft nur, wenn Sie auf die Schaltfläche klicken oder ein LLM-Assistent sie anfordert. Das Ergebnis bleibt bis zur nächsten Prüfung gespeichert.

<a id="rules"></a>

## Was geprüft wird

- Broker und Partitionen: kein Controller, nicht verfügbare und unterreplizierte Partitionen, ein Cluster mit nur einem Broker, ungleich verteilte Leader, Leader nicht auf dem bevorzugten Replikat.
- Haltbarkeit von Topics: Replikationsfaktor 1, `min.insync.replicas` über dem Replikationsfaktor, `min.insync.replicas` gleich dem Replikationsfaktor, `min.insync.replicas` gleich 1, Unclean Leader Election.
- Broker-Konfiguration: automatisches Erstellen von Topics, Standard-Replikationsfaktor 1, Standardwert von `min.insync.replicas` 1, wichtige Einstellungen, die sich zwischen Brokern unterscheiden.
- Consumer Groups: leere Gruppen mit Lag, Gruppen im Rebalance, Mitglieder ohne Partitionen, Lag, der seit der letzten Prüfung gewachsen ist.

<a id="severity"></a>

## Schweregrad und Umgebung

Jeder Befund hat einen Schweregrad: **Kritisch**, **Warnung** oder **Hinweis**. Er hängt von der Umgebung der Verbindung ab. Replikationsfaktor 1 ist zum Beispiel auf Prod kritisch, auf Stage eine Warnung und wird auf Dev, Test und Local nicht angezeigt. Die Umgebung legen Sie im Verbindungsformular fest.

<a id="tab"></a>

## Der Tab „Befunde“

Im Tab **Befunde** sind die Befunde nach Regeln gruppiert; jeder erklärt das Problem und was zu tun ist. Filtern Sie nach **Schweregrad**, **Quelle** und **Objekt** oder suchen Sie nach Namen. Der Objektname öffnet dessen Bildschirm.

Ein Badge mit einer Zahl im Verbindungsbaum und in den Listen der Topics und Gruppen zeigt, wie viele Befunde und offene LLM-Anmerkungen mit dem Schweregrad **Kritisch** oder **Warnung** das Objekt betreffen. Zeigen Sie mit der Maus auf das Badge, um ihre Titel zu sehen.

<a id="hide"></a>

## Befunde ausblenden

Ist ein Befund für ein bestimmtes Objekt zu erwarten, etwa ein Test-Topic mit Replikationsfaktor 1 auf Prod, klicken Sie auf **Für dieses Objekt ausblenden**. Ein ausgeblendeter Befund zählt nicht in den Badges und bleibt auch nach späteren Prüfungen ausgeblendet. Um ausgeblendete Befunde zu sehen, aktivieren Sie „Ausgeblendete anzeigen“; dort finden Sie **Wieder einblenden**.

<a id="notes"></a>

## LLM-Anmerkungen

Ein über MCP verbundener LLM-Assistent (siehe [LLM-Assistenten (MCP)](llm-mcp.md#how-it-works)) kann eine Anmerkung zum Cluster, zu einem Broker, einem Topic oder einer Gruppe hinterlassen. Anmerkungen erscheinen unter **LLM-Anmerkungen** und auf den Bildschirmen der Objekte, versehen mit dem Namen des Clients. Der Text wird unverändert angezeigt, ohne Formatierung und ohne Links.

- **Erledigt** und **Nicht relevant** schließen eine Anmerkung; geschlossene bleiben in einem eingeklappten Abschnitt.
- **Löschen** entfernt eine Anmerkung, **Alle LLM-Anmerkungen löschen** entfernt alle Anmerkungen der Verbindung.
- Eine Anmerkung wird 7 Tage nach dem Erstellen automatisch entfernt. Eine Verbindung kann höchstens 200 offene Anmerkungen haben.

<a id="with-llm"></a>

## Mit einem LLM-Assistenten

Der Assistent führt dieselbe Prüfung mit dem Tool `health_check` aus und sieht dieselben Befunde, auch ausgeblendete, die markiert sind. Ist die letzte Prüfung jünger als 5 Minuten, erhält der Assistent ihr Ergebnis, ohne den Cluster erneut abzufragen. Bitten Sie zum Beispiel: „Prüfe den Zustand des Clusters und hinterlasse Anmerkungen zu den wichtigsten Problemen.“
