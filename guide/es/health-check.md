# Comprobación del clúster y notas

Glafira comprueba un clúster con un conjunto de reglas integradas y muestra los problemas encontrados junto a los objetos: en el árbol de conexiones, en las listas de topics y grupos y en las pantallas de un topic o un grupo. Un asistente LLM puede completar estos hallazgos con sus propias notas. La comprobación solo lee el clúster y no cambia nada en él.

<a id="run"></a>

## Ejecutar una comprobación

Abre **Resumen del clúster** o **Hallazgos** de una conexión y haz clic en **Comprobar clúster**. Una comprobación tarda hasta 45 segundos: Glafira lee los brokers, las particiones, las configuraciones de topics (hasta 2000 topics) y los consumer groups (hasta 500). Si hay más topics o grupos, si algo no se puede leer o si se agotan los 45 segundos, el resultado se marca como **comprobado parcialmente**, y el hallazgo **Comprobación incompleta** muestra lo que no se leyó.

Glafira no consulta el clúster por su cuenta: una comprobación solo se ejecuta cuando haces clic en el botón o cuando la pide un asistente LLM. El resultado se conserva hasta la siguiente comprobación.

<a id="rules"></a>

## Qué se comprueba

- Brokers y particiones: sin controlador, particiones no disponibles y subreplicadas, un clúster de un solo broker, desequilibrio de leaders, leaders fuera de la réplica preferida.
- Durabilidad de los topics: factor de replicación 1, `min.insync.replicas` mayor que el factor de replicación, `min.insync.replicas` igual al factor de replicación, `min.insync.replicas` igual a 1, elección de leader no limpia.
- Configuración de los brokers: creación automática de topics, factor de replicación por defecto 1, `min.insync.replicas` por defecto 1, ajustes importantes que difieren entre brokers.
- Consumer groups: grupos vacíos con lag, grupos en rebalanceo, miembros sin particiones, lag que creció desde la comprobación anterior.

<a id="severity"></a>

## Gravedad y entorno

Cada hallazgo tiene una gravedad: **Crítico**, **Advertencia** o **Información**. Depende del entorno de la conexión. Por ejemplo, el factor de replicación 1 es crítico en prod, una advertencia en stage y no se muestra en dev, test y local. El entorno se define en el formulario de la conexión.

<a id="tab"></a>

## La pestaña “Hallazgos”

En la pestaña **Hallazgos** los hallazgos se agrupan por regla; cada uno explica el problema y qué hacer. Fíltralos por **Gravedad**, **Origen** y **Objeto**, o busca por nombre. El nombre del objeto abre su pantalla.

Una insignia con un número en el árbol de conexiones y en las listas de topics y grupos muestra cuántos hallazgos y notas LLM abiertas con gravedad **Crítico** o **Advertencia** afectan al objeto. Sitúa el puntero sobre la insignia para ver sus títulos.

<a id="hide"></a>

## Ocultar hallazgos

Si un hallazgo es esperado para un objeto concreto, por ejemplo un topic de prueba con factor de replicación 1 en prod, haz clic en **Ocultar para este objeto**. Un hallazgo oculto no cuenta en las insignias y sigue oculto en las comprobaciones siguientes. Para ver los ocultos, activa “Mostrar ocultos”; ahí encontrarás **Restaurar**.

<a id="notes"></a>

## Notas LLM

Un asistente LLM conectado por MCP (consulta [Asistentes LLM (MCP)](llm-mcp.md#how-it-works)) puede dejar una nota sobre el clúster, un broker, un topic o un grupo. Las notas aparecen en **Notas LLM** y en las pantallas de los objetos, firmadas con el nombre del cliente. El texto se muestra tal cual, sin formato ni enlaces.

- **Resuelto** y **No relevante** cierran una nota; las cerradas se guardan en una sección plegada.
- **Eliminar** elimina una nota y **Eliminar todas las notas LLM** elimina todas las notas de la conexión.
- Una nota se elimina automáticamente 7 días después de crearse. Una conexión puede tener como máximo 200 notas abiertas.

<a id="with-llm"></a>

## Con un asistente LLM

El asistente ejecuta la misma comprobación con la herramienta `health_check` y ve los mismos hallazgos, incluidos los ocultos, que aparecen marcados. Si la última comprobación tiene menos de 5 minutos, el asistente recibe su resultado sin volver a consultar el clúster. Pide, por ejemplo: “Comprueba el estado del clúster y deja notas sobre los problemas más importantes”.
