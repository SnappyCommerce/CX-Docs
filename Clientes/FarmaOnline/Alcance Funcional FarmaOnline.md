> Fuente: Analisis_Funcional_Farmaonline_Gen6_v1.docx (https://drive.google.com/file/d/10KCPBWbJjQ4zpqGdSdNP7oqLQFzqAqdo/view?usp=drivesdk)

Snappy Commerce

# Análisis Funcional — Farmaonline, Agente IA

Farmaonline Bot · Relevamiento del comportamiento vigente en producción (Gen 5) como base para la migración a Gen 6

## Cómo usar este documento

Este documento releva el comportamiento actual del Agente IA de Farmaonline — flujos nativos, flujos personalizados y configuración global — siguiendo la estructura del alcance funcional estándar de Snappy Commerce. Cada fila define el flujo, sus activadores y el comportamiento esperado.

Sirve para que Producto valide el comportamiento vigente, detecte vacíos funcionales e inconsistencias de configuración, y tenga documentado cada flujo antes de que pase de la UI de Snappy (Gen 5) a código TypeScript mantenido por Producto (Gen 6). La experiencia del usuario final no cambia con la migración.

**Contexto del bot:** Farmaonline es un bot B2B de soporte operativo dirigido exclusivamente a farmacias que actúan como puntos de retiro. No es un bot de e-commerce ni de atención al cliente final. Las secciones de flujos se adaptan a esta naturaleza B2B.

**Criterio de documentación de flujos:** Se documentan los flujos custom que ejecutan al menos una acción distinta de FAQ, asignación de topic o redirección a menú. Los flujos que únicamente combinan estas tres acciones quedan excluidos de las tablas.

Las notas marcadas con ⚠ señalan flujos desactivados o puntos a confirmar. Los hallazgos y riesgos abiertos se listan al final del documento.

# Requerimientos Funcionales

## Configuración Global

| Flujo nativo | Qué hace el flujo | Valor configurado |
|---|---|---|
| Crear contacto — Campos a solicitar al usuario | Para crear un contacto nuevo, el agente solicita los datos de identificación de la farmacia. Incluye campos adicionales propios del contexto B2B. | firstName, phone + campos adicionales: N° de Farmacia, Nombre de Farmacia |
| Asistencia | Siempre que se dispara asistencia, se solicitan datos de contacto. En horario de atención se escala a un asesor. Fuera de horario se ejecuta el flujo outOfHours. | Contacto: siempre<br>Escalar: dentro de horario<br>OutOfHours: fuera de horario<br>⚠ inSchedule: false |
| Encuesta de satisfacción | Envío automático de encuesta de satisfacción al cierre de conversaciones. | DESACTIVADO |
| Búsqueda de Producto (productSearch) | Flujo nativo de búsqueda en catálogo de productos. No aplica al contexto B2B de Farmaonline. | DESACTIVADO - No tienen integración de ecommerce activa |
| Recomendación de Producto (productRecommendation) | Flujo nativo de recomendación de productos. No aplica al contexto B2B de Farmaonline. | DESACTIVADO - No tienen integración de ecommerce activa |
| Comenzar checkout | Flujo nativo de inicio del proceso de compra. No aplica al contexto B2B de Farmaonline. | No aplica - No tienen integración de ecommerce activa |
| Estado de Pedido (orderStatus) | Consulta de estado de pedido vía integración. No aplica al contexto B2B de Farmaonline. | No aplica - No tienen integración de ecommerce activa |

## Instrucciones del Bot

| Categoría | Instrucción |
|---|---|
| Rol | Asistente virtual de Farmaonline. Objetivo: asistir a las farmacias (puntos de retiro) de forma ágil, clara, cercana y profesional. No actúa como canal comercial ni busca convertir usuarios finales. No ofrece promociones ni intenta vender. |
| Tono y actitud | Conversacional, directo, empático y resolutivo. Buena onda con profesionalidad. Prioriza claridad, brevedad y pasos accionables. |
| Idioma | Español rioplatense con voseo en todos los verbos ("podés", "acá", "decime"). No usar tono neutro, español de España, "che", conjugaciones de "tú", trato de usted ni jerga burda. |
| Pautas de estilo | Saludar con buena onda y ofrecer el menú de opciones. Frases cortas y directas; evitar tecnicismos salvo que la farmacia los use primero. Explicar procesos paso a paso. Ante errores, pedir disculpas y ofrecer la solución concreta. No inventar datos. |
| Acciones habilitadas | Indicar pasos para cambios, devoluciones, facturación y pick-up. Solicitar datos mínimos (N° de orden, DNI) con frases claras para poder proceder. |
| Prohibiciones | No promover descuentos, cupones ni conversiones hacia clientes finales. No dar consejos médicos ni dosis fuera del material aprobado. No inventar plazos, números de tracking ni estados. |
| Manejo de imágenes | Si el mensaje contiene solo una imagen, no analizar, interpretar ni describir su contenido. Responder solicitando que el usuario describa su consulta por escrito. Si reenvía otra imagen sin texto, repetir el mensaje. |
| Manejo de mensajes de voz | Indicar al usuario que escriba su consulta en formato texto. |

## Flujos de Pre-Venta

En el contexto B2B de Farmaonline, esta sección cubre los flujos de incorporación de farmacias y operación diaria como puntos de retiro. Se excluyen los flujos que únicamente combinan acciones de FAQ, asignación de topic o redirección a menú (~30 flujos).

| Flujo | Activadores | Comportamiento esperado |
|---|---|---|
| FarmaciaNueva | Primera conversación de una farmacia sin datos de contacto registrados.<br>✅ "Hola, somos la farmacia X y es la primera vez que escribimos", "Buen día, necesito ayuda" (sin registro previo).<br>❌ No activar para farmacias ya registradas → FarmaciaRecurrente. | Solicita nombre, teléfono, número de identificación de farmacia y nombre de la farmacia. Diferencia si el contacto es nuevo o ya existía y ajusta el mensaje de bienvenida. Luego despliega el menú principal. |
| FarmaciaRecurrente | Inicio de conversación de una farmacia con datos de contacto ya registrados. | Envía mensaje de bienvenida de retorno y despliega el menú principal. |
| NoSeVenEnElMapa | Farmacia indica que no aparece en el mapa de puntos de retiro.<br>✅ "No aparecemos en el mapa", "Los clientes no nos encuentran en la app", "No figuramos como punto de retiro". | Registra topic (visualizaciónEnFOL). Escala a asistencia humana. |
| EntregaDelPedidoSinPalabraClave | Farmacia ya entregó un pedido sin solicitar la palabra clave.<br>✅ "Entregué el pedido sin pedir la clave", "Me olvidé de pedir la clave y ya lo entregué".<br>❌ Si aún no entregó y el cliente no tiene la clave → ReenvioDePalabraClave (excluido de esta tabla). | Envía FAQ con el procedimiento para este caso. Registra topic (cAMBIODEESTADOEntregaSinPC). Escala a asistencia humana. |
| NoApareceBotonAvisarAlCliente | Luego de confirmar que la farmacia tiene el pedido físicamente, no le aparece el botón para avisar al cliente.<br>✅ "No me aparece el botón para avisar al cliente", "Confirmé que tengo el pedido pero no puedo notificarlo".<br>❌ No activar si el pedido no figura en el sistema → CambiosDeEstado. | Envía FAQ con instrucciones sobre el botón de aviso. Registra topic (cAMBIODEESTADONoApareceElBotón). Escala a asistencia humana con ticketReason: "other". |
| ProblemasTecnicos | Farmacia reporta un problema técnico en el Seller Center o en la plataforma.<br>✅ "No me anda el sistema", "Tengo un error en el seller center", "Se cayó la plataforma". | Solicita descripción del problema y evidencia (foto o video). Si no dispone de evidencia, registra "false" y continúa. Escala a asistencia humana. Registra topic (pDRProblemasTécnicosSellerCenter). Envía FAQ de referencia. |
| NoHayInformaciónDeLoConsultado | Consulta de la farmacia que no corresponde a ningún flujo ni FAQ existente. Funciona como fallback genérico. | Envía mensaje solicitando más información ("¿Nos podría resumir cuál es el problema que está experimentando?"). Registra topic (pDROtrasConsultas). |

## Flujos de Post-Venta

En el contexto B2B de Farmaonline, esta sección cubre los flujos de gestión del ciclo de vida de la cuenta de la farmacia.

| Flujo | Activadores | Comportamiento esperado |
|---|---|---|
| InactivaciónDefinitiva | Farmacia solicita darse de baja definitiva como punto de retiro.<br>✅ "Quiero darme de baja", "Vamos a cerrar la farmacia", "Ya no queremos seguir siendo punto de retiro".<br>❌ No activar para cierre temporal → CierrePorTiempoDeterminado. | Envía FAQ con información sobre el proceso de baja definitiva. Escala a asistencia humana con ticketReason: "other". |

## Flujos de Sistema (Automatizados)

| Flujo | Activadores | Comportamiento esperado |
|---|---|---|
| Mensaje en Horario (inHours) | Se activa automáticamente cuando un usuario solicita asistencia dentro del horario de atención configurado. | Envía notificación de aviso al usuario indicando que está siendo conectado con un asesor. Inmediatamente después escala la conversación en vivo. |
| Mensaje Fuera de Horario (outOfHours) | Se activa automáticamente cuando un usuario solicita asistencia fuera del horario de atención configurado. | Envía notificación al usuario informando que el equipo está fuera de horario (L-V 9 a 18 hs) y que será contactado a la brevedad. No escala en vivo. |
| Mensaje de Bienvenida Automático | Sin activador por mensaje del usuario — se dispara desde automatizaciones al iniciar el primer contacto. | DESACTIVADO |
| Seguimiento Proactivo | Sin activador por mensaje del usuario — automático tras un tiempo configurable de inactividad. | DESACTIVADO |
| Cierre por Inactividad + CSAT | Sin activador por mensaje del usuario — automático al quedar la conversación inactiva. | DESACTIVADO |
| Notificaciones / Broadcast | Se dispara desde un envío de notificación o broadcast programado en Labs. | DESACTIVADO |

## Flujos con Lógica Condicional

Sin flujos con lógica condicional documentada en esta versión.

⚠ Cualquier comportamiento no cubierto explícitamente en este documento debe confirmarse con el referente técnico del cliente antes de la implementación.

## Diagramas de Flujo

Representación visual de la lógica de decisión de los flujos condicionales listados arriba, tal como está configurada hoy en producción.

Sin flujos con lógica condicional — sin diagramas en esta versión.

# Requerimientos No Funcionales

| Categoría | Requerimiento |
|---|---|
| Lenguaje | Español rioplatense. Voseo en todos los verbos ("podés", "acá", "decime"). Nunca "che", conjugaciones de "tú", trato de usted, español neutro ni jerga burda. Tono profesional, directo y cercano. |
| Testing | El código/JSON debe pasar el test suite estándar de Snappy (umbral 85%+) antes de cualquier subida a producción. |
| Conversaciones | El agente debe contar con contexto de las conversaciones previas de cada farmacia y poder responder usando esa información (ej.: reconocer a qué ticket se refiere en una conversación nueva). |
| Comportamiento — Fallback | Cuando no cuente con información suficiente en la Base de Conocimientos, debe ejecutar el flujo NoHayInformaciónDeLoConsultado: solicitar más detalle a la farmacia y registrar topic (pDROtrasConsultas). |
| Comportamiento — Imágenes | No analizar, interpretar ni describir imágenes recibidas. Responder solicitando que la farmacia describa su consulta por escrito. |
| Comportamiento — Mensajes de voz | Indicar al usuario que escriba su consulta en formato texto. |
| Horario de atención | Lunes a viernes de 9 a 18 hs. Fuera de ese horario, el agente ejecuta el flujo outOfHours y no escala en vivo.<br>⚠ inSchedule: false — confirmar si el control de horario está activo en producción. |

## Criterios de Aceptación

- Cada flujo migrado a Gen 6 reproduce el comportamiento de Gen 5 documentado en este alcance.
- Farmaonline Bot pasa el test suite estándar de Snappy con un umbral de 85%+.
- El bot diferencia correctamente FarmaciaNueva de FarmaciaRecurrente según los datos de contacto registrados.
- El flujo RequisitoComisionesPDFdigital se implementa con comportamiento definido y aprobado antes de la subida a producción.
- El agente solicita evidencia (foto o video) en ProblemasTecnicos y registra "false" si la farmacia no la tiene disponible.
- El agente no genera tickets ni procesa imágenes ni mensajes de voz.
- El control de horario (inSchedule) se verifica y corrige antes de la migración a Gen 6.
- El agente ejecuta el fallback NoHayInformaciónDeLoConsultado cuando no disponga de información suficiente en su Base de Conocimientos.
