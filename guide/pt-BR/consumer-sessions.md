# Sessões de consumer

Uma sessão de consumer lê tópicos como um membro real de um consumer group.

<a id="what-is"></a>

## O que é uma sessão

A [navegação](browse.md) lê as mensagens sem entrar em um grupo e não altera nada no cluster. Uma sessão é diferente: ela entra em um grupo com o `group.id` indicado, recebe partições do Kafka e pode fazer commit de offsets. Use-a para ler o que um grupo leria, para mover os offsets de um grupo por meio de commits ou para observar como um grupo faz o rebalanceamento.

Uma sessão afeta o cluster. Quando ela entra ou sai, o Kafka redistribui as partições do grupo, e os outros membros passam por um rebalanceamento. Os commits alteram os offsets do grupo, que os aplicativos dele vão usar.

As sessões não estão disponíveis nas conexões com a opção **Somente leitura** ativada: a aba mostra **As sessões de consumer não estão disponíveis para conexões somente leitura: entrar em um grupo altera o estado dele.** Se uma conexão passar a ser somente leitura durante uma sessão, a sessão sai do grupo.

<a id="start-session"></a>

## Iniciar uma sessão

Abra uma sessão de uma destas três formas:

- **Sessão de consumer** abaixo da conexão no painel **Conexões** — um formulário vazio;
- **Ler como grupo** na aba de um tópico — o tópico vem preenchido;
- **Ler como membro** na aba de um grupo — o grupo e os tópicos em que ele tem commits vêm preenchidos.

A cada vez, uma nova aba é aberta, então várias sessões podem rodar ao mesmo tempo.

Na seção **Grupo e tópicos**:

- `group.id` — o grupo em que entrar. Se você não informar um, o aplicativo sugere um novo nome, como `glafira-kafka-anna-1a2b3c`. Um grupo novo não tem commits.
- **Tópicos** — nomes de tópicos separados por vírgulas ou espaços. Ou marque **tópicos por regex** e informe a expressão em **Expressão regular dos tópicos**, por exemplo `orders\..*`. A expressão é comparada a partir do início do nome: `^` é acrescentado se estiver faltando. Os tópicos correspondentes criados depois são adicionados em poucos minutos (na atualização dos metadados); isso causa um rebalanceamento.
- `auto.offset.reset` — onde começar nas partições em que o grupo não tem offset com commit: **do fim (latest)** (o padrão) ou **do início (earliest)**. Onde o grupo tem um offset com commit, a leitura continua a partir dele.
- **commit automático** — faz o commit dos offsets lidos automaticamente. Desativado por padrão. Consulte [Commits](consumer-sessions.md#commits).
- `isolation.level` — `read_committed` (o padrão) mostra apenas as mensagens de transações confirmadas. `read_uncommitted` também mostra as mensagens de transações abertas e abortadas.

Na seção **Formato**, defina **Formato da chave** e **Formato do valor**, como na navegação. Consulte [Formatos](browse.md#formats). O campo **Buffer, mensagens** define quantas mensagens a tabela mantém, de 1 a 100000; o padrão é 5000.

Os filtros funcionam como na navegação. Consulte [Filtros](browse.md#filters).

Clique em **Entrar no grupo**. Em uma conexão prod, o aplicativo pede que você digite o nome da conexão e confirme com **Confirmar**. Enquanto a sessão está em execução, o formulário não pode ser alterado.

<a id="reading"></a>

## Leitura

A linha de estado ao lado dos botões mostra **conectando…** e depois **no grupo**. Ela também mostra quantas mensagens foram lidas, quantas correspondem aos filtros e quantas estão na tabela. Se a conexão com o broker cair, a sessão se reconecta sozinha e indica isso na linha de estado.

As mensagens aparecem na tabela à medida que chegam, e a sessão continua lendo as novas mensagens até você interrompê-la. Quando a tabela fica cheia, as mensagens mais antigas são removidas. Clique em uma linha para ver os detalhes da mensagem, como na navegação. Consulte [Detalhes da mensagem](browse.md#message-details). Selecione `UTC` para mostrar os horários em UTC.

A sessão lê apenas as partições que o Kafka atribuiu a ela. Se o grupo tiver outros membros, eles leem as demais partições. Se o grupo tiver mais membros do que partições, a sessão pode não receber nenhuma partição.

**Desconectar** sai do grupo, e os outros membros recebem as partições da sessão. Fechar a aba da sessão também a faz sair do grupo. Quando a sessão termina, a linha de estado mostra **desconectado**.

Se um tópico não existir, a sessão para com um erro e mostra os nomes a verificar.

<a id="commits"></a>

## Commits

Um commit salva um offset para o grupo. O próximo membro que ler a partição, ou o mesmo aplicativo depois de reiniciar, começa a partir desse offset.

Com a opção **commit automático** ativada, a sessão faz o commit dos offsets lidos em segundo plano a cada poucos segundos e mais uma vez ao sair do grupo. As mensagens ocultadas pelos filtros contam como lidas.

Sem o commit automático, a sessão não faz commit de nada sozinha. Use os botões:

- **Fazer commit até o selecionado** — faz o commit do offset seguinte à mensagem selecionada, apenas na partição dela. O grupo continuará a partir da próxima mensagem. Se a mensagem selecionada for mais antiga que o commit atual, o offset volta para trás.
- **Fazer commit de tudo o que foi lido** — faz o commit da posição atual em todas as partições atribuídas. Isso inclui as mensagens ocultadas pelos filtros e as que já foram removidas da tabela.

Depois de um commit, uma linha mostra os offsets com commit como `topic:partition → offset`. Em uma conexão prod, cada commit pede o nome da conexão.

Se a sessão não responder a tempo, o aplicativo avisa que não se sabe se o commit foi aplicado. Verifique os offsets na aba do grupo. Consulte [Detalhes do grupo](groups.md#group-details).

<a id="pause-partitions"></a>

## Pausar partições

O painel à direita da tabela lista as partições atribuídas à sessão como `topic:partition`. Clique em **Pausar** ao lado de uma partição para parar de lê-la e em **Retomar** para continuar. Uma partição pausada aparece como **em pausa**.

Pausar não faz a sessão sair do grupo, então não há rebalanceamento. A partição continua atribuída à sessão, e os outros membros não a leem enquanto ela está pausada. Pausar não altera nada no cluster e não exige confirmação.

Depois de um rebalanceamento, as partições recém-atribuídas começam sem pausa. Quando a sessão sai do grupo, todas as pausas terminam.

<a id="rebalance-log"></a>

## Log de rebalanceamentos

O **Log de rebalanceamentos**, abaixo da lista de partições, registra cada mudança na atribuição, das mais recentes para as mais antigas. Cada entrada tem um horário e um de três tipos: partições atribuídas à sessão, partições revogadas dela ou um erro de rebalanceamento com o texto dele.

Um rebalanceamento acontece quando um membro entra no grupo ou sai dele, quando os tópicos assinados mudam ou quando um membro deixa de responder a tempo. Para observá-lo, inicie uma segunda sessão com o mesmo `group.id` em outra aba: as partições são divididas entre as duas sessões, e os dois logs mostram isso.

O log mantém as últimas 200 entradas. Antes do primeiro rebalanceamento, ele mostra **Nenhum rebalanceamento até agora.**

<a id="limits"></a>

## Limitações

- As sessões não estão disponíveis nas conexões com a opção **Somente leitura** ativada.
- Uma sessão não pode começar a partir de um offset escolhido. Para definir o início, redefina antes os offsets do grupo: consulte [Redefinir offsets](groups.md#reset-offsets). A redefinição exige um grupo sem membros, então desconecte as sessões desse grupo antes de redefinir.
- A tabela mantém no máximo 100000 mensagens. As mensagens mais antigas são removidas, mas continuam contando como lidas.
- Chaves e valores muito grandes são mostrados apenas em parte, como na navegação.
- Não entre no grupo de um aplicativo em execução, a menos que seja essa a intenção. A sessão toma partições do aplicativo, e o aplicativo não as processa enquanto a sessão as mantém. Os commits da sessão, inclusive os automáticos, movem os offsets do aplicativo.
