> Fuente: Alcance_Funcional_XL_Argentina_Bot_IA.docx (https://drive.google.com/file/d/1hGYckd8fO9V_kWvbAbFFX4WRdgpSZjDj/view?usp=drivesdk)

Snappy Commerce

# Alcance Funcional XL Argentina — Agente IA de Atención al Cliente y Ventas

Versión 1 · 18/11/2026 · Armado a partir de la configuración actual del bot (aiEngineConfig v2) y de la Base de Conocimientos de XL Extra Large (xlshop.com.ar). Tono: español argentino (voseo).

## Cómo usar este documento

Este documento describe la estructura estándar de flujos de un Agente IA de atención al cliente y ventas de Snappy Commerce. Sirve como punto de partida para el relevamiento (discovery) de un cliente nuevo: cada fila define el flujo, sus activadores típicos y el comportamiento esperado en términos generales.

Los textos entre corchetes y las notas marcadas con ⚠ señalan puntos que dependen de la política, integración o contenido propio de cada cliente y deben completarse durante el discovery. Los nombres de los flujos pueden mantenerse o adaptarse a la nomenclatura interna del cliente.

# Requerimientos Funcionales

## Configuración Global

| Flujo nativo | Qué hace el flujo | Configuración por default |
| :-: | :-: | :-: |
| Crear contacto | Solicita los datos de contacto antes de crear un ticket o generar el link de checkout. | Nombre / Apellido / Email: obligatorios (contacto y checkout). |
| Asistencia | No hay atención humana en vivo. Cuando el usuario pide hablar con una persona, el bot genera un ticket (en cualquier horario) y el equipo responde por mail. | Escala: no (sin asistencia en vivo)<br>Ticket: siempre, en y fuera de horario<br>Horario de atención: lunes a viernes de 8 a 17 hs.<br>⚠ Los mensajes de horario están cargados pero desactivados en el orquestador (inHours / outOfHours). Definir si se usan:<br>En horario: «Hola! En breve uno de nuestros asesores se comunicará con vos mediante el mail que nos dejaste.»<br>Fuera de horario: «Hola! En este momento nuestro equipo de Atención al Cliente se encuentra fuera de horario. Nuestro horario es de lunes a viernes entre las 8 y 17 hs. Apenas retomemos la atención, un asesor se contactará con vos mediante el mail que nos dejaste.» |
| Ticket | Cuando se abre un ticket, el bot pide el motivo de la consulta antes de generarlo (si no lo tiene). Puede sumar archivos y un comentario. | Motivo y submotivo obligatorios; archivos y comentario opcionales.<br>Mensaje de confirmación: se generó el caso y se van a contactar por mail dentro de las próximas 24 hs hábiles.<br>⚠ La FAQ de tiempos de respuesta dice hasta 48 hs. Unificar. |
| Encuesta de satisfacción | Al cerrar una conversación, se envía automáticamente una encuesta preguntando cómo fue la atención. | ⚠ A confirmar (no figura en la configuración del bot). |

## Flujos de Pre-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Búsqueda y Recomendación de Producto | Usuario pregunta por un producto, categoría, talle, color o atributo ("tenés zapatillas rojas talle 40?"). | Ejecuta productSearch contra el catálogo de xlshop.com.ar, sumando a la búsqueda las características del producto (color, material, tipo), y devuelve un carousel de resultados. Si no hay resultados o hay demasiada ambigüedad, repregunta.<br>Rubros: carteras, mochilas y billeteras; también calzado, camperas, valijas y accesorios (cinturones, fragancias).<br>Talles y medidas: en la ficha de cada producto. |
| Consulta de Precio / Stock | Usuario pregunta precio, disponibilidad o stock de un producto puntual. | Precio y stock: responde con la información de la ficha del producto vía integración (todos los precios con IVA incluido).<br>Sin stock online: sugiere suscribirse al newsletter para enterarse cuando vuelva y consultar en tiendas físicas.<br>Stock en un local puntual: no lo informa; remite a https://www.xlshop.com.ar/locales para contactar al local.<br>⚠ Confirmar plataforma de e-commerce e integración (el formato de pedido 16XXXXXXX-01 sugiere VTEX). |
| Promociones | Usuario pregunta por descuentos, promos, cupones o "qué ofertas tienen". | Responde con la Base de Conocimientos: promociones no acumulables salvo indicación; promos bancarias por reintegro en el próximo resumen; descuento de primera compra al suscribirse al newsletter; uso de cupones y qué hacer si no funcionan. Promos vigentes en los banners de la home y en https://www.xlshop.com.ar/medios-de-pago.<br>⚠ Confirmar si se cargan promociones activas en Labs para el mensaje nativo. |
| Información de Sucursales / Branches | Usuario pregunta dirección, horarios o sucursal más cercana. | Sucursales deshabilitadas en la configuración (no usa storeSearch). Responde con la FAQ de locales (https://www.xlshop.com.ar/locales) y pide la localidad para orientar.<br>⚠ El flujo Retiro en sucursal deriva las consultas de ubicación y horarios a Sucursales, que está apagado. Definir si se cargan las sucursales en Labs. |
| FAQ Comercial (talles, envíos, medios de pago) | Usuario consulta guía de talles, costos/tiempos de envío, medios de pago o cuotas. | Responde con la Base de Conocimientos (95 FAQs activas):<br>- Envíos: a todo el país salvo Tierra del Fuego; gratis desde $140.000; retiro gratis en sucursales adheridas; express en locales seleccionados; hasta 7 días corridos extra en eventos y liquidación.<br>- Medios de pago: tarjetas de débito y crédito, Mercado Pago, Go Cuotas y MODO (en tiendas físicas, todos).<br>- Facturación: online solo Factura B; Factura A en tiendas físicas.<br>- Otros: cómo comprar, crear cuenta, bolsa de regalo, uso de gift card, sin monto mínimo de compra.<br>⚠ Envío gratis: una FAQ fija $140.000 y otras dicen que depende de la promoción vigente. |
| Checkout Asistido | Usuario quiere comprar / pide el link de compra ("quiero comprar", "pasame el link"). | Solicita nombre, apellido y email y genera el link de checkout de xlshop.com.ar. |
| Checkout Asistido — Gestión del Carrito | Usuario quiere agregar, quitar o modificar productos en un carrito ya iniciado. | ⚠ No definido en la configuración. Confirmar soporte en la integración. |
| Derivación a Vendedor / Asistencia comercial | Usuario pide hablar con un asesor de ventas o la consulta excede el alcance del bot. | No hay asesores en vivo: genera un ticket y avisa que lo contactan por mail.<br>Compras mayoristas y regalos empresariales: informa el mail ventasmayoristas@xl.com.ar (Base de Conocimientos). |

## Flujos de Post-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Estado de Pedido | Usuario pregunta por el estado de su pedido/envío, con o sin número de orden. | ⚠ La configuración no define un flujo de estado de pedido (orderStatus). Hoy responde con la Base de Conocimientos:<br>- El número de seguimiento llega por mail dentro de las 72 hs; también se ve en Mi cuenta > Mis pedidos.<br>- Formato del número de pedido: 16XXXXXXX-01.<br>- Transportistas: Andreani, Correo Argentino y Welivery (express), con sus links de seguimiento.<br>- Entregas parciales: un producto puede salir desde otro local con su propio seguimiento.<br>- Andreani «Pendiente de ingreso» es normal.<br>- Pedido demorado: formulario de contacto (https://www.xlshop.com.ar/contacto) con el número de orden.<br>⚠ Confirmar si se habilita orderStatus vía integración. |
| Factura del pedido | Usuario solicita la factura de una compra. | Responde con la Base de Conocimientos: la factura electrónica llega por mail al confirmarse la compra (revisar spam). Si no llegó, formulario de contacto con el número de pedido. |
| Cambios y Devoluciones | Usuario quiere cambiar o devolver un producto ya comprado. | Hoy responde con la Base de Conocimientos:<br>- Cambios: hasta 30 días corridos desde la recepción, en sucursal con la factura; sin tienda cerca, por formulario de contacto. Productos outlet solo en tiendas outlet.<br>- Devoluciones: hasta 10 días corridos desde la recepción, solo compras online, producto en las mismas condiciones; se inicia por formulario de contacto. Reembolso al mismo medio de pago en 5 a 10 días hábiles.<br>Flujos configurados pero desactivados (exchange y orderReturn): sector postsales; ticket exchanges&returns (orden, motivo, producto pedido a cambio) y ticket devolutions (orden, producto, motivo, estado del producto), con calificación interna del motivo.<br>⚠ Definir si se activan. ⚠ El flujo exchange usa el texto de modifyOrder; corregir. |
| Cancelaciones | Usuario quiere cancelar un pedido. | Hoy responde con la Base de Conocimientos: el pedido se cancela con el Botón de Arrepentimiento (https://www.xlshop.com.ar/boton-de-arrepentimiento) y se hace un pedido nuevo.<br>Flujo configurado pero desactivado (orderCancelation): sector postsales; ticket cancelOrder pidiendo número de orden y motivo, calificado como regret, shipingTime, mistake o paymentMethod.<br>⚠ Unificar canal: unas FAQs mandan al Botón de Arrepentimiento y otras al formulario de contacto. |
| Garantía | Usuario consulta por cobertura o activación de garantía de un producto. | 6 meses desde la fecha de compra. Cubre fallas de fabricación (costuras, herrajes, curtido del cuero, suela). No cubre mal uso, desgaste natural, accidentes, manchas ni exposición al calor. Requiere ticket de compra o número de pedido. No aplica a productos outlet. |
| Reclamos / Ticket | Usuario reporta un problema (producto dañado, faltante, error de facturación, etc.). | Hoy: producto dañado o con fallas y reclamos en general se derivan al formulario de contacto (Base de Conocimientos).<br>Ticket general: motivo y submotivo obligatorios; archivos y comentario opcionales.<br>⚠ Definir si el producto dañado va a ticket (flujo Problema con la compra, hoy desactivado) o se mantiene el formulario. |
| Encuesta de Satisfacción (CSAT) | Se dispara automáticamente al cierre de una gestión o conversación. | ⚠ A confirmar (no figura en la configuración del bot). |
| Derivación a Soporte Post-venta | La consulta post-venta excede el alcance del bot o el usuario pide hablar con un humano. | Genera un ticket (no hay atención en vivo) y avisa que lo contactan por mail dentro de las 24 hs hábiles. Canal oficial: info@xl.com.ar; sin teléfono ni WhatsApp. |

## Flujos de Sistema (Automatizados)

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Mensaje de Bienvenida | Sin activador por mensaje del usuario — se dispara desde automatizaciones al iniciar el primer contacto. | ⚠ No definido en la configuración. Texto y condición de activación a definir por el cliente. |
| Seguimiento Proactivo | Sin activador por mensaje del usuario — automático tras un tiempo configurable de inactividad. | ⚠ No definido en la configuración. Tiempo de inactividad y texto a definir. |
| Cierre por Inactividad + CSAT | Sin activador por mensaje del usuario — automático al quedar la conversación inactiva. | ⚠ No definido en la configuración. Tiempo de inactividad a definir. |
| Notificaciones / Broadcast | Se dispara desde un envío de notificación/broadcast programado en Labs. | Envía el contenido definido en la notificación y puede setear custom contexts o asignar la conversación a un sector/usuario. |

## Flujos con Lógica Condicional (flujos personalizados)

Flujos personalizados configurados en el orquestador del bot. Los desactivados tienen la lógica cargada pero no se disparan; mientras tanto esas consultas se responden con la Base de Conocimientos.

Activos: Compra de gift card y Retiro en sucursal (responden con una FAQ fija). Desactivados: Modificar pedido, Problema con la compra y Compra mayorista.

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Compra de gift card (activo) | Quiere comprar o regalar una gift card, o saber cómo conseguirla. No aplica a saldo o uso (Base de Conocimientos). | Responde con la FAQ de compra: link de OhGiftCard (https://tienda.ohgiftcard.com.ar/brand-profile/105) o compra en sucursales. Sin ticket.<br>⚠ Confirmar que el ID de FAQ configurado (e78636a7…) es esta FAQ. |
| Retiro en sucursal (activo) | Quiere retirar su pedido en un local, saber cómo hacerlo o cuándo está listo. No aplica a ubicaciones u horarios de locales. | Responde con la FAQ de retiro: aviso por mail cuando el pedido está listo; el traslado puede demorar hasta 7 días hábiles; 10 días corridos para retirar, luego se anula; retira solo el titular de la tarjeta con DNI, la tarjeta y el número de pedido.<br>⚠ Confirmar que el ID de FAQ configurado (512b4550…) es esta FAQ. |
| Modificar pedido (desactivado) | Quiere cambiar dirección, cantidad, producto, talle o medio de pago de un pedido antes de recibirlo. | Configurado: sector postsales; ticket modifyOrder con número de orden, tipo de cambio y detalle.<br>Hoy (FAQ): una vez confirmado el pedido no se puede modificar; debe cancelarlo y comprar de nuevo.<br>⚠ El flujo contradice la política: definir si se activa. |
| Problema con la compra (desactivado) | Error al pagar, cupón que no funciona, producto roto, incompleto o defectuoso. | Configurado: sector sales; avisa que es prioridad, deriva a asistencia y crea ticket con el tipo de problema (paymentError, checkoutError, discountCode, shippingCalculation, accountAccess, stockIssue, other).<br>Hoy (FAQ): pasos de autoayuda para pago, carrito y carga de datos; producto con fallas por formulario de contacto. |
| Compra mayorista (desactivado) | Compras al por mayor, revendedores o cotización por volumen. | Sin pasos configurados. Hoy (FAQ): informa el mail ventasmayoristas@xl.com.ar.<br>⚠ Apagar el flujo o cargarle la FAQ de mayoristas. |

⚠ Cualquier comportamiento no cubierto explícitamente en este documento debe confirmarse con el referente técnico del cliente antes de la implementación.

# Diagramas de Flujo (solo tienen que estar las que son personalizadas)

Representación visual de la lógica de decisión de los flujos condicionales listados arriba. Son diagramas base para discovery: los rótulos entre corchetes o marcados con ⚠ en las tablas anteriores deben reflejarse aquí una vez definidos con el cliente.

Versión editable de estos diagramas (formas nativas, sin necesidad de recrearlos): [Diagramas_Flujo_Bot_IA.pptx](https://docs.google.com/presentation/d/1THBgyODVB9IX3UfAmipArhx6RDR1ZC11/edit?slide=id.p1#slide=id.p1), adjunto junto con este documento.

No hay flujos con ramificaciones que requieran diagrama en esta versión: los flujos activos responden con una FAQ fija. ⚠ Si se activan los flujos de posventa, armar sus diagramas.

## Checklist de Configuración por Cliente

Antes de dar por cerrado el alcance funcional de un cliente nuevo, completar y validar:

- Políticas de cambios y devoluciones: cambios 30 días corridos (sucursal con factura o formulario; outlet solo en tiendas outlet); devoluciones 10 días corridos, solo compras online, por formulario. ⚠ Definir si se activan los flujos de cambio, devolución y cancelación.
- Política de garantía: 6 meses por falla de fabricación; no aplica a outlet.
- Transportistas: Andreani, Correo Argentino y Welivery (express). Envíos a todo el país salvo Tierra del Fuego; retiro gratis en sucursales adheridas. ⚠ Unificar la condición de envío gratis.
- Medios de pago: tarjetas de débito y crédito, Mercado Pago, Go Cuotas y MODO. ⚠ Revisar la FAQ de pago en efectivo (no figura como medio de pago) y la de cuotas (desactivada).
- Plataforma: ⚠ a confirmar. Checkout con nombre, apellido y email.
- ticketReasons configurados: cancelOrder, exchanges&returns, modifyOrder y devolutions (sector postsales); Problema con la compra en sector sales. ⚠ Definir motivo y submotivo del ticket general.
- Base de Conocimientos: 99 FAQs (95 activas). ⚠ Unificar duplicadas, corregir el link de Instagram (lhttps://), revisar la FAQ de WhatsApp («escribirnos por este chat») y la de Cyber Monday.
- Mensajes fijos: ⚠ bienvenida, seguimiento y cierre sin definir; mensajes de horario cargados pero desactivados.
- Referente del cliente: ⚠ a completar.
