# Estado de Implementación

**Fecha**: 2026-07-16  
**Entorno**: n8n personal, Telegram y Canvas LMS

## Workflows v1

| Workflow | ID | Responsabilidad | Estado |
| --- | --- | --- | --- |
| `wf_telegram_router` | `vAaVfPUfv6tcyMrV` | Recibe Telegram, clasifica y responde | Activo en producción |
| `wf_cmd_static` | `iSTRgndQX0SeWvmS` | `/start`, `/ayuda`, `/estado` | Sub-workflow |
| `wf_cmd_canvas_queries` | `ymE59cw5zVBWiktC` | `/hoy`, `/semana`, `/proximas` | Sub-workflow con Hito 1 publicado y cerrado |
| `wf_ai_intent_router` | `gxOo4vRoILK2jnEi` | Lenguaje natural a intención validada | Conectado al router |
| `PoliBot_Inicial` | `bJG32QZhycbFenCh` | Referencia del MVP anterior | Archivado |

Workflows archivados de limpieza operativa: `PoliBot_Inicial` y `Test`.

## Validado

- `hola` y saludos equivalentes se convierten en la intención `/start`.
- `/ayuda` devuelve el catálogo de comandos.
- Batería mínima E2E cerrada por webhook el 2026-07-15:
  - `/start` validado en la ejecución `304` y respondió con la presentación del bot.
  - `/estado` validado en la ejecución `306` y respondió con el estado operativo.
  - `/hoy` validado en la ejecución `308` y devolvió tareas reales de Canvas.
  - `/semana` validado en la ejecución `310` y devolvió tareas reales de Canvas.
  - Mensaje ambiguo validado en la ejecución `320` y respondió con una aclaración accionable.
- `/semana` obtiene tareas Canvas, prioriza `assignment.due_at` y usa
  `America/Guayaquil`.
- `wf_cmd_canvas_queries` maneja `429` de Canvas con reintento interno y
  mensaje controlado para Telegram; verificado con simulación en la ejecución
  `322`.
- Cobertura de lenguaje natural validada en el router de IA:
  - `¿Cómo uso el bot?` -> `static_help` -> `/ayuda` en la ejecución `366`.
  - `¿Sigues funcionando?` -> `static_status` -> `/estado` en la ejecución `367`.
  - `¿Qué tengo para próximas semanas?` -> `canvas_tasks/upcoming` -> `/proximas` en la ejecución `368`.
  - `¿Qué tengo que entregar hoy o esta semana?` -> aclaración accionable en la ejecución `369`.
- Variaciones coloquiales adicionales validadas en el router de IA:
  - `¿Qué me toca hoy?` -> `canvas_tasks/today` -> `/hoy` en la ejecución `381`.
  - `¿Qué tengo pendiente esta semana?` -> `canvas_tasks/week` -> `/semana` en la ejecución `382`.
  - `¿Qué viene para la próxima semana?` -> `canvas_tasks/week` -> `/semana` en la ejecución `383`.
  - `¿Qué tengo que entregar hoy o esta semana?` -> aclaración accionable en la ejecución `384`.
- Frases coloquiales adicionales validadas en el router de IA:
  - `¿Qué me queda?` -> `canvas_tasks/upcoming` -> `/proximas` en la ejecución `385`.
  - `¿Qué tengo para después?` -> `canvas_tasks/upcoming` -> `/proximas` en la ejecución `386`.
  - `¿Qué entrego?` -> `canvas_tasks/upcoming` -> `/proximas` en la ejecución `387`.
  - `¿Qué tareas me faltan?` -> `canvas_tasks/upcoming` -> `/proximas` en la ejecución `388`.
- Hito 1 implementado en `wf_cmd_canvas_queries` y verificado con datos
  sanitizados fijados en n8n: `/semana` en la ejecución `391`, `/hoy` en `392` y
  `/proximas` en `393`. Las pruebas cubren orden crítico, alta, media y baja;
  tareas sin fecha visibles al final; exclusión de una tarea marcada como
  completada; conteos `received_count`, `discarded_count`, `total_count` y
  `displayed_count`; límite de 10; y fechas en `America/Guayaquil`.
- El Hito 1 quedó publicado en `wf_cmd_canvas_queries` y validado desde
  Telegram con el formato priorizado visible para el usuario final.
- El formato de tareas conserva el encabezado y marcador horario de
  `PoliBot_Inicial`.
- Las respuestas de error de Canvas evitan exponer detalles técnicos al usuario.
- `Telegram Reply` está configurado explícitamente para enviar mensajes.
- El router de IA transforma una intención simulada `canvas_tasks/week` en
  `route: canvas` y `/semana`.
- Una prueba real de Gemini clasificó “¿Qué tareas tengo que entregar esta
  semana?” y el router devolvió las tareas Canvas correspondientes.
- `/proximas` fue validado en webhook (ejecución `227`) y devolvió tareas
  reales de Canvas con fechas en formato local.
- `/ayuda` fue validado en webhook (ejecución `223`) y devolvió el catálogo de
  comandos esperado.
- Un saludo natural (`hola.`) fue validado en webhook (ejecución `220`), la IA
  lo clasificó como `static_start` y se respondió por `/start`.

## Operación

`wf_telegram_router` ya está activo en producción (`active: true`) y recibe
mensajes por webhook en modo `webhook`. Telegram envía cada mensaje al router,
que delega a sub-workflows y responde por `Telegram Reply`. Mantenga
`PoliBot_Inicial` archivado para evitar competir por el mismo bot.

## Pendiente inmediato

1. Arrancar el Hito 2 de fase 3: definir la fuente de sílabos, el formato de
   ingesta y el contrato de citas.
2. Resolver la decisión de almacenamiento para la memoria breve por `chatId`
   antes de integrar más rutas académicas.
