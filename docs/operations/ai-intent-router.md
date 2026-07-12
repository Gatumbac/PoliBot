# Router de Intención con IA

## Propósito

El router permite que PoliBot entienda mensajes naturales sin obligar al usuario a conocer comandos. Solo clasifica la intención: no consulta Canvas ni envía mensajes a Telegram.

## Flujo

`Telegram -> wf_telegram_router -> wf_ai_intent_router -> workflow especializado -> respuesta`

Los comandos exactos (`/start`, `/ayuda`, `/estado`, `/hoy`, `/semana` y `/proximas`) omiten la IA y se enrutan de forma determinista. Los mensajes naturales pasan por `wf_ai_intent_router` (`gxOo4vRoILK2jnEi`), que responde JSON validado.

## Contrato de clasificación

| Intención | Período | Ruta resultante |
| --- | --- | --- |
| `canvas_tasks` | `today` | `/hoy` |
| `canvas_tasks` | `week` | `/semana` |
| `canvas_tasks` | `upcoming` | `/proximas` |
| `static_start` | `none` | `/start` |
| `static_help` | `none` | `/ayuda` |
| `static_status` | `none` | `/estado` |
| `unknown` o baja confianza | `none` | Solicitar aclaración |

La respuesta contiene `intent`, `period`, `confidence` y, cuando aplica, `clarification`. La capa de IA no recibe herramientas ni credenciales operativas: los workflows especializados conservan la ejecución de acciones.

## Modelo y Operación

Se usa `models/gemini-3.1-flash-lite` con temperatura `0.1` y máximo de `250` tokens. Es adecuado para clasificación breve y de alto volumen. El workflow principal activo es `wf_telegram_router` (`vAaVfPUfv6tcyMrV`).

La ruta natural ya fue validada en Telegram: una consulta por tareas de la semana se clasificó como `canvas_tasks/week` y devolvió tareas reales desde Canvas.

## Próximas Pruebas

Probar frases naturales para hoy, próximas tareas, ayuda, estado y mensajes ambiguos. Los fallbacks deben pedir aclaración y nunca inventar datos académicos.
