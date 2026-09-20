# TAREA 1 - MCP Y SISTEMA DE ARCHIVOS
Investigación e implementación de un MCP

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



**Bibliografia**
https://www.ibm.com/mx-es/think/topics/api
https://aws.amazon.com/es/what-is/api/