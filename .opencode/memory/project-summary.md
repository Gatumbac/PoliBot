# Proyecto: ESPOL Academic Intelligence Agent
**Branch**: main
**Last updated**: 2026-07-12

## Estado actual
- **En progreso**: Definición de siguiente paso de arquitectura tras limpiar warnings del workflow MVP
- **Bloqueado**: ninguno
- **Próximo**: Agregar ruteo por comandos (ej. `/week`) sin tocar aún Canvas

## Cambios recientes
- Se implementó MVP Telegram -> Canvas -> Telegram en `EspolAgent`
- Se corrigió parsing de fechas usando `assignment.due_at`
- Se ajustó timezone a `America/Guayaquil`
- Se restauró nodo `Build Fallback Reply`
- Se limpió warning de `Telegram Reply` dejando `resource=message` y `operation=sendMessage`
