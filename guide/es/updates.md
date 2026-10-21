# Actualizaciones y diagnóstico

Este capítulo explica cómo actualizar la app, la pantalla **Acerca de**, el archivo de diagnóstico y cómo apoyar el proyecto.

<a id="updates"></a>

## Actualizaciones

Al iniciarse, la app comprueba una vez si hay una versión nueva. Si la hay, un aviso sobre las pestañas indica su versión. **Detalles** abre la pantalla **Acerca de**. Si la comprobación falla, por ejemplo sin acceso a internet, no se muestra ningún mensaje.

Para comprobarlo manualmente:

1. Abre **Acerca de** en la parte inferior del panel izquierdo.
2. En la sección **Actualizaciones**, haz clic en **Buscar actualizaciones**.
3. Si hay una versión nueva, la pantalla muestra su número y las notas de la versión. Haz clic en **Instalar y reiniciar**.

La app descarga la actualización, comprueba su firma, la instala y se reinicia. Si la descarga se interrumpe por un problema de red, se reintenta automáticamente, hasta tres intentos en total. Si los tres intentos fallan, la app muestra **La descarga de la actualización se interrumpió — comprueba tu red e inténtalo de nuevo**. Si la comprobación de la firma falla, la app muestra **La firma de la actualización no superó la verificación — la actualización no se instaló**.

La instalación puede pedir una contraseña de administrador, porque la app está instalada para todos los usuarios del equipo: con el instalador `.pkg` en macOS, el instalador `.msi` en Windows o un paquete `.deb` o `.rpm` en Linux. Los usuarios sin derechos de administrador no pueden actualizar una instalación así.

Si se están leyendo mensajes, hay una sesión de consumer en ejecución o se está enviando un lote, la app pregunta antes e indica cuántas sesiones están activas. Haz clic en **Detener sesiones y actualizar** para detenerlas y continuar, o en **Cancelar**. Las conexiones, los ajustes y los demás datos se conservan tras una actualización.

Las actualizaciones se descargan desde la página de versiones en GitHub. Si la comprobación falla con **No se pudo buscar actualizaciones**, revisa tu red o tu proxy. **Las actualizaciones no están configuradas en este build** significa que este build se creó sin la clave del actualizador, por ejemplo un build de desarrollo o de prueba. En ese caso, descarga las versiones nuevas desde la página de versiones: https://github.com/vvnovg/glafira-kafka-releases/releases

Una organización puede desactivar la búsqueda de actualizaciones mediante una política o dirigirla a una réplica interna. Cuando la búsqueda está desactivada, la sección **Actualizaciones** indica **Las actualizaciones las gestiona tu organización** después de hacer clic en **Buscar actualizaciones**. Al igual que **Las actualizaciones no están configuradas en este build**, este mensaje solo aparece tras el clic; en estos casos, la comprobación al iniciar no muestra nada.

<a id="about"></a>

## Acerca de

Abre **Acerca de** en la parte inferior del panel izquierdo. En macOS, también está en el menú Ayuda como **Acerca de Glafira**.

La pantalla muestra una breve descripción de la app y además:

- **Versión** — el número de versión, y **Build** — el identificador del código fuente a partir del cual se creó. Incluye ambos en una solicitud de soporte.
- **Sistema** — el sistema operativo y la arquitectura del procesador.
- **Licencia** — Apache 2.0; el enlace abre el texto de la licencia en el navegador.
- **Política de la organización** — **establecida** o **ninguna**. Los errores de la política se enumeran debajo.
- **Datos:** — la carpeta con los perfiles de conexión, las plantillas, el historial y los ajustes.
- **Registros:** — la carpeta con los archivos de registro. Consulta [Registros](troubleshooting.md#logs).

Las secciones **Actualizaciones** y **Diagnóstico** también están en esta pantalla.

<a id="diagnostics"></a>

## Archivo de diagnóstico

El archivo de diagnóstico reúne lo que el soporte necesita para entender un problema. Para crearlo:

1. Abre **Acerca de**.
2. En la sección **Diagnóstico**, haz clic en **Recopilar diagnóstico**. En macOS, también puedes elegir **Recopilar diagnóstico…** en el menú Ayuda.
3. Elige dónde guardar el archivo. El nombre propuesto es `diagnostics-<date>.zip`.

Una vez guardado el archivo, la pantalla muestra su ruta. El archivo ZIP contiene:

- `info.json`: la versión y el build de la app, el sistema operativo, las rutas de las carpetas de datos y de registros, y el estado de la política;
- `connections.json`: los perfiles de conexión: nombres, entornos, servidores bootstrap, protocolos, usuarios SASL, rutas de los archivos de certificado, propiedades adicionales y tiempos de espera;
- `logs/`: los archivos de registro de la app.

El archivo no contiene contraseñas: se guardan en el almacén de secretos del sistema, y al guardar una conexión se rechazan las propiedades adicionales con contraseñas y secretos. Tampoco incluye los archivos de certificado. Los nombres de host, los usuarios y las rutas de archivo se guardan como texto sin cifrar, y los registros contienen direcciones de brokers procedentes de los mensajes de error de Kafka. Revisa el archivo antes de enviarlo fuera de tu empresa.

La app no envía nada por sí sola. Adjunta tú mismo el archivo a tu solicitud de soporte; consulta [Contactar con el soporte](troubleshooting.md#support).

<a id="support"></a>

## Apoyar el proyecto

Glafira es gratuita y de código abierto bajo la Apache License 2.0. Las donaciones la mantienen viva. **Apoyar el proyecto** en la parte inferior del panel izquierdo, en la pantalla **Acerca de** y, en macOS, en el menú Ayuda abre la página de apoyo en el navegador: https://github.com/vvnovg/glafira-kafka-releases/blob/main/SUPPORT.md
