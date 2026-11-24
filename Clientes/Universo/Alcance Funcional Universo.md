> Fuente: Alcance_Funcional_Universo_Bot_IA.docx (https://drive.google.com/file/d/1ZcvRaeCAdsBQP2B_gW7-S-O8SAQA2aYc/view?usp=drivesdk)

Snappy Commerce

# Alcance Funcional Universo — Agente IA de Atención al Cliente y Ventas

## Cómo usar este documento

Este documento describe la estructura estándar de flujos de un Agente IA de atención al cliente y ventas de Snappy Commerce. Sirve como punto de partida para el relevamiento (discovery) de un cliente nuevo: cada fila define el flujo, sus activadores típicos y el comportamiento esperado en términos generales.

Los textos entre corchetes y las notas marcadas con ⚠ señalan puntos que dependen de la política, integración o contenido propio de cada cliente y deben completarse durante el discovery. Los nombres de los flujos pueden mantenerse o adaptarse a la nomenclatura interna del cliente.

# Requerimientos Funcionales

## Configuración Global

| Flujo nativo | Qué hace el flujo | Configuración por default |
| :-: | :-: | :-: |
| Crear contacto | Si el canal ya aporta un dato de contacto (teléfono en WhatsApp, ID de Instagram/Facebook) o el usuario da un email, se crea el contacto directamente sin pedir más — solo se solicita si no hay ningún dato disponible. | Teléfono / email / WhatsApp ID / Facebook ID — alcanza con uno solo |
| Asistencia | Cuando el usuario pide hablar con una persona, en horario de atención el bot lo deriva directo, sin pedir datos. Fuera de horario, en cambio, genera un ticket de soporte y ahí sí pide los datos de contacto. | Escala: En horario<br>Ticket: Fuera de horario - |
| Ticket | Cuando se abre un ticket, el bot pide el motivo de la consulta antes de generarlo (si no lo tiene) | Motivo obligatorio |
| Encuesta de satisfacción | Al cerrar una conversación, se envía automáticamente una encuesta preguntando cómo fue la atención. | true |

## Flujos de Pre-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Búsqueda y Recomendación de Producto | Usuario pregunta por un producto, categoría, talle, color o atributo ("tenés zapatillas rojas talle 40?"). | ACTIVO => Integración vtex |
| Consulta de Precio / Stock en sucursal | Usuario pregunta precio, disponibilidad o stock de un producto puntual. | ACTIVO => Debe usar el flujo nativo de stock en sucursal |
| Promociones | Usuario pregunta por descuentos, promos, cupones o "qué ofertas tienen". | Envía el mensaje nativo de Promociones con las promociones activas configuradas en Labs. |
| Información de Sucursales / Branches | Usuario pregunta dirección, horarios o sucursal más cercana. | Envía un storeSearch con el mapa y datos de las sucursales cargadas en Labs (dirección, horario, teléfono). |
| FAQ Comercial (talles, envíos, medios de pago) | Usuario consulta guía de talles, costos/tiempos de envío, medios de pago o cuotas. | Responde con el contenido de la Base de Conocimientos (FAQ) configurada para cada tema. [Contenido a cargar por cliente]. |
| Checkout Asistido | Usuario quiere comprar / pide el link de compra ("quiero comprar", "pasame el link"). | Activo |
| Checkout Asistido — Gestión del Carrito | Usuario quiere agregar, quitar o modificar productos en un carrito ya iniciado. | Activo |
| Compra mayorista | Usuario indica que quiere comprar por mayor | Responde con FAQ |
| Compra automotor | Usuario indica que quiere hacer una compra de pintura para vehiculos | Responde con FAQ |
| Compra para edificios | Usuario indica que requiere atención o corporativa o hacer una compra para grandes obras | Responde con FAQ |
| Derivación a Vendedor / Asistencia comercial | Usuario pide hablar con un asesor de ventas o la consulta excede el alcance del bot. | Dispara el flujo de asistencia: pide datos de contacto y escala en vivo o genera ticket según horario de atención. |

## Flujos de Post-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Estado de Pedido | Usuario pregunta por el estado de su pedido/envío, con o sin número de orden. | Envía un mensaje orderStatus con el estado, fecha estimada y tracking (si está disponible) vía la integración del cliente. Operador logístico: andreani. Retiro en sucursal: Tienen dos tipos de retiro en sucursal. wpp a enviar notificación por estado de pedido (retiro en sucursal) |
| Cambios y Devoluciones | Usuario quiere cambiar o devolver un producto ya comprado. | Responde con FAQ |
| Cancelaciones | Usuario quiere cancelar un pedido. | Informa condiciones de cancelación y pregunta si acepta derivar el caso a un asesor; si acepta, escala; si no, cierra el flujo. Chequear la orden y dependiendo del estado responde distinto (estado a facturado: despachado: retiro en sucursal - facturado: definir qué hacer en cada estado). A DEFINIR |
| Garantía | Usuario consulta por cobertura o activación de garantía de un producto. | Responde con FAQ |
| Reclamos / Ticket | Usuario reporta un problema (producto dañado, faltante, error de facturación, etc.). | Genera un ticket en el HelpDesk/CRM con el ticketReason y ticketSubreason correspondientes, solicitando los datos requeridos. |
| Factura | El usuario indica que no le mandaron la factura o solicita el envío de la misma. | Genera un ticket en el HelpDesk/CRM con el ticketReason y ticketSubreason correspondientes, solicitando los datos de contacto. |
| Encuesta de Satisfacción (CSAT) | Se dispara automáticamente al cierre de una gestión o conversación. | Envía la encuesta de satisfacción configurada de forma global. |
| Derivación a Soporte Post-venta | La consulta post-venta excede el alcance del bot o el usuario pide hablar con un humano. | Dispara el flujo de Asistencia: escala en vivo o genera ticket según horario de atención. |

## Flujos de Sistema (Automatizados)

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Mensaje de Bienvenida | Sin activador por mensaje del usuario — se dispara desde automatizaciones al iniciar el primer contacto. | Envía un mensaje fijo (y estacional, si aplica) definido por el cliente. [Contenido, tono y condición de activación a configurar por cliente]. |
| Seguimiento Proactivo | Sin activador por mensaje del usuario — automático tras un tiempo configurable de inactividad. | Envía un mensaje de seguimiento preguntando cómo le fue al usuario con la gestión. [Tiempo de inactividad y activación/desactivación a configurar por cliente]. |
| Cierre por Inactividad + CSAT | Sin activador por mensaje del usuario — automático al quedar la conversación inactiva. | Envía mensaje de cierre y dispara la Encuesta de Satisfacción (CSAT). [Tiempo de inactividad y activación/desactivación a configurar por cliente]. |
| Notificaciones / Broadcast | Se dispara desde un envío de notificación/broadcast programado en Labs. | Envía el contenido definido en la notificación y puede setear custom contexts o asignar la conversación a un sector/usuario. |

## Checklist de Configuración por Cliente

Antes de dar por cerrado el alcance funcional de un cliente nuevo, completar y validar:

Cancelación: falta definir que hacer para cada uno de los estados de la integración

Garantía: Cargar el conocimiento, dependiendo del producto cambia el conocimiento, falta definir que es para cada caso

## Segunda etapa (Pendiente)

Actualización del flujo de búsqueda de producto para implementar una indagación profunda previa a compartir el carrusel de productos al usuario.
Esto se hará en base al archivo compartido por el equipo universo (todavía pendiente).
