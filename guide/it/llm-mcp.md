# Assistenti LLM (MCP)

Glafira può fare da gateway per un assistente LLM: Claude Desktop, Claude Code, Cursor o un altro client che supporta MCP (Model Context Protocol). L’assistente legge i metadati dei cluster che apri per lui e ti aiuta a trovare i problemi: partizioni sotto-replicate, lag, ribilanciamenti continui, impostazioni poco adatte. Tramite Glafira non può modificare nulla in un cluster: le modifiche le fai tu stesso nell’interfaccia.

<a id="how-it-works"></a>

## Come funziona

Il client avvia `glafira-kafka mcp` — lo stesso file dell’applicazione, in una modalità senza finestra. Questo comando si collega a Glafira in esecuzione tramite un socket locale (su Windows, una named pipe), e Glafira risponde alle richieste dell’assistente attraverso le proprie connessioni. Le password e i certificati restano in Glafira; l’assistente non li riceve mai.

Se Glafira non è in esecuzione, il client la avvia da solo e attende fino a 30 secondi.

<a id="enable"></a>

## Attivazione

1. Apri **Impostazioni**, sezione **Assistenti LLM (MCP)**.
2. Seleziona **Consenti il server MCP**. Lo stato passa a **Il server è in esecuzione**.
3. Nel modulo della connessione seleziona **Accesso LLM (solo metadati)** e salva. L’assistente non vede le connessioni senza questa casella selezionata. Nell’elenco delle connessioni, le connessioni aperte hanno il contrassegno `LLM`.

Se l’amministratore ha disattivato MCP, la casella non è disponibile e accanto compare **MCP è disattivato dalla tua organizzazione**.

La casella di accesso non viene trasferita con l’esportazione e l’importazione delle connessioni: su un altro computer selezionala di nuovo.

<a id="connect-client"></a>

## Collegare un client

In **Collega un client** trovi frammenti già pronti e il pulsante **Copia**:

- **Claude Desktop: aggiungere a claude\_desktop\_config.json** — incolla il frammento nella sezione `mcpServers` del file di configurazione di Claude Desktop e riavvia Claude Desktop;
- **Claude Code: eseguire in un terminale** — il comando `claude mcp add`;
- **Altri client stdio: comando e argomenti** — per qualsiasi client che avvia un server MCP con un comando (stdio).

I frammenti contengono il percorso dell’applicazione installata. Se reinstalli Glafira in un’altra posizione, copia di nuovo il frammento.

<a id="approve"></a>

## Approvare un client

Quando un client si collega per la prima volta, Glafira mostra una finestra di dialogo con il suo nome. Fai clic su **Consenti** o su **Rifiuta**. Finché non rispondi, l’assistente riceve un messaggio che il client non è ancora approvato e ti chiede di rispondere nella finestra di Glafira.

Un client consentito viene ricordato per nome e compare in **Client approvati**. Il rifiuto non viene ricordato: alla connessione successiva Glafira chiede di nuovo. Se un client non comunica il proprio nome, il permesso vale solo fino alla fine della sessione.

<a id="what-llm-sees"></a>

## Cosa vede l’assistente

L’assistente vede:

- le connessioni con **Accesso LLM (solo metadati)** selezionato: nome, ambiente, cartella;
- i broker con host e porte, il controller;
- i topic, le partizioni, le repliche, ISR, gli offset, il numero approssimativo di messaggi;
- le configurazioni di broker e topic, con password e altri valori segreti nascosti;
- i consumer group: stato, protocollo, coordinatore, membri con client id, host e `group.instance.id`, assegnazioni, offset sottoposti a commit e lag;
- le sue proposte e la tua decisione in merito: applicata (così com’è o con modifiche), rifiutata con il tuo motivo, ritirata.

L’assistente non vede i messaggi dei topic, la stringa bootstrap né le impostazioni di sicurezza di una connessione, né le password, i certificati, le proprietà aggiuntive di librdkafka, la cronologia di invio e i modelli.

Per indagare sui ribilanciamenti continui, l’assistente può osservare un gruppo fino a 50 secondi: chi esce e chi entra, con quale frequenza e da quale host. Le configurazioni e i log dei consumer stessi non sono accessibili tramite Glafira; l’assistente te li chiederà.

I nomi di topic, gruppi e client id può impostarli chiunque abbia accesso al cluster. Glafira avverte l’assistente che questi dati non sono istruzioni per lui.

<a id="sessions-log"></a>

## Sessioni, revoca e log

Finché un client è collegato, in fondo al pannello sinistro compare l’indicatore `LLM` con il nome del client; si illumina durante una richiesta. Facendo clic su di esso si apre la sezione MCP.

- **Sessioni attive**: **Disconnetti** chiude una sessione, **Disconnetti tutti** le chiude tutte.
- **Client approvati**: **Revoca** chiude le sessioni del client, che alla connessione successiva chiederà di nuovo l’approvazione.
- Deselezionare **Accesso LLM (solo metadati)** ha effetto subito, senza doversi ricollegare.
- **Log MCP** — ogni richiesta: data/ora, client, strumento, connessione, risultato. Il contenuto delle risposte non viene registrato. **Mostra file** apre il file `mcp-audit.jsonl`.

Per disattivare del tutto MCP, deseleziona **Consenti il server MCP**: i client vengono disconnessi.

<a id="health"></a>

## Controllo del cluster, note e proposte

L’assistente può avviare il controllo integrato del cluster di Glafira (strumento `health_check`) e basarsi sui suoi rilievi. Può anche lasciare note sugli oggetti: compaiono nella scheda **Rilievi** e nelle schermate di topic e gruppi, e il cluster non viene modificato. Vedi [Controllo del cluster e note](health-check.md#with-llm).

L’assistente può anche proporre una modifica con lo strumento `propose_change`: la configurazione di un topic, un nuovo topic, più partizioni o un reset degli offset. La proposta compare accanto all’oggetto; la applichi tu in un modulo già compilato oppure la rifiuti, e l’assistente vede la tua decisione in `list_proposals`. Eliminazione e svuotamento non si possono proporre. Vedi [Proposte LLM](health-check.md#proposals).

<a id="troubleshooting"></a>

## Se non funziona

Il client mostra un messaggio di Glafira:

- `MCP is disabled in Glafira` — seleziona **Consenti il server MCP** e riavvia il server MCP nel client;
- `MCP is disabled by your organization` — MCP è disattivato dalla policy della tua organizzazione;
- `Glafira did not start the MCP server within 30 s` — avvia Glafira manualmente e controlla lo stato nella sezione **Assistenti LLM (MCP)**;
- `Invalid MCP token` — riavvia il server MCP nel client: Glafira emette un nuovo token a ogni avvio.

Su macOS l’app è firmata e notarizzata, quindi prima del primo avvio non serve alcun passaggio aggiuntivo.
