# ESPOL Academic Agent - progreso de implementación

Fecha: 2026-07-12
Proyecto: `EspolAgent` (workflowId: `bJG32QZhycbFenCh`)

## Objetivo acordado
Construir un MVP vertical slice en n8n: Telegram -> Canvas API -> respuesta por Telegram, para validar factibilidad técnica del agente académico ESPOL.

## Avances completados
1. Se confirmó acceso MCP a workflows n8n y lectura de detalles de `EspolAgent`.
2. Se implementó flujo base:
   - `Telegram Trigger`
   - `Fetch Canvas Todo` (`HTTP Request` a `GET /api/v1/users/self/todo`)
   - `Format Weekly Tasks` (`Code`)
   - `Telegram Reply`
3. Se agregó ruta de fallback para errores de Canvas:
   - `Fetch Canvas Todo` con `onError: continueErrorOutput`
   - `Build Fallback Reply` (`Set`) conectado al segundo output de error
   - envío de mensaje de fallback por Telegram.
4. Se corrigió bug lógico de parsing de fechas:
   - antes filtraba por `t.due_at` (vacío en /todo)
   - ahora usa `t.assignment?.due_at || t.due_at`.
5. Se corrigió formato horario para Ecuador:
   - `toLocaleString('es-EC', { timeZone: 'America/Guayaquil', hour12: false })`.
6. Se verificó ejecución exitosa end-to-end por MCP (`search_executions` y `get_execution`), incluyendo payload real de Canvas.

## Hallazgos técnicos clave
- El flujo completo sí funciona; tests por nodo individual pueden dar falsos negativos por falta de contexto/input.
- Se observaron errores intermitentes de infraestructura/rate-limit al inicio:
  - `The service is receiving too many requests from you` (rate limit)
  - errores DNS intermitentes del contenedor.
- Para multiusuario (partners), personal token no escala; se recomendó OAuth por usuario (token + refresh token por Telegram user).

## Estado actual funcional
- Bot responde con tareas de la semana obtenidas desde Canvas.
- Fallback restaurado y operativo si falla Canvas.
- Queda warning menor pendiente en `Telegram Reply` (discriminator `resource`) que no bloquea la funcionalidad principal observada.

## Próximos pasos sugeridos
1. Limpiar warning de `Telegram Reply` para dejar workflow sin validaciones pendientes.
2. Agregar comando `/week` para no responder a cualquier mensaje.
3. Añadir retry/cooldown para 429 de Canvas.
4. Diseñar flujo OAuth por usuario para onboarding de partners.
