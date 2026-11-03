# 006 - Idempotencia y consumo de eventos Kafka

## Contexto
La entrega de Kafka es al menos una vez: pueden llegar duplicados, mensajes malformados o fallos transitorios.

## Decisión
- Doble barrera: la comparación de versiones vuelve inofensivo un evento repetido, y `processed_events` evita repetir trabajo.
- El eventId se registra después de sincronizar, no antes: si el proceso cae a mitad, el evento se reprocesa sin efecto.
- Ack manual: el offset se confirma solo después de persistir.
- Los mensajes ilegibles, incompletos o de otro tipo se descartan confirmando el offset, para no bloquear la partición.
- Los fallos recuperables no confirman el offset; se reintenta con espera fija de 2 segundos hasta 4 veces.
- Los campos desconocidos del evento se ignoran (compatibilidad con schemaVersion 1).

## Alternativas descartadas
- Registrar el eventId en la misma transacción que el avance: acopla Kafka al esquema sin aportar garantías, porque la versión ya es idempotente.
- Cola de mensajes fallidos (dead-letter): fuera del alcance de esta etapa.

## Consecuencias
Se tolera la entrega duplicada y los reinicios. Un evento descartado tras agotar los reintentos queda en el log y se recupera con la reconciliación por versión.