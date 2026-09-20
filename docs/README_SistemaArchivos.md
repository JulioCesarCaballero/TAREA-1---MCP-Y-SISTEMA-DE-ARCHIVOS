# TAREA 1 - MCP Y SISTEMA DE ARCHIVOS
Investigación 


## 5. El servidor de sistema de archivos

## 5.1 "FS" no es parte del protocolo

El servidor de sistema de archivos, que suele abreviarse "FS", no forma parte del protocolo. La especificación de MCP define los mensajes, las primitivas y los transportes; cada servidor concreto es una implementación construida encima de ese protocolo. El servidor de sistema de archivos es uno de los pocos servidores de referencia que mantiene el grupo directivo (*steering group*) del proyecto, alojados en el repositorio `modelcontextprotocol/servers`. Ese repositorio existe para demostrar las funciones de MCP y el uso de los SDK, y advierte que sus servidores son ejemplos educativos y no soluciones listas para producción, por lo que cada desarrollador debe evaluar sus propios requisitos de seguridad. El servidor está implementado en Node.js y se publica en npm como `@modelcontextprotocol/server-filesystem`.

Existen muchos otros servidores posibles. La documentación oficial menciona, además de los de archivos, servidores de bases de datos, GitHub, Slack y calendarios, y en diciembre de 2025 Anthropic reportó más de 10 000 servidores públicos activos .

### 5.2 Herramientas que expone y cómo se delimita su alcance

El servidor ofrece sus capacidades como herramientas (*tools*). Las operaciones que pide la consigna se cubren así (Model Context Protocol, s. f.-a):

| Operación | Herramienta | Qué hace | Tipo |
|---|---|---|---|
| Listar | `list_directory` | Lista el contenido de un directorio con el prefijo `[FILE]` o `[DIR]` | Lectura |
| Listar | `list_directory_with_sizes` | Igual, e incluye tamaños y estadísticas resumidas | Lectura |
| Listar | `directory_tree` | Devuelve la estructura recursiva del directorio en JSON | Lectura |
| Leer | `read_text_file` | Lee un archivo como texto UTF-8, con opción de leer solo las primeras (`head`) o últimas (`tail`) líneas | Lectura |
| Leer | `read_media_file` | Lee imágenes o audio y devuelve datos en base64 con su tipo MIME | Lectura |
| Leer | `read_multiple_files` | Lee varios archivos a la vez; un fallo no detiene la operación completa | Lectura |
| Leer | `get_file_info` | Devuelve metadatos: tamaño, fechas, tipo y permisos | Lectura |
| Escribir | `write_file` | Crea un archivo nuevo o sobrescribe uno existente | Escritura (destructiva) |
| Escribir | `edit_file` | Hace ediciones selectivas por coincidencia de texto; admite `dryRun` para previsualizar los cambios | Escritura (destructiva) |
| Crear | `create_directory` | Crea un directorio (y sus padres si hace falta); no falla si ya existe | Escritura (idempotente) |
| Mover | `move_file` | Mueve o renombra archivos y directorios; falla si el destino ya existe | Escritura (destructiva) |
| Buscar | `search_files` | Busca de forma recursiva archivos o directorios por patrón de estilo *glob* y devuelve las rutas completas | Lectura |
| Alcance | `list_allowed_directories` | Muestra los directorios a los que el servidor tiene permitido acceder | Lectura |

El servidor marca cada herramienta con anotaciones de MCP para que el cliente distinga las de solo lectura de las de escritura, y las que pueden ser destructivas (sobrescribir o mover). Todas declaran `openWorldHint: false`, es decir, que solo acceden al sistema de archivos local dentro de los directorios permitidos y nunca a un entorno abierto o externo. La especificación aclara que las descripciones y anotaciones de una herramienta deben considerarse no confiables, salvo que provengan de un servidor de confianza. La guía oficial de instalación describe la capacidad de búsqueda como localización de archivos por nombre o contenido, mientras que el README del servidor documenta `search_files` como coincidencia por patrones de nombre (Model Context Protocol, s. f.-a).

**Directorios permitidos.** El alcance del servidor se delimita con una lista de directorios permitidos, que puede definirse de dos maneras:

1. **Argumentos de línea de comandos al iniciar el servidor.** Cada ruta que se pasa como argumento pasa a ser un directorio permitido.
2. **Roots del protocolo.** Un cliente que las soporte puede enviar sus roots y estas reemplazan por completo los directorios definidos en el servidor. Si el servidor arranca sin argumentos y el cliente no soporta roots (o las envía vacías), el servidor falla durante la inicialización, porque necesita al menos un directorio permitido para operar.

El servidor restringe todas sus operaciones a esos directorios, y solo dentro de ellos permite actuar; con `list_allowed_directories` se puede consultar cuáles están vigentes. Aunque el README recomienda el segundo método, la revisión 2026-07-28 de la especificación declaró obsoletas las roots y recomienda pasar los directorios mediante parámetros de herramientas, URIs de recursos o configuración del servidor. Por ello, en este trabajo el alcance se delimita con argumentos en la configuración del cliente, que es el método que usa la guía oficial de instalación (Model Context Protocol, 2026c):

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/ruta/absoluta/al/directorio-de-trabajo"
      ]
    }
  }
}
```

Cuando se ejecuta con Docker, los directorios se montan bajo `/projects` y, al añadir la opción `ro` al montaje, el servidor los ve como de solo lectura (Model Context Protocol, s. f.-a).

### 5.3 Por qué existe el límite y qué pasaría sin él

El límite existe porque el servidor se ejecuta con los permisos de la cuenta de la persona usuaria, de modo que puede realizar cualquier operación de archivos que esa persona podría hacer manualmente. Sin la lista de directorios permitidos, el alcance del modelo sería todo lo que la cuenta puede tocar: documentos personales, llaves, credenciales, configuraciones y archivos del sistema. Esto vuelve prácticos los riesgos descritos en el punto 2: el modelo se equivoca de ruta o interpreta mal una petición, y las herramientas de escritura pueden sobrescribir o mover archivos sin que el sistema de archivos ofrezca por sí mismo una forma de deshacerlo. Además, un archivo con instrucciones ocultas podría manipular al modelo para que lea información sensible fuera del directorio de trabajo.

El límite aplica el principio de mínimo privilegio, y con él se reduce el daño posible de un error del modelo, de una inyección de instrucciones o de un abuso de sus herramientas. Conviene subrayar que es una restricción implementada en el servidor. La documentación de MCP recuerda que la seguridad real debe reforzarse en el sistema operativo, con permisos de archivos o aislamiento (Model Context Protocol, 2026f), y la sección 6 muestra un caso real en que ese límite falló por un error de implementación.

## 6. Seguridad

### 6.1 Riesgos concretos

| Riesgo | Cómo ocurre | Ejemplo o fuente |
|---|---|---|
| Inyección de instrucciones a través del contenido de un archivo | El modelo procesa las instrucciones de la persona y los datos que lee como tokens en un mismo contexto, y no distingue de forma fiable entre unas y otros. Un archivo con texto como "ignora lo anterior y…" puede tratarse como una orden | Inyección indirecta (Greshake et al., 2023); riesgo número uno de la lista OWASP de 2025 (OWASP Gen AI Security Project, 2025a) |
| Acceso a rutas fuera del directorio autorizado | El servidor valida mal las rutas y deja pasar una fuera del alcance | Vulnerabilidades del propio servidor de referencia (véase abajo) |
| Escritura o borrado no deseados | Herramientas como `write_file`, `edit_file` y `move_file` están marcadas como destructivas: sobrescriben o eliminan el origen | Anotaciones del servidor (Model Context Protocol, s. f.-a) |
| Exfiltración de datos | Si el mismo host conecta también servidores con acceso a Internet o al correo, el contenido leído puede enviarse afuera | La "trifecta letal": datos privados, contenido no confiable y comunicación externa en un mismo agente (Willison, 2025) |
| Servidores o descripciones no confiables | Las herramientas representan ejecución arbitraria de código y sus descripciones pueden ser engañosas | Principios de seguridad de la especificación (Model Context Protocol, 2026d) |

**Caso real: EscapeRoute.** En 2025 se reportaron dos vulnerabilidades en el servidor de sistema de archivos de referencia que rompían precisamente el límite de directorios permitidos (Cymulate, 2025). La CVE-2025-53110 (CVSS 7.3) se debía a que el servidor validaba las rutas con una comparación de prefijo de texto: un directorio hermano cuyo nombre empezaba igual que el permitido (por ejemplo `allow_dir_secreto` frente a `allow_dir`) pasaba la validación. La CVE-2025-53109 (CVSS 8.4) aprovechaba enlaces simbólicos, que el servidor seguía sin volver a comprobar que el destino real estuviera dentro del directorio permitido, lo que daba acceso de lectura y escritura a todo el sistema de archivos y una vía hacia la ejecución de código. Las versiones anteriores a la 0.6.3 (2025.7.1 en npm) estaban afectadas y el problema se corrigió resolviendo la ruta real y validando el límite después de resolverla. Este caso ilustra que "limitar el alcance" es una defensa valiosa, pero solo tan buena como su implementación.

### 6.2 Mitigaciones

| Mitigación | Qué hace | Cómo se aplica en este trabajo |
|---|---|---|
| Confirmación humana antes de ejecutar | La especificación exige que el host obtenga consentimiento explícito antes de invocar cualquier herramienta (Model Context Protocol, 2026d) | Claude Desktop pide aprobación antes de cada operación sobre el sistema de archivos (Model Context Protocol, 2026c); solo es efectiva si la persona revisa lo que aprueba |
| Alcance limitado a un directorio | Aplica el mínimo privilegio y reduce el daño posible (OWASP Gen AI Security Project, 2025b) | Un directorio de trabajo creado para esta tarea, sin usar la raíz del disco ni la carpeta completa del usuario |
| Permisos de solo lectura | Impide que el modelo modifique lo que no debe | Montaje `ro` en Docker (Model Context Protocol, s. f.-a) y permisos de archivos del sistema operativo |
| Revisión de lo que el servidor expone | Permite detectar herramientas o descripciones inesperadas | Consultar el catálogo con `tools/list`, revisar anotaciones y ejecutar `list_allowed_directories`; tratar las descripciones como no confiables salvo que el servidor lo sea (Model Context Protocol, 2026d) |
| Actualizar y fijar versiones | Evita las vulnerabilidades ya corregidas | Verificar que la versión instalada sea posterior a la 2025.7.1 (Cymulate, 2025) |
| Controles del cliente | Los clientes ofrecen aprobaciones y restricciones propias | Cursor permite aprobar servidores por patrón, restringir qué herramientas se ejecutan automáticamente y aplicar modos de red y de aislamiento a servidores locales (Cursor, s. f.); Claude Code distingue alcances local, de proyecto y de usuario, y admite configuración administrada (Anthropic, s. f.) |
| Aislamiento a nivel de sistema operativo | La seguridad real no debe depender solo de la coordinación cliente-servidor (Model Context Protocol, 2026f) | Ejecutar el servidor en un contenedor o con una cuenta de permisos reducidos |
| No combinar capacidades de riesgo | Evita reunir datos privados, contenido no confiable y salida a Internet en un mismo agente (Willison, 2025) | Conectar solo el servidor de archivos en la prueba y no añadir servidores con acceso a Internet |

## 7. Casos de uso

MCP no se limita a un producto: Anthropic reportó que fue adoptado por ChatGPT, Cursor, Gemini, Microsoft Copilot y Visual Studio Code, entre otros (Anthropic, 2025). A continuación se describen cuatro herramientas concretas que lo implementan. Los modelos que emplean (Claude, Gemini, GPT, Qwen, etc.) son una capa distinta de la herramienta; por ejemplo, Qwen es una familia de modelos y no una plataforma de desarrollo, por lo que solo se cita a través de la herramienta específica que la utilice.

| Herramienta | Tipo | Cómo configura MCP | Para qué lo usa |
|---|---|---|---|
| Claude Code | Herramienta de codificación agéntica de Anthropic para la terminal | Servidores añadidos por comando y configurables en alcance local, de proyecto o de usuario (Anthropic, s. f.) | Conectar el agente con herramientas y fuentes externas al código, como bases de datos, repositorios o gestores de tareas |
| Visual Studio Code con GitHub Copilot | Editor de código con modo agente | Archivo `mcp.json` en el espacio de trabajo o en el perfil de usuario (Microsoft, s. f.-a) | Ofrecer al agente herramientas para operaciones con archivos, bases de datos y APIs externas |
| Cursor | Editor de código con agente de IA | `mcp.json` por proyecto (`.cursor/mcp.json`) o global; soporta transportes stdio, SSE y Streamable HTTP, y primitivas como herramientas, recursos, prompts, roots y elicitation (Cursor, s. f.) | Integrar el agente directamente con las herramientas del desarrollador en lugar de explicar la estructura del proyecto una y otra vez |
| Google Antigravity | Entorno de desarrollo agéntico de Google | Administrador de servidores MCP con archivo de configuración editable (Microsoft, s. f.-b) y una tienda de servidores MCP (Google Cloud, 2025) | Conectar a los agentes con servicios de datos como AlloyDB, BigQuery, Spanner, Cloud SQL y Looker dentro del flujo de desarrollo (Google Cloud, 2025) |

### Cómo editan repositorios completos sin subir archivos manualmente

En un chat web, el modelo solo ve lo que la persona pega o adjunta, lo que obliga al flujo manual de copiar el contenido, esperar la respuesta y pegar el resultado descrito en el punto 2. Las herramientas anteriores son aplicaciones instaladas en la computadora de quien programa, con acceso a la carpeta del proyecto. El modelo sigue sin abrir archivos por sí mismo, pero la aplicación le ofrece herramientas y ejecuta lo que él propone: el modelo pide listar, leer, buscar o modificar, la aplicación realiza la operación sobre el espacio de trabajo y devuelve el resultado, y el modelo repite el ciclo sobre tantos archivos como necesite (Anthropic, s. f.).

En ese esquema, MCP es la forma estándar de añadir capacidades a estas herramientas. Un mismo servidor, como el de sistema de archivos o uno de Git y GitHub, puede conectarse a distintos clientes con una configuración que indica qué servidores se ejecutan y a qué directorios pueden acceder (Model Context Protocol, 2026c). Conviene precisar que parte de la edición la realizan las herramientas integradas del propio host (por ejemplo, Claude Code incluye herramientas propias de edición de archivos, ejecución de comandos y búsqueda), mientras que MCP aporta los servidores adicionales y el mismo protocolo para conectarlos (Builder.io, 2026). Los controles de la sección 6 (aprobación humana y alcance limitado) son los que evitan que esa comodidad se convierta en acceso irrestricto.

## Referencias

Anthropic. (s. f.). *Connect Claude Code to tools via MCP*. Claude Code Docs. https://code.claude.com/docs/en/mcp

Anthropic. (2025, 9 de diciembre). *Donating the Model Context Protocol and establishing the Agentic AI Foundation*. https://anthropic.com/news/donating-the-model-context-protocol-and-establishing-of-the-agentic-ai-foundation

Builder.io. (2026, 25 de marzo). *Claude Code MCP servers: How to connect, configure, and use them*. https://www.builder.io/blog/claude-code-mcp-servers

Cursor. (s. f.). *Model Context Protocol (MCP)*. Cursor Docs. https://cursor.com/docs/mcp

Cymulate. (2025). *EscapeRoute: Breaking the scope of Anthropic's Filesystem MCP Server (CVE-2025-53109 & CVE-2025-53110)*. https://cymulate.com/blog/cve-2025-53109-53110-escaperoute-anthropic/

Google Cloud. (2025, 16 de diciembre). *Connect Google Antigravity IDE to Google's Data Cloud services*. Google Cloud Blog. https://cloud.google.com/blog/products/data-analytics/connect-google-antigravity-ide-to-googles-data-cloud-services

Greshake, K., Abdelnabi, S., Mishra, S., Endres, C., Holz, T., & Fritz, M. (2023). *Not what you've signed up for: Compromising real-world LLM-integrated applications with indirect prompt injection*. arXiv. https://arxiv.org/abs/2302.12173

Microsoft. (s. f.-a). *Add and manage MCP servers in VS Code*. Visual Studio Code Docs. https://code.visualstudio.com/docs/agent-customization/mcp-servers

Microsoft. (s. f.-b). *Get started with the Azure MCP Server in Google Antigravity*. Microsoft Learn. https://learn.microsoft.com/en-ca/azure/developer/azure-mcp-server/get-started/tools/anti-gravity

Model Context Protocol. (s. f.-a). *Filesystem MCP server* [README]. GitHub. https://github.com/modelcontextprotocol/servers/blob/main/src/filesystem/README.md

Model Context Protocol. (s. f.-b). *MCP servers* [Repositorio]. GitHub. https://github.com/modelcontextprotocol/servers

Model Context Protocol. (2026c). *Connect to local MCP servers*. https://modelcontextprotocol.io/docs/2026-07-28/develop/connect-local-servers

Model Context Protocol. (2026d). *Specification* (versión 2026-07-28). https://modelcontextprotocol.io/specification/2026-07-28

Model Context Protocol. (2026f). *Understanding MCP clients*. https://modelcontextprotocol.io/docs/2026-07-28/learn/client-concepts

Model Context Protocol. (2026g). *Understanding MCP servers*. https://modelcontextprotocol.io/docs/2026-07-28/learn/server-concepts

OWASP Gen AI Security Project. (2025a). *LLM01:2025 Prompt injection*. OWASP Foundation. https://genai.owasp.org/llmrisk/llm01-prompt-injection/

OWASP Gen AI Security Project. (2025b). *LLM06:2025 Excessive agency*. OWASP Foundation. https://genai.owasp.org/llmrisk/llm062025-excessive-agency/

Willison, S. (2025, 16 de junio). *The lethal trifecta for AI agents: Private data, untrusted content, and external communication*. Simon Willison's Weblog. https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/