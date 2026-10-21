# Consultare i messaggi

I messaggi si consultano nella scheda **Messaggi** di un topic. La consultazione non entra in nessun consumer group e non esegue commit, quindi il cluster non cambia. Funziona anche sulle connessioni con l’opzione **Sola lettura** attiva. Per leggere come membro di un gruppo, usa una [sessione consumer](consumer-sessions.md).

<a id="start-reading"></a>

## Avviare la lettura

1. Apri un topic dall’elenco **Topic**. La scheda **Messaggi** si apre per prima.
2. Scegli da dove iniziare in **Da**. Vedi [Posizione iniziale](browse.md#start-position).
3. Facoltativamente, imposta i formati e i filtri.
4. Fai clic su **Leggi**.

I messaggi compaiono nella tabella man mano che arrivano. La riga accanto ai pulsanti mostra quanti messaggi sono stati esaminati, quanti corrispondono ai filtri e quanti sono in tabella. Mostra anche lo stato: **lettura…**, **limite raggiunto**, **raggiunta la fine del topic** o **interrotto**.

Senza `Live tail`, la lettura si ferma alla fine del topic così com’era quando hai fatto clic su **Leggi**, oppure prima, al raggiungimento del valore di **Limite**. Il limite conta i messaggi che corrispondono ai filtri. Può andare da 1 a 100000; valori elevati rallentano la lettura.

Con `Live tail` attivo, la lettura non si ferma alla fine: i nuovi messaggi vengono aggiunti man mano che vengono scritti. Quando tutti i messaggi vecchi sono stati letti, lo stato mostra **in attesa di nuovi messaggi…**. La tabella conserva gli ultimi messaggi fino al limite e scarta i più vecchi.

**Interrompi** interrompe la lettura e mantiene la tabella. Durante la lettura, il pulsante **Leggi** diventa **Riavvia**: ricomincia con le impostazioni attuali e svuota la tabella. Se la connessione al broker si interrompe, l’app si riconnette da sola e lo segnala nella riga di stato.

Nella tabella i messaggi di partizioni diverse sono mescolati. Kafka mantiene l’ordine solo all’interno di una partizione. La colonna `P` indica la partizione e `Offset` la posizione al suo interno. Seleziona `UTC` per mostrare le ore in UTC anziché nell’ora locale.

<a id="start-position"></a>

## Posizione iniziale

**Posizione iniziale** stabilisce dove inizia la lettura in ogni partizione:

- **Ultimi N** — gli ultimi N messaggi. N viene suddiviso equamente tra le partizioni, quindi con 100 messaggi e 4 partizioni l’app ne prende fino a 25 da ciascuna.
- **Dall’inizio** — dal messaggio più vecchio ancora conservato.
- **Da offset** — dallo stesso offset in ogni partizione. Se una partizione non ha questo offset, la lettura parte dal primo o dall’ultimo offset della partizione. Questa modalità è utile soprattutto con una sola partizione.
- **Da data e ora** — dal primo messaggio con timestamp uguale o successivo all’ora locale indicata. Nelle partizioni senza messaggi più recenti, la lettura inizia dalla fine.
- **Ultimi X minuti** — lo stesso, contando a ritroso da adesso.

Il campo con **partizioni: tutte** limita la lettura ad alcune partizioni. Inserisci i numeri delle partizioni separati da virgole, per esempio `0,2`. Lascialo vuoto per leggere tutte le partizioni.

<a id="filters"></a>

## Filtri

I filtri mostrano solo i messaggi corrispondenti. L’app legge comunque tutti i messaggi dalla posizione iniziale e controlla ciascuno, quindi i filtri non velocizzano la lettura. Tutti i campi compilati devono corrispondere.

- **La chiave contiene** e **Il valore contiene** — testo nella chiave o nel valore.
- `regex` — interpreta questi campi, e il valore dell’intestazione, come espressioni regolari.
- `Aa` — distingue maiuscole e minuscole. Per impostazione predefinita non contano.
- **Intestazione** — il messaggio ha un’intestazione con esattamente questo nome. **Valore dell’intestazione** ne controlla anche il valore.
- `JSONPath` — una condizione su un valore JSON, per esempio `$.order.status == "FAILED"`. Senza `==` o `!=`, il percorso deve semplicemente esistere, per esempio `$.error`. I messaggi il cui valore non è JSON valido non corrispondono.
- **Data/ora inizio** e **Data/ora fine** — il timestamp del messaggio rientra in questo intervallo, in ora locale. Entrambi gli estremi sono inclusi.

I filtri controllano il testo mostrato in tabella, dopo la decodifica. Se il formato del valore è `hex`, il filtro sul valore cerca nel testo esadecimale. I filtri si applicano quando fai clic su **Leggi**; dopo averli cambiati, rileggi.

Un filtro non sposta la posizione iniziale. Per esempio, **Ultimi N** con un filtro trova corrispondenze solo tra gli ultimi N messaggi. Per cercare in tutto il topic, inizia da **Dall’inizio**.

<a id="formats"></a>

## Formati

**Formato** stabilisce come vengono decodificati chiavi e valori, separatamente per **chiave** e **valore**. Cambia solo la visualizzazione, non i dati.

- `auto` — l’impostazione predefinita. Gli oggetti e gli array JSON validi vengono mostrati come JSON, l’altro testo UTF-8 come stringa, i dati binari in esadecimale.
- `string` — testo in UTF-8. I byte che non sono UTF-8 valido vengono sostituiti con `�`.
- `json` — JSON con rientri nei dettagli del messaggio. Se il valore non è JSON valido, i dettagli mostrano l’errore.
- `hex` — ogni byte come due cifre esadecimali.
- `base64` — i byte in Base64.

I dettagli indicano il formato effettivamente usato, per esempio `json` quando `auto` ha riconosciuto un JSON. I valori delle intestazioni vengono mostrati come testo; i valori binari delle intestazioni come `0x` seguito dall’esadecimale. Una chiave o un valore assente viene mostrato come `null`.

Chiavi e valori molto grandi vengono mostrati solo in parte, circa il primo MB di testo. Questo vale anche per `hex` e `base64`. In tal caso i dettagli del messaggio mostrano un avviso.

<a id="message-details"></a>

## Dettagli del messaggio

Fai clic su una riga per aprire i dettagli del messaggio accanto alla tabella. In alto ci sono la partizione, l’offset e l’ora. Sotto:

- **Valore** e **Chiave** — il testo completo, con il formato e la dimensione in byte. Il JSON viene mostrato con i rientri. **Copia** copia il testo.
- **Intestazioni** — una tabella con i nomi e i valori delle intestazioni.

Le colonne della tabella sono `P`, `Offset`, **Data/ora**, **Chiave**, **Valore**, **Intestazioni** (il loro numero) e **Dimensione** (la dimensione di chiave e valore in byte). L’ora è il timestamp del messaggio. A seconda dell’impostazione del topic `message.timestamp.type`, viene impostata dal producer o dal broker.

<a id="export-messages"></a>

## Copiare i messaggi

Glafira non salva in un file i messaggi consultati. Per portare un messaggio fuori dall’app, copialo:

- **Copia tutto come JSON** copia l’intero messaggio come JSON: topic, partizione, offset, timestamp, chiave, valore, intestazioni e dimensione. Chiave e valore sono inclusi così come sono stati decodificati, con il loro formato.
- **Copia** accanto a **Valore** o **Chiave** copia solo quel testo.

Il testo copiato usa il formato scelto per la lettura. Per copiare esattamente i dati binari, leggili come `hex` o `base64`. Chiavi e valori molto grandi vengono copiati solo in parte, così come sono mostrati, anche in `hex` e `base64`. Vedi [Formati](browse.md#formats).

<a id="open-in-producer"></a>

## Aprire nel Producer

**Reinvia** apre il messaggio nella schermata **Producer** della stessa connessione. Il modulo riceve topic, chiave, valore, intestazioni e formati del messaggio. La scelta della partizione è lasciata a Kafka. I segnaposto nel testo del messaggio non vengono sostituiti: la copia viene inviata così come è stata letta.

Non viene inviato nulla subito. Controlla il modulo, modificalo se necessario e invialo dal Producer. Vedi [Producer](producer.md).

Il pulsante è disattivato in due casi, e viene indicato il motivo:

- la chiave o il valore sono stati mostrati solo in parte perché troppo grandi;
- il testo è stato decodificato con perdite perché non è UTF-8 valido. Leggilo come `hex` o `base64` per inviarne una copia esatta.

Sulle connessioni con l’opzione **Sola lettura** attiva, il pulsante è nascosto.

<a id="reset-group-here"></a>

## Reimpostare un gruppo su un messaggio

**Reimposta il gruppo qui…** prepara una reimpostazione degli offset in modo che un consumer group inizi a leggere da questo messaggio.

1. Nei dettagli del messaggio, fai clic su **Reimposta il gruppo qui…**.
2. Scegli un gruppo nell’elenco. Sono elencati solo i gruppi esistenti.
3. Fai clic su **Vai alla reimpostazione**.

Si apre la scheda del gruppo con il modulo di reimpostazione già compilato. In **Strategia** è selezionato **Offset specifico** con l’offset del messaggio. In **Ambito** è selezionato **Partizioni** con il topic e la partizione del messaggio. Per ora non cambia nulla: fai clic su **Anteprima** per vedere i nuovi offset, poi applica lì la reimpostazione. Il gruppo non deve avere membri attivi. Vedi [Consumer group](groups.md).

Sulle connessioni con l’opzione **Sola lettura** attiva, il pulsante è nascosto.
