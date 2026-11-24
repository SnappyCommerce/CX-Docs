> Fuente: Alcance_Funcional_Instituto_Tezza (https://docs.google.com/document/d/1gGYC426DAE6r4GKdb0vSmTD95S2g3h8CXg9Yq1DxDfg/edit?usp=drivesdk&ouid=113568244751416058438)

**Snappy Commerce**

# Alcance Funcional Instituto Tezza

## Cómo usar este documento

Este documento describe la estructura estándar de flujos de un Agente IA de atención al cliente y ventas de Snappy Commerce. Sirve como punto de partida para el relevamiento (discovery) de un cliente nuevo: cada fila define el flujo, sus activadores típicos y el comportamiento esperado en términos generales.

Los textos entre corchetes y las notas marcadas con ⚠ señalan puntos que dependen de la política, integración o contenido propio de cada cliente y deben completarse durante el discovery. Los nombres de los flujos pueden mantenerse o adaptarse a la nomenclatura interna del cliente.

# Requerimientos Funcionales

## Configuración Global

| Flujo nativo | Qué hace el flujo | Valor configurado |
| :-: | :-: | :-: |
| Crear contacto - Campos a solicitar al usuario | Para crear un contacto nuevo, el bot pide teléfono y email. | Nombre, teléfono, mail (optativos) |
| Comenzar checkout - Requisitos de información de contacto | Para generar un checkout, el default pide nombre, apellido y email. | DESACTIVADO |
| Asistencia | Siempre que se dispara asistencia, el bot genera un ticket de soporte primero (pidiendo los datos que falten). Si además es horario de atención, después de crear el ticket la conversación se escala en vivo a un asesor. Fuera de horario, queda solo con el ticket creado, a la espera de gestión. | Contacto: Siempre<br>Escalar: Dentro de horario<br>Ticket: Fuera de horario |
| Encuesta de satisfacción | La Encuesta de Satisfacción automática global está activado | true |
| Estado de Pedido | El flujo default de consulta de estado de pedido es el mismo para todos los clientes. Los datos disponibles (tracking, fecha estimada, etc.) dependen de lo que devuelva la integración del cliente. | DESACTIVADO |
| Inicio de todas las conversaciones | Se activa al comienzo de todas las conversaciones, luego del primer mensaje del usuario. | DESACTIVADO |

## Flujos de Pre-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Pre inscripción | Usuario quiere INSCRIBIRSE o PREINSCRIBIRSE concretamente.<br>Ejemplos de activación:<br>- quiero inscribirme<br>- ¿cómo me anoto?<br>- quiero anotarme<br>- ¿cómo hago para inscribirme?<br>- quiero preinscribirme | Debe solicitar asistencia dentro de horario y crear un caso fuera de horario de atención. |
| Búsqueda y Recomendación de Producto | Usuario pregunta por un producto, categoría, talle, color o atributo ("tenés zapatillas rojas talle 40?"). | DESACTIVADO |
| Consulta de Precio / Stock | Usuario pregunta precio, disponibilidad o stock de un producto puntual. | DESACTIVADO |
| Promociones | Usuario pregunta por descuentos, promos, cupones o "qué ofertas tienen". | DESACTIVADO |
| Información de Sucursales / Branches | Usuario pregunta dirección, horarios o sucursal más cercana. | DESACTIVADO |
| FAQ Comercial (talles, envíos, medios de pago) | Usuario consulta guía de talles, costos/tiempos de envío, medios de pago o cuotas. | Responde con el contenido de la Base de Conocimientos (FAQ) configurada para cada tema.<br>Información cargada en faqs diferentes: Metodo de envio (bs as y cordoba), Metodo de envio (resto del país), etc<br>El bot usará el dato consultado al comienzo de la conversación para definir con qué información responder. |
| Checkout Asistido | Usuario quiere comprar / pide el link de compra ("quiero finalizar la compra", "pasame el link"). | DESACTIVADO |
| Checkout Asistido — Gestión del Carrito | Usuario quiere agregar, quitar o modificar productos en un carrito ya iniciado. | DESACTIVADO |
| Derivación a Vendedor / Asistencia comercial | Usuario pide hablar con un asesor de ventas o la consulta excede el alcance del bot. | Dispara el flujo de asistencia: pide datos de contacto y escala en vivo o genera ticket según horario de atención. |

## Flujos de Post-Venta

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Estado de Pedido | Usuario pregunta por el estado de su pedido/envío | DESACTIVADO |
| Cambios Devoluciones, Cancelaciones | Usuario quiere cambiar o devolver un producto ya comprado. | DESACTIVADO |
| Garantía | Usuario por la garantía de un producto. | DESACTIVADO |
| Reclamos / Ticket | Usuario reporta un problema (producto dañado, faltante, error de facturación, etc.). | DESACTIVADO |
| Encuesta de Satisfacción (CSAT) | Se dispara automáticamente al cierre de una gestión o conversación. | Envía la encuesta de satisfacción configurada de forma global (flujo nativo activado). |

## Flujos de Sistema (Automatizados)

| Flujo | Activadores | Necesidad / Comportamiento esperado |
| :-: | :-: | :-: |
| Mensaje de Bienvenida | Sin activador por mensaje del usuario — se dispara desde automatizaciones al iniciar el primer contacto. | DESACTIVADO |
| Seguimiento Proactivo | Sin activador por mensaje del usuario — automático tras un tiempo configurable de inactividad. | DESACTIVADO |
| Cierre por Inactividad + CSAT | Sin activador por mensaje del usuario — automático al quedar la conversación inactiva. | DESACTIVADO |
| Notificaciones / Broadcast | Se dispara desde un envío de notificación/broadcast programado en Labs. | DESACTIVADO |

# Requerimientos No Funcionales

| Categoría | Requerimiento |
| :-: | :-: |
| Lenguaje | Español rioplatense, tono profesional pero cercano, ameno y cordial. |
| Testing | El código/JSON debe pasar el test suite estándar de Snappy (umbral 85%+) antes de cualquier subida a producción. |
| Conversaciones | El agente debe contar con contexto de las conversaciones previas de cada contacto y poder responder usando esa información (ej.: reconocer a qué ticket se refiere el usuario en una conversación nueva). |
| Comportamiento — Fallback | Cuando no cuente con información suficiente en Conocimientos, debe ofrecer solicitar asistencia de un operador humano. |
| Comportamiento — Proactividad y reencauce | Ante silencio de 5 minutos en un flujo de venta en curso, reactiva la conversación con un mensaje de seguimiento. Ante desvío de tema, responde la consulta puntual y luego reencauza hacia la venta en curso, sin abandonarla. |

## Flujos con Lógica Condicional

Los siguientes flujos tienen ramificaciones internas y requieren un diagrama de decisión propio, construido durante el discovery de cada cliente. Se listan aquí los puntos de ramificación genéricos que ese diagrama debe cubrir:

- **Sin flujos con lógica condicional**

*⚠ Cualquier comportamiento no cubierto explícitamente en este documento debe confirmarse con el referente técnico del cliente antes de la implementación.*

# Diagramas de Flujo

Representación visual de la lógica de decisión de los flujos condicionales listados arriba. Son diagramas base para discovery: los rótulos entre corchetes o marcados con ⚠ en las tablas anteriores deben reflejarse aquí una vez definidos con el cliente.

- **Sin flujos con lógica condicional**

## Checklist de Configuración por Cliente

Antes de dar por cerrado el alcance funcional de un cliente nuevo, completar y validar:

- ticketReasons y sectores de derivación configurados en el HelpDesk / CRM del cliente.
- Contenido de la Base de Conocimientos: FAQs de carreras/cursos, pre inscripciones, y otras consultas frecuentes.

# Test

**TEST**

| Flujo | Usuario: | Bot: |
| :-: | :-: | :-: |
| Enfermería | ¿Cuándo abren las inscripciones de la carrera de la licenciatura en enfermería? | Hola! Te comentamos que nuestras carreras son **tecnicaturas**, en este caso las inscripciones de la carrera de enfermería comienzan en X |
| Psicomotricidad | La carrera de psicomotricidad es una licenciatura? | Hola! Todas nuestras carreras son tecnicaturas. |
| Preinscripción | ¿Cómo me inscribo? | Para poder ayudarte con la inscripción, ¿me podés dejar tu nombre, teléfono y mail para que un asesor se contacte con vos? |
| Preinscripción | Quiero anotarme a Enfermería | ¡Perfecto! Para transferirte con un asesor necesito que me pases tu nombre, tu teléfono y tu mail. |
| Inscripciones | ¿Cuándo abren las inscripciones de nuevo? | Las inscripciones abren en octubre. Podés comenzar tu carrera en Marzo 2027. |
| Inscripciones | ¿Qué papeles tengo que llevar para inscribirme? | Necesitás: DNI original y copia (frente y dorso), Título Secundario legalizado o Constancia en Trámite, Carnet de Vacunación (Hepatitis B y Antitetánica) y foto 4x4. |
| Inscripciones | ¿Hay examen de ingreso? | No, no hay examen ni curso de ingreso. Ingresás directamente al primer año de tu carrera. |
| Costos | ¿Cuánto sale la cuota de Enfermería? | La cuota mensual de Enfermería es de aproximadamente $96.000. Son 10 cuotas al año; en enero y febrero no abonás. |
| Costos | ¿Cuánto cuesta la matrícula de Psicomotricidad? | La matrícula anual de Psicomotricidad es de aproximadamente $180.000, se abona una sola vez por año. |
| Costos | ¿Hay que pagar algo aparte de la cuota? | No, la cuota incluye todo: permisos de examen, recuperatorios, exámenes finales y reincorporaciones. |
| Horarios | ¿Cuándo se cursa Psicomotricidad? | Psicomotricidad se cursa los sábados de 8 a 18hs, de forma presencial en el instituto. |
| Horarios | ¿Enfermería tiene clases online? | Sí, tiene modalidad mixta. Se cursa de lunes a viernes de 16:30 a 21:30hs: 2 días virtuales y 3 presenciales. |
| Horarios | ¿En qué horario se cursa Laboratorio? | Podés cursar 4 días a la semana de 7:30 a 13hs, o los sábados de forma intensiva de 7:30 a 21hs aprox., con un día de prácticas entre semana. |
| Prácticas | ¿Desde qué año tengo prácticas en el hospital? | Desde el primer año tenés prácticas profesionalizantes, ya sea teóricas, en gabinete de simulación o prácticas hospitalarias, según tu carrera. |
| Prácticas | ¿En qué hospitales hacen prácticas de Instrumentación Quirúrgica? | Tienen convenio con todos los hospitales públicos de Capital Federal y con la Clínica San Camilo en el ámbito privado. |
| Equivalencias | Hice materias de Enfermería en otra institución, ¿me las reconocen? | Sí, aceptamos equivalencias de materias realizadas en otras instituciones según resolución del plan de estudios. |
| Equivalencias | ¿Me puedo pasar a Tezza desde otra institución? | Sí, aceptamos pases sin problema, según resolución del plan de estudios. |
| Equivalencias | ¿Puedo cambiarme de carrera? | Sí, podés cambiarte y tomamos tus materias aprobadas como equivalencias, pero cada caso se evalúa de manera particular. |
| Sin secundario | No terminé el secundario, ¿puedo estudiar igual? | Sí, podés ingresar mediante el Artículo 7 si tenés más de 25 años, estudios primarios completos y realizás una entrevista con la Rectora. |
| Título | ¿El título sirve para trabajar en hospitales privados? | Sí, el título es oficial de validez nacional y te avala para ejercer en todo el país, tanto en el ámbito público como privado. |
| Materias libres | ¿Puedo rendir una materia sin haberla cursado? | Sí, podés rendir materias libres sin cursar, pero tenés que asesorarte con la coordinadora de tu carrera. |
| Charlas | ¿Puedo hablar con alguien antes de inscribirme? | ¡Claro! Realizamos charlas informativas online y presenciales, gratuitas, con un representante de la carrera. Agendá en institutotezza.edu.ar/carrera/ |
| Constancia | ¿Cómo pido la constancia de alumno regular? | Podés solicitarla en la plataforma Insignia: ingresá con tu usuario, andá al menú, seleccioná "Constancia de Alumno Regular y Transporte" y completá los datos. |
| Contacto | ¿A qué mail le escribo para el título en trámite? | Te recomendamos escribir a secretaria@institutotezza.edu.ar, donde te van a asesorar correctamente. |
| Ubicación | ¿Dónde queda el instituto? | El instituto está en Antonio Machado 628, CABA. También podés hacer la preinscripción online. |
| Cursos | ¿Tienen cursos cortos o de actualización? | Sí, tenemos Microcredenciales: cursos de actualización académica con validez del Gobierno de CABA y certificado. |
