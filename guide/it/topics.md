# Cluster e topic

Questo capitolo tratta la panoramica del cluster, l’elenco dei topic, la creazione e l’eliminazione dei topic, la schermata del topic e la sua configurazione. Sotto ogni connessione del pannello **Connessioni** ci sono i collegamenti **Panoramica del cluster** e **Topic**. Ognuno si apre in una scheda propria.

<a id="cluster-overview"></a>

## Panoramica del cluster

**Panoramica del cluster** mostra lo stato del cluster al momento del caricamento. Fai clic su **Aggiorna** per ricaricarla.

- `Cluster ID` — l’identificatore del cluster.
- **Controller** — l’ID del broker che il cluster indica come controller. Nei cluster KRaft i client non comunicano direttamente con i controller: qui il cluster indica un broker casuale, non il controller attivo, e il broker può cambiare da un aggiornamento all’altro.
- **Broker** — il numero di broker.
- **Topic (interni)** — tutti i topic, con il numero di topic interni tra parentesi.
- **Partizioni** — il numero di partizioni di tutti i topic.
- `Under-replicated` — le partizioni in cui non tutte le repliche sono sincronizzate (nell’ISR). Di solito un broker è inattivo o non riesce a tenere il passo.
- `Offline` — le partizioni senza leader. Non è possibile né scrivervi né leggerle.

Un valore diverso da zero in `Under-replicated` o `Offline` viene evidenziato.

Sotto si trova la tabella dei broker con `ID` e **Indirizzo**. Il broker indicato nel campo **Controller** è contrassegnato da **controller**. Nei cluster KRaft si tratta dello stesso broker casuale, non del controller attivo. Fai clic su un broker per vederne la configurazione sotto la tabella. La configurazione dei broker si può solo visualizzare. Usa **Cerca parametro** per trovare un parametro per nome e **Solo modificati** per nascondere i parametri con valori predefiniti. Un’icona a forma di lucchetto contrassegna i parametri di sola lettura.

<a id="topic-list"></a>

## Elenco dei topic

**Topic** elenca i topic del cluster. Fai clic su una riga per aprire il topic in una scheda propria.

Le colonne sono **Nome**, **Partizioni**, `RF` (fattore di replica), **Messaggi (≈)**, `cleanup.policy` e **Sotto-repl.** (il numero di partizioni sotto-replicate). Il numero di messaggi è approssimativo: è la differenza tra gli offset finali e iniziali delle partizioni. La compattazione e i marcatori di transazione rendono il numero reale più piccolo. Il numero di messaggi e `cleanup.policy` vengono caricati solo per le righe che raggiungi scorrendo.

Controlli della barra degli strumenti:

- **Cerca (o /regex/)** — digita parte di un nome per trovare i topic; maiuscole e minuscole non contano. Per cercare con un’espressione regolare, racchiudila tra barre, per esempio `/^orders\..*/`.
- **Interni** — mostra i topic interni, come `__consumer_offsets`. Sono contrassegnati da **(interno)**.
- L’elenco dell’aggiornamento automatico — **Autoaggiornamento: disattivato**, **ogni 10 s** o **ogni 30 s**.
- **Aggiorna** — ricarica subito l’elenco e il numero di messaggi.

Accanto alla barra degli strumenti, l’app mostra quanti topic corrispondono alla ricerca rispetto al totale. Sulle connessioni con l’opzione **Sola lettura** attiva, i pulsanti **Crea topic** e **Seleziona** sono nascosti.

<a id="create-topic"></a>

## Creare un topic

1. Nell’elenco dei topic, fai clic su **Crea topic**. Sopra l’elenco si apre il modulo **Nuovo topic**.
2. Inserisci il **Nome del topic**. Può contenere lettere latine, cifre, `.`, `_` e `-`, fino a 249 caratteri. Il modulo avvisa se il nome contiene sia `.` sia `_`: i nomi delle metriche Kafka di questi topic possono entrare in conflitto con quelli di altri topic.
3. Imposta il numero di **Partizioni**. Le partizioni vengono lette in parallelo e l’ordine è garantito solo all’interno di una partizione. Il numero può essere aumentato in seguito, ma mai ridotto.
4. Imposta il **Fattore di replica**, cioè quante copie di ogni partizione conservano i broker. Non può superare il numero di broker, che il modulo mostra sotto il campo. Il valore predefinito è 3, oppure il numero di broker se sono meno. `-1` usa il valore predefinito del broker.
5. Scegli il **Profilo**:
   - **delete (predefinito)** — i messaggi vecchi vengono eliminati in base al tempo.
   - `compact` — viene conservato solo l’ultimo valore per chiave.
   - `compact + delete` — entrambe le cose.
   - **test (conservazione 1 h)** — i messaggi vengono conservati per un’ora.
6. Facoltativamente, aggiungi **Configurazioni aggiuntive (key=value)**, una per riga, per esempio `retention.ms=86400000`. Hanno la precedenza sul profilo. Le righe che iniziano con `#` vengono ignorate.
7. Fai clic su **Crea**.

Quando il modulo è valido, mostra il comando `kafka-topics.sh` equivalente. Puoi copiarlo, per esempio per creare lo stesso topic in un altro cluster.

Su una connessione prod confermi la creazione digitando il nome della connessione. Una volta creato il topic, si apre la sua scheda.

<a id="delete-topics"></a>

## Eliminare i topic

Per eliminare un topic, aprilo e fai clic su **Elimina topic**. Digita il nome del topic per confermare e fai clic su **Elimina**. La scheda del topic si chiude.

Per eliminare più topic contemporaneamente:

1. Nell’elenco dei topic, fai clic su **Seleziona**. Accanto ai nomi compaiono le caselle di controllo.
2. Seleziona i topic. I topic interni non si possono selezionare.
3. Fai clic sul pulsante di eliminazione, che mostra il numero di topic selezionati.
4. Controlla l’elenco dei nomi, digita il loro numero per confermare e fai clic su **Elimina**.

**Annulla selezione** nasconde le caselle di controllo.

Su una connessione prod, entrambi i modi di eliminazione chiedono anche di digitare il nome della connessione.

L’eliminazione non si può annullare: i dati vanno persi. Se alcuni topic non vengono eliminati, per esempio per mancanza di diritti, l’app li elenca con il motivo. Gli altri topic vengono eliminati.

<a id="topic-details"></a>

## Schermata del topic

La scheda del topic mostra il numero di partizioni, il fattore di replica e il numero approssimativo di messaggi. I pulsanti in alto:

- **Aggiorna** — ricarica i dati del topic.
- **Invia messaggio** — apre la schermata **Producer** con questo topic. Vedi [Producer](producer.md).
- **Leggi come gruppo** — avvia una sessione consumer per questo topic. Vedi [Sessioni consumer](consumer-sessions.md).

Sulle connessioni con l’opzione **Sola lettura** attiva, entrambi i pulsanti sono disattivati.

La seconda riga contiene le azioni che modificano il topic. È nascosta sulle connessioni in sola lettura e per i topic interni.

- **Aggiungi partizioni** — inserisci il nuovo totale e fai clic su **Aumenta**. Il numero di partizioni non può essere ridotto. Dopo un aumento, i messaggi con la stessa chiave possono finire in un’altra partizione, quindi l’ordine per chiave non è più garantito.
- **Svuota** — elimina tutti i messaggi fino alla fine attuale delle partizioni selezionate. Il topic resta. Seleziona le partizioni, digita il nome del topic e fai clic su **Svuota partizioni**. Lo svuotamento funziona solo se `cleanup.policy` include `delete`; altrimenti il pulsante è disattivato e viene indicato il motivo. Per un topic `compact,delete`, lo svuotamento elimina anche gli ultimi valori delle chiavi.
- **Elimina topic** — vedi [Eliminare i topic](topics.md#delete-topics).

Su una connessione prod, ciascuna di queste azioni chiede anche il nome della connessione.

Le schede sottostanti:

- **Messaggi** — la consultazione dei messaggi. Vedi [Consultare i messaggi](browse.md).
- **Partizioni** — una tabella con ogni partizione: **Leader**, **Repliche**, `ISR` (repliche sincronizzate), gli offset iniziale (`Low`) e finale (`High`) e il numero di messaggi. Una partizione senza leader mostra **nessuno**.
- **Configurazione** — i parametri del topic. Vedi [Configurazione del topic](topics.md#topic-config).
- **Consumer group** — i consumer group che hanno offset sottoposti a commit per questo topic, con **Stato**, **Partizioni con commit** e **Lag**. Fai clic su un gruppo per aprirlo. Vedi [Consumer group](groups.md).

<a id="topic-config"></a>

## Configurazione del topic

La scheda **Configurazione** elenca tutti i parametri del topic in tre colonne:

- **Parametro** — il nome del parametro. I nomi in grassetto sono impostati esplicitamente: sul topic o nella configurazione del broker.
- **Valore** — il valore attuale. I valori sensibili sono nascosti come `••••••`.
- **Origine** — da dove proviene il valore. Per esempio, `DynamicTopic` è impostato su questo topic, `StaticBroker` e `DynamicBroker` provengono dalla configurazione del broker e `Default` è il valore predefinito di Kafka.

Usa **Cerca parametro** per trovare un parametro per nome e **Solo modificati** per mostrare solo i parametri in grassetto. Un’icona a forma di lucchetto contrassegna i parametri di sola lettura. L’icona `?` accanto a un nome spiega il parametro.

<a id="edit-config"></a>

## Modificare la configurazione

1. Nella scheda **Configurazione**, fai clic su **Modifica**. Sulle connessioni con l’opzione **Sola lettura** attiva, questo pulsante è nascosto.
2. Modifica i valori nella colonna **Valore**. I parametri di sola lettura e quelli sensibili non si possono modificare. Per rimuovere il valore proprio del topic, fai clic su **ripristina predefinito**: si applica allora il valore del broker. Un valore vuoto non è ammesso.
3. Fai clic su **Rivedi le modifiche**. L’app mostra ogni modifica come valore vecchio → valore nuovo; **predefinito** significa che il valore viene ripristinato.
4. Fai clic su **Applica**, oppure su **Indietro** per continuare a modificare.

Nel cluster non cambia nulla finché non applichi le modifiche. **Scarta le modifiche** annulla tutte le modifiche. Finché ci sono modifiche non salvate, il pulsante **Chiudi editor** è disattivato. Su una connessione prod confermi le modifiche digitando il nome della connessione.

Vengono inviati solo i parametri che hai modificato. Gli altri parametri del topic restano invariati. Il broker controlla i valori e rifiuta con un errore quelli non validi.

<a id="config-parameters"></a>

## Parametri del topic

I topic Kafka hanno decine di parametri. Le icone `?` nella tabella della configurazione li spiegano tutti. Questi sono i più usati:

- `cleanup.policy` — che cosa succede ai dati vecchi. `delete` rimuove i segmenti vecchi in base al tempo o alla dimensione. `compact` conserva solo l’ultimo valore per chiave. `compact,delete` fa entrambe le cose.
- `retention.ms` — per quanto tempo vengono conservati i messaggi, in millisecondi. Il valore predefinito è `604800000` (7 giorni); `-1` li conserva per sempre. Funziona se `cleanup.policy` include `delete`.
- `retention.bytes` — la dimensione massima di ogni partizione prima che i segmenti vecchi vengano eliminati. Il valore predefinito `-1` significa nessun limite di dimensione.
- `segment.ms` e `segment.bytes` — quando il broker inizia un nuovo segmento del log. Conservazione e compattazione agiscono su segmenti interi, quindi un messaggio può restare più a lungo di `retention.ms` finché il suo segmento non viene chiuso.
- `min.insync.replicas` — quante repliche devono confermare una scrittura con `acks=all`. Di solito 2 con fattore di replica 3.
- `max.message.bytes` — il batch più grande che il broker accetta per questo topic. Il valore predefinito è circa 1 MB. I producer devono consentire la stessa dimensione (`max.request.size`).
- `compression.type` — come il broker memorizza i messaggi. `producer` mantiene la compressione del producer.
- `message.timestamp.type` — `CreateTime` (l’ora impostata dal producer) o `LogAppendTime` (l’ora in cui il broker ha scritto il messaggio). È l’ora mostrata durante la consultazione dei messaggi.
- `delete.retention.ms` — per quanto tempo i marcatori di eliminazione (messaggi con valore null) restano in un topic compattato. Il valore predefinito è 1 giorno.

Alcuni parametri esistono solo nelle versioni più recenti di Kafka, altri sono deprecati. L’icona `?` lo segnala dove conta. I parametri del tiered storage, come `remote.storage.enable`, funzionano solo se il cluster lo supporta.
