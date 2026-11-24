> Fuente: Grúas San Blas.docx (https://drive.google.com/file/d/1SVu4NC_uDYODwDgJ967WQfC8qXzz_Un2/view?usp=drivesdk)

# Análisis Funcional y Alcance de Proyecto

Migración Gen 5 → Gen 6

| Campo | Valor |
| :-: | :-: |
| Cliente / Proyecto | Grúas San Blas — Blas |
| Versión del documento | v1.0 |
| Fecha | 4 de agosto de 2026 |
| Autor(es) | Snappy Commerce |
| Estado | Borrador — pendiente de revisión de Producto |
| Fuente técnica | JSON de configuración de flujos personalizados (v2) de producción |

# 1. Introducción

Este documento releva el comportamiento actual (Gen 5) del Agente IA Blas — flujos nativos, flujos personalizados y configuración global — como base para su eventual migración a Gen 6, donde estos flujos pasan a configurarse en código (TypeScript) del lado de Producto. El objetivo es documentar lo que existe hoy, y dejar constancia de solicitudes a implementar.

## 1.1 Propósito del documento

Servir de base para que Producto valide el comportamiento actual del Agente IA, detecte vacíos funcionales e inconsistencias de configuración, y tenga documentado cada flujo personalizado vigente antes de que pase de la UI de Snappy a código mantenido por Producto.

## 1.3 Glosario de términos

| Término | Definición |
| :-: | :-: |
| Agente IA | Agente conversacional de atención al cliente configurado sobre la plataforma Snappy Commerce. |
| Gen 5 | Versión actual de la plataforma: los flujos personalizados se configuran desde la UI. |
| Gen 6 | Nueva versión: los flujos personalizados pasan a configurarse en código (TypeScript), mantenido por Producto. La experiencia del usuario final no cambia. |
| Flujo nativo (Gen 5) | Funcionalidad pre-programada de Snappy (Búsqueda de Productos, Promociones, Sucursales, Seguimiento de Pedido, Encuesta de Satisfacción). |
| Flujo personalizado (Gen 5) | Flujo definido para procesos específicos que no cubre un flujo nativo. |
| Orchestrator | Motor que evalúa cada mensaje del usuario y decide qué flujo activar, en base a la descripción y ejemplos de cada activador. |
| sector | Etiqueta de conversación según departamentos para registro interno. |
| topic | Etiqueta de tópicos que clasifican la conversación según los temas tratados. |
| faq | Preguntas frecuentes referenciadas por ID. |
| función "awk" | Combina búsqueda en Conocimientos con un prompt de IA para generar respuesta dinámica. |
| función "notify" | Envía un mensaje fijo (isExactMessage) o semi-libre al usuario. |
| función "ticket" | Genera un caso/ticket con razón y campos adicionales, sin pasar necesariamente por humanAssistance. |
| función "assistance" | Dispara el proceso de asistencia humana definido en Comportamiento. |
| función "survey" | Dispara la Encuesta de Satisfacción como acción dentro de un flujo. |
| Conocimientos | Base de FAQs, conocimientos, promociones y sucursales cargadas en la plataforma. |
| Escalamiento | Transferencia a agente humano (en vivo dentro de horario, o ticket fuera de horario). |

# 2. Objetivos del Proyecto

## 2.1 Objetivo general

Dejar documentado el comportamiento actual del Agente IA Blas — flujos personalizados y configuración global — validado por Producto, como base para su migración a Gen 6.

## 2.2 Objetivos específicos

- Formalizar en un documento el flujo personalizado vigente en producción (1 flujo activo), para que Producto lo valide antes de migrarlo a código.
- Documentar la configuración global de la tienda para flujos nativos y personalizados.
- Identificar inconsistencias de configuración que no deben arrastrarse a la migración a Gen 6.
- Dejar una base de reglas de negocio documentada y trazable a su fuente.

# 3. Alcance del Proyecto

## 3.1 Funciones activas (In Scope)

Flujos personalizados activos (enabled: true):

- presupuesto — Crea un ticket de solicitud de presupuesto (quoteRequest), recolectando razón social y productos de interés.

Configuración global activa:

- createContact — Habilitado. Recolecta firstName, email, phone, y el campo adicional "Razón social". El equipo de Grúas San Blas solicitó que todos los contactos incluyan el campo de Razón social.

## 3.2 Funciones inactivas

No existen flujos personalizados desactivados (enabled: false) en esta versión del JSON.

Los flujos nativos de la plataforma (Búsqueda de Productos, Promociones, Sucursales, Seguimiento de Pedido, Factura) no tienen orchestratorOverwrites configurados — sus estados por defecto aplican. Dado que Grúas San Blas no opera sobre una plataforma de eCommerce estándar, se recomienda revisar cuáles deben desactivarse explícitamente antes del go-live (ver Sección 9, HV-02).

# 4. Actores del Sistema

| Actor | Descripción | Responsabilidades principales |
| :-: | :-: | :-: |
| Usuario final | Usuario que se comunica con Grúas San Blas a través de los canales integrados a Snappy. | Consulta sobre servicios, solicita presupuestos. |
| Agente IA | Agente conversacional Blas, configurado sobre Snappy Commerce. | Responde FAQs, ejecuta flujos, escala cuando corresponde. |
| Equipo de soporte | Equipo humano de atención al cliente de Grúas San Blas. | Recibe tickets y derivaciones en vivo. |
| CS Snappy | Equipo interno de Snappy Commerce. | En Gen 5, configura los flujos personalizados vía UI. |
| Producto | Equipo que en Gen 6 configura los flujos en código TypeScript. | Define y mantiene en TypeScript los flujos; valida este documento como base. |

# 5. Requerimientos Funcionales

## 5.1 Búsqueda de Productos y Configuración de Catálogo

Grúas San Blas no opera sobre una plataforma de eCommerce estándar. Los parámetros de búsqueda de productos y catálogo no están configurados en el JSON actual.

| Requerimiento | Parámetro | Valor configurado |
| :-: | :-: | :-: |
| Búsqueda de productos | orchestratorOverwrites.productSearch.enabled | Apagado |
| Incluir características en búsqueda | productSearch.includeProductCharacteristicsInQuery | Apagado |
| Disponibilidad en sucursal | orchestratorOverwrites.productBranches.enabled | Apagado |

## 5.2 Flujos Personalizados

Cada fila es un requerimiento funcional: el activador describe cuándo Blas debe usar el flujo (incluyendo negativas explícitas, cuando existen) y la columna de comportamiento describe paso a paso qué hace una vez activado, en el orden en que aparece en el JSON.

| Flujo | Activador (textual de Labs) | Comportamiento esperado |
| :-: | :-: | :-: |
| presupuesto | Activa este flow cuando el cliente esta interesado en averiguar un presupuesto. Vas a reconocer frases como quiero saber precios o averiguar presupuesto.<br>NO activar para consultas sobre el estado de un presupuesto ya enviado. | → sector → sales<br>→ ticket reason: quoteRequest / subreason: (vacío)<br>→ Campos adicionales: Razón Social, Productos |

## 5.3 Configuración Global

| Flujo nativo | Qué hace el flujo | Valor configurado |
| :-: | :-: | :-: |
| Crear contacto - Campos | Datos que solicita el agente para crear contacto | firstName, email, phone. Campo adicional: Razón social. |
| Comenzar checkout - Requisitos | Datos para generar checkout | Apagado |
| Agregar producto al carrito | Agrega productos al carrito | Apagado |
| Comenzar checkout | El agente IA podrá generar un link de checkout para el carrito del cliente. | Apagado |
| Mostrar carrito | El agente IA dispondrá de visibilidad sobre los artículos integrados en el carrito de compras del usuario. | Apagado |
| Eliminar producto del carrito | Permite eliminar productos del carrito | Apagado |
| Encuesta de satisfacción | Envía la encuesta de satisfacción | Encendido |
| (Advanced) Multiple prompt | Mostrar opciones como botones | No configurado (default) |
| (Advanced) Branches - Mode | Configuración de sucursales | No configurado (default) |
| (Advanced) Promotions - validToday | Filtro de promociones del día | No configurado (default) |

# 6. Requerimientos No Funcionales

| Categoría | Requerimiento |
| :-: | :-: |
| Lenguaje | Español argentino. |
| Testing | El código/JSON debe pasar el test suite estándar de Snappy (umbral 85%+) antes de cualquier subida a producción. |
| Migración Gen 6 | Cada flujo migrado a TypeScript debe reproducir el mismo comportamiento observado en Gen 5, salvo las inconsistencias identificadas en la sección 9. |
| Conversaciones | Blas debe contar con contexto de conversaciones previas de cada contacto. |

# 7. Integraciones

| Sistema / Canal | Propósito |
| :-: | :-: |
| Web (sitio de desarrollo propio) | Canal de lanzamiento. El cliente opera sobre un sitio propio sin plataforma de eCommerce estándar. |
| WhatsApp Business API | Canal no activo al momento del relevamiento. |
| Otros canales (Instagram, Facebook, etc.) | No activos al momento del relevamiento. |

# 8. Criterios de Aceptación

- El flujo presupuesto migrado a Gen 6 reproduce el comportamiento de Gen 5 documentado en 5.2, con las inconsistencias de la sección 9 ya corregidas.
- Blas pasa el test suite estándar de Snappy con un umbral de 85%+.
- Los flujos nativos no aplicables al modelo de negocio de Grúas San Blas (eCommerce, búsqueda de productos, seguimiento de pedido) están explícitamente desactivados antes del go-live.
- Incorporación de todos los requerimientos no funcionales detallados en el punto 6.

# 9. Hallazgos y Riesgos Identificados

No se encontraron riesgos a partir del análisis del JSON de producción.

# 10. Aprobaciones

| Rol | Nombre | Fecha |
| :-: | :-: | :-: |
| Producto |  |  |
| CS Owner (Snappy) | Sabrina | 04/08/2026 |
