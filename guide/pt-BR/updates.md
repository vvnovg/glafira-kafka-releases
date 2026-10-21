# Atualizações e diagnóstico

Este capítulo trata de como atualizar o aplicativo, da tela **Sobre**, do arquivo de diagnóstico e de como apoiar o projeto.

<a id="updates"></a>

## Atualizações

Ao iniciar, o aplicativo verifica uma vez se há uma nova versão disponível. Se houver, um aviso acima das abas informa a versão. Clicar em **Detalhes** abre a tela **Sobre**. Se a verificação falhar, por exemplo sem acesso à internet, nenhuma mensagem aparece.

Para verificar manualmente:

1. Abra **Sobre** na parte inferior do painel esquerdo.
2. Na seção **Atualizações**, clique em **Verificar atualizações**.
3. Se houver uma nova versão, a tela mostra o número dela e as notas da versão. Clique em **Instalar e reiniciar**.

O aplicativo baixa a atualização, verifica a assinatura, instala e reinicia. Um download interrompido por um problema de rede é repetido automaticamente, até três tentativas no total. Se as três tentativas falharem, o aplicativo mostra a mensagem **O download da atualização foi interrompido — verifique a rede e tente novamente**. Se a verificação da assinatura falhar, o aplicativo mostra a mensagem **A assinatura da atualização não passou na verificação — a atualização não foi instalada**.

A instalação pode pedir uma senha de administrador, porque o aplicativo é instalado para todos os usuários do computador: pelo instalador `.pkg` no macOS, pelo instalador `.msi` no Windows ou por um pacote `.deb` ou `.rpm` no Linux. Usuários sem direitos de administrador não podem atualizar uma instalação desse tipo.

Se houver mensagens sendo lidas, uma sessão de consumer em execução ou um lote sendo enviado, o aplicativo pergunta antes e informa quantas sessões estão ativas. Clique em **Parar as sessões e atualizar** para interrompê-las e continuar, ou em **Cancelar**. As conexões, as configurações e os demais dados são mantidos depois de uma atualização.

As atualizações são baixadas da página de releases no GitHub. Se a verificação falhar com a mensagem **Não foi possível verificar se há atualizações**, verifique a rede ou o proxy. A mensagem **As atualizações não estão configuradas neste build** significa que este build foi feito sem a chave do atualizador, por exemplo um build de desenvolvimento ou de teste. Nesse caso, baixe as novas versões na página de releases: https://github.com/vvnovg/glafira-kafka-releases/releases

Uma organização pode desativar a verificação de atualizações por meio de uma política ou direcioná-la para um espelho interno. Quando a verificação está desativada, a seção **Atualizações** mostra **As atualizações são gerenciadas pela sua organização** depois que você clica em **Verificar atualizações**. Assim como **As atualizações não estão configuradas neste build**, essa mensagem só aparece depois do clique; a verificação ao iniciar não mostra nada nesses casos.

<a id="about"></a>

## Sobre

Abra **Sobre** na parte inferior do painel esquerdo. No macOS, ele também está no menu Ajuda, como **Sobre o Glafira**.

A tela mostra uma descrição curta do aplicativo e:

- **Versão** — o número da versão, e **Build** — o identificador do código-fonte a partir do qual ele foi compilado. Inclua os dois em uma solicitação de suporte.
- **Sistema** — o sistema operacional e a arquitetura do processador.
- **Licença** — Apache 2.0; o link abre o texto da licença no navegador.
- **Política da organização** — **definida** ou **nenhuma**. Os erros na política são listados abaixo.
- **Dados:** — a pasta com os perfis de conexão, os modelos, o histórico e as configurações.
- **Logs:** — a pasta com os arquivos de log. Consulte [Logs](troubleshooting.md#logs).

As seções **Atualizações** e **Diagnóstico** também ficam nessa tela.

<a id="diagnostics"></a>

## Arquivo de diagnóstico

O arquivo de diagnóstico reúne o que o suporte precisa para entender um problema. Para criá-lo:

1. Abra **Sobre**.
2. Na seção **Diagnóstico**, clique em **Coletar diagnóstico**. No macOS, também é possível escolher **Coletar diagnóstico…** no menu Ajuda.
3. Escolha onde salvar o arquivo. O nome sugerido é `diagnostics-<date>.zip`.

Quando o arquivo é salvo, a tela mostra o caminho dele. O arquivo ZIP contém:

- `info.json` — a versão e o build do aplicativo, o sistema operacional, os caminhos das pastas de dados e de logs, e o status da política;
- `connections.json` — os perfis de conexão: nomes, ambientes, servidores bootstrap, protocolos, usuários SASL, caminhos dos arquivos de certificado, propriedades adicionais e tempos limite;
- `logs/` — os arquivos de log do aplicativo.

O arquivo não contém senhas: elas ficam no cofre de segredos do sistema, e as propriedades adicionais com senhas e segredos são recusadas ao salvar uma conexão. Os arquivos de certificado também não são incluídos. Os nomes de host, os usuários e os caminhos de arquivo ficam em texto puro, e os logs contêm endereços de brokers das mensagens de erro do Kafka. Revise o arquivo antes de enviá-lo para fora da sua empresa.

O aplicativo não envia nada por conta própria. Anexe você mesmo o arquivo à sua solicitação de suporte, consulte [Falar com o suporte](troubleshooting.md#support).

<a id="support"></a>

## Apoiar o projeto

O Glafira é gratuito e de código aberto sob a Apache License 2.0. As doações o mantêm vivo. **Apoiar o projeto** na parte inferior do painel esquerdo, na tela **Sobre** e, no macOS, no menu Ajuda abre a página de apoio no navegador: https://github.com/vvnovg/glafira-kafka-releases/blob/main/SUPPORT.md
