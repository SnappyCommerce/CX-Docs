> Fuente: Alcance_Funcional_Portsaid (https://docs.google.com/document/d/1fVz8P4-7JDmGDK2u_JncGA2B5OF6jEKoTvwsARDXimI/edit?usp=drivesdk&ouid=113568244751416058438)

**Snappy Commerce**

# Alcance Funcional — Agente IA Nina (Portsaid)

## Requerimientos Funcionales

### Configuración Global

| Flujo nativo | Qué hace el flujo | Valor configurado |
| :-: | :-: | :-: |
| Crear contacto - Campos a solicitar al usuario | Para crear un contacto nuevo, el bot pide DNI y email. | DNI, email |
| Comenzar checkout - Requisitos de información de contacto | Para generar un checkout, el bot pide nombre, apellido y email. No pide teléfono ni dirección de envío. | firstName, lastName, email |
| Asistencia | Siempre que se dispara asistencia, el bot muestra los datos de contacto de la marca. Si es horario de atención, además escala en vivo a un asesor; fuera de horario, genera un ticket. | Contacto: siempre<br>Escala en vivo: en horario<br>Ticket: fuera de horario |
| Promociones y Sucursales | Se resuelven con la información ya cargada en Conocimientos (flujos nativos), sin necesidad de flujos personalizados adicionales. | true (activado) — contenido cargado en Conocimientos |
| Encuesta de Satisfacción (CSAT) | Se dispara por step propio en 3 flujos personalizados (Cambios, Devoluciones, Pedido cruzado). Además hay 2 toggles globales activos ("al abandonar" y "al finalizar la conversación"); "al transferir" está OFF. | Riesgo de envío duplicado — a confirmar con Producto |
| Estado de Pedido | El flujo nativo de Seguimiento de Pedido pide el número de documento al usuario antes de mostrar el estado de la orden. | true / dato solicitado: número de documento |
| Búsqueda de Producto (productSearch) | Búsqueda por nombre, categoría, marca y variantes (talle, color) contra una base propia de Snappy, sincronizada periódicamente desde el catálogo de VTEX. El stock se verifica en tiempo real contra VTEX antes de confirmar disponibilidad. | true / base propia + sync VTEX + stock real-time |

### Flujos de Pre-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Búsqueda y Recomendación de Producto | Usuario pregunta por un producto, categoría, marca, talle o color. | Ejecuta productSearch contra la base propia de Snappy. Ante 3 o más resultados, indaga por facetas hasta acotar la lista — sin cantidad fija de preguntas; si el usuario ya da todos los datos, no indaga. (Mejora de indagación dinámica en desarrollo en paralelo, fuera de este alcance.)<br>**Datos a recolectar: talle, color, categoría, marca u otras facetas relevantes (variable, según lo que falte).** |
| Consulta de Precio / Stock | Usuario pregunta precio, disponibilidad o stock de un producto puntual. | Responde con la información obtenida vía integración VTEX en tiempo real, no desde la base propia (que no refleja el stock actualizado). |
| Promociones | Usuario pregunta por descuentos, promos, cupones o "qué ofertas tienen". | Responde con el contenido nativo de Promociones, tomado de Conocimientos ya cargado. |
| Información de Sucursales | Usuario pregunta dirección, horarios o sucursal más cercana. | Responde con el contenido nativo de Sucursales, tomado de Conocimientos ya cargado. |
| ComoComprar | Usuario pide instrucciones o pasos para comprar online desde cero. Ej: "cómo compro en la web?". NO usar si tiene un problema activo al pagar (→ Checkout), o quiere comprar mayorista (→ Mayorista). | Fija sector "sales" y tópico "Comocomprar". Muestra la FAQ "¿Cómo comprar? ¿Cómo hago para comprar?". No escala. |
| Mayorista | Usuario quiere una compra mayorista o convertirse en revendedor/franquiciado. NO usar si es una consulta de compra minorista (→ ComoComprar), o un problema con una compra ya hecha (→ Checkout / Devoluciones). | Muestra la FAQ "¿Tienen venta Mayorista?". Fija sector "Sales". Crea un caso con razón "Venta mayorista".<br>**Datos a recolectar: Modelo de negocio (local multimarca / franquicia nacional / internacional).** |
| Looks y Combinaciones (TotalLooks) | Usuario pide recomendaciones de looks o cómo combinar prendas. NO usar si busca un producto específico sin pedir combinaciones (→ búsqueda estándar), o quiere reservar algo en un local (→ Reserva en Local). | Fija tópico "Looks". Muestra la FAQ "¿Qué es TOTAL LOOK?". Hoy es una respuesta informativa fija — no arma combinaciones dinámicas ni indaga prenda base/ocasión. |
| Reserva de Producto en Local | Usuario quiere reservar un producto en un local físico para retirarlo o probarlo. NO usar si pregunta por horario/dirección de un local (→ Sucursales, nativo). | Fija sector "sales". Crea un caso con razón "Productos" y mensaje de confirmación ("Solo nuestras operadoras pueden solicitar y confirmar la reserva…"). No pide qué producto ni en qué local antes de crear el caso. |
| Checkout Asistido (Comenzar Checkout) | Usuario avanza con el flujo nativo "Comenzar Checkout" para generar el link de compra. | Genera el link de checkout con los datos cargados.<br>**Datos a recolectar: nombre, apellido, email.** |
| Checkout — Problema Técnico o de Pago | Usuario tiene un problema técnico o de pago al intentar finalizar una compra en curso. Ej: "no me deja terminar la compra". NO usar si pregunta cómo comprar sin un problema activo (→ ComoComprar), o problema con factura de una compra ya hecha (→ Facturas). | Fija sector "checkout" y tópico "carrito". Escala a un humano. |
| Derivación a Vendedor / Asistencia Comercial | La consulta excede el alcance del bot en pre-venta o el usuario pide hablar con un asesor de ventas. | Dispara el flujo de Asistencia: escala en vivo o genera ticket según horario de atención.<br>**Datos a recolectar: datos de contacto (DNI, email) vía el flujo nativo Crear Contacto.** |

### Flujos de Post-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Estado de Pedido | Usuario pregunta por el estado de su pedido/envío. | El flujo nativo Seguimiento de Pedido muestra el estado, fecha estimada y tracking de la orden.<br>**Datos a recolectar: número de documento.** |
| Cambios (solicitudCambios) | Usuario quiere cambiar un producto ya recibido (talle, color, modelo) por otro. Ej: "quiero hacer un cambio", "me llegó el talle equivocado". NO usar si quiere devolver y recibir reintegro (→ Devoluciones), saber el estado de un cambio ya iniciado (→ Estado de cambio/devolución), le llegó el pedido equivocado (→ Pedido cruzado), o quiere modificar un pedido antes de recibirlo (→ Modificar Pedido). | Fija sector "postsales" y tópico "solicitud de cambio". Muestra la FAQ "¿Cómo puedo hacer un cambio?". Dispara la Encuesta de Satisfacción al cierre. No pide número de pedido ni crea ticket — no escala a un humano. |
| Devoluciones (SolicitudDevoluciones) | Usuario quiere devolver un producto y recibir un reintegro. Ej: "quiero devolver lo que compré". NO usar si quiere cambiar por otro talle/color (→ Cambios), o saber el estado de una devolución ya iniciada (→ Estado de cambio/devolución). | Fija sector "postsales" y tópico "solicitud de devolución". Escala a un humano con razón "Devoluciones". Dispara la Encuesta de Satisfacción al cierre. |
| Estado de Cambio / Devolución | Usuario quiere conocer el estado de un cambio o devolución ya iniciado. Ej: "en qué estado está mi cambio?". NO usar si quiere iniciar un cambio o devolución nuevo (→ Cambios / Devoluciones). | Fija sector "postventas". Muestra el estado de la orden vía el flujo nativo Seguimiento de Pedido.<br>**Datos a recolectar: número de documento.** |
| Modificar Pedido | Usuario quiere modificar talle/color/modelo de un pedido recién hecho, antes de que sea despachado. NO usar si quiere cambiar un producto ya recibido (→ Cambios), o solo la dirección de envío (→ Cambio de Domicilio). | Muestra la FAQ "¿Se puede modificar una compra?". Escala a un humano con razón "Post venta". No pide número de pedido ni qué quiere modificar antes de escalar. |
| Cambio de Domicilio | Usuario quiere modificar la dirección de entrega de un pedido ya confirmado. NO usar si quiere cambiar talle/color/modelo (→ Cambios), o consulta por la dirección de un local físico (→ Sucursales). | Escala directo a un humano con razón "Consultas Post Compra". No pide la nueva dirección antes de escalar. |
| Pedido Demorado | El pedido no llegó o superó la fecha de entrega estimada. NO usar si le llegó el pedido de otra persona (→ Pedido cruzado), o le llegó incompleto (→ Pedido incompleto). | Fija tópico "envío demorado". Escala a un humano. No pide número de pedido. |
| Pedido Cruzado | Usuario recibió un pedido que no le pertenece o con prendas distintas a las compradas. NO usar si el pedido no llegó aún (→ Pedido demorado), le faltan artículos (→ Pedido incompleto), o quiere hacer un cambio voluntario (→ Cambios). | Fija tópico "Pedido cruzado". Escala a un humano con razón "Consultas Post Compra". Dispara la Encuesta de Satisfacción al cierre. |
| Pedido Incompleto | Usuario recibió su pedido pero le faltaron artículos. NO usar si le llegaron productos que no son suyos (→ Pedido cruzado), o el pedido no llegó (→ Pedido demorado). | Fija tópico "pedido incompleto". Escala a un humano con razón "Post venta". No pide qué faltó antes de escalar. |
| Facturas | Consulta o problema con la factura de una compra. NO usar si es un problema con el pago al comprar (→ Checkout), o quiere devolver y recibir reintegro (→ Devoluciones). | Fija sector "Facturación". Muestra 2 FAQs ("¿Qué debo hacer si no me llegó la factura?" y "¿Pueden hacer Factura A de mi compra online?"). Escala a un humano con razón "Problema de Facturación". |
| Autorizar Retiro de Tercero | Usuario compró online y quiere autorizar a otra persona a retirar su pedido. NO usar si quiere reservar un producto (→ Reserva en Local), o cambiar el domicilio de entrega (→ Cambio de Domicilio). | Escala a un humano con razón "Post venta". No pide los datos que la FAQ exige (fotos de DNI, nota firmada) antes de escalar. |
| Formulario de Contacto | Usuario indica explícitamente que quiere completar un formulario o dejar sus datos para ser contactado. NO usar ante una consulta genérica sin mención de formulario (→ Medio de Contacto). | Envía el mensaje exacto "Voy a hacerte unas preguntas para asegurarte que mi compañero te ayude de la mejor manera". Crea un contacto y envía el mensaje exacto "Gracias por la información, escalo tu consulta".<br>**Datos a recolectar: sucursal favorita, fecha de cumpleaños (dd/mm).** |
| Medio de Contacto | Usuario pregunta por qué canal puede contactar a la marca, quiere hablar con un asesor, o cómo lo van a contactar. NO usar si tiene una consulta específica de post-venta (→ el flujo correspondiente), o quiere completar un formulario (→ Formulario). | Muestra la FAQ "¿Cuáles son los medios de contacto?". Escala a un humano con razón "Post venta". |
| Cancelación de Pedido | Usuario quiere cancelar un pedido ya realizado. Ej: "quiero cancelar mi pedido". | Responde con la FAQ de cancelaciones (Conocimientos) y deriva a un humano para gestionar la cancelación. |
| Producto Dañado o con Falla | Usuario recibió un producto dañado o con una falla. Ej: "me llegó roto", "tiene un defecto". | Responde con la FAQ "¿Qué debo hacer si recibo un producto dañado o con falla?" (Conocimientos) y deriva a un humano. |
| Encuesta de Satisfacción (CSAT) | Se dispara automáticamente al cierre de Cambios, Devoluciones y Pedido Cruzado (step propio), además de los 2 toggles globales activos (al abandonar / al finalizar la conversación). | Envía la encuesta "¿Qué te pareció la atención de nuestra asistente virtual?" — Buena (positiva) / Regular (neutral) / Mala atención (negativa). Riesgo de envío duplicado — a confirmar con Producto. |
| Derivación a Soporte Post-venta | La consulta post-venta excede el alcance del bot o el usuario pide hablar con un humano. | Dispara el flujo de Asistencia: escala en vivo o genera ticket según horario de atención.<br>**Datos a recolectar: datos de contacto (DNI, email) vía el flujo nativo Crear Contacto.** |

### Programa de Fidelización Friends

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Consulta de Puntos / Registro (puntos) | Usuario quiere consultar su saldo de puntos Friends o registrarse. Ej: "cuántos puntos tengo?". NO usar si quiere canjear sus puntos (→ Uso de puntos), no puede entrar a su cuenta (→ Acceso a mi cuenta Friends), o su DNI ya está en uso (→ DNI en uso). | Si tiene puntos disponibles, pregunta si quiere saber en qué sucursales canjearlos o menciona el widget Friends de la web; si no tiene puntos, informa que empezará a acumular con la próxima compra.<br>**Datos a recolectar: DNI.** |
| Uso de Puntos (UsopuntosFriends) | Usuario consulta cómo usar o canjear sus puntos Friends al comprar. Ej: "cómo uso mis puntos para pagar?". NO usar si pregunta cuánto tiene acumulado (→ Consulta de puntos), o no se le acreditaron (→ Inconvenientes con Friends). | Muestra la FAQ "¿Cómo canjeo mis puntos de Friends?". No escala. |
| Inconvenientes con Friends | Usuario reporta un error técnico con su cuenta Friends (acreditación, datos incorrectos). Ej: "mis puntos no se acreditaron". NO usar si no puede iniciar sesión (→ Acceso a mi cuenta), o su DNI ya está en uso (→ DNI en uso). | Fija tópico "programas de fidelidad". Escala a un humano con razón "Post venta". |
| Acceso a mi Cuenta Friends | Usuario no puede iniciar sesión o acceder a su cuenta Friends. Ej: "olvidé la contraseña". NO usar si es un problema de acreditación de puntos (→ Inconvenientes con Friends), o su DNI ya está en uso (→ DNI en uso). | Muestra la FAQ "¿Olvidé mi clave para ingresar a Friends?". No escala. |
| DNI en Uso en Friends | Al usuario le aparece que su DNI ya está en uso o asociado a otro mail en Friends. Ej: "mi DNI ya está en uso". NO usar si no puede ingresar por contraseña (→ Acceso a mi cuenta). | Muestra la FAQ "¿Qué pasa si me indica que mi DNI está en uso en Friends?". No escala. |

### Flujos de Sistema (Automatizados)

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Seguimiento Proactivo | Sin activador por mensaje del usuario — se activa automáticamente al finalizar una conversación sin una consulta nueva en curso. | Envía un mensaje (prompt de IA, no exacto) pidiéndole al Agente IA que haga seguimiento según lo conversado; si era una consulta de venta, ofrece asistencia humana. |
