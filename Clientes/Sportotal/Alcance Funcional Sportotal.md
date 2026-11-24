> Fuente: Alcance_Funcional_Sportotal_Bot_IA.docx (https://drive.google.com/file/d/1_qA6ZWy5TDod1d8uWF1q60NtuAe73Zaj/view?usp=drivesdk)

Snappy Commerce

# Alcance Funcional Sportotal — Agente IA de Atención al Cliente y Ventas

## Cómo usar este documento

Este documento describe la estructura estándar de flujos de un Agente IA de atención al cliente y ventas de Snappy Commerce. Sirve como punto de partida para el relevamiento (discovery) de un cliente nuevo: cada fila define el flujo, sus activadores típicos y el comportamiento esperado en términos generales.

Los textos entre corchetes y las notas marcadas con ⚠ señalan puntos que dependen de la política, integración o contenido propio de cada cliente y deben completarse durante el discovery. Los nombres de los flujos pueden mantenerse o adaptarse a la nomenclatura interna del cliente.

# Requerimientos Funcionales

## Configuración Global

| Flujo nativo | Qué hace el flujo | Configuración por default |
| :-: | :-: | :-: |
| Crear contacto | Si el canal ya aporta un dato de contacto (teléfono en WhatsApp, ID de Instagram/Facebook) o el usuario da un email, se crea el contacto directamente sin pedir más — solo se solicita si no hay ningún dato disponible. | Teléfono / email / WhatsApp ID / Facebook ID — alcanza con uno solo. A definir los datos a solicitar. Integracion con wise y wooup |
| Asistencia | Cuando el usuario pide hablar con una persona, siempre crea un ticket | Siempre se crea ticket |
| Ticket | Cuando se abre un ticket, el bot pide el motivo de la consulta antes de generarlo (si no lo tiene) y el número de orden | Motivo obligatorio post venta: Número de orden |
| Encuesta de satisfacción | Al cerrar una conversación, se envía automáticamente una encuesta preguntando cómo fue la atención. | true |

## Flujos de Pre-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Búsqueda y Recomendación de Producto | Usuario pregunta por un producto, categoría, talle, color o atributo ("tenés zapatillas rojas talle 40?"). | Ejecuta productSearch contra el catálogo del cliente y devuelve un carousel de resultados. Si no hay resultados o hay demasiada ambigüedad, repregunta o deriva a asistencia. |
| Consulta de Precio / Stock | Usuario pregunta precio, disponibilidad o stock de un producto puntual. | Responde con la información obtenida vía integración (VTEX). |
| Promociones | Usuario pregunta por descuentos, promos, cupones o "qué ofertas tienen". | Envía el mensaje nativo de Promociones con las promociones activas configuradas en Labs. |
| Información de Sucursales / Branches | Usuario pregunta dirección, horarios o sucursal más cercana. | Envía un storeSearch con el mapa y datos de las sucursales cargadas en Labs (dirección, horario, teléfono). |
| FAQ Comercial (talles, envíos, medios de pago) | Usuario consulta guía de talles, costos/tiempos de envío, medios de pago o cuotas. | Responde con el contenido de la Base de Conocimientos (FAQ) configurada para cada tema. [Contenido a cargar por cliente]. |
| Checkout Asistido | Usuario quiere comprar / pide el link de compra ("quiero comprar", "pasame el link"). | Solicita los datos de contacto configurados (firstName, email) y genera el link de checkout de la plataforma del cliente. |
| Checkout Asistido — Gestión del Carrito | Usuario quiere agregar, quitar o modificar productos en un carrito ya iniciado. | Actualiza el carrito vía integración según la acción solicitada, validando stock de la combinación elegida. |
| Derivación a Vendedor / Asistencia comercial | Usuario pide hablar con un asesor de ventas o la consulta excede el alcance del bot. | Dispara el flujo de asistencia: pide datos de contacto y escala en vivo o genera ticket según horario de atención. |

## Flujos de Post-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Estado de Pedido | Usuario pregunta por el estado de su pedido/envío, con o sin número de orden. | Envía un mensaje orderStatus con el estado, fecha estimada y tracking (si está disponible) vía la integración del cliente. Andreani: Brinda el ultimo estado de Andreani. Correo argentino: brinda el ultimo estado de vtex (probar con pedidos con correo argentino) |
| Factura del pedido | Usuario solicita la factura de una compra. | Envía la factura del pedido correspondiente si es que la integración tiene esa función (probar por mas de una factura, sino faq). Si no funciona bien en vtex faq y derivar con asesor (ver integracion custom) |
| Cambios y Devoluciones | Usuario quiere cambiar o devolver un producto ya comprado. | Faq con el link de pow y preguntar si lo logro resolverlo; si no lo logro resolverlo derivando a un agente solicitando: adjuntar captura del producto y numero de orden |
| Cancelaciones | Usuario quiere cancelar un pedido. | Informa condiciones de cancelación y pregunta si acepta derivar el caso a un asesor; si acepta, escala; si no, cierra el flujo. Crea un ticket |
| Garantía | Usuario consulta por cobertura o activación de garantía de un producto. | Informa plazo y condiciones de garantía configuradas. [Política a definir por cliente]. Crea ticket |
| Reclamos / Ticket | Usuario reporta un problema (producto dañado, faltante, error de facturación, etc.). | Genera un ticket en el HelpDesk/CRM con el ticketReason y ticketSubreason correspondientes, solicitando los datos requeridos. |
| Encuesta de Satisfacción (CSAT) | Se dispara automáticamente al cierre de una gestión o conversación. | Envía la encuesta de satisfacción configurada de forma global. |
| Derivación a Soporte Post-venta | La consulta post-venta excede el alcance del bot o el usuario pide hablar con un humano. | Dispara el flujo de Asistencia: genera ticket con el numero de orden |

## Flujos de Sistema (Automatizados)

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Mensaje de Bienvenida | Sin activador por mensaje del usuario — se dispara desde automatizaciones al iniciar el primer contacto. | Envía un mensaje fijo (y estacional, si aplica) definido por el cliente. [Contenido, tono y condición de activación a configurar por cliente]. |
| Seguimiento Proactivo | Sin activador por mensaje del usuario — automático tras un tiempo configurable de inactividad. | Envía un mensaje de seguimiento preguntando cómo le fue al usuario con la gestión. [Tiempo de inactividad y activación/desactivación a configurar por cliente]. Activarlo y mandar uno solo |
| Cierre por Inactividad + CSAT | Sin activador por mensaje del usuario — automático al quedar la conversación inactiva. | Envía mensaje de cierre y dispara la Encuesta de Satisfacción (CSAT). [Tiempo de inactividad y activación/desactivación a configurar por cliente]. Activar |
| Notificaciones / Broadcast | Se dispara desde un envío de notificación/broadcast programado en Labs. | Envía el contenido definido en la notificación y puede setear custom contexts o asignar la conversación a un sector/usuario. |

## Checklist de Configuración por Cliente

Antes de dar por cerrado el alcance funcional de un cliente nuevo, completar y validar:

- Políticas de cambios y devoluciones (plazos, excepciones por categoría/promoción, canal de gestión).
- Política de garantía (plazo de cobertura, condiciones de aceptación).
- Transportistas / métodos de envío, zonas de cobertura y tiempos estimados.
- Medios de pago, bancos y financieras habilitados, y sus condiciones de cuotas.
- Plataforma de e-commerce utilizada y mecanismo de generación del link de checkout.
- ticketReasons y sectores de derivación configurados en el HelpDesk / CRM del cliente.
- Contenido de la Base de Conocimientos: FAQs de talles/atributos, envíos, promociones y otras consultas frecuentes.
- Mensajes fijos (bienvenida, cierre, seguimiento proactivo) y condición de activación de cada uno.
- Nombre y datos de contacto del referente del cliente para resolver puntos pendientes durante la implementación.
