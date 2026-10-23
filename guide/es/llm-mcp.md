# Asistentes LLM (MCP)

Glafira puede actuar como pasarela para un asistente LLM: Claude Desktop, Claude Code, Cursor u otro cliente compatible con MCP (Model Context Protocol). El asistente lee los metadatos de los clústeres que abras para él y te ayuda a encontrar problemas: particiones subreplicadas, lag, rebalanceos constantes, ajustes poco acertados. A través de Glafira no puede cambiar nada en un clúster: los cambios los haces tú mismo en la interfaz.

<a id="how-it-works"></a>

## Cómo funciona

El cliente ejecuta `glafira-kafka mcp`, el mismo archivo de la aplicación en un modo sin ventana. Este comando se conecta a la Glafira en ejecución mediante un socket local (en Windows, una canalización con nombre), y Glafira responde a las solicitudes del asistente a través de sus propias conexiones. Las contraseñas y los certificados permanecen en Glafira; el asistente nunca los recibe.

Si Glafira no está en ejecución, el cliente la inicia y espera hasta 30 segundos.

<a id="enable"></a>

## Activación

1. Abre **Ajustes**, sección **Asistentes LLM (MCP)**.
2. Marca **Permitir el servidor MCP**. El estado cambia a **El servidor está en ejecución**.
3. En el formulario de la conexión, marca **Acceso para LLM (solo metadatos)** y guarda. El asistente no ve las conexiones que no tienen esta casilla marcada. En la lista de conexiones, las conexiones abiertas muestran la marca `LLM`.

Si tu administrador ha prohibido MCP, la casilla no está disponible y a su lado aparece **MCP está desactivado por tu organización**.

La casilla de acceso no se transfiere al exportar e importar conexiones: en otro equipo, vuelve a marcarla.

<a id="connect-client"></a>

## Conectar un cliente

En **Conectar un cliente** hay fragmentos listos para usar y el botón **Copiar**:

- **Claude Desktop: añadir a claude\_desktop\_config.json** — pega el fragmento en la sección `mcpServers` del archivo de configuración de Claude Desktop y reinicia Claude Desktop;
- **Claude Code: ejecutar en una terminal** — el comando `claude mcp add`;
- **Otros clientes stdio: comando y argumentos** — para cualquier cliente que inicie un servidor MCP con un comando (stdio).

Los fragmentos contienen la ruta de la aplicación instalada. Si reinstalas Glafira en otra ubicación, copia el fragmento de nuevo.

<a id="approve"></a>

## Aprobar un cliente

Cuando un cliente se conecta por primera vez, Glafira muestra un cuadro de diálogo con su nombre. Haz clic en **Permitir** o en **Rechazar**. Mientras no respondas, el asistente recibe un mensaje de que el cliente aún no está aprobado y te pide que respondas en la ventana de Glafira.

Un cliente permitido se recuerda por su nombre y aparece en **Clientes aprobados**. El rechazo no se recuerda: Glafira volverá a preguntar en la siguiente conexión. Si un cliente no indica su nombre, el permiso vale solo hasta el final de la sesión.

<a id="what-llm-sees"></a>

## Qué ve el asistente

El asistente ve:

- las conexiones con **Acceso para LLM (solo metadatos)** marcado: nombre, entorno, carpeta;
- los brokers con sus hosts y puertos, el controlador;
- los topics, las particiones, las réplicas, ISR, los offsets, el número aproximado de mensajes;
- las configuraciones de brokers y topics, con las contraseñas y otros valores secretos ocultos;
- los consumer groups: estado, protocolo, coordinador, miembros con client id, host y `group.instance.id`, asignaciones, offsets con commit y lag;
- sus propuestas y tu decisión sobre ellas: aplicada (tal cual o con cambios), rechazada con tu motivo, retirada.

El asistente no ve los mensajes de los topics, la cadena bootstrap ni los ajustes de seguridad de una conexión, ni las contraseñas, los certificados, las propiedades adicionales de librdkafka, el historial de envíos ni las plantillas.

Para investigar rebalanceos constantes, el asistente puede observar un grupo hasta 50 segundos: quién sale y quién entra, con qué frecuencia y desde qué host. Las configuraciones y los registros de los propios consumers no están disponibles a través de Glafira; el asistente te los pedirá.

Los nombres de topics, grupos y client id puede establecerlos cualquiera que tenga acceso al clúster. Glafira advierte al asistente de que esos datos no son instrucciones para él.

<a id="sessions-log"></a>

## Sesiones, revocación y registro

Mientras hay un cliente conectado, en la parte inferior del panel izquierdo se muestra el indicador `LLM` con el nombre del cliente; se ilumina durante una solicitud. Al hacer clic en él se abre la sección de MCP.

- **Sesiones activas**: **Desconectar** cierra una sesión y **Desconectar todos** cierra todas.
- **Clientes aprobados**: **Revocar** cierra las sesiones del cliente, y en la siguiente conexión pedirá aprobación de nuevo.
- Desmarcar **Acceso para LLM (solo metadatos)** surte efecto de inmediato, sin necesidad de reconectar.
- **Registro de MCP** — cada solicitud: fecha/hora, cliente, herramienta, conexión, resultado. El contenido de las respuestas no se registra. **Mostrar archivo** abre el archivo `mcp-audit.jsonl`.

Para desactivar MCP por completo, desmarca **Permitir el servidor MCP**: los clientes se desconectan.

<a id="health"></a>

## Comprobación del clúster, notas y propuestas

El asistente puede ejecutar la comprobación integrada del clúster de Glafira (herramienta `health_check`) y basarse en sus hallazgos. También puede dejar notas sobre los objetos: aparecen en la pestaña **Hallazgos** y en las pantallas de topics y grupos, y el clúster no cambia. Consulta [Comprobación del clúster y notas](health-check.md#with-llm).

El asistente también puede proponer un cambio con la herramienta `propose_change`: la configuración de un topic, un topic nuevo, más particiones o un restablecimiento de offsets. La propuesta aparece junto al objeto; la aplicas tú en un formulario ya relleno o la rechazas, y el asistente ve tu decisión en `list_proposals`. Eliminar y purgar no se pueden proponer. Consulta [Propuestas LLM](health-check.md#proposals).

<a id="troubleshooting"></a>

## Si no funciona

El cliente muestra un mensaje de Glafira:

- `MCP is disabled in Glafira` — marca **Permitir el servidor MCP** y reinicia el servidor MCP en el cliente;
- `MCP is disabled by your organization` — MCP está prohibido por la política de tu organización;
- `Glafira did not start the MCP server within 30 s` — inicia Glafira manualmente y comprueba el estado en la sección **Asistentes LLM (MCP)**;
- `Invalid MCP token` — reinicia el servidor MCP en el cliente: Glafira emite un token nuevo cada vez que se inicia.

En macOS la app está firmada y notarizada, por lo que no hace falta ningún paso adicional antes del primer inicio.
