# Sesiones de consumer

Una sesión de consumer lee topics como un miembro real de un consumer group.

<a id="what-is"></a>

## Qué es una sesión

La [exploración](browse.md) lee mensajes sin unirse a ningún grupo y no cambia nada en el clúster. Una sesión es distinta: se une a un grupo con el `group.id` indicado, recibe particiones de Kafka y puede hacer commit de offsets. Úsala para leer lo que leería un grupo, para mover los offsets de un grupo mediante commits o para observar cómo se rebalancea un grupo.

Una sesión afecta al clúster. Cuando se une o sale, Kafka redistribuye las particiones del grupo y los demás miembros pasan por un rebalanceo. Los commits cambian los offsets del grupo, que usarán sus aplicaciones.

Las sesiones no están disponibles en las conexiones con la opción **Solo lectura** activada: la pestaña muestra **Las sesiones de consumer no están disponibles para conexiones de solo lectura: unirse a un grupo cambia su estado.** Si una conexión pasa a solo lectura mientras se ejecuta una sesión, la sesión sale del grupo.

<a id="start-session"></a>

## Iniciar una sesión

Abre una sesión de una de estas tres formas:

- **Sesión de consumer** bajo la conexión en el panel **Conexiones**: un formulario vacío;
- **Leer como grupo** en la pestaña de un topic: el topic ya está rellenado;
- **Leer como miembro** en la pestaña de un grupo: el grupo y los topics para los que tiene commits ya están rellenados.

Cada vez se abre una pestaña nueva, así que pueden ejecutarse varias sesiones a la vez.

En **Grupo y topics**:

- `group.id`: el grupo al que unirse. Si no escribes ninguno, la app propone un nombre nuevo, como `glafira-kafka-anna-1a2b3c`. Un grupo nuevo no tiene commits.
- **Topics**: nombres de topics separados por comas o espacios. También puedes marcar **topics por regex** y escribir una expresión en **Expresión regular de topics**, por ejemplo `orders\..*`. La expresión se compara desde el principio del nombre: si falta `^`, se añade. Los topics que coinciden y se crean más tarde se añaden en unos minutos (al actualizarse los metadatos); esto provoca un rebalanceo.
- `auto.offset.reset`: dónde empezar en las particiones para las que el grupo no tiene offset con commit: **desde el final (latest)** (el predeterminado) o **desde el principio (earliest)**. Donde el grupo tiene un offset con commit, la lectura continúa desde él.
- **commit automático**: hace commit de los offsets leídos automáticamente. Desactivado de forma predeterminada. Consulta [Commits](consumer-sessions.md#commits).
- `isolation.level`: `read_committed` (el predeterminado) muestra solo los mensajes de transacciones confirmadas. `read_uncommitted` muestra también los mensajes de transacciones abiertas y anuladas.

En **Formato**, elige los valores de **Formato de clave** y **Formato de valor**, como en la exploración. Consulta [Formatos](browse.md#formats). **Búfer, mensajes** determina cuántos mensajes conserva la tabla, de 1 a 100000; el valor predeterminado es 5000.

Los filtros funcionan como en la exploración. Consulta [Filtros](browse.md#filters).

Haz clic en **Unirse al grupo**. En una conexión prod, la app te pide escribir el nombre de la conexión y confirmar con **Confirmar**. Mientras la sesión se ejecuta, el formulario no se puede cambiar.

<a id="reading"></a>

## Lectura

La línea de estado junto a los botones muestra **conectando…** y luego **en el grupo**. También muestra cuántos mensajes se han leído, cuántos coinciden con los filtros y cuántos hay en la tabla. Si se pierde la conexión con el broker, la sesión se vuelve a conectar sola y lo indica en la línea de estado.

Los mensajes aparecen en la tabla a medida que llegan, y la sesión sigue leyendo mensajes nuevos hasta que la detienes. Cuando la tabla se llena, se eliminan los mensajes más antiguos. Haz clic en una fila para ver los detalles del mensaje, como en la exploración. Consulta [Detalles del mensaje](browse.md#message-details). Selecciona `UTC` para mostrar las horas en UTC.

La sesión lee solo las particiones que Kafka le ha asignado. Si el grupo tiene otros miembros, estos leen las demás particiones. Si el grupo tiene más miembros que particiones, puede que la sesión no reciba ninguna partición.

**Desconectar** sale del grupo, y los demás miembros reciben sus particiones. Cerrar la pestaña de la sesión también hace que salga del grupo. Cuando la sesión ha terminado, la línea de estado muestra **desconectado**.

Si un topic no existe, la sesión se detiene con un error y muestra los nombres que hay que comprobar.

<a id="commits"></a>

## Commits

Un commit guarda un offset para el grupo. El siguiente miembro que lea la partición, o la misma aplicación tras un reinicio, empieza desde ese offset.

Con la opción **commit automático** activada, la sesión hace commit de los offsets leídos en segundo plano cada pocos segundos, y una vez más al salir del grupo. Los mensajes ocultos por los filtros cuentan como leídos.

Sin commit automático, la sesión no hace commit de nada por sí sola. Usa los botones:

- **Commit hasta el seleccionado**: hace commit del offset posterior al mensaje seleccionado, solo en su partición. El grupo continuará desde el mensaje siguiente. Si el mensaje seleccionado es anterior al commit actual, el offset retrocede.
- **Commit de todo lo leído**: hace commit de la posición actual en todas las particiones asignadas. Esto incluye los mensajes ocultos por los filtros y los ya eliminados de la tabla.

Tras un commit, una línea muestra los offsets con commit como `topic:partition → offset`. En una conexión prod, cada commit pide el nombre de la conexión.

Si la sesión no responde a tiempo, la app avisa de que no se sabe si el commit se aplicó. Comprueba los offsets en la pestaña del grupo. Consulta [Detalles del grupo](groups.md#group-details).

<a id="pause-partitions"></a>

## Pausar particiones

El panel a la derecha de la tabla muestra las particiones asignadas a la sesión como `topic:partition`. Haz clic en **Pausar** junto a una partición para dejar de leerla y en **Reanudar** para continuar. Una partición en pausa lleva la marca **en pausa**.

Pausar no hace salir del grupo, así que no hay rebalanceo. La partición sigue asignada a la sesión y los demás miembros no la leen mientras está en pausa. Pausar no cambia nada en el clúster y no requiere confirmación.

Tras un rebalanceo, las particiones recién asignadas empiezan sin pausa. Cuando la sesión sale del grupo, todas las pausas terminan.

<a id="rebalance-log"></a>

## Registro de rebalanceos

**Registro de rebalanceos**, bajo la lista de particiones, registra cada cambio de la asignación, del más reciente al más antiguo. Cada entrada tiene una hora y uno de tres tipos: particiones asignadas a la sesión, particiones retiradas de ella o un error de rebalanceo con su texto.

Un rebalanceo ocurre cuando un miembro se une al grupo o sale de él, cuando cambian los topics suscritos o cuando un miembro deja de responder a tiempo. Para observarlo, inicia una segunda sesión con el mismo `group.id` en otra pestaña: las particiones se reparten entre las dos sesiones y ambos registros lo muestran.

El registro conserva las últimas 200 entradas. Antes del primer rebalanceo, muestra **Aún no hay rebalanceos.**

<a id="limits"></a>

## Limitaciones

- Las sesiones no están disponibles en las conexiones con la opción **Solo lectura** activada.
- Una sesión no puede empezar desde un offset elegido. Para fijar el inicio, restablece antes los offsets del grupo: consulta [Restablecer offsets](groups.md#reset-offsets). El restablecimiento requiere un grupo sin miembros, así que desconecta las sesiones de este grupo antes de restablecer.
- La tabla conserva como máximo 100000 mensajes. Los más antiguos se eliminan, pero siguen contando como leídos.
- Las claves y los valores muy grandes se muestran solo en parte, como en la exploración.
- No te unas al grupo de una aplicación en ejecución, a menos que sea lo que pretendes. La sesión le quita particiones a la aplicación, y la aplicación no las procesa mientras la sesión las tiene. Los commits de la sesión, incluidos los automáticos, mueven los offsets de la aplicación.
