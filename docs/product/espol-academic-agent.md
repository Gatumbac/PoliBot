# ESPOL Academic Intelligence Agent

> Documento de visión y alcance del producto. La arquitectura operativa actual
> se documenta en [`docs/architecture/polibot-v1.md`](../architecture/polibot-v1.md).
### Proyecto para la Semana CIAP 2026 — Club de Inteligencia Artificial ESPOL

---

## Resumen Ejecutivo

El ESPOL Academic Intelligence Agent es un asistente académico personal construido sobre la API REST de Canvas LMS (el Aula Virtual de ESPOL), orquestado con n8n self-hosted y accesible via Telegram. El proyecto resuelve un problema real y documentado: Canvas notifica a los estudiantes con información mínima y sin contexto, los obliga a revisar múltiples plataformas de forma manual, y no ofrece ningún tipo de inteligencia sobre su situación académica.

El diferenciador central es que este agente convierte datos crudos de Canvas en **inteligencia accionable**: no solo informa, sino que alerta proactivamente, organiza el tiempo del estudiante y predice su rendimiento académico en tiempo real.

El costo total del proyecto es **$0** de suscripciones — todo el stack usa APIs gratuitas y hardware reutilizado.

---

## El Problema

### Pain Points Documentados de Estudiantes en ESPOL

Los estudiantes de ESPOL enfrentan una brecha significativa entre la información disponible en el Aula Virtual y su capacidad de actuar sobre ella:

- **Notificaciones sin contexto**: Canvas envía emails del tipo *"nueva tarea en Análisis de Algoritmos"* sin incluir fecha de vencimiento, peso en la nota, ni archivos adjuntos. El estudiante debe entrar manualmente al sistema para obtener esa información.
- **Tareas silenciosas**: Algunos profesores publican tareas o cambian fechas de exámenes sin usar el sistema de anuncios de Canvas, causando que estudiantes pierdan entregas o lleguen a exámenes con fechas incorrectas.
- **Fragmentación de información**: Para entender qué tiene que hacer un día, un estudiante debe revisar Canvas, su correo, WhatsApp de grupos de estudio, y su calendario por separado.
- **Ausencia de visibilidad del riesgo académico**: Canvas muestra notas individuales pero nunca calcula ni comunica el promedio ponderado real ni la nota mínima necesaria para aprobar una materia.
- **Falta de planificación estructurada**: Los estudiantes conocen los sílabos pero no los usan sistemáticamente para planificar su tiempo de estudio. No existe ninguna herramienta que conecte el contenido del sílabo con el calendario personal.
- **Contexto del estudiante trabajador**: Una porción significativa de politécnicos realiza pasantías o trabajos de tiempo parcial mientras estudia. Ninguna herramienta existente adapta la planificación académica a este perfil.

---

## Descubrimiento Técnico Clave

El Aula Virtual de ESPOL corre sobre **Canvas LMS de Instructure**, una plataforma con una **API REST completamente documentada y pública**. La autenticación se realiza mediante un Access Token que cualquier estudiante puede generar desde su perfil en Canvas (`Settings → Approved Integrations → New Access Token`), sin necesidad de permisos especiales ni intervención del departamento de TI de ESPOL.

La documentación completa de la API está disponible en:
```
https://aulavirtual.espol.edu.ec/doc/api/all_resources.html
```

Este descubrimiento es el fundamento técnico del proyecto: los datos académicos completos de un estudiante son programáticamente accesibles sin depender de que ESPOL implemente ningún sistema adicional.

**Validación:** El endpoint de cursos favoritos fue verificado en tiempo real retornando datos reales de matrícula activa:
```
GET https://aulavirtual.espol.edu.ec/api/v1/users/self/favorites/courses
Authorization: Bearer <token>
```

---

## Herramientas y Soluciones Existentes

Las soluciones actuales de IA para Canvas LMS tienen un gap crítico: todas son **institucionales**, lo que significa que requieren que la universidad pague una licencia y las implemente. Si ESPOL no las contrata, los estudiantes no tienen acceso.

| Herramienta | Modelo | Limitación principal |
|---|---|---|
| LearnWise AI | Licencia institucional | ESPOL debe contratarla |
| CampusMind AI | Licencia institucional | ESPOL debe contratarla |
| Edu Connect (App) | Freemium / individual | Solo reactivo, sin planificación ni alertas proactivas |
| ibl.ai | Institucional | Orientado a profesores, no estudiantes |
| Canvas IgniteAI (nativo) | Add-on de pago para ESPOL | ESPOL no lo ha activado |

**El ESPOL Academic Intelligence Agent opera de forma completamente independiente** — el estudiante genera su propio token y el agente funciona sin ninguna acción por parte de la institución.

---

## Arquitectura del Sistema

### Stack Tecnológico

| Componente | Tecnología | Costo |
|---|---|---|
| Orquestador de workflows | n8n Community Edition (self-hosted) | $0 |
| Interfaz de usuario | Telegram Bot API | $0 |
| Fuente de datos académicos | Canvas REST API (ESPOL) | $0 |
| Modelo de lenguaje | Google Gemini 2.5 Flash (API gratuita) | $0 |
| Base de datos del agente | PostgreSQL (via Docker) | $0 |
| Memoria vectorial (RAG) | Qdrant (self-hosted) | $0 |
| Calendario | Google Calendar API | $0 |
| Panel visual | Notion API | $0 |
| Infraestructura | Lenovo AIO 520 reutilizado (Ubuntu Server) | $0 |
| **Total mensual** | | **~$3-5 (electricidad)** |

### Diagrama de Flujo

```
┌─────────────────────────────────────────────────────────────┐
│                    FUENTES DE DATOS                         │
│                                                             │
│  Canvas API ESPOL    Google Calendar    Sílabos (PDF/HTML)  │
│  (tareas, notas,     (tiempo libre,     (temas, horas       │
│   archivos, anuncios) eventos personales) recomendadas)     │
└─────────────────┬───────────────┬───────────────────────────┘
                  │               │
                  ▼               ▼
┌─────────────────────────────────────────────────────────────┐
│                    n8n (Orquestador)                        │
│                                                             │
│  Workflows programados:        Workflows reactivos:         │
│  • Cada 30min → monitor        • Usuario pregunta           │
│  • Cada mañana → briefing        algo por Telegram          │
│  • Cada domingo → plan semanal • Nuevo mensaje recibido     │
│  • Inmediato → deadline <24h                                │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
              ┌───────────────────────┐
              │   Gemini 2.5 Flash    │
              │   (cerebro del agente)│
              │   • Priorización      │
              │   • RAG sobre sílabos │
              │   • Nota proyectada   │
              │   • Plan de estudio   │
              └───────────┬───────────┘
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
   Telegram          Google Calendar     Notion
   (alertas y        (bloques de         (panel
    conversación)     estudio)            académico)
```

---

## Funcionalidades del Agente

### Capa 1 — Informar (Reactivo)

El agente responde preguntas en lenguaje natural sobre la vida académica del estudiante:

- *"¿Qué tareas tengo pendientes esta semana?"* → lista ordenada por fecha y peso en nota con links directos
- *"¿Cuánto tengo en Redes?"* → nota actual + nota proyectada + lo que necesita en el final para pasar
- *"¿Qué dijo el profe de BD en sus últimos anuncios?"* → resumen inteligente de los últimos anuncios
- *"Dame los PDFs de Compiladores de esta semana"* → descarga y envía archivos directamente por Telegram
- *"¿Cuándo son mis exámenes finales?"* → lista de fechas extraída del calendario de Canvas

**Endpoints Canvas utilizados:**
```
GET /api/v1/users/self/todo
GET /api/v1/courses/:id/assignments
GET /api/v1/courses/:id/enrollments?user_id=self
GET /api/v1/announcements?context_codes[]=course_:id
GET /api/v1/courses/:id/files
GET /api/v1/calendar_events
```

### Capa 2 — Alertar (Proactivo)

Workflows programados que corren en n8n sin que el usuario tenga que preguntar:

**Monitor de novedades (cada 30 minutos):**
Detecta cualquier cambio en el Aula Virtual via `activity_stream` y genera alertas con contexto completo. A diferencia del email de Canvas, incluye:
- Nombre y fecha de la tarea
- Peso exacto en la nota final
- Tiempo restante hasta el vencimiento
- Impacto en la nota proyectada si se entrega vs. si no se entrega
- Links directos a la tarea y archivos adjuntos

**Ejemplo de notificación generada:**
```
🚨 Nueva tarea detectada — Análisis de Algoritmos

📌 Tarea 4 - Árboles AVL
⏰ Vence: Domingo 11 May a las 23:59 (4 días)
💯 Vale: 15% de tu nota final
📊 Tu nota actual: 7.8 → si sacas >8 aquí subes a 8.1
📎 enunciado_tarea4.pdf → [abrir]
```

**Recordatorios inteligentes de deadline:**
- 72 horas antes: primer recordatorio con contexto completo
- 24 horas antes: recordatorio urgente con prioridad relativa
- 2 horas antes: alerta final

Si la tarea ya fue entregada, los recordatorios se cancelan automáticamente.

**Detector de cambios críticos en anuncios:**
Gemini analiza el texto de nuevos anuncios buscando palabras y patrones que indiquen cambios en fechas o condiciones de evaluación. Cuando detecta algo como *"el examen se mueve al..."*, genera una alerta inmediata y actualiza el evento en Google Calendar del estudiante.

### Capa 3 — Organizar (Inteligente)

La capa más diferenciadora — convierte al agente en un asesor académico activo:

**Cálculo de nota proyectada:**
```
Para cada materia:
  1. GET /api/v1/courses/:id/assignment_groups  → pesos de cada grupo
  2. GET /api/v1/courses/:id/submissions        → notas actuales
  3. Calcula promedio ponderado real
  4. Simula escenarios: ¿qué nota necesito en el final para pasar?
```

**Detector de riesgo académico:**
Alerta automática cuando la nota proyectada cae bajo 7.0, especificando exactamente qué nota mínima necesita el estudiante en las evaluaciones restantes.

**Generador de plan de estudio semanal (cada domingo):**
Combina tres fuentes de datos:
1. Sílabo de cada materia (parseado por Gemini desde `GET /api/v1/courses/:id?include[]=syllabus_body`) → temas pendientes y horas recomendadas
2. Canvas → fechas de exámenes y tareas próximas
3. Google Calendar → tiempo disponible del estudiante

Genera bloques de estudio reales y los agrega al Google Calendar con un solo click de confirmación en Telegram.

**Modo estudiante-trabajador:**
El agente considera el horario de pasantía o trabajo en la planificación. Genera sesiones cortas de máximo 1.5 horas optimizadas para tiempo limitado, priorizando materias en riesgo.

**Modo emergencia:**
Cuando detecta saturación (más de 4 deadlines en 48 horas o nota proyectada bajando en múltiples materias), activa un modo de gestión de crisis: sugiere qué entregar mínimo viable y en qué orden para maximizar la nota final con el tiempo disponible.

**Parseador de retroalimentación de profes:**
```
GET /api/v1/courses/:id/assignments/:id/submissions/self
  ?include[]=submission_comments
```
Cuando un profesor califica una tarea y deja comentarios, el agente los detecta y los envía por Telegram con contexto de impacto en la nota. Analiza patrones acumulados de retroalimentación para identificar áreas de mejora recurrentes.

---

## Integraciones Externas

| Integración | Propósito | Factor diferenciador |
|---|---|---|
| **Telegram Bot API** | Interfaz principal de usuario | Gratuita, sin fricción de onboarding |
| **Google Calendar** | Creación automática de bloques de estudio | Bidireccional: lee disponibilidad y escribe eventos |
| **Notion API** | Panel académico visual por materia | Genera páginas con sílabo, tareas y notas automáticamente |
| **Google Gemini 2.5 Flash** | Cerebro del agente, NLP, RAG | Tier gratuito suficiente para uso personal (250 req/día) |
| **Qdrant (self-hosted)** | Memoria vectorial para RAG | Indexa PDFs y materiales para responder preguntas sobre contenido |

---

## Costo Total del Proyecto

Todos los componentes utilizan tiers gratuitos o hardware reutilizado:

| Componente | Costo mensual |
|---|---|
| n8n Community Edition | $0 |
| Canvas API de ESPOL | $0 |
| Telegram Bot API | $0 |
| Google Gemini API (tier gratuito) | $0 |
| Google Calendar API | $0 |
| Notion API | $0 |
| Servidor (Lenovo AIO 520 reutilizado) | $0 |
| Electricidad (45W × 24h × 30 días) | ~$3-5/mes |
| **Total** | **~$3-5/mes** |

---

## Roadmap de Implementación

### Semana 1 — Infraestructura y MVP
- [ ] Instalar Ubuntu Server en Lenovo AIO 520
- [ ] Levantar n8n + PostgreSQL con Docker Compose
- [ ] Crear Telegram Bot via `@BotFather`
- [ ] Conectar Canvas API con token de estudiante
- [ ] Primer workflow: responder *"¿qué tareas tengo?"*

### Semana 2 — Alertas Proactivas
- [ ] Monitor de novedades cada 30 minutos
- [ ] Notificaciones contextuales de nuevas tareas
- [ ] Detector de cambios críticos en anuncios
- [ ] Recordatorios de deadline (72h / 24h / 2h)

### Semana 3 — Inteligencia Académica
- [ ] Cálculo de nota proyectada con assignment groups
- [ ] Detector de riesgo académico
- [ ] Integración con Google Calendar
- [ ] Modo emergencia por saturación

### Semana 4 — Organización y Pulido
- [ ] Parseador de sílabos con Gemini
- [ ] Generador de plan de estudio semanal
- [ ] Panel en Notion
- [ ] Exposición pública via Cloudflare Tunnel para la demo
- [ ] Preparación de presentación para Semana CIAP 2026

---

## Estrategia de Demo para la Semana CIAP

El agente se ejecuta en el Lenovo desde casa, expuesto via Cloudflare Tunnel. Durante la presentación:

1. **Demostración en vivo**: preguntar al agente desde el celular frente al jurado con datos académicos reales del presentador
2. **El momento "wow"**: mostrar cómo detecta una tarea recién publicada en Canvas y genera la alerta completa en menos de 30 minutos
3. **El argumento técnico**: mostrar la arquitectura — Canvas API + n8n + Gemini + Telegram — explicando cada componente y por qué se eligió
4. **El argumento de impacto**: comparar la notificación de Canvas con la notificación del agente side-by-side

### Criterios del Jurado vs. Proyecto

| Criterio | Por qué este proyecto gana |
|---|---|
| **Complejidad técnica** | Canvas API + orquestación n8n + múltiples integraciones + LLM + RAG |
| **Impacto y utilidad** | Resuelve problemas reales que todos los politécnicos experimentan |
| **Calidad de presentación** | Demo en vivo con datos reales, imposible de fingir |

---

## Potencial de Escalabilidad (Post-CIAP)

El proyecto tiene un camino natural hacia un producto comercial:

- **Expansión a toda ESPOL**: con OAuth2 institucional (requiere Developer Key de TI de ESPOL), cualquier estudiante podría conectar su cuenta sin generar tokens manualmente
- **Expansión regional**: Canvas es utilizado por múltiples universidades en Ecuador (UEES, UCSG, UCG) y en toda Latinoamérica. Un solo producto sirve a múltiples instituciones
- **Integración con plataformas de bots empresariales**: el stack es compatible con plataformas como Jelou AI para llevar el agente a WhatsApp Business en un contexto institucional
- **Modelo de negocio**: SaaS mensual por institución, o licencia por número de estudiantes activos

El proof of concept construido para el CIAP es la primera versión de un producto que no existe en el mercado ecuatoriano.

---

*Documento generado el 1 de mayo de 2026 — Club CIAP, ESPOL*
