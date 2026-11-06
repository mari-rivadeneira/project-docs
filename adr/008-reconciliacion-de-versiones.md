# 008 - Reconciliación de versiones al arrancar y de forma periódica

## Contexto
Kafka solo notifica y una notificación puede perderse, o el servicio puede haber estado detenido mientras la cátedra
publicaba versiones. Con solo el consumidor, el atraso dura hasta el próximo evento y una base vacía no se inicializa
hasta que llegue uno. El enunciado exige detectar y recuperar la pérdida de cambios.

## Decisión
- Un caso de uso (ReconcileCatalogUseCase) hace un intento: sin copia local reconstruye con snapshot; si la versión
  publicada en Redis es menor a la local o no se puede leer, reconstruye con snapshot; si hay atraso, dispara la
  sincronización incremental hasta la versión publicada; si coinciden, no hace nada.
- Un servicio de aplicación (CatalogReconciliationService) envuelve ese intento con reintentos acotados (4 por defecto)
  y espera creciente (2 s, duplicando hasta 30 s), y registra el error en sync_state.last_error. Si agota los intentos,
  abandona y espera la próxima ejecución.
- Registrar el error es un UPDATE de esa sola columna: guardar el estado completo, leído antes, podría pisar la versión.
- Corre al arrancar (ApplicationReadyEvent) y cada 5 minutos (fixedDelay, configurable). El retardo inicial de la tarea
  periódica es igual al intervalo, para no solaparse con la del arranque.
- Se activa con CATALOG_RECONCILIATION_ENABLED (apagada por defecto) para que los tests no consulten a la cátedra.
- No hay exclusión mutua con el consumidor de Kafka: conviven sin riesgo porque cada versión se aplica bajo el bloqueo
  de sync_state y la comparación de versiones vuelve inofensiva la repetición.

## Alternativas descartadas
- Solo el consumidor Kafka: no cubre notificaciones perdidas ni la base vacía.
- Candado en memoria entre la tarea y el consumidor: no protege con más de una instancia y el bloqueo en la base ya
  resuelve el problema.
- Reintentos sin tope: el enunciado pide evitar ciclos ilimitados.
- Reconstruir con snapshot cuando Redis no responde: no arregla un Redis caído y recarga la API de la cátedra.

## Consecuencias
Con la base vacía la primera sincronización no espera ningún evento. Tras agotar los reintentos el atraso puede durar
hasta el siguiente intervalo. La tarea periódica agrega una lectura a Redis por intervalo. last_error queda disponible
para el endpoint de estado.