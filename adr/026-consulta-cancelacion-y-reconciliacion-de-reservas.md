# 026 - Consulta, cancelación y reconciliación de reservas

## Contexto
Cada usuario debe poder consultar sus reservas y cancelar una confirmada, y el estado local debe converger con el de la cátedra aunque
se pierdan notificaciones (secciones 5, 7 y 8 del enunciado). La cancelación es idempotente en la cátedra: una reserva ya cancelada
devuelve su estado, y una que no está confirmada ni cancelada se rechaza con RESERVATION_NOT_CANCELLABLE.

## Decisión
- Las reservas del usuario se listan desde la base local, filtradas por el usuario del token, de la más nueva a la más vieja, con un
  filtro opcional por estado y un tope de 100. La cátedra informa las reservas de toda la cuenta técnica y no distingue usuarios
  finales, así que el aislamiento es responsabilidad de este servicio.
- Solo se cancela una reserva CONFIRMED y propia. Una ya CANCELLED devuelve su estado (idempotente); cualquier otro estado es un
  conflicto. La reserva de otro usuario responde 404.
- Como la cancelación es idempotente en la cátedra, ante un resultado incierto repetir es seguro: el proceso sigue CONFIRMED y el
  usuario reintenta. A diferencia de la creación del hold, no hace falta registrar un estado intermedio ni reconciliar antes.
- Si la cátedra rechaza con RESERVATION_NOT_CANCELLABLE, el estado local no coincide con el suyo: se consulta el real y se converge. Si
  ya estaba cancelada se devuelve 200; si no, 409.
- La reconciliación consulta GET /api/appointments filtrando por el paciente y la fecha del turno, y aplica solo transiciones
  permitidas: CONFIRMED y CANCELLED se reflejan (pasando por CONFIRMED si se perdieron los dos eventos), FAILED pasa a EXPIRED si ya
  venció la vigencia y a FAILED si no, y un hold sin confirmar que venció y la cátedra no conoce pasa a EXPIRED. PHONE_PENDING no cambia
  nada: sin el eventId del pedido no se puede enviar el teléfono.
- La reconciliación está en un componente propio, que usan el endpoint de refresco, la cancelación y el job de recuperación.
- El estado se relee antes de escribir y no retrocede; un conflicto de versión se reintenta hasta 3 veces.

## Alternativas descartadas
- Listar desde GET /api/appointments: mezcla las reservas de todos los usuarios de la cuenta técnica.
- Reintentar la cancelación dentro de la petición: la repetición es segura, pero demora al usuario; es más simple devolver 503 y dejar
  que reintente, y los eventos igual convergen.
- Un estado intermedio CANCELLING: agrega un estado y su recuperación, y la idempotencia de la cátedra lo vuelve innecesario.

## Consecuencias
- Una reserva puede quedar un tiempo CONFIRMED localmente después de cancelarse en la cátedra, hasta que llegue el evento o se refresque.
- El REST de la cátedra no distingue una reserva vencida de una fallida: con la vigencia cumplida se trata como vencida.
- El listado devuelve como máximo 100 reservas, incluidas las fallidas; la aplicación puede filtrar por estado.