# Sessioni consumer

Una sessione consumer legge i topic come un vero membro di un consumer group.

<a id="what-is"></a>

## Che cos’è una sessione

La [consultazione](browse.md) legge i messaggi senza entrare in un gruppo e non modifica nulla nel cluster. Una sessione è diversa: entra in un gruppo con il `group.id` indicato, riceve le partizioni da Kafka e può eseguire il commit degli offset. Usala per leggere ciò che leggerebbe un gruppo, per spostare gli offset di un gruppo con i commit o per osservare il ribilanciamento di un gruppo.

Una sessione influisce sul cluster. Quando entra o esce, Kafka ridistribuisce le partizioni del gruppo e gli altri membri eseguono un ribilanciamento. I commit modificano gli offset del gruppo, che le sue applicazioni useranno.

Le sessioni non sono disponibili sulle connessioni con l’opzione **Sola lettura** attiva: la scheda mostra **Le sessioni consumer non sono disponibili per le connessioni in sola lettura: unirsi a un gruppo ne modifica lo stato.** Se una connessione passa in sola lettura mentre una sessione è in corso, la sessione esce dal gruppo.

<a id="start-session"></a>

## Avviare una sessione

Una sessione si apre in tre modi:

- **Sessione consumer** sotto la connessione nel pannello **Connessioni** — un modulo vuoto;
- **Leggi come gruppo** nella scheda di un topic — il topic è già compilato;
- **Leggi come membro** nella scheda di un gruppo — sono già compilati il gruppo e i topic per cui ha commit.

Ogni volta si apre una nuova scheda, quindi più sessioni possono essere in corso contemporaneamente.

In **Gruppo e topic**:

- `group.id` — il gruppo in cui entrare. Se non ne inserisci uno, l’app propone un nuovo nome come `glafira-kafka-anna-1a2b3c`. Un nuovo gruppo non ha commit.
- **Topic** — i nomi dei topic separati da virgole o spazi. In alternativa, seleziona **topic per regex** e inserisci un’espressione in **Espressione regolare dei topic**, per esempio `orders\..*`. L’espressione viene confrontata dall’inizio del nome: se manca, viene aggiunto `^`. I topic corrispondenti creati in seguito vengono aggiunti entro qualche minuto (all’aggiornamento dei metadati); questo causa un ribilanciamento.
- `auto.offset.reset` — da dove iniziare nelle partizioni per cui il gruppo non ha un offset sottoposto a commit: **dalla fine (latest)** (il valore predefinito) o **dall’inizio (earliest)**. Dove il gruppo ha un offset sottoposto a commit, la lettura riprende da quello.
- **commit automatico** — esegue automaticamente il commit degli offset letti. Disattivato per impostazione predefinita. Vedi [Commit](consumer-sessions.md#commits).
- `isolation.level` — `read_committed` (il valore predefinito) mostra solo i messaggi delle transazioni confermate. `read_uncommitted` mostra anche i messaggi delle transazioni aperte e annullate.

In **Formato** scegli **Formato della chiave** e **Formato del valore**, come nella consultazione. Vedi [Formati](browse.md#formats). **Buffer, messaggi** stabilisce quanti messaggi conserva la tabella, da 1 a 100000; il valore predefinito è 5000.

I filtri funzionano come nella consultazione. Vedi [Filtri](browse.md#filters).

Fai clic su **Unisciti al gruppo**. Su una connessione prod, l’app chiede di digitare il nome della connessione e di confermare con **Conferma**. Mentre la sessione è in corso, il modulo non si può modificare.

<a id="reading"></a>

## Lettura

La riga di stato accanto ai pulsanti mostra **connessione…**, poi **nel gruppo**. Mostra anche quanti messaggi sono stati letti, quanti corrispondono ai filtri e quanti sono in tabella. Se la connessione al broker si interrompe, la sessione si riconnette da sola e lo segnala nella riga di stato.

I messaggi compaiono nella tabella man mano che arrivano, e la sessione continua a leggere i nuovi messaggi finché non la interrompi. Quando la tabella è piena, i messaggi più vecchi vengono rimossi. Fai clic su una riga per vedere i dettagli del messaggio, come nella consultazione. Vedi [Dettagli del messaggio](browse.md#message-details). Seleziona `UTC` per mostrare le ore in UTC.

La sessione legge solo le partizioni che Kafka le ha assegnato. Se il gruppo ha altri membri, questi leggono le altre partizioni. Se il gruppo ha più membri che partizioni, la sessione potrebbe non ricevere alcuna partizione.

**Disconnetti** fa uscire la sessione dal gruppo, e gli altri membri ricevono le sue partizioni. Anche la chiusura della scheda della sessione fa uscire dal gruppo. Quando la sessione è terminata, la riga di stato mostra **disconnesso**.

Se un topic non esiste, la sessione si arresta con un errore e mostra i nomi da controllare.

<a id="commits"></a>

## Commit

Un commit salva un offset per il gruppo. Il membro successivo che legge la partizione, o la stessa applicazione dopo un riavvio, inizia da quell’offset.

Con l’opzione **commit automatico** attiva, la sessione esegue il commit degli offset letti in background a intervalli di pochi secondi, e un’ultima volta quando esce dal gruppo. I messaggi nascosti dai filtri contano come letti.

Senza commit automatico, la sessione non esegue alcun commit da sola. Usa i pulsanti:

- **Esegui il commit fino al messaggio selezionato** — esegue il commit dell’offset successivo al messaggio selezionato, solo nella sua partizione. Il gruppo riprenderà dal messaggio seguente. Se il messaggio selezionato è più vecchio del commit attuale, l’offset torna indietro.
- **Esegui il commit dei messaggi letti** — esegue il commit della posizione attuale in ogni partizione assegnata. Sono compresi i messaggi nascosti dai filtri e quelli già rimossi dalla tabella.

Dopo un commit, una riga mostra gli offset sottoposti a commit nella forma `topic:partition → offset`. Su una connessione prod, ogni commit chiede il nome della connessione.

Se la sessione non risponde in tempo, l’app avvisa che non è noto se il commit sia stato applicato. Controlla gli offset nella scheda del gruppo. Vedi [Dettagli del gruppo](groups.md#group-details).

<a id="pause-partitions"></a>

## Mettere in pausa le partizioni

Il pannello a destra della tabella elenca le partizioni assegnate alla sessione nella forma `topic:partition`. Fai clic su **Metti in pausa** accanto a una partizione per interromperne la lettura e su **Riprendi** per continuare. Una partizione in pausa è contrassegnata da **in pausa**.

La pausa non fa uscire dal gruppo, quindi non c’è ribilanciamento. La partizione resta assegnata alla sessione e, finché è in pausa, gli altri membri non la leggono. La pausa non modifica nulla nel cluster e non richiede conferma.

Dopo un ribilanciamento, le partizioni appena assegnate non sono in pausa. Quando la sessione esce dal gruppo, tutte le pause terminano.

<a id="rebalance-log"></a>

## Registro dei ribilanciamenti

**Registro ribilanciamenti** sotto l’elenco delle partizioni registra ogni cambiamento dell’assegnazione, dal più recente. Ogni voce ha un’ora e uno di tre tipi: partizioni assegnate alla sessione, partizioni revocate alla sessione o un errore di ribilanciamento con il relativo testo.

Un ribilanciamento avviene quando un membro entra nel gruppo o ne esce, quando cambiano i topic sottoscritti o quando un membro smette di rispondere in tempo. Per osservarlo, avvia in un’altra scheda una seconda sessione con lo stesso `group.id`: le partizioni vengono suddivise tra le due sessioni, ed entrambi i registri lo mostrano.

Il registro conserva le ultime 200 voci. Prima del primo ribilanciamento mostra **Nessun ribilanciamento finora.**

<a id="limits"></a>

## Limiti

- Le sessioni non sono disponibili sulle connessioni con l’opzione **Sola lettura** attiva.
- Una sessione non può iniziare da un offset scelto. Per impostare il punto di partenza, reimposta prima gli offset del gruppo: vedi [Reimpostare gli offset](groups.md#reset-offsets). La reimpostazione richiede un gruppo senza membri, quindi disconnetti le sessioni di questo gruppo prima di reimpostare.
- La tabella conserva al massimo 100000 messaggi. I messaggi più vecchi vengono rimossi, ma contano comunque come letti.
- Chiavi e valori molto grandi vengono mostrati solo in parte, come nella consultazione.
- Non entrare nel gruppo di un’applicazione in esecuzione se non è ciò che vuoi. La sessione sottrae partizioni all’applicazione, che non le elabora finché la sessione le detiene. I commit della sessione, compresi quelli automatici, spostano gli offset dell’applicazione.
