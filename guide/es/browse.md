# Explorar mensajes

Los mensajes se exploran en la pestaña **Mensajes** de un topic. La exploración no se une a ningún consumer group ni hace commit de nada, así que el clúster no cambia. También funciona en las conexiones con la opción **Solo lectura** activada. Para leer como miembro de un grupo, usa una [sesión de consumer](consumer-sessions.md).

<a id="start-reading"></a>

## Empezar a leer

1. Abre un topic desde la lista **Topics**. Primero se abre la pestaña **Mensajes**.
2. En **Desde**, elige dónde empezar. Consulta [Posición inicial](browse.md#start-position).
3. Si quieres, configura los formatos y los filtros.
4. Haz clic en **Leer**.

Los mensajes aparecen en la tabla a medida que llegan. La línea junto a los botones muestra cuántos mensajes se han revisado, cuántos coinciden con los filtros y cuántos hay en la tabla. También muestra el estado: **leyendo…**, **límite alcanzado**, **se alcanzó el final del topic** o **detenido**.

Sin `Live tail`, la lectura se detiene en el final del topic tal como estaba al hacer clic en **Leer**, o antes, al alcanzar el valor de **Límite**. El límite cuenta los mensajes que coinciden con los filtros. Puede ir de 1 a 100000; los valores grandes hacen la lectura más lenta.

Con `Live tail` activado, la lectura no se detiene al final: los mensajes nuevos se añaden a medida que se escriben. Cuando se han leído todos los mensajes antiguos, el estado muestra **esperando mensajes nuevos…**. La tabla conserva los últimos mensajes hasta el límite y descarta los más antiguos.

**Detener** detiene la lectura y conserva la tabla. Mientras se lee, el botón **Leer** pasa a ser **Reiniciar**: vuelve a empezar con los ajustes actuales y vacía la tabla. Si se pierde la conexión con el broker, la app se vuelve a conectar sola y lo indica en la línea de estado.

En la tabla se mezclan mensajes de distintas particiones. Kafka mantiene el orden solo dentro de una partición. La columna `P` muestra la partición y `Offset`, la posición dentro de ella. Selecciona `UTC` para mostrar las horas en UTC en lugar de la hora local.

<a id="start-position"></a>

## Posición inicial

**Posición inicial** determina dónde empieza la lectura en cada partición:

- **Últimos N**: los últimos N mensajes. N se reparte a partes iguales entre las particiones, así que con 100 mensajes y 4 particiones la app toma hasta 25 de cada una.
- **Desde el principio**: desde el mensaje más antiguo que aún se conserva.
- **Desde un offset**: desde el mismo offset en cada partición. Si una partición no tiene ese offset, la lectura empieza en su principio o en su final. Este modo es más útil con una sola partición.
- **Desde fecha y hora**: desde el primer mensaje con una marca de tiempo igual o posterior a la hora local indicada. En las particiones sin mensajes más recientes, la lectura empieza al final.
- **Últimos X minutos**: lo mismo, contando hacia atrás desde ahora.

El campo con el texto **particiones: todas** limita la lectura a algunas particiones. Escribe los números de partición separados por comas, por ejemplo `0,2`. Déjalo vacío para leer todas las particiones.

<a id="filters"></a>

## Filtros

Los filtros muestran solo los mensajes que coinciden. Aun así, la app lee todos los mensajes desde la posición inicial y comprueba cada uno, así que los filtros no aceleran la lectura. Deben coincidir todos los campos rellenados.

- **La clave contiene** y **El valor contiene**: texto en la clave o en el valor.
- `regex`: interpreta estos campos, y el valor del encabezado, como expresiones regulares.
- `Aa`: distingue entre mayúsculas y minúsculas. De forma predeterminada no se distinguen.
- **Encabezado**: el mensaje tiene un encabezado con este nombre exacto. **Valor del encabezado** comprueba además su valor.
- `JSONPath`: una condición sobre un valor JSON, por ejemplo `$.order.status == "FAILED"`. Sin `==` ni `!=`, basta con que la ruta exista, por ejemplo `$.error`. Los mensajes cuyo valor no es JSON válido no coinciden.
- **Fecha/hora desde** y **Fecha/hora hasta**: la marca de tiempo del mensaje está dentro de este intervalo, en hora local. Ambos extremos se incluyen.

Los filtros comprueban el texto que se muestra en la tabla, después de decodificarlo. Si el formato del valor es `hex`, el filtro de valor busca en el texto hexadecimal. Los filtros se aplican al hacer clic en **Leer**; después de cambiarlos, vuelve a leer.

Un filtro no mueve la posición inicial. Por ejemplo, **Últimos N** con un filtro solo encuentra coincidencias entre los últimos N mensajes. Para buscar en todo el topic, empieza con **Desde el principio**.

<a id="formats"></a>

## Formatos

**Formato** determina cómo se decodifican las claves y los valores, por separado para la **clave** y el **valor**. Solo cambia la visualización, no los datos.

- `auto`: el predeterminado. Los objetos y arrays JSON válidos se muestran como JSON, el resto del texto UTF-8 como cadena y los datos binarios en hexadecimal.
- `string`: texto en UTF-8. Los bytes que no son UTF-8 válido se sustituyen por `�`.
- `json`: JSON con sangría en los detalles del mensaje. Si el valor no es JSON válido, los detalles muestran el error.
- `hex`: cada byte como dos dígitos hexadecimales.
- `base64`: los bytes en Base64.

Los detalles muestran qué formato se ha usado realmente, por ejemplo `json` cuando `auto` ha detectado JSON. Los valores de los encabezados se muestran como texto; los valores binarios se muestran como `0x` seguido de hexadecimal. Una clave o un valor ausente se muestra como `null`.

Las claves y los valores muy grandes se muestran solo en parte, aproximadamente el primer MB de texto. Esto también se aplica a `hex` y `base64`. En ese caso, los detalles del mensaje muestran un aviso.

<a id="message-details"></a>

## Detalles del mensaje

Haz clic en una fila para abrir los detalles del mensaje junto a la tabla. Arriba están la partición, el offset y la hora. Debajo:

- **Valor** y **Clave**: el texto completo, con el formato y el tamaño en bytes. El JSON se muestra con sangría. **Copiar** copia el texto.
- **Encabezados**: una tabla con los nombres y valores de los encabezados.

Las columnas de la tabla son `P`, `Offset`, **Fecha/hora**, **Clave**, **Valor**, **Encabezados** (su número) y **Tamaño** (el tamaño de la clave y el valor en bytes). La hora es la marca de tiempo del mensaje. Según el ajuste `message.timestamp.type` del topic, la establece el productor o el broker.

<a id="export-messages"></a>

## Copiar mensajes

Glafira no guarda en un archivo los mensajes explorados. Para sacar un mensaje de la app, cópialo:

- **Copiar todo como JSON** copia el mensaje completo como JSON: topic, partición, offset, marca de tiempo, clave, valor, encabezados y tamaño. La clave y el valor se incluyen decodificados, con su formato.
- **Copiar** junto a **Valor** o **Clave** copia solo ese texto.

El texto copiado usa el formato elegido para la lectura. Para copiar datos binarios con exactitud, léelos como `hex` o `base64`. Las claves y los valores muy grandes se copian solo en parte, tal como se muestran, incluso en `hex` y `base64`. Consulta [Formatos](browse.md#formats).

<a id="open-in-producer"></a>

## Abrir en Producer

**Reenviar** abre el mensaje en el **Producer** de la misma conexión. El formulario recibe el topic, la clave, el valor, los encabezados y los formatos del mensaje. La partición se deja a Kafka. Los marcadores del texto del mensaje no se sustituyen: la copia se envía tal como se leyó.

No se envía nada de inmediato. Revisa el formulario, cámbialo si hace falta y envíalo desde el Producer. Consulta [Producer](producer.md).

El botón está deshabilitado en dos casos, y se muestra el motivo:

- la clave o el valor se mostraron solo en parte por ser demasiado grandes;
- el texto se decodificó con pérdidas porque no es UTF-8 válido. Léelo como `hex` o `base64` para enviar una copia exacta.

En las conexiones con la opción **Solo lectura** activada, el botón está oculto.

<a id="reset-group-here"></a>

## Restablecer un grupo a un mensaje

**Restablecer el grupo aquí…** prepara un restablecimiento de offsets para que un consumer group empiece a leer desde este mensaje.

1. En los detalles del mensaje, haz clic en **Restablecer el grupo aquí…**.
2. Elige un grupo en la lista. Solo aparecen los grupos existentes.
3. Haz clic en **Ir a restablecer**.

Se abre la pestaña del grupo con el formulario de restablecimiento ya rellenado. En **Estrategia** está seleccionado **Offset específico**, con el offset del mensaje. En **Alcance** está seleccionado **Particiones**, con el topic y la partición del mensaje. Todavía no cambia nada: haz clic en **Vista previa** para ver los nuevos offsets y aplica el restablecimiento desde ahí. El grupo no debe tener miembros activos. Consulta [Consumer groups](groups.md).

En las conexiones con la opción **Solo lectura** activada, el botón está oculto.
