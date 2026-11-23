# 024 - Inicio de la reserva: escritura previa, resultado incierto e idempotencia

## Contexto
Iniciar una reserva combina una base local y llamadas REST a la cátedra que pueden fallar a mitad de camino: un timeout no prueba
que la operación haya fallado, y el servicio puede reiniciarse entre una llamada y su registro. El enunciado exige no generar
reservas duplicadas, tolerar timeouts y respuestas perdidas, y que el estado local sea siempre recuperable (secciones 7 y 8).

## Decisión
- Dos pasos, como el enunciado: POST /api/reservations crea el hold y POST /api/reservations/{id}/confirm hace la
  confirmación inicial. Entre los dos, el usuario ve la vigencia del bloqueo.
- Escritura previa: el proceso se guarda en STARTED antes de llamar a la cátedra, y el hold se guarda al recibir su respuesta.
  Si el servicio se cae en el medio, queda un registro STARTED que la recuperación resuelve por tiempo.
- Los casos de uso no son transaccionales: no hay una transacción de base abierta mientras se espera a la cátedra.
- Los POST a la cátedra no se reintentan solos. Un rechazo firme (4xx) deja el proceso FAILED con el code de la cátedra y se
  traduce a un error de la API (turno ocupado, profesional deshabilitado, turno inválido). Un resultado incierto (el pedido pudo
  llegar) deja el proceso FAILED con el motivo HOLD_OUTCOME_UNKNOWN: no se puede usar ni repetir a ciegas un hold que no
  conocemos, y el hold huérfano vence por su propia vigencia.
- Idempotencia al iniciar: si el mismo usuario repite el pedido para el mismo turno, se devuelve el proceso existente; si el
  existente sigue en STARTED, se informa que está en curso. Los procesos finales no bloquean un intento nuevo.
- Idempotencia al confirmar: si el proceso ya avanzó, se devuelve su estado sin llamar a la cátedra; si la cátedra responde que ya
  estaba confirmado (se perdió la respuesta de un intento anterior), el proceso converge a WAITING_FOR_PHONE. Si la llamada no se
  completa, el proceso sigue HELD y repetir es seguro.
- Convergencia con los eventos: el estado se relee antes de escribir y, si ya no puede pasar al estado pedido, se devuelve el
  actual sin retroceder. Un conflicto de versión se reintenta hasta 3 veces.
- La vigencia del hold la decide la cátedra: no se adelanta el juicio con el reloj local. HOLD_EXPIRED pasa el proceso a EXPIRED.
- El usuario y el externalPatientId (user-<id>) salen del token; el cuerpo no puede cambiarlos. La API no expone los
  identificadores internos de la cátedra. El proceso de otro usuario responde 404.
- La validación contra service-catalog (profesional habilitado y turno real de la agenda) ocurre antes de guardar nada, y sus
  consultas no se reintentan: es un pedido del usuario y fallar rápido no deja estado.

## Alternativas descartadas
- Un solo endpoint que crea el hold y confirma: el usuario no vería el bloqueo, y un fallo de la segunda llamada obligaría a
  deshacer la primera.
- Guardar el proceso después de crear el hold: un fallo entre las dos acciones dejaría un hold sin rastro local.
- Una transacción de base alrededor de las llamadas HTTP: mantiene conexiones y bloqueos abiertos mientras se espera a un tercero.
- Reintentar automáticamente el POST del hold: puede duplicar holds.
- Un índice único sobre los turnos en curso: un proceso con su vencimiento perdido bloquearía el turno (ADR 019).

## Consecuencias
- Un hold huérfano (el pedido llegó pero la respuesta se perdió) bloquea el turno hasta su vencimiento, y el mismo usuario puede
  recibir "turno no disponible" por su propio hold hasta entonces.
- Dos pedidos simultáneos del mismo usuario para el mismo turno pueden crear dos procesos; la cátedra rechaza el segundo hold y
  ese proceso queda FAILED.
- Los procesos STARTED que quedan por una caída necesitan la recuperación por tiempo (pendiente).
- Los nombres del paciente vienen en el pedido: son datos descriptivos y no dan acceso a nada.