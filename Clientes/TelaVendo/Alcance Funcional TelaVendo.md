> Fuente: Alcance_Funcional_Telavendo.docx (https://drive.google.com/file/d/1-AsjaINnxJAZlp7Pf_5TMYUQYMWsfdgO/view?usp=drivesdk)

**Snappy Commerce**

# Alcance Funcional Telavendo — Agente IA de Atención al Cliente y Ventas

## Cómo usar este documento

Este documento describe la estructura estándar de flujos de un Agente IA de atención al cliente y ventas de Snappy Commerce. Sirve como punto de partida para el relevamiento (discovery) de un cliente nuevo: cada fila define el flujo, sus activadores típicos y el comportamiento esperado en términos generales.

Los textos entre corchetes y las notas marcadas con ⚠ señalan puntos que dependen de la política, integración o contenido propio de cada cliente y deben completarse durante el discovery. Los nombres de los flujos pueden mantenerse o adaptarse a la nomenclatura interna del cliente.

# Requerimientos Funcionales

## Configuración Global

| Flujo nativo | Qué hace el flujo | Configuración en Telavendo |
| :-: | :-: | :-: |
| Crear contacto | Registra al cliente antes de escalar la conversación o abrir un caso. | Requiere firstName y phone. Ambos obligatorios. |
| Asistencia | Deriva la conversación a una persona cuando el usuario lo pide o cuando un flujo lo dispara. | Pide datos de contacto siempre. Escala en vivo en horario de atención; fuera de horario genera ticket. |
| Ticket | Abre un caso en el HelpDesk. | Motivo obligatorio. Cada flujo define su reason, subreason y campos adicionales. |
| Estado de pedido | Consulta la orden contra la integración del cliente. | true / datos según integración (orderStatus providers). Búsqueda por número de pedido |
| Checkout asistido | Genera el link de pago de la plataforma del cliente. | Pide firstName, lastName y email. No pide dirección de envío. Desactivado |
| Encuesta de satisfacción | Se envía al cerrar una conversación. | Activado |
| Instagram | desactivado | |
| Mercado libre | desactivado | |

# Flujos de Pre-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Saludo<br>Saludo | El cliente inicia la conversación saludando o sin un objetivo claro.<br>Ejemplos: "Hola" · "Buenos días" · "Buenas" · Primer mensaje sin intención identificable.<br>No activar: Si saluda y además hace una consulta concreta, va al flujo de esa consulta. | 1. notify — Envía el mensaje de bienvenida textual, sin ofrecer ayuda adicional al final: "¡Hola! Soy la IA de TELAS. Estoy para ayudarte con todo lo que necesites. Contame qué estás buscando ⚡".<br>¿en que te puedo ayudar? |
| Consulta de producto genérica<br>ConsultaProductoGenerico | El cliente quiere comprar, buscar o pedir información sobre una tela, pero no dice cuál.<br>Ejemplos: "Quiero hacer una compra" · "Quiero comprar una tela" · "¿Qué tienen?" · "Me pueden ayudar a encontrar algo".<br>No activar: Ya nombró una tela concreta (tusor, gasa, polar, toalla, guardapolvo) o un uso concreto → productSearch. Stock sin intención de compra → Stock. Cuándo vuelve un producto → Reingreso. Monto mínimo de compra → FAQ. | 1. notify — Repregunta para poder buscar: tipo de tela, uso que le va a dar, color o cualquier detalle. Si el cliente plantea ir a comprar al local, le aclara que no se vende en el local y le ofrece igual acompañarlo en la compra online. Es una instrucción al agente, no un texto literal: el bot lo reformula.<br>Nuevo sistema de snappy- reglas de negocio ()<br>Leer la variante (ejemplo: descuento por metro) Si pide 10 es distinto a que pida 15 porque hay descuento por cantidad<br>Listado de los productos en el carrito |
| Búsqueda de tela<br>productSearch<br>flujo nativo | Comportamiento por defecto cuando el cliente nombra una tela específica o describe un uso concreto.<br>Ejemplos: "¿Tenés gasa?" · "Busco tela para cortinas" · "Tusor negro, ¿tienen?". | Consulta el catálogo y devuelve un carousel de resultados. Si no hay resultados o la consulta es ambigua, repregunta o deriva al flujo específico: reingreso, no lo encuentra en la web, o más metros de lo publicado. |
| Más metros de lo publicado<br>MasDeLoPublicado | El cliente quiere comprar más metros de los que figuran disponibles en la publicación.<br>Ejemplos: "Quiero comprar más metros del producto" · "El stock publicado no me alcanza" · "Necesito una partida entera".<br>No activar: Consulta de stock normal → Stock. Quiere hablar con un vendedor mayorista → compraMayorista. | 1. sector — Asigna la conversación al sector sales.<br>2. ticket — Abre un caso con motivo sales, pidiendo:<br>• Metros — ¿Cuántos metros necesitás? |
| Compra mayorista o hablar con un vendedor<br>compraMayorista | El cliente pide hablar directamente con soporte o con ventas mayoristas.<br>Ejemplos: "Quiero hablar con soporte mayorista" · "Quiero hablar con un vendedor mayorista" · "Quiero hablar con un vendedor".<br>No activar: Quiere comprar por mayor o pregunta precios mayoristas sin pedir un vendedor → ConsultaProductoGenerico. Partidas enteras o cantidades grandes → MasDeLoPublicado. | 1. sector — Asigna la conversación al sector mayorista.<br>2. notify — Avisa que lo van a contactar, textual: "Gracias por tu consulta. En un momento un asesor mayorista se va a comunicar con vos para darte más información."<br>3. assistance — Dispara asistencia humana con motivo sales: escala en vivo si está en horario, ticket si no.<br>¿Compra por mayor o es un mayorista? ¿cuantos metros quiere? compra recurrente Compra por mayor: resuelve el bot<br>Es mayorista (vip): aclarar que tienen descuentos especiales y hay que pasarle a un asesor (cambio de lenguaje)<br>Preguntar derivar con un humano cuando no se llego a cerrar la venta |
| Pago rechazado<br>PagoRechazado | El pago fue rechazado al intentar comprar.<br>Ejemplos: "Mi pago fue rechazado" · "No me tomó la tarjeta" · "No pude finalizar la compra por un problema de pago".<br>No activar: Ya compró y el pago no se acreditó → ComprobanteDePago. Sigue con problemas tras el primer rechazo → PagaRechazadoTarjeta. | 1. sector — Asigna la conversación al sector cobranzas.<br>2. notify — Da las recomendaciones de reintento, textual: "En este caso te recomendamos intentar con otra tarjeta o medio de pago, probar desde otro dispositivo o navegador". |
| Sigue con problemas de pago con tarjeta<br>PagaRechazadoTarjeta | El cliente sigue teniendo problemas con el pago después de haberlo intentado.<br>Ejemplos: "Sigo teniendo problemas con el pago" · "Probé con otra tarjeta y tampoco" · "Nada me funciona para pagar".<br>No activar: Primer rechazo → PagoRechazado. | 1. assistance — Dispara asistencia humana con motivo errordepago: escala en vivo si está en horario, ticket si no. |

# Flujos de Post-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Estado del pedido<br>orderTracking<br>flujo nativo | El cliente pregunta por el estado de su pedido o envío, con o sin número de orden. Es el comportamiento por defecto para consultas de seguimiento.<br>Ejemplos: "¿Dónde está mi pedido?" · "Quiero saber el estado de mi compra". | 1. orderStatus — Pide el número de pedido si no lo tiene, explicando que llega por mail con la confirmación y que son 6 dígitos que suelen empezar por 3. Devuelve estado, fecha estimada y tracking según la integración.<br>Configurar todos los posibles estados de pedido |
| Consulta por retiro en showroom<br>ConsultaPorRetiro | El cliente quiere consultar el estado de su pedido para poder retirarlo en el showroom.<br>Ejemplos: "Quiero retirar mi pedido" · "¿Mi pedido está listo para retirar?" · "¿Puedo pasar a retirar?".<br>No activar: Solo pregunta dónde está el showroom o los horarios → InfoShowroom. Quiere ver telas en persona sin mencionar retiro → ConsultaProductoGenerico. Pedido demorado → pedidoRetrasado. Quiere cancelar → CancelacionDePedido. Estado en general → orderTracking. | 1. orderStatus — Consulta el estado de la orden para confirmar si está lista para retirar. |
| Pedido retrasado<br>pedidoRetrasado | El envío está demorado o el cliente reporta que ya pasó la fecha de entrega prometida.<br>Ejemplos: "Ya pasó la fecha estimada de entrega" · "Mi pedido no llegó" · "El envío tardó más de lo esperado".<br>No activar: Visita fallida → visitaFallida. Solo quiere saber el estado → orderTracking. Quiere hacer un pedido nuevo o pedir presupuesto → flujo de venta. | 1. sector — Asigna la conversación al sector envios.<br>2. ticket — Abre un caso orderStatus / pedidoAtrasado pidiendo:<br>• Orden — ¿Me decís tu número de orden? Si ya lo mencionó antes, lo guarda y saltea la pregunta.<br>• Detalle — ¿Podés explicarme un poco más para que el agente tenga más contexto?<br>Solicitar el numero de orden<br>Explicar que le hable a shipnow (faq)<br>El link que se genera de shipnow investigar para brindar ese link |
| Cambio de dirección de envío<br>CambioDeDireccion | El cliente quiere cambiar la dirección de entrega de su pedido.<br>Ejemplos: "Quiero cambiar la dirección de entrega" · "Me equivoqué en la dirección".<br>No activar: Cambio de producto → cambios. Cancelación → CancelacionDePedido. | 1. faq — Informa la política de cambio de dirección.<br>2. sector — Asigna la conversación al sector envios.<br>3. ticket — Abre un caso modifyOrder / cambiarDireccionDeEnvio pidiendo:<br>• Número de pedido — Si ya lo mencionó antes, lo guarda y saltea la pregunta.<br>• Dirección nueva — Calle, número, CP, piso, depto y lote si es barrio cerrado. Aclara que se va a evaluar el caso antes de confirmar el cambio. |
| Cancelación de pedido<br>CancelacionDePedido | El cliente quiere cancelar su pedido.<br>Ejemplos: "Quiero cancelar mi compra" · "Cancelá mi pedido" · "No quiero recibir el pedido".<br>No activar: Cambio de producto → cambios. Devolución después de recibirlo → Devoluciones. | 1. sector — Asigna la conversación al sector Cancelacion.<br>2. notify — Anticipa que la cancelación se evalúa: "Vamos a proceder a analizar el estado del pedido para evaluar la posibilidad de avanzar con la cancelación."<br>3. ticket — Abre un caso cancelOrder pidiendo:<br>• Número de orden — Si ya lo mencionó antes, lo guarda y saltea la pregunta. |
| Estado de la devolución<br>EstadoDevolucion | El cliente quiere saber en qué está su devolución.<br>Ejemplos: "¿Qué pasó con mi devolución?" · "Quiero saber el estado de mi devolución" · "¿Cuándo me devuelven el dinero?".<br>No activar: Ya avisó y tiene un ticket abierto → ConsultaEstadoDeDevolucion. | 1. menu — Pregunta en qué etapa está la devolución y ramifica hacia el aviso de devolución realizada o la consulta con caso ya abierto.<br>Diagrama 2<br>BRINDA EL ID DEL CASO Y EL BOT DEVUELVE EL ESTADO DE CASO<br>DEFINIR LA ORGANIZACION DE LOS TICKETS |
| Aviso de devolución realizada<br>ComprobanteDevolucion | El cliente indica que ya hizo la devolución física, es decir que dejó el paquete en el correo.<br>Ejemplos: "Ya devolví el producto" · "Ya dejé la tela en el correo" · "No, todavía no avisé".<br>No activar: Quiere iniciar una devolución → Devoluciones. Quiere saber el estado → EstadoDevolucion. | 1. sector — Asigna la conversación al sector devolucion.<br>2. assistance — Dispara asistencia humana con motivo devolutions.<br>Diagrama 2 |
| Consulta de estado con caso ya abierto<br>ConsultaEstadoDeDevolucion | El cliente ya hizo la devolución, ya tiene un ticket abierto y quiere saber cómo viene.<br>Ejemplos: "Sí, ya me abrieron un ticket" · "Ya tengo un caso abierto" · "Ya avisé al equipo".<br>No activar: No tiene ticket abierto → EstadoDevolucion. | 1. notify — Reconoce el caso existente, textual: "Ya recibimos tu ticket de devolucion, de todas formas vamos a abrir otro para que te quedes tranquilo".<br>2. sector — Asigna la conversación al sector devolucion.<br>3. ticket — Abre un caso devolutions / other con descripción "Ya abrio un caso, quiero info", pidiendo:<br>• Número de pedido — Si no lo tiene, saltea la pregunta.<br>Diagrama 2<br>⚠ Abre un caso nuevo aunque ya exista uno por la misma devolución. A confirmar. |
| Reclamo de producto<br>ReclamoDeProducto | El cliente reporta un problema con un producto recibido: faltante, dañado, manchado o incorrecto.<br>Ejemplos: "Me llegó la tela con manchas" · "Me mandaron la tela equivocada" · "Me falta parte del pedido".<br>No activar: Ya tiene un ticket abierto por el mismo pedido. Devolución sin defecto → Devoluciones. Reclamo con lenguaje agresivo → ReclamosFuertes. | 1. sector — Asigna la conversación al sector reclamos.<br>2. assistance — Dispara asistencia humana con motivo other, pidiendo antes:<br>• Número de pedido — ¿Me podés pasar el número de pedido? |
| Reclamos fuertes<br>ReclamosFuertes | El cliente usa lenguaje agresivo o amenazante, o menciona acciones legales.<br>Ejemplos: "Qué vergüenza esta empresa" · "Son unos estafadores" · "Voy a hacer la denuncia en Defensa del Consumidor" · "Los voy a escrachar en redes".<br>No activar: Reclamo de producto sin lenguaje agresivo → ReclamoDeProducto. Cliente disconforme pero sin amenazas ni insultos → flujo correspondiente al reclamo. | 1. assistance — Deriva de inmediato a una persona con motivo other, sin pedir datos del pedido ni intentar resolver por bot. |
| Comprobante de pago o pago no acreditado<br>ComprobanteDePago | El cliente indica que su pago no fue acreditado o quiere enviar el comprobante.<br>Ejemplos: "No se acreditó el pago de mi compra" · "Quiero mandar el comprobante de pago" · "Pagué pero no me confirmaron la compra".<br>No activar: Pago rechazado por la tarjeta → PagoRechazado. Sigue con problemas de tarjeta → PagaRechazadoTarjeta. | 1. sector — Asigna la conversación al sector cobranzas.<br>2. assistance — Dispara asistencia humana con motivo errordepago.<br>Solicitar el numero de orden- confirmar si esta acreditado y si no esta confirmado: Puede ser porque no coincide los datos de la tarjeta los costos son distintos asesor |
| Facturas<br>Facturas | El cliente solicita la factura de una compra o avisa que no le llegó.<br>Ejemplos: "Quiero la factura de mi compra" · "No me llegó la factura" · "No encuentro mi factura".<br>No activar: Factura con datos erróneos → DatosErroneosFactura. | 1. sector — Asigna la conversación al sector cobranzas.<br>2. assistance — Dispara asistencia humana con motivo billing.<br>Avisarle al cliente que le llega por email, preguntarle si lo vio sino asignarle a un agente |
| Pedido con pago pendiente<br>orderStatus:awaiting-payment | Sin activador por mensaje del usuario. Se dispara cuando la consulta de estado devuelve una orden en awaiting-payment: generada pero sin pago acreditado. | 1. sector — Asigna la conversación al sector cobranzas.<br>2. assistance — Dispara asistencia humana con motivo buyingIssues. |

## Flujos de Sistema (Automatizados)

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Mensaje de Bienvenida | Sin activador por mensaje del usuario — se dispara desde automatizaciones al iniciar el primer contacto. | Envía un mensaje fijo (y estacional, si aplica) definido por el cliente. [Contenido, tono y condición de activación a configurar por cliente]. |
| Seguimiento Proactivo | Sin activador por mensaje del usuario — automático tras un tiempo configurable de inactividad. | Envía un mensaje de seguimiento preguntando cómo le fue al usuario con la gestión. [Tiempo de inactividad y activación/desactivación a configurar por cliente]. |
| Cierre por Inactividad + CSAT | Sin activador por mensaje del usuario — automático al quedar la conversación inactiva. | Envía mensaje de cierre y dispara la Encuesta de Satisfacción (CSAT). [Tiempo de inactividad y activación/desactivación a configurar por cliente]. |
| Notificaciones / Broadcast | Se dispara desde un envío de notificación/broadcast programado en Labs. | Envía el contenido definido en la notificación y puede setear custom contexts o asignar la conversación a un sector/usuario. |

# Requerimientos No Funcionales

| Categoría | Requerimiento |
| :-: | :-: |
| Lenguaje | Rol mixto (atención al cliente y ventas), uso de emojis medio, español rioplatense. |
| Testing | El código/JSON debe pasar el test suite estándar de Snappy (umbral 85%+) antes de cualquier subida a producción. |
| Conversaciones | El agente debe contar con contexto de las conversaciones previas de cada contacto y poder responder usando esa información (ej.: reconocer a qué ticket se refiere el usuario en una conversación nueva). |
| Comportamiento — Fallback | Cuando no cuente con información suficiente en Conocimientos, debe ofrecer solicitar asistencia de un operador humano. |
| Comportamiento — Proactividad y reencauce | Ante silencio de 5 minutos en un flujo de venta en curso, reactiva la conversación con un mensaje de seguimiento. Ante desvío de tema, responde la consulta puntual y luego reencauza hacia la venta en curso, sin abandonarla. |

# Flujos con Lógica Condicional (flujos personalizados)

Los siguientes flujos tienen ramificaciones internas y requieren un diagrama de decisión propio. Los demás flujos son lineales y quedan descriptos por completo en las tablas anteriores.

# Diagramas de Flujo

Representación visual de la lógica de decisión de los flujos condicionales listados arriba.

Versión editable de estos diagramas (formas nativas, sin necesidad de recrearlos): Diagramas_Flujo_Telavendo.pptx, adjunto junto con este documento.

# Checklist de Configuración por Cliente

Antes de dar por cerrado el alcance funcional, completar y validar:

- Cierre de la venta: con carrito y recomendación desactivados, definir si el bot solo muestra producto o si se habilita el checkout asistido con los datos ya configurados.
- Identificadores de sectores: validar que sales, envios, mayorista, reclamos, devolucion, Cancelacion, CyDdentrodeplazo y CyDfueradeplazo existan con ese nombre exacto en el HelpDesk.
- Horario de atención: confirmar la franja que separa escalado en vivo de generación de ticket, incluidos fines de semana y feriados.
- Duplicación de casos de devolución: decidir si ConsultaEstadoDeDevolucion sigue abriendo un ticket nuevo cuando ya existe uno.
- Factura automática: verificar si la integración puede devolver el comprobante para no derivar toda consulta de factura a Cobranzas.
- Automatizaciones: definir mensaje de bienvenida, seguimiento proactivo, cierre por inactividad y encuesta de satisfacción (CSAT): contenido, tiempos y activación.
- Contenido de la Base de Conocimientos: validar los artículos de stock, reingreso, búsqueda en la web, showroom, sucursales minorista y mayorista, costo de envío, envíos gratis, puntos de pago, visita fallida, cambio de dirección y datos de facturación.
- Menús: validar las opciones del menú principal, de cambios, de devoluciones, de estado de devolución, de pago rechazado y de búsqueda de tela.
- Política de 30 días: confirmar excepciones por categoría o promoción y desde qué fecha se cuenta el plazo.
- Nombre y datos de contacto del referente de Telavendo para resolver los puntos pendientes durante la implementación.
