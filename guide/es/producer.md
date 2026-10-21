# Producer

El **Producer** envía mensajes a Kafka: de uno en uno desde un formulario o muchos a la vez. Ábrelo desde la conexión en el panel **Conexiones** o con **Enviar mensaje** en la pestaña de un topic, que rellena el topic. Cada conexión tiene su propia pestaña de Producer.

En las conexiones con la opción **Solo lectura** activada, el Producer se abre con el aviso **La conexión es de solo lectura — no se puede enviar.** Los botones de envío están ocultos, pero las plantillas y el historial se pueden consultar.

<a id="send-message"></a>

## Enviar un mensaje

1. En **Topic**, escribe el topic. El campo sugiere los topics del clúster, sin los internos.
2. Si quieres, escribe un número en **Partición**. Cuando el campo muestra **auto**, la partición se elige según la clave: los mensajes con la misma clave van a la misma partición, como con el cliente de Java. Los mensajes sin clave van a una partición aleatoria.
3. Rellena los campos **Clave** y **Valor** y elige sus formatos. Consulta [Formatos de clave y valor](producer.md#key-value-formats).
4. Si quieres, añade encabezados.
5. Haz clic en **Enviar**.

Tras un envío correcto, una línea bajo el formulario muestra la partición, el offset y la hora del mensaje. Con `acks` en `0`, el broker no devuelve ningún offset y se muestra `—` en su lugar.

Si el número de partición es demasiado grande o el broker no confirma el mensaje dentro del tiempo de espera de la conexión, la app muestra un error. Un topic que no existe también da error, salvo que el broker cree topics automáticamente (`auto.create.topics.enable`). Con la creación automática desactivada, este error puede tardar hasta unos 30 s en aparecer (el tiempo de espera de la conexión). Los errores del formulario, como un JSON no válido o un marcador desconocido, se muestran antes de enviar nada.

En una conexión prod, el envío te pide escribir el nombre de la conexión y confirmar con **Enviar a prod**. Consulta [Entornos y prod](connections.md#environments).

**Limpiar formulario** vacía el formulario y devuelve los ajustes del productor a sus valores predeterminados. El topic se conserva.

<a id="key-value-formats"></a>

## Formatos de clave y valor

**Formato de clave** y **Formato de valor** determinan cómo se convierte en bytes el texto del campo:

- `string`: el texto tal cual, en UTF-8.
- `json`: se comprueba que el texto sea JSON válido y se envía tal como se escribió, sin reformatear.
- `hex`: dos dígitos hexadecimales por byte, por ejemplo `48656c6c6f`. Los espacios y los saltos de línea se ignoran.
- `base64`: bytes en Base64 estándar.
- `null`: sin clave o sin valor. El campo queda vacío y, al escribir texto, el formato cambia a `string`.

El valor predeterminado es `null` para la clave y `json` para el valor. Para enviar un mensaje con clave, basta con escribirla: el formato cambia a `string`. Una cadena vacía también es una clave, no una ausente: si eliges `string` y dejas el campo vacío, todos esos mensajes obtienen el mismo hash y van a una misma partición.

Un mensaje con valor `null` es un tombstone. En un topic con `cleanup.policy=compact`, indica a Kafka que elimine los mensajes anteriores con la misma clave.

<a id="headers"></a>

## Encabezados

Los encabezados son pares de nombre y valor que se envían con el mensaje. Haz clic en **Agregar encabezado** para añadir una fila y rellena **Nombre**, **Valor** y **Formato**. El botón `×` elimina la fila.

El formato del valor de un encabezado es `string` (texto en UTF-8) o `hex` (bytes como dígitos hexadecimales). Las filas con el nombre vacío no se envían. Kafka permite varios encabezados con el mismo nombre, y se envían todos.

Los marcadores funcionan en los valores de los encabezados, pero no en sus nombres.

<a id="placeholders"></a>

## Marcadores

Los marcadores se sustituyen por valores en cada envío. Funcionan en la clave, el valor y los valores de los encabezados. La lista se muestra bajo los encabezados:

- `{{uuid}}`: un UUID aleatorio nuevo, por ejemplo `0b6f6f2c-3c1e-4a8e-9a54-2f0d6f1d7c21`.
- `{{now}}`: la hora actual en UTC en formato ISO 8601 con milisegundos, por ejemplo `2026-10-15T12:30:00.000Z`.
- `{{nowMillis}}`: la hora actual en milisegundos desde 1970-01-01 UTC.
- `{{seq}}`: el número de la copia en un lote, empezando por 0. En un envío individual, es `0`.
- `{{randomInt 1 100}}`: un entero aleatorio entre los dos números, ambos incluidos.
- `{{randomString 8}}`: una cadena aleatoria de letras latinas y dígitos de la longitud indicada, hasta 100000.
- `{{env NAME}}`: el valor de la variable de entorno `NAME`. Si no está definida, no se envía nada y se muestra un error.

Se permiten espacios dentro de las llaves. Un marcador desconocido o la falta de `}}` detiene el envío con un error. Para enviar `{{` como texto, escribe `\{{`.

Con el formato `json`, el JSON se comprueba después de sustituir los marcadores. Los marcadores que producen texto necesitan comillas: `{"id": "{{uuid}}"}`. Los números no: `{"n": {{seq}}}`.

`{{env NAME}}` lee el entorno del proceso de Glafira. Si inicias Glafira desde el Dock o el menú Inicio, pueden faltar las variables definidas solo en el perfil de tu terminal.

Cuando un mensaje procede de [Abrir en Producer](browse.md#open-in-producer) o del historial, cada `{{` se escribe como `\{{`. Así el mensaje se envía exactamente como era, sin sustituir nada.

<a id="templates"></a>

## Plantillas

Una plantilla guarda todo el formulario: topic, partición, clave, valor, encabezados, formatos y ajustes del productor. Los marcadores se guardan tal como están escritos y se sustituyen en cada envío.

Para guardar una plantilla, haz clic en **Guardar como plantilla**, escribe un nombre en **Nombre de la plantilla** y haz clic en **Guardar**. El topic puede estar vacío. De forma predeterminada, la plantilla pertenece a esta conexión. Marca **Para todas las conexiones** para mostrarla en todas las conexiones; esas plantillas llevan la marca **(todas las conexiones)** en la lista.

Para usar una plantilla, elígela en la lista **Plantilla**. El formulario se sustituye por la plantilla. Puedes editar el formulario antes de enviar; la plantilla no cambia.

Guardar siempre crea una plantilla nueva. Para cambiar una, cárgala, edita el formulario, guárdala con otro nombre y elimina la antigua. Para eliminar la plantilla seleccionada, haz clic en **Eliminar plantilla** y confirma con **Eliminar**.

Las plantillas de una conexión se eliminan junto con la conexión.

<a id="history"></a>

## Historial

**Historial (últimos 100)** muestra los últimos 100 envíos individuales de esta conexión, del más reciente al más antiguo. Los mensajes de los envíos por lotes no se añaden. Cada fila muestra la hora, el topic, la partición y el offset, y el principio de la clave.

El historial guarda el mensaje tal como se envió, con los marcadores ya sustituidos. **Reenviar** lo carga en el formulario junto con sus ajustes del productor. No se envía nada hasta que haces clic en **Enviar**. La copia se envía exactamente como está guardada: por ejemplo, se vuelve a enviar el mismo UUID.

Una clave, un valor o un valor de encabezado de más de 64 KB se guarda solo en parte. En esas filas, **Reenviar** está deshabilitado.

El historial se guarda en este equipo como texto sin cifrar, incluidos los valores de `{{env NAME}}`. Si esto es importante, no envíes secretos a través del Producer. El historial de una conexión se elimina junto con la conexión.

<a id="batch-copies"></a>

## N copias del formulario

**Envío por lotes**, bajo el formulario, envía muchos mensajes en una sola ejecución. En el modo **N copias del formulario**, envía el formulario el número de veces indicado.

1. Rellena el formulario como para un envío individual.
2. Selecciona **N copias del formulario** y escribe el número en **Copias**, de 1 a 1000000.
3. Si quieres, indica un valor en **Velocidad**. Consulta [Velocidad de envío](producer.md#batch-rate).
4. Haz clic en **Iniciar**.

Los marcadores se sustituyen por separado en cada copia. `{{seq}}` es el número de copia, de 0 a N−1, así que cada mensaje puede tener su propia clave, por ejemplo `order-{{seq}}`. Los errores de los marcadores se comprueban en la primera copia antes de iniciar la ejecución.

Durante la ejecución, el panel muestra cuántos mensajes se han enviado y cuántos han fallado. Al terminar, muestra **completado**. Los errores se enumeran con el número de copia, contando desde 1. La lista muestra los 20 primeros errores; si hay más, una nota debajo indica el total.

**Cancelar** detiene la ejecución. Los mensajes que aún no han salido de la app no se envían, y el panel muestra cuántos son. La app espera hasta 2 segundos a que el broker confirme los mensajes ya enviados. Los que siguen sin confirmar cuentan como errores, porque no se sabe si se escribieron. Cerrar la pestaña del Producer también detiene la ejecución.

En una conexión prod, la ejecución pide el nombre de la conexión y se confirma con **Iniciar en prod**.

<a id="batch-file"></a>

## Enviar desde un archivo

En el modo **desde archivo**, cada línea o fila de un archivo se convierte en un mensaje. Se admiten archivos JSON Lines (`.jsonl`, `.ndjson`) y CSV (`.csv`).

1. En el formulario de arriba, escribe el topic en **Topic**. Todos los mensajes van a este topic.
2. Selecciona **desde archivo** y haz clic en **Elegir archivo…**.
3. Haz clic en **Iniciar**.

La clave, el valor, los encabezados y la partición del formulario no se usan, y los marcadores no se sustituyen. Se aplican los ajustes del productor del formulario.

En JSON Lines, cada línea es un objeto JSON, por ejemplo `{"key": "k1", "value": {"id": 1}, "headers": {"source": "test"}, "partition": 0}`. Todos los campos son opcionales:

- `key` y `value`: una cadena JSON se envía como su texto, sin comillas. Cualquier otro valor JSON, como un objeto o un número, se envía como texto JSON. `null` o un campo ausente significa sin clave o sin valor.
- `headers`: un objeto con valores de tipo cadena.
- `partition`: un número de partición. Si falta, la partición se elige según la clave.

Las líneas vacías se omiten. El archivo debe estar en UTF-8.

En CSV, la primera fila contiene los nombres de las columnas. La columna `value` es obligatoria; las columnas `key`, `partition` y `header.NAME` son opcionales, por ejemplo `header.source`. Una celda `key` vacía significa sin clave, y una celda de encabezado vacía significa que ese encabezado no se envía. Una celda `value` vacía envía un valor vacío, no `null`, así que desde CSV no se puede enviar un tombstone.

El archivo se lee como un flujo, así que el número total no se conoce de antemano y se muestra como `—`. Una línea o un registro de más de 16 MB, un JSON no válido o una partición negativa cuentan como error de esa línea, y la ejecución continúa con la siguiente. Los errores se muestran con el número de línea.

<a id="batch-rate"></a>

## Velocidad de envío

**Velocidad** limita un lote al número indicado de mensajes por segundo. Escribe un entero igual o mayor que 1. Un campo vacío significa **sin límite**.

Los mensajes se reparten de forma uniforme en el tiempo: a 10 mensajes por segundo, sale un mensaje cada 100 ms aproximadamente. Usa el límite para no sobrecargar un clúster compartido o para simular una carga constante.

Sin límite, la app envía tan rápido como el broker acepta. La velocidad se aplica a los dos modos de envío por lotes.

<a id="producer-settings"></a>

## Ajustes del productor

Los **Ajustes del productor** se aplican a los envíos individuales y a los lotes. Se guardan en las plantillas y en el historial.

`acks` determina cuántas réplicas deben confirmar una escritura:

- `all`: el predeterminado. El leader espera a todas las réplicas sincronizadas. Es la opción más segura. Si hay menos réplicas sincronizadas de las que exige el ajuste `min.insync.replicas` del topic, la escritura falla.
- `1`: solo confirma el leader. Más rápido, pero un mensaje puede perderse si el leader falla antes de que los followers lo copien.
- `0`: el productor no espera ninguna confirmación. Los mensajes pueden perderse y no se devuelve ningún offset.

La opción **compresión** comprime los batches de mensajes: `none` (el predeterminado), `gzip`, `snappy`, `lz4` o `zstd`. La compresión ahorra tráfico de red y espacio en disco a costa de algo de CPU. `zstd` y `lz4` son opciones habituales. Para un único mensaje pequeño, la ganancia es pequeña.

La opción **idempotencia** establece `enable.idempotence`. Evita duplicados cuando el cliente reintenta un envío tras un error de red. Requiere `acks` en `all`; con otros valores, la app muestra un error y no envía nada.
