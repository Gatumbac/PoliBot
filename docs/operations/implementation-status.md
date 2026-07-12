# Estado de Implementación

**Fecha**: 2026-07-12  
**Entorno**: n8n personal, Telegram y Canvas LMS

## Workflows v1

| Workflow | ID | Responsabilidad | Estado |
| --- | --- | --- | --- |
| `wf_telegram_router` | `vAaVfPUfv6tcyMrV` | Recibe Telegram, clasifica y responde | Pendiente de publicar |
| `wf_cmd_static` | `iSTRgndQX0SeWvmS` | `/start`, `/ayuda`, `/estado` | Sub-workflow |
| `wf_cmd_canvas_queries` | `ymE59cw5zVBWiktC` | `/hoy`, `/semana`, `/proximas` | Sub-workflow |
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

## Operación

Para producción, publique solo `wf_telegram_router`. Telegram enviará cada
mensaje al webhook de n8n y el router invocará los sub-workflows. Mantenga
`PoliBot_Inicial` inactivo para evitar competir por el mismo bot.

## Pendiente inmediato

1. Publicar `wf_telegram_router`.
2. Probar desde Telegram `/start`, `/ayuda`, `/hoy`, `/semana`, `/proximas` y
   un mensaje no reconocido.
3. Revisar las ejecuciones de n8n y confirmar el fallback de Canvas.
