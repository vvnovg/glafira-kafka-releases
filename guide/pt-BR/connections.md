# Conexões

Uma conexão é um perfil salvo de um cluster Kafka: endereços, configurações de segurança e senhas. As conexões ficam listadas no painel **Conexões**, à esquerda. É possível organizá-las em pastas e levá-las para outro computador com a exportação e a importação.

<a id="create"></a>

## Criar uma conexão

1. Clique em `+` ( **Nova conexão** ) no cabeçalho do painel **Conexões**. Para criar a conexão diretamente em uma pasta, abra o menu da pasta e escolha **Nova conexão aqui**.
2. Em **Nome**, digite um nome. Ele aparece no painel e nos títulos das abas.
3. Em **Servidores bootstrap**, informe um ou mais endereços no formato `host:port`, separados por vírgulas, por exemplo `kafka1:9092,kafka2:9092`. Basta um broker acessível: o cliente descobre os demais sozinho.
4. Escolha o protocolo em **Protocolo** e preencha os campos que ele adiciona. Consulte [Protocolos de segurança](connections.md#security).
5. Se quiser, escolha uma pasta em **Pasta**, um ambiente em **Ambiente** e, em **Cor**, a cor do ponto ao lado do nome.
6. Clique em **Testar conexão** e depois em **Salvar**.

Depois de salvar, o formulário fecha e a visão geral do cluster é aberta.

Mais dois campos servem para casos especiais. **Tempo limite da solicitação, ms** é o tempo de espera pela resposta de um broker. O padrão é `30000` ms, e o mínimo, `1000`. Aumente esse valor em redes lentas e VPNs. **Propriedades adicionais do librdkafka** aceita qualquer configuração do librdkafka como `key=value`, uma por linha. Linhas que começam com `#` são ignoradas. Senhas e chaves não são permitidas ali: propriedades cujo nome contém `password` ou `secret`, `sasl.jaas.config`, `ssl.key.pem` e `ssl.keystore.key` são rejeitadas ao salvar.

Para alterar uma conexão depois, clique em `✎` ( **Editar conexão** ) ao lado do nome dela. Para excluí-la, abra o formulário, clique em **Excluir** e confirme com **Sim, excluir**.

<a id="test"></a>

## Testar a conexão

**Testar conexão** verifica as configurações do formulário etapa por etapa e para na primeira etapa que falhar:

1. **Configuração** — os endereços, os arquivos de certificado, as senhas e o usuário SASL.
2. **DNS / TCP** — os nomes de host são resolvidos e uma conexão TCP com pelo menos um servidor bootstrap é aberta.
3. **Kafka (TLS, SASL, metadados)** — o handshake TLS e o login SASL são bem-sucedidos, e o broker retorna os metadados. Em caso de sucesso, a etapa mostra o número de brokers e tópicos.

O teste usa os valores que estão no formulário no momento, inclusive senhas digitadas e ainda não salvas. Nada é salvo nem alterado no cluster. Se uma etapa falhar, o texto do erro e os detalhes aparecem abaixo dela. Os erros comuns são explicados em [Não é possível conectar](troubleshooting.md#cannot-connect).

<a id="security"></a>

## Protocolos de segurança

O protocolo escolhido em **Protocolo** precisa corresponder ao listener do broker ao qual você se conecta. Os administradores do Kafka sabem qual usar.

- `PLAINTEXT` — sem criptografia e sem login. Típico de clusters locais e de teste.
- `SSL` — o tráfego é criptografado com TLS. Com um certificado de cliente, é mTLS.
- `SASL_PLAINTEXT` — login com usuário e senha, sem criptografia.
- `SASL_SSL` — login com usuário e senha sobre TLS.

Quando você muda o protocolo, o formulário mostra apenas os campos de que ele precisa.

<a id="sasl"></a>

## SASL

Para `SASL_PLAINTEXT` e `SASL_SSL`, preencha três campos:

- **Mecanismo SASL** — `PLAIN`, `SCRAM-SHA-256` ou `SCRAM-SHA-512`. O cluster define qual funciona.
- **Usuário**.
- **Senha**. Ela é salva no cofre de segredos do sistema, não no perfil. Consulte [Senhas](connections.md#passwords).

O SCRAM nunca envia a própria senha. O `PLAIN` a envia como está, então com `SASL_PLAINTEXT` a senha trafega em texto puro. Sempre que possível, use `SASL_SSL` para clusters fora do seu computador.

<a id="tls"></a>

## SSL e a CA confiável

Para `SSL` e `SASL_SSL`, o formulário adiciona as configurações de TLS.

**Certificado da CA confiável** é o certificado da autoridade (CA) que assinou os certificados dos brokers. Deixe o campo vazio se o sistema operacional já confiar nessa CA: nesse caso, são usados os certificados raiz do sistema. Caso contrário, informe o caminho completo do arquivo ou clique em **Procurar…**. Depois que você escolhe um arquivo, aparece o campo **Formato da CA**. Para PKCS#12 e JKS, há também o campo **Senha do truststore**.

A opção **Verificar nome do host** vem ativada por padrão. Ela verifica se o certificado do broker foi emitido para o nome de host ao qual você se conecta. Desative-a apenas em ambientes de teste, por exemplo quando os brokers usam certificados autoassinados com outros nomes.

<a id="mtls"></a>

## Certificado do cliente (mTLS)

Se os brokers autenticam os clientes por certificado, ative a opção **Certificado do cliente (mTLS)**. Os administradores do Kafka emitem o certificado e a chave. Em seguida, preencha:

- **Certificado do cliente** — o caminho do arquivo do certificado.
- **Formato do certificado do cliente** — PEM, PKCS#12 ou JKS.
- Para PEM: **Chave do cliente** — o caminho do arquivo da chave privada, e **Senha da chave (se estiver criptografada)**, se a chave estiver criptografada.
- Para PKCS#12: **Senha do keystore**.
- Para JKS: **Senha do keystore**, **Senha da chave (se for diferente)**, se a chave tiver senha própria, e **Alias da chave**, se o repositório tiver várias chaves.

Os caminhos precisam ser absolutos. Os arquivos são lidos a cada conexão do aplicativo, então, depois que o certificado é renovado, basta reconectar. O aplicativo verifica se a chave corresponde ao certificado e se o certificado não expirou.

Um caso comum é um cluster no Kubernetes, em que o secret do cliente contém `tls.crt`, `tls.key` e `ca.crt`. Salve-os em arquivos, escolha o protocolo `SSL`, indique `ca.crt` em **Certificado da CA confiável** e use `tls.crt` e `tls.key` como certificado e chave do cliente em PEM, com senhas vazias.

<a id="certificate-formats"></a>

## Formatos de certificado

O Glafira lê três formatos. O formato é escolhido automaticamente pela extensão do arquivo; confira-o no campo de formato e altere-o se necessário.

- PEM — um arquivo de texto que começa com `-----BEGIN`. Extensões `.pem`, `.crt`, `.cer`, `.key`. No certificado do cliente, a chave fica em um arquivo separado. A CA confiável também pode ser um certificado DER binário.
- PKCS#12 — um repositório binário com certificado e chave. Extensões `.p12`, `.pfx`.
- JKS — um keystore Java. Extensões `.jks`, `.keystore`, `.truststore`.

Dois tipos de arquivo precisam ser convertidos antes:

- Repositórios JCEKS não são suportados. Converta-os para PKCS#12:

```bash
keytool -importkeystore -srcstoretype JCEKS -srckeystore client.jceks -deststoretype PKCS12 -destkeystore client.p12
```

- Repositórios PKCS#12 criptografados com algoritmos legados (RC2, 3DES) podem não abrir. Se o aplicativo informar um algoritmo legado, recrie o arquivo com AES:

```bash
openssl pkcs12 -export -keypbe AES-256-CBC -certpbe AES-256-CBC -in tls.crt -inkey tls.key -out client.p12
```

<a id="passwords"></a>

## Senhas

A senha SASL e as senhas de chaves e repositórios são salvas no cofre de segredos do sistema: Chaves no macOS, Gerenciador de Credenciais no Windows, Secret Service no Linux. Elas nunca são gravadas no perfil da conexão, nos arquivos de exportação nem no arquivo de diagnóstico.

Quando você edita uma conexão salva, os campos de senha ficam vazios e mostram **deixe vazio para não alterar**. Deixe o campo vazio para manter a senha salva ou digite uma nova para substituí-la. Para excluir uma senha de certificado salva, marque **Remover senha salva** ao lado do campo. Ao salvar, as senhas que a conexão não usa mais são excluídas, por exemplo depois de mudar o protocolo para `PLAINTEXT`.

Se o aplicativo não conseguir acessar o cofre de segredos, um erro aparece ao salvar. No Linux, verifique se o GNOME Keyring, o KWallet ou outro provedor do Secret Service está em execução na sua sessão.

<a id="environments"></a>

## Ambientes e prod

O campo **Ambiente** marca a conexão: `local`, `dev`, `test`, `stage` ou `prod`. O ambiente aparece como um selo colorido ao lado do nome no painel, para que você sempre saiba com qual cluster está trabalhando.

`prod` adiciona proteção. Qualquer alteração em uma conexão prod, como criar um tópico, enviar uma mensagem, redefinir offsets ou iniciar uma sessão de consumer, pede que você digite o nome da conexão. Excluir e limpar pedem o nome do objeto em qualquer conexão (excluir vários tópicos pede a quantidade deles), e em prod você digita os dois. Essas confirmações aparecem dentro da janela do aplicativo.

Uma nova conexão prod é criada com a opção **Somente leitura** ativada. Desmarque a caixa se precisar fazer alterações.

<a id="read-only"></a>

## Somente leitura

A opção **Somente leitura** bloqueia tudo o que altera o cluster: criar, alterar, limpar e excluir tópicos, enviar mensagens, redefinir offsets, excluir grupos e iniciar sessões de consumer. Os botões dessas ações ficam ocultos ou desativados, e o núcleo do aplicativo também rejeita essas operações. A navegação e a leitura continuam disponíveis. Um ícone de cadeado ao lado do nome no painel marca as conexões somente leitura.

Use esse modo quando precisar apenas consultar os dados, principalmente em clusters compartilhados. O modo somente leitura é uma proteção dentro do aplicativo: ele não substitui os direitos de acesso (ACLs) no Kafka.

<a id="folders"></a>

## Pastas

As pastas ajudam a organizar as conexões, por exemplo por projeto ou por ambiente. As pastas podem ser aninhadas.

- Para criar uma pasta no nível superior, clique em `+📁` ( **Nova pasta** ) no cabeçalho do painel. Para criar uma subpasta, abra o menu da pasta e escolha **Nova pasta**. Digite o nome e pressione `Enter`, ou pressione `Esc` para cancelar.
- Para abrir o menu da pasta, clique em `⋯` ( **Mais ações** ) na linha da pasta ou clique nela com o botão direito. O menu tem **Renomear**, **Excluir** e os comandos de exportação e importação.
- Só é possível excluir uma pasta vazia. Antes, mova ou exclua as conexões e subpastas dela.
- Os nomes das pastas precisam ser únicos em um mesmo nível e ter no máximo 100 caracteres.

Para mover uma conexão, arraste-a para uma pasta ou para outra conexão, para colocá-la antes dessa. Para movê-la para o nível superior, solte-a no cabeçalho do painel ou no espaço vazio abaixo da lista. Também é possível escolher a pasta em **Pasta**, no formulário da conexão; **(raiz)** significa o nível superior. As pastas são arrastadas da mesma forma. Solte uma pasta na borda superior de outra para colocá-la antes dessa. Uma pasta não pode ser movida para dentro de uma subpasta dela mesma.

Clique no nome de uma pasta para recolhê-la ou expandi-la. O aplicativo lembra quais pastas estão recolhidas. Use o campo **Filtrar conexões**, acima da árvore, para encontrar conexões e pastas pelo nome. As pastas correspondentes aparecem com todo o conteúdo.

<a id="export"></a>

## Exportação

A exportação salva conexões e pastas em um arquivo, por exemplo para compartilhá-las com colegas ou levá-las para outro computador.

1. Para exportar tudo, abra o menu `⋯` do cabeçalho do painel e escolha **Exportar tudo…**. Para exportar uma pasta com as subpastas dela, escolha **Exportar pasta…** no menu da pasta.
2. Escolha onde salvar o arquivo. O nome sugerido é `glafira-connections-<date>.glafira-connections.json` ou `<folder>.glafira-connections.json`.

Quando o arquivo é salvo, o painel mostra quantas conexões e pastas ele contém.

O arquivo traz as configurações das conexões e a estrutura de pastas. Ele não contém senhas nem propriedades secretas. Os arquivos de certificado também não são incluídos: o arquivo guarda apenas os caminhos deles. Hosts, usuários e caminhos ficam em texto puro, então compartilhe o arquivo apenas com quem pode vê-los.

<a id="import"></a>

## Importação

1. Abra o menu `⋯` do cabeçalho do painel e escolha **Importar…**. Para importar diretamente para uma pasta, escolha **Importar aqui…** no menu dela.
2. Escolha um arquivo `.glafira-connections.json`. A caixa de diálogo **Importar conexões** é aberta.
3. Em **Importar para**, escolha para onde vão as novas pastas e conexões. **(raiz)** é o nível superior.
4. Mantenha a opção **Atualizar conexões existentes** ativada para atualizar com os dados do arquivo as conexões que já existem. Desative-a para mantê-las como estão.
5. Confira a pré-visualização e clique em **Importar**, ou em **Cancelar** para fechar a caixa de diálogo sem alterações.

A pré-visualização é atualizada a cada alteração na caixa de diálogo. Ela mostra os totais e listas expansíveis: **Novos**, **Serão atualizados**, **Sem alterações** e **Ignorados**, com o motivo de cada conexão ignorada.

Como o aplicativo associa o arquivo às suas conexões:

- As conexões são associadas por um ID interno, não pelo nome. Uma conexão é considerada existente se o arquivo contém essa mesma conexão: por exemplo, ela foi exportada deste computador ou importada antes da mesma origem.
- Uma conexão atualizada recebe as configurações do arquivo. As suas senhas e a posição da conexão na árvore são mantidas.
- As pastas existentes não são renomeadas nem movidas. As novas conexões do arquivo vão para elas mesmo assim.
- Se uma nova conexão tiver o mesmo nome de outra na mesma pasta, um número é acrescentado, por exemplo `kafka-dev (2)`.
- As conexões que esta versão do Glafira não suporta, ou que contêm segredos nas propriedades adicionais, são ignoradas.

A pré-visualização também avisa quando os arquivos de certificado não são encontrados neste computador. As senhas não estão no arquivo, então, depois da importação, o painel informa quantas conexões precisam delas. Abra cada uma dessas conexões, digite as senhas e salve.
