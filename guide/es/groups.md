# Consumer groups

Un consumer group es un conjunto de consumidores que se reparten la lectura de los topics. Kafka asigna cada partición a un miembro del grupo y guarda, para cada partición, el offset del que el grupo ha hecho commit. Tras un reinicio, las aplicaciones siguen leyendo desde ese offset.

Abre **Consumer groups** bajo la conexión en el panel **Conexiones**. Los grupos de un topic también aparecen en la pestaña **Consumer groups** del topic. Consulta [Pantalla del topic](topics.md#topic-details).

<a id="group-list"></a>

## Lista de grupos

La lista muestra todos los consumer groups del clúster. Las columnas:

- **Grupo**: el ID del grupo (`group.id`).
- **Estado**: el estado que indica Kafka, por ejemplo `Stable` (los miembros están leyendo), `Empty` (sin miembros), `PreparingRebalance` o `CompletingRebalance` (las particiones se están redistribuyendo) y `Dead` (el grupo ya no existe).
- **Miembros**: el número de miembros activos.
- `Assignor`: la estrategia con la que se reparten las particiones, por ejemplo `range`.
- **Lag**: el lag total del grupo. Consulta [Lag](groups.md#lag).
- **Topics**: los topics para los que el grupo tiene offsets con commit.

Escribe en **Buscar grupos** para mostrar los grupos cuyo ID contiene el texto; no se distingue entre mayúsculas y minúsculas. La lista de estados de al lado muestra solo los grupos en un estado; **Todos los estados** quita este filtro. El contador indica cuántos grupos se muestran y cuántos hay en total.

Los miembros, el assignor y el lag se cargan solo para las filas visibles, a medida que te desplazas. Hasta que llegan, la celda muestra `…`. Si no se pueden cargar, la celda muestra `—`; sitúa el puntero sobre ella para ver el motivo.

**Actualizar** vuelve a cargar la lista y sus datos. Si algunos brokers no han respondido, un aviso indica que la lista puede estar incompleta.

<a id="lag"></a>

## Lag

El lag es el número de mensajes que el grupo aún no ha leído. Para cada partición, es el final del log menos el offset con commit. El lag nunca se muestra por debajo de cero.

El lag depende solo de los offsets con commit. Un consumidor activo puede haber leído más allá de los offsets con commit, así que su lag real puede ser menor. Cuando el grupo vuelve a hacer commit, el lag baja.

Una partición sin offset con commit muestra **sin commit** y no cuenta en el total. El lag total de un grupo muestra `—` cuando el grupo no tiene ningún commit (**sin commits**) o cuando no se han podido leer los offsets de alguna partición (**lag desconocido: error de offsets de partición**).

El lag cuenta offsets, no mensajes. En los topics con compactación o transacciones, algunos offsets no contienen mensajes visibles, así que al grupo le pueden quedar por leer menos mensajes de los que indica el lag.

Los valores son una instantánea del momento de la carga. Para seguir el lag, abre el grupo y activa la actualización automática.

<a id="group-details"></a>

## Detalles del grupo

Haz clic en un grupo de la lista para abrir su pestaña. Arriba, un resumen muestra el estado, el número de miembros y el assignor.

- **Autoactualización**: **Autoactualización: desactivada**, **cada 5 s** o **cada 10 s**.
- **Actualizar**: vuelve a cargar al momento.
- **Leer como miembro**: inicia una [sesión de consumer](consumer-sessions.md) en este grupo. El formulario de la sesión recibe el ID del grupo y los topics para los que el grupo tiene commits. El botón está deshabilitado en las conexiones con la opción **Solo lectura** activada; sitúa el puntero sobre el botón para ver el motivo.

**Particiones** muestra las particiones para las que el grupo tiene commits y las particiones asignadas a sus miembros. El lag total del grupo se muestra sobre la tabla. Las columnas son **Topic**, **Partición**, `Committed` (el offset con commit), `Log end` (el offset final de la partición), **Lag** y **Miembro** (el client ID del miembro que lee la partición).

**Miembros** muestra los miembros activos: `Client ID`, **Host**, `Member ID` y **Asignación** (las particiones del miembro como `topic:partition`). Un grupo sin miembros muestra **No hay miembros activos.**

La segunda fila contiene **Restablecer offsets** y **Eliminar grupo**. Está oculta en las conexiones de solo lectura.

<a id="reset-offsets"></a>

## Restablecer offsets

Restablecer offsets mueve los offsets con commit de un grupo. Después, el grupo vuelve a leer mensajes o se los salta. Los offsets solo se pueden restablecer en un grupo sin miembros activos (estado `Empty` o `Dead`). Si no, los miembros sobrescribirían los nuevos offsets con sus propios commits. Detén antes las aplicaciones del grupo.

1. En la pestaña del grupo, haz clic en **Restablecer offsets**. Mientras el grupo tiene miembros, el botón está deshabilitado; sitúa el puntero sobre el botón para ver el motivo.
2. En **Estrategia**, elige la estrategia. Consulta [Estrategias de restablecimiento](groups.md#reset-strategies).
3. En **Alcance**, elige el alcance. Consulta [Alcance del restablecimiento](groups.md#reset-scope).
4. Haz clic en **Vista previa**.
5. Revisa la tabla y haz clic en el botón de aplicar. Muestra el número de particiones que cambiarán.

La vista previa no cambia nada. Para cada partición muestra el topic, la partición, **Actual** (el offset con commit), **Nuevo** (el offset de destino) y `Low / High` (los offsets inicial y final de la partición). Una partición para la que no se puede calcular un destino se marca con **Sin offset de destino — no se aplicará** y el motivo. Esas particiones se omiten.

Solo se aplica el plan que muestra la vista previa. Cualquier cambio en el formulario oculta la vista previa, y hay que volver a hacer clic en **Vista previa**. En una conexión prod, aplicar pide el nombre de la conexión; en las demás conexiones, se aplica al momento.

Antes de aplicar, la app vuelve a comprobar que el grupo no tiene miembros. Después de aplicar, **Resultado** enumera cada partición con su nuevo offset o un error.

También puedes preparar un restablecimiento a partir de un mensaje: consulta [Restablecer un grupo a un mensaje](browse.md#reset-group-here). **Cerrar** cierra el panel de restablecimiento.

<a id="reset-strategies"></a>

## Estrategias de restablecimiento

**Estrategia** determina el offset de destino de cada partición del alcance:

- **Desde el principio (earliest)**: el primer offset que aún se conserva. El grupo vuelve a leer todo lo que guarda el topic.
- **Desde el final (latest)**: el final de la partición. El grupo se salta todo lo acumulado y lee solo los mensajes nuevos.
- **Por fecha y hora**: el primer mensaje escrito en la fecha y hora indicadas en **Fecha y hora (local)** o después. En las particiones sin mensajes más recientes, el destino es el final.
- **Offset específico**: el valor indicado en **Offset**, un entero igual o mayor que 0, en todas las particiones. Si una partición no tiene ese offset, el destino es su offset inicial o final.
- **Desplazar ±N**: mueve el offset con commit según el valor de **Desplazamiento (±N)**. Un número negativo vuelve a leer N mensajes hacia atrás; uno positivo se salta N mensajes. El resultado se mantiene entre los offsets inicial y final. Las particiones sin offset con commit no se pueden desplazar y se omiten.

La hora es tu hora local, con segundos. Kafka busca el offset según las marcas de tiempo de los mensajes.

<a id="reset-scope"></a>

## Alcance del restablecimiento

**Alcance** determina qué particiones se restablecen. Las demás particiones conservan sus offsets.

- **Todo el grupo**: todas las particiones para las que el grupo tiene offsets con commit. Si el grupo aún no tiene commits, elige topics.
- **Topics**: todas las particiones de los topics indicados, incluidas las particiones sin commits. Separa los nombres con comas, espacios o saltos de línea. Úsalo para que un grupo empiece a leer un topic nuevo desde el punto elegido.
- **Particiones**: solo las particiones indicadas. Escríbelas en **Particiones (topic:partición)** como `topic:partition`, separadas por comas o espacios, por ejemplo `orders:0, orders:3`.

<a id="delete-group"></a>

## Eliminar un grupo

Al eliminar un grupo se eliminan todos sus offsets con commit. Solo se puede eliminar un grupo sin miembros, en estado `Empty`.

1. En la pestaña del grupo, haz clic en **Eliminar grupo**. Mientras el grupo tiene miembros, el botón está deshabilitado; sitúa el puntero sobre el botón para ver el motivo.
2. Escribe el ID del grupo para confirmar. En una conexión prod, escribe también el nombre de la conexión.
3. Haz clic en **Eliminar**.

Tras la eliminación, la pestaña del grupo se cierra. Si una aplicación con este `group.id` vuelve a iniciarse, Kafka crea el grupo de nuevo. Sin offsets con commit, la aplicación empieza donde indica su ajuste `auto.offset.reset`: desde el principio o desde el final.

En las conexiones con la opción **Solo lectura** activada, el botón está oculto.
