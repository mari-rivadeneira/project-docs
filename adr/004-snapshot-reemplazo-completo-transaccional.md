# 004 - Snapshot como reemplazo completo transaccional

## Contexto
El enunciado exige reconstruir la copia local sin corromper el estado y registrar exactamente la versión del snapshot.

## Decisión
- Una única transacción: bloqueo de `sync_state`, borrado en orden inverso de FK, reinserción en orden de FK y registro de la versión.
- Se adopta la versión del snapshot tal cual, aunque sea menor que la local.
- Un snapshot con colecciones nulas se rechaza: null no equivale a vacío y vaciaría el catálogo.
- La descarga HTTP ocurre fuera de la transacción.
- Los borrados se hacen en batch, para que se ejecuten antes de reinsertar los mismos IDs.

## Alternativas descartadas
- Upsert sin borrar: deja entidades huérfanas que cátedra ya eliminó.
- Tablas de staging con intercambio: más complejidad sin necesidad.

## Consecuencias
Atomicidad verificada con un test de integración de rollback. Mientras dura la transacción, las lecturas ven la copia anterior.