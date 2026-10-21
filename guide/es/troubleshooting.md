# Solución de problemas

Este capítulo explica los errores de conexión habituales y qué comprobar en cada caso. También indica dónde están los registros y qué incluir en una solicitud de soporte. En macOS, el elemento **Solución de problemas** del menú Ayuda abre este capítulo.

Un mensaje de error tiene un texto breve y, en la mayoría de las pantallas, el código del error entre paréntesis, por ejemplo `TIMEOUT`. Despliega **Detalles** para ver el mensaje original del cliente de Kafka (librdkafka). A menudo indica la dirección exacta del broker y la causa.

<a id="cannot-connect"></a>

## No se puede conectar

Empieza por **Probar conexión** en el formulario de la conexión; consulta [Probar la conexión](connections.md#test). La prueba se detiene en el primer paso que falla, y el paso indica dónde buscar:

- **Configuración**: el problema está en el propio formulario: una dirección sin puerto, un archivo de certificado que falta o no se puede leer, una contraseña de certificado incorrecta. El error indica el campo. Para los archivos de certificado, consulta [Errores de TLS](troubleshooting.md#tls-errors).
- **DNS / TCP**: el equipo no puede llegar a los servidores bootstrap. Consulta [DNS y TCP](troubleshooting.md#dns-tcp).
- **Kafka (TLS, SASL, metadatos)**: la red funciona, pero han fallado el handshake TLS, el inicio de sesión SASL o la solicitud de metadatos. Consulta [Errores de TLS](troubleshooting.md#tls-errors), [Errores de SASL](troubleshooting.md#sasl-errors) y [Tiempos de espera](troubleshooting.md#timeouts).

La causa más habitual es un protocolo que no corresponde al puerto. Cada listener del broker tiene su propio puerto y protocolo. Pregunta a tus administradores de Kafka qué puerto corresponde a qué protocolo; consulta [Protocolos de seguridad](connections.md#security).

Si la prueba se supera, pero la lectura o el envío fallan, la causa suele estar en las direcciones de los brokers. El cliente obtiene la lista de brokers del servidor bootstrap y después se conecta a cada broker en la dirección que este anuncia (`advertised.listeners`). Si estas direcciones son internas, por ejemplo nombres de host de Docker o Kubernetes, no son accesibles desde tu equipo. Las direcciones aparecen en la tabla de brokers del resumen del clúster; consulta [Resumen del clúster](topics.md#cluster-overview). Pide a tus administradores un listener accesible desde fuera.

<a id="dns-tcp"></a>

## DNS y TCP

**Ninguno de los servidores bootstrap es accesible por TCP** significa que la app no ha podido abrir una conexión TCP con ninguna dirección de **Servidores bootstrap**. Cada dirección dispone de hasta 5 segundos. Los detalles muestran la última dirección probada y el motivo. `DNS` en los detalles significa que no se ha podido resolver el nombre de host.

Comprueba:

- el nombre de host y el puerto, sin un prefijo de protocolo como `https://`;
- que la VPN esté activada, si el clúster está en una red privada;
- que un firewall no bloquee el puerto;
- en un broker en Docker, que el puerto esté publicado en el host, por ejemplo `-p 9092:9092`.

Para comprobar el puerto fuera de la app, ejecuta `nc -vz <host> <port>` en macOS o Linux, o `Test-NetConnection <host> -Port <port>` en PowerShell en Windows.

**No se pudo conectar con los brokers** después de superar el paso TCP suele significar que un broker cerró la conexión o que más tarde no se pudo llegar a él. Comprueba también las direcciones de los brokers; consulta [No se puede conectar](troubleshooting.md#cannot-connect).

<a id="tls-errors"></a>

## Errores de TLS

Errores en el paso **Kafka (TLS, SASL, metadatos)** durante el handshake TLS:

- **El broker acepta conexiones sin TLS en esta dirección — comprueba el puerto o elige un protocolo sin SSL**.
- **El certificado del broker no está firmado por una CA de confianza — revisa “CA de confianza”**. En **Certificado de CA de confianza**, indica el archivo de la CA que firmó los certificados de los brokers. Tus administradores pueden darte este archivo. Con el campo vacío, solo se confía en los certificados raíz del sistema.
- **El nombre de host del broker no coincide con su certificado**. Conéctate con el nombre para el que se emitió el certificado, no con una dirección IP ni un alias. Solo en entornos de prueba puedes desactivar **Verificar nombre de host**.
- **El broker rechazó el certificado de cliente**. El broker exige un certificado de cliente o no confía en el que usas. Comprueba que la opción **Certificado de cliente (mTLS)** esté activada y que el certificado se haya emitido para este clúster. Consulta [Certificado de cliente (mTLS)](connections.md#mtls).
- **Error de conexión TLS**: otro problema de TLS. Despliega **Detalles** para ver la causa.

Si has elegido un protocolo sin SSL y aun así ves un error de TLS, probablemente el puerto pertenece a un listener TLS. En ese caso, los detalles dicen `connecting to a SSL listener?`. Elige `SSL` o `SASL_SSL`, o usa el puerto de un listener sin TLS.

Los problemas con los archivos de certificado se notifican en el paso **Configuración**, antes de que la app se conecte. El mensaje empieza por el nombre del campo: CA de confianza, certificado de cliente o clave de cliente. Casos habituales:

- La ruta no es absoluta o el archivo no se puede leer. Escribe la ruta completa o elige el archivo con **Examinar…**.
- **Clave de cliente: la clave está cifrada — indica la contraseña de la clave**, **Clave de cliente: contraseña de la clave incorrecta**, **Certificado de cliente: contraseña del almacén PKCS\#12 incorrecta** o **Contraseña del almacén JKS incorrecta (o el archivo está dañado)**: escribe la contraseña correcta en el formulario.
- **La clave de cliente no corresponde al certificado de cliente**: el archivo de la clave pertenece a otro certificado.
- El certificado de cliente ha caducado o aún no es válido: el mensaje indica la fecha. Solicita un certificado nuevo.
- El almacén PKCS#12 usa un algoritmo heredado o está en formato JCEKS. Convierte el archivo como se indica en [Formatos de certificado](connections.md#certificate-formats).

<a id="sasl-errors"></a>

## Errores de SASL

**Error de autenticación** significa que el broker no ha aceptado el inicio de sesión SASL. Comprueba:

- el usuario y la contraseña. Al editar una conexión guardada, un campo de contraseña vacío conserva la contraseña guardada. Vuelve a escribir la contraseña si no estás seguro de que sea correcta;
- **Mecanismo SASL**. Debe ser el mecanismo para el que se creó tu usuario. Las credenciales SCRAM se crean por separado para `SCRAM-SHA-256` y `SCRAM-SHA-512`, y los usuarios `PLAIN` se configuran aparte;
- el protocolo: `SASL_SSL` y `SASL_PLAINTEXT` suelen usar puertos distintos.

**La contraseña SASL de la conexión no está establecida**: no hay ninguna contraseña guardada para esta conexión, por ejemplo después de una importación. Abre la conexión, escribe la contraseña y guárdala. Consulta [Contraseñas](connections.md#passwords).

**No se puede acceder al almacén de secretos del sistema**: la app no puede leer la contraseña del almacén de secretos del sistema. En macOS, permite el acceso cuando el Llavero lo pida. En Linux, comprueba que GNOME Keyring, KWallet u otro proveedor de Secret Service se esté ejecutando en tu sesión.

**No tienes autorización para realizar la operación** es distinto: el inicio de sesión se ha completado, pero el usuario de Kafka no tiene permisos (ACL) para esta operación, por ejemplo leer un topic o un grupo. Pide los permisos a tus administradores. Ten en cuenta que Kafka no informa de ningún error sobre los topics que no tienes permiso para describir: simplemente no aparecen en la lista de topics.

<a id="timeouts"></a>

## Tiempos de espera

**Tiempo de espera agotado** significa que el broker no ha respondido a tiempo. Las causas típicas son una red o una VPN lentas, un clúster sobrecargado o direcciones de brokers inaccesibles; consulta [No se puede conectar](troubleshooting.md#cannot-connect).

Un problema de TLS o SASL puede parecer un tiempo de espera agotado. La app intenta encontrar la causa real y mostrarla en su lugar, pero no siempre es posible. Despliega **Detalles**: mensajes como `SSL handshake failed` o `SASL authentication error` señalan la causa.

Si la red simplemente es lenta, aumenta el valor de **Tiempo de espera de solicitud, ms** en el formulario de la conexión. El valor predeterminado es `30000` ms. Este valor se aplica a las solicitudes a los brokers, al envío de mensajes y a la lectura. La prueba de conexión usa sus propios límites (5 segundos por cada dirección TCP y 10 segundos para el paso de Kafka), así que un valor mayor no cambia la prueba.

<a id="read-only-blocked"></a>

## Una acción no está disponible o se rechaza

Si un botón está oculto o deshabilitado, o se rechaza una acción, comprueba estas causas:

- La conexión es de solo lectura. La indica un icono de candado junto a su nombre. Los botones que cambian el clúster están ocultos o deshabilitados. Si aun así una solicitud de ese tipo llega al núcleo de la app, por ejemplo porque el modo se activó con una pantalla abierta, el error indica que la conexión es de solo lectura. Para hacer cambios, abre la conexión, desactiva la opción **Solo lectura** y guarda. Consulta [Solo lectura](connections.md#read-only).
- La conexión es prod. Los cambios te piden escribir el nombre de la conexión. Mientras el nombre no coincida, la acción no se realiza. Consulta [Entornos y prod](connections.md#environments).
- El consumer group tiene miembros activos. Los offsets solo se pueden restablecer y un grupo solo se puede eliminar cuando no tiene miembros. Consulta [Restablecer offsets](groups.md#reset-offsets).
- El ajuste lo gestiona tu organización. Una política puede fijar el idioma de la interfaz y la búsqueda de actualizaciones, y estos ajustes no se pueden cambiar en la app. Consulta [Ajustes establecidos por tu organización](getting-started.md#managed-settings).
- Kafka ha rechazado la operación con **No tienes autorización para realizar la operación**. Es una cuestión de permisos en el clúster, no de los ajustes de Glafira. Consulta [Errores de SASL](troubleshooting.md#sasl-errors).

<a id="logs"></a>

## Registros

La app escribe un registro de su funcionamiento y de los errores del cliente de Kafka. La carpeta exacta se muestra en la pantalla **Acerca de**, en **Registros:** De forma predeterminada:

- macOS: `~/Library/Logs/io.github.vvnovg.glafira.kafka`
- Windows: `%LOCALAPPDATA%\io.github.vvnovg.glafira.kafka\logs`
- Linux: `~/.local/share/io.github.vvnovg.glafira.kafka/logs`

El archivo actual es `app.log`. Cuando alcanza 5 MB, se renombra a `app.1.log`, y los archivos anteriores pasan a `app.2.log` y así sucesivamente. La app conserva hasta cinco archivos y elimina el más antiguo.

Las contraseñas no se escriben en los registros. Sí se escriben las direcciones de los brokers y los textos de los errores de Kafka. No hace falta reunir los archivos a mano: el archivo de diagnóstico los incluye; consulta [Archivo de diagnóstico](updates.md#diagnostics).

<a id="support"></a>

## Contactar con el soporte

Antes de contactar con el soporte, ejecuta **Probar conexión** y anota qué paso falla. Después envía la solicitud a quien se encargue del soporte de Glafira en tu caso, por ejemplo tus administradores. Incluye:

- el archivo de diagnóstico; consulta [Archivo de diagnóstico](updates.md#diagnostics). Contiene la versión, el build, los registros y los perfiles de conexión sin contraseñas;
- qué hiciste, qué esperabas y qué ocurrió, paso a paso;
- el texto completo del error, junto con el contenido de **Detalles**. Puedes seleccionar y copiar este texto;
- una captura de pantalla, si el problema está en la interfaz;
- la hora en que se produjo el error, para encontrarlo en los registros.

Si no puedes crear el archivo de diagnóstico, indica los valores de **Versión** y **Build** de la pantalla **Acerca de** y adjunta los archivos de registro de la carpeta de registros.
