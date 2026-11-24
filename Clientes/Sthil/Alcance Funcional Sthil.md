> Fuente: Alcance_Funcional_Stihl (https://docs.google.com/document/d/101ugTseggERC8Azwy5XOaxkvjqmH3wxE3-WrCe770ZU/edit?usp=drivesdk&ouid=113568244751416058438)

**Snappy Commerce**

# Alcance Funcional Stihl

## Cómo usar este documento

Este documento describe la estructura estándar de flujos de un Agente IA de atención al cliente y ventas de Snappy Commerce. Sirve como punto de partida para el relevamiento (discovery) de un cliente nuevo: cada fila define el flujo, sus activadores típicos y el comportamiento esperado en términos generales.

Los textos entre corchetes y las notas marcadas con ⚠ señalan puntos que dependen de la política, integración o contenido propio de cada cliente y deben completarse durante el discovery. Los nombres de los flujos pueden mantenerse o adaptarse a la nomenclatura interna del cliente.

# Requerimientos Funcionales

## Configuración Global

| **Flujo nativo** | **Qué hace el flujo** | **Valor configurado** |
|---|---|---|
| Crear contacto - Campos a solicitar al usuario | Para crear un contacto nuevo, el bot pide teléfono y email. | Nombre, teléfono, mail (optativos) |
| Comenzar checkout - Requisitos de información de contacto | Para generar un checkout, el default pide nombre, apellido y email. | firstName, lastName, email |
| Asistencia | Siempre que se dispara asistencia, el bot genera un ticket de soporte primero (pidiendo los datos que falten). Si además es horario de atención, después de crear el ticket la conversación se escala en vivo a un asesor. Fuera de horario, queda solo con el ticket creado, a la espera de gestión. | Contacto: Siempre<br>Escalar: Dentro de horario<br>Ticket: Fuera de horario |
| Encuesta de satisfacción | La Encuesta de Satisfacción automática global está activado | true |
| Estado de Pedido | El flujo default de consulta de estado de pedido es el mismo para todos los clientes. Los datos disponibles (tracking, fecha estimada, etc.) dependen de lo que devuelva la integración del cliente. | true / datos según integración (orderStatus providers)<br>Busqueda por numero de pedido<br>Andreani vinculado |
| Inicio de todas las conversaciones | Se activa al comienzo de todas las conversaciones, luego del primer mensaje del usuario. | Se consulta al usuario si es de **bs as o cordoba o del interior del pais** y guardar la información de la respuesta para poder responder según el conocimiento<br>**Diagrama 0** |

## Flujos de Pre-Venta

| **Flujo** | **Activadores** | **Necesidad / Comportamiento esperado** |
|---|---|---|
| Búsqueda y Recomendación de Producto | Usuario pregunta por un producto, categoría, talle, color o atributo ("tenés zapatillas rojas talle 40?"). | true / resultados según integración y catálogo del cliente<br>Cuentan con un flujo custom que varía su comportamiento en base a ubicación del usuario **Diagrama 1** |
| Consulta de Precio / Stock | Usuario pregunta precio, disponibilidad o stock de un producto puntual. | Responde con la información obtenida vía integración (VTEX). |
| Promociones | Usuario pregunta por descuentos, promos, cupones o "qué ofertas tienen". | Debe responder con FAQ => ¿Qué promociones vigentes hay? |
| Información de Sucursales / Branches | Usuario pregunta dirección, horarios o sucursal más cercana. | Responde con la información cargada en conocimientos (branches) de sucursales (mapa nativo) |
| FAQ Comercial (talles, envíos, medios de pago) | Usuario consulta guía de talles, costos/tiempos de envío, medios de pago o cuotas. | Responde con el contenido de la Base de Conocimientos (FAQ) configurada para cada tema.<br>Información cargada en faqs diferentes: Metodo de envio (bs as y cordoba), Metodo de envio (resto del país), etc<br>El bot usará el dato consultado al comienzo de la conversación para definir con qué información responder. |
| Checkout Asistido | Usuario quiere comprar / pide el link de compra ("quiero finalizar la compra", "pasame el link"). | **Si el cliente indicó al comienzo de la conversación que es de buenos aires o córdoba:**<br>Solicita los datos de contacto configurados (firstName, email y lastName) y genera el link de checkout de VTEX<br>**Si el cliente indicó que NO es de buenos aires o córdoba:**<br>Solicita asistencia con datos adicionales provincia y localidad |
| Checkout Asistido — Gestión del Carrito | Usuario quiere agregar, quitar o modificar productos en un carrito ya iniciado. | Actualiza el carrito vía integración según la acción solicitada, validando stock de la combinación elegida. |
| Derivación a Vendedor / Asistencia comercial | Usuario pide hablar con un asesor de ventas o la consulta excede el alcance del bot. | Dispara el flujo de asistencia: pide datos de contacto y escala en vivo o genera ticket según horario de atención. |
| Consulta Repuestos | Usuario consulta sobre REPUESTOS o ACCESORIOS, ya sea mencionando una pieza por su nombre técnico, preguntando por compatibilidad con un modelo, consultando precios o características de un repuesto. | Responde con FAQ => ¿Cómo puedo obtener piezas de repuesto para mi máquina? |

## Flujos de Post-Venta

| **Flujo** | **Activadores** | **Necesidad / Comportamiento esperado** |
|---|---|---|
| Estado de Pedido | Usuario pregunta por el estado de su pedido/envío | Envía un mensaje orderStatus con el estado, fecha estimada y tracking (si está disponible) vía la integración del cliente.<br>Integración Andreani activada |
| Cambios Devoluciones, Cancelaciones | Usuario quiere cambiar o devolver un producto ya comprado. | Debe responder con conocimientos/faq =><br>Información:<br>Conocimiento: VTEX Sacar info de: <https://tienda.stihl.com.ar/cambios-y-devoluciones>.<br>FAQ: Concesionaria => PENDIENTE STIHL DEFINIR |
| Garantía | Usuario por la garantía de un producto. | Responder con una única FAQ:<br>Pueden reclamar la garantía en cualquier concesionaria, es lo mismo para vtex y presencial |
| Reclamos / Ticket | Usuario reporta un problema (producto dañado, faltante, error de facturación, etc.). | Tiene que responder con una única FAQ => (Pendiente Stihl) |
| Encuesta de Satisfacción (CSAT) | Se dispara automáticamente al cierre de una gestión o conversación. | Envía la encuesta de satisfacción configurada de forma global (flujo nativo activado). |
| Derivación a Soporte Post-venta | La consulta post-venta excede el alcance del bot o el usuario pide hablar con un humano. | Dispara el flujo de asistencia: pide datos de contacto y escala en vivo o genera ticket según horario de atención. |

## Flujos de Sistema (Automatizados)

| **Flujo** | **Activadores** | **Necesidad / Comportamiento esperado** |
|---|---|---|
| Mensaje de Bienvenida | Sin activador por mensaje del usuario — se dispara desde automatizaciones al iniciar el primer contacto. | Pendiente del lado del cliente enviar el copy. |
| Seguimiento Proactivo | Sin activador por mensaje del usuario — automático tras un tiempo configurable de inactividad. | Mensaje de seguimiento en base al contexto de la conversación.<br>SOLO DEBE HACERSE **UN** SEGUIMIENTO POR CONVERSACIÓN. |
| Cierre por Inactividad + CSAT | Sin activador por mensaje del usuario — automático al quedar la conversación inactiva. | Desactivado para este cliente. |
| Notificaciones / Broadcast | Se dispara desde un envío de notificación/broadcast programado en Labs. | Cuentan con tres automatizaciones de notificaciones.<br>1. Carritos abandonados VTEX.<br>2. Notificacion "derivacion" para las ordenes estado "Transfered"<br>3. Notificacion "seguimiento derivacion" para las ordenes estado "Waiting seller handling" |

# Requerimientos No Funcionales

| **Categoría** | **Requerimiento** |
|---|---|
| Lenguaje | Español rioplatense, tono profesional pero cercano, ameno y cordial. |
| Testing | El código/JSON debe pasar el test suite estándar de Snappy (umbral 85%+) antes de cualquier subida a producción. |
| Conversaciones | El agente debe contar con contexto de las conversaciones previas de cada contacto y poder responder usando esa información (ej.: reconocer a qué ticket se refiere el usuario en una conversación nueva). |
| Comportamiento — Fallback | Cuando no cuente con información suficiente en Conocimientos, debe ofrecer solicitar asistencia de un operador humano. |
| Comportamiento — Proactividad y reencauce | Ante silencio de 5 minutos en un flujo de venta en curso, reactiva la conversación con un mensaje de seguimiento. Ante desvío de tema, responde la consulta puntual y luego reencauza hacia la venta en curso, sin abandonarla. |

## Flujos con Lógica Condicional

Los siguientes flujos tienen ramificaciones internas y requieren un diagrama de decisión propio, construido durante el discovery de cada cliente. Se listan aquí los puntos de ramificación genéricos que ese diagrama debe cubrir:

- Consulta inicial: Cuando el usuario ingresa primero se pregunta de donde es para poder brindarle la informacion requerida segundo su localidad
- Product search: dependiendo en donde se encuentre va para búsqueda de producto o de brindarle un asesor

*⚠ Cualquier comportamiento no cubierto explícitamente en este documento debe confirmarse con el referente técnico del cliente antes de la implementación.*

# Diagramas de Flujo

Representación visual de la lógica de decisión de los flujos condicionales listados arriba. Son diagramas base para discovery: los rótulos entre corchetes o marcados con ⚠ en las tablas anteriores deben reflejarse aquí una vez definidos con el cliente.

*Versión editable de estos diagramas (formas nativas, sin necesidad de recrearlos):* [*Diagramas_Flujo_Bot_IA.pptx*](https://docs.google.com/presentation/d/1kb8Y6jXdkEBeeNXafHDiukxiLIUOeW3H/edit?slide=id.p8#slide=id.p8)*, adjunto junto con este documento.*

*(los diagramas son imágenes y no están incluidos en el texto exportado del documento)*

## Checklist de Configuración por Cliente

Antes de dar por cerrado el alcance funcional de un cliente nuevo, completar y validar:

- Políticas de cambios y devoluciones: se solicita número de pedido, teléfono, prenda a cambiar y motivo, y se deriva a ticket.
- Operadores logísticos/ métodos de envío, zonas de cobertura y tiempos estimados.
- Plataforma de e-commerce: Vtex.
- ticketReasons y sectores de derivación configurados en el HelpDesk / CRM del cliente.
- Contenido de la Base de Conocimientos: FAQs de talles/atributos, envíos, promociones y otras consultas frecuentes.
- Mensajes fijos (bienvenida, cierre, seguimiento proactivo) y condición de activación de cada uno.
- Nombre y datos de contacto del referente del cliente para resolver puntos pendientes durante la implementación.

# Test

**TEST**

| **Flujo** | **Usuario:** | **Bot:** |
|---|---|---|
| Promociones | | <https://tienda.stihl.com.ar/promociones?_q=promociones&map=ft> |
| Cambios | | |
| Devoluciones | | |
| Reclamos | | |
| Garantía | | |
| Servicio técnico | | |
| Método de pago:<br>Si es de buenos aires o córdoba: Responder con la info de conocimiento: https://tienda.stihl.com.ar/metodos-de-pago<br>Si no es de buenos aires o córdoba: Responder con faq => Pendiente cargar<br>Método de envío: Responder con FAQ => ¿Hacen envio? | | |
| Propuestas Comerciales | Usuario presenta una PROPUESTA COMERCIAL, solicita sponsoreo, pide donaciones o quiere establecer una relación comercial con la marca | Responde con FAQ => Cómo me comunico con el equipo de marketing? |
| Servicio Tecnico | Usuario busca REPARACIÓN o SERVICIO TÉCNICO para un equipo, o cuando reporta que un equipo no funciona sin haber hecho una compra reciente. | Responder con FAQ => Dónde puedo pedir servicio técnico?<br>Queda pendiente que actualicen la FAQ |

# Segunda Etapa

**CAMBIOS Segunda etapa**

1. Cuando el bot detecte un mensaje en otro idioma debe indicar que es stihl argentina y debe dejar de responder.
2. Ante reacciones o menciones en historias de redes sociales que no conteste y NO solicite asistencia.
3. **Gestion de reclamos y garantías: PENDIENTE ORGANIZAR UNA LLAMADA PARA RELEVARLO**
