> Fuente: Alcance_Funcional_Weis_Bot_IA_v2.docx (https://drive.google.com/file/d/1NNVv2jcxnZ8GHzvQnGb5r7J5tRzQoaur/view?usp=drivesdk)

Snappy Commerce

# Alcance Funcional Weis — Agente IA de Atención al Cliente y Ventas

Versión 2 · 17/11/2026 · Ajustada con lo acordado en la reunión de relevamiento con Javier Devoto (Weis) y con el análisis del bot actual de Weis Argentina.

## Cómo usar este documento

Este documento describe la estructura estándar de flujos de un Agente IA de atención al cliente y ventas de Snappy Commerce. Sirve como punto de partida para el relevamiento (discovery) de un cliente nuevo: cada fila define el flujo, sus activadores típicos y el comportamiento esperado en términos generales.

Los textos entre corchetes y las notas marcadas con ⚠ señalan puntos que dependen de la política, integración o contenido propio de cada cliente y deben completarse durante el discovery. Los nombres de los flujos pueden mantenerse o adaptarse a la nomenclatura interna del cliente.

Criterio de esta versión: ante diferencias, prevalece lo acordado en la reunión de relevamiento del 17/11/2026; el comportamiento del bot actual se usa para completar lo que no se definió. Los puntos marcados con ⚠ siguen pendientes de confirmación con el cliente.

# Requerimientos Funcionales

## Configuración Global

| Flujo nativo | Qué hace el flujo | Configuración por default |
| :-: | :-: | :-: |
| Crear contacto | Solicita los datos de contacto antes de transferir con un asesor o crear un ticket. Si el canal ya aporta un dato (teléfono en WhatsApp, ID de Instagram/Facebook), no lo vuelve a pedir. | Nombre / Email / Teléfono: obligatorios antes de derivar o abrir un ticket.<br>El email es indispensable para unificar contactos duplicados (acordado en el relevamiento del 17/11).<br>⚠ Definir qué hace el bot si el cliente se niega a dar el email. |
| Asistencia | Cuando el usuario pide hablar con una persona, el bot primero pide el motivo y responde si puede (ver Derivación a Vendedor / Asistencia comercial). Si corresponde derivar, en horario de atención transfiere con un asesor y fuera de horario genera un ticket. En ambos casos solicita antes los datos de contacto. | Escala: en horario<br>Ticket: fuera de horario<br>Horario de atención: lunes a viernes hábiles de 9 a 17 hs (excepto feriados).<br>Mensaje fuera de horario:<br>En este momento no contamos con un asistente disponible. Nuestro horario de atención es de lunes a viernes, de 9 a 17 hs. Tomamos nota de tu consulta y nos comunicaremos con vos a la brevedad. |
| Ticket | Cuando se abre un ticket, el bot pide el motivo de la consulta antes de generarlo (si no lo tiene) | Motivo obligatorio<br>Mensaje de confirmación del ticket:<br>He creado un ticket para tu consulta. Un asesor se pondrá en contacto contigo para darte una respuesta. Recordá que nuestro horario de atención es de lunes a viernes (hábiles) de 9 a 17 hs. |
| Encuesta de satisfacción | Al cerrar una conversación, se envía automáticamente una encuesta preguntando cómo fue la atención. También se incluye en el mensaje de seguimiento por inactividad. | true (decisión delegada en Snappy durante el relevamiento) |
| Falta de información | Si el bot no tiene información para responder una consulta, lo informa y ofrece transferir con un asesor. | Ofrecer derivación (acordado en el relevamiento del 17/11). |
| Canales | Canales donde responde el bot. | Con bot: WhatsApp, Instagram y Facebook.<br>Sin bot: Mercado Libre.<br>La atención es solo por mensajes (no hay atención telefónica). |

## Flujos de Pre-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Búsqueda y Recomendación de Producto | Usuario pregunta por un producto, categoría, talle, color o atributo ("tenés zapatillas rojas talle 40?"). | Ejecuta productSearch contra el catálogo de Tiendanube y devuelve un carousel de resultados. Si no hay resultados o hay demasiada ambigüedad, repregunta o deriva a asistencia.<br>Reglas de marca:<br>- Los bastones son de trail running. Si preguntan si sirven para trekking: sí, también se pueden usar.<br>- Weis no vende productos COROS ni indumentaria NNormal (ver Flujos personalizados).<br>- Nunca nombrar fábricas ni proveedores; como mucho el país de origen, si figura en la publicación. |
| Consulta de Precio / Stock | Usuario pregunta precio, disponibilidad o stock de un producto puntual. | Precio: responde con la información de la integración de Tiendanube.<br>Stock: el bot no afirma ni niega disponibilidad. Muestra el producto o el link del buscador (https://www.weis.com.ar/search/?q=PALABRA) e indica que, si el talle aparece para seleccionar en la publicación, hay stock.<br>Si el cliente ya sabe que está agotado: informa la opción "Avisame cuando haya stock" de la web; si no aparece, el producto no vuelve a ingresar.<br>⚠ Confirmar si se mantiene este criterio o el bot informa el stock desde la integración. |
| Promociones | Usuario pregunta por descuentos, promos, cupones o "qué ofertas tienen". | Envía el mensaje nativo de Promociones con las promociones activas configuradas en Labs.<br>Cuando el cliente muestra interés en comprar, envía una sola vez por conversación el mensaje "Beneficios para tu compra" (cuotas y envío).<br>Descuentos por cantidad para grupos/teams y uso de cupones (incluido GOcuotas, cupón REGALOGO): Base de Conocimientos.<br>⚠ Hoy no hay promociones vigentes en Labs (las 4 cargadas vencieron; la última el 12/11). Cargar las vigentes antes de salir a producción. |
| Puntos de Venta y Retiro | Usuario pregunta dónde comprar, si hay local o sucursal, dónde ver o probar productos, la dirección del depósito o si puede retirar en Esker. | Weis no tiene locales ni sucursales propias: no usa storeSearch (no hay sucursales cargadas en Labs).<br>Responde con el contenido de Conocimientos "¿Dónde puedo comprar?" (mapa de comercios y Esker).<br>El depósito de Lomas de Zamora es solo punto de retiro, sin atención al público. Esker no es punto de retiro.<br>Nunca referirse a un comercio como "nuestra tienda" o "nuestro local". |
| FAQ Comercial (talles, envíos, medios de pago) | Usuario consulta guía de talles, costos/tiempos de envío, medios de pago o cuotas. | Responde con el contenido de la Base de Conocimientos (FAQ), migrado desde las FAQs del bot actual:<br>- Talles: chalecos e indumentaria, bastones según altura, zapatillas NNormal.<br>- Envíos: costo y envío gratis, plazos de entrega, envíos internacionales, intentos de entrega por transportista.<br>- Medios de pago y cuotas (incluye GOcuotas) y cómo comprar. |
| Checkout Asistido | Usuario quiere comprar / pide el link de compra ("quiero comprar", "pasame el link"). | Solicita los datos de contacto configurados (firstName, email) y genera el link de checkout de Tiendanube.<br>Si ya tiene productos en el carrito, le pasa el link para finalizar la compra; si no, explica cómo comprar y ofrece armar el carrito. |
| Modificar pedido | Usuario quiere modificar un pedido que todavía no recibió: agregar o quitar productos, cambiar talle, color o cantidad, cambiar el punto de retiro o quién lo recibe, o cambiar el medio de pago. | ETAPA 1:<br>Responde con un mensaje fijo (textos a enviar por Javier Devoto).<br>ETAPA 2:<br>Valida el estado del pedido y luego responde:<br>1- Si está en preparación (pago, preparando, procesando): pregunta qué quiere cambiar, sin pedir otros datos del pedido, y transfiere con un asesor, que confirma por el mismo chat.<br>2- Si ya fue despachado (enviado, en camino, en sucursal, para retirar): informa que ya no se puede modificar. Si es un cambio de talle, color o producto, indica que al recibirlo puede cambiarlo por Undo, la plataforma de autogestión (30 días corridos). Deriva solo si el cliente insiste.<br>Medio de pago: nunca se puede cambiar sobre una orden. Si el pedido no fue despachado, ofrece cancelar la compra y volver a comprar; si confirma, deriva a un asesor para cancelarla.<br>⚠ Definir la respuesta para los estados entregado, cancelado y pendiente de pago. |
| Modificar dirección | Usuario quiere quiere modificar su dirección de entrega | ETAPA 1:<br>Responde con un mensaje fijo (textos a enviar por Javier Devoto).<br>ETAPA 2:<br>Valida el estado del pedido y luego responde:<br>1- Si está en preparación: solicita la dirección nueva completa (calle, número, piso y departamento, localidad y código postal) y transfiere con un asesor, que la corrige y confirma por el mismo chat.<br>2- Si ya fue despachado: informa que ya no es posible cambiar la dirección de entrega. Deriva solo si el cliente insiste.<br>⚠ Definir la respuesta para los estados entregado, cancelado y pendiente de pago. |
| Checkout Asistido — Gestión del Carrito | Usuario quiere agregar, quitar o modificar productos en un carrito ya iniciado. | Actualiza el carrito vía integración según la acción solicitada, validando stock de la combinación elegida.<br>⚠ Confirmar soporte en la integración de Tiendanube (el bot actual no lo hace). |
| Derivación a Vendedor / Asistencia comercial | Usuario pide hablar con un asesor de ventas o la consulta excede el alcance del bot. | Objetivo: reducir al mínimo la intervención humana (acordado en el relevamiento del 17/11).<br>1- Si pide un asesor sin decir el motivo: pregunta el motivo, sin derivar ni pedir datos.<br>2- Si el motivo tiene respuesta: responde primero y pregunta "¿Con esto lo resolvés o querés que igual te contacte un asesor?".<br>3- Deriva (solicitando los datos de contacto) si: confirma que quiere un asesor, insiste por segunda vez, no hay información para responder o el caso solo lo puede resolver una persona (producto fallado, incompleto o distinto; pedido demorado sin movimiento; modificar o cancelar un pedido; problema con un pago ya hecho; garantía; caso en curso; queja formal). |

## Flujos de Post-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Estado de Pedido | Usuario pregunta por el estado de su pedido/envío, con o sin número de orden. | Envía un mensaje orderStatus con el estado, fecha estimada y tracking (si está disponible) vía la integración (Tiendanube / Andreani).<br>Busca el pedido con el número de orden (figura en el mail de confirmación). Si el cliente pasa un código de seguimiento del correo, pide el número de orden.<br>Si el pedido está en preparación, explica los plazos de entrega sin derivar.<br>Retiro en Smart Locker de PUDO o punto HOP de Andreani: Base de Conocimientos. |
| Factura del pedido | Usuario solicita la factura de una compra. | Responde con FAQ: la factura A o B se genera automáticamente según el DNI o CUIT cargado en la compra. Si necesita factura A y no cargó los datos correctos, debe escribir a facturacion@weis.run.<br>⚠ Falta definir la respuesta para "no me llegó la factura". |
| Cambios y Devoluciones | Usuario quiere cambiar o devolver un producto ya comprado. | Responde con FAQ de Undo (plataforma de autogestión: https://rma.pow.la/clients_gestions/88). Para ingresar: número de pedido con # adelante y el mail de la compra.<br>Se envía a Undo cuando:<br>- Ya recibió el pedido y quiere cambiar talle, color o producto, o devolverlo, dentro de los 30 días corridos desde que lo recibió (sin uso, con embalaje y etiquetas).<br>- Pregunta si puede llevarlo a un local, a Esker o a Garibaldi: primero tiene que hacer la solicitud en Undo (solo puede entregarlo donde Undo le ofrezca).<br>- El pedido ya fue despachado y quiere otro talle, color o producto, o cancelarlo: al recibirlo lo gestiona por Undo.<br>- Tiene problemas para usar Undo: el bot lo guía en el paso donde se trabó, sin derivar de entrada.<br>Ofrece cambio por producto o por cupón; la devolución de dinero solo si el cliente la pide. Cada compra tiene un solo cambio sin cargo: si cambia varios productos, cargarlos en una misma solicitud.<br>No se envía a Undo (deriva a un asesor):<br>- Todavía no recibió el pedido (va a Modificar pedido o Cancelaciones).<br>- El producto llegó fallado o distinto al comprado (es un reclamo).<br>- Con el # y el mail correctos no puede ingresar a Undo.<br>- Undo le indica que el pedido ya tiene un cambio: si todavía no entregó el paquete, el asesor cancela la primera solicitud para cargar todo junto; si ya lo entregó, la segunda solicitud tiene costo de envío.<br>- Quiere cancelar una solicitud de cambio (no se puede desde Undo).<br>Incluye además: estado del cambio o la devolución, plazos del reintegro y aviso de que el cliente ya envió el paquete (agradece, sin derivar). Si el talle no le calza o no le gustó, es un cambio, no un reclamo.<br>⚠ Productos "sin cambio ni devolución": la FAQ actual dice que si hubo error de envío o falla la devolución se hace por Undo, pero otra FAQ dice que un error de envío lo resuelve un asesor. Definir cuál aplica. |
| Cancelaciones | Usuario quiere cancelar un pedido. | No hay formulario de cancelación.<br>1- Si el pedido no fue despachado: confirma que quiere cancelar y deriva a un asesor, que cancela la orden; el reintegro va al mismo medio de pago.<br>2- Si ya fue despachado: informa que no se puede cancelar y que al recibirlo puede hacer la devolución por Undo. Deriva solo si insiste.<br>3- Si ya lo recibió: es una devolución (ver Cambios y Devoluciones).<br>Undo no se usa para cancelar pedidos: solo para la devolución una vez recibido. |
| Garantía | Usuario consulta por cobertura o activación de garantía de un producto. | Informa que el período y el alcance de la garantía están en https://weis.com.ar/politica-de-cambios/. Bastón roto: Crash Replacement (Base de Conocimientos). Reclamo de garantía de un producto puntual: deriva a un asesor.<br>⚠ Completar plazo y condiciones de garantía en Conocimientos (hoy solo hay un link). |
| Reclamos / Ticket | Usuario reporta un problema: producto dañado, roto, incompleto o distinto al comprado. | Solicita los datos de contacto y genera un ticket en el sector Postventa con el ticketReason y ticketSubreason correspondientes.<br>No se envía a Undo: un error de envío o una falla lo resuelve un asesor.<br>No es reclamo: talle o gusto (va a Cambios), bastones que no traban (va a Uso de bastones), pago rechazado al comprar (va a Pago rechazado).<br>⚠ Definir motivo y submotivo (hoy el bot usa "Consultas Post Compra"). |
| Encuesta de Satisfacción (CSAT) | Se dispara automáticamente al cierre de una gestión o conversación. | Envía la encuesta de satisfacción configurada de forma global. |
| Derivación a Soporte Post-venta | La consulta post-venta excede el alcance del bot o el usuario pide hablar con un humano. | Mismo criterio que Derivación a Vendedor / Asistencia comercial: pide el motivo, resuelve si puede y deriva solo en los casos definidos, solicitando antes los datos de contacto. |

## Flujos de Sistema (Automatizados)

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Mensaje de Bienvenida | Sin activador por mensaje del usuario — se dispara desde automatizaciones al iniciar el primer contacto. | Saludo directo, sin menú de opciones y sin mencionar "asesor" (acordado en el relevamiento del 17/11).<br>Hola! [emoji] Soy el asistente virtual de Weis.<br>Podés hacerme tu consulta ahora mismo y te voy a responder. |
| Seguimiento Proactivo | Automático a los 10 minutos de inactividad. | Envía un único mensaje de seguimiento preguntando si la consulta quedó resuelta. Incluye la encuesta de satisfacción.<br>⚠ Texto del mensaje a definir. |
| Cierre por Inactividad + CSAT | Sin activador por mensaje del usuario — automático al quedar la conversación inactiva. | Si no hay respuesta al seguimiento, cierra la conversación automáticamente. Uno por conversación + incluye encuesta.<br>⚠ Tiempo hasta el cierre a definir. |
| Notificaciones / Broadcast | Se dispara desde un envío de notificación/broadcast programado en Labs. | Envía el contenido definido en la notificación y puede setear custom contexts o asignar la conversación a un sector/usuario. |

## Flujos con Lógica Condicional (flujos personalizados)

Los siguientes flujos tienen ramificaciones internas y requieren un diagrama de decisión propio, construido durante el discovery de cada cliente. Se listan aquí los puntos de ramificación genéricos que ese diagrama debe cubrir:

Primera Etapa: mensajes fijos para Modificar pedido y Modificar dirección (textos a enviar por Javier Devoto), mientras se desarrollan los flujos de la segunda etapa.

Segunda Etapa:

Modificación de pedido: el bot pide el número de pedido y valida su estado. Si está en preparación, pregunta qué quiere cambiar y solicita asistencia. Si ya fue despachado, indica que no es posible modificarlo; si es un cambio de talle, color o producto, informa que al recibirlo puede hacerlo por Undo (plataforma de autogestión). Deriva solo si el cliente insiste.

Modificación de dirección: el bot pide el número de pedido y valida su estado. Si está en preparación, solicita la nueva dirección y solicita asistencia. Si ya fue despachado, indica que no es posible cambiar la dirección. Deriva solo si el cliente insiste.

⚠ Tomas Sayar: validar con desarrollo la factibilidad de validar el estado del pedido antes de derivar.

## Flujos personalizados heredados del bot actual

Flujos del bot actual que cambian el comportamiento (derivar, no derivar o redirigir) y deben mantenerse en Gen 6:

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Consultas COROS | Menciona COROS o pregunta por relojes GPS, smartwatch, pulsómetros, ciclocomputadores o productos COROS. | Weis no vende COROS: responde con el WhatsApp de COROS, sin pedir datos ni derivar. Tiene prioridad sobre la búsqueda de productos. |
| Sponsoreo / Atletas | Propone colaboración, canje, sponsoreo, ser embajador, contenido, evento o carrera. | Responde con el mail sponsoreo@weis.run, sin pedir datos ni derivar. Tiene prioridad sobre la asistencia y la creación de contacto. (Búsquedas laborales: rrhh@weis.run.) |
| Uso de bastones | El cliente no logra armar, plegar o trabar los bastones, o pregunta si están fallados. | Explica cómo armarlos y plegarlos (imágenes paso a paso en la publicación de la web). Si ya recibió los pasos y sigue sin poder, o manda fotos o video, deriva a un asesor. Solo es reclamo si el bastón llegó roto. |
| Pago rechazado | Mientras compra: tarjeta rechazada, "posible fraude", no le deja pagar o el pago da error. | Responde con los pasos de la Base de Conocimientos, sin pedir datos ni derivar. Deriva solo si ya probó los pasos y sigue sin poder pagar. |
| Pago pendiente | Ya compró y no pagó o no se acreditó: hasta cuándo se reserva, si se canceló la orden. | Informa el plazo de reserva. La cancelación por falta de pago es automática: no deriva. |
| Queja sin consulta | Enojo o queja general sin una pregunta concreta (precios, atención del bot, despedida enojada). | Una sola respuesta corta y respetuosa, sin ofrecer productos, sin preguntas y sin derivar. |

El resto de los temas del bot actual (talles, cuidado y lavado, compatibilidades de carcaj y soft flask, Number Belt, ventas por mayor, proveedores, retiro en depósito y Esker, envíos internacionales, teléfono de contacto) se migran como Base de Conocimientos. No migrar: Feriados de noviembre (vencido) ni el menú principal.

⚠ Cualquier comportamiento no cubierto explícitamente en este documento debe confirmarse con el referente técnico del cliente antes de la implementación.

# Diagramas de Flujo (solo tienen que estar las que son personalizadas)

Representación visual de la lógica de decisión de los flujos condicionales listados arriba. Son diagramas base para discovery: los rótulos entre corchetes o marcados con ⚠ en las tablas anteriores deben reflejarse aquí una vez definidos con el cliente.

Versión editable de estos diagramas (formas nativas, sin necesidad de recrearlos): [Diagramas_Flujo_Bot_IA.pptx](https://docs.google.com/presentation/d/1THBgyODVB9IX3UfAmipArhx6RDR1ZC11/edit?slide=id.p1#slide=id.p1), adjunto junto con este documento.

⚠ Pendiente: diagramas de Modificar pedido y Modificar dirección (segunda etapa).

## Checklist de Configuración por Cliente

Antes de dar por cerrado el alcance funcional de un cliente nuevo, completar y validar:

- Políticas de cambios y devoluciones: Undo (https://rma.pow.la/clients_gestions/88), 30 días corridos desde la recepción, un cambio sin cargo por compra. ⚠ Definir el circuito de productos "sin cambio ni devolución" con error de envío o falla.
- Política de garantía: ⚠ pendiente (hoy solo hay un link a weis.com.ar/politica-de-cambios).
- Transportistas: Andreani (2 intentos y 7 días en sucursal), Welivery y Jipink (2 intentos); retiro en el depósito de Lomas de Zamora, Smart Locker de PUDO y puntos HOP. Zonas y plazos: migrar desde las FAQs actuales.
- Medios de pago y cuotas: en la FAQ actual (incluye GOcuotas). ⚠ Validar si siguen vigentes las promos de MODO (FAQs desactivadas).
- Plataforma: Tiendanube. ⚠ Confirmar generación del link de checkout y gestión del carrito.
- ticketReasons y sectores: sectores Postventa y Soporte. ⚠ Definir motivo y submotivo por flujo.
- Base de Conocimientos: migrar las FAQs activas del bot actual y revisar las desactivadas.
- Mensajes fijos: bienvenida y criterio de seguimiento definidos. ⚠ Faltan los textos de seguimiento, cierre y etapa 1 de modificación (Javier Devoto).
- Referente del cliente: Javier Devoto (javier.devoto@weis.run). Disponible para las pruebas hasta el miércoles 2 de diciemb...

> Nota de conversión: el contenido que devolvió el conector de Drive termina cortado en este punto (último ítem del checklist incompleto). El emoji del saludo llegó corrupto y se marcó como [emoji].
