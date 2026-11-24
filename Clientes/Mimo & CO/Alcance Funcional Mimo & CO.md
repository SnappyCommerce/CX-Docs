> Fuente: Flujos_Activadores_Comportamiento_MimoCo (https://docs.google.com/document/d/1m_Dsv6KFbXGkk9vHWKwYEwF90uyjpSiNFXWZaORwTGQ/edit?usp=drivesdk&ouid=113568244751416058438)

# Flujos, Activadores y Comportamiento Esperado

**Mimo & Co — Agente IA · Documento de referencia única (extraído de configuración v2)**

*Este documento es la fuente única de verdad sobre cada flujo del Agente IA: qué lo activa y qué debe hacer. Cada fila es autocontenida. Los flujos con condicionales o ramificaciones internas están señalados con un enlace al diagrama correspondiente en el Anexo.*

*Los links a mimo.com.ar en los flujos de post-venta (Locales, Cambio de producto, Devolución de pedido, Cancelación de pedido, Estado de pedido) son una decisión de diseño estratégica documentada por Producto — permiten que el cliente autogestione su trámite de forma independiente, sin depender del bot. No es una inconsistencia a corregir; se mantiene igual en la migración a Gen 6.*

# Flujos de Pre-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| **Promociones** | **Sí activa:** "¿Qué promociones tienen?", "¿Hay 2x1?", "¿Cuotas sin interés con Visa?", "Tengo tarjeta Galicia, ¿cuotas?"<br>**No activa:** Pago de una compra puntual · Nueva temporada · Stock/disponibilidad | Responde consultando la base de Conocimientos sobre promociones vigentes. |
| **Búsqueda y Recomendación de Producto** | **Sí activa:** Mención de un producto puntual, una categoría, o una consulta abierta por estilo/ocasión (sin frase fija de activación) | La búsqueda de SKUs ya filtra por stock disponible — el conteo de resultados (1 exacto / pocos / muchos / pista vaga / 0 resultados) considera solo lo que hay disponible, no el catálogo completo. Al preguntar talle o color, siempre da opciones concretas dentro de lo que hay en stock (ej: ¿color negro o marrón? ¿talle 42 o 43?), nunca una pregunta abierta. Como el usuario elige entre opciones ya disponibles, no hace falta un chequeo de stock aparte al final — elegir la combinación ya alcanza para agregarla al carrito. Máximo 2 rondas de preguntas de afine antes de mostrar una muestra.<br>*⚠ Las facetas de filtro salen de las colecciones de VTEX, no de las categorías — la información de categorías en VTEX está desactualizada. La búsqueda corre contra una base propia de Snappy sincronizada periódicamente con VTEX, no contra VTEX en vivo. Sinónimos/variantes (ej. "zapatillas" vs. "sneakers") todavía dependen de un bug conocido del buscador de VTEX — pendiente definir con Producto si se resuelve en la base propia o se compensa desde el Agente IA.*<br>***→ Ver Diagrama 2 (Anexo)*** |
| **Checkout Asistido** | **Sí activa:** Se activa automáticamente al confirmarse el agregado al carrito — no se activa por un mensaje directo del usuario ni sin ese contexto previo | Solicita nombre y mail solo si faltan, comparte el link de checkout, y ofrece cross-sell una única vez por conversación, todo en el mismo intercambio. La gestión del carrito (agregar, borrar, modificar) antes de pagar nunca genera un link nuevo.<br>**→ Ver Diagrama 3 (Anexo)** |
| **Venta mayorista** | **Sí activa:** Usuario muestra interés en comprar al por mayor (sin ejemplos textuales cargados) | Al estar desactivado, si el usuario pregunta esto, el bot responde con lo que tenga cargado en Conocimientos — no ejecuta la lógica de indagación ni arma una solicitud interna. |
| **Problema al comprar** | **Sí activa:** Usuario tiene un problema técnico al intentar completar una compra (sin ejemplos textuales cargados) | Al estar desactivado, si el usuario pregunta esto, el bot responde con lo que tenga cargado en Conocimientos — no deriva a un asesor por esta vía. |

# Flujos de Post-Venta

*Locales, Cambio de producto, Devolución de pedido, Cancelación de pedido y Estado de pedido comparten un link a mimo.com.ar en vez de resolver la consulta en el chat — es una decisión de diseño deliberada (autogestión independiente del cliente), no una inconsistencia.*

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| **Locales** | **Sí activa:** "¿Dónde está el local?", "Sucursales cercanas", "¿Cuál es el horario?", "Dirección del local"<br>**No activa:** Consulta de stock en un local puntual → Búsqueda de Producto | Comparte el link a mimo.com.ar/locales, donde el cliente busca el local más cercano con dirección, horario y contacto. |
| **Cancelación de pedido** | **Sí activa:** "Quiero cancelar mi pedido", "Cancelar mi compra", "No quiero recibirlo más"<br>**No activa:** Consulta por modificar el pedido · Consulta por política de cancelación | Comparte el link de gestión de cambios y devoluciones, donde el cliente completa la solicitud de forma independiente. |
| **Cambio de producto** | **Sí activa:** "Quiero cambiar un producto", "Quiero hacer un cambio", "Quiero cambiar mi compra"<br>**No activa:** Consulta por política de cambios → Políticas de Cambios | Comparte el link de gestión de cambios y devoluciones. |
| **Políticas de Cambios** | **Sí activa:** "¿Cómo hago un cambio?", "¿Cuál es la política de cambios y devoluciones?", "¿Puedo cambiar un producto que compré?", "¿Cuántos días tengo para devolver algo?"<br>**No activa:** Intención de realizar el cambio en sí → Cambio de producto | Responde consultando la base de Conocimientos sobre políticas de cambios y devoluciones — la única de este grupo que responde en el chat en vez de derivar a un link, porque es solo información, no una gestión que requiera datos del pedido. |
| **Devolución de pedido** | **Sí activa:** "Quiero devolver un producto", "Devolución de mi pedido completo"<br>**No activa:** Consulta por política de devoluciones → Políticas de Cambios | Comparte el link de gestión de cambios y devoluciones. |
| **Estado de pedido** | **Sí activa:** "¿Dónde está mi pedido?", "Seguimiento de mi compra" | Comparte el link de acceso a la cuenta del cliente (login), donde puede ver el estado de sus pedidos por su cuenta. |
| **Modificación de pedido** | **Sí activa:** Cliente quiere modificar un pedido antes de recibirlo (sin ejemplos textuales cargados) | Al estar desactivado, si el usuario pregunta esto, el bot responde con lo que tenga cargado en Conocimientos — no ejecuta la lógica de recolectar datos ni arma una solicitud interna. |

# Flujos de Sistema y Asistencia

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| **Asistencia Humana** | **Sí activa:** Usuario pide hablar con un asesor humano, en cualquier momento | Se ramifica según el horario: en horario, deriva a un asesor en vivo. Fuera de horario, envía un mensaje fijo informando el horario de atención (lunes a viernes de 9:00 a 18:00 hs, excepto feriados) y además genera una solicitud interna. El contacto queda guardado para seguimiento en ambos casos.<br>**→ Ver Diagrama 1 (Anexo)** |

# Anexo

**Diagrama 1 — Asistencia Humana**

**Diagrama 2 — Búsqueda y Recomendación de Producto**

**Diagrama 3 — Checkout Asistido**

*(Los diagramas son imágenes en el documento original y no se incluyen en la conversión.)*

# Testeos

# Búsqueda y Recomendación de Producto: conversaciones de ejemplo

*Mimo & Co, Agente IA. Propuesta para reemplazar la fila "Búsqueda y Recomendación de Producto" del documento Flujos_Activadores_Comportamiento_MimoCo. Pendiente de revisión de Jess. Los productos y precios de los ejemplos son ilustrativos.*

## Comportamiento general

Ejecuta la búsqueda contra el catálogo del cliente, filtrada por stock, y muestra un carrusel con los resultados. Antes de buscar necesita saber tipo de prenda, si es para nena o nene y la edad, y los pide juntos en un solo mensaje y con ejemplos concretos. Nunca pregunta el talle de ropa: lo recomienda por edad con la tabla oficial de mimo.com.ar/talles, y si la edad cae entre dos talles recomienda el más grande. Tras mostrar productos, escribe un mensaje con la recomendación de talle, una viñeta por producto y, en un renglón aparte, la pregunta de cierre sobre el carrito. Tono cercano y profesional, sin emojis (máximo uno), sin repetir la misma apertura y con un solo mensaje por turno.

## 1. Pedido vago

**Cliente:** Quiero ver ropa

**Bot:** Te ayudo a encontrar lo que buscás. Contame un poquito:

- ¿Qué tipo de prenda buscás (remeras, bodies, pantalones, conjuntos, vestidos)?
- ¿Es para una nena o un nene?
- ¿Qué edad tiene?

Así puedo recomendarte lo que mejor le quede.

*Regla: no busca hasta tener tipo de prenda, sexo y edad. Hace las preguntas juntas, con ejemplos y explicando para qué las pide. Aplica también a "Recomendame algo" y "No sé, quiero mirar".*

## 2. Solo tipo de prenda

**Cliente:** Bodies

**Bot:** Dale, ¿es para una nena o un nene? ¿Qué edad tiene? Así te muestro lo que mejor le quede.

*Regla: no muestra productos todavía. Pregunta lo que falta en un solo mensaje.*

## 3. Solo destinatario y edad

**Cliente:** Busco algo para mi nene de 3 años

**Bot:** ¿Estás buscando algo en particular, como remeras, pantalones, conjuntos o buzos? ¿Es para todos los días o para alguna ocasión especial?

*Regla: ya sabe sexo y edad, así que no los repregunta. Pide el tipo de prenda con ejemplos.*

## 4. Edad sin unidad

**Cliente:** Un body para mi nena, tiene 2

**Bot:** ¿2 meses o 2 años?

*Regla: un número solo nunca se interpreta. El talle cambia por completo entre bebé y niño.*

## 5. Pedido completo

**Cliente:** Remeras para mi nena de 4 años

**Bot:** [carrusel de productos]

**Bot:** Para 4 años te recomiendo el talle 06 (4-6 años). Estas son las opciones:

- REMERA KID TWO - de algodón, con estampa al frente. Precio: $14.900
- REMERA KID PARTY - con detalle de brillos para ocasiones especiales. Precio: $27.900

¿Te gustó alguna de estas opciones para agregar al carrito? ¿O preferís ver otras opciones?

*Regla: carrusel más un mensaje con talle recomendado por edad, una viñeta por producto y la pregunta de cierre en un renglón aparte. No pregunta el talle. Si la edad cae entre dos talles, recomienda el más grande.*

## 6. Cliente que quiere ver ya

**Cliente:** Mostrame vestidos, no quiero contestar preguntas

**Bot:** [carrusel de vestidos]

**Bot:** Te dejo estas opciones para empezar:

- VESTIDO KID MALASIA - liviano, con tirantes y falda amplia. Precio: $37.900
- VESTIDO JR EPI - con cintura marcada y lazo. Precio: $41.900

¿Te gustó alguno para agregar al carrito? ¿O preferís ver otras opciones?

*Regla: si el cliente pide ver productos ya, busca con lo que tiene y recién después afina. Como no sabe la edad, no recomienda talle todavía.*

## 7. Afine con opciones concretas

**Cliente:** Un vestido para mi hija de 5 años

**Bot:** ¿Lo buscás para una fiesta, para algo casual o para todos los días?

**Cliente:** No sé, lo que tengan

**Bot:** [carrusel] y mensaje con talle 06 (4-6 años), viñetas y la pregunta de cierre.

*Regla: las preguntas de afine siempre ofrecen opciones concretas (ocasión: fiesta, casual o diario; color: hasta tres opciones), nunca preguntas abiertas como "¿qué color o estilo buscás?". Máximo 2 rondas antes de mostrar una muestra.*

## 8. Regalo

**Cliente:** Busco un regalo para un hijo

**Bot:** Qué lindo, te ayudo a elegir. ¿Qué edad tiene tu hijo? ¿Preferís ropa, calzado o una Gift Card? Si tenés un presupuesto en mente, también contame.

*Regla: pregunta para quién es, edad y si prefiere ropa, calzado o Gift Card. La palabra "regalo" no va en la búsqueda: busca por tipo de prenda, edad y ocasión.*

## 9. Calzado con presupuesto

**Cliente:** Busco zapatillas para mi nene, hasta $50.000

**Bot:** Dale. ¿Qué número calza habitualmente? Si no lo sabés, pasame el largo del pie en centímetros, de talón a punta.

**Cliente:** Calza 30

**Bot:** [carrusel] Estas son las opciones en talle 30 dentro de tu presupuesto:

- ZAPA JR CORDON - de lona, con puntera reforzada y suela flexible. Precio: $27.900
- PANCHITA JR BASIC - práctica para el verano, con elásticos laterales. Precio: $31.900

¿Te gustó alguna de estas opciones para agregar al carrito? ¿O preferís ver otras opciones?

*Regla: en calzado pregunta el número habitual o el largo del pie, y nunca deduce el número por la edad. Con el número y el presupuesto, busca con ambos.*

## 10. Sinónimos

**Cliente:** Busco sneakers para nene

**Bot:** Dale. ¿Qué edad tiene o qué número calza?

*Regla: entiende "sneakers" como zapatillas o zapas. Mientras el buscador de VTEX tenga el bug de sinónimos, es un riesgo conocido.*

## 11. Sin resultados o sin stock en el talle

**Cliente:** Zapas en talle 30, hasta $20.000

**Bot:** En talle 30 no tengo zapas por debajo de $20.000, pero tengo estas opciones cercanas en precio:

- PANCHITA JR MALAGA - de capellada textil y calce estándar. Precio: $19.900
- ZAPA JR CORDON - de lona, con puntera reforzada. Precio: $27.900

¿Te gustó alguna de estas opciones para agregar al carrito? ¿O preferís ver otras opciones?

*Regla: si no hay lo pedido, lo dice con una alternativa concreta (otro precio, color, tipo de prenda o talle cercano). Nunca arranca con "no encontré" si igual muestra productos, ni anuncia productos que no muestra.*

## 12. Precio

**Cliente:** ¿Cuánto vale la primera?

**Bot:** La REMERA KID TWO sale $14.900. Para 4 años te recomiendo el talle 06. ¿Querés que te la agregue al carrito?

*Regla: responde lo que preguntó y cierra con una sola pregunta que empuja a comprar.*

## 13. Objeción de precio

**Cliente:** Uf, está muy caro

**Bot:** Te entiendo. Podemos buscar algo similar en otro rango. ¿Hasta qué precio te gustaría gastar? Así te muestro alternativas.

*Regla: reconoce la objeción y pregunta el presupuesto. No dice "no encontré opciones" ni repite la búsqueda sin ese dato.*

## 14. Descuentos

**Cliente:** ¿Tiene descuento?

**Bot (si el producto trae descuento):** Sí, la REMERA KID TWO tiene 20% OFF. Precio final: $11.920. ¿Querés que te la agregue al carrito?

**Bot (si no trae descuento):** Hoy la REMERA KID TWO está a su precio de $14.900. ¿Querés que te la agregue al carrito?

*Regla: menciona un descuento solo si los datos del producto lo traen. Nunca habla de financiación ni cuotas en esta respuesta.*

## 15. Agregar al carrito

**Cliente:** Agregame la primera

**Bot:** Pasa a Checkout Asistido (flujo aparte, sin cambios).

## Se mantiene del documento actual

Las facetas de filtro salen de las colecciones de VTEX, no de las categorías (la información de categorías en VTEX está desactualizada). La búsqueda corre contra una base propia de Snappy sincronizada periódicamente con VTEX, no contra VTEX en vivo. Ver Diagrama 2 (Anexo).
