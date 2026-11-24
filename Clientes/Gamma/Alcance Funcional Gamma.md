> Fuente: Alcance Funcional Gamma Bot IA v1 (2026-11-13) (https://docs.google.com/document/d/1mb2sblZCxdwgwRQDvH4ROt0tu5ehZBbIWe10ZJNbuak/edit?usp=drivesdk&ouid=113568244751416058438)

**Snappy Commerce**

**Alcance Funcional — Agente IA de Atención al Cliente y Ventas**

Cliente: Gamma (Grupo Simpa) · Relevamiento a partir del alcance genérico

*Estado de los flujos. Activo: se configura como en el estándar. Activo con adaptación: el flujo sigue el estándar con cambios por la política de Gamma. Deshabilitado: no se configura, y su respuesta sale de la Base de Conocimientos. ⚠ indica un punto que hay que confirmar antes de implementar.*

# 1. Requerimientos Funcionales

## 1.1 Configuración Global

| **Flujo nativo** | **Qué hace el flujo** | **Configuración para Gamma** |
| :-: | :-: | :-: |
| **Crear contacto** | Si el canal ya aporta un dato (teléfono en WhatsApp, ID de Instagram/Facebook) o el usuario da un email, crea el contacto sin pedir más. Solo pide datos si no hay ninguno. | Teléfono / email / WhatsApp ID / Facebook ID: alcanza con uno solo |
| **Asistencia** | Cuando el usuario pide hablar con una persona, dentro de horario deriva directo, sin pedir datos. Fuera de horario genera un ticket de soporte y pide los datos de contacto. | Escala: en horario. Ticket: fuera de horario. Horario de referencia: lunes a viernes de 9 a 18 hs (canal ecommerce) ⚠ confirmar en la plataforma |
| **Ticket** | Antes de generar un ticket, pide el motivo de la consulta si no lo tiene. | Motivo obligatorio |
| **Encuesta de satisfacción** | Al cerrar una conversación envía la encuesta de atención. | true |

## 1.2 Flujos de Pre-Venta

| **Flujo** | **Activadores** | **Necesidad / Comportamiento esperado** | **Estado** |
| :-: | :-: | :-: | :-: |
| **Búsqueda y Recomendación de Producto** | Usuario pregunta por un producto, categoría, talle, color o atributo. | Ejecuta productSearch contra el catálogo y devuelve un carousel de resultados. Si no hay resultados o hay demasiada ambigüedad, repregunta o deriva a asistencia. Diagrama 1. | Activo (sin cambios, según indicación) |
| **Consulta de Precio / Stock** | Usuario pregunta por precio, disponibilidad o stock de un producto puntual. | Responde con la información de la integración del cliente. ⚠ Confirmar si Gamma tiene integración de precio y stock. Las FAQs indican que precio y stock se ven en la web: si no hay integración, responde por Conocimientos con el link de la tienda. | Activo, sujeto a integración ⚠ |
| **Promociones** | Usuario pregunta por descuentos, promociones, cupones u ofertas. | Flujo nativo deshabilitado. La respuesta sale de Conocimientos: promociones bancarias (3 cuotas sin interés con Visa, Master y Amex; BBVA 12 cuotas; Galicia 12 cuotas; ICBC 9 cuotas), cupones no acumulables y descuento del newsletter en la primera compra. | Deshabilitado (responde por Conocimientos) |
| **Información de Sucursales / Branches** | Usuario pregunta por dirección, horarios o sucursal más cercana. | Flujo deshabilitado: Gamma no tiene sucursales. Respuesta por Conocimientos: no hay sucursales; distribuidores y ferreterías asociadas con link para ubicar el más cercano; Pick Up Garín y Palermo solo para retiro de pedidos, con horarios. | Deshabilitado (responde por Conocimientos) |
| **Stock por sucursal** | Usuario pregunta si hay stock en una sucursal o local. | Flujo deshabilitado. Respuesta por Conocimientos: el stock disponible es el de la web. Si un producto no está online (por ejemplo, estufa garrafera), deriva a distribuidores oficiales con link. | Deshabilitado (responde por Conocimientos) |
| **FAQ Comercial (envíos, medios de pago, compra)** | Consulta sobre costo y plazo de envío, medios de pago, cómo comprar, compra mínima o ventas internacionales. | Responde desde la Base de Conocimientos. Envío a todo el país; costo y plazo se calculan con el código postal en la ficha del producto o en el checkout; no hay envío gratis a domicilio ni compra mínima; solo se vende dentro de Argentina. Guía de talles: no aplica, el catálogo no tiene talles. | Activo |
| **Productos de otra marca (GA.MA)** | Usuario pregunta por planchitas de pelo, secadores o máquinas de cortar pelo. | Aclara que esos productos son de la marca GA.MA, que tiene un nombre parecido, y no de Gamma. Para reclamos o consultas indica contactar a GA.MA. Ofrece el link de la tienda Gamma. | Activo (por Conocimientos) |
| **Checkout Asistido** | Usuario quiere comprar o pide el link de compra ("quiero comprar", "pasame el link"). | Genera el link de checkout de la plataforma del cliente. No solicita datos de contacto (firstName ni email). | Activo, sin solicitud de datos |
| **Checkout Asistido — Gestión del Carrito** | Usuario quiere agregar, quitar o modificar productos en un carrito ya iniciado. | Actualiza el carrito vía integración según la acción pedida, validando el stock de la combinación elegida. | Activo |
| **Venta mayorista / reventa** | Usuario pregunta por compra mayorista, reventa o "comprar x mayor". | Pide la zona de la consulta y pasa el contacto del vendedor correspondiente. No genera ticket. ⚠ Falta la tabla zona → vendedor (pendiente del cliente). | Activo ⚠ pendiente tabla |
| **Derivación a Vendedor / Asistencia comercial** | Usuario pide hablar con un asesor de ventas o la consulta excede el alcance del bot. | Dispara el flujo de Asistencia: deriva en horario; fuera de horario genera ticket de soporte. | Activo |

## 1.3 Flujos de Post-Venta

| **Flujo** | **Activadores** | **Necesidad / Comportamiento esperado** | **Estado** |
| :-: | :-: | :-: | :-: |
| **Estado de Pedido** | Usuario pregunta por el estado de su pedido o envío, con o sin número de orden. | Envía estado, fecha estimada y tracking vía orderStatus si la integración lo permite. Si no, indica que las actualizaciones llegan por mail al correo registrado en la compra, y que el seguimiento está en Mi cuenta \> Mis pedidos. El número de seguimiento llega por mail dentro de las 24 hs de la confirmación. | Activo ⚠ depende de integración |
| **Envío retrasado, pedido incompleto o no entregado** | Usuario reporta que el envío se demoró, que faltan productos o que no llegó el pedido. | Envío retrasado: si ya se venció el plazo de despacho informado en la compra, pide número de pedido y documento y deriva. Pedido incompleto: pide número de pedido, documento y productos faltantes, y deriva. Nadie para recibir: se hace un segundo intento de entrega que el repartidor coordina con el cliente. | Activo (adaptado, condicional) |
| **Factura del pedido** | Usuario pide la factura de una compra. | Factura electrónica: se solicita desde la web o por mail a tienda@gammaherramientas.com.ar indicando número de pedido. Depende de que la integración tenga esta función ⚠. | Activo |
| **Factura A** | Usuario pide factura A o factura a nombre de una empresa. | Pide enviar por mail la constancia de inscripción en AFIP y el certificado de Ingresos Brutos. Los cambios de factura solo se hacen dentro del mismo mes de la compra. Las compras con promociones bancarias son solo consumidor final: no se emite Factura A en esos casos. | Activo (condicional) |
| **Cambios y Devoluciones** | Usuario quiere cambiar o devolver un producto ya comprado. | Informa la política. ⚠ Plazo en conflicto: 10 o 30 días corridos desde la recepción. El cambio es solo por el mismo producto, por rotura, falla de fabricación o detalle estético; no se cambia por un producto distinto. El producto debe volver con etiqueta y lacre intactos, en packaging original. La solicitud se hace por mail a tienda@gammaherramientas.com.ar con producto y motivo. Las actualizaciones llegan por email. | Activo ⚠ plazo |
| **Cancelaciones** | Usuario quiere cancelar un pedido. | Adaptado. Pide el número de pedido y deriva a un asesor para validar y procesar la devolución del dinero (escala en horario; ticket fuera de horario). Si el pedido ya fue despachado, el trámite se hace por mail a tienda@gammaherramientas.com.ar. No se ofrece la opción de no derivar del estándar: la política de Gamma siempre valida y devuelve. | Activo (adaptado) |
| **Modificar pedido o dirección** | Usuario quiere cambiar productos, cantidades o la dirección de un pedido. | Los pedidos no se modifican: hay que anular la compra y volver a comprar con los datos correctos. Para cambiar la dirección de envío, deriva a un asesor. Se puede cambiar según el estado del pedido ⚠ (regla por definir). | Activo (adaptado) |
| **Reintegros** | Usuario pide que le devuelvan el dinero o un reintegro. | No hay reintegros, salvo por compras canceladas o devoluciones de compras hechas en el sitio oficial. | Activo |
| **Garantía** | Usuario consulta por cobertura o activación de garantía de un producto. | Gamma, como importador, da 24 meses de garantía por defectos de fabricación. Hay que conservar la factura de compra. Para hacerla efectiva, deriva al servicio técnico oficial. ⚠ Definir si la garantía genera ticket (supuesto: no). | Activo ⚠ ticket |
| **Servicio técnico, repuestos y puesta en marcha** | Equipo que no funciona, busca repuestos, servicio técnico cercano o primera puesta en marcha. | Servicio técnico oficial: link para ubicar el más cercano; con factura para validar garantía. Repuestos: solo por WhatsApp al 11 3935 6407, lunes a viernes de 9 a 16 hs. Puesta en marcha gratuita en equipos a explosión (motosierra, grupo electrógeno, desmalezadora) con factura, con link para elegir provincia. | Activo (fila agregada, no está en el estándar) |
| **Reclamos / Ticket** | Usuario reporta un problema: producto dañado, faltante, error de cobro o reclamo. | Genera un ticket con el ticketReason y ticketSubreason correspondientes, pidiendo número de pedido y factura. Motivos: cobro incorrecto, pedido incompleto, envío retrasado, modificación de dirección, reclamo y otro problema. | Activo |
| **Encuesta de Satisfacción (CSAT)** | Se dispara al cierre de una gestión o conversación. | Envía la encuesta de satisfacción configurada de forma global. | Activo |
| **Derivación a Soporte Post-venta** | La consulta post-venta excede el alcance del bot o el usuario pide hablar con una persona. | Dispara el flujo de Asistencia: escala en vivo en horario; genera ticket fuera de horario. | Activo |

## 1.4 Flujos de Sistema (Automatizados)

| **Flujo** | **Activadores** | **Necesidad / Comportamiento esperado** | **Estado** |
| :-: | :-: | :-: | :-: |
| **Mensaje de Bienvenida** | Sin mensaje del usuario: se dispara desde automatizaciones al iniciar el primer contacto. | Mensaje fijo definido por el cliente. ⚠ Contenido, tono y condición de activación pendientes. | Pendiente de contenido |
| **Seguimiento Proactivo** | Sin mensaje del usuario: automático tras un tiempo de inactividad. | Envía un mensaje preguntando cómo le fue con la gestión. ⚠ Tiempo de inactividad pendiente. | Pendiente de definir |
| **Cierre por Inactividad + CSAT** | Sin mensaje del usuario: automático al quedar la conversación inactiva. | Envía mensaje de cierre y dispara la encuesta de satisfacción. ⚠ Tiempo de inactividad pendiente. | Pendiente de definir |
| **Notificaciones / Broadcast** | Envío de notificación o broadcast programado en Labs. | Envía el contenido definido en la notificación. Uso para Gamma pendiente de definir. | A definir |

# 2. Flujos con Lógica Condicional (flujos personalizados)

Estos flujos tienen ramificaciones propias de la política de Gamma. Los puntos de decisión son:

- Cambios y devoluciones: ¿el pedido está dentro del plazo? ¿El producto vuelve con etiqueta y lacre? ¿El motivo es rotura, falla o detalle estético? Si no cumple, se rechaza o se deriva a asesor por caso particular.
- Factura A: ¿la compra fue con promoción bancaria? Si sí, no se emite. ¿Está dentro del mismo mes? Si no, se pide validación. Si sí, pide la documentación por mail.
- Cancelación: ¿el pedido fue despachado? No: deriva a asesor para validar y devolver. Sí: trámite por mail.
- Envío retrasado: ¿venció el plazo de despacho informado? Sí: pide número de pedido y documento y deriva. No: informa el seguimiento.
- Garantía: ¿tiene la factura de compra? Sin factura no se activa la garantía, según la política de Gamma. Con factura, deriva al servicio técnico oficial.

⚠ Los diagramas de estos flujos se construyen en el Diagramas\_Flujo\_Bot\_IA una vez confirmadas las decisiones de política.

# 3. Diagramas de Flujo

Pendiente. Se deben construir diagramas para los flujos condicionales de la sección 2 (Cambios y devoluciones, Factura A, Cancelaciones, Envío retrasado y Garantía), con las decisiones que queden confirmadas.

# 4. Registro de actualizaciones

| **Fecha** | **Versión** | **Qué cambió** | **Motivo** |
| :-: | :-: | :-: | :-: |
| **13/11/2026** | v1 | Primer relevamiento de Gamma sobre el alcance genérico. Búsqueda de productos sin cambios. Promociones y sucursales deshabilitadas con respuesta por Conocimientos. Asistencia activa (escala en horario, ticket fuera de horario). Checkout sin solicitud de datos. Post-venta adaptado a la política de Gamma | Indicaciones del equipo; falta revisión de Peter |
