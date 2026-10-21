# Primeros pasos

Glafira for Apache Kafka es un cliente de escritorio para clústeres de Kafka. Permite explorar topics y consumer groups, leer y enviar mensajes y restablecer offsets. Este capítulo trata la instalación, el primer inicio, un Kafka local para practicar y el idioma de la interfaz.

<a id="install-macos"></a>

## Instalación en macOS

Glafira funciona en macOS 11 o posterior. Descarga el instalador desde la página de versiones: https://github.com/vvnovg/glafira-kafka-releases/releases

Elige el archivo para tu Mac: `aarch64` para Apple silicon (M1 y posteriores), `x64` para Intel.

1. Abre el archivo `.dmg`.
2. Arrastra Glafira for Apache Kafka a la carpeta Aplicaciones.
3. Inicia la app desde Aplicaciones o Launchpad.

También puedes usar el instalador `.pkg`. Instala la app en `/Applications` para todos los usuarios.

Los builds beta aún no están firmados por Apple, así que macOS bloquea el primer inicio. Para permitirlo:

1. Intenta abrir la app una vez y cierra el aviso.
2. Abre Ajustes del Sistema → Privacidad y seguridad.
3. Desplázate hasta el mensaje sobre Glafira for Apache Kafka y haz clic en “Abrir igualmente”. Si se solicita, confirma con tu contraseña.

En macOS 14 y anteriores también puedes hacer clic derecho en la app, elegir “Abrir” y, de nuevo, “Abrir”.

Si esto no funciona, quita una vez el atributo de cuarentena en Terminal:

```bash
xattr -dr com.apple.quarantine "/Applications/Glafira for Apache Kafka.app"
```

Si no se abre el propio `.dmg` descargado, quítale antes el atributo:

```bash
xattr -d com.apple.quarantine ~/Downloads/Glafira_for_Apache_Kafka_*.dmg
```

Las actualizaciones instaladas desde la propia app no requieren estos pasos.

<a id="install-windows"></a>

## Instalación en Windows

Descarga el instalador `.msi` (`x64`) desde la página de versiones: https://github.com/vvnovg/glafira-kafka-releases/releases

1. Ejecuta el archivo `.msi`.
2. Confirma la instalación en el aviso de Windows. La app se instala para todos los usuarios del equipo, por lo que se necesitan derechos de administrador.
3. Inicia Glafira for Apache Kafka desde el menú Inicio.

Los builds beta aún no están firmados. Si SmartScreen bloquea el instalador, haz clic en “Más información” y luego en “Ejecutar de todas formas”. También puedes desbloquear el archivo en PowerShell antes de ejecutarlo:

```powershell
Unblock-File -Path "$env:USERPROFILE\Downloads\Glafira_for_Apache_Kafka_*_x64_en-US.msi"
```

<a id="install-linux"></a>

## Instalación en Linux

Hay builds para `x86_64` en tres formatos. Descarga el que corresponda a tu distribución desde la página de versiones: https://github.com/vvnovg/glafira-kafka-releases/releases

- Debian, Ubuntu y derivadas: `sudo apt install ./<file>.deb`
- Fedora, RHEL y derivadas: `sudo dnf install ./<file>.rpm`
- Cualquier distribución: haz ejecutable el AppImage con `chmod +x <file>.AppImage` y ejecútalo.

Tras instalar un paquete, inicia la app desde el menú de aplicaciones o ejecuta `glafira-kafka` en una terminal.

Glafira guarda las contraseñas en Secret Service (por ejemplo, GNOME Keyring o KWallet). Asegúrate de que uno de ellos se esté ejecutando en tu sesión. Si no, la app no podrá guardar las contraseñas de las conexiones.

<a id="first-run"></a>

## Primer inicio

La ventana tiene dos partes. A la izquierda está el panel **Conexiones**. A la derecha están las pestañas: cada pantalla que abres, como una lista de topics o el Producer, tiene su propia pestaña.

Mientras no hay conexiones, el panel muestra una breve indicación y el área de pestañas propone añadir una conexión. Para empezar:

1. Haz clic en `+` ( **Nueva conexión** ) en la cabecera del panel **Conexiones**.
2. Rellena los campos **Nombre** y **Servidores bootstrap**, por ejemplo `localhost:9092`.
3. Haz clic en **Probar conexión** para comprobar los ajustes y luego en **Guardar**.

Al guardar, se abre el resumen del clúster. Bajo cada conexión del panel hay enlaces a **Resumen del clúster**, **Topics**, **Consumer groups**, **Producer** y **Sesión de consumer**. Todos los ajustes de conexión se describen en [Conexiones](connections.md#create).

En la parte inferior del panel hay enlaces a **Ajustes**, **Apoyar el proyecto**, **Ayuda** y **Acerca de**. El signo `?` junto a un campo muestra una breve ayuda sobre él. Pulsa `F1` en cualquier parte de la ventana para abrir el capítulo de la guía sobre la pantalla actual. En macOS, la guía también está en el menú Ayuda.

Si usabas Kafka Explorer (el nombre anterior de la app), Glafira copia en el primer inicio sus conexiones, plantillas e historial de envíos. Las contraseñas también se copian del almacén de secretos del sistema. No se sobrescribe nada que ya exista y los datos antiguos no se eliminan. macOS puede pedir una vez acceso a los elementos antiguos del Llavero.

<a id="local-cluster"></a>

## Kafka local para practicar

Para probar la app sin un clúster compartido, inicia Kafka en tu equipo con Docker. Basta con un broker:

```bash
docker run -d --name kafka -p 9092:9092 apache/kafka:3.9.0
```

Después crea una conexión con el servidor bootstrap `localhost:9092`, el protocolo `PLAINTEXT` y el entorno `local`. Cuando ya no necesites el broker, elimínalo con `docker rm -f kafka`.

La indicación del panel **Conexiones** vacío menciona `docker compose -f dev/docker-compose.yml up -d`. Este archivo forma parte del código fuente de Glafira. Si tienes el código fuente, este comando inicia un broker con tres listeners:

- `localhost:9092` — `PLAINTEXT`.
- `localhost:9094` — `SASL_PLAINTEXT` con `PLAIN` (usuarios `tester` / `tester-secret` y `admin` / `admin-secret`) y SCRAM (`scram-user` / `scram-secret` para `SCRAM-SHA-256` y `SCRAM-SHA-512`).
- `localhost:9095` — `SSL` con certificados de cliente (mTLS). Los certificados de prueba están en `dev/certs/out`.

Estos usuarios y certificados son solo para pruebas locales.

<a id="language"></a>

## Idioma de la interfaz

La interfaz está disponible en inglés, ruso, alemán, francés, español, italiano, polaco, portugués (Brasil), chino (simplificado y tradicional), japonés y coreano. Todos los idiomas son gratuitos.

Para cambiar el idioma:

1. Haz clic en **Ajustes** en la parte inferior del panel izquierdo.
2. Elige un idioma en **Idioma de la interfaz**.

El cambio se aplica al instante, también a esta guía y al menú de macOS. La opción **Idioma del sistema** (predeterminada) sigue el idioma del sistema operativo. Si ese idioma no está en la lista, la interfaz se muestra en inglés.

<a id="managed-settings"></a>

## Ajustes establecidos por tu organización

En una empresa, los administradores pueden fijar algunas opciones para todos los usuarios mediante una política. Estas opciones no se pueden cambiar en la app. Una política puede:

- fijar el idioma de la interfaz: la lista de idiomas en **Ajustes** queda entonces deshabilitada, con una nota de que el idioma lo ha establecido tu organización;
- desactivar la búsqueda de actualizaciones o dirigirla a una réplica interna.

Para saber si se aplica una política, abre **Acerca de** y consulta **Política de la organización**. Si la política tiene un error, se muestra ahí. Para dudas sobre la política, contacta con tus administradores. Las actualizaciones se describen en [Actualizaciones y diagnóstico](updates.md).
