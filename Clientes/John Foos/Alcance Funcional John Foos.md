> Fuente: Analisis_Funcional_JohnFoos_Gen6_v1.docx (https://drive.google.com/file/d/1aXGaHGw8e5dKv-9QdF81FmiFfyPSyuq7/view?usp=drivesdk)

**Snappy Commerce**

# Análisis Funcional — John Foos, Agente IA

*John Foos Bot · Relevamiento del comportamiento vigente en producción (Gen 5) como base para la migración a Gen 6*

# Cómo usar este documento

Este documento releva el comportamiento actual del Agente IA de John Foos — flujos nativos, flujos personalizados y configuración global — siguiendo la estructura del alcance funcional estándar de Snappy Commerce. Cada fila define el flujo, sus activadores y el comportamiento esperado.

Sirve para que Producto valide el comportamiento vigente, detecte vacíos funcionales e inconsistencias de configuración, y tenga documentado cada flujo antes de que pase de la UI de Snappy (Gen 5) a código TypeScript mantenido por Producto (Gen 6). La experiencia del usuario final no cambia con la migración.

Las notas marcadas con ⚠ señalan flujos desactivados o puntos a confirmar. Los hallazgos y riesgos abiertos se listan al final del documento.

# Requerimientos Funcionales

## Configuración Global

| Flujo nativo | Qué hace el flujo | Valor configurado |
| :-: | :-: | :-: |
| Crear contacto — Campos | Solicita los datos indicados al usuario para crear un nuevo contacto. | firstName, email, phone |
| Comenzar checkout — Requisitos | Datos de contacto requeridos para iniciar un checkout. | sin campos obligatorios |
| Asistencia | Cuando un flujo dispara "assistance", solicita datos de contacto. En horario de atención escala en vivo; fuera de horario genera un ticket. onlineAssistance: false | contact: always<br>escalate: in_hours<br>ticket: out_of_hours |
| Encuesta de satisfacción | Activa una encuesta de satisfacción al finalizar una conversación. | DESACTIVADO |
| Encuesta al cierre (endConversationSurvey) | Activa una encuesta automática al cierre de conversación. | DESACTIVADO |
| Búsqueda de Producto (productSearch) | Permite buscar y recomendar productos del catálogo. | ACTIVADO |
| Estado de Pedido (orderStatus) | Consulta el estado de un pedido a partir de los datos del cliente. | ACTIVADO |
| Sucursales (branches) | Provee información sobre sucursales físicas. | DESACTIVADO |
| Stock en locales (productBranches) | Consulta disponibilidad de stock en sucursal física. | DESACTIVADO |
| Factura (orderInvoice) | Permite solicitar o consultar la factura electrónica de una compra. | DESACTIVADO |
| Fin de conversación (endConversation) | Permite cerrar una conversación de forma automática. | DESACTIVADO |
| Promociones nativas | Informa sobre promociones y descuentos vigentes. | DESACTIVADO |
| Opciones múltiples (multipleprompt) | Muestra opciones al usuario en forma de botones interactivos. | DESACTIVADO |
| Canal MercadoLibre | Canal de mensajería a través de MercadoLibre. | DESACTIVADO |

## Flujos de Preventa

| Flujo | Activadores | Comportamiento esperado |
| :-: | :-: | :-: |
| purchaseIssue | ✅ Usuario tiene PROBLEMAS PARA COMPRAR O PAGAR: errores en el checkout, pago rechazado, falla técnica. "no puedo pagar", "me da error en el checkout", "me rechazó la tarjeta".<br>❌ Cancelar pedido → orderCancelation · Devolución → orderReturn · Métodos de pago → FAQ | sector → sales<br>assistance (ticketReason: buyingIssues)<br>Campos adicionales:<br>- Tipo de error (Error de tarjeta / Error técnico / Otro)<br>- Descripción del problema |

## Flujos de Postventa

| Flujo | Activadores | Comportamiento esperado |
| :-: | :-: | :-: |
| defectiveProduct | ✅ Usuario reporta un producto ROTO, DAÑADO, FALLADO o DEFECTUOSO. "me llegó roto", "vino con un defecto", "no funciona", "se rompió apenas lo usé".<br>❌ No me gustó → orderReturn · Quiero otro talle/color → Cambio | sector → postsales<br>notify: "¡Lamento mucho que hayas recibido un producto defectuoso! ð" (isExactMessage)<br>ticket (reason: PostVenta)<br>Campos:<br>- Número de orden (consultingData)<br>- Descripción del defecto<br>- Imagen del defecto (file)<br>- Solución deseada (cambio / reembolso / reparación) |
| orderCancelation | ✅ Usuario quiere CANCELAR un pedido u orden ANTES de recibirlo. "quiero cancelar mi pedido", "cancelá mi compra", "ya no quiero el producto", "me arrepentí".<br>❌ Devolver algo ya recibido → orderReturn · Cambio de lo ya recibido → Cambio | sector → postsales<br>ticket (reason: cancelOrder)<br>Campos:<br>- Número de orden (consultingData)<br>- Motivo de cancelación (opcional — sin guardar estado del pedido) |
| Reclamos | ✅ Usuario quiere HACER UN RECLAMO FORMAL o hablar con un supervisor humano. "quiero hacer un reclamo", "necesito hablar con un supervisor", "voy a ir a defensa del consumidor".<br>❌ Me llegó roto → defectiveProduct · Devolución → orderReturn · Cambio de talle → Cambio | sector → support<br>notify: "Entiendo tu frustración y tus preocupaciones son muy importantes para nosotros. ð" (isExactMessage)<br>assistance (priority: high, ticketReason: PostVenta) |

## Flujos de Sistema (Automatizados)

| Flujo | Activadores | Comportamiento esperado |
| :-: | :-: | :-: |
| Mensaje de bienvenida | Sin activador por mensaje del usuario — se dispara desde automatizaciones al iniciar el primer contacto. | DESACTIVADO |
| Seguimiento proactivo | Sin activador por mensaje del usuario — automático tras un tiempo configurable de inactividad. | DESACTIVADO |
| Cierre por inactividad + CSAT | Sin activador por mensaje del usuario — automático al quedar la conversación inactiva. | DESACTIVADO |
| Notificaciones / Broadcast | Se dispara desde un envío de notificación/broadcast programado en Labs. | DESACTIVADO |

# Requerimientos No Funcionales

| Categoría | Requerimiento |
| :-: | :-: |
| Lenguaje | Español rioplatense, tono profesional pero cercano, ameno y cordial. Voseo en todos los verbos (querés, podés, decime, tenés). Sin español neutro, ni trato de usted. |
| Rol del agente | Preventa y postventa (role: both). |
| Testing | El código/JSON debe pasar el test suite estándar de Snappy (umbral 85%+) antes de cualquier subida a producción. |
| Conversaciones | El agente debe contar con contexto de las conversaciones previas de cada contacto y poder responder usando esa información. |
| Comportamiento — Fallback | Cuando no cuente con información suficiente en Conocimientos, debe ofrecer asistencia de un operador humano. |
| Comportamiento — Opciones múltiples | El bot no presenta botones de opciones al usuario (showOptions: false). |
| Canal desactivado | MercadoLibre está desactivado como canal de entrada. |

## Flujos con Lógica Condicional

Sin flujos con lógica condicional. Todos los flujos configurados son lineales.

# Diagramas de Flujo

Sin flujos con lógica condicional. No aplica.

# Test

| Categoría | Input | Output |
| :-: | :-: | :-: |
| Envío gratis | ¿Tienen envío gratis? | Contamos con envío gratis en compras superiores a $130.000. |
| Envío gratis | ¿Cuánto cuesta el envío? | El costo depende de la ubicación de entrega. Si tu compra supera los $130.000, el envío es sin cargo. |
| Promociones | ¿Tienen alguna promo? | ¡Sí! Podés ver todas nuestras promociones en: www.johnfoos.com/promociones |
| Promociones — Hot Days | ¿Qué promos tienen en Hot Days? | En Hot Days tenemos hasta 40% de descuento. Cuotas: 6 sin interés con Visa/Mastercard/Amex, 5 con Naranja X, 4 con Go Cuotas. Envío gratis en compras mayores a $130.000. |
| Cambio | Quiero cambiar el talle | Aceptamos cambios hasta 30 días corridos de recibido el producto, en perfectas condiciones y con empaque original. Más info: www.johnfoos.com/cambios-y-devoluciones |
| Cambio | ¿Cómo hago un cambio? | Aceptamos cambios hasta 30 días corridos de recibido el producto, en perfectas condiciones y con empaque original. Más info: www.johnfoos.com/cambios-y-devoluciones |
| Devolución | Quiero devolver el producto | Aceptamos devoluciones dentro de los 10 días de recibida la compra. Para gestionarla: www.johnfoos.com/cambios-y-devoluciones |
| Devolución | ¿Cómo hago una devolución? | Aceptamos devoluciones dentro de los 10 días de recibida la compra. Para gestionarla: www.johnfoos.com/cambios-y-devoluciones |
| Problemas para comprar | No puedo completar el pago | [sector sales] → [solicitar asistencia — ticketReason: buyingIssues] → [solicitar tipo de error y descripción del problema] |
| Problemas para comprar | Me rechazó la tarjeta al pagar | [sector sales] → [solicitar asistencia — ticketReason: buyingIssues] → [solicitar tipo de error y descripción del problema] |
| Producto defectuoso | Me llegó un producto roto | "¡Lamento mucho que hayas recibido un producto defectuoso! ð" → [solicitar número de orden, descripción del defecto, imagen, solución deseada] |
| Producto defectuoso | Llegó dañado | "¡Lamento mucho que hayas recibido un producto defectuoso! ð" → [solicitar número de orden, descripción del defecto, imagen, solución deseada] |
| Cancelación | Quiero cancelar mi pedido | [sector postsales] → [solicitar número de orden] → [solicitar motivo de cancelación — opcional] |
| Cancelación | Me arrepentí, cancelá la orden | [sector postsales] → [solicitar número de orden] → [solicitar motivo de cancelación — opcional] |
| Reclamos | Quiero hacer un reclamo | "Entiendo tu frustración y tus preocupaciones son muy importantes para nosotros. ð" → [solicitar asistencia — priority: high] |
| Reclamos | Quiero hablar con un supervisor | "Entiendo tu frustración y tus preocupaciones son muy importantes para nosotros. ð" → [solicitar asistencia — priority: high] |
| Pagos | ¿Qué medios de pago aceptan? | Aceptamos tarjetas de débito y crédito vía MercadoPago. También podés pagar en efectivo en RapiPago o Pago Fácil. |
| Pagos | ¿Tienen cuotas sin interés? | 3 y 6 cuotas sin interés con Visa, Mastercard y Amex (Plan Ahora). 5 cuotas con Naranja X. 4 cuotas con débito vía Go Cuotas. |
| Pagos | ¿Se puede pagar por transferencia? | No aceptamos transferencias bancarias. Podés pagar con tarjeta, MercadoPago, Go Cuotas, RapiPago o Pago Fácil. |
| Envíos | ¿Cuánto tarda el envío? | CABA hasta 3 días hábiles, GBA hasta 4, Interior hasta 6 y Patagonia hasta 10. |
| Envíos | ¿Hacen envíos internacionales? | Por el momento no realizamos envíos internacionales. |
| Envíos | ¿A través de qué correo envían? | Despachamos todas las compras a través de Correo Andreani. |
| Talles | ¿Cómo elijo mi talle? | En cada producto encontrás su tabla de referencia de talles. Video tutorial: youtube.com/watch?v=1gazArlLA0Q |
| Stock | ¿Qué hago si un producto está agotado? | Podés dejar tu mail en la página del producto y te avisamos cuando ese talle vuelva a tener stock. |
| Garantía | ¿Cómo activo la garantía de mis zapatillas? | Escaneá el código QR en la plantilla de tus zapatillas con la cámara de tu celular, completá los datos y listo. |
| Sucursales | ¿Tienen locales físicos? | No contamos con locales oficiales. Podés comprar en nuestro Shop Online y retirar en cualquier sucursal de Correo Andreani. |
| Factura | ¿Cómo solicito mi factura? | Una vez confirmada la compra te llega un mail con la factura electrónica adjunta. Si no la recibiste, escribinos a shoponline@johnfoos.com. |
| Mayorista | ¿Cómo hago una compra mayorista? | Completá este formulario: forms.gle/4FouzCTey5RRRX1P6 |
| Descuentos | ¿Cómo obtengo el descuento de primera compra? | Al suscribirte al newsletter recibís un código de descuento exclusivo para usar en tu |

*(Nota: el contenido devuelto por Drive termina truncado en esta última fila; es el final del texto disponible. Los "ð" son emojis que el origen devolvió corruptos.)*
