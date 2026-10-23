# Assistentes LLM (MCP)

O Glafira pode funcionar como gateway para um assistente LLM: Claude Desktop, Claude Code, Cursor ou outro cliente com suporte a MCP (Model Context Protocol). O assistente lê os metadados dos clusters que você abrir para ele e ajuda a encontrar problemas: partições sub-replicadas, lag, rebalanceamentos constantes, configurações inadequadas. Pelo Glafira ele não pode alterar nada em um cluster: as alterações você mesmo faz na interface.

<a id="how-it-works"></a>

## Como funciona

O cliente executa `glafira-kafka mcp` — o mesmo arquivo do aplicativo, em um modo sem janela. Esse comando se conecta ao Glafira em execução por um socket local (no Windows, um pipe nomeado), e o Glafira responde às solicitações do assistente por meio das próprias conexões. As senhas e os certificados ficam no Glafira; o assistente nunca os recebe.

Se o Glafira não estiver em execução, o cliente o inicia e espera até 30 segundos.

<a id="enable"></a>

## Ativação

1. Abra **Configurações**, seção **Assistentes LLM (MCP)**.
2. Marque **Permitir o servidor MCP**. O status muda para **O servidor está em execução**.
3. No formulário da conexão, marque **Acesso para LLM (somente metadados)** e salve. O assistente não vê as conexões sem essa caixa marcada. Na lista de conexões, as conexões abertas têm o indicativo `LLM`.

Se o administrador tiver desativado o MCP, a caixa fica indisponível e ao lado aparece **O MCP está desativado pela sua organização**.

A caixa de acesso não é transferida na exportação e na importação de conexões: em outro computador, marque-a de novo.

<a id="connect-client"></a>

## Conectar um cliente

Em **Conectar um cliente** há trechos prontos e o botão **Copiar**:

- **Claude Desktop: adicionar ao claude\_desktop\_config.json** — cole o trecho na seção `mcpServers` do arquivo de configuração do Claude Desktop e reinicie o Claude Desktop;
- **Claude Code: executar em um terminal** — o comando `claude mcp add`;
- **Outros clientes stdio: comando e argumentos** — para qualquer cliente que inicia um servidor MCP por comando (stdio).

Os trechos contêm o caminho do aplicativo instalado. Se você reinstalar o Glafira em outro local, copie o trecho de novo.

<a id="approve"></a>

## Aprovar um cliente

Quando um cliente se conecta pela primeira vez, o Glafira mostra uma caixa de diálogo com o nome dele. Clique em **Permitir** ou em **Rejeitar**. Enquanto você não responde, o assistente recebe uma mensagem de que o cliente ainda não foi aprovado e pede que você responda na janela do Glafira.

Um cliente permitido é lembrado pelo nome e aparece em **Clientes aprovados**. A rejeição não é lembrada: na próxima conexão, o Glafira pergunta de novo. Se o cliente não informar o nome, a permissão vale somente até o fim da sessão.

<a id="what-llm-sees"></a>

## O que o assistente vê

O assistente vê:

- as conexões com **Acesso para LLM (somente metadados)** marcado: nome, ambiente, pasta;
- os brokers com hosts e portas, o controlador;
- os tópicos, as partições, as réplicas, ISR, os offsets, o número aproximado de mensagens;
- as configurações de brokers e tópicos, com senhas e outros valores secretos ocultos;
- os consumer groups: estado, protocolo, coordenador, membros com client id, host e `group.instance.id`, atribuições, offsets com commit e lag;
- as próprias propostas e a sua decisão sobre elas: aplicada (como está ou com alterações), rejeitada com o seu motivo, retirada.

O assistente não vê as mensagens dos tópicos, a string de bootstrap nem as configurações de segurança de uma conexão, nem as senhas, os certificados, as propriedades adicionais do librdkafka, o histórico de envio e os modelos.

Para investigar rebalanceamentos constantes, o assistente pode observar um grupo por até 50 segundos: quem sai e quem entra, com que frequência e de qual host. As configurações e os logs dos próprios consumers não ficam disponíveis pelo Glafira; o assistente vai pedi-los a você.

Os nomes de tópicos, grupos e client ids podem ser definidos por qualquer pessoa com acesso ao cluster. O Glafira avisa o assistente de que esses dados não são instruções para ele.

<a id="sessions-log"></a>

## Sessões, revogação e log

Enquanto um cliente está conectado, na parte inferior do painel esquerdo aparece o indicador `LLM` com o nome do cliente; ele se acende durante uma solicitação. Ao clicar nele, a seção do MCP é aberta.

- **Sessões ativas**: **Desconectar** encerra uma sessão e **Desconectar todos** encerra todas.
- **Clientes aprovados**: **Revogar** encerra as sessões do cliente, e na próxima conexão ele pede aprovação de novo.
- Desmarcar **Acesso para LLM (somente metadados)** vale imediatamente, sem reconectar.
- **Log do MCP** — cada solicitação: data/hora, cliente, ferramenta, conexão, resultado. O conteúdo das respostas não é gravado. **Mostrar arquivo** abre o arquivo `mcp-audit.jsonl`.

Para desativar o MCP por completo, desmarque **Permitir o servidor MCP**: os clientes são desconectados.

<a id="health"></a>

## Verificação do cluster, notas e propostas

O assistente pode executar a verificação integrada do cluster do Glafira (ferramenta `health_check`) e se basear nas constatações dela. Ele também pode deixar notas sobre os objetos: elas aparecem na aba **Constatações** e nas telas de tópicos e grupos, e o cluster não é alterado. Consulte [Verificação do cluster e notas](health-check.md#with-llm).

O assistente também pode propor uma alteração com a ferramenta `propose_change`: as configurações de um tópico, um novo tópico, mais partições ou uma redefinição de offsets. A proposta aparece ao lado do objeto; você mesmo a aplica em um formulário pré-preenchido ou a rejeita, e o assistente vê sua decisão em `list_proposals`. Exclusão e limpeza não podem ser propostas. Consulte [Propostas de LLM](health-check.md#proposals).

<a id="troubleshooting"></a>

## Se não funcionar

O cliente mostra uma mensagem do Glafira:

- `MCP is disabled in Glafira` — marque **Permitir o servidor MCP** e reinicie o servidor MCP no cliente;
- `MCP is disabled by your organization` — o MCP está desativado pela política da sua organização;
- `Glafira did not start the MCP server within 30 s` — inicie o Glafira manualmente e veja o status na seção **Assistentes LLM (MCP)**;
- `Invalid MCP token` — reinicie o servidor MCP no cliente: o Glafira emite um novo token a cada inicialização.

No macOS, o aplicativo é assinado e notarizado, por isso nenhuma etapa adicional é necessária antes da primeira execução.
