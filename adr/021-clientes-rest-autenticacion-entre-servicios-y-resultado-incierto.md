# 021 - Clientes REST: autenticación entre servicios, tiempos de espera y resultado incierto

## Contexto
service-turns llama a la API REST de la cátedra (holds, confirmación, ocupaciones, reservas, cancelación) y a service-catalog.
El enunciado exige definir si entre servicios se propaga el JWT del usuario o se usa uno técnico (sección 9), definir
timeouts, reintentos y traducción de errores (sección 8), y no repetir operaciones por un timeout sin saber si se
ejecutaron. Si service-catalog no está disponible, no pueden iniciarse operaciones que requieran validar información vigente.

## Decisión
- A la cátedra se envía el JWT técnico del proyecto, que viene de una variable de entorno. A service-catalog se reenvía el
  JWT del usuario que originó la operación, como parámetro explícito de cada llamada. Así service-catalog ve la identidad real
  y la trazabilidad se conserva. Cada servicio valida firma, vigencia y permisos del token que recibe.
- Los puertos devuelven tipos del dominio; los DTO del contrato quedan en infraestructura y ignoran los campos desconocidos.
- Los errores se traducen a dos familias. Un rechazo (4xx) es una respuesta firme: la operación no tuvo efecto, y se conserva
  el status y el code de la cátedra. Una indisponibilidad (red, timeout, 5xx o respuesta inválida) lleva la indicación
  outcomeUnknown: en una operación con efectos, el pedido pudo haberse ejecutado y hay que consultar o reconciliar antes de
  repetirlo. Es falsa cuando el pedido seguro no salió (conexión rechazada o timeout al conectar) y en las consultas.
- Los clientes no reintentan. La política de reintentos pertenece a los casos de uso, que conocen el significado de cada
  operación: las consultas se reintentan de forma acotada y las operaciones con efectos se reconcilian primero.
- Tiempos de espera configurables: la cátedra, 5 s para conectar y 10 s para leer; service-catalog, 2 s y 5 s.
- Un estado de ocupación desconocido se trata como turno ocupado; un estado de reserva desconocido se informa como
  indisponibilidad.

## Alternativas descartadas
- Un JWT técnico entre servicios: se pierde la identidad del usuario y la trazabilidad, y exige otro secreto o rol.
- Obtener el token del contexto de seguridad dentro del cliente: acopla el cliente al hilo de la petición y lo oculta; el
  parámetro explícito es más claro y más fácil de probar.
- Reintentos automáticos en el cliente: un POST repetido a ciegas puede duplicar holds.
- Una única excepción para todos los errores: obligaría a repetir la distinción entre rechazo e indisponibilidad en cada caso de uso.

## Consecuencias
- Los casos de uso deben decidir qué hacer ante cada excepción: mostrar el error, reintentar una consulta o reconciliar.
- Los procesos sin usuario (recuperación y vencimiento) no llaman a service-catalog.
- Si el JWT del usuario vence entre dos llamadas, service-catalog lo rechaza con 401: se informa al usuario.
- Distinguir si el pedido no salió depende del tipo y del mensaje de la excepción de red del cliente HTTP; ante la duda se
  asume que pudo ejecutarse.