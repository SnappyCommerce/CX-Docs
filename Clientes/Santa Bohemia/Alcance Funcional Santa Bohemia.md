> Fuente: Alcance_Funcional_SantaBohemia (https://docs.google.com/document/d/11ta1NSe6ZgFnpf7O-57-h4jlXBsQlukKwl4VUdWHVZw/edit?usp=drivesdk&ouid=113568244751416058438)

# Pestaña 1

**Snappy Commerce**

**Alcance Funcional Santa Bohemia**

**Cómo usar este documento**

Este documento describe la estructura estándar de flujos de un Agente IA de atención al cliente y ventas de Snappy Commerce. Sirve como punto de partida para el relevamiento (discovery) de un cliente nuevo: cada fila define el flujo, sus activadores típicos y el comportamiento esperado en términos generales.

Los textos entre corchetes y las notas marcadas con ⚠ señalan puntos que dependen de la política, integración o contenido propio de cada cliente y deben completarse durante el discovery. Los nombres de los flujos pueden mantenerse o adaptarse a la nomenclatura interna del cliente.

# Requerimientos Funcionales

## Configuración Global

| Flujo nativo | Qué hace el flujo | Valor configurado |
| :-: | :-: | :-: |
| Crear contacto - Campos a solicitar al usuario | Para crear un contacto nuevo, el bot pide teléfono y email. | nombre, teléfono (optativos) |
| Comenzar checkout - Requisitos de información de contacto | Para generar un checkout, el default pide nombre, apellido y email. | firstName, lastName, email |
| Asistencia | Siempre que se dispara asistencia, el bot genera un ticket de soporte primero (pidiendo los datos que falten). Si además es horario de atención, después de crear el ticket la conversación se escala en vivo a un asesor. Fuera de horario, queda solo con el ticket creado, a la espera de gestión. | Contacto: Siempre<br>Escalar: Nunca<br>Ticket: Nunca |
| Promociones | El flujo nativo de Promociones está siempre activado. En runtime, el bot puede chequear si hay promociones activas cargadas (hasPromotions) antes de disparar el mensaje — si no hay datos cargados en Labs, el flow debe contemplar una respuesta alternativa en vez de un mensaje vacío. | true (activado)<br>Usan las promociones por URL: <https://santabohemia.com/promociones/> |
| Encuesta de satisfacción | La Encuesta de Satisfacción automática global está activado | true |
| Estado de Pedido | El flujo default de consulta de estado de pedido es el mismo para todos los clientes. Los datos disponibles (tracking, fecha estimada, etc.) dependen de lo que devuelva la integración del cliente. | true / datos según integración (orderStatus providers)<br>Busqueda por numero de pedido<br>Shipnow vinculado |
| Búsqueda de Producto (productSearch) | El flujo default de búsqueda y recomendación de producto es el mismo para todos los clientes. Los resultados, facetas y filtros disponibles dependen del catálogo/integración del cliente. | true / resultados según integración y catálogo del cliente |

## Flujos de Pre-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Búsqueda y Recomendación de Producto | Usuario pregunta por un producto, categoría, talle, color o atributo ("tenés zapatillas rojas talle 40?"). | Ejecuta productSearch contra el catálogo del cliente y devuelve un carousel de resultados. Si no hay resultados o hay demasiada ambigüedad, repregunta o deriva a asistencia. |
| Consulta de Precio / Stock | Usuario pregunta precio, disponibilidad o stock de un producto puntual. | Responde con la información obtenida vía integración (Tienda Nube). |
| Información de Sucursales / Branches | Usuario pregunta dirección, horarios o sucursal más cercana. | Responde con la información cargada en conocimientos (branches) de sucursales (mapa nativo) |
| Checkout Asistido | Usuario quiere comprar / pide el link de compra ("quiero finalizar la compra", "pasame el link"). | Solicita los datos de contacto configurados (firstName, email y lastName) y genera el link de checkout de Tienda Nube. |
| Checkout Asistido — Gestión del Carrito | Usuario quiere agregar, quitar o modificar productos en un carrito ya iniciado. | Actualiza el carrito vía integración según la acción solicitada, validando stock de la combinación elegida. |
| Asistencia | Usuario indica que necesita asistencia o hablar con un asesor | Conocimiento => "Para hablar con un asesor podes hacerlo por estos canales: Mail hola@santabohemia.com o al whatsapp (11) 5933-8248" |

## Flujos de Post-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Estado de Pedido | Usuario pregunta por el estado de su pedido/envío | Envía un mensaje orderStatus con el estado, fecha estimada y tracking (si está disponible) vía la integración de Shipnow |
| Asistencia | Usuario indica que necesita asistencia o hablar con un asesor | Conocimiento => "Para hablar con un asesor podes hacerlo por estos canales: Mail hola@santabohemia.com o al whatsapp (11) 5933-8248" |

## Flujos de Sistema (Automatizados)

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Seguimiento Proactivo | Sin activador por mensaje del usuario — automático tras un tiempo configurable de inactividad. | Desactivado para este cliente. |
| Cierre por Inactividad + CSAT | Sin activador por mensaje del usuario — automático al quedar la conversación inactiva. | Desactivado para este cliente. |
| Notificaciones / Broadcast | Se dispara desde un envío de notificación/broadcast programado en Labs. | El cliente no usa envios de plantillas |

## Flujos con Lógica Condicional

Los siguientes flujos tienen ramificaciones internas y requieren un diagrama de decisión propio, construido durante el discovery de cada cliente. Se listan aquí los puntos de ramificación genéricos que ese diagrama debe cubrir:

- **Sin flujos custom**

*⚠ Cualquier comportamiento no cubierto explícitamente en este documento debe confirmarse con el referente técnico del cliente antes de la implementación.*

## Checklist de Configuración por Cliente

Antes de dar por cerrado el alcance funcional de un cliente nuevo, completar y validar:

- Políticas de cambios y devoluciones: se solicita número de pedido, teléfono, prenda a cambiar y motivo, y se deriva a ticket.
- Operadores logísticos/ métodos de envío, zonas de cobertura y tiempos estimados.
- Plataforma de e-commerce: Tiendanube.
- ticketReasons y sectores de derivación configurados en el HelpDesk / CRM del cliente.
- Contenido de la Base de Conocimientos: FAQs de talles/atributos, envíos, promociones y otras consultas frecuentes.
- Mensajes fijos (bienvenida, cierre, seguimiento proactivo) y condición de activación de cada uno.
- Nombre y datos de contacto del referente del cliente para resolver puntos pendientes durante la implementación.

## Criterios de Aceptación

- Cada comportamiento migrado a Gen 6 reproduce el comportamiento de Gen 5 documentado en este alcance, con los hallazgos ya corregidos.
- Santa Bot pasa el test suite estándar de Snappy con un umbral de 85%+.
- Cuando el usuario indique que quiere hablar con una persona o el bot no tenga información suficiente en sus conocimientos para responder una consulta, debe indicarle al usuario que si necesita ayuda de un asesor puede ponerse en contacto al mail hola@santabohemia.com o al whatsapp (11) 5933-8248

## Errores actuales.

- Si el usuario insiste, el bot solicita datos de contacto e intenta transferir con un operador. En estos casos el bot debe aferrarse a la información cargada en conocimientos ya que esta tienda no tiene atención en vivo.

# Test

**TEST**

| Flujo | Usuario: | Bot: |
| :-: | :-: | :-: |
| Promociones | ¿Qué promos tienen? | Te muestro nuestras promociones vigentes el día de hoy.<br>[Carrusel de promociones] |
| Cambios | Necesito hacer un cambio | Podés realizar cambios dentro de los 30 días corridos desde que recibiste tu pedido, presentando número de orden, ticket o factura.<br>Para realizar el cambio, las prendas deben estar sin uso, en perfectas condiciones y con sus etiquetas originales.<br>Podés gestionarlo de dos formas:<br>Cambio en sucursal:<br>- Podés acercarte a cualquiera de nuestras sucursales (excepto Dino Mall, Córdoba).<br>- Importante: Los cambios de las líneas Kids y Feria se gestionan únicamente en Beláustegui 2770, CABA.<br>Cambio online:<br>Ingresá al [Portal de Cambios](https://santabohemiacambios.pow.la/clients_gestions/735) con tu número de pedido (incluyendo el #) y tu email de compra. |
| Devoluciones | ¿Cómo puedo hacer para gestionar una devolución? | Gestionar una devolución es muy sencillo. Solo tenés que enviarnos un mail a hola@santabohemia.com o escribirnos al (11) 5933-8248, que son nuestros únicos canales con atención humana donde podemos realizar esta gestión.<br>Cuando nos contactes, te pedimos que uses el asunto o mensaje "Devolución + tu número de pedido". Ahí mismo contanos qué producto querés devolver y por qué; cuantos más detalles y fotos nos puedas mandar, ¡mucho mejor!<br>Una vez que nos escribas, te vamos a responder con todas las instrucciones y la documentación necesaria para que nos envíes el paquete de forma fácil y rápida.<br>Por último, recordá que para poder avanzar con la devolución, la prenda tiene que estar tal cual la recibiste: en perfecto estado, sin uso y con sus etiquetas puestas para poder ser recomercializada. |
| Reclamos | Me llego una remera manchada | ¡Lamentamos mucho lo sucedido con tu pedido! Queremos darte la mejor solución lo antes posible, por lo que te ofrecemos tres alternativas: podés hacer el cambio de la prenda totalmente sin cargo (nos hacemos cargo del 100% de los costos de retiro y del nuevo envío), quedarte con el producto y recibir un descuento para tu próxima compra, o realizar la devolución de la prenda y te reintegramos el importe total abonado.<br>Por favor decime cuál de estas opciones preferís para indicarte los pasos a seguir y gestionar la solución de inmediato. |
| Cancelaciones | ¿Cómo cancelo mi compra? | Podés cancelar tu compra comunicándote con nosotros vía WhatsApp al (11) 5933-8248, escribiendo o gestionándola desde el Botón de Arrepentimiento. Recuerda tener a mano tu número de pedido. |
| Asistencia | Necesito hablar con una persona<br>Pasame con un asesor | Podés contactarte directamente con nuestro equipo de atención humana escribiéndonos por WhatsApp al (11) 5933-8248 o por e-mail a hola@santabohemia.com.<br>Nuestro tiempo de respuesta habitual es de hasta 48 horas hábiles. Para que podamos ayudarte más rápido, te sugerimos incluir tu número de pedido y todos los detalles de tu consulta al momento de escribirnos. ¿Querés que te ayude con algo más mientras tanto? |
| Gift Card | ¿Venden Giftcards? | ¡Sí, vendemos Gift Cards! Es la opción ideal para hacer un regalo especial y darle la libertad a esa persona de elegir lo que más le guste.<br>Podés usarla tanto en cualquiera de nuestras sucursales como en nuestra tienda online. Para usarla en la web, solo tenés que contactarnos por mail a hola@santabohemia.com o escribirnos por WhatsApp al (11) 5933-8248 y te ayudamos a gestionarla.<br>Tené en cuenta que es de uso único, no reintegra dinero y tiene una validez de 90 días desde la compra. Se aplica únicamente al valor de los productos (no incluye el costo de envío), se debe canjear el monto total en esa misma compra y, ¡lo mejor de todo, es que podés acumularla con las promociones y descuentos vigentes! |
