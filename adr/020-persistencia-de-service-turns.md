# 020 - Persistencia de service-turns: bloqueo optimista y registro atómico de eventos

## Contexto
Los eventos de Kafka y los pedidos REST pueden modificar el mismo proceso de reserva casi a la vez, y los eventos pueden
llegar repetidos. El enunciado exige idempotencia, convergencia sin reservas duplicadas y que cada usuario solo vea lo suyo.

## Decisión
- Bloqueo optimista con una columna de versión: si dos operaciones leen el mismo proceso y las dos lo modifican, la segunda
  falla con ConcurrentUpdateException en lugar de pisar el cambio de la primera. Quien la recibe vuelve a leer y reintenta.
- Dos operaciones separadas en el puerto: create (inserta con versión 0) y update (modifica con la versión leída). Con un
  id generado por el servicio, un único guardar de Spring Data no distingue entre insertar y actualizar. update fuerza el
  guardado (saveAndFlush) para que el conflicto se vea en el adaptador y no recién al cerrar una transacción externa.
- La propiedad por usuario se aplica en la consulta: findByIdAndUserId devuelve vacío si el proceso es de otro usuario, así
  que no se puede distinguir de uno inexistente (404 en lugar de 403, sin revelar que existe).
- Los eventos procesados se registran con INSERT ... ON CONFLICT DO NOTHING: es atómico, no lanza excepción si el evento ya
  estaba y se puede llamar dentro de la transacción que aplica el cambio, de modo que el cambio de estado y el registro del
  evento se confirman juntos o ninguno.
- Los estados que pueden vencer se derivan de la máquina de estados (los que admiten la transición a EXPIRED), no de una
  lista aparte.
- Las pruebas de los adaptadores corren contra PostgreSQL real y sin transacción de prueba, para que cada operación confirme
  su transacción y se pueda probar el conflicto de versiones y la concurrencia.

## Alternativas descartadas
- Bloqueo pesimista (SELECT ... FOR UPDATE): service-catalog lo usa porque serializa una única fila global; acá cada proceso
  casi nunca tiene dos escritores a la vez, y reintentar es barato.
- Un solo método para insertar y actualizar: ambiguo con identificadores asignados.
- Consultar si el evento existe y después insertarlo: deja una ventana de carrera entre las dos sentencias.
- Capturar la violación de clave duplicada: deja inutilizable la transacción que la contiene.

## Consecuencias
- Los casos de uso deben manejar ConcurrentUpdateException: releer y reintentar un número acotado de veces.
- Si el conflicto ocurre dentro de una transacción externa, esa transacción queda marcada para deshacerse: el reintento debe
  empezar una nueva.
- Una segunda reserva confirmada para el mismo turno falla por el índice único parcial y se propaga como error de integridad.