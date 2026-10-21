# Verificação do cluster e notas

O Glafira verifica um cluster com um conjunto de regras integradas e mostra os problemas encontrados ao lado dos objetos: na árvore de conexões, nas listas de tópicos e grupos e nas telas de um tópico ou grupo. Um assistente LLM pode complementar essas constatações com as próprias notas. A verificação apenas lê o cluster e não muda nada nele.

<a id="run"></a>

## Executar uma verificação

Abra **Visão geral do cluster** ou **Constatações** de uma conexão e clique em **Verificar cluster**. Uma verificação leva até 45 segundos: o Glafira lê os brokers, as partições, as configurações de tópicos (até 2000 tópicos) e os consumer groups (até 500). Se houver mais tópicos ou grupos, se algo não puder ser lido ou se os 45 segundos se esgotarem, o resultado é marcado como **verificado parcialmente**, e a constatação **Verificação incompleta** mostra o que não foi lido.

O Glafira não consulta o cluster por conta própria: uma verificação só é executada quando você clica no botão ou quando um assistente LLM a solicita. O resultado fica guardado até a próxima verificação.

<a id="rules"></a>

## O que é verificado

- Brokers e partições: sem controlador, partições indisponíveis e sub-replicadas, cluster com um único broker, desequilíbrio de leaders, leaders fora da réplica preferida.
- Durabilidade dos tópicos: fator de replicação 1, `min.insync.replicas` maior que o fator de replicação, `min.insync.replicas` igual ao fator de replicação, `min.insync.replicas` igual a 1, eleição de leader não limpa.
- Configuração dos brokers: criação automática de tópicos, fator de replicação padrão 1, `min.insync.replicas` padrão 1, configurações importantes que diferem entre os brokers.
- Consumer groups: grupos vazios com lag, grupos em rebalanceamento, membros sem partições, lag que cresceu desde a verificação anterior.

<a id="severity"></a>

## Gravidade e ambiente

Cada constatação tem uma gravidade: **Crítico**, **Aviso** ou **Informação**. Ela depende do ambiente da conexão. Por exemplo, fator de replicação 1 é crítico em prod, um aviso em stage e não é mostrado em dev, test e local. O ambiente é definido no formulário da conexão.

<a id="tab"></a>

## A aba “Constatações”

Na aba **Constatações**, as constatações são agrupadas por regra; cada uma explica o problema e o que fazer. Filtre por **Gravidade**, **Origem** e **Objeto** ou pesquise pelo nome. O nome do objeto abre a tela dele.

Um selo com um número na árvore de conexões e nas listas de tópicos e grupos mostra quantas constatações e notas de LLM abertas com gravidade **Crítico** ou **Aviso** dizem respeito ao objeto. Aponte para o selo para ver os títulos delas.

<a id="hide"></a>

## Ocultar constatações

Se uma constatação é esperada para um objeto específico, por exemplo um tópico de teste com fator de replicação 1 em prod, clique em **Ocultar para este objeto**. Uma constatação oculta não conta nos selos e continua oculta nas verificações seguintes. Para ver as ocultas, marque “Mostrar ocultas”; ali você encontra **Restaurar**.

<a id="notes"></a>

## Notas de LLM

Um assistente LLM conectado por MCP (consulte [Assistentes LLM (MCP)](llm-mcp.md#how-it-works)) pode deixar uma nota sobre o cluster, um broker, um tópico ou um grupo. As notas aparecem em **Notas de LLM** e nas telas dos objetos, assinadas com o nome do cliente. O texto é mostrado como está, sem formatação nem links.

- **Resolvido** e **Não relevante** fecham uma nota; as fechadas ficam em uma seção recolhida.
- **Excluir** exclui uma nota, e **Excluir todas as notas de LLM** exclui todas as notas da conexão.
- Uma nota é excluída automaticamente 7 dias depois de criada. Uma conexão pode ter no máximo 200 notas abertas.

<a id="with-llm"></a>

## Com um assistente LLM

O assistente executa a mesma verificação com a ferramenta `health_check` e vê as mesmas constatações, inclusive as ocultas, que aparecem marcadas. Se a última verificação tem menos de 5 minutos, o assistente recebe o resultado dela sem consultar o cluster de novo. Peça, por exemplo: “Verifique a saúde do cluster e deixe notas sobre os problemas mais importantes.”
