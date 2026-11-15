# 019 - Modelo de datos de service-turns

## Contexto
service-turns es el único dueño de sus datos, con su propia base y sus propias migraciones. Cada proceso de reserva debe
quedar asociado al usuario que lo inició y conservar el estado local aunque el servicio se reinicie.

## Decisión
- Una sola tabla, reservation_processes: la reserva confirmada es el mismo proceso en estado CONFIRMED, y cancelarla es
  una transición más. Se agrega processed_events para la idempotencia por eventId.
- La clave es un UUID generado por el servicio, que no se puede adivinar. Los identificadores de la cátedra
  (catedra_process_id, hold_id, reservation_id) son opcionales hasta que existen y únicos cuando existen.
- user_id es el dueño: sale siempre del token y no se acepta del cliente.
- Se guardan datos descriptivos históricos del profesional (nombre y apellido), que no se usan como catálogo vigente: el
  catálogo vigente se consulta a service-catalog.
- Columna version para bloqueo optimista: los eventos de Kafka y los pedidos REST pueden tocar el mismo proceso a la vez.
- Restricciones en la base como red de seguridad: estados válidos, ventana horaria, identificadores de la cátedra después de
  STARTED, id de reserva en CONFIRMED y CANCELLED, teléfono y requestEventId en PHONE_SUBMITTED.
- Índice único parcial: nunca dos reservas CONFIRMED para el mismo turno.

## Alternativas descartadas
- Dos tablas, proceso y reserva: duplican datos y separan la cancelación de la máquina de estados.
- Índice único sobre los turnos en curso: si se pierde un evento de vencimiento, un hold vencido bloquearía el turno hasta
  que se recupere. Para los holds, la cátedra es la autoridad.
- Identificadores numéricos secuenciales: se pueden adivinar.
- Guardar una copia del catálogo: el enunciado lo prohíbe.

## Consecuencias
- La propiedad por usuario se aplica en el código y en las consultas; la base no la fuerza.
- No hay historial de transiciones: solo el estado actual, el motivo y las fechas clave.
- Los datos descriptivos del profesional pueden quedar desactualizados respecto del catálogo, y es lo esperado.