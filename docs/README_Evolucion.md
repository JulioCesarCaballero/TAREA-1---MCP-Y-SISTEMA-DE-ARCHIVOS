# TAREA 1 - MCP Y SISTEMA DE ARCHIVOS
Investigación e implementación de un MCP

**1.- Evolución de los modelos**

**Definición de un LM*

Un modelo de lenguaje (LM) es un modelo que estima la probabilidad de una secuencia de tokens; en la práctica, predice el siguiente token a partir de los anteriores. Los primeros LM fueron estadísticos (n-gramas), y luego surgieron los neuronales, las RNN y las LSTM. Un modelo de lenguaje grande (LLM) es un LM que usa la arquitectura Transformer, tiene miles de millones de parámetros y se preentrena con enormes volúmenes de texto. No hay un umbral formal que separe a un LM de un LLM: la diferencia es de escala, y a esa escala aparecen capacidades de generalización que los modelos pequeños no muestran.

Un lenguaje LLM son una categoría de modelos de aprendizaje profundo entrenados con una inmensa cantidades de datos, lo que lo hace capaz de comprender y generar un lenguaje natural y otros tipos de contenido para realizar una amplia gamas de tareas. Los LLM se basan en la arquitectura de redes neuronales llamados transformador que se destaca en en el manejo de de secuencias de palabras y la caputra de patrones de texto.Los LLM funcionan cómo maquinas gigantes de predicción estadistica que predicen rapidamente la siguiente palabra de una secuencia, apreden de patrones de texto y generan  un lenguaje  que sigue esos patrones.

Los LLM representan un gran salto en la que los humanos interactuán con la tecnología porque son el primer sistema de IA que puede manejar el lenguaje humano no estrcuturado a escala, lo que permite una comunicación natural con las maquinas. Mientras que los motores de busqueda de busqueda tradicionales y otros sistemas programados emplean algoritmos  para hacer coincidir palabras clave, los LLM capturan un contexto, matrices y razonamiento más profundo.

**Evolución del lenguaje LM*

Los LLM son la colminación de décadas de progreso en procesamiento de lenguajen natural (PLN) e investigación en machine learning, y su desarrollo es en gran parte del auge en los avances en inteligencia artificial a finales de la década 2010 y 2020. Se centran en la comprensión y generación del lenguaje humano, basados en el estudio de la semántica, que explora la organización, evolución y conexión de las palabras dentro de un idioma. El desarollo de estos lenguajes comenzó con algoritmos más sencillos, pero han evolucionado tanto hasta emplear enfoques de aprendizaje profundo que utilizan una gran cantidad de parámetros.

Su evolución comenzó con el estudio de cómo las palabras se interconectan dentro del marco del lenguaje y cómo transmiten significado, lo que condujo a la creación de modelos fundamentales de aprendizaje automático. Surgió el concepto de modelos de espacio vectorial , donde las palabras se representaban como vectores (incrustaciones de palabras), lo que permitía a las máquinas capturar la similitud semántica. Posteriormente, se produjeron avances como las redes neuronales recurrentes (RNN) y las redes de memoria a largo y corto plazo (LSTM), capaces de procesar secuencias de palabras, algo fundamental para tareas como la traducción automática.

Los modelos de lenguaje natural ( LLM) actuales, como GPT-3.5 y T5 de Google , no solo son expertos en la generación de texto, sino también en la clasificación, el resumen y la respuesta a preguntas. Estos modelos se benefician de una mayor capacidad de procesamiento y conjuntos de datos de entrenamiento más amplios, lo que permite una comprensión del lenguaje más compleja y matizada. Innovaciones como el aprendizaje autosupervisado , el ajuste de instrucciones y el aprovechamiento de la retroalimentación humana durante el entrenamiento han mejorado significativamente el rendimiento y las aplicaciones prácticas.

El punto de quiebre llegó con el Transformer, cuyo mecanismo de atención permitió procesar secuencias en paralelo y aprovechar mucho más cómputo y datos. Con esa arquitectura se pasó a preentrenar modelos con cantidades masivas de texto y a aumentar sus parámetros, y se observó que el rendimiento mejora de forma predecible con la escala (Kaplan et al., 2020). Modelos como GPT-3 (Brown et al., 2020) mostraron que a gran escala aparecen capacidades como resolver tareas a partir de unos pocos ejemplos.

**Modelos con razonamiento éxplicito**

Un LLM convencional genera la respuesta casi de inmediato,token por token. Un modelo con razonamiento éxplicito, genera una serie de pasos intermedios (una cadena de pensamiento) en la que descompone el problema donde hace diferentes pruebas, detecta errores y corrige todo antes de dar una respuesta final. Algunos ejemplos serian los modelos de Open IA, DeepSeek, Claude donde su modelo se basa en el pensamiento extendido, estos modelos "piensan"  produciendo más texto antes de responder. Cada token intermedio funciona como un pequeño espacio de trabajo, donde el modelo puede apoyarse en lo que ya tiene escrito o generado para resolver problemas que en un solo paso serían dificiles de resolver (matematicas, fisica, programación,planeación).

El razonamiento no aparece solo por aumentar el tamaño

Un modelo más grande suele saber más y expresarse mejor, pero el comportamiento de razonar paso a paso, verificarse y corregirse no es una consecuencia automática de tener más parámetros. Proviene de tres fuentes:

1. Técnicas de prompting. Incluir ejemplos con pasos intermedios en el prompt mejora el desempeño en problemas aritméticos y de sentido común, observaron que incluso la instrucción "piensa paso a paso" produce mejoras. Esto no crea una capacidad nueva: activa una que ya existe, y funciona mejor en modelos suficientemente grandes. Por eso el tamaño es una condición de partida, no la explicación completa.

2. Técnicas de entrenamiento. Los modelos de razonamiento actuales no se limitan a "responder cuando se les pide pensar": se entrenan para hacerlo.

**Aprendizaje por refuerzo con recompensas verificables:** se le da al modelo problemas cuya respuesta se puede comprobar automáticamente (un resultado matemático, un código que pasa pruebas) y se refuerzan las cadenas de pensamiento que llegan a respuestas correctas. DeepSeek-AI (2025) reportó que con este tipo de entrenamiento surgieron comportamientos como alargar el razonamiento, reflexionar y verificar sus propios pasos.

**Supervisión de procesos:** En lugar de premiar solo la respuesta final, se evalúa cada paso del razonamiento.

**Destilación:** un modelo pequeño puede aprender a razonar imitando las trazas de un modelo grande. Esto refuerza la idea de que el razonamiento depende de cómo se entrena, no solo de cuántos parámetros se tienen.

3. Cómputo adicional en el momento de la inferencia (test-time compute). Al responder, el modelo gasta más cómputo de dos formas:

generando cadenas de pensamiento más largas; generando varias soluciones candidatas y eligiendo la mejor, por votación de mayoría o con un verificador que califica las respuestas.

**Bibliografias**
Stryker, C. (2021, 6 de octubre). ¿Qué son los LLM (grandes modelos de lenguaje)? IBM
https://www.ibm.com/mx-es/think/topics/large-language-models

Sofia. (2024, 15 de marzo). A brief history of large language models (LLM). Parsio.
https://parsio.io/blog/a-brief-history-of-llm/

IBM. (s. f.). What is a reasoning model? IBM Think.
https://www.ibm.com/think/topics/reasoning-model

AlphaXiv. (s. f.). Modelos de razonamiento pueden ser eficaces sin pensar.
https://www.alphaxiv.org/es/abs/2504.09858

Aprender BIG DATA. (s. f.). Modelos de razonamiento y por qué importan
https://aprenderbigdata.com/modelos-de-razonamiento/