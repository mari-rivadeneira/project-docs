# 023 - Cálculo de la disponibilidad de turnos

## Contexto
El enunciado exige que service-turns construya la disponibilidad combinando las reglas semanales vigentes de service-catalog con
las ocupaciones que informa la cátedra (sección 7), sin duplicar una réplica del catálogo (sección 3). Si service-catalog no
está disponible, no pueden iniciarse operaciones que requieran validar información vigente (sección 8).

## Decisión
- Los turnos del día salen de las agendas habilitadas del día de la semana, partidas en turnos de slotDurationMinutes. Solo
  cuentan los turnos que entran enteros en la agenda. Varias agendas del mismo día se unen, sin repetir turnos.
- Las ocupaciones de la cátedra (reservas confirmadas y holds activos) descuentan. Un turno se considera ocupado si alguna
  ocupación de esa fecha empieza dentro de él: es conservador y cubre un cambio de duración de turnos en el catálogo.
- La cátedra es la autoridad de los holds: no se consultan los procesos locales para calcular la disponibilidad (ADR 019).
- Si la fecha es hoy, se descartan los turnos que ya empezaron, con un reloj en la zona horaria de atención, configurable. Una
  fecha anterior a hoy es un error de validación.
- Si el día no tiene turnos posibles, no se consulta a la cátedra.
- El profesional y sus agendas se piden a service-catalog reenviando el JWT del usuario (ADR 021). Un profesional inexistente
  responde 404 y uno deshabilitado, 409.
- Las consultas se reintentan una vez ante una indisponibilidad, con un tope de 2 intentos; un rechazo no se reintenta. Si
  service-catalog o la cátedra siguen sin responder, se informa 503: no se inventa disponibilidad.
- Se devuelven solo los turnos disponibles, con hora de inicio y de fin en hora local de atención.

## Alternativas descartadas
- Guardar una copia de las agendas o de las ocupaciones: el enunciado prohíbe replicar el catálogo, y una copia de las
  ocupaciones mostraría turnos que ya no están libres.
- Considerar también los procesos locales como ocupación: duplica la autoridad de la cátedra y puede bloquear un turno por un
  evento de vencimiento perdido.
- Coincidencia exacta de la hora de inicio: dejaría libre un turno si cambió la duración de los turnos en el catálogo.
- Devolver todos los turnos con su estado: la aplicación solo necesita los libres.

## Consecuencias
- La disponibilidad es informativa: entre que se muestra y se pide el hold, otra persona puede tomar el turno; la creación del
  hold lo resuelve con un 409.
- Cada consulta implica llamadas a service-catalog y a la cátedra; con las ocupaciones pedidas por día, se consulta solo lo necesario.
- La zona horaria de atención se asume (Argentina) y es configurable; si la cátedra usara otra, las horas se interpretarían mal.
- La fecha límite hacia adelante no está acotada.