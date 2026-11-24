> Fuente: Analisis_Funcional_Wanama_Gen6_v1.docx (https://drive.google.com/file/d/19LcPAfNLEhIRI2Dqpejgr7_fuQlngvV9/view?usp=drivesdk)

Snappy Commerce

Análisis Funcional — Wanama, Agente IA

Wanama Bot · Relevamiento del comportamiento vigente en producción (Gen 5) como base para la migración a Gen 6

# Cómo usar este documento

Este documento releva el comportamiento actual del Agente IA de Wanama — flujos nativos, flujos personalizados y configuración global — siguiendo la estructura del alcance funcional estándar de Snappy Commerce. Cada fila define el flujo, sus activadores y el comportamiento esperado.

Sirve para que Producto valide el comportamiento vigente, detecte vacíos funcionales e inconsistencias de configuración, y tenga documentado cada flujo antes de que pase de la UI de Snappy (Gen 5) a código TypeScript mantenido por Producto (Gen 6). La experiencia del usuario final no cambia con la migración.

Las configuraciones inactivas se marcan con ❌ Inactivo y los puntos a confirmar con ⚠️. Los hallazgos y riesgos abiertos se listan al final del documento.

# Requerimientos Funcionales

## Configuración Global

| Flujo nativo | Qué hace el flujo | Valor configurado |
| :-: | :-: | :-: |
| Crear contacto — Campos | Solicita los datos indicados al usuario para crear un nuevo contacto. | firstName, email, phone |
| Asistencia — Horario | Cuando un flujo dispara "assistance", en horario de atención escala en vivo; fuera de horario genera un ticket. | Lunes a viernes 8 a 18 hs<br>Sábados 9 a 12 hs |
| Personalidad | Rol e idioma del agente. | role: both<br>language: argentino |
| Saludo inicial | Mensaje con el que el bot abre la conversación. | "Hola! Soy el asistente virtual de Wanama. En que te podemos ayudar?" |
| Menú principal | Muestra el menú de opciones al saludar (mainMenuGreeting). | ✅ Activo<br>Asistencia para comprar · Buscar productos · Mi Pedido · Promociones · Sucursales |
| Búsqueda de Producto (productSearch) | Permite buscar y recomendar productos del catálogo. | ✅ Activo |
| Estado de Pedido (orderStatus) | Consulta el estado de un pedido a partir de los datos del cliente. | ✅ Activo<br>Integración e-commerce: POW-Hermes |
| Sucursales (branches) | Provee información sobre sucursales físicas. | ✅ Activo<br>38 sucursales cargadas |
| Stock en locales (productBranches) | Consulta disponibilidad de stock en sucursal física. | ❌ Inactivo<br>La consulta se resuelve con el flujo StockEnSucursal (assistance). |
| Factura (orderInvoice) | Permite solicitar o consultar la factura electrónica de una compra. | ❌ Inactivo (orchestratorOverwrites)<br>Se reemplaza por el flujo personalizado Factura. |
| Promociones nativas | Informa sobre promociones y descuentos vigentes. | ✅ Activo |
| Calificación de FAQs | Permite al usuario calificar las respuestas de FAQs. | ✅ Activo |
| Encuesta al abandonar | Calificación automática del agente cuando el usuario deja la conversación. | ✅ Activo |
| Encuesta al cierre | Calificación automática del agente al cierre de la conversación. | ❌ Inactivo |
| Encuesta al transferir | Calificación automática del agente al transferir la conversación. | ❌ Inactivo |
| Canales con bot | Canales de mensajería donde responde el agente. | Web, WhatsApp, Redes sociales |
| Canal MercadoLibre | Canal de mensajería a través de MercadoLibre. | ❌ Inactivo (botless) |

## Flujos de Preventa

| Flujo | Activadores | Comportamiento esperado |
| :-: | :-: | :-: |
| StockEnSucursal | ✅ Usuario quiere SABER EL STOCK de un producto en una SUCURSAL específica.<br>"¿tienen stock de este vestido en el Solar?", "¿hay talle M de este jean en Palermo?", "quiero ir a probármelo, ¿lo tienen en el local?".<br>❌ Cambio o devolución en un local → CambiosYDevoluciones · Problema con un producto comprado → ReclamoPorProductoRecibido · Saludo sin motivo → MenuPrincipal | assistance (ticketReason: questionsBefore) |
| CuponDeBienvenida | ✅ Usuario consulta por un CUPÓN DE DESCUENTO o de bienvenida.<br>"¿tienen cupón de bienvenida?", "cupón del 10%", "¿tienen código de descuento?". | faq → "Tiene cupón de bienvenida?"<br>Responde: cupón WELCOMEWNM, 10% de descuento aplicable en el checkout. |
| WANAMAKIDS | ✅ Usuario consulta por WANAMA KIDS o por un pedido de Kids.<br>"wanama kids", "kids", "tengo un pedido de kids". | faq → "Wanama Kids"<br>Deriva a atcliente@wanamakids.com.<br>Sin sector (default). |

## Flujos de Postventa

| Flujo | Activadores | Comportamiento esperado |
| :-: | :-: | :-: |
| CambiosYDevoluciones | ✅ Usuario quiere CAMBIAR o DEVOLVER un producto que ya compró (por gusto o talle).<br>"quiero hacer un cambio", "me queda chico, ¿lo puedo cambiar?", "no me gustó, quiero devolverlo".<br>❌ Producto roto, fallado o equivocado → ReclamoPorProductoRecibido · Stock en local → StockEnSucursal · Saludo sin motivo → MenuPrincipal | sector → postsalesassistance (ticketReason: PostVenta)<br>Campos adicionales:<br>- Número de pedido (no confundir con el teléfono) |
| ReclamoPorProductoRecibido | ✅ Usuario RECLAMA porque el producto recibido está ROTO, FALLADO, MANCHADO o EQUIVOCADO (talle, color).<br>"me llegó en mal estado", "vino manchado", "me mandaron otro talle", "no es lo que pedí".<br>❌ Cambio por gusto o talle → CambiosYDevoluciones · Stock en local → StockEnSucursal · Saludo sin motivo → MenuPrincipal | sector → postsalesassistance (ticketReason: PostVenta)<br>Campos adicionales:<br>- Número de pedido (no confundir con el teléfono) |
| Factura ⚠️ (nuevo) | ✅ Usuario necesita la FACTURA de su compra o no la recibió.<br>"necesito mi factura", "no me llegó la factura", "¿cómo pido la factura de mi compra?".<br>❌ Cambio o devolución → CambiosYDevoluciones · Reclamo por producto → ReclamoPorProductoRecibido | assistance (ticketReason: billing)<br>Sin sector ni campos adicionales.<br>⚠️ Reemplaza a la FAQ de factura (eliminada). Pendiente de crear en el bot. |

## Flujos de Sistema y Derivación

| Flujo | Activadores | Comportamiento esperado |
| :-: | :-: | :-: |
| Notificaciones / Broadcast | Se dispara desde un envío de notificación/broadcast programado en Labs. | ❌ Inactivo (sin campañas) |

# Requerimientos No Funcionales

| Categoría | Requerimiento |
| :-: | :-: |
| Lenguaje | Español rioplatense, tono cercano y cordial. Voseo en todos los verbos (querés, podés, decime, tenés). Sin español neutro ni trato de usted. |
| Rol del agente | Preventa y postventa (role: both). |
| Testing | El código/JSON debe pasar el test suite estándar de Snappy (umbral 85%+) antes de cualquier subida a producción. |
| Conversaciones | El agente debe contar con contexto de las conversaciones previas de cada contacto y poder responder usando esa información. |
| Comportamiento — Fallback | Cuando no cuente con información suficiente en Conocimientos, debe ofrecer asistencia de un operador humano. |
| Horario de atención | Lunes a viernes de 8 a 18 hs y sábados de 9 a 12 hs. Fuera de horario, la asistencia genera ticket. |
| Canales | Web y WhatsApp con bot. MercadoLibre sin bot (botless). |
| Integraciones | POW-Hermes (e-commerce), Andreani, Meta, WhatsApp. |
| Multimarca | Las consultas de Wanama Kids se derivan a atcliente@wanamakids.com; el agente responde solo por Wanama Mujer. |

## Flujos con Lógica Condicional

Sin flujos con lógica condicional. Todos los flujos configurados son lineales.

# Diagramas de Flujo

Sin flujos con lógica condicional. No aplica.

# Test

| Categoría | Input | Output |
| :-: | :-: | :-: |
| Cambio | Quiero cambiar el talle | \[sector postsales\] → \[solicitar asistencia — ticketReason: PostVenta\] → \[solicitar número de pedido\] |
| Devolución | No me gustó, quiero devolverlo | \[sector postsales\] → \[solicitar asistencia — ticketReason: PostVenta\] → \[solicitar número de pedido\] |
| Reclamo | Me llegó roto | \[sector postsales\] → \[solicitar asistencia — ticketReason: PostVenta\] → \[solicitar número de pedido\] |
| Reclamo | Me mandaron otro color | \[sector postsales\] → \[solicitar asistencia — ticketReason: PostVenta\] → \[solicitar número de pedido\] |
| Stock en local | ¿Tienen talle M de este jean en el Solar? | \[solicitar asistencia — ticketReason: questionsBefore\] |
| Factura | Necesito la factura de mi compra | \[solicitar asistencia — ticketReason: billing\] |
| Cupón | ¿Tienen cupón de bienvenida? | ¡Sí! Contamos con un cupón de bienvenida: WELCOMEWNM. Se aplica en el checkout y descuenta un 10% del total de la compra. |
| Wanama Kids | Tengo un pedido de Wanama Kids | Somos Wanama Mujer. Para consultas por un pedido de Kids, comunicate a atcliente@wanamakids.com. |
| Menú | Hola | \[menú principal: Asistencia para comprar · Buscar productos · Mi Pedido · Promociones · Sucursales\] |
| Envío gratis | ¿Tienen envío gratis? | Envío gratis en compras superiores a $300.000. No aplica a compras con Oh! GiftCard. |
| Envíos | ¿Hacen envíos al exterior? | Por el momento no realizamos envíos al exterior. |
| Envíos | ¿Tienen envío en el día? | Contamos con envíos Same Day en CABA y GBA comprando antes de las 11:30 hs (solo con tarjeta de crédito, débito o dinero en cuenta). |
| Envíos | ¿Cuánto tarda Andreani a domicilio? | Entre 3 y 10 días hábiles según la región. Entregas de lunes a viernes de 8 a 20 hs y sábados de 8 a 12 hs. |
| Pagos | ¿Qué medios de pago aceptan? | Tarjeta de crédito y débito, efectivo (Pago Fácil, Rapipago, Bapropagos), transferencia o depósito, saldo de MercadoPago, Oh! GiftCard y Go Cuotas. |
| Promociones | ¿Tienen cuotas sin interés? | \[promociones nativas vigentes\] — ej.: ICBC 3 y 6 cuotas sin interés con Visa y Mastercard. |
| Confirmación | ¿Cuándo se confirma mi compra? | Tarjeta de crédito: mail dentro de las 24 hs. Cupón de pago: al acreditarse (hasta 72 hs). Transferencia: enviar el comprobante a transferencias@wanama.com. |
| Seguimiento | ¿Cómo hago el seguimiento de mi pedido? | Te enviamos por mail el número de seguimiento dentro de las 72 hs de confirmada la compra. Revisá también la casilla de spam. |
| Política de cambio | ¿Cuántos días tengo para cambiar online? | Tenés 30 días corridos desde que recibiste el pedido. El primer cambio es gratuito; desde el segundo, el envío lo abona el cliente. |
| Agregar producto | ¿Puedo agregar un producto a una orden ya confirmada? | No es posible modificar o agregar productos a pedidos ya realizados. Te sugerimos hacer una nueva compra. |
| Gift card | ¿Cómo uso mi Oh! GiftCard en la web? | En el checkout elegí la opción OH GIFT CARD y completá el número y el código de seguridad. Tiene 180 días de validez. |
| Recepción | ¿Tengo que estar yo para recibir el pedido? | No es necesario. Puede recibirlo cualquier adulto presentando su documento. |
| Precios | ¿Los precios online son los mismos que en los locales? | No necesariamente. Los locales y la web pueden tener promociones y precios distintos. |
| Mayorista | ¿Venden por mayor? | Ingresá a la sección Mayoristas/Franquicias y completá el formulario: https://www.wanama.com/franquicias |
| Empleo | ¿Dónde mando mi CV? | Envianos tu CV a rrhh@wanama.com. |
| Horario | ¿En qué horario atienden? | Lunes a viernes de 8 a 18 hs y sábados de 9 a 12 hs. |
| Sucursales | ¿Tienen un local en Córdoba? | \[sucursales nativas\] — ej.: Paseo del Jockey, Patio Olmos Shopping, Córdoba Shopping, Villa Allende Shopping. |
| Cuenta | ¿Cómo creo una cuenta? | Ingresá al sitio y hacé clic en "Registrarse" (arriba a la derecha), luego completá tus datos. |

# Hallazgos y Riesgos Abiertos

| Punto | Detalle |
| :-: | :-: |
| ⚠️ Retiro en tienda | Tres FAQs se contradicen ("¿Cuentan con retiro en sucursal?", "¿Realizan envíos?" y "LOCALES PARA PICKUP STORE WANAMA"). Consulta enviada al cliente por mail. |
| ⚠️ FAQ de otra marca | La FAQ "LOCALES PARA PICKUP STORE - JOHN L. COOK" está cargada en Wanama y se activa con "Locales" y "Tiendas". Se consulta al cliente si se elimina. |
| ⚠️ FAQ incompleta | La respuesta de "¿Puedo modificar mi pedido?" está cortada en el caso de pago realizado y pedido no despachado. Se consulta al cliente el procedimiento. |
