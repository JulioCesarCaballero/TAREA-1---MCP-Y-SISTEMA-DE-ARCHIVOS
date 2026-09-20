# TAREA 1 - MCP Y SISTEMA DE ARCHIVOS
Investigación

**3. MCP FRENTE A UNA API**

**¿Qué es una API?**

Una API (application programming interface), o interfaz de programación de aplicaciones, es un conjunto de reglas o protocolos que permiten que las aplicaciones de software se comuniquen entre sí para intercambiar datos, características y funcionalidades.Las API simplifican y aceleran el desarrollo de software y aplicaciones permitiendo a los desarrolladores integrar datos, servicios y capacidades de otras aplicaciones, en lugar de desarrollarlas desde cero.

Las API permiten compartir solo la información necesaria, manteniendo ocultos otros detalles internos del sistema, lo que ayuda a la seguridad del sistema. Los servidores o dispositivos no tienen que exponer completamente los datos: las API permiten compartir pequeños paquetes de datos, relevantes para la solicitud específica.La documentación de la API es como un manual de instrucciones técnicas que proporciona detalles sobre una API e información para los desarrolladores sobre cómo trabajar con una API y sus servicios. 

**¿Cómo funcionan?**

La arquitectura de las API suele explicarse en términos de cliente y servidor. La aplicación que envía la solicitud se llama cliente, y la que envía la respuesta se llama servidor.Las API pueden funcionar de diferentes maneras , según el momento y el motivo de su creación, la API más popular son las API de REST: define un conjunto de funciones como GET, PUT, DELETE, etc. que los clientes pueden utilizar para acceder a los datos del servidor, los clientes y los servidores intercambian datos mediante el protocolo HTTP.

La principal característica de la API de REST es que no tiene estado, la ausencia de estado significa que los servidores no guardan los datos del cliente entre las solicitudes

**¿Qué beneficios ofrecen las API de REST?**
Las API de REST ofrecen cuatro beneficios principales:

**1. Integración:**Las API se utilizan para integrar nuevas aplicaciones con los sistemas de software existentes. Esto aumenta la velocidad de desarrollo, ya que no hay que escribir cada funcionalidad desde cero. 

**2. Innovación:**Sectores enteros pueden cambiar con la llegada de una nueva aplicación. Las empresas deben responder con rapidez y respaldar la rápida implementación de servicios innovadores.

**3. Ampliación:**Las API presentan una oportunidad única para que las empresas satisfagan las necesidades de sus clientes en diferentes plataformas. Por ejemplo, la API de mapas permite la integración de información de los mapas en sitios web, Android, iOS, etc. Cualquier empresa puede dar un acceso similar a sus bases de datos internas mediante el uso de API gratuitas o de pago.

**4. Facilidad de mantenimiento*La API actúa como una puerta de enlace entre dos sistemas. Cada sistema está obligado a hacer cambios internos para que la API no se vea afectada, de este modo, cualquier cambio futuro que haga una de las partes en el código no afectará a la otra.

**¿Qué es un MCP (model context protocol)?**

El MCP crea una conexión bidireccional estandarizada para las aplicaciones de IA, lo que permite que los LLM se conecten fácilmente con varias fuentes de datos y herramientas. MCP se basa en conceptos existentes como el uso de herramientas y la llamada a funciones, pero los estandariza. Esto reduce la necesidad de conexiones personalizadas para cada nuevo modelo de IA y sistema externo. Donde permite que los LLM usen datos actuales del mundo real, realicen acciones y accedan a funciones especializadas que no se incluyen en su entrenamiento original.

En MCP, un servidor publico da un catálogo de herramientas (*tools*), donde cada una se identifica con un nombre único y se describe con una explicación en lenguaje natural y un esquema JSON de sus parámetros (`inputSchema`). El cliente obtiene el catálogo con una solicitud `tools/list` y ejecuta una herramienta con `tools/call`. Por ejemplo, un servidor puede responder:

{
  "tools": [
    {
      "name": "get_weather",
      "description": "Get current weather information for a location",
      "inputSchema": {
        "type": "object",
        "properties": { "location": { "type": "string" } },
        "required": ["location"]
      }
    }
  ]
}
```
La especificación describe las herramientas como controladas por el modelo: este puede descubrirse e invocarlas a partir de su comprensión del contexto y de lo que pide el usuario. Así, el catálogo se lee en tiempo de ejecución y la decisión de qué herramienta usar la toma el modelo y no el código, conviene precisar que el modelo solo propone la llamada; quien la ejecuta es la aplicación anfitriona, y la especificación establece que el anfitrión debe obtener consentimiento explícito del usuario antes de invocar una herramienta, aunque el protocolo por sí mismo no puede imponerlo.

## 3.3 Tabla comparativa

| Criterio | API (por ejemplo, REST) | MCP |
|---|---|---|
| Quién decide qué se invoca | La persona que desarrolla, al escribir el código | El modelo, en tiempo de ejecución, según lo que pidió el usuario; la aplicación anfitriona ejecuta y solicita consentimiento |
| Cómo se descubren las capacidades | Mediante documentación o una especificación estática (p. ej. OpenAPI) leída en desarrollo | El servidor publica su catálogo con `tools/list` y el cliente lo consulta en ejecución |
| Acoplamiento cliente-servicio | Alto: el cliente codifica los endpoints, parámetros y formatos de cada servicio | Bajo respecto a cada servicio: el cliente implementa el protocolo una vez y se conecta a cualquier servidor |
| Formato de los mensajes | Propio de cada servicio (en REST, HTTP con URLs y métodos) | JSON-RPC 2.0 uniforme (`tools/list`, `tools/call`) |
| Autenticación y consentimiento | Cada servicio define la suya (claves, tokens, OAuth). El consentimiento no forma parte del contrato; lo resuelve la aplicación | Para transportes HTTP, la especificación de autorización se apoya en OAuth. El consentimiento antes de invocar herramientas es un principio de la especificación, implementado por el anfitrión |
| Reutilización entre aplicaciones | Cada integración se programa para un servicio y una aplicación (N × M) | Un servidor sirve a cualquier cliente compatible (N + M) |
| Estado | REST no conserva estado entre peticiones | Desde la revisión 2026-07-28, solicitudes autocontenidas; antes mantenía sesiones |
| Consumidor principal | Código escrito por desarrolladores | Aplicaciones basadas en modelos de lenguaje |

**3.4 MCP no sustituye a las APIs**

MCP no reemplaza a las APIs: es una capa que se coloca por encima, un servidor MCP casi siempre envuelve una API o un recurso ya existente, y traduce cada herramienta publicada en la llamada correspondiente al servicio subyacente. La propia especificación menciona entre sus ejemplos de uso de herramientas la consulta a bases de datos y las llamadas a APIs. La lógica de negocio, los datos y la infraestructura siguen en la API; MCP aporta el descubrimiento en ejecución y una interfaz uniforme que un modelo puede utilizar. Por eso las dos tecnologías son complementarias. El servidor de sistema de archivos de este trabajo lo ilustra: no envuelve una API web, sino un recurso local (el sistema de archivos), y lo hace utilizable por un modelo.

**Conclusión**

Una API es un contrato en el que la persona desarrolladora decide de antemano qué se va a llamar.Por otro lado un MCP añade un catálogo descubrible en tiempo de ejecución que permite al modelo elegir la herramienta, con el anfitrión mediando y el usuario consintiendo. No compiten: el servidor MCP se apoya en la API o recurso que ya existe.


**Bibliografia**
Amazon Web Services. (s. f.). ¿Qué es una interfaz de programación de aplicaciones (API)? https://aws.amazon.com/es/what-is/api/

Anthropic. (2025, 9 de diciembre). Donating the Model Context Protocol and establishing the Agentic AI Foundation. https://www.anthropic.com/news/donating-the-model-context-protocol-and-establishing-of-the-agentic-ai-foundation

Goodwin, M. (2024). ¿Qué es una API? IBM. https://www.ibm.com/mx-es/think/topics/api