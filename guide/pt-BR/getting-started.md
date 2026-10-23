# Primeiros passos

O Glafira for Apache Kafka é um cliente desktop para clusters Kafka. Com ele, você navega por tópicos e consumer groups, lê e envia mensagens e redefine offsets. Este capítulo trata da instalação, da primeira execução, de um Kafka local para praticar e do idioma da interface.

<a id="install-macos"></a>

## Instalação no macOS

O Glafira funciona no macOS 11 ou posterior. Baixe o instalador na página de releases: https://github.com/vvnovg/glafira-kafka-releases/releases

Escolha o arquivo adequado ao seu Mac: `aarch64` para Apple silicon (M1 e posteriores), `x64` para Intel.

1. Abra o arquivo `.dmg`.
2. Arraste o Glafira for Apache Kafka para a pasta Aplicativos.
3. Inicie o aplicativo pela pasta Aplicativos ou pelo Launchpad.

Também é possível usar o instalador `.pkg`. Ele coloca o aplicativo em `/Applications` para todos os usuários.

Os builds são assinados pela Apple e notarizados, por isso o aplicativo abre imediatamente, sem avisos de segurança.

Se mesmo assim o macOS informar que o aplicativo está danificado ou não pode ser verificado, o download está corrompido: baixe o `.dmg` ou o `.pkg` novamente na página de releases.

<a id="install-windows"></a>

## Instalação no Windows

Baixe o instalador `.msi` (`x64`) na página de releases: https://github.com/vvnovg/glafira-kafka-releases/releases

1. Execute o arquivo `.msi`.
2. Confirme a instalação na janela do Windows. O aplicativo é instalado para todos os usuários do computador, por isso são necessários direitos de administrador.
3. Inicie o Glafira for Apache Kafka pelo menu Iniciar.

Os builds beta ainda não são assinados. Se o SmartScreen bloquear o instalador, clique em “Mais informações” e depois em “Executar assim mesmo”. Também é possível desbloquear o arquivo no PowerShell antes de executá-lo:

```powershell
Unblock-File -Path "$env:USERPROFILE\Downloads\Glafira_for_Apache_Kafka_*_x64_en-US.msi"
```

<a id="install-linux"></a>

## Instalação no Linux

Há builds para `x86_64` em três formatos. Baixe o adequado à sua distribuição na página de releases: https://github.com/vvnovg/glafira-kafka-releases/releases

- Debian, Ubuntu e derivados: `sudo apt install ./<file>.deb`
- Fedora, RHEL e derivados: `sudo dnf install ./<file>.rpm`
- Qualquer distribuição: torne o AppImage executável com `chmod +x <file>.AppImage` e execute-o.

Depois de instalar um pacote, inicie o aplicativo pelo menu de aplicativos ou execute `glafira-kafka` em um terminal.

O Glafira guarda as senhas no Secret Service (por exemplo, GNOME Keyring ou KWallet). Verifique se um deles está em execução na sua sessão. Caso contrário, o aplicativo não consegue salvar as senhas das conexões.

<a id="first-run"></a>

## Primeira execução

A janela tem duas partes. À esquerda fica o painel **Conexões**. À direita ficam as abas: cada tela aberta, como uma lista de tópicos ou o Producer, ganha sua própria aba.

Enquanto não há conexões, o painel mostra uma dica curta, e a área de abas sugere adicionar uma conexão. Para começar:

1. Clique em `+` ( **Nova conexão** ) no cabeçalho do painel **Conexões**.
2. Preencha os campos **Nome** e **Servidores bootstrap**, por exemplo `localhost:9092`.
3. Clique em **Testar conexão** para verificar as configurações e depois em **Salvar**.

Depois de salvar, abre-se a visão geral do cluster. Abaixo de cada conexão no painel há links para **Visão geral do cluster**, **Tópicos**, **Consumer groups**, **Producer** e **Sessão de consumer**. Todas as configurações de conexão estão descritas em [Conexões](connections.md#create).

Na parte inferior do painel ficam os links **Configurações**, **Apoiar o projeto**, **Ajuda** e **Sobre**. O sinal `?` ao lado de um campo mostra uma dica curta sobre ele. Pressione `F1` em qualquer lugar da janela para abrir o capítulo do guia sobre a tela atual. No macOS, o guia também está no menu Ajuda.

Se você usava o Kafka Explorer (o nome anterior do aplicativo), o Glafira copia na primeira execução as conexões, os modelos e o histórico de envios dele. As senhas também são copiadas do cofre de segredos do sistema. Nada do que já existe é sobrescrito, e os dados antigos não são excluídos. O macOS pode pedir uma vez acesso aos itens antigos das Chaves.

<a id="local-cluster"></a>

## Kafka local para praticar

Para experimentar o aplicativo sem um cluster compartilhado, inicie o Kafka no seu computador com o Docker. Um broker é suficiente:

```bash
docker run -d --name kafka -p 9092:9092 apache/kafka:3.9.0
```

Em seguida, crie uma conexão com o servidor bootstrap `localhost:9092`, o protocolo `PLAINTEXT` e o ambiente `local`. Quando não precisar mais do broker, remova-o com `docker rm -f kafka`.

A dica no painel **Conexões** vazio menciona `docker compose -f dev/docker-compose.yml up -d`. Esse arquivo faz parte do código-fonte do Glafira. Se você tiver o código-fonte, esse comando inicia um broker com três listeners:

- `localhost:9092` — `PLAINTEXT`.
- `localhost:9094` — `SASL_PLAINTEXT` com `PLAIN` (usuários `tester` / `tester-secret` e `admin` / `admin-secret`) e SCRAM (`scram-user` / `scram-secret` para `SCRAM-SHA-256` e `SCRAM-SHA-512`).
- `localhost:9095` — `SSL` com certificados de cliente (mTLS). Os certificados de teste ficam em `dev/certs/out`.

Esses usuários e certificados servem apenas para testes locais.

<a id="language"></a>

## Idioma da interface

A interface está disponível em inglês, russo, alemão, francês, espanhol, italiano, polonês, português (Brasil), chinês (simplificado e tradicional), japonês e coreano. Todos os idiomas são gratuitos.

Para mudar o idioma:

1. Clique em **Configurações** na parte inferior do painel esquerdo.
2. Escolha um idioma em **Idioma da interface**.

A mudança vale na hora, inclusive para este guia e para o menu do macOS. A opção **Idioma do sistema** (o padrão) segue o idioma do sistema operacional. Se esse idioma não estiver na lista, a interface fica em inglês.

<a id="managed-settings"></a>

## Configurações definidas pela sua organização

Em uma empresa, os administradores podem definir algumas opções para todos os usuários por meio de uma política. Essas opções não podem ser alteradas no aplicativo. Uma política pode:

- definir o idioma da interface — a lista de idiomas em **Configurações** fica indisponível, com um aviso de que o idioma foi definido pela sua organização;
- desativar a verificação de atualizações ou direcioná-la para um espelho interno.

Para saber se uma política está em vigor, abra **Sobre** e veja o item **Política da organização**. Se a política tiver um erro, ele aparece ali. Para dúvidas sobre a política, fale com os administradores. As atualizações estão descritas em [Atualizações e diagnóstico](updates.md).
