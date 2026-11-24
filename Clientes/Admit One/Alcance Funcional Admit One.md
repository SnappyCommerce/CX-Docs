> Fuente: Alcance Funcional - Admit One (https://docs.google.com/document/d/1ZXwyMfAVuOR2tyArHetxNGrbjO3Ik5r-joNEodw9bCg/edit?usp=drivesdk&ouid=113568244751416058438)

Snappy Commerce

# Alcance Funcional — Agente IA de Atención al Cliente — Admit One

Documento de alcance basado en la configuración vigente del agente (config v2) · Octubre 2026

## Cómo usar este documento

Este documento describe los flujos configurados actualmente en el Agente IA de Admit One, marca argentina de indumentaria deportiva. Cada fila define el flujo, sus activadores y el comportamiento esperado. Los puntos marcados como [Pendiente de confirmar con el cliente] no surgen de la configuración actual y deben validarse con el referente del cliente.

## Requerimientos Funcionales

### Configuración Global

| Flujo nativo | Qué hace el flujo | Valor configurado |
|---|---|---|
| Crear contacto - Campos a solicitar al usuario | Para crear un contacto nuevo, el agente IA solicita nombre, apellido, email y teléfono. Antes de pedirlos aclara que son para que un asesor humano pueda recontactarlo o informarle novedades de su caso. | firstName, lastName, email, phone |
| Comenzar checkout - Requisitos de información de contacto | No configurado explícitamente; el agente no tiene flujos de venta asistida. | No configurado |
| Asistencia | Toda derivación a un asesor humano requiere generar un ticket. Al derivar, el agente informa el horario de atención (días hábiles de 9 a 17 hs) y no vuelve a intervenir en la conversación. | default.requirements: ticket |
| Personalidad | Tono amable, respetuoso, empático, con algunos emojis y expresiones rioplatenses ("Dale", "Te cuento", "Mirá"). No responde consultas ajenas a la indumentaria deportiva de la marca. Nunca informa estados de pedido en inglés. | Configurado |
| Promociones | Flujo nativo de Promociones. | No configurado [Pendiente de confirmar con el cliente] Responde con faq |
| Factura del pedido | Flujo nativo de Facturación. | No configurado [Pendiente de confirmar con el cliente] Responde con faq |
| Encuesta de satisfacción | Encuesta de Satisfacción automática global. | Activo |
| Branches | Flujo nativo de sucursales/locales. | activo |

### Flujos de Pre-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
|---|---|---|
| Mayorista | El usuario quiere realizar una compra mayorista o al por mayor ("venta mayorista", "comprar por mayor", "quiero revender"). | Asigna la conversación al sector **mayorista** y responde con la FAQ de venta mayorista (id 6b9883a8-06b6-487e-a254-15c79b8cc781). Indica que se comunique al número de venta mayorista o complete el formulario "Sumá tu local". En ningún caso consulta ni sugiere categorías o productos. |

### Flujos de Post-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
|---|---|---|
| Cambio | "Quiero hacer un cambio", "cambiar producto", consultas sobre condiciones de cambio. | Saluda amablemente e indica las condiciones de cambio [Pendiente de confirmar con el cliente]. Asigna el sector **postsales** y genera un ticket con motivo **exchanges&returns**, solicitando:<br>• Número de orden<br>• Prenda a cambiar<br>• Motivo del cambio |
| Devolución | "Devolver producto", "cómo devuelvo un producto", "políticas de devolución" — sobre un pedido ya recibido. | Asigna el sector **postsales** y genera un ticket con motivo **devolutions**, solicitando:<br>• Número de orden<br>• Prenda a devolver o pedido completo<br>• Motivo — el agente lo clasifica internamente, sin decírselo al cliente, en: arrepentimiento, talla o insatisfacción<br>• Dirección de retiro |

### Flujos de Sistema (Automatizados)

| Flujo | Activadores | Necesidad / Comportamiento esperado |
|---|---|---|
| Mensaje de Bienvenida | Sin activador por mensaje del usuario — se dispara desde automatizaciones al iniciar el primer contacto. | Enviar un mensaje fijo definido por el cliente. [Pendiente de confirmar con el cliente] |
| Seguimiento Proactivo | Sin activador por mensaje del usuario — automático tras un tiempo configurable de inactividad. | No configurado actualmente. [Pendiente de confirmar con el cliente] |
| Cierre por Inactividad + CSAT | Sin activador por mensaje del usuario — automático al quedar la conversación inactiva. | No configurado actualmente (la encuesta global no está activa). [Pendiente de confirmar con el cliente] |

## Flujos con Lógica Condicional

Los siguientes flujos tienen ramificaciones internas a documentar en un diagrama de decisión:

- **Cambio / Devolución:** el agente debe distinguir si el usuario quiere cambiar (exchanges&returns) o devolver (devolutions); en ambos casos recolecta los campos del ticket y deriva a postsales. Ramifica según si el usuario ya brindó número de orden y datos de contacto.
- **Derivación a asesor:** siempre requiere ticket; luego de informar la derivación y el horario (días hábiles 9 a 17 hs) el agente deja de intervenir.
- **Mayorista:** respuesta fija vía FAQ, sin ramificaciones de catálogo.
- **Fuera de alcance:** cualquier consulta ajena a la indumentaria deportiva de la marca se rechaza indicando que no puede ayudar.

Cualquier comportamiento no cubierto explícitamente en este documento debe confirmarse con el referente técnico del cliente antes de la implementación.

## Checklist de Configuración por Cliente

Puntos a completar y validar con Admit One:

- Políticas de cambios y devoluciones (plazos, condiciones, excepciones por categoría/promoción) — el flujo de Cambio las menciona pero no están cargadas.
- Número de venta mayorista y link al formulario "Sumá tu local"; validar contenido de la FAQ mayorista.
- Política de garantía (plazo de cobertura, condiciones de aceptación).
- Transportistas / métodos de envío, zonas de cobertura y tiempos estimados.
- Medios de pago y condiciones de cuotas.
- Plataforma de e-commerce e integración para Estado de Pedido (hoy sin flujo; los estados deben mostrarse en español).
- ticketReasons (exchanges&returns, devolutions) y sectores (postsales, mayorista) configurados en el HelpDesk / CRM.
- Contenido de la Base de Conocimientos: guía de talles, envíos, promociones y otras consultas frecuentes.
- Mensajes fijos (bienvenida, cierre, seguimiento proactivo) y activación de Promociones, Facturación, Encuesta y Branches.
- Nombre y datos de contacto del referente del cliente.
