> Fuente: Analisis_Funcional_Naldo_Gen6_v1 (https://docs.google.com/document/d/1bs1UFDiYj2xhOZo69KSHjz2NG7YdgYy0_AxIRrkOwvg/edit?usp=drivesdk&ouid=113568244751416058438)

**Snappy Commerce**

# Análisis Funcional — Naldo, Agente IA

*Naldo Bot · Relevamiento del comportamiento vigente en producción (Gen 5) como base para la migración a Gen 6*

## Cómo usar este documento

Este documento releva el comportamiento actual del Agente IA de Naldo (flujos nativos, flujos personalizados y configuración global) siguiendo la estructura del alcance funcional estándar de Snappy Commerce. Cada fila define el flujo, sus activadores y el comportamiento esperado.

Sirve para que Producto valide el comportamiento vigente, detecte vacíos funcionales e inconsistencias de configuración, y tenga documentado cada flujo antes de que pase de la UI de Snappy (Gen 5) a código TypeScript mantenido por Producto (Gen 6). La experiencia del usuario final no cambia con la migración.

**Criterio de documentación de flujos:** se documentan los flujos custom que ejecutan al menos una acción distinta de FAQ, asignación de topic o redirección a menú. Los flujos que solo responden con una FAQ quedan excluidos de las tablas.

*Las notas marcadas con ⚠ señalan configuraciones ausentes o puntos a confirmar. Los hallazgos y puntos abiertos se listan al final del documento.*

# Requerimientos Funcionales

## Configuración Global

| Flujo nativo | Qué hace el flujo | Valor configurado |
| :-: | :-: | :-: |
| País / versión de configuración | País de operación del bot y versión del esquema de comportamiento. | Argentina (Ar) · v2 |
| Crear contacto | Para crear un contacto nuevo, el agente solicita estos datos al usuario. | ACTIVADO. firstName, email, phone + campo adicional: documento de identidad del cliente (sap.Documento\_identidad) |
| Comenzar checkout (generateCheckout) | Genera el checkout de compra y comparte el link de pago, sin pedir datos al usuario. | No solicita datos: no pide nombre, apellido, email ni dirección de envío. Alcanza con compartir el link de compra (del carrito o generado). |
| Promociones | Responde las consultas de promociones solo con la información cargada en Conocimientos (no hay consulta por herramienta). Si no hay una promoción vigente informada, aclara que hay productos con promociones incluidas en su precio y ofrece la ayuda de una persona del equipo. | ACTIVADO |
| Búsqueda de Producto (productSearch) | Flujo nativo de búsqueda en catálogo de productos. | ACTIVADO |
| Recomendación de Producto (productRecommendation) | Flujo nativo de recomendación de productos. | ACTIVADO |
| Interacciones de redes sociales | El agente atiende también las interacciones de redes sociales. | ACTIVADO (includedRRSSinteractions) |
| Estado de Pedido (orderStatus) | Consulta el estado de un pedido vía integración. Hoy se usa solo dentro del flujo de cancelación. | Se usa en cancelarPedido (ver Flujos de Post-Venta) |
| Asistencia | Datos de contacto, escalamiento dentro de horario y flujo outOfHours fuera de horario. | ⚠ No figura en el JSON — a completar |
| Encuesta de satisfacción | Envío automático de encuesta al cierre de conversaciones. | ⚠ No figura en el JSON — a completar (hay dos flujos con survey, ver Flujos de Sistema) |

## Flujos de Pre-Venta

*Quedan fuera por ser solo FAQ: FacturaA. SolicitudDeCredito queda en Pre-Venta a la espera de confirmación.*

| Flujo | Activadores | Comportamiento esperado |
| :-: | :-: | :-: |
| **mayorista** | **Sí activa:** "Compra mayorista", "¿Se puede comprar al por mayor?", "Descuento por cantidad", "Venta a empresas". Pensado para consultar promociones especiales por venta mayorista o reventa. | 1) Deriva al sector sales.<br>2) Genera un ticket con motivo wholeSale y solicita dos datos: **Producto** (producto específico o variedad para armar un catálogo) y **Cantidad** (cuántos productos busca).<br>3) Envía la encuesta de satisfacción (survey). |
| **SolicitudDeCredito** *(clasificación a confirmar)* | **Sí activa:** el usuario quiere hacer una evaluación crediticia o solicitar un crédito. | 1) Deriva al sector administration.<br>2) Envía la FAQ de solicitud de crédito.<br>3) Ejecuta la búsqueda de sucursales cargadas en Snappy. |

## Flujos de Post-Venta

| Flujo | Activadores | Comportamiento esperado |
| :-: | :-: | :-: |
| **FlujopostventaQRok** | **Sí activa:** "Me llegó y no anda", "Necesito ayuda postcompra", "Casi no lo usé y se rompió", "Garantía", "Posventa", "Asistencia postventa", "Mi producto está roto".<br>**No activa:** Cambios, Devoluciones, Seguimiento de pedido, Boleta, Factura, "Dónde está mi compra", Recibo, Cancelar compra, Deuda, Crédito personal. | 1) Envía el mensaje exacto "Gracias por contactarte con el servicio de post venta de Naldo" (sin ofrecer asistencia adicional).<br>2) Deriva al sector flujopv y registra el topic flujoPV.<br>3) Genera un ticket con subreason **garantia** y solicita tres datos: **Producto** que presentó la falla, **Falla** observada y **Sucursal** donde compró.<br>4) Notifica: "El equipo de posventa se pondrá en contacto con vos dentro de las próximas 24hs hábiles. Muchas gracias!"<br>*⚠ El campo reason del ticket está vacío.* |
| **cancelarPedido** | **Sí activa:** el usuario quiere cancelar o anular un pedido, o se arrepintió de la compra. | 1) Indica al cliente que, para avanzar con la cancelación, hace falta conocer el estado del pedido (sin ofrecer asistencia adicional).<br>2) Pide el número de pedido o el email de la compra (uno solo), consulta el estado del pedido (orderStatus, namespace cancelacion) y ramifica según el estado. Si el pedido está entregado o ya cancelado, indica que lo revisará una persona del equipo y ofrece la derivación. |
| **cancelacion:orderStatus: approved, preparing, awaiting-payment, created, invoiced** | Se activa solo desde cancelarPedido. | Solicita el Motivo de la cancelación (si el cliente ya lo dijo, lo guarda y avanza), pide nombre, email, teléfono y DNI si todavía no los tiene, y genera un ticket con reason after, subreason cancelarPedido y el motivo como campo adicional. |
| **cancelacion:orderStatus: shipped** | Se activa solo desde cancelarPedido. | Informa que el pedido está en camino y no puede cancelarse hasta recibirlo; una vez recibido, se puede continuar con la devolución. No ofrece asistencia adicional. No genera ticket. |
| **cancelacion:orderStatus: inexistent** | Se activa solo desde cancelarPedido. | Avisa que no existe un pedido con el número compartido y pide verificar si es correcto. No genera ticket ni solicita datos adicionales. Si order_status devuelve un estado desconocido, se trata como inexistente. |
| **createTicket:override:postsales** | Override de la creación de tickets de postventa. Sin descripción cargada. | Ejecuta la búsqueda de sucursales cargadas en Snappy y luego crea el ticket. |
| **Cambio** *(a confirmar)* | **Sí activa:** "Cambios", "Quiero hacer un cambio de mi producto", "Quiero cambiar algo de mi compra". | Deriva al sector support y envía la FAQ de política de cambios. |

## Flujos de Sistema (Automatizados)

*Pendiente: se completa en el Bloque B, una vez aprobado el Checkpoint A y confirmadas las automatizaciones activas (seguimiento, SeguimientoCSAT, unsuscribe, CrediNaldo, bienvenida, notificaciones salientes).*

## Flujos con Lógica Condicional

*Pendiente: se completa en el Bloque B. Candidato detectado: cancelarPedido, que ramifica por estado del pedido (approved/preparing/awaiting-payment/created/invoiced, shipped, inexistent). CrediNaldo a confirmar según el contenido de su menú.*

## Diagramas de Flujo

*Pendiente: se generan en el Bloque C, una vez validados los flujos condicionales.*

## Requerimientos No Funcionales

*Pendiente: se completa en el Bloque B (horario de asistencia humana e instrucciones del bot: tono, idioma, prohibiciones, imágenes y audios).*

## Hallazgos y puntos abiertos

- **Estados de cancelación sin cubrir:** solo hay ramas para approved, preparing, awaiting-payment, created, invoiced, shipped e inexistent. Resuelto: si el pedido está entregado o ya cancelado, se deriva a una persona del equipo (ver cancelarPedido).
- **Contradicciones en las FAQs:** la política de cambios dice "hasta 30 días corridos" y también "luego de los 10 días no realizamos cambios". El plazo de reembolso figura como 30 días, 10 días y 5 a 10 días hábiles según la FAQ.
- **Descripciones vacías** en seguimiento, SeguimientoCSAT, los tres flujos de cancelación por estado y el override createTicket:override:postsales.
- **IDs de FAQ:** los flujos Cambio, FacturaA y SolicitudDeCredito referencian FAQs por UUID y el Excel no tiene columna de ID, por lo que no se pudo verificar qué FAQ responde cada uno.
- **Ticket de FlujopostventaQRok** con reason vacío y subreason garantia.
- **Datos no incluidos en el JSON:** plataforma de e-commerce, horario de asistencia humana, instrucciones del bot (tono, idioma, prohibiciones, imágenes y audios).
