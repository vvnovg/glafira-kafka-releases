# Primi passi

Glafira for Apache Kafka è un client desktop per i cluster Kafka. Permette di consultare topic e consumer group, leggere e inviare messaggi e reimpostare gli offset. Questo capitolo tratta l’installazione, il primo avvio, un Kafka locale per fare pratica e la lingua dell’interfaccia.

<a id="install-macos"></a>

## Installazione su macOS

Glafira funziona su macOS 11 o versioni successive. Scarica il programma di installazione dalla pagina delle release: https://github.com/vvnovg/glafira-kafka-releases/releases

Scegli il file adatto al tuo Mac: `aarch64` per Apple silicon (M1 e successivi), `x64` per Intel.

1. Apri il file `.dmg`.
2. Trascina Glafira for Apache Kafka nella cartella Applicazioni.
3. Avvia l’app da Applicazioni o da Launchpad.

Puoi usare anche il programma di installazione `.pkg`, che installa l’app in `/Applications` per tutti gli utenti.

Le build sono firmate da Apple e notarizzate, quindi l’app si apre subito, senza avvisi di sicurezza.

Se macOS segnala comunque che l’app è danneggiata o non può essere verificata, il download è corrotto: scarica di nuovo il `.dmg` o il `.pkg` dalla pagina delle release.

<a id="install-windows"></a>

## Installazione su Windows

Scarica il programma di installazione `.msi` (`x64`) dalla pagina delle release: https://github.com/vvnovg/glafira-kafka-releases/releases

1. Esegui il file `.msi`.
2. Conferma l’installazione nella richiesta di Windows. L’app viene installata per tutti gli utenti del computer, quindi servono i diritti di amministratore.
3. Avvia Glafira for Apache Kafka dal menu Start.

Le build beta non sono ancora firmate. Se SmartScreen blocca il programma di installazione, fai clic su “Ulteriori informazioni” e poi su “Esegui comunque”. Puoi anche sbloccare il file in PowerShell prima di eseguirlo:

```powershell
Unblock-File -Path "$env:USERPROFILE\Downloads\Glafira_for_Apache_Kafka_*_x64_en-US.msi"
```

<a id="install-linux"></a>

## Installazione su Linux

Le build sono disponibili per `x86_64` in tre formati. Scarica quello adatto alla tua distribuzione dalla pagina delle release: https://github.com/vvnovg/glafira-kafka-releases/releases

- Debian, Ubuntu e derivate: `sudo apt install ./<file>.deb`
- Fedora, RHEL e derivate: `sudo dnf install ./<file>.rpm`
- Qualsiasi distribuzione: rendi eseguibile l’AppImage con `chmod +x <file>.AppImage` e avviala.

Dopo aver installato un pacchetto, avvia l’app dal menu delle applicazioni o esegui `glafira-kafka` in un terminale.

Glafira conserva le password nel Secret Service (per esempio GNOME Keyring o KWallet). Assicurati che uno di questi servizi sia in esecuzione nella tua sessione, altrimenti l’app non può salvare le password delle connessioni.

<a id="first-run"></a>

## Primo avvio

La finestra ha due parti. A sinistra c’è il pannello **Connessioni**. A destra ci sono le schede: ogni schermata che apri, come un elenco di topic o il Producer, ha una scheda propria.

Finché non ci sono connessioni, il pannello mostra un breve suggerimento e l’area delle schede propone di aggiungere una connessione. Per iniziare:

1. Fai clic su `+` ( **Nuova connessione** ) nell’intestazione del pannello **Connessioni**.
2. Compila i campi **Nome** e **Server bootstrap**, per esempio `localhost:9092`.
3. Fai clic su **Prova connessione** per verificare le impostazioni, poi su **Salva**.

Dopo il salvataggio si apre la panoramica del cluster. Sotto ogni connessione del pannello ci sono i collegamenti **Panoramica del cluster**, **Topic**, **Consumer group**, **Producer** e **Sessione consumer**. Tutte le impostazioni della connessione sono descritte in [Connessioni](connections.md#create).

In fondo al pannello ci sono i collegamenti **Impostazioni**, **Sostieni il progetto**, **Aiuto** e **Informazioni**. Il simbolo `?` accanto a un campo mostra un breve suggerimento. Premi `F1` in qualsiasi punto della finestra per aprire il capitolo della guida relativo alla schermata corrente. Su macOS la guida si trova anche nel menu Aiuto.

Se usavi Kafka Explorer (il nome precedente dell’app), al primo avvio Glafira ne copia le connessioni, i modelli e la cronologia degli invii. Anche le password vengono copiate dall’archivio dei segreti di sistema. Nulla di ciò che esiste già viene sovrascritto e i vecchi dati non vengono eliminati. macOS può chiedere una volta l’accesso ai vecchi elementi del Portachiavi.

<a id="local-cluster"></a>

## Kafka locale per fare pratica

Per provare l’app senza un cluster condiviso, avvia Kafka sul tuo computer in Docker. Basta un broker:

```bash
docker run -d --name kafka -p 9092:9092 apache/kafka:3.9.0
```

Poi crea una connessione con server bootstrap `localhost:9092`, protocollo `PLAINTEXT` e ambiente `local`. Quando il broker non ti serve più, rimuovilo con `docker rm -f kafka`.

Il suggerimento nel pannello **Connessioni** vuoto cita `docker compose -f dev/docker-compose.yml up -d`. Questo file fa parte del codice sorgente di Glafira. Se hai il codice sorgente, il comando avvia un broker con tre listener:

- `localhost:9092` — `PLAINTEXT`.
- `localhost:9094` — `SASL_PLAINTEXT` con `PLAIN` (utenti `tester` / `tester-secret` e `admin` / `admin-secret`) e SCRAM (`scram-user` / `scram-secret` per `SCRAM-SHA-256` e `SCRAM-SHA-512`).
- `localhost:9095` — `SSL` con certificati client (mTLS). I certificati di prova si trovano in `dev/certs/out`.

Questi utenti e certificati servono solo per i test locali.

<a id="language"></a>

## Lingua dell’interfaccia

L’interfaccia è disponibile in inglese, russo, tedesco, francese, spagnolo, italiano, polacco, portoghese (Brasile), cinese (semplificato e tradizionale), giapponese e coreano. Tutte le lingue sono gratuite.

Per cambiare lingua:

1. Fai clic su **Impostazioni** in fondo al pannello sinistro.
2. Scegli una lingua in **Lingua dell’interfaccia**.

La modifica si applica subito, anche a questa guida e al menu di macOS. **Lingua di sistema** (l’opzione predefinita) segue la lingua del sistema operativo. Se quella lingua non è nell’elenco, l’interfaccia è in inglese.

<a id="managed-settings"></a>

## Impostazioni definite dalla tua organizzazione

In un’azienda gli amministratori possono impostare alcune opzioni per tutti gli utenti tramite una policy. Queste opzioni non si possono cambiare nell’app. Una policy può:

- impostare la lingua dell’interfaccia — l’elenco delle lingue in **Impostazioni** non è allora disponibile e una nota indica che la lingua è impostata dalla tua organizzazione;
- disattivare la verifica degli aggiornamenti o indirizzarla a un mirror interno.

Per sapere se è applicata una policy, apri **Informazioni** e guarda la voce **Policy dell’organizzazione**. Se la policy contiene un errore, questo compare lì. Per domande sulla policy, rivolgiti ai tuoi amministratori. Gli aggiornamenti sono descritti in [Aggiornamenti e diagnostica](updates.md).
