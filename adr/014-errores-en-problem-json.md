# 014 - Errores de la API en problem+json con un handler global

## Contexto
Todos los errores deben tener el mismo formato y un código estable, y las excepciones de negocio no deben contener lógica HTTP.

## Decisión
- Un @RestControllerAdvice (GlobalExceptionHandler) traduce las excepciones a application/problem+json con title,
  status, detail, path y code, más fieldErrors en las validaciones. El cliente decide por status y code, nunca por detail.
- Códigos: VALIDATION_ERROR, LOGIN_ALREADY_USED y EMAIL_ALREADY_USED (400), BAD_REQUEST (400), INVALID_CREDENTIALS y
  UNAUTHORIZED (401), FORBIDDEN (403), NOT_FOUND (404), METHOD_NOT_ALLOWED (405) e INTERNAL_ERROR (500).
- El mismo handler traduce los errores estándar de Spring (ruta inexistente, método no permitido, tipo de contenido no
  soportado) para que tengan code y detalle en español. Un error inesperado responde 500 con un texto genérico, sin
  exponer mensajes internos, y se registra en el log.
- Los 401 y 403 de los filtros de seguridad los escribe ProblemDetailsSecurityHandler con el mismo formato, porque
  esos filtros cortan el pedido antes de llegar a los controllers y el handler global no los ve.

## Alternativas descartadas
- try/catch en cada método del controlador: repite código y deja fuera los errores que Spring lanza antes del controlador.
- Un handler por funcionalidad: tiene sentido con muchos módulos; en un servicio acotado, uno global es más fácil de seguir.

## Consecuencias
Una excepción nueva necesita su método en el handler; sin él termina como 500. Spring agrega algunos campos propios a
ciertas respuestas, y los clientes ignoran lo que no conocen.