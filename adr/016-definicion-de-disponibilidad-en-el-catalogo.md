# 016 - Definición del filtro de disponibilidad del catálogo

## Contexto
El enunciado pide filtros por categoría, nombre, estado habilitado y disponibilidad, resueltos con la copia local
(secciones 4.1 y 6.2), y no admite consultar a la cátedra en cada búsqueda. La disponibilidad concreta de turnos se
construye en service-turns con los horarios y las ocupaciones (sección 7). El enunciado no define qué es "disponibilidad"
en el catálogo.

## Decisión
En el catálogo, un profesional está disponible si tiene al menos una agenda semanal habilitada (enabled = true),
opcionalmente en un día de la semana dado. Se resuelve con una consulta sobre la copia local, en
GET /api/catalog/professionals (ver ADR 017), con los parámetros hasSchedules y dayOfWeek. Los turnos libres de una
fecha concreta los calcula service-turns.
- dayOfWeek implica hasSchedules = true: combinarlo con hasSchedules = false es contradictorio y se rechaza con un 400.

## Alternativas descartadas
- "Tiene turnos libres en una fecha": necesita las ocupaciones de la cátedra en cada búsqueda, lo que contradice la
  sección 6.2.
- Incluir el estado habilitado del profesional dentro de la disponibilidad: ya existe como filtro propio.

## Consecuencias
- Un profesional con agenda habilitada puede no tener turnos libres en una fecha: la aplicación debe mostrar la
  disponibilidad real desde service-turns.
- Las agendas deshabilitadas se ignoran para este filtro, pero se devuelven marcadas con enabled = false.