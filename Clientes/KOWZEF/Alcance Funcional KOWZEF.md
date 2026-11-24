> Fuente: Analisis_Funcional_Kosiuko_Gen6_v1 (https://docs.google.com/document/d/1W9_DYhr5YgJhM5XTSY5QXi93U4AstJvWeX0csiMJ-KM/edit?usp=drivesdk&ouid=113568244751416058438)

# Análisis Funcional — Kosiuko, Agente IA (Gen 6 v1)

2026-11-06

## Cómo usar este documento

Este documento releva el comportamiento vigente del Agente IA de Kosiuko (kosiuko.com, equipo KOWZEF) en Gen 5, como base para su migración a Gen 6. Sigue la estructura del alcance funcional estándar de Snappy Commerce: cada fila define el flujo, sus activadores y el comportamiento esperado.

Sirve para que Producto valide el comportamiento vigente y tenga documentado cada flujo antes de pasar de la UI de Snappy (Gen 5) a código mantenido por Producto (Gen 6). La experiencia del usuario final no cambia con la migración.

Las notas marcadas con ⚠ señalan puntos a confirmar o diferencias entre la configuración actual y el requerimiento. Las configuraciones inactivas se documentan como ❌ Inactivo.

## Requerimientos Funcionales

### Configuración Global

| Flujo nativo | Qué hace el flujo | Valor configurado |
| :-: | :-: | :-: |
| Crear contacto — Campos | Solicita los datos indicados al usuario para crear un contacto. | firstName, email, phone |
| Ticket | Al abrir un ticket, el bot pide los datos requeridos. | Requeridos: ticketReason, ticketSubreason. Opcionales: archivos y comentario |
| Asistencia | Deriva a un asesor cuando un flujo dispara "assistance". | ⚠ Contacto, escalamiento en vivo y ticket fuera de horario no figuran en la config. A confirmar. El bot deja de responder al pedirse asistencia (botlessOnAssistanceRequest) |
| Calificación automática del agente | Pide calificar la atención al salir, finalizar o transferir la conversación. | ✅ Activo. ⚠ A confirmar si equivale a la encuesta de satisfacción (CSAT) |
| Búsqueda de Producto (productSearch) | Permite buscar y recomendar productos del catálogo. | ✅ Activo |
| Promociones nativas | Informa sobre promociones y descuentos vigentes. | ✅ Activo |
| Sucursales (branches) | Provee información sobre sucursales físicas. | ✅ Activo. ⚠ Hoy figura ❌ Inactivo en la config de Gen 5; el requerimiento es que esté prendido |
| Estado de Pedido (orderTracking) | Consulta el estado de un pedido. | ✅ Activo |
| Stock en locales (productBranches) | Consulta disponibilidad de stock en sucursal física. | ❌ Inactivo (lo cubre el flujo ConsultaPorStock con una FAQ) |
| Factura (orderInvoice) | Permite solicitar la factura electrónica de una compra. | ❌ Inactivo |
| Ver carrito (showCart) | Muestra el carrito del usuario. | ❌ Inactivo |
| Agregar al carrito (addProductToCart) | Agrega productos al carrito desde el chat. | ❌ Inactivo |
| Comenzar checkout (checkoutCart) | Genera el link de checkout. | ❌ Inactivo |
| Fin de conversación (endConversation) | Permite cerrar una conversación de forma automática. | ❌ Inactivo |
| Canal MercadoLibre | Canal de mensajería a través de MercadoLibre. | ❌ Inactivo |
| Canales | Canales donde opera el bot. | Web y WhatsApp activos. Instagram y Facebook sin bot |
| Saludo inicial | Mensaje de apertura del bot. | "¡Hola! soy la asistente virtual de KSK 👩🏻‍💻 ¿Necesitas ayuda?" |

### Flujos de Preventa

| Flujo | Activadores | Comportamiento esperado |
| :-: | :-: | :-: |
| Búsqueda y Recomendación de Producto (productSearch) | ✅ Usuario pregunta por un producto, categoría, talle, color o atributo.<br>❌ Consulta de stock en sucursal → ConsultaPorStock | Ejecuta productSearch contra el catálogo y devuelve un carousel. Si no hay resultados o hay ambigüedad, repregunta o deriva a asistencia. ⚠ Activadores tomados del estándar, a validar |
| Promociones (nativo) | ✅ Usuario pregunta por descuentos, promos, cupones u ofertas | Envía el mensaje nativo de Promociones con las vigentes. ⚠ Contenido a validar con el cliente |
| Información de Sucursales (branches) | ✅ Dirección, horarios o sucursal más cercana.<br>❌ Stock de un producto en sucursal → ConsultaPorStock | Informa las sucursales cargadas en la tienda mediante el flujo nativo. ⚠ Hoy ❌ Inactivo en Gen 5; requerimiento: debe estar prendido |
| ConsultaPorStock | ✅ "¿Tienen stock en la sucursal de Palermo?", "¿hay disponibilidad en el local de Rosario?", "¿puedo ir a buscarlo a la sucursal?".<br>❌ Teléfono o dirección de una sucursal · Stock online → responde directo · Quiere comprar → búsqueda | Responde con la FAQ "¿Puedo verificar el stock de un producto en alguna de sus sucursales físicas?": indica contactarse con la sucursal y comparte el listado de tiendas físicas (kosiuko.com/stores) |

### Flujos de Postventa

| Flujo | Activadores | Comportamiento esperado |
| :-: | :-: | :-: |
| Estado de pedido (orderTracking) | ✅ Estado o seguimiento del pedido o envío, con o sin número de orden.<br>❌ Estado de un cambio o devolución → EstadoCambioDevolucion | Nativo activo. Consulta el estado del pedido vía integración |
| GestionarCambio | ✅ "Quiero hacer un cambio", "quiero cambiar de talle", "ya pasé los 30 días, ¿puedo cambiarlo?".<br>❌ Devolver y recuperar el dinero → GestionarDevolucion · Estado de un cambio ya iniciado → EstadoCambioDevolucion · Stock en sucursal → ConsultaPorStock | sector → postsales. notify (mensaje exacto): "Si necesitás gestionar un cambio o devolución, solicitá asistencia y uno de nuestros asesores te ayudará. Recordá que los cambios presenciales pueden realizarse en cualquiera de nuestras tiendas oficiales dentro de los 30 días corridos, presentando el ticket o factura de cambio." Comportamiento definido por el cliente: el flujo no deriva a asistencia por sí mismo |
| GestionarDevolucion | ✅ "Quiero devolver", "quiero que me devuelvan el dinero", "ya pasaron los 10 días, ¿puedo devolver igual?".<br>❌ Cambio de talle o color → GestionarCambio · Estado de una devolución → EstadoCambioDevolucion · Estado del envío → orderTracking | sector → postsales. assistance (ticketReason: devolutions) |
| EstadoCambioDevolucion | ✅ "¿Cómo va mi devolución?", "¿en qué estado está mi cambio?", "¿ya aprobaron mi devolución?".<br>❌ Iniciar un cambio o devolución nuevo · Estado del envío en general → orderTracking | sector → postsales. assistance (ticketReason: returns) |
| RetiraOtro | ✅ "Quiero que lo retire otra persona", "¿cómo autorizo a alguien para que retire?".<br>❌ Dónde queda la sucursal · Seguimiento genérico → orderTracking · Stock en tienda → ConsultaPorStock | sector → postsales. assistance (sin ticketReason) |

### Flujos de Sistema (Automatizados)

| Flujo | Activadores | Comportamiento esperado |
| :-: | :-: | :-: |
| AsistenciaNotificaciones | ✅ Únicamente cuando el usuario envía el mensaje exacto "¡Quiero el beneficio!".<br>❌ Cualquier otra frase, parafraseo o variación | ❌ Inactivo. ⚠ Hoy está activo en Gen 5 (assistance, ticketReason: carritoabandonado); el requerimiento es que esté apagado |
| Mensaje de bienvenida | Sin activador por mensaje del usuario. Se dispara al iniciar el primer contacto | ✅ Activo. Saludo: "¡Hola! soy la asistente virtual de KSK 👩🏻‍💻 ¿Necesitas ayuda?" |
| Seguimiento proactivo | Sin activador por mensaje del usuario. Automático tras un tiempo de inactividad | ❌ Inactivo |
| Cierre por inactividad + CSAT | Sin activador por mensaje del usuario. Automático al quedar la conversación inactiva | ❌ Inactivo |
| Notificaciones / Broadcast | Se dispara desde un envío de notificación/broadcast programado en Labs | ❌ Inactivo |

## Requerimientos No Funcionales

| Categoría | Requerimiento |
| :-: | :-: |
| Lenguaje | Español rioplatense, tono cercano, cordial y resolutivo. Voseo en todos los verbos (querés, podés, decime, tenés). Sin español neutro ni trato de usted. Emojis en nivel medio |
| Rol del agente | Preventa y postventa |
| Canales | Opera en web y WhatsApp. Instagram y Facebook sin bot. MercadoLibre ❌ Inactivo |
| Testing | El código/JSON debe pasar el test suite estándar de Snappy (umbral 85%+) antes de cualquier subida a producción |
| Conversaciones | El agente debe contar con el contexto de las conversaciones previas de cada contacto y poder responder usando esa información |
| Comportamiento — Fallback | Cuando no cuente con información suficiente en Conocimientos, debe ofrecer asistencia de un operador humano |

### Flujos con Lógica Condicional

Sin flujos con lógica condicional. Todos los flujos configurados son lineales.

## Diagramas de Flujo

Sin flujos con lógica condicional. No aplica.

## Test

| Categoría | Input | Output |
| :-: | :-: | :-: |
| Búsqueda de producto — Consulta abierta | Busco una campera negra | \[carrusel de productos\] |
| Búsqueda de producto — Producto puntual | Quiero ver los jeans rectos | \[carrusel de productos\] |
| Promociones | ¿Qué promociones tienen? | \[mensaje nativo de Promociones\] |
| Sucursales | ¿Dónde queda el local más cercano? | \[información de sucursales\] |
| Stock en sucursal | ¿Tienen stock en la sucursal de Palermo? | "Deberás contactarte con la sucursal para consultar su stock disponible, te dejamos el link con el listado de tiendas físicas: https://www.kosiuko.com/stores" |
| Estado de pedido | ¿Dónde está mi pedido? | \[estado de pedido\] |
| Cambio | Quiero hacer un cambio | \[sector postsales\] → "Si necesitás gestionar un cambio o devolución, solicitá asistencia y uno de nuestros asesores te ayudará. Recordá que los cambios presenciales pueden realizarse en cualquiera de nuestras tiendas oficiales dentro de los 30 días corridos, presentando el ticket o factura de cambio." |
| Cambio — Fuera de plazo | Lo recibí hace 2 meses, ¿puedo cambiarlo igual? | Mismo mensaje que el caso anterior |
| Devolución | Quiero devolver un producto | \[sector postsales\] → \[solicitar asistencia — ticketReason: devolutions\] |
| Devolución — Fuera de plazo | Ya pasaron los 10 días, ¿puedo devolver igual? | \[sector postsales\] → \[solicitar asistencia — ticketReason: devolutions\] |
| Estado de cambio o devolución | ¿Cómo va mi devolución? | \[sector postsales\] → \[solicitar asistencia — ticketReason: returns\] |
| Retiro por tercero | Va a retirar mi pedido mi pareja | \[sector postsales\] → \[solicitar asistencia\] |
| Asistencia humana | Quiero hablar con un asesor | \[solicitar asistencia\] |
| Políticas de cambios y devoluciones | ¿Cuáles son las políticas de cambios y devoluciones? | \[Respuesta por Conocimientos\] Los cambios se aceptan hasta 30 días corridos desde la recepción del pedido. Las devoluciones, dentro de los 10 días corridos y solo por el canal online. Ambos plazos son sin excepciones |
| Envíos | ¿Cómo se hacen los envíos? | \[Respuesta por Conocimientos\] Andreani a domicilio: AMBA 3 a 4 días hábiles, resto del país 7 días hábiles. Saires: AMBA 72 hs hábiles. Same Day: compras antes de las 12 PM, solo CABA y GBA. Pick Up Store: gratis en tiendas adheridas |
| Medios de pago | ¿Cuáles son los medios de pago? | \[Respuesta por Conocimientos\] Tarjeta de crédito o débito, dinero en cuenta de MercadoPago, Mercado Crédito, efectivo por Rapipago o Pago Fácil. No se acepta transferencia bancaria |
| Seguimiento | ¿Dónde encuentro el número de seguimiento? | \[Respuesta por Conocimientos\] En el email de la compra o en el menú "Soporte en mi compra realizada" |
| Reembolso | ¿Cuánto tarda el reembolso? | \[Respuesta por Conocimientos\] Entre 5 y 10 días hábiles, según el medio de pago |
| Producto dañado | Me llegó un pedido dañado | \[Respuesta por Conocimientos\] Se lamenta el inconveniente y se indica el menú "Soporte en compra realizada" para el cambio o devolución |
| Primera compra | ¿Tienen descuento para la primera compra? | \[Respuesta por Conocimientos\] 10% OFF al suscribirse al newsletter, con el código como cupón al finalizar la compra |
| Horarios de atención | ¿Cuáles son los horarios de atención? | \[Respuesta por Conocimientos\] Lunes a viernes de 9 a 18 hs |
| Venta mayorista | ¿Hacen ventas mayoristas? | \[Respuesta por Conocimientos\] Se indica escribir por mail a dlancini@kosiuko.com |
| Garantía | ¿Qué garantía tienen los productos? | \[Respuesta por Conocimientos\] Los productos tienen garantía; ante un defecto de fabricación se contactan dentro de los plazos establecidos para el cambio o devolución |
| App y puntos | ¿Cómo funciona el sistema de puntos de la app? | \[Respuesta por Conocimientos\] 10% del monto de cada compra en puntos, canjeables por cupones en la sección REWARDS CLUB de la app. Se identifican por DNI |
| Outlet | ¿Tienen outlet? | \[Respuesta por Conocimientos\] Sí, en https://www.kosiukooutlet.com/ |
| Preorder | ¿Qué son los productos de preorder? | \[Respuesta por Conocimientos\] Compra anticipada de productos que aún no están en locales, con fecha estimada de entrega en el detalle del producto |
| Fallback | (consulta sin información en Conocimientos) | "No tengo información, voy a solicitar asistencia de un asesor" \[solicitar asistencia\] |
| Flujo inactivo | ¡Quiero el beneficio! | No ejecuta ningún flujo. Responde con lo que tenga cargado en C |

\[El texto devuelto por Drive termina cortado en "cargado en C"; el resto del documento no estaba disponible.\]
