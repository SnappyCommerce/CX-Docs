> Fuente: Alcance_Funcional_Hudson_UY (https://docs.google.com/document/d/1BhEu_ADFagyhonZCkqXj0RfvpBaJ5O8k2bN8YDsTxRY/edit?usp=drivesdk&ouid=113568244751416058438)

**Snappy Commerce**

**Alcance Funcional Genérico — Hudson Uruguay**

*Documento base para nuevos clientes · Plantilla de referencia completada con la configuración estándar del bot IA*

## Cómo usar este documento

Este documento describe la estructura estándar de flujos de un Agente IA de atención al cliente y ventas de Snappy Commerce. Sirve como punto de partida para el relevamiento (discovery) de un cliente nuevo: cada fila define el flujo, sus activadores típicos y el comportamiento esperado en términos generales.

Los textos entre corchetes y las notas marcadas con ⚠ señalan puntos que dependen de la política, integración o contenido propio de cada cliente y deben completarse durante el discovery. Los nombres de los flujos pueden mantenerse o adaptarse a la nomenclatura interna del cliente.

# Requerimientos Funcionales

| Flujo nativo | Qué hace el flujo | Valor configurado |
| :-: | :-: | :-: |
| Crear contacto — campos a solicitar | Para crear un contacto nuevo, el bot pide los datos mínimos de identificación. | Nombre, teléfono, mail (optativos) |
| Comenzar checkout — requisitos de información de contacto | Para generar un checkout, el default pide nombre, apellido y email. | firstName, lastName, email |
| Asistencia | Al dispararse asistencia, el bot genera un ticket de soporte (pidiendo los datos que falten). En horario de atención la conversación se escala en vivo a un asesor; fuera de horario queda sólo el ticket a la espera de gestión. | Contacto: Siempre<br>Escalar: Dentro de horario<br>Ticket: Fuera de horario Unicamente cuando es una venta |
| Encuesta de satisfacción | Encuesta de Satisfacción automática global. | true |
| Estado de Pedido | Consulta de estado de pedido. Los datos disponibles (tracking, fecha estimada) dependen de lo que devuelva la integración. | PENDIENTE integración<br>Logística: UES Pendiente de definición |
| Horario de atención | Define cuándo se escala en vivo y cuándo se genera sólo ticket. | Días hábiles de 10 a 17 h de Argentina (GMT-3) |

## Flujos de Pre-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Búsqueda y Recomendación de Producto | El usuario pregunta por un producto, categoría, línea, medida, material o atributo ("tenés sartenes aptas para inducción?", "qué cacerola me recomendás"). | Búsqueda de producto con Shopify |
| Consulta de Precio / Stock | El usuario pregunta precio o disponibilidad de un producto puntual. | Búsqueda de producto con Shopify |
| Promociones y descuentos | El usuario pregunta por descuentos, promos, cupones u ofertas vigentes. | Conocimiento: Las cuotas y promociones se visualizan en la publicación de cada producto. Hay descuento de primera compra suscribiéndose al newsletter en https://www.hudsoncocina.com.uy/ |
| Medios de pago | El usuario consulta formas de pago, cuotas o financiación. | Conocimiento: Recomienda simular la compra en la tienda online: elegir el producto y avanzar hasta el resumen de compra, donde se muestran los métodos de pago disponibles. |
| Checkout Asistido | El usuario quiere comprar o pide el link de compra ("quiero finalizar la compra", "pasame el link"). | Solicita los datos de contacto configurados (firstName, lastName, email) y genera el link de checkout. |
| Checkout Asistido — Gestión del Carrito | El usuario quiere agregar, quitar o modificar productos de un carrito ya iniciado. | Actualiza el carrito vía integración según la acción solicitada, validando stock. |
| Problemas durante la compra | El usuario no puede agregar productos al carrito, le rechazan la tarjeta o se le canceló la compra. | Conocimiento No deriva |
| Venta mayorista / comercializar productos | El usuario consulta por compras mayoristas o por comercializar productos Hudson. | Deriva a ventas.uy@hudsoncocina.com, pidiendo que envíe la consulta o propuesta por ese medio. a definir |
| Branches | El usuario pregunta si hay tienda física o dónde ver los productos. | Conocimiento: No hay tienda física. Se puede elegir un punto de retiro al momento de la compra. Además hay dos bazares que venden productos Hudson: Bazar Zoom (18 de Julio 1707) y News (Av. 18 de Julio 1549, CP 11200). |
| Derivación a Vendedor / Asistencia comercial | El usuario pide hablar con un asesor de ventas o la consulta excede el alcance del bot. | El bot consulta el motivo al usuario, para ventas solicita asistencia, si no es ventas deriva al mail correspondiente. |

## Flujos de Post-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Modificación de un pedido en curso | El usuario quiere cambiar dirección, mail u otro dato de un pedido aún no despachado. | Conocimiento: reclamos.uy@hudsoncocina.com indicando nombre, apellido, mail, número de compra y los datos a modificar. |
| Cambios y Devoluciones | El usuario quiere cambiar o devolver un producto ya comprado. | Plazo: 30 días desde la compra.<br>Si compró por Mercado Libre: derivar a la mensajería interna de la plataforma.<br>Si compró por la tienda online: enviar a reclamos.uy@hudsoncocina.com los datos |
| Pedido incompleto | El usuario recibió menos productos de los que compró o le falta una pieza. | Si compró por Mercado Libre: mensajería interna de la plataforma.<br>Si compró por la tienda online: enviar a reclamos.uy@hudsoncocina.com los datos+ Que diferencie |
| Reclamos / Ticket | El usuario reporta un problema (producto dañado, faltante, error de facturación, etc.). Ver el listado completo de 25 tipos en el Anexo A. | TODA solicitud de reclamo se deriva a reclamos.uy@hudsoncocina.com. |
| Encuesta de Satisfacción (CSAT) | Se dispara automáticamente al cierre de una gestión o conversación. | true |
| Derivación a Soporte Post-venta | La consulta post-venta excede el alcance del bot o el usuario pide hablar con un humano. | No derivamos y damos el conocimiento del email |

## Flujos de Sistema (Automatizados)

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Mensaje de Bienvenida | Sin activador por mensaje del usuario — se dispara desde automatizaciones al iniciar el primer contacto. | apagado |
| Seguimiento Proactivo | Automático tras un tiempo configurable de inactividad (referencia: 5 minutos en un flujo de venta en curso). | apagado |
| Cierre por Inactividad + CSAT | Automático al quedar la conversación inactiva. | prendido |
| Notificaciones / Broadcast | Se dispara desde un envío de notificación o broadcast programado. | prendido |

# Requerimientos No Funcionales

| Categoría | Requerimiento |
| :-: | :-: |
| Lenguaje | Español rioplatense con voseo, cercano y profesional. |
| Personalidad | Amable, motivado, servicial y observador. *Sos un asistente profesional y amigable de Hudson Cocina, la marca de productos para cocina más confiable de Uruguay. Tenés conocimiento profundo sobre todos nuestros productos. Sos entusiasta, servicial y te enfocás en ayudar a los clientes a encontrar el producto perfecto para sus necesidades culinarias. Hablás de manera cercana pero profesional, reflejando la calidad y confianza que representa Hudson.* |
| Testing | El código/JSON debe pasar el test suite estándar de Snappy (umbral 85%+) antes de cualquier subida a producción. |
| Conversaciones | El agente debe contar con contexto de las conversaciones previas de cada contacto y poder responder usando esa información (por ejemplo, reconocer a qué reclamo se refiere el usuario en una conversación nueva). |
| Comportamiento — Fallback | Cuando no cuente con información suficiente en Conocimientos, no debe suponer ni inventar. Debe reconocerlo con sinceridad amable y derivar al formulario de contacto o al correo que corresponda según el tema. |

## Flujos con Lógica Condicional

Los siguientes flujos tienen ramificaciones internas y requieren un diagrama de decisión propio. Se listan aquí los puntos de ramificación que ese diagrama debe cubrir:

- Cambios y devoluciones:
- Pedido incompleto:
- Solicitud de asistencia: el bot consulta el motivo de la solicitud de asistencia, si se trata de ventas solicita asistencia, si no se trata de ventas deriva al mail correspondiente.

*⚠ Cualquier comportamiento no cubierto explícitamente en este documento debe confirmarse con el referente técnico del cliente antes de la implementación.*

# Diagramas de Flujo

Representación visual de la lógica de decisión de los flujos condicionales listados arriba. Se entregan como archivo editable adjunto a este documento ([Diagramas_Flujo_Bot_IA_UY.pptx](https://docs.google.com/presentation/d/1MUEhQlCSoieWhs0loZhAEqcFv8OQbuqJ/edit?slide=id.p1#slide=id.p1)). Los rótulos marcados como PENDIENTE deben reflejarse en los diagramas una vez definidos con Hudson.

**Diagrama 3 - Solicitud de asistencia.**

```
┌──────────────────────────────────┐
│  Usuario pide hablar con un asesor │
└─────────────────┬────────────────┘
                  │
                  ▼
        ┌───────────────────┐
        │  Bot indaga el    │
        │  motivo           │
        └─────────┬─────────┘
                  │
                  ▼
           ◇─────────────────◇
           │ ¿Cuál es el motivo? │
           ◇─────────────────◇
          /                    \
    Ventas                      Reclamo / Cambios / etc
        /                          \
       ▼                            ▼
┌─────────────────┐       ┌─────────────────────┐
│ Bot solicita    │       │ Bot deriva al correo │
│ asistencia al   │       │ de la tienda         │
│ asesor          │       │                      │
│ disponible      │       │                      │
└─────────────────┘       └─────────────────────┘
```

# Checklist de Configuración - Uruguay

Antes de dar por cerrado el alcance funcional de la operación de Uruguay, completar y validar:

- Políticas de cambios y devoluciones: plazo de 30 días, datos a solicitar y derivación a reclamos.uy@hudsoncocina.com.
- Operador logístico (UES), zonas de cobertura, costos y tiempos estimados.
- Plataforma de e-commerce e integración de estado de pedido / tracking.
- ticketReasons y sectores de derivación configurados en el HelpDesk / CRM.
- Contenido de la Base de Conocimientos: características de producto, envíos, promociones y preguntas frecuentes
- Casillas de correo activas y monitoreadas: reclamos.uy@hudsoncocina.com, ventas.uy@hudsoncocina.com, catalina.modarelli@hudsoncocina.com.
- Número de WhatsApp de atención de Uruguay.
- Nombre y datos de contacto del referente de Hudson para Uruguay.

# Parámetros de la operación de Uruguay

Referencia rápida de todos los valores que cambian entre países. Es la tabla a revisar al portar el agente de un país a otro.

| Parámetro | Valor para Uruguay |
| :-: | :-: |
| Tienda online | https://www.hudsoncocina.com.uy/ |
| Mail de reclamos | reclamos.uy@hudsoncocina.com |
| Mail de ventas / mayorista | ventas.uy@hudsoncocina.com |
| Mail de marketing | catalina.modarelli@hudsoncocina.com |
| WhatsApp de atención | +5491121853997 (número argentino — PENDIENTE confirmar número local) |
| Horario de atención | Días hábiles de 10 a 17 h de Argentina (GMT-3) |
| Transportista / logística | UES |
| Tiempos de entrega | Hasta 48 h / 72 h con Mercado Envíos y en el transcurso del día con FLEX. |
| Costos de envío | UES Fulfilment Montevideo: costo normal $120. UES Fulfilment Interior: costo normal $120. |
| Retiro / puntos de venta | No hay tienda física. Se puede elegir un punto de retiro al momento de la compra. Además hay dos bazares que venden productos Hudson: Bazar Zoom (18 de Julio 1707) y News (Av. 18 de Julio 1549, CP 11200). |
| Marketplaces | Mercado Libre |
| Documento de identidad solicitado | CI |
| Formato de dirección | Calle, número de calle, número de apartamento, departamento y localidad |
| Tabla comparativa | https://www.hudsoncocina.com.uy/pages/comparativa |
| Usos y cuidados | https://www.hudsoncocina.com.uy/pages/usos-y-cuidados |
| Preguntas frecuentes / términos | https://www.hudsoncocina.com.uy/pages/terminos-y-condiciones |
| Ofertas | No hay página de ofertas; sólo descuento por newsletter. |
| Formulario de contacto | PENDIENTE — definir formulario de contacto de Uruguay |
| Registro de lenguaje | Español rioplatense con voseo, cercano y profesional |
| Garantía | 6 meses ante fallas de fabricación |
| Plazo de cambio / devolución | 30 días desde la compra |
