::: titlepage
TRABAJO FINAL DE CARRERA\
**Automatización de Gestión Empresarial\
con n8n: Asistente Inteligente para\
Taller Mecánico vía Telegram**\
Línea de Investigación: Workflows Aplicados a la Gestión Empresarial y Marketing\
Tecnicatura Universitaria en Programación\
Universidad Tecnológica Nacional\
Mendoza, 2026\
Profesor: Cortez, Alberto Alejandro\
Estudiantes: Palmero, Manuel; Rojas, Uriel\
:::

# Resumen {#resumen .unnumbered}

El presente Trabajo Final de Carrera desarrolla, implementa y valida un sistema de automatización para la gestión operativa de un taller mecánico de pequeña escala, utilizando la plataforma de orquestación de workflows de código abierto n8n integrada con la aplicación de mensajería Telegram como canal conversacional. La arquitectura se fundamenta en la concepción de dos agentes de inteligencia artificial diferenciados bajo el principio de privilegio mínimo: un Agente de Servicio Técnico, orientado a la atención al cliente para el registro de turnos y consulta de inventario, y un Agente Administrativo, restringido al personal interno para la gestión completa de la base de datos. Como motor de razonamiento se empleó el modelo Mistral Large, integrado mediante el framework LangChain en su implementación nativa de n8n. La persistencia de datos se gestiona en Google Sheets, estructurada en tres entidades (Stock, Clientes y Turnos) vinculadas relacionalmente mediante claves de identificación. Metodológicamente se adoptó un diseño de estudio de caso aplicado a un taller mecánico simulado caracterizado mediante entrevista semiestructurada con un mecánico en actividad. El sistema fue sometido a un protocolo de validación funcional compuesto por pruebas unitarias, de integración, de enrutamiento y de coherencia conversacional, cuyos resultados se documentan con capturas de pantalla y métricas de éxito. Los hallazgos confirman la viabilidad técnica de la solución en su condición de prototipo de investigación validado en condiciones controladas: el sistema completó satisfactoriamente el flujo de registro de turnos, las operaciones CRUD sobre el inventario y la discriminación de roles. El mecanismo de autenticación administrativa se implementa mediante la validación del identificador de usuario de Telegram (User ID) contra una lista de identificadores autorizados cargada manualmente en el workflow, y el resultado de dicha validación se almacena en una variable booleana denominada `admin` que determina el enrutamiento hacia el panel administrativo. Se identifican como limitaciones la ausencia de persistencia de memoria conversacional entre sesiones y las posibles condiciones de carrera bajo concurrencia. El trabajo aporta un modelo replicable de adopción de tecnologías low-code para PyMEs del sector automotriz argentino y traza líneas futuras orientadas a reforzar la escalabilidad, incorporar persistencia conversacional e integrar el sistema con calendarios externos. Su transformación en un sistema apto para despliegue productivo requiere, adicionalmente, un análisis de conformidad con la Ley 25.326 de Protección de Datos Personales, dado que los mensajes procesados contienen información personal de clientes.

**Palabras clave:** automatización de procesos de negocio, low-code, agentes de inteligencia artificial, LangChain, PyMEs, Telegram Bot.

# Introducción

## Contexto y Justificación

La transformación digital se ha consolidado como un eje estratégico para la competitividad empresarial contemporánea, y la automatización de procesos de negocio (Business Process Automation, BPA) ocupa dentro de ella un lugar central (van der Aalst et al., 2018). Para las pequeñas y medianas empresas (PyMEs), la adopción tecnológica enfrenta restricciones presupuestarias y de capital humano especializado que tornan particularmente atractivas las soluciones de código abierto y de bajo costo de implementación (Eller et al., 2020; Sahay et al., 2020).

En el sector automotriz, los talleres mecánicos de pequeña escala presentan procesos operativos altamente susceptibles de automatización. La gestión de turnos, el control de inventario de repuestos y la comunicación con clientes suelen sostenerse mediante registros manuales o herramientas ofimáticas básicas, lo que genera ineficiencias documentadas en estudios sobre digitalización de PyMEs de servicios (Eller et al., 2020).

La plataforma n8n se inscribe en la categoría de herramientas low-code de orquestación de workflows, las cuales han sido estudiadas como facilitadoras de la transformación digital en organizaciones con bajo grado de madurez tecnológica (Sahay et al., 2020). Su carácter de código abierto y la posibilidad de despliegue autoalojado proveen control sobre la infraestructura y la privacidad de los datos, atributos relevantes para entornos empresariales con datos sensibles de clientes.

La selección de Telegram como interfaz se fundamenta en la robustez y gratuidad de su API de bots, así como en su penetración en el mercado argentino. Por su parte, la incorporación de un modelo de lenguaje de gran escala (LLM) como Mistral permite ofrecer una interfaz conversacional en lenguaje natural, eliminando la curva de aprendizaje asociada a aplicaciones tradicionales y alineándose con la tendencia de adopción de agentes conversacionales en servicios al cliente (Adamopoulou & Moussiades, 2020).

## Problemática Identificada

La gestión operativa en talleres mecánicos de pequeña escala presenta deficiencias estructurales recurrentes. Los principales problemas detectados se sintetizan a continuación.

La gestión de turnos resulta ineficiente cuando se sostiene mediante agendas físicas, planillas de cálculo simples o llamadas telefónicas, lo que produce pérdida de oportunidades fuera del horario comercial, dificultades para el seguimiento centralizado de la disponibilidad y solapamientos involuntarios. El control de inventario rara vez se actualiza en tiempo real, lo que obliga al personal a realizar verificaciones manuales y favorece la aparición de faltantes críticos al momento de una reparación. La gestión de clientes adolece de dispersión de la información y duplicación de registros, dificultando la consulta de historiales y la personalización de la atención. Finalmente, la carga operativa se concentra desproporcionadamente sobre el mecánico o encargado, quien debe atender simultáneamente el teléfono, agendar turnos, consultar stock y gestionar facturación, lo que desvía su atención de las tareas técnicas y reduce la productividad general del taller.

La solución propuesta busca mitigar estos problemas mediante la centralización de datos, la automatización basada en reglas explícitas y la incorporación de capacidades conversacionales de inteligencia artificial para gestionar la interacción inicial con el cliente.

### Hipótesis de Trabajo

Dado el carácter aplicado y orientado al desarrollo tecnológico del presente trabajo, no se formula una hipótesis verificable en el sentido estricto de la investigación empírica cuantitativa. En su lugar, se plantean los siguientes supuestos de trabajo que guían las decisiones de diseño e implementación.

Primero, la integración de n8n con un modelo de lenguaje de gran escala mediante el framework LangChain y su exposición a través de la API de Telegram Bot permite construir un sistema de automatización conversacional técnicamente viable para la gestión operativa de un taller mecánico PyME, utilizando exclusivamente herramientas de código abierto o gratuitas.

Segundo, la separación de privilegios entre un agente público y un agente administrativo, combinada con la validación del identificador de usuario de Telegram contra una lista de identificadores autorizados, constituye un mecanismo de control de acceso funcional para el contexto de un prototipo de investigación. La validación funcional del sistema mediante pruebas unitarias, de integración y de enrutamiento constituye el mecanismo de verificación de ambos supuestos.

## Objetivos

### Objetivo General

Desarrollar y validar un workflow operativo dentro de la plataforma n8n que automatice la gestión de turnos, la administración de la base de clientes y el control de stock de repuestos en un taller mecánico, accesible mediante un bot de Telegram con dos niveles de acceso diferenciados.

### Objetivos Específicos

1.  Diseñar e implementar un agente de inteligencia artificial de atención al cliente capaz de interpretar peticiones en lenguaje natural para registrar turnos, incorporar nuevos clientes y consultar la disponibilidad de repuestos.

2.  Implementar un agente de gestión administrativa con acceso restringido que permita consultar el inventario completo, actualizar el stock y acceder a la lista consolidada de turnos y clientes.

3.  Establecer la integración entre n8n y Google Sheets como repositorio centralizado de datos, garantizando la integridad de la información mediante autenticación OAuth2.

4.  Configurar el modelo Mistral Large como motor de razonamiento de ambos agentes mediante los nodos nativos de n8n para LangChain.

5.  Desarrollar una lógica de enrutamiento que diferencie los canales de acceso público y privado mediante la validación del identificador de usuario de Telegram (User ID) contra una lista de identificadores de administradores autorizados, almacenando el resultado en una variable booleana denominada `admin` que determina el flujo de ejecución a través de un nodo Switch.

6.  Validar el sistema mediante un protocolo de pruebas funcionales unitarias, de integración y de seguridad, documentando los resultados con evidencia empírica.

# Marco Teórico

## Automatización de Procesos de Negocio (BPA/RPA)

La automatización de procesos de negocio constituye un campo consolidado de la disciplina de los sistemas de información, definido como la aplicación de tecnología para ejecutar tareas recurrentes con mínima intervención humana (van der Aalst et al., 2018). Dentro de este campo, la automatización robótica de procesos (Robotic Process Automation, RPA) ha emergido como una subdisciplina enfocada en la imitación de interacciones humanas con sistemas de software mediante agentes automatizados (Syed et al., 2020).

La distinción entre BPA y RPA es relevante para situar el presente trabajo. Mientras la RPA opera tradicionalmente sobre interfaces gráficas de aplicaciones existentes, la BPA orquesta servicios mediante interfaces de programación de aplicaciones (API), lo que permite mayor robustez, mantenibilidad y escalabilidad (Hofmann et al., 2020). El sistema desarrollado en este trabajo se inscribe plenamente en el paradigma de la BPA orientada a APIs.

Estudios recientes sobre adopción de BPA en PyMEs señalan que las principales barreras no son tecnológicas sino organizacionales, vinculadas a la falta de capital humano especializado y a la dificultad para identificar procesos automatizables (Eller et al., 2020). En este contexto, las plataformas low-code se han posicionado como mediadoras entre la complejidad técnica y la realidad operativa de las pequeñas organizaciones (Sahay et al., 2020).

## Plataformas Low-Code y n8n

Las plataformas low-code permiten construir aplicaciones y workflows mediante interfaces visuales, reduciendo significativamente la cantidad de código manual requerido (Sahay et al., 2020). Dentro del ecosistema de orquestación de workflows, las alternativas de mayor difusión incluyen Zapier, Make (anteriormente Integromat), Microsoft Power Automate, Apache Airflow y n8n.

La selección de n8n para este trabajo se fundamenta en una comparación deliberada entre alternativas. Zapier y Make ofrecen modelos comerciales basados en suscripción con limitaciones por volumen de ejecuciones, lo que resulta inapropiado para una PyME con presupuesto restringido. Apache Airflow, si bien es de código abierto, está orientado a pipelines de datos y requiere conocimientos avanzados de programación en Python. Power Automate presenta integración profunda con el ecosistema Microsoft pero costos asociados a las licencias. n8n, en cambio, ofrece un modelo de código abierto con licencia fair-code que permite el despliegue autoalojado sin costos por ejecución, soporte nativo para integración con LLM mediante LangChain y una arquitectura modular basada en nodos compatible con el paradigma visual de las plataformas comerciales.

La arquitectura de n8n se sustenta en el concepto de nodo, entendido como unidad funcional que ejecuta una acción específica (envío de un mensaje, consulta a una API, escritura en una base de datos) o que actúa como disparador (trigger) iniciando la ejecución del workflow ante un evento externo. Los workflows se construyen conectando nodos en un canvas visual, donde los datos fluyen secuencialmente y cada nodo opera sobre la salida de su predecesor. Este paradigma admite estructuras de control complejas como bucles, ramificaciones condicionales y orquestación paralela.

## Agentes Conversacionales basados en LLM

Los agentes conversacionales basados en modelos de lenguaje de gran escala constituyen una evolución de los chatbots tradicionales basados en reglas o intención (Adamopoulou & Moussiades, 2020). A diferencia de estos, los agentes basados en LLM no requieren la enumeración exhaustiva de patrones de entrada, sino que utilizan la capacidad generalizadora del modelo para interpretar el lenguaje natural y razonar sobre las acciones a ejecutar.

El paradigma de agente con herramientas (Tool-Using Agent), formalizado por Yao et al. (2023) en el framework ReAct, propone un ciclo iterativo en el cual el LLM alterna entre razonamiento y acción. El modelo recibe el mensaje del usuario, razona sobre la intención, selecciona una herramienta del conjunto disponible, ejecuta la herramienta, observa el resultado y decide si requiere invocaciones adicionales o si puede generar la respuesta final. Este ciclo permite descomponer tareas complejas en operaciones discretas verificables.

## Integración del Framework LangChain

LangChain es un framework de código abierto diseñado para facilitar el desarrollo de aplicaciones impulsadas por modelos de lenguaje, formalizando abstracciones para el manejo de prompts, memoria, herramientas y agentes (Chase, 2022). Su integración nativa en n8n a partir de la versión 1.0 permite construir agentes Tool-Using sin necesidad de programación directa, mediante nodos que encapsulan los componentes del framework.

En la arquitectura del presente proyecto, las herramientas (tools) son nodos de Google Sheets configurados para realizar operaciones específicas de lectura (Get Many, Get by Filter) o escritura (Append Row, Update Row). El agente no accede directamente a la base de datos; conoce únicamente las descripciones textuales de las herramientas y delega al razonamiento del LLM la selección dinámica de la herramienta adecuada para cada solicitud. Esta separación permite implementar el principio de privilegio mínimo asignando a cada agente solo el subconjunto de herramientas necesario para su rol.

Un componente técnico central en la integración de LangChain con Google Sheets dentro de n8n es la expresión `$fromAI()`. Esta expresión actúa como marcador dinámico en la configuración de los nodos herramienta: cuando el agente LLM decide invocar una herramienta, n8n reemplaza en tiempo de ejecución cada `$fromAI()` por el valor que el modelo de lenguaje determina apropiado para ese parámetro en función del contexto conversacional. Esto permite, por ejemplo, que el agente extraiga el DNI del cliente directamente del texto del mensaje y lo inyecte como filtro en la consulta a Google Sheets sin que el desarrollador deba programar dicha extracción de manera explícita.

## Modelos de Lenguaje de Gran Escala: Mistral AI

Mistral AI es una empresa europea fundada en 2023 dedicada al desarrollo de modelos de lenguaje de gran escala con énfasis en modelos abiertos (Jiang et al., 2023). El modelo Mistral 7B, presentado en su artículo fundacional, demostró rendimiento superior a modelos propietarios de mayor tamaño en tareas de razonamiento y generación de texto, validando su elección para entornos con requerimientos de baja latencia.

En el workflow desarrollado, el modelo se integra mediante el nodo Mistral Cloud Chat Model configurado con la variante `mistral-large-latest`, que recibe el prompt de sistema, el historial de conversación y las descripciones de las herramientas disponibles, devolviendo la decisión sobre la acción a ejecutar. La arquitectura instancia dos nodos separados, uno por agente, lo que permite parametrización independiente de la temperatura y eventual diferenciación del modelo subyacente en futuras iteraciones.

## Persistencia de Datos en Hojas de Cálculo Conectadas

La elección de Google Sheets como sistema de persistencia se fundamenta en consideraciones pragmáticas alineadas con el contexto de adopción tecnológica en PyMEs. Investigaciones sobre digitalización de pequeñas empresas señalan que la familiaridad del usuario con la herramienta es un predictor más significativo de adopción exitosa que la sofisticación técnica de la solución (Eller et al., 2020). Las hojas de cálculo conectadas a APIs proveen una interfaz familiar para el administrador, permiten edición colaborativa en tiempo real y eliminan la curva de aprendizaje asociada a consolas de bases de datos relacionales.

Las limitaciones de este enfoque son reconocidas en la sección [4.7](#sec:limitaciones){reference-type="ref" reference="sec:limitaciones"} y abordadas en las líneas de trabajo futuro: ausencia de transacciones ACID, riesgo de condiciones de carrera bajo concurrencia y techo de escalabilidad que impone la API de Google Sheets.

## Telegram Bot API como Interfaz Conversacional

Telegram Bot API es la interfaz de programación que permite el desarrollo de bots automatizados en la plataforma de mensajería Telegram. Su elección frente a alternativas como WhatsApp Business API se fundamenta en tres criterios. Primero, gratuidad: la API de Telegram no impone costos por mensaje ni requiere suscripción comercial. Segundo, robustez técnica: ofrece soporte para webhooks, mensajes interactivos, archivos adjuntos y estados de conversación. Tercero, penetración regional: de acuerdo con datos de Statista (2025) sobre distribución de aplicaciones de mensajería en Argentina, Telegram figura entre las plataformas de mayor crecimiento en usuarios activos mensuales durante el período 2021--2025.

La comunicación entre el bot y n8n se implementa mediante webhooks. El nodo Telegram Trigger expone un endpoint HTTPS al que la API de Telegram envía solicitudes POST cada vez que un usuario interactúa con el bot. La respuesta procesada se envía mediante el nodo Telegram Send Message.

## Antecedentes y Estado del Arte

### Metodología de búsqueda

La revisión de antecedentes se realizó mediante búsqueda en las bases Google Scholar, Semantic Scholar y Redalyc, utilizando los descriptores *chatbot small business automation*, *LLM business process automation*, *n8n workflow automation*, *conversational agent appointment scheduling*, *PyME automatización taller mecánico* y sus combinaciones en inglés y español. Se priorizaron publicaciones de los últimos cinco años (2020--2025).

### Automatización conversacional y agentes con uso de herramientas

En el campo de la automatización conversacional orientada a servicios, Adamopoulou y Moussiades (2020) ofrecen una revisión comprehensiva del estado de los chatbots, distinguiendo los sistemas basados en reglas de los basados en modelos de lenguaje y señalando que la adopción en PyMEs de servicios enfrenta principalmente barreras de costo y complejidad de configuración, no de disponibilidad tecnológica. En el ámbito específico de agentes con capacidad de acción sobre sistemas externos, el framework ReAct de Yao et al. (2023) representa el antecedente académico más directo al paradigma Tool-Using Agent implementado en este trabajo.

### Digitalización de PyMEs en el sector automotriz y de servicios

Respecto de la digitalización de PyMEs en el sector automotriz y de servicios, Eller et al. (2020) documentan que las principales barreras para la adopción de herramientas digitales en pequeñas empresas no son de naturaleza tecnológica sino organizacional, destacando la escasez de capital humano especializado. No se encontraron estudios académicos que documenten implementaciones específicas de automatización conversacional en talleres mecánicos de pequeña escala en Argentina o Latinoamérica, lo que constituye un vacío que el presente trabajo busca contribuir a cubrir.

### Brecha identificada y posicionamiento del trabajo

En cuanto a la combinación específica de n8n con modelos de lenguaje de gran escala para automatización empresarial, la documentación académica revisada es efectivamente limitada: los trabajos disponibles sobre plataformas low-code (Sahay et al., 2020) no contemplan integración con LLM, mientras que los trabajos sobre agentes LLM en producción no abordan plataformas de orquestación visual.

No se identificaron trabajos previos que implementen el patrón específico de dos agentes LLM con separación de privilegios sobre Google Sheets como capa de persistencia, accesibles mediante un bot de Telegram, en el contexto de una PyME de servicios. En consecuencia, el aporte de este trabajo, acotado a su naturaleza de tecnicatura, reside en documentar un caso de implementación replicable que integra estas tecnologías bajo restricciones reales de presupuesto y capital humano.

# Marco Metodológico

## Tipo y Enfoque de Investigación

La presente investigación se clasifica como aplicada, según la taxonomía de Hernández-Sampieri y Mendoza (2018), por buscar la resolución de un problema concreto mediante el desarrollo de una solución tecnológica. En cuanto a su alcance, combina componentes descriptivos (caracterización del caso de estudio y descripción de la arquitectura) con componentes evaluativos (validación funcional del sistema mediante pruebas).

En cuanto al enfoque, el trabajo se inscribe en la modalidad de investigación-desarrollo (I+D aplicada), combinando una fase de diagnóstico cualitativo del caso de estudio con una fase de construcción y validación técnica del sistema. La validación funcional incorpora métricas cuantitativas básicas sin pretender generalización estadística.

## Diseño de Investigación: Estudio de Caso

Se adopta el diseño de estudio de caso instrumental único (Stake, 1995), entendido como el examen detallado de una situación particular con el propósito de iluminar un fenómeno más amplio. El caso seleccionado es un taller mecánico de pequeña escala ubicado en el Gran Mendoza, cuyas características operativas se describen en la sección [3.3](#sec:caracterizacion){reference-type="ref" reference="sec:caracterizacion"}.

## Caracterización del Caso de Estudio {#sec:caracterizacion}

El caso seleccionado corresponde a un taller mecánico independiente con dos años de antigüedad, dedicado a reparaciones generales y mantenimiento preventivo de vehículos livianos. La estructura organizativa se compone de un mecánico propietario que cumple simultáneamente funciones técnicas y administrativas, y un asistente con dedicación parcial.

Mediante una entrevista semiestructurada de cuarenta y cinco minutos de duración, registrada con consentimiento del entrevistado y desgrabada para su análisis (el guion se incluye en el Anexo [\[anexo:entrevista\]](#anexo:entrevista){reference-type="ref" reference="anexo:entrevista"}), se relevaron los siguientes datos operativos. El taller atiende un promedio de doce a quince vehículos semanales. La gestión de turnos se realiza mediante una agenda física combinada con mensajería de WhatsApp, sin sistematización digital. El control de inventario se lleva en una planilla de Excel actualizada de manera irregular, y el entrevistado reportó al menos tres episodios mensuales de faltante crítico de repuestos durante reparaciones en curso. La base de clientes registra aproximadamente trescientos vehículos, sin centralización digital del historial de servicios.

## Instrumentos de Relevamiento

Para la caracterización del caso se utilizaron tres instrumentos complementarios. Primero, una entrevista semiestructurada al propietario del taller, organizada en cuatro bloques temáticos: gestión de turnos, control de inventario, comunicación con clientes y disposición a la adopción tecnológica. Segundo, observación no participante de dos jornadas laborales completas, registrando los puntos de fricción operativa en una grilla previamente diseñada. Tercero, revisión documental de la planilla de inventario y de la agenda de turnos del último mes.

## Procedimiento de Desarrollo Iterativo

El desarrollo del sistema siguió un modelo iterativo e incremental, organizado en cinco iteraciones documentadas:

1.  **Iteración 1**: Flujo mínimo viable con trigger de Telegram, agente único de servicio al cliente y herramienta de registro de turnos.

2.  **Iteración 2**: Verificación de cliente existente por DNI y registro condicional. Corrección de errores en inyección dinámica mediante `$fromAI()`.

3.  **Iteración 3**: Segundo agente y lógica de enrutamiento mediante nodo Switch.

4.  **Iteración 4**: Mecanismo de confirmación explícita para modificación de stock y subworkflow de verificación de disponibilidad.

5.  **Iteración 5**: Refinamiento de prompts, ajuste de formato de respuesta para Telegram y pruebas integrales.

## Técnicas de Análisis y Validación

La validación funcional del sistema se diseñó como un protocolo de pruebas estructurado en tres niveles: pruebas unitarias (verificación aislada de cada herramienta), pruebas de integración (flujos conversacionales completos) y pruebas de enrutamiento y seguridad (discriminación de roles). Los resultados completos se presentan en el Capítulo [6](#cap:resultados){reference-type="ref" reference="cap:resultados"}.

# Diseño de la Solución

## Arquitectura General del Workflow

El workflow constituye el núcleo operativo del sistema y actúa como motor de orquestación que coordina la recepción de mensajes, el razonamiento de los agentes y la interacción con la base de datos. La arquitectura es modular y se compone de cinco capas funcionales:

1.  **Capa de entrada**: Nodo Telegram Trigger, configurado para escuchar eventos de tipo `message`.

2.  **Capa de validación de acceso**: Nodo de procesamiento (Code) que extrae el campo `from.id`, lo compara contra una lista de administradores autorizados y establece la variable booleana `admin`.

3.  **Capa de enrutamiento**: Nodo Switch que evalúa la variable `admin` y el contenido del mensaje para dirigir a una de tres rutas.

4.  **Capa de agentes**: Dos ramas independientes, cada una con su nodo AI Agent (tipo Tools Agent), su instancia dedicada de Mistral Cloud Chat Model y su memoria Window Buffer.

5.  **Capa de datos**: Nodos Google Sheets Tool distribuidos entre los agentes según el principio de privilegio mínimo, más un subworkflow de consulta de disponibilidad.

## Lógica de Enrutamiento {#sec:enrutamiento}

El workflow incorpora un nodo de procesamiento inmediatamente posterior al Telegram Trigger que extrae el campo `from.id` del objeto de mensaje, correspondiente al identificador numérico único del usuario emisor en la plataforma Telegram. Dicho identificador se compara contra una lista de identificadores de administradores autorizados, cargada manualmente como parámetro de configuración del workflow. El resultado de esta comparación se almacena en una variable booleana denominada `admin`, cuyo valor es verdadero si el User ID del emisor figura en la lista de autorizados, y falso en caso contrario.

El nodo Switch evalúa las siguientes condiciones en orden de prioridad:

::: center
  Condición                                     Destino                   Descripción
  --------------------------------------------- ------------------------- -------------------------
  `admin` es verdadera                          Agente Administrativo     Acceso directo al panel
  Texto igual a `"/start"` y `admin` es falsa   Mensaje de Bienvenida     Mensaje estático
  Condición por defecto                         Agente Servicio Técnico   Mensajes de clientes
:::

## Agente de Servicio al Cliente (Público) {#sec:agente-cliente}

El Agente de Servicio Técnico está diseñado para interactuar con el cliente bajo un protocolo conversacional estricto definido en su prompt de sistema. Su flujo de registro de turnos sigue una secuencia que prioriza la verificación de disponibilidad antes de solicitar datos personales: consulta de disponibilidad, verificación del cliente por DNI, registro condicional del cliente nuevo, captura de detalles del servicio y confirmación final con invocación de la herramienta de registro.

El agente público cuenta con memoria Window Buffer con ventana de 30 mensajes y clave de sesión basada en el identificador de chat de Telegram, lo que permite mantener el contexto durante una conversación activa.

Las herramientas asignadas al agente público son las siguientes:

-   `Consultar_Stock_Publico1`: Get Many sobre la hoja Stock, sin columnas de costo.

-   `Obtener Clientes`: Get Many sobre la hoja Clientes (uso exclusivo para cálculo de IDs).

-   `Obtener El ID por DNI`: Get by Filter sobre Clientes filtrando por DNI.

-   `Herramienta_Verificar_DNI1`: Get by Filter sobre Clientes filtrando por DNI (verificación redundante).

-   `Registrar Cliente1`: Append Row sobre la hoja Clientes.

-   `Obtener Turnos`: Get Many sobre la hoja Turnos (uso exclusivo para cálculo de IDs).

-   `Obtener Turno Por id`: Get by Filter sobre Turnos filtrando por ID_Cliente.

-   `Agendar_Turno`: Append Row sobre la hoja Turnos.

-   `Herramienta_Cancelar_Turno1`: Update Row sobre Turnos (cambio de estado a Cancelado).

-   `Modificar_turno`: Update Row sobre Turnos (reprogramación de turno).

-   `Consultar Disponibilidad Tool`: Workflow Tool que invoca un subworkflow dedicado para verificar disponibilidad de turnos.

### Subworkflow de Consulta de Disponibilidad

El sistema incorpora un subworkflow independiente denominado `SubWorkflow_Consultar_Disponibilidad`, invocado como herramienta por el agente público. Este subworkflow recibe dos parámetros (`fecha` y `horario`), consulta la hoja Turnos de Google Sheets y determina si el horario solicitado está disponible. La lógica de verificación excluye los turnos con estado `Cancelado` para evitar falsos positivos. Retorna `LIBRE` o `OCUPADO` según el resultado.

## Agente Administrativo (Privado)

El Agente Administrativo está protegido por el mecanismo de validación de User ID descrito en la sección [4.2](#sec:enrutamiento){reference-type="ref" reference="sec:enrutamiento"}. Solo los usuarios cuyo identificador de Telegram figure en la lista de administradores autorizados son enrutados hacia este agente. El agente administrativo dispone de un conjunto de herramientas que incluyen capacidades de modificación de datos:

-   `Obtener Repuestos`: Get Many sobre la hoja Stock, con acceso completo a todas las columnas.

-   `Obtener Clientes`: Get Many sobre la hoja Clientes (lista completa).

-   `Obtener Turnos`: Get Many sobre la hoja Turnos (todos los registros).

-   `Actualizar Stock1`: Update Row sobre la hoja Stock.

## Mecanismo de Seguridad y Confirmación

El sistema implementa dos niveles de seguridad diferenciados. El primer nivel opera sobre el enrutamiento: antes de que cualquier mensaje alcance al Agente Administrativo, el workflow extrae el User ID del emisor desde el campo `from.id` del objeto de actualización de Telegram y lo compara contra una lista de identificadores de administradores autorizados. Esta comparación se realiza mediante un nodo de procesamiento previo al Switch, que establece la variable booleana `admin` como verdadera únicamente si el identificador figura en la lista. El nodo Switch evalúa esta variable y deriva el flujo hacia el Agente Administrativo solo cuando `admin` es verdadera. Este mecanismo impide que el contenido del mensaje pueda utilizarse para manipular el enrutamiento.

El segundo nivel opera sobre las operaciones de modificación de datos: el prompt de sistema del Agente Administrativo incluye una instrucción de confirmación explícita previa a la ejecución de `Actualizar Stock1`. El agente debe consultar al usuario antes de invocar la herramienta y ejecutarla únicamente ante una respuesta afirmativa explícita.

## Estructura de la Base de Datos {#sec:schema}

El documento `Taller mecánico` en Google Sheets se estructura en tres hojas vinculadas relacionalmente.

### Hoja Stock {#hoja-stock .unnumbered}

  Columna           Tipo       Descripción
  ----------------- ---------- -----------------------------------
  Repuesto          Texto      Nombre del producto
  Código            Texto      Identificador único del repuesto
  Stock Actual      Numérico   Cantidad disponible en inventario
  Stock Mínimo      Numérico   Umbral mínimo para reposición
  Precio Unitario   Numérico   Precio de venta al público
  Categoría         Texto      Clasificación del repuesto

### Hoja Clientes {#hoja-clientes .unnumbered}

  Columna              Tipo       Descripción
  -------------------- ---------- -----------------------------------------------------
  ID_Cliente           Numérico   Identificador único del cliente
  Nombre y Apellido    Texto      Nombre completo
  DNI                  Texto      Documento Nacional de Identidad (clave de búsqueda)
  Dirección            Texto      Dirección del cliente
  Número De Teléfono   Texto      Teléfono de contacto

### Hoja Turnos {#hoja-turnos .unnumbered}

  Columna           Tipo       Descripción
  ----------------- ---------- -----------------------------------------------------
  ID_Turno          Numérico   Identificador único del turno
  ID_Cliente        Numérico   Clave foránea hacia Clientes
  Auto              Texto      Marca, modelo y año del vehículo
  Motivo            Texto      Motivo de la visita / diagnóstico
  Fecha del Turno   Texto      Fecha programada (formato d/MM/yyyy)
  Estado            Texto      Estado del turno (Pendiente, Cancelado, Completado)
  Horario           Texto      Horario programado (formato H:mm:ss)

La integración con n8n se realiza mediante autenticación OAuth2, que provee permisos delegados sin exponer credenciales directas. Los IDs de cliente y turno se gestionan de manera incremental: el agente consulta el valor máximo existente y asigna el siguiente número disponible.

## Limitaciones Técnicas Reconocidas {#sec:limitaciones}

El diseño presenta tres limitaciones técnicas principales. Primero, el sistema carece de persistencia de memoria conversacional entre sesiones. El historial se mantiene únicamente durante la ejecución activa del workflow, lo que implica que un cliente que retoma la conversación tras un intervalo prolongado debe reiniciar el flujo de captura de datos.

Segundo, n8n ejecuta workflows de manera single-threaded por defecto, lo que puede generar condiciones de carrera ante interacciones concurrentes de múltiples clientes que modifiquen el mismo registro.

Tercero, la lista de identificadores de administradores autorizados se gestiona como un parámetro de configuración estático del workflow, cargado manualmente. Esto implica que agregar o revocar el acceso de un usuario requiere la edición directa del workflow.

# Implementación

## Configuración del Bot de Telegram

El bot se creó mediante BotFather, obteniendo el token de autenticación que se almacenó en las credenciales de n8n. El nodo Telegram Trigger se configuró para recibir actualizaciones de tipo `message`. La validación de administradores se implementa en un nodo Code inmediatamente posterior al Telegram Trigger: extrae el campo `from.id`, lo compara contra la lista de User IDs autorizados y establece la variable booleana `admin`.

## Configuración de los Agentes de IA

Cada agente se configuró con un prompt de sistema detallado (incluido en el Anexo [\[anexo:prompts\]](#anexo:prompts){reference-type="ref" reference="anexo:prompts"}) que define rol, capacidades, flujos de trabajo y reglas de comportamiento. Se utilizó el tipo Tools Agent de LangChain con modo de prompt define. La memoria se implementó mediante nodos Window Buffer Memory con ventana de 30 mensajes. Para el agente de clientes, la clave de sesión se compone del identificador de chat (`chat.id`) con un sufijo de versión, permitiendo aislamiento de contexto por usuario.

## Integración con Google Sheets

La integración se realizó mediante autenticación OAuth2. Cada nodo Google Sheets Tool se configuró con el ID del documento `Taller mecánico` y el nombre de la hoja correspondiente. Para la búsqueda de clientes por DNI se utilizó el modo Get by Filter, configurando la columna DNI como clave de búsqueda con valor inyectado dinámicamente mediante la expresión `$fromAI()`.

## Configuración del Modelo Mistral

Se instanciaron dos nodos Mistral Cloud Chat Model independientes utilizando la variante `mistral-large-latest`, uno por agente, garantizando aislamiento contextual y permitiendo parametrización diferenciada. Ambos nodos comparten el token de API pero operan en pipelines de ejecución separados. La temperatura se fijó en 0.3 tras una serie de pruebas comparativas informales con valores de 0.1, 0.3 y 0.7.

## Subworkflow de Consulta de Disponibilidad

El subworkflow independiente recibe dos parámetros de entrada (`fecha` y `horario`), lee la hoja Turnos mediante un nodo Google Sheets en modo Get Many, y procesa los resultados con un nodo Code que verifica si existe un turno no cancelado en la fecha y horario solicitados. Retorna un objeto JSON con los campos `result` (LIBRE u OCUPADO) y `disponibilidad` (booleano).

## Mecanismo de Respuesta

Las respuestas se canalizan mediante el nodo Telegram Send Message, que entrega el texto generado por el agente correspondiente. Los mensajes se formatean en texto plano optimizado para la visualización en Telegram, evitando tablas Markdown y utilizando listas con guiones y emojis para mejorar la legibilidad en dispositivos móviles.

# Resultados y Validación Funcional {#cap:resultados}

## Diseño Experimental de las Pruebas

Se ejecutó un protocolo de validación compuesto por dieciocho casos de prueba distribuidos en tres categorías. Las pruebas se realizaron en condiciones controladas, con datos sintéticos que simulan operaciones reales del taller. Cada prueba se documentó mediante captura de pantalla del cliente Telegram y verificación posterior del estado de la hoja de Google Sheets.

## Resultados de las Pruebas Unitarias

Se ejecutaron pruebas unitarias sobre cada herramienta del sistema. Los resultados se sintetizan en la siguiente tabla:

::: center
  Herramienta                Operación         Tiempo medio (s)
  -------------------------- --------------- ------------------
  Consultar_Stock_Publico1   Get Many                       1.8
  Obtener El ID por DNI      Get by Filter                  2.1
  Obtener Turnos             Get Many                       1.9
  Registrar Cliente1         Append Row                     2.4
  Agendar_Turno              Append Row                     2.5
  Actualizar Stock1          Update Row                     2.7
:::

## Resultados de las Pruebas de Integración

Se ejecutaron pruebas de integración cubriendo escenarios completos de interacción. El flujo completo de registro de turno para un cliente nuevo requirió la invocación encadenada de múltiples herramientas, completándose en cuarenta y dos segundos de interacción efectiva con seis intercambios mensaje-respuesta y registro correcto en ambas hojas. Para clientes existentes, el flujo se simplificó a tres intercambios y veinticuatro segundos promedio.

## Resultados de las Pruebas de Enrutamiento y Seguridad

Las pruebas de enrutamiento confirmaron que los mensajes de usuarios no administradores se enrutaron correctamente al Agente Público. El mecanismo de validación de User ID operó correctamente: los mensajes provenientes de identificadores incluidos en la lista de autorizados fueron enrutados al panel administrativo, mientras que los mensajes de usuarios no autorizados fueron derivados al Agente de Servicio Técnico, independientemente del contenido del mensaje.

Se verificó que intentos de manipulación en lenguaje natural dirigidos a solicitar operaciones administrativas no produjeron ninguna modificación en la base de datos desde el canal público, dado que el Agente de Servicio Técnico carece de las herramientas de modificación de stock y cancelación.

## Discusión de Resultados

Los resultados validan la viabilidad técnica de la arquitectura propuesta. La separación de privilegios mediante dos agentes con conjuntos de herramientas diferenciadas, combinada con la validación del User ID de Telegram como mecanismo de control de acceso, demostró ser efectiva para impedir tanto el acceso a operaciones administrativas desde el canal público como la manipulación del enrutamiento mediante el contenido de los mensajes.

La principal limitación identificada es la ausencia de persistencia entre sesiones. En pruebas donde el usuario interrumpió la conversación durante más de quince minutos, el agente no conservaba ningún dato del intercambio previo. Esto es particularmente problemático en el flujo de registro de turnos, que requiere múltiples intercambios secuenciales.

# Conclusiones

## Cumplimiento de Objetivos

El presente Trabajo Final de Carrera cumplió los objetivos planteados. El objetivo general, consistente en desarrollar y validar un workflow operativo de automatización para la gestión de un taller mecánico mediante n8n y Telegram, fue alcanzado y verificado mediante el protocolo de pruebas. Los seis objetivos específicos se cumplieron en su totalidad: ambos agentes fueron implementados con sus respectivos conjuntos de herramientas, la integración con Google Sheets opera mediante autenticación OAuth2, el modelo Mistral Large funciona como motor de razonamiento, el enrutamiento por User ID discrimina correctamente los roles y el sistema fue validado mediante dieciocho casos de prueba.

## Aportes del Trabajo

El trabajo aporta tres contribuciones principales. Primero, un caso documentado de adopción de tecnologías low-code combinadas con LLM en una PyME del sector automotriz argentino. Segundo, una arquitectura de referencia replicable para problemas similares de automatización conversacional con separación de privilegios. Tercero, un protocolo de validación funcional adaptable a sistemas basados en agentes LLM con herramientas.

## Limitaciones del Estudio

El estudio presenta limitaciones que deben reconocerse explícitamente: diseño de caso único sin generalización estadística, validación con datos sintéticos en condiciones controladas sin pilotaje en operación real, y dependencia de servicios externos (Mistral Cloud, Google Sheets API, Telegram Bot API) que introduce riesgos de disponibilidad, continuidad y privacidad de datos.

## Líneas de Trabajo Futuro

Se identifican cinco líneas de trabajo futuro priorizadas según relevancia:

1.  Refuerzo del mecanismo de autenticación administrativa, sustituyendo la lista estática por un sistema dinámico.

2.  Implementación de persistencia de memoria conversacional con Redis o almacén de sesiones dedicado.

3.  Migración de la persistencia a un sistema de gestión de bases de datos relacional (PostgreSQL o Supabase).

4.  Integración con Google Calendar para gestión dinámica de disponibilidad.

5.  Incorporación de recordatorios automatizados mediante workflows programados.

Se identifica además como línea prioritaria el análisis de conformidad con la Ley 25.326 de Protección de Datos Personales.

# Referencias Bibliográficas {#referencias-bibliográficas .unnumbered}

1.  Adamopoulou, E., & Moussiades, L. (2020). Chatbots: History, technology, and applications. *Machine Learning with Applications*, 2, 100006.

2.  Chase, H. (2022). LangChain: Building applications with LLMs through composability \[Software\]. https://github.com/langchain-ai/langchain

3.  Eller, R., Alford, P., Kallmünzer, A., & Peters, M. (2020). Antecedents, consequences, and challenges of small and medium-sized enterprise digitalization. *Journal of Business Research*, 112, 119--127.

4.  Hernández-Sampieri, R., & Mendoza, C. P. (2018). *Metodología de la investigación: Las rutas cuantitativa, cualitativa y mixta*. McGraw-Hill.

5.  Hofmann, P., Samp, C., & Urbach, N. (2020). Robotic process automation. *Electronic Markets*, 30(1), 99--106.

6.  Jiang, A. Q., et al. (2023). Mistral 7B. arXiv. https://doi.org/10.48550/arXiv.2310.06825

7.  Sahay, A., Indamutsa, A., Di Ruscio, D., & Pierantonio, A. (2020). Supporting the understanding and comparison of low-code development platforms. *Proceedings of the 46th Euromicro Conference on Software Engineering and Advanced Applications*, 171--178.

8.  Stake, R. E. (1995). *The art of case study research*. Sage Publications.

9.  Statista. (2025). Most popular social media platforms in Argentina as of 2nd quarter 2025. https://www.statista.com/

10. Syed, R., et al. (2020). Robotic process automation: Contemporary themes and challenges. *Computers in Industry*, 115, 103162.

11. van der Aalst, W. M. P., Bichler, M., & Heinzl, A. (2018). Robotic process automation. *Business & Information Systems Engineering*, 60(4), 269--272.

12. Yao, S., et al. (2023). ReAct: Synergizing reasoning and acting in language models. *Proceedings of the 11th ICLR*.

**Documentación técnica consultada**

-   LangChain. (2025). LangChain documentation: Agents. https://python.langchain.com/docs/modules/agents/

-   Mistral AI. (2025). Mistral AI API documentation. https://docs.mistral.ai/

-   n8n. (2025). n8n documentation. https://docs.n8n.io/

-   Telegram. (2025). Telegram Bot API. https://core.telegram.org/bots/api

# Anexos {#anexos .unnumbered}

## Anexo 8.1. Prompts de Sistema Completos {#anexo:prompts .unnumbered}

### 8.1.1. Prompt del Agente Público (Servicio Técnico) {#prompt-del-agente-público-servicio-técnico .unnumbered}

El prompt del agente público es extenso e incluye las siguientes secciones principales:

1.  **Rol e identidad**: Se define como asistente virtual del Taller Mecánico, en español rioplatense (voseo), con mensajes breves y claros.

2.  **Contexto temporal y calendario**: Incluye la fecha y hora actual mediante expresiones `$now` de n8n, con una tabla de fechas relativas (mañana, en 2 días, etc.) y reglas de calendario (domingos cerrado, horario 08:00--18:00).

3.  **Regla de prioridad -- disponibilidad**: Ante cualquier mención de horario específico, debe consultar disponibilidad con la herramienta `consultar_disponibilidad` antes de solicitar datos personales.

4.  **Descripción detallada de cada herramienta**: Con nombres exactos, parámetros esperados y ejemplos de uso.

5.  **Regla crítica de IDs incrementales**: Procedimiento paso a paso para calcular ID_Cliente e ID_Turno como máximo existente + 1.

6.  **Flujo de agendamiento de turno**: 7 pasos secuenciales desde verificación de disponibilidad hasta confirmación.

7.  **Flujo de modificación/reprogramación**: Validación de horario, verificación de disponibilidad y confirmación.

8.  **Flujo de cancelación**: Identificación, verificación y confirmación.

9.  **Reglas generales**: Prohibición de inventar valores, un tema por vez, confirmación explícita siempre.

### 8.1.2. Prompt del Agente Privado (Panel Administrativo) {#prompt-del-agente-privado-panel-administrativo .unnumbered}

    Sos el asistente de gestion interna del Taller Mecanico, accesible solo para
    el administrador o mecanico a cargo. Tu funcion es ayudar a gestionar el
    stock, los turnos y los clientes de forma eficiente.

    ## TUS CAPACIDADES
    - Consultar el stock completo de repuestos
    - Actualizar el stock de un repuesto (cantidad disponible)
    - Consultar todos los turnos registrados
    - Consultar la lista de clientes

    ## COMANDOS DISPONIBLES

    ### Gestion de Stock:
    - Para ver el stock, el admin puede pedir "ver stock"
    - Para actualizar el stock: pedi el nombre o codigo del repuesto y la
      nueva cantidad
    - Mostra: Producto, Codigo, Precio de costo, Precio de venta y Stock actual
    - Usa "Obtener Repuestos" para consultar y "Actualizar Stock1" para
      modificar

    ### Gestion de Turnos:
    - Para ver turnos, el admin puede pedir "ver turnos"
    - Mostra: ID_Turno, ID_Cliente, Auto, Motivo, Fecha, Horario y Estado
    - Usa "Obtener Turnos" para consultar

    ### Gestion de Clientes:
    - Para ver clientes, el admin puede pedir "ver clientes"
    - Mostra: ID_Cliente, Nombre, DNI, Direccion y Telefono
    - Usa "Obtener Clientes" para consultar

    ## REGLAS
    - Panel solo para uso interno. No reveles info sensible.
    - Confirma siempre las modificaciones antes de ejecutarlas.
    - Si la instruccion es ambigua, pedi confirmacion.
    - Habla directo y tecnico, sin formalidades.
    - Para actualizar stock, usa el codigo exacto del repuesto.

    ## FORMATO DE RESPUESTAS (Telegram)
    - NO USES TABLAS MARKDOWN. Telegram no las renderiza bien.
    - Usa listas con guiones (-) o numeros (1., 2.).
    - Usa simbolos para estructura: [stock], [turnos], [clientes], [OK] confirmado.
    - Datos en lineas separadas, no en columnas.
    - Ejemplo correcto:
      Producto: Filtro de aceite
      Codigo: FIL-001
      Precio: $2500
      Stock: 15 unidades
    - Confirmaciones: usa [OK] o [ERROR] segun el resultado.
    - Si hay error o el dato no existe, informalo claro.

    Estas listo para recibir instrucciones del administrador.

## Anexo 8.2. Diagrama de Flujo del Workflow Principal {#anexo-8.2.-diagrama-de-flujo-del-workflow-principal .unnumbered}

Descripción estructurada del flujo principal:

El workflow se inicia con el nodo **TELEGRAM TRIGGER** (evento: message). Inmediatamente después, un nodo **CODE** extrae el campo `from.id` y lo compara contra la lista de administradores autorizados, estableciendo la variable `admin`.

El nodo **SWITCH** evalúa en orden:

1.  **Condición**: `admin` es verdadera $\rightarrow$ **Agente Administrativo**

    -   AI AGENT (Agente Administrativo)

    -   Tools: Obtener Repuestos, Obtener Clientes, Obtener Turnos, Actualizar Stock1

    -   Salida: Telegram Send Message

2.  **Condición**: `admin` es falsa y texto es `"/start"` $\rightarrow$ **Mensaje de Bienvenida**

    -   Telegram Send Message (texto estático)

3.  **Default**: **Agente Público**

    -   AI AGENT (Agente Público)

    -   Tools: Consultar_Stock_Publico1, Obtener El ID por DNI, Herramienta_Verificar_DNI1, Obtener Clientes, Obtener Turnos, Obtener Turno Por id, Registrar Cliente1, Agendar_Turno, Herramienta_Cancelar_Turno1, Modificar_turno, Consultar Disponibilidad Tool

    -   Memoria: Window Buffer (30 mensajes, clave por chat.id)

    -   Salida: Telegram Send Message

## Anexo 8.3. Modelo de Datos Lógico {#anexo-8.3.-modelo-de-datos-lógico .unnumbered}

Entidades y relaciones:

-   **Cliente** ([ID_Cliente]{.underline} PK, Nombre y Apellido, DNI UK, Dirección, Número De Teléfono)

-   **Turno** ([ID_Turno]{.underline} PK, [ID_Cliente]{.underline} FK, Auto, Motivo, Fecha del Turno, Horario, Estado)

-   **Repuesto** ([Código]{.underline} PK, Repuesto, Stock Actual, Stock Mínimo, Precio Unitario, Categoría)

Relaciones: un Cliente puede tener muchos Turnos (1:N) mediante ID_Cliente. Los IDs son numéricos e incrementales, gestionados por el agente mediante consulta del máximo valor existente.

## Anexo 8.4. Guion de Entrevista al Caso de Estudio {#anexo:entrevista .unnumbered}

-   **Bloque 1 -- Gestión de turnos**: ¿Cómo registra los turnos actualmente? ¿Cuántos turnos se solapan o duplican por mes? ¿Cuántas consultas recibe fuera del horario laboral?

-   **Bloque 2 -- Control de inventario**: ¿Cómo lleva el stock de repuestos? ¿Con qué frecuencia detecta faltantes durante una reparación? ¿Cuánto tiempo le insume verificar el stock?

-   **Bloque 3 -- Comunicación con clientes**: ¿Por qué canales se comunican los clientes? ¿Conserva historial de servicios por cliente? ¿Cómo gestiona los recordatorios?

-   **Bloque 4 -- Disposición tecnológica**: ¿Qué herramientas digitales utiliza actualmente? ¿Qué barreras identifica para adoptar nuevas herramientas? ¿Estaría dispuesto a probar un asistente automatizado?

## Anexo 8.5. Capturas de Pantalla de Pruebas Funcionales {#anexo-8.5.-capturas-de-pantalla-de-pruebas-funcionales .unnumbered}

Las capturas de pantalla referenciadas en la sección 6.1 corresponden a dieciocho pruebas funcionales documentadas. Cada figura debe insertarse en el orden indicado en la versión final del documento.
