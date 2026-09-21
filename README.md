# TAREA 1 - MCP Y SISTEMA DE ARCHIVOS
Investigación e implementación de un MCP


**1. ELECCIÓN DEL CLIENTE**

**Cliente elegido:** Visual Studio Code con GitHub Copilot (modo agente), versión [completar con la versión instalada], sobre [sistema operativo y versión].

### Justificación

1. **Es el entorno donde se desarrolla el proyecto.** Como además de esta práctica se desarrollará un proyecto con la herramienta, se buscó un cliente que también fuera un entorno de desarrollo completo. En VS Code se redactan la documentación y el código, se usa la terminal y se lleva el control de versiones. En modo agente, Copilot planifica el trabajo, determina los archivos y el contexto relevantes, edita el código e invoca herramientas (Microsoft, s. f.-c).

2. **Soporta MCP con una configuración transparente.** Los servidores se definen en un archivo `mcp.json`, ubicado en el espacio de trabajo (`.vscode/mcp.json`) o en el perfil de usuario, con autocompletado para editarlo. El archivo tiene una sección `servers` con los servidores y una sección opcional `inputs` para datos sensibles como claves.

3. **Facilita verificar el servidor y sus herramientas.** VS Code permite iniciar, detener, deshabilitar y consultar los registros de cada servidor, por ejemplo con el comando `MCP: List Servers`. Un servidor deshabilitado no arranca y sus herramientas quedan fuera del chat. Además, el agente cuenta con un selector para configurar qué herramientas puede usar.

4. **Incorpora seguridad.** Si la persona no confía en un servidor, este no se inicia y el chat continúa sin sus herramientas. Antes de una llamada a una herramienta, el agente puede pedir aprobación, y la persona debe revisar la acción y aprobarla si corresponde con la tarea. Esto permite documentar la confirmación humana descrita en la sección 6.

5. **Es accesible.** El plan gratuito de GitHub Copilot admite el modo agente, aunque con límites de uso que pueden cambiar.

6. **Permite ampliar el trabajo.** El mismo editor sirve para escribir, ejecutar y conectar un servidor MCP propio.

### Limitaciones de la elección

- El modo agente requiere una cuenta de GitHub con acceso a Copilot y está sujeto a cuotas de uso.
- Ejecutar el servidor con `npx` requiere tener Node.js instalado.
- El agente tiene herramientas integradas propias; deben distinguirse de las del servidor MCP al documentar las pruebas.

### Actualización de la sección 4.1 (roles en el ejemplo instalado)

| Rol | Quién lo cumple en el ejemplo |
|---|---|
| Host | Visual Studio Code con GitHub Copilot: la aplicación con la que se conversa, que contiene el chat y muestra las solicitudes de aprobación |
| Cliente MCP | Componente interno de VS Code que el host crea para hablar con el servidor `filesystem` |
| Servidor MCP | El paquete `@modelcontextprotocol/server-filesystem`, que el host lanza con `npx` como proceso hijo |
| Modelo | El modelo seleccionado en el chat de Copilot, que decide qué herramienta conviene usar |
| Usuario | Formula la petición y aprueba cada operación antes de que se ejecute |

**2. INSTALACIÓN DEL SERVIDOR DE SISTEMA DE ARCHIVOS**


### Referencias

Anthropic. (s. f.-a). *Connect Claude Code to tools via MCP*. Claude Code Docs. https://code.claude.com/docs/en/mcp

Anthropic. (s. f.-b). *Overview*. Claude Code Docs. https://code.claude.com/docs/en/overview

Cursor. (s. f.-a). *Agent overview*. Cursor Docs. https://cursor.com/docs/agent/overview
