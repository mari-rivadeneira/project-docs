# 005 - Sincronización incremental con recuperación por snapshot

## Contexto
Kafka solo notifica; la fuente del catálogo es Redis. Las notificaciones pueden duplicarse, perderse o llegar tarde.

## Decisión
- El caso de uso incremental no es transaccional: lee Redis y delega cada versión a un caso de uso que abre su propia transacción.
- Cada versión toma un bloqueo pesimista sobre `sync_state`, verifica continuidad (local = versión - 1) y referencias, hace upsert conservando las bajas lógicas y avanza la versión.
- Una versión menor o igual a la local se ignora.
- Se recupera con snapshot si nunca se sincronizó, si la versión local quedó fuera de la ventana de Redis, o si falta historial o es inconsistente (CatalogHistoryUnavailableException).
- Si Redis no responde (CatedraRedisException) no se hace snapshot: el error se propaga para reintentar.
- Se lee cada entidad con HGET por ID, nunca con HGETALL.

## Alternativas descartadas
- Una sola transacción para todo el rango: pierde el progreso ante un fallo y bloquea más tiempo.
- Snapshot siempre: costoso e innecesario.

## Consecuencias
El progreso parcial se conserva, la recuperación es automática y los casos están cubiertos por tests unitarios y de integración.