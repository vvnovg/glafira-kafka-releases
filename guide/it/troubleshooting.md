# Risoluzione dei problemi

Questo capitolo spiega gli errori di connessione più comuni e che cosa controllare per ciascuno. Indica anche dove si trovano i log e che cosa includere in una richiesta di assistenza. Su macOS, il menu Aiuto apre questo capitolo con **Risoluzione dei problemi**.

Un messaggio di errore ha un breve testo e, nella maggior parte delle schermate, il codice dell’errore tra parentesi, per esempio `TIMEOUT`. Espandi **Dettagli** per vedere il messaggio originale del client Kafka (librdkafka). Spesso indica l’indirizzo esatto del broker e la causa.

<a id="cannot-connect"></a>

## Impossibile connettersi

Inizia con **Prova connessione** nel modulo della connessione, vedi [Provare la connessione](connections.md#test). La prova si ferma al primo passaggio non riuscito, e il passaggio indica dove cercare:

- **Configurazione** — il problema è nel modulo stesso: un indirizzo senza porta, un file di certificato mancante o illeggibile, una password del certificato errata. L’errore indica il campo. Per i file dei certificati, vedi [Errori TLS](troubleshooting.md#tls-errors).
- **DNS / TCP** — il computer non riesce a raggiungere i server bootstrap. Vedi [DNS e TCP](troubleshooting.md#dns-tcp).
- **Kafka (TLS, SASL, metadati)** — la rete funziona, ma l’handshake TLS, l’accesso SASL o la richiesta dei metadati non sono riusciti. Vedi [Errori TLS](troubleshooting.md#tls-errors), [Errori SASL](troubleshooting.md#sasl-errors) e [Timeout](troubleshooting.md#timeouts).

La causa più comune è un protocollo che non corrisponde alla porta. Ogni listener del broker ha una porta e un protocollo propri. Chiedi ai tuoi amministratori Kafka quale porta corrisponde a quale protocollo, vedi [Protocolli di sicurezza](connections.md#security).

Se la prova riesce ma la lettura o l’invio non funzionano, la causa sono spesso gli indirizzi dei broker. Il client ottiene l’elenco dei broker dal server bootstrap e poi si connette a ciascun broker all’indirizzo che il broker pubblica (`advertised.listeners`). Se questi indirizzi sono interni, per esempio nomi host di Docker o Kubernetes, non sono raggiungibili dal tuo computer. Gli indirizzi sono elencati nella tabella dei broker della panoramica del cluster, vedi [Panoramica del cluster](topics.md#cluster-overview). Chiedi ai tuoi amministratori un listener raggiungibile dall’esterno.

<a id="dns-tcp"></a>

## DNS e TCP

**Nessuno dei server bootstrap è raggiungibile tramite TCP** significa che l’app non è riuscita ad aprire una connessione TCP verso nessuno degli indirizzi in **Server bootstrap**. Per ogni indirizzo l’attesa massima è di 5 secondi. I dettagli mostrano l’ultimo indirizzo provato e il motivo. `DNS` nei dettagli significa che il nome host non è stato risolto.

Controlla:

- il nome host e la porta, senza un prefisso di protocollo come `https://`;
- che la VPN sia attiva, se il cluster si trova in una rete privata;
- che un firewall non blocchi la porta;
- per un broker in Docker, che la porta sia pubblicata sull’host, per esempio `-p 9092:9092`.

Per verificare la porta al di fuori dell’app, esegui `nc -vz <host> <port>` su macOS o Linux, oppure `Test-NetConnection <host> -Port <port>` in PowerShell su Windows.

**Impossibile connettersi ai broker** dopo che il passaggio TCP è riuscito significa di solito che un broker ha chiuso la connessione o non è stato raggiungibile in seguito. Controlla anche gli indirizzi dei broker, vedi [Impossibile connettersi](troubleshooting.md#cannot-connect).

<a id="tls-errors"></a>

## Errori TLS

Errori nel passaggio **Kafka (TLS, SASL, metadati)** durante l’handshake TLS:

- **Il broker accetta connessioni senza TLS su questo indirizzo — controlla la porta o scegli un protocollo senza SSL**.
- **Il certificato del broker non è firmato da una CA attendibile — controlla “CA attendibile”**. In **Certificato CA attendibile** indica il file della CA che ha firmato i certificati dei broker. I tuoi amministratori possono fornirti questo file. Con il campo vuoto, sono considerati attendibili solo i certificati radice di sistema.
- **Il nome host del broker non corrisponde al suo certificato**. Connettiti con il nome per cui è stato emesso il certificato, non con un indirizzo IP o un alias. Solo negli ambienti di test puoi disattivare **Verifica nome host**.
- **Il broker ha rifiutato il certificato client**. Il broker richiede un certificato client, oppure non considera attendibile quello che usi. Verifica che **Certificato client (mTLS)** sia attivo e che il certificato sia stato emesso per questo cluster. Vedi [Certificato client (mTLS)](connections.md#mtls).
- **Errore di connessione TLS** — un altro problema TLS. Espandi **Dettagli** per conoscerne la causa.

Se hai scelto un protocollo senza SSL e vedi comunque un errore TLS, probabilmente la porta appartiene a un listener TLS. In tal caso i dettagli riportano `connecting to a SSL listener?`. Scegli `SSL` o `SASL_SSL`, oppure usa la porta di un listener senza TLS.

I problemi con i file dei certificati vengono segnalati nel passaggio **Configurazione**, prima che l’app si connetta. Il messaggio inizia con il nome del campo: CA attendibile, certificato client o chiave client. Casi comuni:

- Il percorso non è assoluto o il file non si può leggere. Inserisci il percorso completo o scegli il file con **Sfoglia…**.
- **Chiave client: la chiave è cifrata — specifica la password della chiave**, **Chiave client: password della chiave errata**, **Certificato client: password dell’archivio PKCS\#12 errata** o **Password dell’archivio JKS errata (o il file è danneggiato)** — inserisci la password corretta nel modulo.
- **La chiave client non corrisponde al certificato client** — il file della chiave appartiene a un altro certificato.
- Il certificato client è scaduto o non è ancora valido — il messaggio indica la data. Richiedi un nuovo certificato.
- L’archivio PKCS#12 usa un algoritmo obsoleto, oppure l’archivio è in formato JCEKS. Converti il file come indicato in [Formati dei certificati](connections.md#certificate-formats).

<a id="sasl-errors"></a>

## Errori SASL

**Autenticazione non riuscita** significa che il broker non ha accettato l’accesso SASL. Controlla:

- il nome utente e la password. Quando modifichi una connessione salvata, un campo password vuoto mantiene la password salvata. Digita di nuovo la password se non sei sicuro che sia corretta;
- **Meccanismo SASL**. Deve essere il meccanismo per cui è stato creato il tuo utente. Le credenziali SCRAM vengono create separatamente per `SCRAM-SHA-256` e `SCRAM-SHA-512`, e gli utenti `PLAIN` vengono configurati a loro volta separatamente;
- il protocollo: `SASL_SSL` e `SASL_PLAINTEXT` usano di solito porte diverse.

**La password SASL della connessione non è impostata** — per questa connessione non è salvata alcuna password, per esempio dopo un’importazione. Apri la connessione, inserisci la password e salva. Vedi [Password](connections.md#passwords).

**Impossibile accedere all’archivio dei segreti di sistema** — l’app non riesce a leggere la password dall’archivio dei segreti di sistema. Su macOS, consenti l’accesso quando il Portachiavi lo chiede. Su Linux, verifica che GNOME Keyring, KWallet o un altro provider Secret Service sia in esecuzione nella tua sessione.

**Non sei autorizzato a eseguire l’operazione** è un caso diverso: l’accesso è riuscito, ma l’utente Kafka non ha i diritti (ACL) per questa operazione, per esempio per leggere un topic o un gruppo. Chiedi i diritti ai tuoi amministratori. Tieni presente che Kafka non segnala alcun errore per i topic che non hai il permesso di descrivere: semplicemente non compaiono nell’elenco dei topic.

<a id="timeouts"></a>

## Timeout

**Timeout dell’operazione scaduto** significa che il broker non ha risposto in tempo. Le cause tipiche sono una rete o una VPN lenta, un cluster sovraccarico o indirizzi dei broker non raggiungibili, vedi [Impossibile connettersi](troubleshooting.md#cannot-connect).

Un problema TLS o SASL può sembrare un timeout. L’app cerca di trovare la causa reale e di mostrarla al suo posto, ma non sempre è possibile. Espandi **Dettagli**: messaggi come `SSL handshake failed` o `SASL authentication error` indicano la causa.

Se la rete è semplicemente lenta, aumenta **Timeout della richiesta, ms** nel modulo della connessione. Il valore predefinito è `30000` ms. Questo valore si applica alle richieste ai broker, all’invio dei messaggi e alla lettura. La prova della connessione usa limiti propri (5 secondi per ogni indirizzo TCP e 10 secondi per il passaggio Kafka), quindi un valore più alto non cambia la prova.

<a id="read-only-blocked"></a>

## Un’azione non è disponibile o viene rifiutata

Se un pulsante è nascosto o disattivato, o un’azione viene rifiutata, controlla queste cause:

- La connessione è in sola lettura. La contrassegna un’icona a forma di lucchetto accanto al nome. I pulsanti che modificano il cluster sono nascosti o disattivati. Se una richiesta di questo tipo arriva comunque al core dell’app, per esempio dopo che la modalità è stata attivata mentre una schermata era aperta, l’errore indica che la connessione è in sola lettura. Per apportare modifiche, apri la connessione, disattiva l’opzione **Sola lettura** e salva. Vedi [Sola lettura](connections.md#read-only).
- La connessione è prod. Le modifiche chiedono di digitare il nome della connessione. Finché il nome non corrisponde, l’azione non viene eseguita. Vedi [Ambienti e prod](connections.md#environments).
- Il consumer group ha membri attivi. Gli offset si possono reimpostare e un gruppo si può eliminare solo quando non ha membri. Vedi [Reimpostare gli offset](groups.md#reset-offsets).
- L’impostazione è gestita dalla tua organizzazione. Una policy può impostare la lingua dell’interfaccia e la verifica degli aggiornamenti, e queste impostazioni non si possono cambiare nell’app. Vedi [Impostazioni definite dalla tua organizzazione](getting-started.md#managed-settings).
- Kafka ha rifiutato l’operazione con **Non sei autorizzato a eseguire l’operazione**. È una questione di diritti nel cluster, non di impostazioni di Glafira. Vedi [Errori SASL](troubleshooting.md#sasl-errors).

<a id="logs"></a>

## Log

L’app scrive un log della propria attività e degli errori del client Kafka. La cartella esatta è indicata nella schermata **Informazioni**, alla voce **Log:** Per impostazione predefinita:

- macOS: `~/Library/Logs/io.github.vvnovg.glafira.kafka`
- Windows: `%LOCALAPPDATA%\io.github.vvnovg.glafira.kafka\logs`
- Linux: `~/.local/share/io.github.vvnovg.glafira.kafka/logs`

Il file corrente è `app.log`. Quando raggiunge 5 MB, viene rinominato in `app.1.log`, e i file più vecchi passano a `app.2.log` e così via. L’app conserva fino a cinque file ed elimina il più vecchio.

Le password non vengono scritte nei log. Vengono invece scritti gli indirizzi dei broker e i testi degli errori di Kafka. Non serve raccogliere i file a mano: l’archivio diagnostico li include, vedi [Archivio diagnostico](updates.md#diagnostics).

<a id="support"></a>

## Contattare l’assistenza

Prima di contattare l’assistenza, esegui **Prova connessione** e annota quale passaggio non riesce. Poi invia la richiesta a chi si occupa dell’assistenza su Glafira per te, per esempio i tuoi amministratori. Includi:

- l’archivio diagnostico, vedi [Archivio diagnostico](updates.md#diagnostics). Contiene la versione, la build, i log e i profili delle connessioni senza password;
- che cosa hai fatto, che cosa ti aspettavi e che cosa è successo, passo per passo;
- il testo completo dell’errore, insieme al contenuto di **Dettagli**. Puoi selezionare e copiare questo testo;
- uno screenshot, se il problema riguarda l’interfaccia;
- l’ora in cui si è verificato l’errore, per trovarlo nei log.

Se non riesci a creare l’archivio, indica i valori di **Versione** e **Build** della schermata **Informazioni** e allega i file di log dalla cartella dei log.
