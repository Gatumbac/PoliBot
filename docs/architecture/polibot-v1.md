# Arquitectura de PoliBot v1

## Objetivo

PoliBot responde consultas académicas por Telegram con datos de Canvas. La v1
prioriza datos confiables y operación observable sobre autonomía total.

## Diseño modular

```text
Telegram
  -> wf_telegram_router
       -> wf_cmd_static
       -> wf_cmd_canvas_queries -> Canvas REST API
  -> Telegram Reply
```

El router es el único workflow que recibe mensajes externos y envía respuestas.
Los sub-workflows no se publican: reciben un contrato interno y devuelven una
respuesta para que el router la entregue.

```json
{ "chatId": "123", "command": "/semana", "text": "..." }
```

## Rutas actuales

| Entrada | Destino | Resultado |
| --- | --- | --- |
| `hola`, saludos, `/start` | `wf_cmd_static` | Presentación |
| `/ayuda`, `/estado` | `wf_cmd_static` | Respuesta fija |
| `/hoy`, `/semana`, `/proximas` | `wf_cmd_canvas_queries` | Tareas de Canvas |
| Cualquier otra entrada | Router | Solicita usar `/ayuda` |

Canvas se consulta con credenciales de n8n, nunca con tokens en nodos Code. El
formateador usa `assignment.due_at || due_at`, calcula una prioridad determinista,
limita la salida a 10 tareas y presenta fechas en `America/Guayaquil`. La salida
incluye `priority`, `reason`, `total_count` y `displayed_count` para indicar la
urgencia y si quedaron tareas fuera del límite.

## Capa de IA futura

`wf_ai_intent_router` (`gxOo4vRoILK2jnEi`) implementa el router de intención
y está conectado a la ruta de lenguaje natural de `wf_telegram_router`. La IA
es un **router de intención**, no una fuente de datos ni un ejecutor directo de
acciones. Recibe lenguaje natural y devuelve JSON validable:

```json
{
  "intent": "canvas_tasks",
  "period": "week",
  "confidence": 0.96,
  "needs_clarification": false
}
```

El workflow valida la intención y llama una herramienta determinista. Si la
confianza es baja, pregunta al usuario; no inventa tareas ni accede a
credenciales. Las reglas actuales se conservan como ruta rápida para comandos
conocidos.

El contrato de salida solo permite `static`, `canvas` o `clarify`. Las rutas
`static` y `canvas` devuelven un comando canónico; `clarify` devuelve una
pregunta. Esto evita que una clasificación incompleta active Canvas.

## Guardrails y observabilidad

- Las respuestas académicas se generan desde resultados actuales de Canvas.
- Acciones con efecto externo, como crear eventos, requieren confirmación.
- Se registra intención, herramienta, resultado, error y duración en las
  ejecuciones de n8n.
- Se prueba el flujo completo por Telegram; pruebas de nodos aislados no
  sustituyen la validación end-to-end.
