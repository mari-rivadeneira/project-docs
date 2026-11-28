# 027 - Recuperación y vencimiento de reservas

## Contexto
Un proceso de reserva puede quedar a medias por un reinicio, por una respuesta perdida o por un evento de Kafka que nunca llegó (un
vencimiento o una confirmación). El enunciado exige recuperarse de reinicios, mensajes perdidos y vencimientos, sin ciclos de reintento
ilimitados ni estados locales imposibles de recuperar (sección 8).

## Decisión
- Un job se ejecuta al arrancar y cada 60 segundos (configurable, y se puede desactivar). Al arrancar retoma lo que quedó pendiente
  antes de un reinicio.
- Procesos STARTED sin respuesta de la cátedra pasado un tiempo de gracia (5 minutos) se cierran como FAILED con el motivo
  START_ABANDONED. Si la cátedra llegó a crear un hold sin que lo supiéramos, vence por su propia vigencia (ADR 024).
- Los procesos en curso cuya vigencia terminó se reconcilian con la cátedra (ADR 026) en lugar de marcarse vencidos a ciegas: la cátedra
  es la autoridad. Un evento de confirmación perdido converge a CONFIRMED y uno de vencimiento perdido, a EXPIRED.
- Cada proceso se trata por separado. Si la cátedra no responde, se corta la corrida y se reintenta en la siguiente; un fallo puntual
  no detiene a los demás.
- Cada corrida está acotada por un tamaño de lote (50), con los más viejos primero.
- Es seguro con varias instancias: el bloqueo optimista de cada proceso evita que se pisen y reconciliar es idempotente.
- No es transaccional: hay llamadas HTTP, y cada proceso se guarda por su cuenta.

## Alternativas descartadas
- Marcar vencido todo proceso cuya vigencia pasó, sin consultar: una confirmación cuyo evento se perdió quedaría como vencida.
- Un hilo por proceso o una cola de trabajos: es más infraestructura de la que hace falta con este volumen.
- Reintentar de forma continua: un ciclo sin tope contra una cátedra caída.

## Consecuencias
- Un evento perdido se recupera con un retraso de hasta la vigencia del proceso más el intervalo del job; el usuario puede acelerarlo
  con el refresco de la reserva (ADR 026).
- Una reserva confirmada que se cancela fuera de este servicio, sin que llegue el evento, no la recupera el job: la recupera el refresco.
- Si la cátedra siguiera informando PHONE_PENDING más allá de la vigencia, esos procesos se consultarían en cada corrida y podrían
  ocupar el lote; se asume que la cátedra los cierra al vencer.