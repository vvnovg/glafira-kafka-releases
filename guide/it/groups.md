# Consumer group

Un consumer group è un insieme di consumer che si dividono la lettura dei topic. Kafka assegna ogni partizione a un membro del gruppo e conserva, per ogni partizione, l’offset di cui il gruppo ha eseguito il commit. Dopo un riavvio, le applicazioni riprendono a leggere da quell’offset.

Apri **Consumer group** sotto la connessione nel pannello **Connessioni**. I gruppi di un singolo topic si trovano anche nella scheda **Consumer group** del topic. Vedi [Schermata del topic](topics.md#topic-details).

<a id="group-list"></a>

## Elenco dei gruppi

L’elenco mostra tutti i consumer group del cluster. Le colonne:

- **Gruppo** — l’ID del gruppo (`group.id`).
- **Stato** — lo stato riportato da Kafka, per esempio `Stable` (i membri stanno leggendo), `Empty` (nessun membro), `PreparingRebalance` o `CompletingRebalance` (le partizioni vengono ridistribuite) e `Dead` (il gruppo non esiste più).
- **Membri** — il numero di membri attivi.
- `Assignor` — la strategia usata per distribuire le partizioni, per esempio `range`.
- **Lag** — il lag totale del gruppo. Vedi [Lag](groups.md#lag).
- **Topic** — i topic per cui il gruppo ha offset sottoposti a commit.

Digita in **Cerca gruppi** per mostrare i gruppi il cui ID contiene il testo; maiuscole e minuscole non contano. L’elenco degli stati accanto mostra solo i gruppi in un determinato stato; **Tutti gli stati** rimuove questo filtro. Il contatore indica quanti gruppi sono mostrati e quanti ce ne sono in totale.

Membri, assignor e lag vengono caricati solo per le righe visibili, man mano che scorri. Finché non arrivano, la cella mostra `…`. Se non si possono caricare, la cella mostra `—`; passaci sopra con il puntatore per vedere il motivo.

**Aggiorna** ricarica l’elenco e i suoi dati. Se alcuni broker non hanno risposto, un avviso indica che l’elenco potrebbe essere incompleto.

<a id="lag"></a>

## Lag

Il lag è il numero di messaggi che il gruppo non ha ancora letto. Per ogni partizione è la fine del log meno l’offset sottoposto a commit. Il lag non viene mai mostrato sotto lo zero.

Il lag dipende solo dagli offset sottoposti a commit. Un consumer attivo può aver letto oltre l’ultimo commit, quindi il suo lag reale può essere più piccolo. Quando il gruppo esegue di nuovo il commit, il lag diminuisce.

Una partizione senza offset sottoposto a commit mostra **nessun commit** e non rientra nel totale. Il lag totale di un gruppo mostra `—` quando il gruppo non ha alcun commit (**nessun commit**) o quando non è stato possibile leggere gli offset di una partizione (**lag sconosciuto: errore negli offset delle partizioni**).

Il lag conta gli offset, non i messaggi. Nei topic con compattazione o transazioni, alcuni offset non contengono messaggi visibili, quindi al gruppo possono restare da leggere meno messaggi di quanto indichi il lag.

I valori sono un’istantanea al momento del caricamento. Per seguire il lag, apri il gruppo e attiva l’aggiornamento automatico.

<a id="group-details"></a>

## Dettagli del gruppo

Fai clic su un gruppo nell’elenco per aprirne la scheda. In alto, un riepilogo mostra lo stato, il numero di membri e l’assignor.

- **Autoaggiornamento** — **Autoaggiornamento: disattivato**, **ogni 5 s** o **ogni 10 s**.
- **Aggiorna** — ricarica subito.
- **Leggi come membro** — avvia una [sessione consumer](consumer-sessions.md) in questo gruppo. Il modulo della sessione riceve l’ID del gruppo e i topic per cui il gruppo ha commit. Il pulsante è disattivato sulle connessioni con l’opzione **Sola lettura** attiva; passa il puntatore sul pulsante per vedere il motivo.

**Partizioni** elenca le partizioni per cui il gruppo ha commit e quelle assegnate ai suoi membri. Il lag totale del gruppo è mostrato sopra la tabella. Le colonne sono **Topic**, **Partizione**, `Committed` (l’offset sottoposto a commit), `Log end` (l’offset finale della partizione), **Lag** e **Membro** (il client ID del membro che legge la partizione).

**Membri** elenca i membri attivi: `Client ID`, **Host**, `Member ID` e **Assegnazione** (le partizioni del membro nella forma `topic:partition`). Un gruppo senza membri mostra **Nessun membro attivo.**

La seconda riga contiene **Reimposta gli offset** e **Elimina gruppo**. È nascosta sulle connessioni in sola lettura.

<a id="reset-offsets"></a>

## Reimpostare gli offset

La reimpostazione sposta gli offset sottoposti a commit di un gruppo. Il gruppo rilegge così dei messaggi oppure li salta. Gli offset si possono reimpostare solo per un gruppo senza membri attivi (stato `Empty` o `Dead`). Altrimenti i membri sovrascriverebbero i nuovi offset con i propri commit. Arresta prima le applicazioni del gruppo.

1. Nella scheda del gruppo, fai clic su **Reimposta gli offset**. Finché il gruppo ha membri, il pulsante è disattivato; passa il puntatore sul pulsante per vedere il motivo.
2. Scegli la **Strategia**. Vedi [Strategie di reimpostazione](groups.md#reset-strategies).
3. Scegli un **Ambito**. Vedi [Ambito della reimpostazione](groups.md#reset-scope).
4. Fai clic su **Anteprima**.
5. Controlla la tabella e fai clic sul pulsante di applicazione, che mostra il numero di partizioni che cambieranno.

L’anteprima non modifica nulla. Per ogni partizione mostra il topic, la partizione, **Attuale** (l’offset sottoposto a commit), **Nuovo** (l’offset di destinazione) e `Low / High` (il primo offset e l’offset finale della partizione). Una partizione per cui non si può calcolare una destinazione è contrassegnata da **Nessun offset di destinazione — non verrà applicato**, con il motivo. Queste partizioni vengono saltate.

Viene applicato solo il piano mostrato nell’anteprima. Qualsiasi modifica nel modulo nasconde l’anteprima e occorre fare di nuovo clic su **Anteprima**. Su una connessione prod, l’applicazione chiede il nome della connessione; sulle altre connessioni avviene subito.

Prima di applicare, l’app verifica di nuovo che il gruppo non abbia membri. Dopo l’applicazione, **Risultato** elenca ogni partizione con il nuovo offset o un errore.

Puoi anche preparare una reimpostazione a partire da un messaggio: vedi [Reimpostare un gruppo su un messaggio](browse.md#reset-group-here). **Chiudi** chiude il pannello di reimpostazione.

<a id="reset-strategies"></a>

## Strategie di reimpostazione

**Strategia** stabilisce l’offset di destinazione per ogni partizione dell’ambito:

- **Dall’inizio (earliest)** — il primo offset ancora conservato. Il gruppo rilegge tutto ciò che il topic conserva.
- **Dalla fine (latest)** — la fine della partizione. Il gruppo salta tutto ciò che si è accumulato e legge solo i nuovi messaggi.
- **Per data e ora** — il primo messaggio scritto alla data e ora indicate in **Data e ora (locale)** o successivamente. Nelle partizioni senza messaggi più recenti, la destinazione è la fine.
- **Offset specifico** — il valore indicato in **Offset**, un intero pari o superiore a 0, in ogni partizione. Se una partizione non ha questo offset, la destinazione è il suo primo offset o il suo offset finale.
- **Sposta di ±N** — sposta l’offset sottoposto a commit del valore di **Spostamento (±N)**. Un numero negativo fa rileggere N messaggi indietro, uno positivo fa saltare N messaggi. Il risultato resta compreso tra il primo offset e l’offset finale. Le partizioni senza offset sottoposto a commit non si possono spostare e vengono saltate.

L’ora è la tua ora locale, con i secondi. Kafka trova l’offset in base ai timestamp dei messaggi.

<a id="reset-scope"></a>

## Ambito della reimpostazione

**Ambito** stabilisce quali partizioni vengono reimpostate. Le altre partizioni mantengono i propri offset.

- **Intero gruppo** — tutte le partizioni per cui il gruppo ha offset sottoposti a commit. Se il gruppo non ha ancora commit, scegli invece i topic.
- **Topic** — tutte le partizioni dei topic elencati, comprese quelle senza commit. Separa i nomi con virgole, spazi o a capo. Usalo per far iniziare a un gruppo la lettura di un nuovo topic da un punto scelto.
- **Partizioni** — solo le partizioni elencate. Inseriscile in **Partizioni (topic:partizione)** nella forma `topic:partition`, separate da virgole o spazi, per esempio `orders:0, orders:3`.

<a id="delete-group"></a>

## Eliminare un gruppo

L’eliminazione di un gruppo rimuove tutti i suoi offset sottoposti a commit. Si può eliminare solo un gruppo senza membri, nello stato `Empty`.

1. Nella scheda del gruppo, fai clic su **Elimina gruppo**. Finché il gruppo ha membri, il pulsante è disattivato; passa il puntatore sul pulsante per vedere il motivo.
2. Digita l’ID del gruppo per confermare. Su una connessione prod, digita anche il nome della connessione.
3. Fai clic su **Elimina**.

Dopo l’eliminazione, la scheda del gruppo si chiude. Se un’applicazione con questo `group.id` si avvia di nuovo, Kafka ricrea il gruppo. Senza offset sottoposti a commit, l’applicazione inizia dal punto indicato dalla sua impostazione `auto.offset.reset`: dall’inizio o dalla fine.

Sulle connessioni con l’opzione **Sola lettura** attiva, il pulsante è nascosto.
