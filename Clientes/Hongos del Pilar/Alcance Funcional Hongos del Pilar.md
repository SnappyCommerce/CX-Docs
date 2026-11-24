> Fuente: Alcance_Funcional_HongosDelPilar (https://docs.google.com/document/d/15IC21_IM0WRJOF_k_oQ1i7lwLAmsPb6n6Ohrz0bVRQg/edit?usp=drivesdk&ouid=113568244751416058438)

**Snappy Commerce**

**Alcance Funcional Genérico — Agente IA de Atención al Cliente y Ventas**

*Documento base para nuevos clientes · Plantilla de referencia completada con la configuración estándar del bot IA*

## Cómo usar este documento

Este documento describe la estructura estándar de flujos de un Agente IA de atención al cliente y ventas de Snappy Commerce. Sirve como punto de partida para el relevamiento (discovery) de un cliente nuevo: cada fila define el flujo, sus activadores típicos y el comportamiento esperado en términos generales.

Los textos entre corchetes y las notas marcadas con ⚠ señalan puntos que dependen de la política, integración o contenido propio de cada cliente y deben completarse durante el discovery. Los nombres de los flujos pueden mantenerse o adaptarse a la nomenclatura interna del cliente.

# Requerimientos Funcionales

## Configuración Global

| Flujo nativo | Qué hace el flujo | Configuración por default |
| :-: | :-: | :-: |
| Crear contacto | Si el canal ya aporta un dato de contacto (teléfono en WhatsApp, ID de Instagram/Facebook) o el usuario da un email, se crea el contacto directamente sin pedir más — solo se solicita si no hay ningún dato disponible. | Teléfono / email / WhatsApp ID / Facebook ID — alcanza con uno solo |
| Asistencia | Cuando el usuario pide hablar con una persona, en horario de atención el bot lo deriva directo, sin pedir datos. Fuera de horario, en cambio, genera un ticket de soporte y ahí sí pide los datos de contacto. | Datos de contacto: Siempre<br>Escala: En horario<br>Ticket: Fuera de horario |
| Ticket | Cuando se abre un ticket, el bot pide el motivo de la consulta antes de generarlo (si no lo tiene) | Motivo obligatorio |
| Encuesta de satisfacción | Al cerrar una conversación, se envía automáticamente una encuesta preguntando cómo fue la atención. | true |

## Flujos de Pre-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Búsqueda y Recomendación de Producto | Usuario pregunta por un producto, categoría, talle, color o atributo ("tenés zapatillas rojas talle 40?"). | Le devuelve información de compra mínima y frecuencia de compra necesaria + transferencia a asesor para poder evaluarlo como posible cliente |
| Consulta de Precio / Stock | Usuario pregunta precio, disponibilidad o stock de un producto puntual. | Responde con conocimiento comentando al usuario que la información de Stock y precios no está disponible y transfiere a un asesor para que pueda analizar el caso |
| Promociones | Usuario pregunta por descuentos, promos, cupones o "qué ofertas tienen". | Envía el mensaje nativo de Promociones con las promociones activas configuradas en Labs. |
| Información de Sucursales / Branches | Usuario pregunta dirección, horarios o sucursal más cercana. | Envía un storeSearch con el mapa y datos de las sucursales cargadas en Labs (dirección, horario, teléfono). (Inactivo) Responde con conocimiento |
| FAQ Comercial (talles, envíos, medios de pago) | Usuario consulta guía de talles, costos/tiempos de envío, medios de pago o cuotas. | Responde con el contenido de la Base de Conocimientos (FAQ) configurada para cada tema. [Contenido a cargar por cliente]. |
| Checkout Asistido | Usuario quiere comprar / pide el link de compra ("quiero comprar", "pasame el link"). | Solicita los datos de contacto configurados (firstName, email) y genera el link de checkout de la plataforma del cliente.<br>(Inactivo) No tienen ecommerce para checkout |
| Checkout Asistido — Gestión del Carrito | Usuario quiere agregar, quitar o modificar productos en un carrito ya iniciado. | Actualiza el carrito vía integración según la acción solicitada, validando stock de la combinación elegida. (Inactivo) No tienen ecommerce para checkout |
| Derivación a Vendedor / Asistencia comercial | Usuario pide hablar con un asesor de ventas o la consulta excede el alcance del bot. | Dispara el flujo de asistencia: pide datos de contacto y escala en vivo o genera ticket según horario de atención. |

## Flujos de Post-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Estado de Pedido | Usuario pregunta por el estado de su pedido/envío, con o sin número de orden. | Utilizan una aplicación de desarrollo propio (BAPS) para la gestión de estados de pedidos. Esta aplicación envía mensajes personalizados según el estado. Analizar implementación entre equipos |
| Factura del pedido | Usuario solicita la factura de una compra. | Deriva a un asesor para que pueda buscarle la información de su factura al cliente. |
| Cambios y Devoluciones | Usuario quiere cambiar o devolver un producto ya comprado. | Informa la política de cambios/devoluciones del cliente y deriva al proceso de creación de tickets del flujo "Reclamos" [Política a definir por cliente]. |
| Cancelaciones | Usuario quiere cancelar un pedido. | Informa condiciones de cancelación y deriva al proceso de creación de tickets del flujo "Reclamos" |
| Garantía | Usuario consulta por cobertura o activación de garantía de un producto. | Informa plazo y condiciones de garantía configuradas. [Política a definir por cliente]. |
| Reclamos / Ticket | Usuario reporta un problema (producto dañado, faltante, error de facturación, etc.). | Genera un ticket en el HelpDesk/CRM con el ticketReason y ticketSubreason correspondientes, solicitando los datos requeridos.<br>Datos requeridos: Tomarlos desde el Form |
| Encuesta de Satisfacción (CSAT) | Se dispara automáticamente al cierre de una gestión o conversación. | Envía la encuesta de satisfacción configurada de forma global. |
| Derivación a Soporte Post-venta | La consulta post-venta excede el alcance del bot o el usuario pide hablar con un humano. | Dispara el flujo de asistencia: escala en vivo o genera ticket según horario de atención. |

## Flujos de Sistema (Automatizados)

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Mensaje de Bienvenida | Sin activador por mensaje del usuario — se dispara desde automatizaciones al iniciar el primer contacto. | Envía un mensaje fijo (y estacional, si aplica) definido por el cliente. [Contenido, tono y condición de activación a configurar por cliente]. |
| Seguimiento Proactivo | Sin activador por mensaje del usuario — automático tras un tiempo configurable de inactividad. | Envía un mensaje de seguimiento preguntando cómo le fue al usuario con la gestión. [Tiempo de inactividad y activación/desactivación a configurar por cliente]. |
| Cierre por Inactividad + CSAT | Sin activador por mensaje del usuario — automático al quedar la conversación inactiva. | Envía mensaje de cierre y dispara la Encuesta de Satisfacción (CSAT). [Tiempo de inactividad y activación/desactivación a configurar por cliente]. |
| Notificaciones / Broadcast | Se dispara desde un envío de notificación/broadcast programado en Labs. | Envía el contenido definido en la notificación y puede setear custom contexts o asignar la conversación a un sector/usuario. |

## Flujos con Lógica Condicional

Los siguientes flujos tienen ramificaciones internas y requieren un diagrama de decisión propio, construido durante el discovery de cada cliente. Se listan aquí los puntos de ramificación genéricos que ese diagrama debe cubrir:

- Búsqueda y Recomendación de Producto: Ramifica según la cantidad de resultados encontrados y si el catálogo del cliente tiene facetas/filtros configurados.
- Checkout Asistido: Ramifica según si ya hay nombre y mail en el contexto de la conversación, y si existe ambigüedad de grafía en esos datos.
- Checkout Asistido — Gestión del Carrito: Ramifica según el tipo de acción (agregar / borrar / modificar) y si hay stock disponible en la nueva combinación.
- Cambios y Devoluciones: [Puntos de ramificación a definir en el discovery — plazos, categoría, canal de gestión].
- Cancelaciones: Ramifica según si el usuario acepta o no derivar el caso a un asesor.
- Estado de Pedido: Ramifica según el statusCode devuelto por la integración (en tránsito, entregado, con demora, etc.) y si hay tracking disponible.

*⚠ Cualquier comportamiento no cubierto explícitamente en este documento debe confirmarse con el referente técnico del cliente antes de la implementación.*

# Diagramas de Flujo (solo tienen que estar las que son personalizadas)

Representación visual de la lógica de decisión de los flujos condicionales listados arriba. Son diagramas base para discovery: los rótulos entre corchetes o marcados con ⚠ en las tablas anteriores deben reflejarse aquí una vez definidos con el cliente.

*Versión editable de estos diagramas (formas nativas, sin necesidad de recrearlos):* [*Diagramas_Flujo_Bot_IA.pptx*](https://docs.google.com/presentation/d/1THBgyODVB9IX3UfAmipArhx6RDR1ZC11/edit?slide=id.p1#slide=id.p1)*, adjunto junto con este documento.*

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
