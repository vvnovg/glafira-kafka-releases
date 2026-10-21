# Clúster y topics

Este capítulo trata el resumen del clúster, la lista de topics, la creación y eliminación de topics, la pantalla del topic y su configuración. Bajo cada conexión del panel **Conexiones** hay enlaces a **Resumen del clúster** y **Topics**. Cada uno se abre en su propia pestaña.

<a id="cluster-overview"></a>

## Resumen del clúster

**Resumen del clúster** muestra el estado del clúster en el momento en que se cargó. Haz clic en **Actualizar** para volver a cargarlo.

- `Cluster ID`: el identificador del clúster.
- **Controlador**: el ID del broker que el clúster indica como controlador. En los clústeres KRaft, los clientes no se comunican directamente con los controladores: el clúster indica aquí un broker aleatorio, no el controlador activo, y puede cambiar entre actualizaciones.
- **Brokers**: el número de brokers.
- **Topics (internos)**: todos los topics, con el número de topics internos entre paréntesis.
- **Particiones**: el número de particiones de todos los topics.
- `Under-replicated`: particiones en las que no todas las réplicas están sincronizadas (en el ISR). Normalmente un broker está caído o no da abasto.
- `Offline`: particiones sin leader. Nadie puede escribir en ellas ni leer de ellas.

Un valor distinto de cero en `Under-replicated` u `Offline` se resalta.

Debajo está la tabla de brokers con `ID` y **Dirección**. El broker del campo **Controlador** lleva la marca **controlador**. En los clústeres KRaft, es el mismo broker aleatorio, no el controlador activo. Haz clic en un broker para ver su configuración bajo la tabla. La configuración del broker solo se puede consultar. Usa **Buscar parámetro** para buscar un parámetro por nombre y **Solo modificados** para ocultar los parámetros con valores predeterminados. Un icono de candado indica los parámetros de solo lectura.

<a id="topic-list"></a>

## Lista de topics

**Topics** muestra los topics del clúster. Haz clic en una fila para abrir el topic en su propia pestaña.

Las columnas son **Nombre**, **Particiones**, `RF` (factor de replicación), **Mensajes (≈)**, `cleanup.policy` y **Subrepl.** (el número de particiones subreplicadas). El número de mensajes es aproximado: es la diferencia entre los offsets final e inicial de las particiones. La compactación y los marcadores de transacción hacen que el número real sea menor. El número de mensajes y `cleanup.policy` se cargan solo para las filas a las que te desplazas.

Controles de la barra de herramientas:

- **Buscar (o /regex/)**: escribe parte de un nombre para buscar topics; no se distingue entre mayúsculas y minúsculas. Para buscar con una expresión regular, escríbela entre barras, por ejemplo `/^orders\..*/`.
- **Internos**: muestra los topics internos, como `__consumer_offsets`. Llevan la marca **(interno)**.
- La lista de actualización automática: **Autoactualización: desactivada**, **cada 10 s** o **cada 30 s**.
- **Actualizar**: vuelve a cargar al momento la lista y el número de mensajes.

Junto a la barra de herramientas, la app muestra cuántos topics coinciden con la búsqueda respecto al total. En las conexiones con la opción **Solo lectura** activada, los botones **Crear topic** y **Seleccionar** están ocultos.

<a id="create-topic"></a>

## Crear un topic

1. En la lista de topics, haz clic en **Crear topic**. El formulario **Nuevo topic** se abre sobre la lista.
2. En **Nombre del topic**, escribe el nombre. Puede contener letras latinas, dígitos, `.`, `_` y `-`, hasta 249 caracteres. El formulario avisa si el nombre contiene a la vez `.` y `_`: los nombres de las métricas de Kafka de esos topics pueden coincidir con los de otros topics.
3. En **Particiones**, indica el número de particiones. Las particiones se leen en paralelo y el orden solo está garantizado dentro de una misma partición. El número puede aumentarse más adelante, pero nunca reducirse.
4. En **Factor de replicación**, indica cuántas copias de cada partición guardan los brokers. No puede ser mayor que el número de brokers, que el formulario muestra bajo el campo. El valor predeterminado es 3, o el número de brokers si hay menos. `-1` usa el valor predeterminado del broker.
5. En **Preajuste**, elige un preajuste:
   - **delete (predeterminado)**: los mensajes antiguos se eliminan por tiempo.
   - `compact`: solo se conserva el último valor de cada clave.
   - `compact + delete`: ambas cosas.
   - **prueba (retención 1 h)**: los mensajes se conservan una hora.
6. Si quieres, añade parámetros en **Configuración adicional (key=value)**, uno por línea, por ejemplo `retention.ms=86400000`. Tienen prioridad sobre el preajuste. Las líneas que empiezan por `#` se ignoran.
7. Haz clic en **Crear**.

Cuando el formulario es válido, muestra el comando `kafka-topics.sh` equivalente. Puedes copiarlo, por ejemplo, para crear el mismo topic en otro clúster.

En una conexión prod, confirmas la creación escribiendo el nombre de la conexión. Una vez creado el topic, se abre su pestaña.

<a id="delete-topics"></a>

## Eliminar topics

Para eliminar un topic, ábrelo y haz clic en **Eliminar topic**. Escribe el nombre del topic para confirmar y haz clic en **Eliminar**. La pestaña del topic se cierra.

Para eliminar varios topics a la vez:

1. En la lista de topics, haz clic en **Seleccionar**. Aparecen casillas junto a los nombres.
2. Marca los topics. Los topics internos no se pueden marcar.
3. Haz clic en el botón de eliminar. Muestra el número de topics marcados.
4. Revisa la lista de nombres, escribe su número para confirmar y haz clic en **Eliminar**.

**Cancelar selección** oculta las casillas.

En una conexión prod, las dos formas de eliminar piden además escribir el nombre de la conexión.

La eliminación no se puede deshacer: los datos se pierden. Si algunos topics no se eliminan, por ejemplo por falta de permisos, la app los enumera con el motivo. Los demás topics se eliminan.

<a id="topic-details"></a>

## Pantalla del topic

La pestaña del topic muestra el número de particiones, el factor de replicación y el número aproximado de mensajes. Los botones de la parte superior:

- **Actualizar**: vuelve a cargar los datos del topic.
- **Enviar mensaje**: abre el **Producer** con este topic. Consulta [Producer](producer.md).
- **Leer como grupo**: inicia una sesión de consumer para este topic. Consulta [Sesiones de consumer](consumer-sessions.md).

En las conexiones con la opción **Solo lectura** activada, ambos botones están deshabilitados.

La segunda fila contiene acciones que cambian el topic. Está oculta en las conexiones de solo lectura y en los topics internos.

- **Agregar particiones**: escribe el nuevo total y haz clic en **Aumentar**. El número de particiones no se puede reducir. Tras aumentarlo, los mensajes con la misma clave pueden ir a otra partición, así que se rompe el orden por clave.
- **Purgar**: elimina todos los mensajes hasta el final actual de las particiones seleccionadas. El topic en sí se conserva. Selecciona las particiones, escribe el nombre del topic y haz clic en **Purgar particiones**. La purga solo funciona si `cleanup.policy` incluye `delete`; si no, el botón está deshabilitado y se muestra el motivo. En un topic `compact,delete`, la purga también elimina los últimos valores de las claves.
- **Eliminar topic**: consulta [Eliminar topics](topics.md#delete-topics).

En una conexión prod, cada una de estas acciones pide además el nombre de la conexión.

Las pestañas de abajo:

- **Mensajes**: explorar mensajes. Consulta [Explorar mensajes](browse.md).
- **Particiones**: una tabla con cada partición: **Leader**, **Réplicas**, `ISR` (réplicas sincronizadas), los offsets inicial (`Low`) y final (`High`) y el número de mensajes. Una partición sin leader muestra **ninguno**.
- **Configuración**: los parámetros del topic. Consulta [Configuración del topic](topics.md#topic-config).
- **Consumer groups**: los consumer groups que tienen offsets con commit para este topic, con **Estado**, **Particiones con commits** y **Lag**. Haz clic en un grupo para abrirlo. Consulta [Consumer groups](groups.md).

<a id="topic-config"></a>

## Configuración del topic

La pestaña **Configuración** muestra todos los parámetros del topic en tres columnas:

- **Parámetro**: el nombre del parámetro. Los nombres en negrita están establecidos explícitamente: en el topic o en la configuración del broker.
- **Valor**: el valor actual. Los valores confidenciales se ocultan como `••••••`.
- **Origen**: de dónde procede el valor. Por ejemplo, `DynamicTopic` se ha establecido en este topic, `StaticBroker` y `DynamicBroker` proceden de la configuración del broker y `Default` es el valor predeterminado de Kafka.

Usa **Buscar parámetro** para buscar un parámetro por nombre y **Solo modificados** para mostrar solo los parámetros en negrita. Un icono de candado indica los parámetros de solo lectura. El icono `?` junto a un nombre explica el parámetro.

<a id="edit-config"></a>

## Cambiar la configuración

1. En la pestaña **Configuración**, haz clic en **Editar**. En las conexiones con la opción **Solo lectura** activada, este botón está oculto.
2. Cambia los valores en la columna **Valor**. Los parámetros de solo lectura y los confidenciales no se pueden editar. Para quitar el valor propio del topic, haz clic en **restablecer**: entonces se aplica el valor del broker. No se permite un valor vacío.
3. Haz clic en **Revisar cambios**. La app muestra cada cambio como valor anterior → valor nuevo; **predeterminado** significa que el valor se restablece.
4. Haz clic en **Aplicar**, o en **Atrás** para seguir editando.

Nada cambia en el clúster hasta que aplicas los cambios. **Descartar ediciones** cancela todas las ediciones. Mientras haya ediciones sin guardar, **Cerrar editor** está deshabilitado. En una conexión prod, confirmas los cambios escribiendo el nombre de la conexión.

Solo se envían los parámetros que has cambiado. Los demás parámetros del topic no se modifican. El broker comprueba los valores y rechaza con un error los que no son válidos.

<a id="config-parameters"></a>

## Parámetros del topic

Los topics de Kafka tienen decenas de parámetros. Los iconos `?` de la tabla de configuración explican cada uno. Los que más se usan son estos:

- `cleanup.policy`: qué ocurre con los datos antiguos. `delete` elimina los segmentos antiguos por tiempo o por tamaño. `compact` conserva solo el último valor de cada clave. `compact,delete` hace ambas cosas.
- `retention.ms`: cuánto tiempo se conservan los mensajes, en milisegundos. El valor predeterminado es `604800000` (7 días); `-1` los conserva para siempre. Funciona si `cleanup.policy` incluye `delete`.
- `retention.bytes`: el tamaño máximo de cada partición antes de que se eliminen los segmentos antiguos. El valor predeterminado `-1` significa sin límite de tamaño.
- `segment.ms` y `segment.bytes`: cuándo empieza el broker un nuevo segmento del log. La retención y la compactación trabajan con segmentos completos, así que un mensaje puede permanecer más tiempo que `retention.ms` hasta que se cierre su segmento.
- `min.insync.replicas`: cuántas réplicas deben confirmar una escritura con `acks=all`. Normalmente 2 con un factor de replicación de 3.
- `max.message.bytes`: el batch más grande que el broker acepta para este topic. El valor predeterminado es de 1 MB aproximadamente. Los productores deben permitir el mismo tamaño (`max.request.size`).
- `compression.type`: cómo guarda el broker los mensajes. `producer` mantiene la compresión del productor.
- `message.timestamp.type`: `CreateTime` (la hora establecida por el productor) o `LogAppendTime` (la hora en que el broker escribió el mensaje). Es la hora que se muestra al explorar mensajes.
- `delete.retention.ms`: cuánto tiempo permanecen los marcadores de eliminación (mensajes con valor nulo) en un topic compactado. El valor predeterminado es 1 día.

Algunos parámetros solo existen en versiones más recientes de Kafka y otros están obsoletos. El icono `?` lo indica cuando es relevante. Los parámetros de tiered storage, como `remote.storage.enable`, solo funcionan si el clúster lo admite.
