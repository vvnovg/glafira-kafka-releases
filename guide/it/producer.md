# Producer

La schermata **Producer** invia messaggi a Kafka: uno alla volta da un modulo, oppure molti insieme. Aprila dalla connessione nel pannello **Connessioni**, oppure con **Invia messaggio** nella scheda di un topic, che compila il topic. Ogni connessione ha una propria scheda Producer.

Sulle connessioni con l’opzione **Sola lettura** attiva, il Producer si apre con l’avviso **La connessione è in sola lettura — l’invio non è disponibile.** I pulsanti di invio sono nascosti, ma i modelli e la cronologia restano consultabili.

<a id="send-message"></a>

## Inviare un messaggio

1. Inserisci il topic in **Topic**. Il campo suggerisce i topic del cluster, esclusi quelli interni.
2. Facoltativamente, inserisci un numero in **Partizione**. Quando il campo mostra **auto**, la partizione viene scelta in base alla chiave: i messaggi con la stessa chiave finiscono nella stessa partizione, come con il client Java. I messaggi senza chiave finiscono in una partizione casuale.
3. Compila **Chiave** e **Valore** e scegline i formati. Vedi [Formati di chiave e valore](producer.md#key-value-formats).
4. Facoltativamente, aggiungi delle intestazioni.
5. Fai clic su **Invia**.

Dopo un invio riuscito, una riga sotto il modulo mostra la partizione, l’offset e l’ora del messaggio. Con `acks` impostato su `0`, il broker non restituisce un offset e al suo posto viene mostrato `—`.

Se il numero di partizione è troppo grande, o il broker non conferma il messaggio entro il timeout della connessione, l’app mostra un errore. Anche un topic inesistente dà errore, a meno che il broker non crei i topic automaticamente (`auto.create.topics.enable`). Con la creazione automatica disattivata, questo errore può comparire dopo circa 30 s (il timeout della connessione). Gli errori nel modulo, come un JSON non valido o un segnaposto sconosciuto, vengono mostrati prima di inviare qualsiasi cosa.

Su una connessione prod, l’invio chiede di digitare il nome della connessione e di confermare con **Invia a prod**. Vedi [Ambienti e prod](connections.md#environments).

**Svuota modulo** svuota il modulo e riporta le impostazioni del producer ai valori predefiniti. Il topic resta.

<a id="key-value-formats"></a>

## Formati di chiave e valore

**Formato della chiave** e **Formato del valore** stabiliscono come il testo del campo viene convertito in byte:

- `string` — il testo così com’è, in UTF-8.
- `json` — il testo viene controllato come JSON valido e inviato così come è stato digitato, senza riformattarlo.
- `hex` — due cifre esadecimali per byte, per esempio `48656c6c6f`. Spazi e a capo vengono ignorati.
- `base64` — byte in Base64 standard.
- `null` — nessuna chiave o nessun valore. Il campo è vuoto e digitando del testo il formato passa a `string`.

Il valore predefinito è `null` per la chiave e `json` per il valore. Per inviare un messaggio con chiave, basta digitarla: il formato passa a `string`. Una stringa vuota è comunque una chiave, non una chiave assente: se scegli `string` e lasci il campo vuoto, tutti questi messaggi hanno lo stesso hash e finiscono in un’unica partizione.

Un messaggio con valore `null` è un tombstone. In un topic con `cleanup.policy=compact`, indica a Kafka di rimuovere i messaggi precedenti con la stessa chiave.

<a id="headers"></a>

## Intestazioni

Le intestazioni sono coppie nome-valore inviate con il messaggio. Fai clic su **Aggiungi intestazione** per aggiungere una riga, poi compila **Nome**, **Valore** e **Formato**. Il pulsante `×` rimuove la riga.

Il formato del valore di un’intestazione è `string` (testo in UTF-8) o `hex` (byte come cifre esadecimali). Le righe con nome vuoto non vengono inviate. Kafka consente più intestazioni con lo stesso nome, e vengono inviate tutte.

I segnaposto funzionano nei valori delle intestazioni, ma non nei nomi.

<a id="placeholders"></a>

## Segnaposto

I segnaposto vengono sostituiti con dei valori a ogni invio. Funzionano nella chiave, nel valore e nei valori delle intestazioni. L’elenco è mostrato sotto le intestazioni:

- `{{uuid}}` — un nuovo UUID casuale, per esempio `0b6f6f2c-3c1e-4a8e-9a54-2f0d6f1d7c21`.
- `{{now}}` — l’ora corrente in UTC nel formato ISO 8601 con millisecondi, per esempio `2026-10-15T12:30:00.000Z`.
- `{{nowMillis}}` — l’ora corrente in millisecondi dal 1970-01-01 UTC.
- `{{seq}}` — il numero della copia in un invio in batch, a partire da 0. Per un invio singolo è `0`.
- `{{randomInt 1 100}}` — un numero intero casuale compreso tra i due numeri, estremi inclusi.
- `{{randomString 8}}` — una stringa casuale di lettere latine e cifre della lunghezza indicata, fino a 100000.
- `{{env NAME}}` — il valore della variabile d’ambiente `NAME`. Se non è impostata, non viene inviato nulla e compare un errore.

Gli spazi all’interno delle graffe sono ammessi. Un segnaposto sconosciuto o una `}}` mancante bloccano l’invio con un errore. Per inviare `{{` come testo, scrivi `\{{`.

Con il formato `json`, il JSON viene controllato dopo la sostituzione dei segnaposto. I segnaposto che producono testo richiedono le virgolette: `{"id": "{{uuid}}"}`. I numeri no: `{"n": {{seq}}}`.

`{{env NAME}}` legge l’ambiente del processo Glafira. Se avvii Glafira dal Dock o dal menu Start, le variabili impostate solo nel profilo del terminale potrebbero mancare.

Quando un messaggio proviene da [Aprire nel Producer](browse.md#open-in-producer) o dalla cronologia, ogni `{{` al suo interno viene scritto come `\{{`. Il messaggio viene quindi inviato esattamente com’era, senza sostituire nulla.

<a id="templates"></a>

## Modelli

Un modello salva l’intero modulo: topic, partizione, chiave, valore, intestazioni, formati e impostazioni del producer. I segnaposto vengono salvati così come sono scritti e sostituiti a ogni invio.

Per salvare un modello, fai clic su **Salva come modello**, inserisci il nome in **Nome del modello** e fai clic su **Salva**. Il topic può essere vuoto. Per impostazione predefinita, il modello appartiene a questa connessione. Seleziona **Per tutte le connessioni** per mostrarlo su tutte le connessioni; nell’elenco questi modelli sono contrassegnati da **(tutte le connessioni)**.

Per usare un modello, sceglilo nell’elenco **Modello**. Il modulo viene sostituito con il modello. Puoi modificare il modulo prima dell’invio; il modello non cambia.

Il salvataggio crea sempre un nuovo modello. Per modificarne uno, caricalo, modifica il modulo, salvalo con un nuovo nome ed elimina quello vecchio. Per eliminare il modello selezionato, fai clic su **Elimina modello** e conferma con **Elimina**.

I modelli di una connessione vengono eliminati insieme alla connessione.

<a id="history"></a>

## Cronologia

**Cronologia (ultimi 100)** elenca gli ultimi 100 invii singoli di questa connessione, dal più recente. I messaggi degli invii in batch non vengono aggiunti. Ogni riga mostra l’ora, il topic, la partizione con l’offset e l’inizio della chiave.

La cronologia conserva il messaggio così come è stato inviato, con i segnaposto già sostituiti. **Reinvia** lo carica nel modulo insieme alle sue impostazioni del producer. Non viene inviato nulla finché non fai clic su **Invia**. La copia viene inviata esattamente come è stata conservata: per esempio, viene inviato di nuovo lo stesso UUID.

Una chiave, un valore o un valore di intestazione più lungo di 64 KB viene conservato solo in parte. Per queste righe, **Reinvia** è disattivato.

La cronologia è conservata su questo computer in testo in chiaro, compresi i valori di `{{env NAME}}`. Se per te è importante, non inviare segreti tramite il Producer. La cronologia di una connessione viene eliminata insieme alla connessione.

<a id="batch-copies"></a>

## N copie del modulo

**Invio in batch** sotto il modulo invia molti messaggi in un’unica esecuzione. In modalità **N copie del modulo** invia il modulo il numero di volte indicato.

1. Compila il modulo come per un invio singolo.
2. Seleziona **N copie del modulo** e inserisci il numero in **Copie**, da 1 a 1000000.
3. Facoltativamente, imposta la **Velocità**. Vedi [Velocità di invio](producer.md#batch-rate).
4. Fai clic su **Avvia**.

I segnaposto vengono sostituiti separatamente per ogni copia. `{{seq}}` è il numero della copia, da 0 a N−1, quindi ogni messaggio può avere una chiave propria, per esempio `order-{{seq}}`. Gli errori nei segnaposto vengono controllati sulla prima copia prima che l’esecuzione inizi.

Durante l’esecuzione, il pannello mostra quanti messaggi sono stati inviati e quanti non sono riusciti. Alla fine mostra **completato**. Gli errori sono elencati con il numero della copia, contato da 1. L’elenco mostra i primi 20 errori; se sono di più, una nota sotto indica il totale.

**Annulla** interrompe l’esecuzione. I messaggi che non hanno ancora lasciato l’app non vengono inviati e il pannello ne mostra il numero. L’app attende fino a 2 secondi che il broker confermi i messaggi già inviati. I messaggi ancora non confermati contano come errori, perché non si sa se sono stati scritti. Anche la chiusura della scheda Producer interrompe l’esecuzione.

Su una connessione prod, l’esecuzione chiede il nome della connessione e si conferma con **Avvia su prod**.

<a id="batch-file"></a>

## Invio da file

In modalità **da file**, ogni riga o record di un file diventa un messaggio. I file supportati sono JSON Lines (`.jsonl`, `.ndjson`) e CSV (`.csv`).

1. Inserisci il topic in **Topic** nel modulo sopra. Tutti i messaggi vanno in questo topic.
2. Seleziona **da file** e fai clic su **Scegli file…**.
3. Fai clic su **Avvia**.

Chiave, valore, intestazioni e partizione del modulo non vengono usati e i segnaposto non vengono sostituiti. Si applicano le impostazioni del producer del modulo.

In JSON Lines ogni riga è un oggetto JSON, per esempio `{"key": "k1", "value": {"id": 1}, "headers": {"source": "test"}, "partition": 0}`. Tutti i campi sono facoltativi:

- `key` e `value` — una stringa JSON viene inviata come testo, senza virgolette. Qualsiasi altro valore JSON, come un oggetto o un numero, viene inviato come testo JSON. `null` o un campo assente significano nessuna chiave o nessun valore.
- `headers` — un oggetto con valori stringa.
- `partition` — un numero di partizione. Senza di esso, la partizione viene scelta in base alla chiave.

Le righe vuote vengono saltate. Il file deve essere in UTF-8.

In CSV la prima riga contiene i nomi delle colonne. La colonna `value` è obbligatoria; le colonne `key`, `partition` e `header.NAME` sono facoltative, per esempio `header.source`. Una cella `key` vuota significa nessuna chiave, e una cella di intestazione vuota significa che quell’intestazione non c’è. Una cella `value` vuota invia un valore vuoto, non `null`, quindi da CSV non si può inviare un tombstone.

Il file viene letto come flusso, quindi il totale non è noto in anticipo e viene mostrato come `—`. Una riga o un record più grande di 16 MB, un JSON non valido o una partizione negativa contano come errore per quella riga, e l’esecuzione prosegue con la riga successiva. Gli errori vengono mostrati con il numero di riga.

<a id="batch-rate"></a>

## Velocità di invio

**Velocità** limita un invio in batch al numero indicato di messaggi al secondo. Inserisci un numero intero pari o superiore a 1. Un campo vuoto significa **illimitata**.

I messaggi vengono distribuiti in modo uniforme nel tempo: a 10 messaggi al secondo, ne parte uno circa ogni 100 ms. Usa il limite per non sovraccaricare un cluster condiviso o per simulare un carico costante.

Senza limite, l’app invia alla velocità con cui il broker accetta i messaggi. La velocità si applica a entrambe le modalità di invio in batch.

<a id="producer-settings"></a>

## Impostazioni del producer

Le **Impostazioni del producer** si applicano agli invii singoli e a quelli in batch. Vengono salvate nei modelli e nella cronologia.

`acks` stabilisce quante repliche devono confermare una scrittura:

- `all` — il valore predefinito. Il leader attende tutte le repliche sincronizzate. È la scelta più sicura. Se le repliche sincronizzate sono meno di quanto richiede l’impostazione del topic `min.insync.replicas`, la scrittura non riesce.
- `1` — conferma solo il leader. Più veloce, ma un messaggio può andare perso se il leader si guasta prima che i follower lo copino.
- `0` — il producer non attende alcuna conferma. I messaggi possono andare persi e non viene restituito alcun offset.

L’opzione **compressione** comprime i batch di messaggi: `none` (il valore predefinito), `gzip`, `snappy`, `lz4` o `zstd`. La compressione riduce il traffico di rete e lo spazio su disco al prezzo di un po’ di CPU. `zstd` e `lz4` sono scelte comuni. Per un singolo messaggio piccolo il vantaggio è minimo.

L’opzione **idempotenza** imposta `enable.idempotence`. Evita i duplicati quando il client ripete un invio dopo un errore di rete. Richiede `acks` impostato su `all`; con altri valori, l’app mostra un errore e non invia nulla.
