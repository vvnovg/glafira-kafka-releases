# Solução de problemas

Este capítulo explica os erros de conexão comuns e o que verificar em cada caso. Ele também mostra onde ficam os logs e o que incluir em uma solicitação de suporte. No macOS, o item **Solução de problemas** do menu Ajuda abre este capítulo.

Uma mensagem de erro tem um texto curto e, na maioria das telas, o código do erro entre parênteses, por exemplo `TIMEOUT`. Expanda **Detalhes** para ver a mensagem original do cliente Kafka (librdkafka). Muitas vezes, ela indica o endereço exato do broker e a causa.

<a id="cannot-connect"></a>

## Não é possível conectar

Comece por **Testar conexão** no formulário da conexão, consulte [Testar a conexão](connections.md#test). O teste para na primeira etapa que falhar, e a etapa indica onde procurar:

- **Configuração** — o problema está no próprio formulário: um endereço sem porta, um arquivo de certificado ausente ou ilegível, uma senha de certificado errada. O erro indica o campo. Para arquivos de certificado, consulte [Erros de TLS](troubleshooting.md#tls-errors).
- **DNS / TCP** — o computador não consegue alcançar os servidores bootstrap. Consulte [DNS e TCP](troubleshooting.md#dns-tcp).
- **Kafka (TLS, SASL, metadados)** — a rede funciona, mas o handshake TLS, o login SASL ou a solicitação de metadados falhou. Consulte [Erros de TLS](troubleshooting.md#tls-errors), [Erros de SASL](troubleshooting.md#sasl-errors) e [Tempos limite](troubleshooting.md#timeouts).

A causa mais comum é um protocolo que não corresponde à porta. Cada listener do broker tem sua própria porta e seu próprio protocolo. Pergunte aos administradores do Kafka qual porta corresponde a qual protocolo, consulte [Protocolos de segurança](connections.md#security).

Se o teste passar, mas a leitura ou o envio falhar, a causa costuma estar nos endereços dos brokers. O cliente recebe a lista de brokers do servidor bootstrap e depois se conecta a cada broker no endereço que o broker anuncia (`advertised.listeners`). Se esses endereços forem internos, por exemplo nomes de host do Docker ou do Kubernetes, eles não são acessíveis a partir do seu computador. Os endereços aparecem na tabela de brokers da visão geral do cluster, consulte [Visão geral do cluster](topics.md#cluster-overview). Peça aos administradores um listener acessível de fora.

<a id="dns-tcp"></a>

## DNS e TCP

A mensagem **Nenhum servidor bootstrap está acessível por TCP** significa que o aplicativo não conseguiu abrir uma conexão TCP com nenhum endereço de **Servidores bootstrap**. Cada endereço tem até 5 segundos. Os detalhes mostram o último endereço tentado e o motivo. `DNS` nos detalhes significa que não foi possível resolver o nome do host.

Verifique:

- o nome do host e a porta, sem um prefixo de protocolo como `https://`;
- se a VPN está ativada, caso o cluster esteja em uma rede privada;
- se um firewall não está bloqueando a porta;
- no caso de um broker no Docker, se a porta está publicada no host, por exemplo `-p 9092:9092`.

Para verificar a porta fora do aplicativo, execute `nc -vz <host> <port>` no macOS ou no Linux, ou `Test-NetConnection <host> -Port <port>` no PowerShell, no Windows.

A mensagem **Não foi possível conectar aos brokers** depois que a etapa TCP passou normalmente significa que um broker fechou a conexão ou ficou inacessível depois. Verifique também os endereços dos brokers, consulte [Não é possível conectar](troubleshooting.md#cannot-connect).

<a id="tls-errors"></a>

## Erros de TLS

Erros na etapa **Kafka (TLS, SASL, metadados)** durante o handshake TLS:

- **O broker aceita conexões sem TLS neste endereço — verifique a porta ou escolha um protocolo sem SSL**.
- **O certificado do broker não foi assinado por uma CA confiável — verifique “CA confiável”**. Em **Certificado da CA confiável**, indique o arquivo da CA que assinou os certificados dos brokers. Os administradores podem fornecer esse arquivo. Com o campo vazio, apenas os certificados raiz do sistema são considerados confiáveis.
- **O nome do host do broker não corresponde ao certificado dele**. Conecte-se pelo nome para o qual o certificado foi emitido, e não por um endereço IP ou um alias. Somente em ambientes de teste, você pode desativar a opção **Verificar nome do host**.
- **O broker rejeitou o certificado do cliente**. O broker exige um certificado de cliente ou não confia no que você usa. Verifique se a opção **Certificado do cliente (mTLS)** está ativada e se o certificado foi emitido para este cluster. Consulte [Certificado do cliente (mTLS)](connections.md#mtls).
- **Erro na conexão TLS** — outro problema de TLS. Expanda **Detalhes** para ver a causa.

Se você escolheu um protocolo sem SSL e mesmo assim vê um erro de TLS, a porta provavelmente é de um listener TLS. Nesse caso, os detalhes dizem `connecting to a SSL listener?`. Escolha `SSL` ou `SASL_SSL`, ou use a porta de um listener sem TLS.

Os problemas com arquivos de certificado são informados na etapa **Configuração**, antes de o aplicativo se conectar. A mensagem começa com o nome do campo: CA confiável, certificado do cliente ou chave do cliente. Casos comuns:

- O caminho não é absoluto ou o arquivo não pode ser lido. Informe o caminho completo ou escolha o arquivo com **Procurar…**.
- **Chave do cliente: a chave está criptografada — informe a senha da chave**, **Chave do cliente: senha da chave incorreta**, **Certificado do cliente: senha do repositório PKCS\#12 incorreta** ou **Senha do repositório JKS incorreta (ou o arquivo está corrompido)** — digite a senha correta no formulário.
- **A chave do cliente não corresponde ao certificado do cliente** — o arquivo da chave pertence a outro certificado.
- O certificado do cliente expirou ou ainda não é válido — a mensagem informa a data. Peça um novo certificado.
- O repositório PKCS#12 usa um algoritmo legado ou está no formato JCEKS. Converta o arquivo como mostrado em [Formatos de certificado](connections.md#certificate-formats).

<a id="sasl-errors"></a>

## Erros de SASL

A mensagem **Falha na autenticação** significa que o broker não aceitou o login SASL. Verifique:

- o usuário e a senha. Quando você edita uma conexão salva, um campo de senha vazio mantém a senha salva. Digite a senha de novo se não tiver certeza de que ela está correta;
- o campo **Mecanismo SASL**: ele precisa indicar o mecanismo para o qual seu usuário foi criado. As credenciais SCRAM são criadas separadamente para `SCRAM-SHA-256` e `SCRAM-SHA-512`, e os usuários `PLAIN` são configurados também à parte;
- o protocolo: `SASL_SSL` e `SASL_PLAINTEXT` normalmente usam portas diferentes.

**A senha SASL da conexão não foi definida** — não há senha salva para esta conexão, por exemplo depois de uma importação. Abra a conexão, digite a senha e salve. Consulte [Senhas](connections.md#passwords).

**Não foi possível acessar o cofre de segredos do sistema** — o aplicativo não consegue ler a senha do cofre de segredos do sistema. No macOS, permita o acesso quando as Chaves pedirem. No Linux, verifique se o GNOME Keyring, o KWallet ou outro provedor do Secret Service está em execução na sua sessão.

**Sem permissão para executar a operação** é diferente: o login foi bem-sucedido, mas o usuário do Kafka não tem permissões (ACLs) para essa operação, por exemplo ler um tópico ou um grupo. Peça as permissões aos administradores. Observe que o Kafka não informa erro para os tópicos que você não tem permissão para descrever: eles simplesmente não aparecem na lista de tópicos.

<a id="timeouts"></a>

## Tempos limite

A mensagem **A operação excedeu o tempo limite** significa que o broker não respondeu a tempo. As causas típicas são uma rede ou VPN lenta, um cluster sobrecarregado ou endereços de brokers inacessíveis, consulte [Não é possível conectar](troubleshooting.md#cannot-connect).

Um problema de TLS ou SASL pode parecer um tempo limite esgotado. O aplicativo tenta encontrar a causa real e a mostra no lugar, mas isso nem sempre é possível. Expanda **Detalhes**: mensagens como `SSL handshake failed` ou `SASL authentication error` indicam a causa.

Se a rede for apenas lenta, aumente o valor de **Tempo limite da solicitação, ms** no formulário da conexão. O padrão é `30000` ms. Esse valor vale para as solicitações aos brokers, para o envio de mensagens e para a leitura. O teste de conexão usa limites próprios (5 segundos para cada endereço TCP e 10 segundos para a etapa do Kafka), então um valor maior não altera o teste.

<a id="read-only-blocked"></a>

## Uma ação está indisponível ou é recusada

Se um botão estiver oculto ou desativado, ou se uma ação for recusada, verifique estas causas:

- A conexão é somente leitura. Um ícone de cadeado ao lado do nome dela a identifica. Os botões que alteram o cluster ficam ocultos ou desativados. Se uma solicitação desse tipo ainda assim chegar ao núcleo do aplicativo, por exemplo depois que o modo foi ativado com uma tela aberta, o erro informa que a conexão é somente leitura. Para fazer alterações, abra a conexão, desative a opção **Somente leitura** e salve. Consulte [Somente leitura](connections.md#read-only).
- A conexão é prod. As alterações pedem que você digite o nome da conexão. Até que o nome corresponda, a ação não é executada. Consulte [Ambientes e prod](connections.md#environments).
- O consumer group tem membros ativos. Só é possível redefinir os offsets e excluir um grupo quando ele não tem membros. Consulte [Redefinir offsets](groups.md#reset-offsets).
- A configuração é gerenciada pela sua organização. Uma política pode definir o idioma da interface e a verificação de atualizações, e esses itens não podem ser alterados no aplicativo. Consulte [Configurações definidas pela sua organização](getting-started.md#managed-settings).
- O Kafka recusou a operação com a mensagem **Sem permissão para executar a operação**. É uma questão de permissões no cluster, não de configurações do Glafira. Consulte [Erros de SASL](troubleshooting.md#sasl-errors).

<a id="logs"></a>

## Logs

O aplicativo grava um log do próprio funcionamento e dos erros do cliente Kafka. A pasta exata aparece na tela **Sobre**, no item **Logs:** Por padrão:

- macOS: `~/Library/Logs/io.github.vvnovg.glafira.kafka`
- Windows: `%LOCALAPPDATA%\io.github.vvnovg.glafira.kafka\logs`
- Linux: `~/.local/share/io.github.vvnovg.glafira.kafka/logs`

O arquivo atual é `app.log`. Quando ele chega a 5 MB, é renomeado para `app.1.log`, e os arquivos mais antigos passam para `app.2.log` e assim por diante. O aplicativo mantém até cinco arquivos e exclui o mais antigo.

As senhas não são gravadas nos logs. Os endereços dos brokers e os textos dos erros do Kafka são gravados. Não é preciso reunir os arquivos manualmente: o arquivo de diagnóstico os inclui, consulte [Arquivo de diagnóstico](updates.md#diagnostics).

<a id="support"></a>

## Falar com o suporte

Antes de falar com o suporte, execute **Testar conexão** e anote qual etapa falha. Depois, envie a solicitação a quem dá suporte ao Glafira para você, por exemplo os administradores. Inclua:

- o arquivo de diagnóstico, consulte [Arquivo de diagnóstico](updates.md#diagnostics). Ele contém a versão, o build, os logs e os perfis de conexão sem senhas;
- o que você fez, o que esperava e o que aconteceu, passo a passo;
- o texto completo do erro, junto com o conteúdo de **Detalhes**. É possível selecionar e copiar esse texto;
- uma captura de tela, se o problema estiver na interface;
- o horário em que o erro ocorreu, para encontrá-lo nos logs.

Se não for possível criar o arquivo de diagnóstico, informe os valores de **Versão** e **Build** da tela **Sobre** e anexe os arquivos de log da pasta de logs.
