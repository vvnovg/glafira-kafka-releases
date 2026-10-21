# Connessioni

Una connessione è il profilo salvato di un cluster Kafka: indirizzi, impostazioni di sicurezza e password. Le connessioni sono elencate nel pannello **Connessioni** a sinistra. Puoi organizzarle in cartelle e trasferirle su un altro computer con l’esportazione e l’importazione.

<a id="create"></a>

## Creare una connessione

1. Fai clic su `+` ( **Nuova connessione** ) nell’intestazione del pannello **Connessioni**. Per creare la connessione direttamente in una cartella, apri il menu della cartella e scegli **Nuova connessione qui**.
2. Inserisci il **Nome**. Compare nel pannello e nei titoli delle schede.
3. In **Server bootstrap** inserisci uno o più indirizzi nel formato `host:port`, separati da virgole, per esempio `kafka1:9092,kafka2:9092`. Basta un broker raggiungibile: il client trova gli altri da solo.
4. Scegli il **Protocollo** e compila i campi che aggiunge. Vedi [Protocolli di sicurezza](connections.md#security).
5. Facoltativamente, scegli la **Cartella**, un **Ambiente** e il **Colore** del pallino accanto al nome.
6. Fai clic su **Prova connessione**, poi su **Salva**.

Dopo il salvataggio il modulo si chiude e si apre la panoramica del cluster.

Altri due campi servono per casi particolari. **Timeout della richiesta, ms** indica quanto attendere la risposta di un broker. Il valore predefinito è `30000` ms, il minimo `1000`. Aumentalo per reti lente e VPN. **Proprietà librdkafka aggiuntive** accetta qualsiasi impostazione di librdkafka nella forma `key=value`, una per riga. Le righe che iniziano con `#` vengono ignorate. Password e chiavi non sono ammesse: le proprietà il cui nome contiene `password` o `secret`, `sasl.jaas.config`, `ssl.key.pem` e `ssl.keystore.key` vengono rifiutate al salvataggio.

Per modificare una connessione in seguito, fai clic su `✎` ( **Modifica connessione** ) accanto al nome. Per eliminarla, apri il modulo, fai clic su **Elimina** e conferma con **Sì, elimina**.

<a id="test"></a>

## Provare la connessione

**Prova connessione** verifica passo per passo le impostazioni del modulo e si ferma al primo passaggio non riuscito:

1. **Configurazione** — gli indirizzi, i file dei certificati, le password e il nome utente SASL.
2. **DNS / TCP** — i nomi host vengono risolti e si apre una connessione TCP verso almeno un server bootstrap.
3. **Kafka (TLS, SASL, metadati)** — l’handshake TLS e l’accesso SASL riescono e il broker restituisce i metadati. In caso di successo, il passaggio mostra il numero di broker e di topic.

La prova usa i valori presenti nel modulo, comprese le password digitate ma non ancora salvate. Non viene salvato nulla e nel cluster non cambia nulla. Se un passaggio non riesce, sotto di esso compaiono il testo dell’errore e i dettagli. Gli errori più comuni sono spiegati in [Impossibile connettersi](troubleshooting.md#cannot-connect).

<a id="security"></a>

## Protocolli di sicurezza

Il protocollo scelto in **Protocollo** deve corrispondere al listener del broker a cui ti connetti. I tuoi amministratori Kafka sanno quale usare.

- `PLAINTEXT` — nessuna cifratura e nessun accesso. Tipico dei cluster locali e di test.
- `SSL` — il traffico è cifrato con TLS. Con un certificato client si tratta di mTLS.
- `SASL_PLAINTEXT` — accesso con nome utente e password, senza cifratura.
- `SASL_SSL` — accesso con nome utente e password su TLS.

Quando cambi protocollo, il modulo mostra solo i campi che servono a quel protocollo.

<a id="sasl"></a>

## SASL

Per `SASL_PLAINTEXT` e `SASL_SSL` compila tre campi:

- **Meccanismo SASL** — `PLAIN`, `SCRAM-SHA-256` o `SCRAM-SHA-512`. È il cluster a stabilire quale funziona.
- **Nome utente**.
- **Password**. Viene salvata nell’archivio dei segreti di sistema, non nel profilo. Vedi [Password](connections.md#passwords).

SCRAM non invia mai la password vera e propria. `PLAIN` la invia così com’è, quindi con `SASL_PLAINTEXT` la password viaggia in chiaro. Quando possibile, usa `SASL_SSL` per i cluster esterni al tuo computer.

<a id="tls"></a>

## SSL e la CA attendibile

Per `SSL` e `SASL_SSL` il modulo aggiunge le impostazioni TLS.

**Certificato CA attendibile** è il certificato dell’autorità (CA) che ha firmato i certificati dei broker. Lascia il campo vuoto se il sistema operativo considera già attendibile questa CA: in tal caso vengono usati i certificati radice di sistema. Altrimenti inserisci il percorso completo del file o fai clic su **Sfoglia…**. Dopo aver scelto un file compare il campo **Formato CA**. Per PKCS#12 e JKS c’è anche il campo **Password del truststore**.

L’opzione **Verifica nome host** è attiva per impostazione predefinita. Verifica che il certificato del broker sia stato emesso per il nome host a cui ti connetti. Disattivala solo negli ambienti di test, per esempio quando i broker usano certificati autofirmati con altri nomi.

<a id="mtls"></a>

## Certificato client (mTLS)

Se i broker autenticano i client tramite certificato, attiva **Certificato client (mTLS)**. Il certificato e la chiave vengono emessi dai tuoi amministratori Kafka. Poi compila:

- **Certificato client** — il percorso del file del certificato.
- **Formato del certificato client** — PEM, PKCS#12 o JKS.
- Per PEM: **Chiave client** — il percorso del file della chiave privata, e **Password della chiave (se cifrata)** se la chiave è cifrata.
- Per PKCS#12: **Password del keystore**.
- Per JKS: **Password del keystore**, **Password della chiave (se diversa)** se la chiave ha una password propria, e **Alias della chiave** se l’archivio contiene più chiavi.

I percorsi devono essere assoluti. I file vengono letti a ogni connessione, quindi dopo il rinnovo del certificato basta riconnettersi. L’app verifica che la chiave corrisponda al certificato e che il certificato non sia scaduto.

Un caso comune è un cluster in Kubernetes, dove il secret del client contiene `tls.crt`, `tls.key` e `ca.crt`. Salvali in file, scegli il protocollo `SSL`, imposta **Certificato CA attendibile** su `ca.crt` e usa `tls.crt` e `tls.key` come certificato e chiave client PEM con password vuote.

<a id="certificate-formats"></a>

## Formati dei certificati

Glafira legge tre formati. Il formato viene scelto automaticamente in base all’estensione del file; controllalo nel campo del formato e cambialo se necessario.

- PEM — un file di testo che inizia con `-----BEGIN`. Estensioni `.pem`, `.crt`, `.cer`, `.key`. Per un certificato client, la chiave è un file separato. La CA attendibile può essere anche un certificato DER binario.
- PKCS#12 — un archivio binario con certificato e chiave. Estensioni `.p12`, `.pfx`.
- JKS — un keystore Java. Estensioni `.jks`, `.keystore`, `.truststore`.

Due tipi di file vanno prima convertiti:

- Gli archivi JCEKS non sono supportati. Convertili in PKCS#12:

```bash
keytool -importkeystore -srcstoretype JCEKS -srckeystore client.jceks -deststoretype PKCS12 -destkeystore client.p12
```

- Gli archivi PKCS#12 cifrati con algoritmi obsoleti (RC2, 3DES) potrebbero non aprirsi. Se l’app segnala un algoritmo obsoleto, ricrea il file con AES:

```bash
openssl pkcs12 -export -keypbe AES-256-CBC -certpbe AES-256-CBC -in tls.crt -inkey tls.key -out client.p12
```

<a id="passwords"></a>

## Password

La password SASL e le password di chiavi e archivi vengono salvate nell’archivio dei segreti di sistema: Portachiavi su macOS, Gestione credenziali su Windows, Secret Service su Linux. Non vengono mai scritte nel profilo della connessione, nei file di esportazione né nell’archivio diagnostico.

Quando modifichi una connessione salvata, i campi delle password sono vuoti e mostrano **lascia vuoto per non modificarla**. Lascia un campo vuoto per mantenere la password salvata, oppure digitane una nuova per sostituirla. Per eliminare la password salvata di un certificato, seleziona **Rimuovi password salvata** accanto al campo. Al salvataggio vengono eliminate le password che la connessione non usa più, per esempio dopo il passaggio del protocollo a `PLAINTEXT`.

Se l’app non riesce ad accedere all’archivio dei segreti, al salvataggio compare un errore. Su Linux verifica che GNOME Keyring, KWallet o un altro provider Secret Service sia in esecuzione nella tua sessione.

<a id="environments"></a>

## Ambienti e prod

**Ambiente** contrassegna la connessione: `local`, `dev`, `test`, `stage` o `prod`. L’ambiente compare come etichetta colorata accanto al nome nel pannello, così vedi sempre con quale cluster stai lavorando.

`prod` aggiunge una protezione. Qualsiasi modifica su una connessione prod, come creare un topic, inviare un messaggio, reimpostare gli offset o avviare una sessione consumer, richiede di digitare il nome della connessione. L’eliminazione e lo svuotamento chiedono il nome dell’oggetto su qualsiasi connessione (l’eliminazione di più topic ne chiede il numero), e su prod vanno digitati entrambi. Queste conferme compaiono all’interno della finestra dell’app.

Una nuova connessione prod nasce con l’opzione **Sola lettura** attiva. Deseleziona la casella se devi apportare modifiche.

<a id="read-only"></a>

## Sola lettura

L’opzione **Sola lettura** blocca tutto ciò che modifica il cluster: creazione, modifica, svuotamento ed eliminazione dei topic, invio di messaggi, reimpostazione degli offset, eliminazione dei gruppi e avvio di sessioni consumer. I pulsanti di queste azioni sono nascosti o disattivati, e anche il core dell’app rifiuta tali operazioni. La consultazione e la lettura restano disponibili. Un’icona a forma di lucchetto accanto al nome nel pannello contrassegna le connessioni in sola lettura.

Usa questa modalità quando devi solo guardare i dati, soprattutto sui cluster condivisi. La sola lettura è una protezione interna all’app: non sostituisce i diritti di accesso (ACL) in Kafka.

<a id="folders"></a>

## Cartelle

Le cartelle aiutano a organizzare le connessioni, per esempio per progetto o per ambiente. Le cartelle possono essere annidate.

- Per creare una cartella al primo livello, fai clic su `+📁` ( **Nuova cartella** ) nell’intestazione del pannello. Per creare una sottocartella, apri il menu della cartella e scegli **Nuova cartella**. Digita il nome e premi `Enter`, oppure premi `Esc` per annullare.
- Per aprire il menu della cartella, fai clic su `⋯` ( **Altre azioni** ) nella riga della cartella o fai clic con il tasto destro sulla cartella. Il menu contiene **Rinomina**, **Elimina** e i comandi di esportazione e importazione.
- Si può eliminare solo una cartella vuota. Sposta o elimina prima le sue connessioni e sottocartelle.
- I nomi delle cartelle devono essere univoci all’interno di uno stesso livello e lunghi al massimo 100 caratteri.

Per spostare una connessione, trascinala su una cartella, oppure su un’altra connessione per posizionarla prima di quella. Per spostarla al primo livello, rilasciala sull’intestazione del pannello o nello spazio vuoto sotto l’elenco. Puoi anche scegliere la cartella in **Cartella** nel modulo della connessione; **(radice)** indica il primo livello. Le cartelle si trascinano allo stesso modo. Rilascia una cartella sul bordo superiore di un’altra per posizionarla prima di quella. Una cartella non può essere spostata in una propria sottocartella.

Fai clic sul nome di una cartella per comprimerla o espanderla. L’app ricorda quali cartelle sono compresse. Usa **Filtra connessioni** sopra l’albero per trovare connessioni e cartelle per nome. Le cartelle corrispondenti vengono mostrate con tutto il loro contenuto.

<a id="export"></a>

## Esportazione

L’esportazione salva connessioni e cartelle in un file, per esempio per condividerle con i colleghi o trasferirle su un altro computer.

1. Per esportare tutto, apri il menu `⋯` dell’intestazione del pannello e scegli **Esporta tutto…**. Per esportare una cartella con le sue sottocartelle, scegli **Esporta cartella…** nel suo menu.
2. Scegli dove salvare il file. Il nome proposto è `glafira-connections-<date>.glafira-connections.json` o `<folder>.glafira-connections.json`.

Una volta salvato il file, il pannello mostra quante connessioni e cartelle contiene.

Il file contiene le impostazioni delle connessioni e la struttura delle cartelle. Non contiene password né proprietà segrete. Non include nemmeno i file dei certificati: ne conserva solo i percorsi. Host, nomi utente e percorsi sono memorizzati come testo in chiaro, quindi condividi il file solo con chi è autorizzato a vederli.

<a id="import"></a>

## Importazione

1. Apri il menu `⋯` dell’intestazione del pannello e scegli **Importa…**. Per importare direttamente in una cartella, scegli **Importa qui…** nel suo menu.
2. Scegli un file `.glafira-connections.json`. Si apre la finestra **Importa connessioni**.
3. In **Importa in** scegli dove vanno le nuove cartelle e connessioni. **(radice)** è il primo livello.
4. Lascia attiva l’opzione **Aggiorna le connessioni esistenti** per aggiornare dal file le connessioni che esistono già. Disattivala per lasciarle invariate.
5. Controlla l’anteprima e fai clic su **Importa**, oppure su **Annulla** per chiudere la finestra senza modifiche.

L’anteprima si aggiorna a ogni modifica nella finestra. Mostra i totali ed elenchi espandibili: **Nuovi**, **Verranno aggiornati**, **Invariati** e **Saltati**, con il motivo per ogni connessione saltata.

Come l’app abbina il file alle tue connessioni:

- Le connessioni vengono abbinate tramite un ID interno, non per nome. Una connessione è considerata esistente se il file contiene la stessa connessione: per esempio, è stata esportata da questo computer o importata in precedenza dalla stessa origine.
- Una connessione aggiornata riceve le impostazioni dal file. Le tue password e la sua posizione nell’albero vengono mantenute.
- Le cartelle esistenti non vengono rinominate né spostate. Le nuove connessioni del file vengono comunque inserite al loro interno.
- Se una nuova connessione ha lo stesso nome di un’altra nella stessa cartella, viene aggiunto un numero, per esempio `kafka-dev (2)`.
- Le connessioni che questa versione di Glafira non supporta, o che contengono segreti nelle proprietà aggiuntive, vengono saltate.

L’anteprima avvisa anche quando i file dei certificati non si trovano su questo computer. Le password non sono nel file, quindi dopo l’importazione il pannello indica quante connessioni ne hanno bisogno. Apri ciascuna di queste connessioni, inserisci le password e salva.
