# 025 - Eventos de la cátedra y envío del teléfono por Kafka

## Contexto
El resultado de una reserva llega de forma asincrónica por Kafka: la cátedra pide el teléfono, lo confirma, lo rechaza, o informa un
vencimiento, un proceso inválido o una cancelación. La entrega es al menos una vez, los mensajes pueden repetirse o llegar tarde, y
el enunciado exige idempotencia, no retroceder un proceso que ya alcanzó un estado final y recuperarse sin corromper el estado local.

## Decisión
- Consumidor del topic de acciones: la clave es el reservationProcessId, así que los eventos de un proceso conservan el orden.
  Se interpretan los seis tipos del contrato ignorando campos desconocidos. Un mensaje ilegible, incompleto, de un tipo desconocido o
  de otra versión de esquema se descarta y se confirma su offset: reintentarlo no lo arregla y bloquearía la partición.
- Aplicar un evento es una única transacción: se busca el proceso, se registra el eventId en processed_events y se aplica el cambio.
  Si el cambio falla, el registro se deshace y el evento se puede reprocesar (ADR 020).
- La máquina de estados decide si el evento aplica. Uno tardío, repetido con otro eventId o que llega a un estado final se ignora, pero
  queda registrado: su segunda entrega es un duplicado. Así una confirmación tardía no reabre un proceso vencido, un vencimiento tardío
  no afecta a una reserva confirmada y la cancelación que hizo este mismo servicio no se aplica dos veces.
- Un evento de un proceso desconocido se descarta con un aviso y no se registra: no hay a qué aplicarlo, y la reconciliación por REST
  es el camino de recuperación.
- Los conflictos de versión entre un evento y un pedido REST se reintentan hasta 3 veces fuera de la transacción: un conflicto la
  deja marcada para deshacerse, así que cada intento empieza una nueva. Otros fallos (la base caída) los reintenta Kafka: 4 veces
  con 2 segundos de espera, y después se registran y se continúa.
- El offset se confirma recién después de persistir.
- Teléfono: se publica primero y se registra después. Si la publicación falla, no se guardó nada y el usuario reintenta sin riesgo. Si
  se publicó pero el registro falla, el proceso sigue en PHONE_REQUESTED y converge cuando llega AppointmentConfirmed, porque la
  máquina de estados admite ese salto. La cátedra tolera un nuevo envío con un eventId nuevo.
- El productor espera la confirmación del broker, con tiempos acotados (3 s de metadata y 10 s de entrega) para no colgar el pedido.
- Repetir el mismo teléfono mientras se espera el resultado no vuelve a publicar; otro teléfono solo se acepta tras un rechazo.
- El teléfono no se escribe en los logs ni se devuelve en la API.

## Alternativas descartadas
- Registrar el evento antes de aplicarlo, en otra transacción: si el cambio falla, el evento ya figura como procesado y se pierde.
- Rechazar o reencolar los eventos que no corresponden al estado: no son errores, y reintentarlos no cambia nada.
- Guardar primero el teléfono y publicar después con un proceso aparte (outbox): es más robusto, pero agrega una tabla y un proceso de
  reenvío; la convergencia por el evento de confirmación y la recuperación por REST alcanzan para este alcance.
- Un consumidor distinto por tipo de evento: se pierde el orden entre eventos de un mismo proceso.

## Consecuencias
- Si el teléfono se publica y el registro falla, el usuario ve un error aunque el envío funcionó: al consultar, el estado converge.
- processed_events crece con cada evento: no hay purga (límite conocido, se resolvería con retención por fecha).
- Los eventos de un proceso que nunca se guardó localmente (por ejemplo, un hold huérfano) se descartan.
- Si el consumer group es el mismo que el de service-catalog, los dos servicios se provocan rebalanceos al reiniciarse.