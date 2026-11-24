> Fuente: Alcance_Funcional_Vivero_Mario (https://docs.google.com/document/d/1r6evsRmSNUya4dxmbLIQ_wWZn4sY6HXJNukVoXkPZ8I/edit?usp=drivesdk&ouid=113568244751416058438)

**Snappy Commerce**

# Alcance Funcional — Agente IA Vivero Mario

*Agente IA de Atención al Cliente y Ventas · Documento completado con la configuración actual del bot IA (config v2)*

## Cómo usar este documento

Este documento describe los flujos configurados en el Agente IA de atención al cliente y ventas de Vivero Mario, siguiendo la estructura estándar de Snappy Commerce: cada fila define el flujo, sus activadores y el comportamiento esperado.

Personalidad del bot: rol "both" (atención al cliente + ventas) · idioma/tono argentino. Horario de atención comercial: todos los días de 10:30 a 18:30 hs.

Las notas marcadas con ⚠ señalan puntos pendientes o inconsistencias de configuración que deben confirmarse con el cliente antes de dar por cerrado el alcance.

# Requerimientos Funcionales

## Configuración Global

| Flujo nativo | Qué hace el flujo | Configuración Vivero Mario |
| :-: | :-: | :-: |
| Crear contacto | Si el canal ya aporta el teléfono (WhatsApp) se crea el contacto directamente sin pedir más — solo se solicita si no está disponible. | Teléfono (phone) obligatorio |
| Asistencia | Cuando el usuario pide hablar con una persona, en horario de atención el bot lo deriva directo y envía el mensaje del flujo inHours. Fuera de horario genera un ticket y envía el mensaje del flujo outOfHours. En ambos casos crea/actualiza el contacto. | Escala: En horario<br>Ticket: Fuera de horario<br>Contacto: Siempre<br>Asistencia online: false |
| Ticket | Al abrir un ticket se notifica al usuario. El ID externo es obligatorio pero no se le informa al usuario. Si no se puede crear, responde con el mensaje de fallback. | Notificación: "Vamos a derivar tu consulta con un asesor comercial"<br>Fallback: "No tengo los permisos suficientes como para poder brindarte una respuesta"<br>Sin campos adicionales por default |
| Búsqueda de productos | Incluye las características del producto en la búsqueda contra el catálogo. | includeProductCharacteristicsInQuery: true |
| Encuesta de satisfacción | Al cerrar una conversación, se envía automáticamente una encuesta preguntando cómo fue la atención. | true |
| Carrito / Checkout / Estado de pedido / Factura / Sucursales | Funciones nativas del orquestador para gestión de carrito, checkout asistido, tracking, factura y búsqueda de sucursales. | Deshabilitadas |

*⚠ En el orquestador hoy solo están habilitados: Cafetería, Notificacionesdestock, EnvioCv, Seguimiento, inHours y outOfHours. El resto de los flujos están configurados pero con enabled: false, por lo que no se activan por intención del usuario.*

## Flujos de Pre-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Venta Minorista | Usuario quiere comprar, consulta disponibilidad de un producto, métodos de pago o pide el link de la tienda ("¿tienen esa maceta en stock?", "¿aceptan tarjeta?"). | Asigna sector VentaMinorista, responde con la FAQ de venta minorista y consulta la Base de Conocimientos.<br>⚠ La consulta a la Base de Conocimientos tiene la query vacía.<br>⚠ Flujo configurado pero deshabilitado en el orquestador (enabled: false). |
| Venta Mayorista | Usuario pregunta por compras al por mayor, reventa, volumen o descuento por cantidad. | Asigna sector y responde con la FAQ de venta mayorista.<br>⚠ El sector configurado es "Venta Minorista"; confirmar sector correcto.<br>⚠ Flujo configurado pero deshabilitado en el orquestador (enabled: false). |
| Recomendación de Plantas | Usuario pide asesoramiento sobre qué planta elegir según luz, espacio o estilo ("¿qué planta va bien en un lugar oscuro?"). | Asigna sector Venta Minorista, responde FAQ y genera ticket (questionsBefore) con la notificación "Si preferís que te contacte un asesor, necesitamos saber un poco más sobre tu espacio", solicitando: Iluminación, Ubicación (tipo de espacio) e imágenes del espacio. Setea el topic recomendacinDePlantas.<br>⚠ Flujo configurado pero deshabilitado en el orquestador (enabled: false). |
| Recomendación de Macetas | Usuario pide asesoramiento sobre qué maceta o contenedor elegir (material, tamaño, forma, estilo). | Asigna sector Venta Minorista, responde 6 FAQs de macetas y genera ticket (questionsBefore / característicasDeProductos) con la notificación "Pronto te va a contactar un asesor comercial".<br>⚠ Flujo configurado pero deshabilitado en el orquestador (enabled: false). |
| Notificaciones de stock | Usuario pregunta por un producto del catálogo online que no tiene stock o pide que le avisen cuando vuelva. | Indica que en la tienda online existe la opción de aviso automático de reposición, brinda el link del producto y responde con la FAQ correspondiente. |
| Gift Card | Usuario quiere regalar un valor específico, una gift card u orden de compra. | Responde con la FAQ de Gift Card.<br>⚠ Flujo configurado pero deshabilitado en el orquestador (enabled: false). |
| Entregas a Domicilio | Usuario consulta costo de envío, zonas de cobertura, tiempos o días de entrega. | Responde con la FAQ de envíos.<br>⚠ Flujo configurado pero deshabilitado en el orquestador (enabled: false). |
| Sucursal Castelar | Usuario pregunta explícitamente por la sucursal Castelar / zona oeste (dirección, horarios, retiro, stock). | Asigna sector Castelar y responde con la FAQ de la sucursal.<br>⚠ Flujo configurado pero deshabilitado en el orquestador (enabled: false). |
| Cafetería | Usuario consulta explícitamente por la cafetería: horarios, menú, reservas, ingreso con mascotas. | Asigna sector Cafetería y responde con 2 FAQs de la cafetería. |
| Ambientaciones | Usuario quiere alquilar plantas/macetas de forma temporal para eventos, sesiones de fotos o decoración. | Asigna sector ambientaciones, responde FAQ y genera ticket (quoteRequest) "Ambientaciones y Alquileres para eventos" pidiendo lugar, días y horarios del evento. Notifica que un asesor se contactará en horario 10:30 a 18:30 hs con el presupuesto.<br>⚠ Flujo configurado pero deshabilitado en el orquestador (enabled: false). |
| Presupuestos y Jardinería | Usuario consulta por paisajismo, jardinería a domicilio, mantenimiento, poda o diseño de jardín. | Asigna sector Paisajismo y Jardinería y genera ticket (quoteRequest) solicitando imágenes/planos, medidas, barrio y descripción del proyecto. Luego dispara asistencia informando que un asesor se contactará en horario comercial.<br>⚠ Flujo configurado pero deshabilitado en el orquestador (enabled: false). |
| Problemas técnicos con Tienda Nube | Usuario reporta error en checkout, pago, cupón, carrito o página caída. | Asigna sector venta minorista y envía el mensaje fijo: "Tienda nube puede presentar inestabilidad en el sistema, te pedimos que intentes hacer la compra nuevamente más tarde".<br>⚠ Flujo configurado pero deshabilitado en el orquestador (enabled: false). |
| Derivación a Vendedor / Asistencia comercial | Usuario pide hablar con un asesor o la consulta excede el alcance del bot. | Dispara el flujo de Asistencia: escala en vivo (inHours) o genera ticket (outOfHours) según horario de atención. |

## Flujos de Post-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Estado de Pedido / Factura | Usuario pregunta por el estado de su pedido o solicita la factura. | No disponible: orderTracking y orderInvoice deshabilitados. Se resuelve vía asistencia. |
| Cambios y Devoluciones | Usuario quiere cambiar o devolver un producto ya recibido sin defecto grave. | Asigna sector Reclamos, informa la política (FAQ de políticas) y dispara asistencia con ticket (returns) pidiendo imágenes actuales del producto y comprobante de compra.<br>⚠ Flujo configurado pero deshabilitado en el orquestador (enabled: false). |
| Cancelaciones | Usuario quiere cancelar un pedido que todavía no recibió. | Asigna sector Reclamos, informa la política (FAQ) y genera ticket (cancelOrder) pidiendo número y origen del pedido (tienda online o mostrador Palermo), motivo e imágenes. Notifica disculpas y próximo contacto.<br>⚠ Flujo configurado pero deshabilitado en el orquestador (enabled: false). |
| Reclamos / Ticket | Usuario reporta producto roto, enfermo, dañado o pedido incompleto. | Asigna sector Reclamos y dispara asistencia con ticket (PostVenta) pidiendo número y origen del pedido, motivo del reclamo y cómo quiere resolverlo, e imágenes del estado de la mercadería. Responde con la FAQ de políticas.<br>⚠ Flujo configurado pero deshabilitado en el orquestador (enabled: false). |
| Encuesta de Satisfacción (CSAT) | Se dispara automáticamente al cierre de una conversación. | Envía la encuesta de satisfacción configurada de forma global. |
| Derivación a Soporte Post-venta | La consulta excede el alcance del bot o el usuario pide hablar con un humano. | Dispara el flujo de Asistencia: escala en vivo o genera ticket según horario de atención. |

## Flujos de Sistema (Automatizados)

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Asistencia en horario (inHours) | Sin activador por mensaje del usuario — se dispara al pedir asistencia dentro del horario comercial. | Envía el mensaje fijo: "Te pedimos que esperes hasta que un asesor se pueda comunicar en el horario de atención comercial todos los días de 10,30 a 18,30 hs". |
| Asistencia fuera de horario (outOfHours) | Sin activador por mensaje del usuario — se dispara al pedir asistencia fuera del horario comercial. | Envía un mensaje fijo.<br>⚠ El texto configurado es el placeholder "Mensaje a enviar"; falta definir el mensaje real. |
| Seguimiento Proactivo | Sin activador por mensaje del usuario — automático. | Setea el topic followUp y envía el mensaje fijo "Te puedo ayudar en algo más?".<br>⚠ Sin descripción en el orquestador; definir tiempo/condición de activación. |
| Exclusión por números de teléfono | Interno: el teléfono del contacto está en el listado de números excluidos (líneas internas, testeo). | Asigna sector "contactos frecuentes excluidos" y dispara asistencia con ticket (other).<br>⚠ Flujo configurado pero deshabilitado en el orquestador (enabled: false). |

## Otros Flujos

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Envío de CV | Usuario expresa interés en trabajar en Vivero Mario ("¿están buscando personal?"). | Responde con la FAQ de envío de CV. |
| Proveedores | Persona externa (vendedor, distribuidor) quiere ofrecer productos o servicios. | Asigna sector Proveedores y responde con la FAQ de proveedores.<br>⚠ Flujo configurado pero deshabilitado en el orquestador (enabled: false). |

## Flujos con Lógica Condicional (flujos personalizados)

Los flujos configurados son secuenciales (sector → FAQ → ticket/asistencia) y no presentan ramificaciones internas, por lo que no requieren diagrama de decisión propio.

*⚠ Cualquier comportamiento no cubierto explícitamente en este documento debe confirmarse con el referente técnico del cliente antes de la implementación.*

## Puntos pendientes a validar

- Definir el texto real del mensaje outOfHours (hoy "Mensaje a enviar").
- Confirmar qué flujos deben habilitarse en el orquestador (la mayoría está en enabled: false).
- Venta Mayorista asigna el sector "Venta Minorista": confirmar sector correcto.
- Unificar nombres de sector: "VentaMinorista" / "Venta Minorista" / "venta minorista"; "Castelar " y "contactos frecuentes excluidos " tienen espacio final.
- Agregar descripción y condición de activación al flujo Seguimiento.
- Definir la query de Base de Conocimientos del flujo Venta Minorista (hoy vacía).

## Checklist de Configuración por Cliente

Antes de dar por cerrado el alcance funcional, completar y validar:

- Políticas de cambios, devoluciones y cancelación (FAQ de políticas).
- Zonas de cobertura, costos y tiempos de envío (FAQ Entregas a Domicilio).
- Medios de pago y link de la tienda online (Tienda Nube).
- Sucursales Palermo y Castelar: dirección y horarios.
- ticketReasons en uso: PostVenta, returns, cancelOrder, quoteRequest, questionsBefore, other.
- Sectores de derivación: VentaMinorista, Reclamos, Cafetería, Castelar, ambientaciones, Proveedores, Paisajismo y Jardinería, contactos frecuentes excluidos.
- Contenido de la Base de Conocimientos: FAQs referenciadas por cada flujo cargadas y vigentes.
- Mensajes fijos (inHours, outOfHours, Seguimiento) y condición de activación de cada uno.
- Nombre y datos de contacto del referente de Vivero Mario para resolver puntos pendientes.
