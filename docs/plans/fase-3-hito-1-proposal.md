# Propuesta de Implementación - Fase 3, Hito 1

**Estado**: implementado, publicado y validado en Telegram.

## Nombre

Resumen y priorización determinista de tareas Canvas.

## Objetivo

Convertir las respuestas actuales de `/hoy`, `/semana` y `/proximas` en una
lista ordenada por urgencia, con una explicación breve y verificable para cada
tarea.

El hito no cambia la fuente de datos. Canvas continúa siendo la fuente de
verdad y la prioridad se calcula en n8n a partir de los datos recibidos. La IA
puede seguir clasificando lenguaje natural, pero no decide el orden ni inventa
información académica.

## Resultado visible para el estudiante

Ante una consulta como `¿qué tengo pendiente esta semana?`, el bot responderá
con las tareas más urgentes primero:

```text
🗂️ Tareas priorizadas

1. 🔴 Entrega del proyecto
   📚 Estructuras de Datos
   ⏰ Vence: hoy, 23:59
   🎯 Prioridad: crítica
   💡 Motivo: vence en menos de 24 horas

2. 🟠 Quiz de lectura
   📚 Bases de Datos
   ⏰ Vence: viernes, 23:59
   🎯 Prioridad: alta
   💡 Motivo: vence en menos de 72 horas
```

El formato debe conservar `America/Guayaquil` y los enlaces de Canvas cuando
estén disponibles. La propuesta reduce el límite actual de 15 a un máximo de
10 tareas priorizadas para proteger la legibilidad. Si no existen tareas, se
mantiene una respuesta explícita de lista vacía.

## Alcance de la primera versión

- Enriquecer la salida de `wf_cmd_canvas_queries`.
- Mantener `/hoy`, `/semana` y `/proximas` como comandos canónicos.
- Mantener las frases naturales existentes y hacer que terminen en la misma
  salida enriquecida.
- Calcular una prioridad reproducible usando fecha de vencimiento y estado
  disponible en la respuesta de Canvas.
- Mostrar como máximo 10 tareas priorizadas para proteger la legibilidad en
  Telegram.
- Conservar el manejo actual de errores, límites de Canvas y `429`.
- Registrar en la ejecución la cantidad de tareas recibidas, descartadas y
  mostradas.

## Fuera de alcance

- Usar Gemini u otro modelo para ordenar tareas.
- Inferir prioridad a partir del peso de una tarea mientras ese dato no esté
  disponible de forma consistente.
- Crear recordatorios o mensajes proactivos.
- Crear eventos en Google Calendar.
- Consultar sílabos, notas o archivos.
- Cambiar el contrato de autenticación de Canvas.
- Introducir memoria conversacional.

## Contrato de entrada

La función de priorización recibirá la lista ya obtenida por el workflow de
Canvas. Debe tolerar que algunos campos sean opcionales.

```json
{
  "id": "assignment-123",
  "name": "Entrega del proyecto",
  "course_name": "Estructuras de Datos",
  "due_at": "2026-07-17T04:59:00.000Z",
  "html_url": "https://aulavirtual.espol.edu.ec/courses/1/assignments/123",
  "completed": false
}
```

Reglas de normalización:

- Usar `assignment.due_at` cuando exista y, como fallback, `due_at`.
- Convertir la fecha a `America/Guayaquil` únicamente para calcular y
  presentar la fecha local; no modificar el valor original de Canvas.
- Considerar una tarea sin fecha como válida, pero asignarle la prioridad más
  baja y mostrar `sin fecha de vencimiento`.
- No asumir que un campo opcional existe. Si falta materia, enlace o estado,
  omitir ese dato en vez de inventarlo.
- Preservar el conjunto de tareas que actualmente devuelve cada período. La
  primera versión no debe cambiar el significado de `/hoy`, `/semana` o
  `/proximas`.

## Regla de prioridad

La prioridad se calcula con la diferencia entre `now` y la fecha de
vencimiento, usando la zona horaria `America/Guayaquil`.

| Condición | Nivel | Marcador |
| --- | --- | --- |
| Vencida | Crítica | 🔴 |
| Vence en 24 horas o menos | Crítica | 🔴 |
| Vence en más de 24 y hasta 72 horas | Alta | 🟠 |
| Vence en más de 72 horas y hasta 7 días | Media | 🟡 |
| Vence en más de 7 días | Baja | 🟢 |
| No tiene fecha | Baja | ⚪ |

La prioridad no depende de un llamado al modelo ni de una decisión aleatoria.
Para que el orden sea estable, las tareas se ordenan en este orden:

1. Nivel de prioridad, de crítica a baja.
2. Fecha de vencimiento ascendente; las tareas sin fecha van al final.
3. Nombre de la materia en orden alfabético.
4. Nombre de la tarea en orden alfabético.
5. Identificador de Canvas como último desempate.

Una tarea vencida debe mostrar un motivo distinto de una tarea próxima:

- `vence en menos de 24 horas`;
- `vence en menos de 72 horas`;
- `vence esta semana`;
- `ya está vencida`;
- `no tiene fecha de vencimiento registrada`.

El motivo es una plantilla determinista. No se genera con IA.

## Decisión sobre tareas completadas

La primera implementación no debe inventar un nuevo criterio de completitud.
Debe conservar el filtro que ya aplica `wf_cmd_canvas_queries` y excluir una
tarea solo cuando Canvas entregue explícitamente un estado que indique que ya
fue completada.

Si el endpoint devuelve una tarea sin estado, se la conserva para no ocultar
trabajo potencialmente pendiente. Esta decisión debe quedar registrada en la
implementación si el payload real de Canvas presenta una variación.

## Contrato de salida

El sub-workflow debe devolver una respuesta lista para `Telegram Reply`, sin
exponer el payload completo de Canvas.

```json
{
  "ok": true,
  "period": "week",
  "total_count": 2,
  "displayed_count": 2,
  "items": [
    {
      "id": "assignment-123",
      "name": "Entrega del proyecto",
      "course": "Estructuras de Datos",
      "due_at": "2026-07-17T04:59:00.000Z",
      "priority": "critical",
      "reason": "vence en menos de 24 horas",
      "html_url": "https://aulavirtual.espol.edu.ec/courses/1/assignments/123"
    }
  ],
  "message": "..."
}
```

Campos mínimos de `items`:

- `id`, si Canvas lo proporciona;
- `name`;
- `priority`;
- `reason`;
- `due_at`, cuando exista;
- `course`, cuando exista;
- `html_url`, cuando exista.

El mensaje de Telegram debe ser generado por una plantilla del workflow. El
modelo no debe redactar ni transformar los datos académicos en esta etapa.

## Cambios propuestos en n8n

1. Revisar el nodo que normaliza la respuesta de Canvas dentro de
   `wf_cmd_canvas_queries`.
2. Añadir una transformación determinista para calcular nivel, motivo y orden.
3. Aplicar el límite de 10 elementos después de ordenar.
4. Informar `total_count` y `displayed_count` para hacer visible si hubo tareas
   que quedaron fuera por el límite.
5. Añadir el formato enriquecido para Telegram, manteniendo los fallbacks
   actuales.
6. Verificar que los errores de Canvas, incluido `429`, sigan terminando en el
   mensaje controlado existente.
7. No modificar las rutas exactas del router ni el contrato de
   `wf_ai_intent_router`.

La propuesta no requiere un nuevo workflow. La responsabilidad permanece en
`wf_cmd_canvas_queries` porque el sub-workflow ya posee la consulta a Canvas,
la normalización de fechas y la respuesta de las consultas de tareas.

## Validación

La validación debe hacerse desde Telegram y luego comprobarse en la ejecución
correspondiente de n8n. Las pruebas aisladas de un nodo no son suficientes.

| Caso | Entrada | Resultado esperado |
| --- | --- | --- |
| Tarea vencida | `/hoy` o consulta natural equivalente | Aparece primero como crítica y con motivo de vencimiento |
| Vence pronto | `/semana` | Las tareas de menor tiempo restante aparecen antes |
| Varias tareas empatadas | `/semana` | El orden se mantiene estable entre ejecuciones |
| Sin fecha | `/proximas` | Aparece al final como baja, con texto explícito |
| Sin tareas | período sin pendientes | Mensaje claro de lista vacía |
| Error Canvas | consulta normal durante error | Fallback actual, sin payload técnico |
| `429` | consulta durante rate limit | Mensaje actual de espera, sin romper el formato |
| Lenguaje natural | `¿Qué tengo pendiente esta semana?` | Misma salida que `/semana` |

## Criterios de aceptación

- La misma entrada y el mismo conjunto de datos producen el mismo orden.
- Ninguna prioridad depende de una respuesta del modelo.
- Las fechas se presentan en `America/Guayaquil`.
- Las tareas sin fecha no desaparecen ni se presentan como si tuvieran fecha.
- La respuesta no supera 10 tareas priorizadas y distingue el total recibido
  de la cantidad mostrada.
- Los comandos exactos y las frases naturales ya validadas continúan
  funcionando.
- Los fallbacks de Canvas y `429` se conservan.
- La respuesta no expone tokens, payloads completos ni detalles técnicos.
- Existe evidencia de al menos una validación por Telegram para cada período:
  `today`, `week` y `upcoming`.
- `docs/operations/implementation-status.md` se actualiza con las
  ejecuciones verificadas antes de declarar cerrado el hito.

## Riesgos y decisiones pendientes

- El payload real de Canvas debe confirmar el nombre exacto del campo de
  completitud. Si no existe, se conserva el comportamiento actual y no se
  agrega filtrado especulativo.
- La ventana de 24/72 horas depende del instante de consulta; la ejecución
  debe usar una sola marca `now` para todas las tareas.
- El límite de 10 puede ocultar tareas posteriores. La respuesta debe indicar
  cuántas tareas adicionales quedaron fuera cuando `total_count` sea mayor que
  `displayed_count`.
- Los pesos de nota no se incluyen hasta que Canvas los entregue de forma
  consistente y se defina una regla independiente de urgencia.

## Secuencia de implementación

1. Capturar un payload real sanitizado de cada período y confirmar los campos
   disponibles.
2. Implementar la normalización y el cálculo determinista dentro de
   `wf_cmd_canvas_queries`.
3. Probar el formateo con tareas críticas, próximas, sin fecha y empates.
4. Ejecutar la batería E2E por Telegram para los tres períodos.
5. Actualizar el estado operativo y decidir si el Hito 1 cumple sus criterios
   de salida.

## Decisión solicitada

Aprobar esta propuesta como contrato de trabajo del Hito 1. Una vez aprobada,
la implementación puede comenzar revisando primero el payload real de Canvas y
no requiere esperar a los hitos de RAG o memoria.

## Cierre

El contrato fue cumplido en `wf_cmd_canvas_queries`. Esta propuesta queda como
referencia del diseño y no como trabajo pendiente.
