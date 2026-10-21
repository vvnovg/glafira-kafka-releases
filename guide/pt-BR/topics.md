# Cluster e tópicos

Este capítulo trata da visão geral do cluster, da lista de tópicos, da criação e exclusão de tópicos, da tela do tópico e da configuração de tópicos. Abaixo de cada conexão no painel **Conexões** há links para **Visão geral do cluster** e **Tópicos**. Cada um abre em sua própria aba.

<a id="cluster-overview"></a>

## Visão geral do cluster

A tela **Visão geral do cluster** mostra o estado do cluster no momento em que foi carregada. Clique em **Atualizar** para carregá-la de novo.

- `Cluster ID` — o identificador do cluster.
- **Controlador** — o ID do broker que o cluster informa como controlador. Em clusters KRaft, os clientes não se comunicam diretamente com os controladores: aqui o cluster informa um broker aleatório, não o controlador ativo, e esse broker pode mudar de uma atualização para outra.
- **Brokers** — o número de brokers.
- **Tópicos (internos)** — todos os tópicos, com o número de tópicos internos entre parênteses.
- **Partições** — o número de partições em todos os tópicos.
- `Under-replicated` — partições em que nem todas as réplicas estão sincronizadas (no ISR). Normalmente, um broker está fora do ar ou não consegue acompanhar.
- `Offline` — partições sem leader. Ninguém consegue gravar nelas nem ler delas.

Um valor diferente de zero em `Under-replicated` ou `Offline` fica destacado.

Abaixo fica a tabela de brokers, com `ID` e **Endereço**. O broker do campo **Controlador** é marcado com **controlador**. Em clusters KRaft, é o mesmo broker aleatório, não o controlador ativo. Clique em um broker para ver a configuração dele abaixo da tabela. A configuração do broker pode apenas ser consultada. Use o campo **Pesquisar parâmetro** para encontrar um parâmetro pelo nome e a opção **Somente alterados** para ocultar os parâmetros com valores padrão. Um ícone de cadeado marca os parâmetros somente leitura.

<a id="topic-list"></a>

## Lista de tópicos

A tela **Tópicos** lista os tópicos do cluster. Clique em uma linha para abrir o tópico em sua própria aba.

As colunas são **Nome**, **Partições**, `RF` (fator de replicação), **Mensagens (≈)**, `cleanup.policy` e **Sub-repl.** (o número de partições sub-replicadas). A contagem de mensagens é aproximada: é a diferença entre os offsets final e inicial das partições. A compactação e os marcadores de transação tornam o número real menor. A contagem de mensagens e o valor de `cleanup.policy` são carregados apenas para as linhas que ficam visíveis durante a rolagem.

Controles da barra de ferramentas:

- **Pesquisar (ou /regex/)** — digite parte de um nome para encontrar tópicos; maiúsculas e minúsculas não importam. Para pesquisar com uma expressão regular, coloque-a entre barras, por exemplo `/^orders\..*/`.
- **Internos** — mostra os tópicos internos, como `__consumer_offsets`. Eles são marcados com **(interno)**.
- A lista de atualização automática — **Atualização automática: desativada**, **a cada 10 s** ou **a cada 30 s**.
- **Atualizar** — recarrega a lista e a contagem de mensagens na hora.

Ao lado da barra de ferramentas, o aplicativo mostra quantos tópicos correspondem à pesquisa em relação ao total. Nas conexões com a opção **Somente leitura** ativada, os botões **Criar tópico** e **Selecionar** ficam ocultos.

<a id="create-topic"></a>

## Criar um tópico

1. Na lista de tópicos, clique em **Criar tópico**. O formulário **Novo tópico** é aberto acima da lista.
2. Em **Nome do tópico**, digite o nome. Ele pode conter letras latinas, dígitos, `.`, `_` e `-`, com até 249 caracteres. O formulário avisa se o nome tiver `.` e `_` ao mesmo tempo: os nomes das métricas do Kafka desses tópicos podem colidir com os de outros tópicos.
3. Em **Partições**, defina o número de partições. As partições são lidas em paralelo, e a ordem é garantida apenas dentro de uma partição. O número pode ser aumentado depois, mas nunca reduzido.
4. Em **Fator de replicação**, defina quantas cópias de cada partição os brokers mantêm. O valor não pode ser maior que o número de brokers, que o formulário mostra abaixo do campo. O padrão é 3, ou o número de brokers, se houver menos. `-1` usa o padrão do broker.
5. Em **Predefinição**, escolha uma opção:
   - **delete (padrão)** — as mensagens antigas são excluídas por tempo.
   - `compact` — só o valor mais recente de cada chave é mantido.
   - `compact + delete` — as duas coisas.
   - **teste (retenção 1 h)** — as mensagens são mantidas por uma hora.
6. Se quiser, adicione parâmetros em **Configurações adicionais (key=value)**, um por linha, por exemplo `retention.ms=86400000`. Eles substituem os da predefinição. Linhas que começam com `#` são ignoradas.
7. Clique em **Criar**.

Quando o formulário é válido, ele mostra o comando `kafka-topics.sh` equivalente. Você pode copiá-lo, por exemplo, para criar o mesmo tópico em outro cluster.

Em uma conexão prod, você confirma a criação digitando o nome da conexão. Depois que o tópico é criado, a aba dele é aberta.

<a id="delete-topics"></a>

## Excluir tópicos

Para excluir um tópico, abra-o e clique em **Excluir tópico**. Digite o nome do tópico para confirmar e clique em **Excluir**. A aba do tópico é fechada.

Para excluir vários tópicos de uma vez:

1. Na lista de tópicos, clique em **Selecionar**. Aparecem caixas de seleção ao lado dos nomes.
2. Selecione os tópicos. Os tópicos internos não podem ser selecionados.
3. Clique no botão de exclusão. Ele mostra o número de tópicos selecionados.
4. Confira a lista de nomes, digite a quantidade para confirmar e clique em **Excluir**.

**Cancelar seleção** oculta as caixas de seleção.

Em uma conexão prod, as duas formas de exclusão também pedem que você digite o nome da conexão.

A exclusão não pode ser desfeita: os dados são perdidos. Se alguns tópicos não forem excluídos, por exemplo por falta de permissão, o aplicativo os lista com o motivo. Os outros tópicos são excluídos.

<a id="topic-details"></a>

## Tela do tópico

A aba do tópico mostra o número de partições, o fator de replicação e a contagem aproximada de mensagens. Os botões na parte superior:

- **Atualizar** — recarrega os dados do tópico.
- **Enviar mensagem** — abre a tela **Producer** com este tópico. Consulte [Producer](producer.md).
- **Ler como grupo** — inicia uma sessão de consumer para este tópico. Consulte [Sessões de consumer](consumer-sessions.md).

Nas conexões com a opção **Somente leitura** ativada, os dois botões ficam desativados.

A segunda linha tem as ações que alteram o tópico. Ela fica oculta nas conexões somente leitura e para os tópicos internos.

- **Adicionar partições** — informe o novo total e clique em **Aumentar**. O número de partições não pode ser reduzido. Depois de um aumento, mensagens com a mesma chave podem ir para outra partição, e a ordem por chave se perde.
- **Limpar** — exclui todas as mensagens até o fim atual das partições selecionadas. O tópico em si permanece. Selecione as partições, digite o nome do tópico e clique em **Limpar partições**. A limpeza só funciona quando `cleanup.policy` inclui `delete`; caso contrário, o botão fica desativado e o motivo é mostrado. Em um tópico `compact,delete`, a limpeza também exclui os valores mais recentes das chaves.
- **Excluir tópico** — consulte [Excluir tópicos](topics.md#delete-topics).

Em uma conexão prod, cada uma dessas ações também pede o nome da conexão.

As abas abaixo:

- **Mensagens** — navegação pelas mensagens. Consulte [Navegar pelas mensagens](browse.md).
- **Partições** — uma tabela com cada partição: **Leader**, **Réplicas**, `ISR` (réplicas sincronizadas), os offsets inicial (`Low`) e final (`High`) e o número de mensagens. Em uma partição sem leader, aparece **nenhum**.
- **Configuração** — os parâmetros do tópico. Consulte [Configuração do tópico](topics.md#topic-config).
- **Consumer groups** — os consumer groups que têm offsets com commit neste tópico, com **Estado**, **Partições com commits** e **Lag**. Clique em um grupo para abri-lo. Consulte [Consumer groups](groups.md).

<a id="topic-config"></a>

## Configuração do tópico

A aba **Configuração** lista todos os parâmetros do tópico em três colunas:

- **Parâmetro** — o nome do parâmetro. Os nomes em negrito estão definidos explicitamente: no tópico ou na configuração do broker.
- **Valor** — o valor atual. Os valores sensíveis ficam ocultos como `••••••`.
- **Origem** — de onde vem o valor. Por exemplo, `DynamicTopic` está definido neste tópico, `StaticBroker` e `DynamicBroker` vêm da configuração do broker, e `Default` é o padrão do Kafka.

Use o campo **Pesquisar parâmetro** para encontrar um parâmetro pelo nome e a opção **Somente alterados** para mostrar apenas os parâmetros em negrito. Um ícone de cadeado marca os parâmetros somente leitura. O ícone `?` ao lado de um nome explica o parâmetro.

<a id="edit-config"></a>

## Alterar a configuração

1. Na aba **Configuração**, clique em **Editar**. Nas conexões com a opção **Somente leitura** ativada, esse botão fica oculto.
2. Altere os valores na coluna **Valor**. Os parâmetros somente leitura e sensíveis não podem ser editados. Para remover o valor próprio do tópico, clique em **restaurar padrão**: passa a valer o valor do broker. Um valor vazio não é permitido.
3. Clique em **Revisar alterações**. O aplicativo mostra cada alteração como valor antigo → valor novo; **padrão** indica que o valor volta ao padrão.
4. Clique em **Aplicar**, ou em **Voltar** para continuar editando.

Nada muda no cluster até você aplicar. **Descartar edições** cancela todas as edições. Enquanto houver edições não salvas, o botão **Fechar editor** fica desativado. Em uma conexão prod, você confirma as alterações digitando o nome da conexão.

Apenas os parâmetros alterados são enviados. Os demais parâmetros do tópico continuam como estão. O broker verifica os valores e rejeita os inválidos com um erro.

<a id="config-parameters"></a>

## Parâmetros do tópico

Os tópicos do Kafka têm dezenas de parâmetros. Os ícones `?` na tabela de configuração explicam cada um deles. Os mais usados são:

- `cleanup.policy` — o que acontece com os dados antigos. `delete` remove os segmentos antigos por tempo ou tamanho. `compact` mantém só o valor mais recente de cada chave. `compact,delete` faz as duas coisas.
- `retention.ms` — por quanto tempo as mensagens são mantidas, em milissegundos. O padrão é `604800000` (7 dias); `-1` as mantém para sempre. Funciona quando `cleanup.policy` inclui `delete`.
- `retention.bytes` — o tamanho máximo de cada partição antes que os segmentos antigos sejam excluídos. O padrão `-1` significa sem limite de tamanho.
- `segment.ms` e `segment.bytes` — quando o broker inicia um novo segmento do log. A retenção e a compactação atuam sobre segmentos inteiros, então uma mensagem pode ficar mais tempo que `retention.ms`, até que o segmento dela seja fechado.
- `min.insync.replicas` — quantas réplicas precisam confirmar uma gravação com `acks=all`. Normalmente 2 com fator de replicação 3.
- `max.message.bytes` — o maior batch que o broker aceita para este tópico. O padrão é de cerca de 1 MB. Os produtores precisam permitir o mesmo tamanho (`max.request.size`).
- `compression.type` — como o broker armazena as mensagens. `producer` mantém a compressão do produtor.
- `message.timestamp.type` — `CreateTime` (o horário definido pelo produtor) ou `LogAppendTime` (o horário em que o broker gravou a mensagem). É esse o horário mostrado na navegação pelas mensagens.
- `delete.retention.ms` — por quanto tempo os marcadores de exclusão (mensagens com valor nulo) permanecem em um tópico compactado. O padrão é 1 dia.

Alguns parâmetros existem apenas em versões mais recentes do Kafka, e alguns estão obsoletos. O ícone `?` avisa quando isso importa. Os parâmetros de tiered storage, como `remote.storage.enable`, só funcionam se o cluster tiver suporte a esse recurso.
