> Fuente: Analisis_Funcional_ArenaColombia_Gen6_v1 (https://docs.google.com/document/d/1_KixT-EQ1fM2QVqa8RO51ZJlpu5tfBCCdKjWMFnnEsI/edit?usp=drivesdk&ouid=113568244751416058438)

**Snappy Commerce**

# Análisis Funcional — Arena Colombia, Agente IA

*Arena Bot · Relevamiento del comportamiento vigente en producción (Gen 5) como base para la migración a Gen 6*

# Cómo usar este documento

Este documento releva el comportamiento actual del Agente IA de Arena Colombia — flujos nativos, flujos personalizados y configuración global — siguiendo la estructura del alcance funcional estándar de Snappy Commerce. Cada fila define el flujo, sus activadores y el comportamiento esperado.

Sirve para que Producto valide el comportamiento vigente, detecte vacíos funcionales e inconsistencias de configuración, y tenga documentado cada flujo antes de que pase de la UI de Snappy (Gen 5) a código TypeScript mantenido por Producto (Gen 6). La experiencia del usuario final no cambia con la migración.

*Las notas marcadas con ⚠ señalan flujos desactivados o puntos a confirmar. Los hallazgos y riesgos abiertos se listan al final del documento.*

# Requerimientos Funcionales

## Configuración Global

| Flujo nativo | Qué hace el flujo | Valor configurado |
| :-: | :-: | :-: |
| Crear contacto — Campos a solicitar al usuario | Para crear un contacto nuevo, el agente solicita los datos de identificación del usuario. Incluye un campo adicional de identificación fiscal para el contexto colombiano. | firstName, email, phone + Cédula o NIT (campo adicional) |
| Comenzar checkout — Requisitos de información de contacto | Para generar un checkout, solicita nombre, teléfono, email y dirección de envío completa. La dirección es obligatoria para calcular costo y método de entrega. | firstName, phone, email + dirección de envío (shippingAddress: true) |
| Asistencia | Siempre que se dispara asistencia, se solicitan datos de contacto. En horario de atención se escala a un asesor humano. Fuera de horario se ejecuta el flujo OutOfHours: informa el horario de atención y redirige al formulario de contacto. | Contacto: siempre<br>Escalar: dentro de horario<br>OutOfHours: fuera de horario |
| Encuesta de satisfacción | Encuesta de satisfacción automática (CSAT) al cierre de conversaciones. | DESACTIVADO |

## Flujos de Pre-Venta

| Flujo | Activadores | Comportamiento esperado |
| :-: | :-: | :-: |
| Búsqueda y Recomendación de Producto | Usuario pregunta por un producto, categoría, talla, color u otro atributo.<br>✅ "¿Tienen trajes de baño talla M?", "Busco gafas de natación"<br>❌ Consulta de stock en tienda → Consultas por Stock | Ejecuta productSearch contra el catálogo de Arena Colombia y devuelve un carousel de resultados. Si no hay coincidencias o la consulta es ambigua, repregunta o deriva a asistencia. |
| Checkout Asistido | Usuario quiere comprar o solicita el link de compra. Incluye la opción de retiro en tienda física.<br>✅ "Quiero comprar", "¿Cómo pago?", "Quiero retirar en tienda" | Solicita firstName, phone, email y dirección de envío si no están en contexto. Genera el link de checkout. Si el usuario elige retiro en tienda, debe ingresar la dirección del local como destino de envío y luego preguntar para que la tienda confirme cuándo puede pasar a retirar. |
| Promociones | Usuario pregunta por descuentos, promociones u ofertas.<br>✅ "¿Qué promociones tienen?", "¿Hay descuentos?", "HotSale" | Responde con el enlace a la página de promociones: https://www.arenacolombia.com/Promociones |
| Información de Tiendas Físicas | Usuario pregunta por dirección, horarios o tienda más cercana.<br>✅ "¿Dónde tienen tiendas?", "¿Hay local en Bogotá?" | Ejecuta la búsqueda de sucursales cargadas en Snappy. |
| Consultas por Stock | Consulta por stock o disponibilidad en una sucursal física.<br>✅ "¿Tienen este producto en el local de Bucaramanga?", "¿Hay stock en la sucursal?"<br>❌ Stock en la web → productSearch | Responde con el contenido de la FAQ: el stock disponible en tiendas físicas se puede verificar contactando directamente al punto de venta. El usuario puede encontrar los teléfonos y horarios de atención de cada tienda en la página web. |
| FAQ Comercial | Usuario consulta sobre talles/medidas, medios de pago, costos de envío, cómo comprar online, cupones de descuento, newsletter u otras consultas frecuentes de preventa. | Responde con el contenido de la Base de Conocimientos (FAQs) configurada para cada tema. |

## Flujos de Post-Venta

| Flujo | Activadores | Comportamiento esperado |
| :-: | :-: | :-: |
| Estado de Pedido | Usuario pregunta por el estado de su pedido, fecha de entrega o número de guía.<br>✅ "¿Dónde está mi pedido?", "¿Cuándo llega mi compra?" | Solicita número de pedido o cédula/NIT. Consulta el estado vía integración y devuelve la información disponible (estado, tracking, fecha estimada según integración). |
| Cambios | Usuario quiere cambiar un producto ya recibido.<br>✅ "Quiero cambiar una prenda", "¿Cómo hago un cambio?", "Recibí un talle equivocado" | Ramifica según los días transcurridos desde la recepción y la disponibilidad de tienda física Arena en la ciudad del usuario. Ver Diagrama 1. |
| Devoluciones | Usuario quiere devolver un producto y solicitar reembolso.<br>✅ "Quiero devolver un producto", "¿Cómo pido la devolución del dinero?" | Responde por FAQ: aplica únicamente por derecho de retracto dentro de los 5 días hábiles siguientes a la recepción. Redirige al formulario PQRS: https://forms.cloud.microsoft/r/TqdfD7VhAZ<br>Reembolso en hasta 30 días calendario desde la recepción en sede principal. |
| Garantías | Usuario reporta un producto con defecto de fabricación o falla de calidad.<br>✅ "Me llegó un producto dañado", "Tiene una falla de fábrica" | Responde por FAQ: garantía válida por 30 días calendario desde la recepción. Redirige al formulario PQRS: https://forms.cloud.microsoft/r/TqdfD7VhAZ<br>Tiempo de respuesta: hasta 15 días hábiles. |
| Reclamos | Usuario reporta un problema post-compra (pedido errado, error de facturación, etc.).<br>✅ "Me llegó un pedido incorrecto", "Tengo un reclamo", "Error en factura" | Responde por FAQ que redirige al formulario PQRS: https://forms.cloud.microsoft/r/TqdfD7VhAZ<br>No genera ticket. Tiempo de inicio de gestión: hasta 15 días hábiles. |
| Cancelaciones | Usuario quiere cancelar su compra.<br>✅ "Quiero cancelar mi pedido", "Cancelar compra" | Responde por FAQ: debe comunicarse con la línea de atención al cliente (310 761 7408) dentro del horario de atención. La cancelación está sujeta al estado del pedido. |

## Flujos de Sistema (Automatizados)

| Flujo | Activadores | Comportamiento esperado |
| :-: | :-: | :-: |
| Menú Principal | El usuario inicia la conversación con un saludo o solicita el menú principal.<br>✅ "Hola", "Buenos días", "Menú", "Volver al inicio" | Saluda al usuario: "¡Hola! Bienvenido a Arena Colombia" (el emoji del documento fuente no se exportó). Muestra el menú principal con las opciones disponibles. Si ninguna opción aplica, indica que puede escribir su consulta libremente. |
| Menú Pedido | El usuario menciona la palabra "pedido" en cualquier mensaje.<br>✅ "Estado de pedido", "Cancelar pedido", "Modificar pedido", "Cambiar dirección" | Muestra el submenú de opciones relacionadas con pedidos para redirigir la consulta al flujo correspondiente (estado, modificación, cancelación). |
| OutOfHours | Se activa automáticamente cuando el usuario solicita asistencia fuera del horario de atención configurado. | ¡Hola! (emoji no exportado) En este momento, nuestro equipo de Atención al Cliente se encuentra fuera de horario. El horario de atención es de lunes a jueves de 7 a 16hs y viernes de 7 a 13hs. En caso de necesitar asistencia, por favor completar el siguiente formulario https://forms.cloud.microsoft/r/TqdfD7VhAZ para que nuestro equipo esté al tanto y pueda asistirte cuando retome la atención. Muchas gracias. |
| Mensaje de Bienvenida Automático | Sin activador por mensaje del usuario — se dispara desde automatizaciones al iniciar el primer contacto. | DESACTIVADO |
| Seguimiento Proactivo | Sin activador por mensaje del usuario — automático tras un tiempo configurable de inactividad. | DESACTIVADO |
| Cierre por Inactividad + CSAT | Sin activador por mensaje del usuario — automático al quedar la conversación inactiva. | DESACTIVADO |
| Notificaciones / Broadcast | Se dispara desde un envío de notificación o broadcast programado en Labs. | DESACTIVADO |

## Flujos con Lógica Condicional

Los siguientes flujos tienen ramificaciones internas y cuentan con un diagrama de decisión propio en la sección Diagramas de Flujo:

- Cambios: Ramifica según los días transcurridos desde la recepción del producto y la disponibilidad de tienda física Arena en la ciudad del usuario.

*⚠ Cualquier comportamiento no cubierto explícitamente en este documento debe confirmarse con el referente técnico del cliente antes de la implementación.*

# Diagramas de Flujo

Representación visual de la lógica de decisión de los flujos condicionales listados arriba, tal como está configurada hoy en producción.

## Diagrama 1 — Cambios

*(El diagrama es una imagen en el documento fuente y no se exporta como texto.)*

# Requerimientos No Funcionales

| Categoría | Requerimiento |
| :-: | :-: |
| Lenguaje | Español colombiano. Tono profesional pero cercano, ameno y cordial. ⚠ Las FAQs actuales presentan uso inconsistente entre "podés" (voseo), "puedes" (tú) y "usted" — se recomienda unificar en Gen 6 con el tratamiento que defina el cliente. |
| Testing | El código/JSON debe pasar el test suite estándar de Snappy (umbral 85%+) antes de cualquier subida a producción. |
| Conversaciones | El agente debe contar con contexto de las conversaciones previas de cada contacto y poder responder usando esa información. |
| Identificación del usuario | Para consultas de estado de pedido, el agente solicita número de pedido o cédula/NIT (campo adicional configurado en createContact). |
| Comportamiento — Fallback | Cuando no cuente con información suficiente en la Base de Conocimientos, debe redirigir al formulario PQRS o a los canales de contacto (cliente@arenacolombia.com / 310 761 7408). |
| Comportamiento — Fuera de horario | Fuera del horario de atención (lunes a jueves 7:00-16:00hs, viernes 7:00-13:00hs), el agente no escala a humano: ejecuta el flujo OutOfHours y redirige al formulario de contacto. |
| Horario de atención | Lunes a jueves: 7:00 a.m. a 4:00 p.m. / Viernes: 7:00 a.m. a 1:00 p.m. (No se atiende sábados, domingos ni festivos) |

# Criterios de Aceptación

- Cada flujo migrado a Gen 6 reproduce el comportamiento de Gen 5 documentado en este alcance.
- Arena Bot pasa el test suite estándar de Snappy con un umbral de 85%+.
- El flujo de Cambios aplica correctamente la lógica condicional: días transcurridos (≤/> 30) y disponibilidad de tienda Arena en la ciudad.
- El flujo OutOfHours se ejecuta correctamente fuera del horario de atención configurado, con el mensaje y el enlace al formulario.
- El agente identifica al usuario por número de pedido o cédula/NIT en las consultas de estado de pedido.
- El flujo de Reclamos no genera ticket: redirige directamente al formulario PQRS.
- El agente redirige al formulario PQRS o canales de contacto cuando no disponga de información suficiente en su Base de Conocimientos.

# Testeos

**Consideraciones para CX**

- Los ejemplos que se describen a continuación deben **reflejar el comportamiento que espera el cliente**, independientemente del comportamiento actual.
- Contemplar la cantidad de mensajes que tendrá el output.
- Asegurar que las **indicaciones no se contradigan** (ejemplo: búsqueda por un producto puntual y uso de facetas).
- Indicar las **acciones entre corchetes,** por ejemplo: [carrusel de productos], [solicitar asistencia].
- Cada X tiempo se va a correr el test de los casos indicados a continuación. Por lo tanto, **los ejemplos se deben actualizar si algún comportamiento se modifica**.
- Debe incluir todas las funcionalidades encendidas, todos los flows nativos y custom y los conocimientos principales asociados a ellos.

| Categoría | Input | Output |
| :-: | :-: | :-: |
| **Asistencia (Fuera de horario)** | Hola! Necesito asistencia | "¡Hola! (emoji no exportado) En este momento, nuestro equipo de Atención al Cliente se encuentra fuera de horario. El horario de atención es de lunes a jueves de 7 a 16hs y viernes de 7 a 13hs. En caso de necesitar asistencia, por favor completa el siguiente formulario https://forms.cloud.microsoft/r/TqdfD7VhAZ para que nuestro equipo esté al tanto y pueda asistirte cuando retome la atención. Muchas gracias."<br>**Nota:** Validar con cliente si quieren que para fuera de horario cree el contacto |
| **Asistencia (Dentro de horario)** | Hola! Necesito asistencia | |
| **Creación de contacto** | Crear contacto | Para crear tu contacto, necesito que me digas tu nombre, número de cédula o NIT, correo electrónico y número de teléfono. ¿Podrías proporcionarme esos datos? |
| **Búsqueda de productos** (producto puntual) | Hola! Estoy buscando las GAFAS THE ONE MASK MIRROR | [Producto]<br>Encontré las GAFAS THE ONE MASK MIRROR para ti. ¿Qué color prefieres? Las opciones disponibles son: f3f1ee, a6a659, 797777, fce903 y negro. ¿Cuál te gustaría?<br>Para "GAFAS THE ONE MASK MIRROR", ¿qué Color Print preferís? Opciones: Color Print: f3f1ee, a6a659, 797777, fce903, 000000.<br>Para "GAFAS THE ONE MASK JR", ¿qué Color Print preferís? Opciones: Color Print: ed0f26_01a7db, 8dd2ea_f3f3f3.<br>Para "THE ONE MASK", ¿qué Color Print preferís? Opciones: Color Print: 5D755F_000000, 686567_FFFFFF, 5f5f5f_323232, 00b8d5_eaeaea, 2d2f34_fafafa, ffffff_08cbfd, 1b1612_f7f7f7.<br>Para "THE ONE MIRROR", ¿qué Color Print preferís? Opciones: Color Print: 009bdb_003f73, 1b1e38_00a3dd, 5D755F_000000, FFFFFF_308CB4, 1b191f_f4f7fa.<br>Para "Gafas de natación The One Plus Mirror", ¿qué Color Print preferís? Opciones: Color Print: 000000_4F4E53, 0062a4_3e3732.<br>Para "AIRSPEED MIRROR", ¿qué Color Print preferís? Opciones: Color Print: 778e8d_1b1920, 7f9389_302e2c, cddeea_0d1318, d8dad9_1055a0, 212121_fdca00, e6e6e4, ffffff_CEB546, C4D023_0000ff.<br>Para "Gafas de natación espejo Zoom X-Fit Mirror", ¿qué Color Print preferís? Opciones: Color Print: 009bdb_003f73, 000000.<br>Para "Gafas de natación Cobra Edge Swipe Mirror", ¿qué Color Print preferís? Opciones: Color Print: 171717_12b2c5, ffffff, 0062a4_3e3732 |
| **Búsqueda de producto** (Coincidencias múltiples) | Quiero Jammer powerskin | Encontré varios modelos de Jammer Powerskin. Por favor, dime qué talle buscas para el producto "Powerskin Primo Jammer Hombre". Las tallas disponibles son: 22, 24, 26, 28, 30, 32 y 34. ¿Cuál prefieres?[Carrusel] |
| **Búsqueda de producto (Categorías)** | Quiero ver productos de competencia natación para hombre | Aquí tienes algunas opciones de productos para competencia de natación para hombre. Te muestro gorros, gafas y maletines ideales para esta actividad. ¿Quieres que te muestre más detalles o algún producto en particular? |
| **Búsqueda de producto (Categorías)** | Que productos para hombre tienen? | Tenemos varios productos para hombre, como bañadores Jammer para entrenamiento y competencia, pantalonetas deportivas y ropa de playa. Por ejemplo, el Jammer Threefold R, el Jammer Arena Diamond, y la pantaloneta Hombre Liso.¿Quieres que te muestre opciones y me digas qué talla y color prefieres para alguno de estos productos? [Carrusel] |
| **Conocimientos (No tiene información)** | Me podrias decir de que están hechos los productos? | No encontré información específica sobre los materiales de los productos en este momento. Si quieres, puedo conectarte con un asesor para que te brinde detalles más precisos. ¿Querés que lo haga? |
| **Conocimientos** | No se cual es mi talle | [Respuesta por Conocimientos, ingresar el contenido de la FAQ]<br>Para elegir tu talle correcto, te recomiendo medir tus medidas corporales directamente y luego consultar la guía de tallas que tenemos en la página de cada producto. Ahí encontrarás una tabla detallada que te ayudará a escoger la mejor opción para ti.¿Querés que te ayude a encontrar la guía de tallas de algún producto en particular? |
| **Conocimientos** | Cuanto puede llegar a tardar mi envio? | [Respuesta por Conocimientos, ingresar el contenido de la FAQ]<br>El tiempo de entrega depende de la ciudad y dirección de destino. Puedes hacer una simulación de compra ingresando tus datos de envío y antes de pagar, el sistema te mostrará el costo y tiempo estimado de entrega. Una vez completes tu pedido y sea despachado, recibirás la información de seguimiento. ¿Querés que te ayude con algo más? |
| **Conocimientos** | Hacen envíos a Argentina? | [Respuesta por Conocimientos, ingresar el contenido de la FAQ]<br>Para compras fuera de Colombia, como Argentina, debes ingresar a la página de Arena correspondiente a tu país. Nosotros realizamos envíos solo dentro de Colombia desde esta tienda.¿Querés que te ayude a encontrar la página de Arena para Argentina? |
| **Promociones** | Me podrias mostrar las promociones disponibles? | No puedo confirmar las promociones actuales en este momento. Si querés, te puedo conectar con un asesor para que te informe sobre las ofertas vigentes. ¿Querés que lo haga? |
| **Sucursales (consulta abierta)** | Donde puedo encontrar sus tiendas físicas? | Puedes encontrar nuestras tiendas físicas en varias ciudades de Colombia. Aquí te dejo algunas ubicaciones destacadas con sus direcciones y teléfonos:<br>- Arena Victoria Centro Comercial: Carrera 11 Bis # 17 - 20, teléfono 6063489251.<br>- Arena Único Bucaramanga: Transversal Central con Calle 103, Local #234, teléfono 6077029531.<br>- Arena Yumbo, Punto de Fábrica Acopi: Carrera 35 # 10 - 707, teléfono (2)4851874.<br>- Arena Villavicencio, Único Centro Comercial - Local #45A: Carrera 22 # 8C - 67, teléfono (8)6718022.<br>- Arena Popayán, Terraplaza Centro Comercial - Local #186: Carrera 9 # 73BN - 22, teléfono (2)8373074.<br>- Arena Pasto, Único Centro Comercial - Local #55: Calle 22 #6 - 61 Urbanización Pucalpa, teléfono (2)7364371.<br>- Arena Palmira, Llanogrande Centro Comercial - Local #133: Calle 31 # 44 - 239, teléfono (2)2874503.<br>- Arena Neiva, Único Centro Comercial - Local #47: Calle 64 # 1D - 140, teléfono (8)8650505.<br>- Arena Montería, Alamedas Centro Comercial - Local #B112: Calle 44 # 8 - 43, teléfono (4)7919170.<br>- Arena Medellín, Centro Comercial Oviedo - Local #2321: CRA 43ª #6SUR-15, teléfono 3629319.<br>- Arena Cúcuta, Jardín Plaza Cúcuta: Centro Comercial Jardín Plaza, Anillo vial oriental #L76, teléfono (7)5893074.<br>- Arena Cali, varias ubicaciones como Piscinas Panamericanas Hernando Botero O'Byrne, Jardín Plaza Centro Comercial, Único Centro Comercial, Palmetto Plaza Centro Comercial, Chipichape Centro Comercial.<br>- Arena Bogotá, varias ubicaciones como Santafé Centro Comercial, Centro Mayor, Complejo Acuático Simón Bolívar, Centro Comercial Plaza Central.<br>- Arena Barranquilla, Alegra Centro Comercial y Único Centro Comercial.<br>Si quieres información más detallada o de alguna ciudad en específico, dime y te ayudo. (emoji no exportado) |
| **Sucursales (proximidad)** | | Para ayudarte a encontrar la sucursal más cercana, ¿podrías decirme en qué ciudad o zona te encuentras? Así te puedo indicar la sucursal Arena más próxima a ti.<br>-Bucamaranga<br>La sucursal más cercana a Bucaramanga es Arena Único Bucaramanga, ubicada en Transversal Central con Calle 103, en el segundo piso, Local #234. El horario es de lunes a viernes de 10:00 a 20:30 y sábados y domingos de 10:00 a 21:00. ¿Quieres que te ayude con algo más? |
| **Consulta por stock sucursal** | Y tienen stock del Jammer pro en esa sucursal? | No tengo acceso directo al stock específico de productos en sucursales físicas. Para confirmar si el Jammer Pro está disponible en Arena Único Bucaramanga, te recomiendo comunicarte directamente con la tienda al teléfono 6077029531. ¿Quieres que te ayude con algo más? |
