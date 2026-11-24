> Fuente: Alcance Funcional Yagmour - Bot IA (https://docs.google.com/document/d/1JLW0B7UTdh2l4FGiIXx6fECvxB2wcuEjKPhesJOs7-Y/edit?usp=drivesdk&ouid=113568244751416058438)

**Snappy Commerce**

**Alcance Funcional — Agente IA de Atención al Cliente y Ventas · Yagmour**

*Documento de alcance del cliente Yagmour · Argentina · Basado en la configuración actual del bot IA (v2)*

## Cómo usar este documento

Este documento describe la estructura de flujos del Agente IA de atención al cliente y ventas de Yagmour en Snappy Commerce. Cada fila define el flujo, sus activadores y el comportamiento configurado actualmente en el bot.

Los textos entre corchetes y las notas marcadas con ⚠ señalan puntos que dependen del contenido cargado en Labs (FAQs, menús, base de conocimientos) o que deben validarse con el referente de Yagmour. Los nombres de los flujos respetan la nomenclatura interna del bot.

# Requerimientos Funcionales

## Configuración Global

| **Flujo nativo** | **Qué hace el flujo** | **Configuración por default** |
| :-: | :-: | :-: |
| Crear contacto | Si el canal ya aporta un dato de contacto (teléfono en WhatsApp, ID de Instagram/Facebook) o el usuario da un email, se crea el contacto directamente sin pedir más — solo se solicita si no hay ningún dato disponible. | Teléfono / email — alcanza con uno solo<br>Sin campos adicionales |
| Asistencia | Cuando el usuario pide hablar con una persona, en horario de atención el bot lo deriva directo, sin pedir datos. Fuera de horario, en cambio, genera un ticket de soporte y ahí sí pide los datos de contacto. | Escala: En horario<br>Ticket: Fuera de horario |
| Ticket | Cuando se abre un ticket, el bot pide el motivo y el submotivo de la consulta antes de generarlo (si no los tiene). Opcionalmente puede adjuntar archivos y un comentario. | Obligatorio: ticketReason + ticketSubreason<br>Opcional: archivos, comentario |
| Encuesta de satisfacción | Al cerrar una conversación, se envía automáticamente una encuesta preguntando cómo fue la atención. | true |
| Interacciones en Redes Sociales | El bot también responde las interacciones que llegan por redes sociales (Instagram / Facebook), con los mismos flujos. | true |

## Flujos de Pre-Venta

| **Flujo** | **Activadores** | **Necesidad / Comportamiento esperado** |
| :-: | :-: | :-: |
| BuscadorDeProductos (Búsqueda y Recomendación) | Usuario quiere ver productos o pide el catálogo ("quiero ver camperas de mujer", "¿qué remeras tienen?", "\[emoji no recuperable en la fuente\] Comprar por chat"). | Ejecuta productDiscovery contra el catálogo de Yagmour (incluye características del producto en la búsqueda) y devuelve los resultados. ⚠ El activador del orquestador está deshabilitado actualmente: confirmar si debe habilitarse. |
| Promociones | Usuario pregunta por promociones, descuentos, ofertas o códigos vigentes (no CYBER). | Envía el menú de Promociones configurado en Labs. \[Contenido del menú a mantener actualizado por Yagmour\]. |
| CYBER | Usuario menciona explícitamente la palabra CYBER y consulta por el evento (fechas, mecánica, qué incluye). | Responde con las FAQs del evento CYBER (2 respuestas encadenadas). \[Contenido a actualizar en cada edición del evento\]. |
| Descuento de bienvenida | Usuario se registró en la web y no recibió el descuento / cupón de bienvenida. | Responde con la FAQ de Descuento de bienvenida indicando cómo obtenerlo o qué hacer si no llegó. |
| GIFTCARD | Usuario consulta por la compra o el uso de una gift card / tarjeta de regalo (incluye vencimiento). | Responde con 3 FAQs encadenadas sobre la Gift Card (compra, uso y condiciones). |
| GuíaDeTalles | Usuario consulta por la guía de talles, medidas de una prenda o qué talle le corresponde. | Envía el menú de Guía de Talles configurado en Labs. |
| InformacionDeEnvios | Usuario consulta costos de envío, plazos de entrega, correo utilizado o envío rápido a su zona. | Consulta la Base de Conocimientos (costo, tiempo de envío y empresa de correo) y complementa con 2 FAQs de envíos. |
| EnvioGratis | Usuario consulta si hay envío gratis o el monto mínimo para acceder a él. | Responde con la FAQ de Envío Gratis. \[Monto mínimo a mantener actualizado\]. |
| Stockensucursales | Usuario pregunta si un producto específico está disponible en una sucursal puntual o si se puede reservar. | Responde con la FAQ de Stock en sucursales (cómo consultar disponibilidad en el local). |
| Ventamayorista | Usuario consulta por compras mayoristas, reventa o franquicias de la marca. | Responde con la FAQ de Venta mayorista / franquicias. |
| AsistenciaProcesoCompra | Usuario indica que no puede comprar sin especificar un error concreto ("no me deja avanzar", "no puedo comprar"). | Envía un menú de diagnóstico para identificar en qué paso se traba (ver producto, carrito, pago, dirección) y guiarlo. |
| ErrorTecnicoEnCompra | Usuario identifica un error técnico puntual: pago rechazado, bug del sitio, falta de stock o error al finalizar la compra. | Responde con la FAQ de errores en la compra y luego abre un flujo de asistencia/ticket solicitando descripción del error y captura de pantalla. |

## Flujos de Post-Venta

| **Flujo** | **Activadores** | **Necesidad / Comportamiento esperado** |
| :-: | :-: | :-: |
| SeguimientoDeCompra | Usuario consulta dónde está su pedido, cuándo llega, quiere rastrear el envío o retirar en sucursal. | Envía el menú de Seguimiento de Compra configurado en Labs (opciones según envío a domicilio o retiro en sucursal). |
| FacturadeCompra | Usuario solicita la factura o comprobante de su compra (incluye factura A / empresa). | Envía el menú de Factura de Compra configurado en Labs. |
| PoliticaCambio | Usuario pregunta por condiciones, plazos o requisitos para cambiar un producto (sin iniciar el trámite). | Responde con 2 FAQs encadenadas sobre la política de cambios. |
| RealizarCambio | Usuario quiere iniciar un cambio de un producto comprado en la tienda propia (talle, color, otro producto). | Responde con la FAQ que explica el paso a paso para realizar el cambio. |
| CambioMercadoLibre | Usuario quiere cambiar un producto comprado en Mercado Libre. | Responde con la FAQ específica de cambios para compras de Mercado Libre. |
| CambioCyber | Usuario quiere cambiar un producto comprado durante CYBER days. | Responde con la FAQ específica de cambios para compras realizadas en CYBER. |
| PoliticaDevoluciones | Usuario pregunta por condiciones o plazos para devolver un producto y recibir el reintegro. | Responde con la FAQ de política de devoluciones. |
| RealizarDevolucion | Usuario quiere iniciar una devolución y obtener el reintegro del dinero. | Responde con la FAQ que explica cómo realizar la devolución. |
| EstadoCambioDevolucion | Usuario ya inició un cambio o devolución y consulta el estado o avance del trámite. | Responde con la FAQ de estado de cambios y devoluciones (plazos y canal de consulta). |
| Encuesta de Satisfacción (CSAT) | Se dispara automáticamente al cierre de una gestión o conversación. | Envía la encuesta de satisfacción configurada de forma global. |
| Derivación a Soporte | La consulta excede el alcance del bot o el usuario pide hablar con un humano. | Dispara el flujo de Asistencia: escala en vivo o genera ticket según horario de atención. |

## Flujos de Sistema (Automatizados)

| **Flujo** | **Activadores** | **Necesidad / Comportamiento esperado** |
| :-: | :-: | :-: |
| Seguimiento1 (Seguimiento Proactivo) | Sin activador por mensaje del usuario — se dispara de forma automática. | Envía un mensaje de seguimiento generado por IA en base al contexto de la conversación. \[Tiempo de inactividad a confirmar\]. |
| Cierre por Inactividad + CSAT | Sin activador por mensaje del usuario — automático al quedar la conversación inactiva. | Envía mensaje de cierre y dispara la Encuesta de Satisfacción (CSAT). |

## Flujos con Lógica Condicional (flujos personalizados)

Los siguientes flujos tienen ramificaciones internas y requieren un diagrama de decisión propio, construido durante el discovery de cada cliente. Se listan aquí los puntos de ramificación genéricos que ese diagrama debe cubrir:

- ErrorTecnicoEnCompra: FAQ inicial → ticket de asistencia. Campo "Descripción del error" clasificado en: Método de pago / Falta de stock / Bug en la tienda / Cualquier motivo que impida finalizar la compra. Campo "Imagen" (captura del error): si el usuario no la tiene se guarda "No aplica" y se saltea.
- AsistenciaProcesoCompra: menú de diagnóstico según el paso en que se traba el usuario (ver producto, agregar al carrito, pagar, cargar dirección). Si identifica un error puntual, deriva a ErrorTecnicoEnCompra.
- Cambios: ramificación según canal de compra — tienda propia (RealizarCambio), Mercado Libre (CambioMercadoLibre) o CYBER (CambioCyber); consultas informativas van a PoliticaCambio y el seguimiento a EstadoCambioDevolucion.
- SeguimientoDeCompra: menú con ramas según tipo de entrega (envío a domicilio / retiro en sucursal).

*⚠ Cualquier comportamiento no cubierto explícitamente en este documento debe confirmarse con el referente técnico del cliente antes de la implementación.*

# Diagramas de Flujo (solo tienen que estar las que son personalizadas)

Representación visual de la lógica de decisión de los flujos condicionales listados arriba (ErrorTecnicoEnCompra, AsistenciaProcesoCompra, Cambios y SeguimientoDeCompra). ⚠ Diagramas a construir y validar con Yagmour.

## Checklist de Configuración por Cliente

Antes de dar por cerrado el alcance funcional de un cliente nuevo, completar y validar:

- Confirmar si se habilita el activador de Buscar Productos (hoy deshabilitado en el orquestador).
- Validar contenido de las FAQs: CYBER, Gift Card, Envíos, Envío Gratis, Cambios (tienda, Mercado Libre, CYBER), Devoluciones, Stock en sucursales, Venta mayorista y Descuento de bienvenida.
- Validar opciones de los menús: Promociones, Guía de Talles, Seguimiento de Compra, Factura de Compra y Asistencia en el Proceso de Compra.
- Base de Conocimientos de envíos: costos, plazos y empresa de correo actualizados.
- ticketReasons / ticketSubreasons y sectores de derivación configurados en el HelpDesk.
- Horario de atención para la derivación en vivo vs. ticket.
- Tiempo de inactividad y mensaje del seguimiento proactivo (Seguimiento 1).
- Definir si los flujos sin descripción (newFlow) deben eliminarse.
- Nombre y datos de contacto del referente de Yagmour para resolver puntos pendientes.

**SEGUNDA ETAPA**

1. Si un cliente habla de una sucursal el bot debe indicarle que debe comunicarse con la sucursal para que puedan ayudarla, la info de sucursales esta cargada en branches.
2. Si el cliente consulta por talle, el bot debe responder con las medidas generales de la categoría del produc

\[El texto devuelto por Drive termina cortado en "del produc"; el resto del documento no estaba disponible.\]
