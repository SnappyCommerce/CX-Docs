> Fuente: Alcance_Funcional_Pastalinda_Bot_IA.docx (https://drive.google.com/file/d/1AsLUlB5I9UForRz__pdcUDZcjjDnpx8g/view?usp=drivesdk)

Snappy Commerce

Alcance Funcional Genérico — Agente IA de Atención al Cliente y Ventas

Documento base para nuevos clientes · Plantilla de referencia completada con la configuración estándar del bot IA

## Cómo usar este documento

Este documento describe la estructura estándar de flujos de un Agente IA de atención al cliente y ventas de Snappy Commerce. Sirve como punto de partida para el relevamiento (discovery) de un cliente nuevo: cada fila define el flujo, sus activadores típicos y el comportamiento esperado en términos generales.

Los textos entre corchetes y las notas marcadas con ⚠ señalan puntos que dependen de la política, integración o contenido propio de cada cliente y deben completarse durante el discovery. Los nombres de los flujos pueden mantenerse o adaptarse a la nomenclatura interna del cliente.

# Requerimientos Funcionales

## Configuración Global

| Flujo nativo | Qué hace el flujo | Configuración por default |
| :-: | :-: | :-: |
| Crear contacto | Si el canal ya aporta un dato de contacto (teléfono en WhatsApp, ID de Instagram/Facebook) o el usuario da un email, se crea el contacto directamente sin pedir más — solo se solicita si no hay ningún dato disponible. | Nombre, teléfono y email. (Obligatorios) |
| Asistencia | Cuando el usuario pide hablar con una persona, en horario de atención el bot lo deriva directo, sin pedir datos. Fuera de horario, en cambio, genera un ticket de soporte y ahí sí pide los datos de contacto. | Escala: En horario<br>Ticket: Fuera de horario |
| Ticket | Cuando se abre un ticket, el bot pide el motivo de la consulta antes de generarlo (si no lo tiene) | Motivo obligatorio |
| Encuesta de satisfacción | Al cerrar una conversación, se envía automáticamente una encuesta preguntando cómo fue la atención. | true |

## Flujos de Pre-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Búsqueda y Recomendación de Producto | Usuario pregunta por un producto, categoría, talle, color o atributo ("tenés zapatillas rojas talle 40?"). | Ejecuta productSearch contra el catálogo del cliente y devuelve un carousel de resultados. Si no hay resultados o hay demasiada ambigüedad, repregunta o deriva a asistencia.<br>Diagrama 1<br>TN |
| Consulta de Precio / Stock | Usuario pregunta precio, disponibilidad o stock de un producto puntual. | Responde con la información obtenida vía integración (VTEX u otra plataforma configurada). \[Fuente de stock/precio a confirmar por cliente\].<br>TN |
| Promociones | Usuario pregunta por descuentos, promos, cupones o "qué ofertas tienen". | Envía el mensaje nativo de Promociones con las promociones activas configuradas en Labs. |
| Información de Sucursales / Branches | Usuario pregunta dirección, horarios o sucursal más cercana. | Si es del interior del pais responde con FAQ: ¿Tienen locales en el interior?<br>Si no responder con FAQ: ¿Dónde está el showroom? |
| FAQ Comercial (talles, envíos, medios de pago) | Usuario consulta guía de talles, costos/tiempos de envío, medios de pago o cuotas. | Responde con el contenido de la Base de Conocimientos (FAQ) configurada para cada tema. \[Contenido a cargar por cliente\]. |
| Checkout Asistido | Usuario quiere comprar / pide el link de compra ("quiero comprar", "pasame el link"). | Solicita los datos de contacto configurados (firstName, email) y genera el link de checkout de la plataforma del cliente.<br>Diagrama |
| Checkout Asistido — Gestión del Carrito | Usuario quiere agregar, quitar o modificar productos en un carrito ya iniciado. | Actualiza el carrito vía integración según la acción solicitada, validando stock de la combinación elegida.<br>Diagrama |
| Derivación a Vendedor / Asistencia comercial | Usuario pide hablar con un asesor de ventas o la consulta excede el alcance del bot. | Dispara el flujo de asistencia: pide datos de contacto y escala en vivo o genera ticket según horario de atención. |

## Flujos de Post-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Estado de Pedido | Usuario pregunta por el estado de su pedido/envío, con o sin número de orden. | Envía un mensaje orderStatus con el estado, fecha estimada y tracking (si está disponible) vía la integración del cliente.<br>Falta que nos compartan el token Shipnow |
| Factura del pedido | Usuario solicita la factura de una compra. | Responder con la faq "Dónde está mi factura?" |
| Cambios y Devoluciones | Usuario quiere cambiar o devolver un producto ya comprado. | Informa la política de cambios/devoluciones y transfiere con un asesor. |
| Cancelaciones | Usuario quiere cancelar un pedido. | Transferir con un asesor |
| Garantía | Usuario consulta por cobertura o activación de garantía de un producto. | Transferir con un asesor |
| Reclamos | Usuario reporta un problema (producto dañado, faltante, error de facturación, etc.). | Transferir con un asesor |
| Encuesta de Satisfacción (CSAT) | Se dispara automáticamente al cierre de una gestión o conversación. | Envía la encuesta de satisfacción configurada de forma global. |
| Derivación a Soporte Post-venta | La consulta post-venta excede el alcance del bot o el usuario pide hablar con un humano. | Dispara el flujo de Asistencia: escala en vivo o genera ticket según horario de atención. |

## Flujos de Sistema (Automatizados)

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Mensaje de Bienvenida | Mensaje de saludo del chatbot | ¡Hola! Soy el asistente de Pastalinda \[emoji no recuperable en la fuente\] Contame qué estás buscando: productos, promos, tu pedido o service. Si necesitás hablar con alguien del equipo, avisame y te derivo. |
| Seguimiento Proactivo | Sin activador por mensaje del usuario — automático tras un tiempo configurable de inactividad. | Envía un mensaje de seguimiento preguntando cómo le fue al usuario con la gestión. \[Tiempo de inactividad y activación/desactivación a configurar por cliente\]. |
| Cierre por Inactividad + CSAT | Sin activador por mensaje del usuario — automático al quedar la conversación inactiva. | Envía mensaje de cierre y dispara la Encuesta de Satisfacción (CSAT). \[Tiempo de inactividad y activación/desactivación a configurar por cliente\]. |
| Notificaciones / Broadcast | Se dispara desde un envío de notificación/broadcast programado en Labs. | Envía el contenido definido en la notificación y puede setear custom contexts o asignar la conversación a un sector/usuario. |

## Flujos con Lógica Condicional (flujos personalizados)

Los siguientes flujos tienen ramificaciones internas y requieren un diagrama de decisión propio, construido durante el discovery de cada cliente. Se listan aquí los puntos de ramificación genéricos que ese diagrama debe cubrir:

⚠ Cualquier comportamiento no cubierto explícitamente en este documento debe confirmarse con el referente técnico del cliente antes de la implementación.

# Diagramas de Flujo (solo tienen que estar las que son personalizadas)

Representación visual de la lógica de decisión de los flujos condicionales listados arriba. Son diagramas base para discovery: los rótulos entre corchetes o marcados con ⚠ en las tablas anteriores deben reflejarse aquí una vez definidos con el cliente.

Versión editable de estos diagramas (formas nativas, sin necesidad de recrearlos): [Diagramas\_Flujo\_Bot\_IA.pptx](https://docs.google.com/presentation/d/1THBgyODVB9IX3UfAmipArhx6RDR1ZC11/edit?slide=id.p1#slide=id.p1), adjunto junto con este documento.

## Checklist de Configuración por Cliente

Antes de dar por cerrado el alcance funcional de un cliente nuevo, completar y validar:

Políticas de cambios y devoluciones (plazos, excepciones por categoría/promoción, canal de gestión).

Política de garantía (plazo de cobertura, condiciones de aceptación).

Transportistas / métodos de envío, zonas de cobertura y tiempos estimados.

Medios de pago, bancos y financieras habilitados, y sus condiciones de cuotas.

Plataforma de e-commerce utilizada y mecanismo de generación del link de checkout.

ticketReasons y sectores de derivación configurados en el HelpDesk / CRM del cliente.

Contenido de la Base de Conocimientos: FAQs de talles/atributos, envíos, promociones y otras consultas frecuentes.

Mensajes fijos (bienvenida, cierre, seguimiento proactivo) y condición de activación de cada uno.

Nombre y datos de contacto del referente del cliente para resolver puntos pendientes durante la implementaci

\[El texto devuelto por Drive termina cortado en "implementaci"; el resto del documento no estaba disponible.\]
