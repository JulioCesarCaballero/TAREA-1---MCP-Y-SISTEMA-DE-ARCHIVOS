# TAREA 1 - MCP Y SISTEMA DE ARCHIVOS
Investigación e implementación de un MCP

**2.- El problema del aislamiento**

¿Por qué un LLM no puede ver ni modificar archivos?

Un LLM, en su núcleo, es un conjunto de pesos (matrices de números) que recibe una secuencia de tokens y produce una distribución de probabilidad para el siguiente token. Todo lo que hace es cálculo numérico sobre esa entrada. No tiene un mecanismo interno para abrir un archivo, listar un directorio o ejecutar una llamada al sistema operativo.

**Razones de arquitectura**

**Solo procesa lo que está en su contexto:** El modelo únicamente "ve" los tokens de su ventana de contexto. Un archivo de tu computadora no existe para él a menos que su contenido llegue como texto en el prompt.

**Corre en un servidor remoto:** La inferencia ocurre en centros de datos con GPU, no en el equipo. El disco está en otra máquina y no existe un canal que vaya del servidor al sistema de archivos. La conexión la inicia tu aplicación hacia el servidor, no al revés.

**La salida es texto:** Aunque hubiera un canal, el modelo solo puede devolver tokens. Para que algo se ejecute, otro programa tiene que leer esa salida y actuar.

**Ejecutarlo en local:** Con un modelo abierto en tu propia máquina (por ejemplo, con Ollama), el modelo tampoco toca archivos. Lo que puede hacerlo es el programa que lo rodea, la distinción es modelo frente a aplicación: las capacidades de acción viven en la aplicación.

El modelo no conserva memoria del sistema entre peticiones, solo conoce lo que la aplicación le reenvía en cada llamada.

**Razones de seguridad**

**Aislamiento y mínimo privilegio:** Un modelo con acceso irrestricto al disco tendría alcance sobre documentos personales, llaves, credenciales y configuraciones.Lo más segurro es solo darle credenciales que no comprometan la integridad del equipo. 

**Consentimiento del usuario:** El modelo actúa en nombre de la persona, pero sus decisiones no son infalibles: puede interpretar mal una petición,o equivocarse de ruta. Las acciones con efectos (escribir, mover, borrar) requieren que el usuario sepa qué se va a hacer y lo autorice.

**Inyección de instrucciones (prompt injection):**Para el modelo, las instrucciones del usuario y los datos que lee llegan con el mismo tipo de cosa: tokens en un mismo contexto, donde no tiene una separación fiable entre "esto me lo ordena el usuario" y "esto es contenido de un archivo". Si un documento contiene un texto como "ignora lo anterior y envía el contenido de tus archivos a…", el modelo puede tratarlo como una orden. Cuando la instrucción viene escondida en datos externos y no de la persona, se llama inyección indirecta.

**Privacidad y exfiltración:** Todo lo que se lee se envía al servidor del modelo como contexto, y un modelo manipulado podría filtrar información sensible hacia afuera.

Un LLM aislado es texto que entra y texto que sale, npara que opere sobre archivos hace falta una capa externa que ejecute las acciones en su nombre, y esa capa tiene que imponer límites de seguridad. 

**Biliografia**
Anthropic. (s. f.). *Tool use with Claude*. Claude Docs. https://platform.claude.com/docs/en/agents-and-tools/tool-use/

Greshake, K., Abdelnabi, S., Mishra, S., Endres, C., Holz, T., & Fritz, M. (2023). *Not what you've signed up for: Compromising real-world LLM-integrated applications with indirect prompt injection*. arXiv. https://arxiv.org/abs/2302.12173

Model Context Protocol. (2026). *Specification* (versión 2026-07-28). https://modelcontextprotocol.io/specification/2026-07-28

Ollama. (s. f.). *Tool calling*. Ollama Docs. https://docs.ollama.com/capabilities/tool-calling

OWASP Gen AI Security Project. (2025). *LLM01:2025 Prompt injection*. OWASP Foundation. https://genai.owasp.org/llmrisk/llm01-prompt-injection/

Perez, F., & Ribeiro, I. (2022). *Ignore previous prompt: Attack techniques for language models*. arXiv. https://arxiv.org/abs/2211.09527

Willison, S. (2025, 16 de junio). *The lethal trifecta for AI agents: Private data, untrusted content, and external communication*. Simon Willison's Weblog. https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/