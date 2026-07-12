# PoliBot

Asistente académico para estudiantes de ESPOL, construido con n8n, Telegram y
Canvas LMS. El repositorio conserva la visión del producto, arquitectura,
estado operativo y planes; los workflows se administran en la instancia n8n.

## Estado actual

PoliBot v1 cuenta con un router de Telegram, respuestas estáticas y consultas
de tareas Canvas. Los workflows están creados y validados con pruebas
simuladas; falta publicar el router y realizar la validación final con Telegram.

## Documentación

- [Arquitectura de PoliBot v1](docs/architecture/polibot-v1.md)
- [Visión del ESPOL Academic Intelligence Agent](docs/product/espol-academic-agent.md)
- [Estado de implementación](docs/operations/implementation-status.md)
- [Roadmap](docs/plans/roadmap.md)
- [Guía de contribución](AGENTS.md)

## Estructura

```text
docs/
  architecture/  # Diseño técnico y contratos entre workflows
  operations/    # Estado, validación y operación en n8n
  plans/         # Fases y criterios de salida
  product/       # Problema, visión y alcance del producto
```

No se versionan tokens, credenciales de n8n ni datos académicos de estudiantes.
