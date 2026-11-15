# 018 - Máquina de estados del proceso de reserva

## Contexto
La reserva mezcla REST (el hold y la confirmación inicial) y Kafka (el pedido de teléfono y el resultado), con mensajes
que llegan en momentos distintos, repetidos o tarde. El enunciado exige converger sin reservas duplicadas ni retroceder un
proceso que ya llegó a un estado final (sección 7) y que cada alumno defina la máquina de estados local (sección 10).

## Decisión
- Once estados: STARTED (intención registrada, todavía sin hold), HELD (hold creado), WAITING_FOR_PHONE (confirmación
  inicial aceptada), PHONE_REQUESTED (la cátedra pidió el teléfono), PHONE_SUBMITTED (teléfono enviado), PHONE_REJECTED
  (teléfono rechazado, se puede reenviar), CONFIRMED, y los finales CANCELLED, EXPIRED, INVALID y FAILED.
- Las transiciones permitidas están declaradas explícitamente en el enumerado ReservationStatus; cualquier otra se rechaza.
  Los estados finales no tienen salida. CONFIRMED solo admite la cancelación.
- El modelo solo cambia de estado con métodos de transición. Si la transición no está permitida lanza
  InvalidStateTransitionException y no modifica nada.
- Un evento de Kafka puede llegar antes que la respuesta REST: HELD puede pasar directamente a PHONE_REQUESTED, y la
  respuesta REST tardía ya no aplica y se ignora.
- Un teléfono rechazado no es final: PHONE_REJECTED vuelve a PHONE_SUBMITTED hasta que el proceso venza.
- Si se pierde un evento, la consulta REST puede llevar WAITING_FOR_PHONE, PHONE_REQUESTED o PHONE_SUBMITTED a CONFIRMED.
- STARTED se guarda antes de llamar a la cátedra: si se pierde la respuesta o se cae el servicio, queda un registro
  recuperable. Solo puede pasar a HELD o a FAILED.
- Quien procesa un evento consulta canTransitionTo: si no corresponde, el evento es tardío o repetido y se ignora. Un
  comando inválido del usuario se informa como conflicto.

## Alternativas descartadas
- Un orden lineal en el que el estado solo avanza: impide reenviar el teléfono tras un rechazo.
- Indicadores booleanos sueltos (confirmado, vencido...): permiten estados imposibles, como confirmado y vencido a la vez.
- Dejar cancelar procesos que no están confirmados: el enunciado cancela reservas confirmadas.

## Consecuencias
- El diagrama de estados debe mantenerse junto con el enumerado.
- Los casos de uso tienen que decidir si ignoran un evento o informan un error.
- La tabla de transiciones se verifica con tests unitarios.