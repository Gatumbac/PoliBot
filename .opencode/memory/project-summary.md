# Proyecto: ESPOL Academic Intelligence Agent
**Branch**: main
**Last updated**: 2026-07-16

## Estado actual
- **En progreso**: Variaciones de lenguaje natural para consultas Canvas
- **Bloqueado**: Ninguno
- **Próximo**: Probar frases coloquiales para hoy y próxima semana

## Cambios recientes
- Se publicó `wf_telegram_router` y responde mensajes reales por Telegram
- Se integró `wf_ai_intent_router` con Gemini 3.1 Flash-Lite para lenguaje natural
- La IA clasifica tareas semanales y enruta la consulta a Canvas correctamente
- Se cerró la batería E2E webhook para `/start`, `/estado`, `/hoy` y `/semana`
- `wf_cmd_canvas_queries` ya maneja `429` con respuesta controlada en Telegram
- Se validaron `ayuda`, `estado`, `próximas` y ambigüedad en el router de IA
- Se archivó `PoliBot_Inicial` para dejar solo el stack v1 activo
- Se archivó el workflow genérico `Test`
