# Plan de Implementación - Fase 3

## Objetivo

Convertir PoliBot de un asistente reactivo que responde comandos y consultas
de Canvas en un asistente académico enriquecido que:

- resume y prioriza tareas,
- aporta contexto de sílabos con fuentes citables,
- recuerda contexto breve por `chatId` sin mezclarlo con datos académicos.

## Alcance mínimo

La fase 3 se considera lista para arrancar solo si estas tres capacidades están
definidas y probadas en un circuito cerrado:

1. Priorización de tareas Canvas con salida determinista.
2. Recuperación de contexto desde sílabos con trazabilidad de fuente.
3. Memoria breve por chat, separada de Canvas y de credenciales.

No entran en el primer corte:

- Google Calendar,
- OAuth por estudiante,
- alertas proactivas nuevas,
- predicción avanzada de nota,
- modo emergencia,
- paneles externos.

## Principios de diseño

- La IA sugiere, pero no inventa datos.
- Canvas sigue siendo la fuente de verdad para tareas y fechas.
- El sílabo solo aporta contexto adicional, nunca sustituye a Canvas.
- La memoria breve no almacena datos académicos sensibles.
- Toda salida enriquecida debe poder rastrearse a una fuente o a una regla
  determinista.

## Dependencias previas

Antes de implementar la fase 3, hay que tener cerrados estos puntos:

- `wf_telegram_router` operativo y estable.
- `wf_cmd_canvas_queries` funcionando con manejo de `429`.
- `wf_ai_intent_router` validado para `today`, `week`, `upcoming` y
  aclaraciones.
- Decisión sobre el almacén de memoria breve.
- Decisión sobre la fuente de sílabos y su formato de ingesta.

## Hito 1 - Resumen y priorización de tareas

### Objetivo

Convertir la consulta de tareas en una respuesta con orden de ejecución y
justificación breve.

### Trabajo requerido

- Definir una regla de priorización estable.
- Tomar los datos actuales de Canvas como entrada.
- Construir una salida que incluya:
  - tarea,
  - materia,
  - vencimiento,
  - prioridad,
  - motivo breve.
- Limitar el número de tareas mostradas para mantener legibilidad.
- Mantener el formato horario de `America/Guayaquil`.

### Criterio de salida

La consulta de tareas devuelve una lista ordenada y reproducible, sin que el
modelo reordene arbitrariamente ni invente prioridades.

### Riesgos

- Sobrecargar la respuesta con texto largo.
- Mezclar sugerencias del modelo con reglas deterministas sin control.
- Cambiar la prioridad entre ejecuciones sin una regla clara.

## Hito 2 - Contexto de sílabos con RAG

### Objetivo

Responder preguntas de contexto académico usando sílabos o materiales
indexados, con referencias citables.

### Trabajo requerido

- Definir las fuentes:
  - `syllabus_body` de Canvas,
  - PDFs o HTML de sílabos si existen.
- Diseñar la ingesta:
  - extracción,
  - limpieza,
  - fragmentación,
  - indexación.
- Definir metadatos mínimos:
  - curso,
  - unidad o tema,
  - fuente,
  - fecha de ingesta,
  - referencia al documento.
- Definir el formato de respuesta:
  - resumen corto,
  - fragmentos recuperados,
  - fuente visible cuando aplique.

### Criterio de salida

Una pregunta sobre contenido de sílabo devuelve una respuesta con fuente
recuperada y sin inventar contenido fuera del material indexado.

### Riesgos

- Sílabos desactualizados.
- Recuperación de fragmentos irrelevantes.
- Respuestas sin trazabilidad de fuente.

## Hito 3 - Memoria breve por `chatId`

### Objetivo

Guardar contexto conversacional mínimo para mejorar continuidad sin persistir
datos académicos sensibles.

### Trabajo requerido

- Definir qué se guarda:
  - último tema consultado,
  - preferencia de profundidad,
  - periodo reciente de interés.
- Definir qué no se guarda:
  - notas,
  - tareas,
  - tokens,
  - datos personales innecesarios.
- Elegir almacenamiento:
  - tabla PostgreSQL,
  - KV simple,
  - otro mecanismo controlado.
- Definir TTL o política de expiración.

### Criterio de salida

El bot usa contexto mínimo para hacer mejores respuestas, pero no expone ni
contamina datos académicos con memoria conversacional.

### Riesgos

- Mezclar memoria y datos de Canvas.
- Guardar más de lo necesario.
- Crear dependencia invisible de contexto persistente.

## Hito 4 - Integración y guardrails

### Objetivo

Unificar las tres capacidades sin romper la ruta actual de comandos y sin
degradar la confiabilidad.

### Trabajo requerido

- Definir qué intentos naturales activan cada subruta.
- Mantener comandos exactos como ruta rápida.
- Asegurar que la ruta de aclaración siga funcionando.
- Aplicar validaciones de estructura para la salida del modelo.
- Registrar:
  - intención,
  - fuente usada,
  - resultado,
  - error,
  - duración.

### Criterio de salida

El usuario puede usar lenguaje natural para obtener respuestas enriquecidas y,
si la entrada es ambigua, el bot pide aclaración en vez de improvisar.

### Riesgos

- Aumentar latencia.
- Crear rutas demasiado difusas.
- Romper la simplicidad del router actual.

## Hito 5 - Validación y salida

### Objetivo

Demostrar que la fase 3 añade valor real y no solo más complejidad.

### Trabajo requerido

- Preparar casos de prueba manuales en Telegram.
- Validar:
  - prioridad correcta,
  - citas de sílabo cuando aplique,
  - memoria breve efectiva,
  - fallbacks controlados.
- Documentar resultados en `docs/operations/implementation-status.md`.
- Actualizar el roadmap al cerrar el hito.

### Criterio de salida

La fase 3 queda cerrada cuando las tres capacidades mínimas están activas,
probadas por Telegram y documentadas con evidencia operativa.

## Secuencia recomendada

1. Definir contratos y datos.
2. Implementar priorización de tareas.
3. Implementar ingesta y RAG de sílabos.
4. Implementar memoria breve.
5. Integrar rutas y guardrails.
6. Validar por Telegram y cerrar documentación.

## Decisión de arranque

La fase 3 no debe empezar como “todo a la vez”. Debe arrancar por el Hito 1 y
bloquear el avance a los siguientes hitos hasta que el contrato de salida del
hito actual esté validado.
