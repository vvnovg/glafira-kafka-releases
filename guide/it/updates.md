# Aggiornamenti e diagnostica

Questo capitolo tratta come aggiornare l’app, la schermata **Informazioni**, l’archivio diagnostico e come sostenere il progetto.

<a id="updates"></a>

## Aggiornamenti

All’avvio, l’app verifica una volta se è disponibile una nuova versione. Se c’è, un avviso sopra le schede ne indica la versione. **Dettagli** apre la schermata **Informazioni**. Se la verifica non riesce, per esempio senza accesso a internet, non compare alcun messaggio.

Per verificare manualmente:

1. Apri **Informazioni** in fondo al pannello sinistro.
2. In **Aggiornamenti**, fai clic su **Verifica aggiornamenti**.
3. Se è disponibile una nuova versione, la schermata ne mostra il numero e le note di rilascio. Fai clic su **Installa e riavvia**.

L’app scarica l’aggiornamento, ne verifica la firma, lo installa e si riavvia. Un download interrotto da un problema di rete viene ripetuto automaticamente, fino a tre tentativi in totale. Se tutti e tre i tentativi falliscono, l’app segnala **Il download dell’aggiornamento è stato interrotto — controlla la rete e riprova**. Se la verifica della firma non riesce, l’app segnala **La firma dell’aggiornamento non ha superato la verifica — l’aggiornamento non è stato installato**.

L’installazione può chiedere una password di amministratore, perché l’app è installata per tutti gli utenti del computer: dal programma di installazione `.pkg` su macOS, dal programma di installazione `.msi` su Windows o da un pacchetto `.deb` o `.rpm` su Linux. Gli utenti senza diritti di amministratore non possono aggiornare un’installazione di questo tipo.

Se è in corso la lettura di messaggi, una sessione consumer o un invio in batch, l’app chiede prima conferma e indica quante sessioni sono attive. Fai clic su **Interrompi le sessioni e aggiorna** per interromperle e continuare, oppure su **Annulla**. Connessioni, impostazioni e altri dati vengono conservati dopo un aggiornamento.

Gli aggiornamenti vengono scaricati dalla pagina delle release su GitHub. Se la verifica non riesce con **Verifica degli aggiornamenti non riuscita**, controlla la rete o il proxy. **Gli aggiornamenti non sono configurati in questa build** significa che questa build è stata creata senza la chiave del programma di aggiornamento, per esempio una build di sviluppo o di test. In tal caso scarica le nuove versioni dalla pagina delle release: https://github.com/vvnovg/glafira-kafka-releases/releases

Un’organizzazione può disattivare la verifica degli aggiornamenti tramite una policy, oppure indirizzarla a un mirror interno. Quando la verifica è disattivata, la sezione **Aggiornamenti** indica **Gli aggiornamenti sono gestiti dalla tua organizzazione** dopo il clic su **Verifica aggiornamenti**. Come **Gli aggiornamenti non sono configurati in questa build**, questo messaggio compare solo dopo il clic; in questi casi la verifica all’avvio non mostra nulla.

<a id="about"></a>

## Informazioni

Apri **Informazioni** in fondo al pannello sinistro. Su macOS la schermata si trova anche nel menu Aiuto come **Informazioni su Glafira**.

La schermata mostra una breve descrizione dell’app e:

- **Versione** — il numero di versione, e **Build** — l’identificatore del codice sorgente da cui è stata compilata l’app. Indica entrambi in una richiesta di assistenza.
- **Sistema** — il sistema operativo e l’architettura del processore.
- **Licenza** — Apache 2.0; il link apre il testo della licenza nel browser.
- **Policy dell’organizzazione** — **impostata** o **nessuna**. Gli errori della policy sono elencati sotto.
- **Dati:** — la cartella con i profili delle connessioni, i modelli, la cronologia e le impostazioni.
- **Log:** — la cartella con i file di log. Vedi [Log](troubleshooting.md#logs).

In questa schermata si trovano anche le sezioni **Aggiornamenti** e **Diagnostica**.

<a id="diagnostics"></a>

## Archivio diagnostico

L’archivio diagnostico raccoglie ciò che serve all’assistenza per capire un problema. Per crearlo:

1. Apri **Informazioni**.
2. In **Diagnostica**, fai clic su **Raccogli diagnostica**. Su macOS puoi anche scegliere **Raccogli diagnostica…** nel menu Aiuto.
3. Scegli dove salvare il file. Il nome proposto è `diagnostics-<date>.zip`.

Una volta salvato l’archivio, la schermata ne mostra il percorso. Il file ZIP contiene:

- `info.json` — la versione e la build dell’app, il sistema operativo, i percorsi delle cartelle dei dati e dei log, e lo stato della policy;
- `connections.json` — i profili delle connessioni: nomi, ambienti, server bootstrap, protocolli, nomi utente SASL, percorsi dei file dei certificati, proprietà aggiuntive e timeout;
- `logs/` — i file di log dell’app.

L’archivio non contiene password: sono conservate nell’archivio dei segreti di sistema, e le proprietà aggiuntive con password e segreti vengono rifiutate al salvataggio di una connessione. Non sono inclusi nemmeno i file dei certificati. Nomi host, nomi utente e percorsi dei file sono memorizzati come testo in chiaro, e i log contengono gli indirizzi dei broker presenti nei messaggi di errore di Kafka. Esamina l’archivio prima di inviarlo fuori dalla tua azienda.

L’app non invia nulla da sola. Allega tu l’archivio alla richiesta di assistenza, vedi [Contattare l’assistenza](troubleshooting.md#support).

<a id="support"></a>

## Sostieni il progetto

Glafira è gratuita e open source con licenza Apache 2.0. Le donazioni la fanno vivere. **Sostieni il progetto** in fondo al pannello sinistro, nella schermata **Informazioni** e, su macOS, nel menu Aiuto apre la pagina di sostegno nel browser: https://github.com/vvnovg/glafira-kafka-releases/blob/main/SUPPORT.md
