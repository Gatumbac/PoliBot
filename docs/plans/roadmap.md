# Roadmap de PoliBot

## Fase 1: MVP confiable

**Estado**: completada; router activo en producción.

- `wf_telegram_router` está activo en producción y recibe mensajes por webhook.
- Validar comandos y fallback con Telegram y Canvas reales.
- Añadir control de reintentos o cooldown para respuestas 429 de Canvas.

**Criterio de salida**: un mensaje de Telegram activa automáticamente el router
y cada comando v1 devuelve una respuesta clara o un fallback controlado. Este
criterio ya quedó cubierto por las validaciones documentadas en
`docs/operations/implementation-status.md`.

## Fase 2: Conversación natural

- `wf_ai_intent_router` está conectado y validado con consultas reales de
  tareas semanales usando Gemini.
- Validar el JSON antes de llamar workflows de Canvas.
- Mantener comandos exactos como ruta rápida y fallback estable.

**Criterio de salida**: el bot entiende equivalencias como “qué tengo esta
semana” sin permitir que el modelo invente datos o elija credenciales.

## Fase 3: Respuestas académicas enriquecidas

- Resumir y priorizar tareas usando datos obtenidos de Canvas.
- Añadir contexto de sílabos mediante RAG, con fuentes citables.
- Incorporar memoria breve por `chatId`, separada de los datos académicos.

## Fase 4: Acciones y multiusuario

- Integrar Google Calendar con confirmación explícita antes de crear eventos.
- Implementar OAuth por estudiante; no escalar el token personal del owner.
- Añadir alertas proactivas, auditoría y evaluación periódica de calidad.
