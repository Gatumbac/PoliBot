# Roadmap de PoliBot

## Fase 1: MVP confiable

**Estado**: implementación terminada; pendiente de publicación.

- Publicar `wf_telegram_router`.
- Validar comandos y fallback con Telegram y Canvas reales.
- Añadir control de reintentos o cooldown para respuestas 429 de Canvas.

**Criterio de salida**: un mensaje de Telegram activa automáticamente el router
y cada comando v1 devuelve una respuesta clara o un fallback controlado.

## Fase 2: Conversación natural

- Crear `wf_ai_intent_router` para convertir consultas libres en intención,
  parámetros, confianza y necesidad de aclaración.
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
