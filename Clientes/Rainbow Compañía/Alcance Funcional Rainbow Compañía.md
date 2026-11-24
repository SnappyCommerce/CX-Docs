> Fuente: Analisis_Funcional_Rainbow_Gen6_v3 (https://docs.google.com/document/d/1H07zZGG1VXFlIXA0_yASUSCqFTyvXhPf-C6Vl8w1Q5U/edit?usp=drivesdk&ouid=113568244751416058438)

**Snappy Commerce**

# Análisis Funcional — Rainbow, Agente IA

*Maia · Relevamiento del comportamiento vigente en producción (Gen 5) como base para la migración a Gen 6*

## Cómo usar este documento

Este documento releva el comportamiento actual del Agente IA de Rainbow — flujos nativos, flujos personalizados y configuración global — siguiendo la estructura del alcance funcional estándar de Snappy Commerce. Cada fila define el flujo, sus activadores y el comportamiento esperado.

Sirve para que Producto valide el comportamiento vigente, detecte vacíos funcionales e inconsistencias de configuración, y tenga documentado cada flujo antes de que pase de la UI de Snappy (Gen 5) a código TypeScript mantenido por Producto (Gen 6). La experiencia del usuario final no cambia con la migración.

*Las notas marcadas con ⚠ señalan flujos desactivados o puntos a confirmar.*

## Requerimientos Funcionales

### Configuración Global

| Flujo nativo | Qué hace el flujo | Valor configurado |
| :-: | :-: | :-: |
| Crear contacto — Campos a solicitar al usuario | Para crear un contacto nuevo, el agente IA solicita de base estos datos de identificación del usuario. | firstName, phone, email |
| Comenzar checkout — Requisitos de información de contacto | Para generar un checkout alcanza con nombre y email — no pide teléfono en ese paso. | firstName, email |
| Asistencia | Toda vez que un flujo dispara "assistance", siempre se piden datos de contacto. Si es horario de atención se escala en vivo; si no, se genera un ticket y se dispara el flujo outOfHours. | contact: always · escalate: in_hours · ticket + outOfHours: out_of_hours |
| Encuesta de satisfacción | Encuesta de satisfacción automática (CSAT) al cierre de conversaciones. | ACTIVADO |
| Estado de Pedido | Flujo nativo de consulta de estado de pedido. | ACTIVADO - Búsqueda por ID y por mail |
| Búsqueda de producto nativa | Búsqueda y recomendación ejecutada contra el catálogo del cliente. | ACTIVADO — incluye características del producto en la consulta |
| Sucursales / Branches | Información de sucursales físicas mediante el módulo nativo de Snappy. | DESACTIVADO - Responde con conocimiento |
| Promociones | Promociones activas cargadas en Labs. | DESACTIVADO - Responde con conocimiento |
| Factura de pedido (orderInvoice) | Facturación nativa de pedido. | No está disponible para TN (responde con conocimiento) - pendiente evaluar cómo automatizar el envío de facturas con FInnegans |
| Sucursales en producto (productBranches) | Disponibilidad de producto por sucursal. | DESACTIVADO - Responde con conocimiento |
| País | País de operación del bot. | AR |
| Idioma / Personalidad | Modo de respuesta y dialecto del agente. | Ventas y soporte / Español rioplatense |

### Flujos de Pre-Venta

| Flujo | Activadores | Comportamiento esperado |
| :-: | :-: | :-: |
| Saludo | hola · buen día · buenas tardes · buenas · hola ¿cómo están? · quiero más información<br>Saludo acompañado de una consulta concreta → flujo correspondiente | Envía el mensaje de bienvenida: "¡Hola! Soy Maia, la asistente virtual de Rainbow y estoy para ayudarte." y que le pregunte al usuario en que le puede ayudar |

### Flujos de Post-Venta

| Flujo | Activadores | Comportamiento esperado |
| :-: | :-: | :-: |
| Cambios | quiero hacer un cambio · ¿cómo hago para cambiar algo? · quiero devolver un producto · ¿puedo cambiar el talle? · ¿cuál es la política de cambios?<br>Producto defectuoso, roto o incompleto → comportamiento default · Cancelar antes de recibir → CancelarPedido · Compra por Mercado Libre → postventaMercadoLibre | Muestra la FAQ de política de cambios y devoluciones + solicita asistencia humana (ticketReason: returns). Datos del ticket:<br>- Número de Venta/Factura de compra<br>- Foto/video<br>- Descripción del problema |
| Garantía | ¿cuánto tiempo de garantía tiene? · ¿qué cubre la garantía? · ¿cómo hago válida la garantía? · ¿hay falla cubierta por garantía?<br>Producto llegó roto, incompleto o equivocado → comportamiento default · Cancelar pedido → CancelarPedido | Muestra la FAQ de garantía + solicita asistencia humana (ticketReason: PostVenta). Campos adicionales solicitados: Número de venta · Fecha de compra · Canal de venta (Mercado Libre o Tienda Nube) · Foto o video de la falla. |
| CancelarPedido | quiero cancelar mi pedido · ¿cómo cancelo? · necesito cancelar la compra · me arrepentí, quiero cancelar antes de que salga · ¿se puede cancelar?<br>Devolución de producto ya recibido → Cambios · Problema con pedido recibido → comportamiento default | Muestra la FAQ de cancelación de pedido + abre un ticket (reason: order · subreason: cancelOrder). Notificación al cliente: "Gracias por la información, serás contactado a la brevedad"<br>2 caminos: Si el pedido ya fue recibido crea ticket con los siguientes campos:<br>- Número de Venta/Factura de compra<br>- Foto/video<br>- Motivo de cancelación<br>Si el pedido aun no llego solicita asistencia para que puedan validar el estado |
| PedidoDemorado | no me llegó el pedido · mi pedido está demorado · ya pasó la fecha de entrega y no recibí nada · hace días que espero mi compra · el seguimiento no se actualiza<br>Producto llegó roto, incompleto o equivocado → comportamiento default · Falla con el uso → Garantía · Cancelar pedido → CancelarPedido | Envía el mensaje exacto: "¡Gracias por avisarnos! Ya derivamos tu consulta al equipo de Post Venta. En breve uno de nuestros agentes te va a atender de forma personalizada para revisar el estado de tu pedido. Por favor, aguardá unos minutos." + solicita asistencia humana (ticketReason: PostVenta). Fuera de atención que solicite el número de venta/factura de compra |
| postventaMercadoLibre | compré por Mercado Libre y tengo un problema · me llegó incompleto, lo compré en ML · quiero devolver algo que compré en ML · ¿cómo hago un reclamo en ML? · compré por ML y me arrepentí<br>Inconveniente con compra directa en Rainbow → comportamiento default o Cambios · No menciona Mercado Libre → comportamiento default | El agente IA clasifica el inconveniente con un prompt estructurado: (1) Devolución por decisión propia → deriva al canal de ML. (2) Pieza o accesorio faltante → control de horario. (3) Falla o producto roto → control de horario. (4) Motivo no claro → pregunta al cliente qué ocurrió y vuelve a clasificar: si coincide con 1 deriva a ML; si coincide con 2 o 3, o no cumple ningún criterio, va al control de horario. No promete soluciones antes de que el equipo revise. Control de horario: dentro de horario (lunes a viernes, 9:30 a 16:30 hs) solicita asistencia humana para que un asesor gestione el caso. Fuera de horario pide foto/video + número de compra y crea el ticket con esos campos (flujo outOfHours). |

### Flujos de Sistema (Automatizados)

| Flujo | Activadores | Comportamiento esperado |
| :-: | :-: | :-: |
| Saludo / Bienvenida | Sin activador por mensaje del usuario — se dispara desde automatizaciones al iniciar el primer contacto. | Activo — envía mensaje de bienvenida de Maia y despliega el menú de opciones principal. |
| outOfHours | Se dispara automáticamente cuando se solicita asistencia humana fuera del horario de atención. | Activo — informa el horario de atención (lunes a viernes de 9:30 a 16:30 hs) y deja el ticket abierto para gestión al día hábil siguiente. |
| Seguimiento Proactivo | Sin activador por mensaje del usuario — automático tras un tiempo configurable de inactividad. | ⚠ No relevado — confirmar con el cliente. |
| Cierre por Inactividad + CSAT | Sin activador por mensaje del usuario — automático al quedar la conversación inactiva. | ⚠ No relevado — confirmar con el cliente. |

### Flujos con Lógica Condicional

Los siguientes flujos tienen ramificaciones internas y cuentan con un diagrama de decisión propio en la sección Diagramas de Flujo:

- postventaMercadoLibre: Ramifica según el tipo de inconveniente reportado por el cliente (devolución por decisión propia / pieza faltante / falla / motivo no claro). Cada rama define si el caso se resuelve directamente con Rainbow o se deriva al canal de Mercado Libre.

## Diagramas de Flujo

### Diagrama 1 — postventaMercadoLibre

*Referencias: el horario de atención es de lunes a viernes de 9:30 a 16:30 hs. Fuera de horario se solicita foto/video + número de compra y se crea el ticket con esos campos (flujo outOfHours).*

*(El diagrama es una imagen en el documento fuente y no se exporta como texto.)*

## Requerimientos No Funcionales

| Categoría | Requerimiento |
| :-: | :-: |
| Lenguaje | Español rioplatense, tono amigable y profesional. Voseo en todos los verbos (querés, podés, decime, tenés). Sin jerga informal, sin trato de usted, sin español neutro o de España. |
| Testing | El código/JSON debe pasar el test suite estándar de Snappy (umbral 85%+) antes de cualquier subida a producción. |
| Conversaciones | El agente debe contar con contexto de las conversaciones previas de cada contacto y poder responder usando esa información (ej.: reconocer a qué ticket se refiere el usuario en una conversación nueva). |
| Comportamiento — Fallback | Cuando no cuente con información suficiente en Conocimientos, debe ofrecer solicitar asistencia de un operador humano. |
| Comportamiento — Proactividad y reencauce | Ante silencio de 5 minutos en un flujo de venta en curso, reactiva la conversación con un mensaje de seguimiento. Ante desvío de tema, responde la consulta puntual y luego reencauza hacia la venta en curso, sin abandonarla. |

## Criterios de Aceptación

- Cada flujo migrado a Gen 6 reproduce el comportamiento de Gen 5 documentado en este alcance.
- Maia pasa el test suite estándar de Snappy con un umbral de 85%+.
- El agente deriva a un operador humano cuando no disponga de información en sus Conocimientos para resolver la consulta.
- El agente clasifica correctamente los reclamos de Mercado Libre según el tipo de problema reportado.
- Se incorporan todos los requerimientos no funcionales detallados en este documento.
