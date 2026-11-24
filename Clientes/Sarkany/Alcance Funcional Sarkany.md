> Fuente: Analisis_Funcional_Sarkany_V2 (https://docs.google.com/document/d/1OQRRa836ZaagJ_lIZEBYY38lX-bwDXgcW6tikl_0rBk/edit?usp=drivesdk&ouid=113568244751416058438)

**Snappy Commerce**

# Análisis Funcional — Ricky Sarkany, Agente IA

*Ricky Assistant · Relevamiento del comportamiento vigente en producción (Gen 5) como base para la migración a Gen 6*

## Cómo usar este documento

Este documento releva el comportamiento actual del Agente IA de Ricky Sarkany — flujos nativos, flujos personalizados y configuración global — siguiendo la estructura del alcance funcional estándar de Snappy Commerce. Cada fila define el flujo, sus activadores y el comportamiento esperado.

Sirve para que Producto valide el comportamiento vigente, detecte vacíos funcionales e inconsistencias de configuración, y tenga documentado cada flujo antes de que pase de la UI de Snappy (Gen 5) a código TypeScript mantenido por Producto (Gen 6). La experiencia del usuario final no cambia con la migración.

*Las notas marcadas con ⚠ señalan flujos desactivados o puntos a confirmar. Los hallazgos y riesgos abiertos se listan al final del documento.*

# Requerimientos Funcionales

## Configuración Global

| **Flujo nativo** | **Qué hace el flujo** | **Valor configurado** |
|---|---|---|
| Crear contacto - Campos a solicitar al usuario | Para crear un contacto nuevo, el agente IA solicita de base estos 3 datos. | firstName, email, phone |
| Comenzar checkout - Requisitos de información de contacto | Para generar un checkout alcanza con nombre y email — no pide teléfono en ese paso, a diferencia de createContact. | firstName, email |
| Asistencia | Toda vez que un flujo dispara "assistance", siempre se piden datos de contacto. Si es horario de atención se escala en vivo; si no, se genera un ticket. | contact: always<br>escalate: in_hours<br>ticket: out_of_hours |
| Encuesta de satisfacción | La Encuesta de Satisfacción automática global está desactivada. Hoy no hay ningún mecanismo automático de CSAT activo. | true |
| Sucursales / Branches (productBranches) | Flujo nativo en donde brinda información brindada en snappy | true |
| Stock en local | El flujo nativo de disponibilidad de stock en sucursal está desactivado — lo reemplaza StockEnLocales, que deriva la consulta a un humano. | false |

## Flujos de Pre-Venta

| **Flujo** | **Activadores** | **Comportamiento esperado** |
|---|---|---|
| Búsqueda y Recomendación de Producto | Usuario pregunta por un producto, categoría, talle, color o atributo ("tenés zapatillas rojas talle 40?"). | Ejecuta productSearch contra el catálogo del cliente y devuelve un carousel de resultados. Si no hay resultados o hay demasiada ambigüedad, repregunta o deriva a asistencia |
| Checkout Asistido | Se activa automáticamente al confirmarse el agregado al carrito. No se activa sin ese contexto previo. | Pide nombre y email solo si no están en contexto (con confirmación de grafía), ofrece cross-sell una sola vez y comparte un único link de checkout de VTEX. Incluye seguimiento si no completa la compra. |
| Checkout Asistido — Gestión del Carrito | Usuario quiere agregar, borrar o modificar productos del carrito antes de finalizar. | Reabre Búsqueda según la acción, valida stock y actualiza el mismo ítem. Si el carrito queda vacío, vuelve a Búsqueda. Regla común: el link se comparte una sola vez, al confirmar que terminó de armar el carrito. |
| PromocionesBancarias | Consultas sobre promociones, descuentos, cuotas bancarias o medios de pago.<br>✅ "¿Qué promociones tienen?", "¿Hay 2x1?", "¿Cuotas sin interés con Visa?", "Tengo tarjeta Galicia, ¿cuotas?"<br>❌ Pago de compra específica → productSearch/checkout · Nueva temporada → NuevaTemporada · Stock → StockEnLocales | Busca promos en Conocimientos (sync automático con la web). Distingue bancos de financieras (configurado en el comportamiento). Reemplaza al flujo nativo de Promociones. |
| StockEnLocales | Consulta por stock o disponibilidad en una sucursal física.<br>✅ "¿Tienen este producto en el local de Palermo?", "¿Hay stock en la sucursal?"<br>❌ Stock en la web → productSearch | sector → sales · topic → consultaDeStock<br>Solicitud de asistencia → ticketReason: questionsDuring<br>Campo adicional: nombre del local (se solicita si no fue indicado). |
| ProblemasParaComprar | Usuario indica que tiene problemas para comprar.<br>✅ "No me anda la web", "Error en el checkout", "No puedo pagar", "No me toma la tarjeta"<br>❌ Usuario que pregunta cómo comprar → productSearch o FAQ<br>También se activa cuando el bot detecta la falla durante un flujo de venta en curso (el link de VTEX no se genera, o no se confirma el agregado al carrito). | assistance → ticketReason: buyingIssues<br>Campo adicional: captura de pantalla donde se vea el error. Si no la tiene, guarda "false" y continúa. |

## Flujos de Post-Venta

| **Flujo** | **Activadores** | **Comportamiento esperado** |
|---|---|---|
| Estado de Pedido | Usuario quiere saber en qué estado está su pedido, cuándo llega, o busca el número de seguimiento. | Envía un mensaje orderStatus con el estado, fecha estimada y tracking (si está disponible) vía la integración del cliente. |
| Estado de pedido no existe | Sub-estado interno: se dispara cuando la búsqueda de estado de pedido no encuentra el pedido. | Responde: "Parece que el pedido no existe, si necesitás ayuda podés escribir 'hablar con una asesora'" |
| Reclamos | Usuario reporta un problema (producto dañado, faltante, error de facturación, etc.). | Genera un ticket en el HelpDesk/CRM con el ticketReason y ticketSubreason correspondientes, solicitando los datos requeridos. |
| CambiosYDevoluciones | Usuario quiere hacer un cambio o devolución, sin especificar si fue por web o local.<br>✅ "Quiero hacer un cambio", "Quiero devolver un producto" | sector → postsales · topic → infoSobreCambiosYDevoluciones<br>Ramifica por origen de compra (local físico / online Dreamstore / evento especial) y aplica las excepciones cargadas. Diagrama 1. |
| EstadoDeCambioODevolucion | Usuario ya gestionó un cambio o devolución y quiere saber el estado.<br>✅ "¿Cuál es el estado de mi cambio?", "Ya hice el cambio, ¿cuándo llega?"<br>❌ Quiere iniciar un cambio → CambiosYDevoluciones | assistance → ticketReason: after |
| Cancelaciones | Usuario quiere cancelar su compra.<br>Ejemplos: "Quiero cancelar mi compra", "Cancelar pedido", "No quiero recibir el pedido" | Dependiendo de si compro online o en local fisico responde distintas faqs. Diagrama 2. |
| PaginaFraudulenta | Usuario compró en una página fraudulenta o reporta una copia de la marca.<br>✅ "Estafa", "ajustaelmodelo.shop", "Hay una copia de su página", "Plagiaron" | Solicitud de asistencia → ticketReason: fraude |

## Flujos de Sistema (Automatizados)

| **Flujo** | **Activadores** | **Comportamiento esperado** |
|---|---|---|
| MensajeDeBienvenida | Sin activador por mensaje del usuario — se dispara desde automatizaciones. | Activo |
| SeguimientoProactivo | Sin activador por mensaje del usuario — automatización por inactividad. | ⚠ Desactivado. La automatización existe pero está inactiva. |
| CierrePorInactividad + CSAT | Sin activador por mensaje del usuario — automatización al quedar la conversación inactiva. | Activo |
| Post-notification action | Respuesta de un usuario a una notificación o plantilla de WhatsApp. | Activo — dispara una solicitud de asistencia inmediata. |

*⚠ Hoy los cuatro flujos automatizados están desactivados salvo el post-notification action: no hay bienvenida diaria, ni seguimiento proactivo, ni CSAT automático.*

## Flujos con Lógica Condicional

Los siguientes flujos tienen ramificaciones internas y cuentan con un diagrama de decisión propio en la sección Diagramas de Flujo:

- CambiosYDevoluciones: Ramifica según el origen de la compra (local físico / online Dreamstore / evento especial) y por las excepciones cargadas (Outlet, temporada anterior, 2x1, Live IG, SARKANY ID). Si no encaja, escala a un asesor.
- Cancelaciones: Ramifica según si el usuario acepta o no derivar el caso a un asesor. El estado de despacho lo confirma el asesor, no el bot.

## Diagramas de Flujo

Representación visual de la lógica de decisión de los flujos condicionales listados arriba, tal como está configurada hoy en producción.

*Versión editable de estos diagramas (formas nativas, sin necesidad de recrearlos)*

### Diagrama 1- Cambios y Devoluciones

*(imagen/diagrama no incluido en el texto exportado del documento)*

### Diagrama 2- Cancelaciones

*(imagen/diagrama no incluido en el texto exportado del documento)*

# Requerimientos No Funcionales

| **Categoría** | **Requerimiento** |
|---|---|
| Lenguaje | Español rioplatense, tono profesional pero cercano, ameno y cordial. Voseo en todos los verbos (querés, podés, decime, tenés). Nunca "che", ni conjugación de "tú", ni español neutro o de España, ni trato de usted, ni jerga burda. |
| Testing | El código/JSON debe pasar el test suite estándar de Snappy (umbral 85%+) antes de cualquier subida a producción. |
| Conversaciones | El agente debe contar con contexto de las conversaciones previas de cada contacto y poder responder usando esa información (ej.: reconocer a qué ticket se refiere el usuario en una conversación nueva). |
| Comportamiento | Personalizar el primer mensaje del Agente IA por canal. |
| Comportamiento — Fallback | Cuando no cuente con información suficiente en Conocimientos, debe ofrecer solicitar asistencia de un operador humano. |
| Comportamiento — Proactividad y reencauce | Ante silencio de 5 minutos en un flujo de venta en curso, reactiva la conversación con un mensaje de seguimiento. Ante desvío de tema, responde la consulta puntual y luego reencauza hacia la venta en curso, sin abandonarla. |

## Integraciones

| **Sistema / Canal** | **Propósito** |
|---|---|
| WhatsApp Business API | Recepción y envío de mensajes de WhatsApp. |
| eCommerce VTEX y VTEX IO | Búsqueda de productos, checkout, carritos abandonados. |
| Meta | Canales Instagram y Facebook (mensajes directos y comentarios). |
| Mails | Envío y recepción de emails. |
| Shipnow | Envíos (CABA: Same Day / Next Day). |
| Andreani | Envíos al resto del país y logística de cambios contraentrega. |

## Criterios de Aceptación

- Cada flujo migrado a Gen 6 reproduce el comportamiento de Gen 5 documentado en este alcance, con los hallazgos ya corregidos.
- Ricky Assistant pasa el test suite estándar de Snappy con un umbral de 85%+.
- El agente diferencia tarjetas bancarias de tarjetas de entidades financieras al informar cuotas y promociones.
- Se incorporan todos los requerimientos no funcionales detallados en este documento.
- El agente deriva a un operador humano cuando no disponga de información en sus Conocimientos para resolver la consulta.

# Test

**Consideraciones para CX**

- Los ejemplos que se describen a continuación deben **reflejar el comportamiento que espera el cliente**, independientemente del comportamiento actual.
- Contemplar la cantidad de mensajes que tendrá el output.
- Asegurar que las **indicaciones no se contradigan** (ejemplo: búsqueda por un producto puntual y uso de facetas).
- Indicar las **acciones entre corchetes,** por ejemplo: [carrusel de productos], [solicitar asistencia].
- Cada X tiempo se va a correr el test de los casos indicados a continuación. Por lo tanto, **los ejemplos se deben actualizar si algún comportamiento se modifica**.
- Debe incluir todas las funcionalidades encendidas, todos los flows nativos y custom y los conocimientos principales asociados a ellos.

| **Categoría** | **Input** | **Output** |
|---|---|---|
| **Búsqueda de producto** - Producto puntual | Quiero el Stiletto Amanda | Tenemos el Stiletto Amanda disponible. Es un taco alto en cuero charol con hebilla metálica dorada. Precio: $145.000 (antes $165.000) |
| **Búsqueda de producto** - Coincidencias múltiples | Quiero el Amanda | Encontré varios modelos Amanda: Stiletto Amanda Negro ($145.000), Amanda Charol Nude ($138.000) y Amanda Metalizado ($152.000). ¿Cuál te interesa? |
| **Búsqueda de producto -** Categorías | Quiero ver zapatos | Estos son algunos de nuestros zapatos más elegidos: [carrusel]. ¿Te gusta alguno o buscás algo en particular? |
| **Búsqueda de producto -** Consulta abierta | Busco un zapato elegante para una fiesta de noche | Para una fiesta de noche te recomiendo estos: [carrusel de opciones]. ¿Alguno te gusta? |
| **Búsqueda de producto -** Por facetas | Busco el zapato Ricky. | Para confirmarte el modelo, decime por favor: 1) ¿Color? 2) ¿Qué talle usás?" |
| **Conocimientos -** Si el bot no tiene información | Hola necesito saber cual es la diferencia entre la suela inyectada y la suela de goma eva? | No tengo información, voy a solicitar asistencia de un asesor<br>[solicitar asistencia] |
| Dudas sobre talle | No sé cuál es mi talle. | [Respuesta por Conocimientos, ingresar el **contenido** de la FAQ]<br>Tenés que medir tu pie apoyándolo sobre una hoja de papel contra una pared. Marca donde termina el dedo más largo y mide la distancia desde la pared hasta esa marca en centímetros. Luego, compará esa medida con nuestra tabla de talles. Guía de tallas de calzado: Talle 35 ARG: 23.0 cm \| US 5 \| EUR 36 Talle 36 ARG: 23.5 cm \| US 6 \| EUR 37 Talle 37 ARG: 24.0 cm \| US 7 \| EUR 38 Talle 38 ARG: 24.5 cm \| US 8 \| EUR 39 Talle 39 ARG: 25.0 cm \| US 9 \| EUR 40 Talle 40 ARG: 25.5 cm \| US 10 \| EUR 41  No tenemos talles 41, 42, 43 o 44 de mujer. |
| Dudas sobre talle | ¿Cuál sería mi talle? | [Respuesta por Conocimientos, ingresar el **contenido** de la FAQ]<br>Tenés que medir tu pie apoyándolo sobre una hoja de papel contra una pared. Marca donde termina el dedo más largo y mide la distancia desde la pared hasta esa marca en centímetros. Luego, compará esa medida con nuestra tabla de talles. Guía de tallas de calzado: Talle 35 ARG: 23.0 cm \| US 5 \| EUR 36 Talle 36 ARG: 23.5 cm \| US 6 \| EUR 37 Talle 37 ARG: 24.0 cm \| US 7 \| EUR 38 Talle 38 ARG: 24.5 cm \| US 8 \| EUR 39 Talle 39 ARG: 25.0 cm \| US 9 \| EUR 40 Talle 40 ARG: 25.5 cm \| US 10 \| EUR 41  No tenemos talles 41, 42, 43 o 44 de mujer. |
| Dudas sobre talle | Guía de talles | [Respuesta por Conocimientos, ingresar el **contenido** de la FAQ]<br>Tenés que medir tu pie apoyándolo sobre una hoja de papel contra una pared. Marca donde termina el dedo más largo y mide la distancia desde la pared hasta esa marca en centímetros. Luego, compará esa medida con nuestra tabla de talles. Guía de tallas de calzado: Talle 35 ARG: 23.0 cm \| US 5 \| EUR 36 Talle 36 ARG: 23.5 cm \| US 6 \| EUR 37 Talle 37 ARG: 24.0 cm \| US 7 \| EUR 38 Talle 38 ARG: 24.5 cm \| US 8 \| EUR 39 Talle 39 ARG: 25.0 cm \| US 9 \| EUR 40 Talle 40 ARG: 25.5 cm \| US 10 \| EUR 41  No tenemos talles 41, 42, 43 o 44 de mujer. |
| Dudas sobre talle | ¿Qué talle me queda? | [Respuesta por Conocimientos, ingresar el **contenido** de la FAQ]<br>Tenés que medir tu pie apoyándolo sobre una hoja de papel contra una pared. Marca donde termina el dedo más largo y mide la distancia desde la pared hasta esa marca en centímetros. Luego, compará esa medida con nuestra tabla de talles. Guía de tallas de calzado: Talle 35 ARG: 23.0 cm \| US 5 \| EUR 36 Talle 36 ARG: 23.5 cm \| US 6 \| EUR 37 Talle 37 ARG: 24.0 cm \| US 7 \| EUR 38 Talle 38 ARG: 24.5 cm \| US 8 \| EUR 39 Talle 39 ARG: 25.0 cm \| US 9 \| EUR 40 Talle 40 ARG: 25.5 cm \| US 10 \| EUR 41  No tenemos talles 41, 42, 43 o 44 de mujer. |
| Informacion de envios | ¿Cuánto tarda el envío? | El envío con Andreani tarda entre 7 y 15 días hábiles, dependiendo de la zona. Podés elegir que te lo envíen a domicilio o a una sucursal del correo. Si no pueden entregarlo, hacen dos visitas y luego queda en custodia 5 días hábiles en la sucursal más cercana para que lo retires. Si tampoco lo retirás, el pedido vuelve a nuestro depósito y te contactamos.<br>Importante: si la dirección es inexistente, el pedido vuelve directo a nuestras oficinas sin pasar por la sucursal. No podemos confirmar el tiempo exacto de entrega en este momento. ¿Querés que te ayude con algo más? |
| Informacion de envios | ¿Hacen envíos a [ciudad]? | Tenés disponible en la página de cada producto la opción de calcular tiempos y costos de envío. También podés consultarlo en este link: https://www.rickysarkany.com/tiempos-y-metodos-de-envios<br>\*Los envíos Same Day y Next Day aplican para algunos CP de CABA y GBA. Al colocar los datos de envío y CP en el checkout de compra te indicará si podés acceder a esta modalidad de envío. |
| Informacion de envios | ¿Tienen envío gratis? | Tenemos envios gratuitos a partir de 370000 |
| **Cambios y Devoluciones** - Plazos | ¿Cuántos días tengo para cambiar un producto? | **[Respuesta por FAQ]** Podés realizar el cambio dentro de los 30 días corridos a partir de la fecha de recepción de tu compra. El producto debe estar sin uso, en perfectas condiciones y con sus etiquetas y empaque original. |
| **Cambios y Devoluciones** - Costo logístico | ¿El primer cambio tiene costo de envío? | **[Respuesta por Conocimiento]** El primer cambio por talle o modelo no tiene costo de envío dentro del territorio nacional. A partir del segundo cambio de un mismo pedido, los costos logísticos corren por cuenta del cliente. |
| **Métodos de Pago** - Financiación | ¿Qué medios de pago aceptan y cuántas cuotas ofrecen? | **[Respuesta por FAQ]** Aceptamos tarjetas de crédito y débito (Visa, Mastercard, AMEX), Mercado Pago y transferencia bancaria. Contamos con 3 y 6 cuotas sin interés en productos seleccionados. |
| **Seguimiento de Pedido** - Tracking | ¿Cómo hago el seguimiento de mi pedido? | **[Order Status]** Para poder buscar el estado de tu pedido necesito que me indiques el número de tu compra. |
| **Locales y Horarios** - Puntos de venta | ¿Dónde tienen locales en CABA? | **[Respuesta por FAQ]** Contamos con sucursales en Palermo, Recoleta Mall, Alto Palermo, Unicenter y Galerías Pacífico. Podés consultar direcciones y horarios en la sección "Locales" de nuestra web. |
| **Facturación** - Factura A | ¿Emiten Factura A? | **[Respuesta por Conocimiento]** Sí, emitimos Factura A. Para solicitarla, debes seleccionar la opción correspondiente e ingresar tu CUIT y razón social en el checkout antes de finalizar la compra. |
| **Cuidado de Producto** - Mantenimiento | ¿Cómo limpio unas botas de gamuza? | **[Respuesta por Conocimiento]** Para calzado de gamuza, recomendamos usar un cepillo de cerdas suaves seco para quitar el polvo superficial y aplicar un protector en aerosol específico. Evitá el contacto directo con agua. |
| **Cupones** - Uso de cupón | Tengo un código de descuento, ¿dónde lo ingreso? | **[Respuesta por FAQ]** Podés ingresar tu código promocional en la casilla "Cupón de descuento" dentro del carrito de compras, antes de confirmar la opción de pago. |
| **Gift Cards** - Compra y uso | ¿Venden tarjetas de regalo digitales? | **[Respuesta por FAQ]** Sí, ofrecemos Gift Cards virtuales por montos fijos. Se envían por e-mail al destinatario con un código único para canjear en la tienda online dentro de los 180 días. |
| **Información de Envíos** - Retiro | ¿Puedo retirar mi compra por un local? | **[Respuesta por FAQ]** Sí, contamos con la opción de retiro gratuito en sucursales seleccionadas. Podés elegir el punto de retiro durante el proceso de compra en el checkout. |
