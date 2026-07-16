# Estado de Implementación

**Fecha**: 2026-07-14  
**Entorno**: n8n personal, Telegram y Canvas LMS

## Workflows v1

| Workflow | ID | Responsabilidad | Estado |
| --- | --- | --- | --- |
| `wf_telegram_router` | `vAaVfPUfv6tcyMrV` | Recibe Telegram, clasifica y responde | Activo en producción |
| `wf_cmd_static` | `iSTRgndQX0SeWvmS` | `/start`, `/ayuda`, `/estado` | Sub-workflow |
| `wf_cmd_canvas_queries` | `ymE59cw5zVBWiktC` | `/hoy`, `/semana`, `/proximas` | Sub-workflow |
| `wf_ai_intent_router` | `gxOo4vRoILK2jnEi` | Lenguaje natural a intención validada | Conectado al router |
| `PoliBot_Inicial` | `bJG32QZhycbFenCh` | Referencia del MVP anterior | Mantener inactivo |

## Validado

- `hola` y saludos equivalentes se convierten en la intención `/start`.
- `/ayuda` devuelve el catálogo de comandos.
- `/semana` obtiene tareas Canvas, prioriza `assignment.due_at` y usa
  `America/Guayaquil`.
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
`PoliBot_Inicial` inactivo para evitar competir por el mismo bot.

## Pendiente inmediato

1. Completar evidencia webhook de `/start`, `/estado`, `/hoy` y `/semana` en
   la misma ventana de validación para cerrar la batería mínima E2E.
2. Ejecutar pruebas de mensajes ambiguos/no reconocidos y confirmar que la
   respuesta de aclaración sea consistente y accionable.
3. Implementar manejo explícito de `429` y reintentos/backoff en
   `wf_cmd_canvas_queries` para robustez de producción.
4. Expandir validación de frases naturales para hoy, próximas, ayuda, estado y
   ambigüedad antes de agregar nuevas intenciones.
