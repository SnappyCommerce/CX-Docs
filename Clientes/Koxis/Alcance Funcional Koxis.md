> Fuente: Alcance_Funcional_Koxis.docx (https://drive.google.com/file/d/1QD3JXAPsLOpjPGgsAgOnQcbqvkSN4j_M/view?usp=drivesdk)

**Snappy Commerce**

# Alcance Funcional Koxis — Agente IA de Atención al Cliente y Ventas

*Documento base para nuevos clientes · Plantilla de referencia completada con la configuración estándar del bot IA*

## Cómo usar este documento

Este documento describe la estructura estándar de flujos de un Agente IA de atención al cliente y ventas de Snappy Commerce. Sirve como punto de partida para el relevamiento (discovery) de un cliente nuevo: cada fila define el flujo, sus activadores típicos y el comportamiento esperado en términos generales.

Los textos entre corchetes y las notas marcadas con ⚠ señalan puntos que dependen de la política, integración o contenido propio de cada cliente y deben completarse durante el discovery. Los nombres de los flujos pueden mantenerse o adaptarse a la nomenclatura interna del cliente.

# Requerimientos Funcionales

## Configuración Global

| Flujo nativo | Qué hace el flujo | Valor configurado |
| :-: | :-: | :-: |
| Crear contacto - Campos a solicitar al usuario | Para crear un contacto nuevo, el bot pide teléfono y email. | teléfono, email |
| Comenzar checkout - Requisitos de información de contacto | Para generar un checkout, el default pide nombre, apellido y email. | firstName, lastName, email |
| Asistencia | Siempre que se dispara asistencia, el bot genera un ticket de soporte primero (pidiendo los datos que falten). Si además es horario de atención, después de crear el ticket la conversación se escala en vivo a un asesor. Fuera de horario, queda solo con el ticket creado, a la espera de gestión. | Ticket: siempre<br>Escala en vivo: además del ticket, solo en horario de atención |
| Promociones | El flujo nativo de Promociones está siempre activado. En runtime, el bot puede chequear si hay promociones activas cargadas (hasPromotions) antes de disparar el mensaje — si no hay datos cargados en Labs, el flow debe contemplar una respuesta alternativa en vez de un mensaje vacío. | true (activado)<br>hasPromotions: depende de si el cliente cargó promociones en Labs |
| Encuesta de satisfacción | La Encuesta de Satisfacción automática global está activado | true |
| Estado de Pedido | El flujo default de consulta de estado de pedido es el mismo para todos los clientes. Los datos disponibles (tracking, fecha estimada, etc.) dependen de lo que devuelva la integración del cliente. | true / datos según integración (orderStatus providers) |
| Búsqueda de Producto (productSearch) | El flujo default de búsqueda y recomendación de producto es el mismo para todos los clientes. Los resultados, facetas y filtros disponibles dependen del catálogo/integración del cliente. | true / resultados según integración y catálogo del cliente |

## Flujos de Pre-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Búsqueda y Recomendación de Producto | Usuario pregunta por un producto, categoría, talle, color o atributo ("tenés zapatillas rojas talle 40?"). | Ejecuta productSearch contra el catálogo del cliente y devuelve un carousel de resultados. Si no hay resultados o hay demasiada ambigüedad, repregunta o deriva a asistencia. |
| Consulta de Precio / Stock | Usuario pregunta precio, disponibilidad o stock de un producto puntual. | Responde con la información obtenida vía integración (Tienda Nube). |
| Promociones | Usuario pregunta por descuentos, promos, cupones o "qué ofertas tienen". | Envía el mensaje nativo de Promociones con las promociones activas configuradas en Labs. |
| Información de Sucursales / Branches | Usuario pregunta dirección, horarios o sucursal más cercana. | Responde con la FAQ de sucursales (sin mapa nativo) y redirige al usuario a la web de la tienda para el detalle. |
| FAQ Comercial (talles, envíos, medios de pago) | Usuario consulta guía de talles, costos/tiempos de envío, medios de pago o cuotas. | Responde con el contenido de la Base de Conocimientos (FAQ) configurada para cada tema. [Contenido a cargar por cliente]. |
| Aviso de Stock (Suscripción a notificaciones) | Usuario pide que le avisen cuando un producto sin stock vuelva a estar disponible. | Desactivado para este cliente — el bot no ofrece esta opción. |
| Checkout Asistido | Usuario quiere comprar / pide el link de compra ("quiero comprar", "pasame el link"). | Solicita los datos de contacto configurados (firstName, email y telefono) y genera el link de checkout de la plataforma del cliente. |
| Checkout Asistido — Gestión del Carrito | Usuario quiere agregar, quitar o modificar productos en un carrito ya iniciado. | Actualiza el carrito vía integración según la acción solicitada, validando stock de la combinación elegida. |
| Derivación a Vendedor / Asistencia comercial | Usuario pide hablar con un asesor de ventas o la consulta excede el alcance del bot. | Dispara el flujo de Asistencia: pide datos de contacto y escala en vivo o genera ticket según horario de atención. |

## Flujos de Post-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Estado de Pedido | Usuario pregunta por el estado de su pedido/envío | Envía un mensaje orderStatus con el estado, fecha estimada y tracking (si está disponible) vía la integración del cliente. |
| Cambios y Devoluciones | Usuario quiere cambiar o devolver un producto ya comprado. | Solicita número de pedido y teléfono de contacto, pregunta qué prenda va a cambiar y por cual prenda y genera un ticket con todos los datos relevados. Diagrama 2. |
| Cancelaciones | Usuario quiere cancelar un pedido. | Solicita el número de pedido (si no lo tiene) y deriva directo a un asesor con ese dato en el contexto, sin preguntar si acepta o no. Diagrama 1. |
| Garantía | Usuario por la garantía de un producto. | Informa plazo y condiciones de garantía configuradas. [Política a definir por cliente]. |
| Reclamos / Ticket | Usuario reporta un problema (producto dañado, faltante, error de facturación, etc.). | Solicita número de pedido, pregunta qué fue lo que pasó y pide los datos de contacto, y genera un ticket con todos los datos relevados. Diagrama 3. |
| Encuesta de Satisfacción (CSAT) | Se dispara automáticamente al cierre de una gestión o conversación. | Envía la encuesta de satisfacción configurada de forma global (flujo nativo activado). |
| Derivación a Soporte Post-venta | La consulta post-venta excede el alcance del bot o el usuario pide hablar con un humano. | Dispara el flujo de Asistencia: pide datos de contacto y escala en vivo o genera ticket según horario de atención. |

## Flujos de Sistema (Automatizados)

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Mensaje de Bienvenida | Sin activador por mensaje del usuario — se dispara desde automatizaciones al iniciar el primer contacto. | Envía un mensaje fijo (y estacional, si aplica) definido por el cliente. [Contenido, tono y condición de activación a configurar por cliente]. |
| Seguimiento Proactivo | Sin activador por mensaje del usuario — automático tras un tiempo configurable de inactividad. | Envía un mensaje de seguimiento preguntando cómo le fue al usuario con la gestión. [Tiempo de inactividad y activación/desactivación a configurar por cliente]. |
| Cierre por Inactividad + CSAT | Sin activador por mensaje del usuario — automático al quedar la conversación inactiva. | Envía mensaje de cierre y dispara la Encuesta de Satisfacción (CSAT). [Tiempo de inactividad y activación/desactivación a configurar por cliente]. |
| Notificaciones / Broadcast | Se dispara desde un envío de notificación/broadcast programado en Labs. | Envía el contenido definido en la notificación y puede setear custom contexts o asignar la conversación a un sector/usuario. |

## Flujos con Lógica Condicional

Los siguientes flujos tienen ramificaciones internas y requieren un diagrama de decisión propio, construido durante el discovery de cada cliente. Se listan aquí los puntos de ramificación genéricos que ese diagrama debe cubrir:

Cambios y Devoluciones: Solicita número de pedido y teléfono, qué prenda cambia y por qué motivo, y siempre termina generando un ticket.

Cancelaciones: Ramifica según si el usuario ya tiene o no el número de pedido; en ambos casos termina derivando a un asesor.

Reclamos: Solicita número de pedido, descripción del problema y datos de contacto, y siempre termina generando un ticket.

Asistencia: Siempre genera un ticket; además, si es horario de atención, también escala en vivo a un asesor después de crear el ticket.

⚠ Cualquier comportamiento no cubierto explícitamente en este documento debe confirmarse con el referente técnico del cliente antes de la implementación.

# Diagramas de Flujo

Representación visual de la lógica de decisión de los flujos condicionales listados arriba. Son diagramas base para discovery: los rótulos entre corchetes o marcados con ⚠ en las tablas anteriores deben reflejarse aquí una vez definidos con el cliente.

Versión editable de estos diagramas (formas nativas, sin necesidad de recrearlos): [Diagramas_Flujo_Bot_IA.pptx](https://docs.google.com/presentation/d/1KlNj351L_E-98zQtmy3NNyPCU259dwls/edit?slide=id.p1#slide=id.p1), adjunto junto con este documento.

## Checklist de Configuración por Cliente

Antes de dar por cerrado el alcance funcional de un cliente nuevo, completar y validar:

Políticas de cambios y devoluciones: se solicita número de pedido, teléfono, prenda a cambiar y motivo, y se deriva a ticket.

Operadores logísticos/ métodos de envío, zonas de cobertura y tiempos estimados.

Plataforma de e-commerce: Tiendanube.

ticketReasons y sectores de derivación configurados en el HelpDesk / CRM del cliente.

Contenido de la Base de Conocimientos: FAQs de talles/atributos, envíos, promociones y otras consultas frecuentes.

Mensajes fijos (bienvenida, cierre, seguimiento proactivo) y condición de activación de cada uno.
