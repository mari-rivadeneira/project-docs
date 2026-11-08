# 017 - API de consulta del catálogo local

## Contexto
El enunciado exige buscar y filtrar profesionales con datos locales por categoría, nombre, estado habilitado y
disponibilidad, y que esos filtros puedan usarse desde la aplicación. service-turns, además, debe obtener del catálogo
la información vigente antes de operar.

## Decisión
- Endpoints, todos protegidos por JWT: GET /api/catalog/professionals (búsqueda con categoryId, name, enabled,
  hasSchedules, dayOfWeek, page y size), GET /api/catalog/professionals/{id}, GET /api/catalog/professionals/{id}/schedules,
  GET /api/catalog/categories y GET /api/catalog/sync/status.
- Los filtros se combinan y los que no llegan no se aplican. El nombre busca en nombre, apellido o "nombre apellido",
  sin distinguir mayúsculas, y el % y el _ que escriba el usuario se buscan como texto, no como comodines.
- El orden es fijo (apellido, nombre e id) para que la paginación sea estable.
- Límites: size de 1 a 100, page de 0 a 100000 y name de hasta 100 caracteres. Un valor fuera de rango o una combinación
  contradictoria responde 400 VALIDATION_ERROR con el detalle por campo, sin ajustarlo en silencio. Un valor con formato
  incorrecto responde 400 BAD_REQUEST y un id inexistente, 404 NOT_FOUND.
- Se devuelven también los profesionales, las categorías y las agendas deshabilitados, con su campo enabled: quien
  consume decide qué mostrar, y service-turns necesita saber si un profesional está deshabilitado para rechazar la reserva.
- GET /api/catalog/sync/status informa la versión, el momento y el modo de la última sincronización y solo el tipo del
  último error, no su texto, porque el texto puede incluir direcciones internas de la cátedra.
- La búsqueda se hace a través de un puerto de salida propio (ProfessionalSearchRepository) y arma la consulta con solo
  los filtros presentes.

## Alternativas descartadas
- Parámetros opcionales con la forma (:p is null or ...): PostgreSQL falla al deducir el tipo de un parámetro nulo en
  funciones como lower().
- Specifications de Spring Data: es lo más habitual, pero con entidades sin relaciones y una sola consulta el JPQL
  dinámico es más corto y más fácil de leer; si los filtros crecieran, conviene migrar.
- Ajustar en silencio size o page: oculta errores del cliente.
- Excluir los deshabilitados por defecto: service-turns y la aplicación perderían información necesaria; el filtro enabled ya existe.
- Consultar a la cátedra en cada búsqueda: la sección 6.2 del enunciado lo prohíbe.

## Consecuencias
- La búsqueda por nombre usa like '%texto%', que no aprovecha índices: con un catálogo chico no importa; si creciera,
  habría que indexar o usar búsqueda de texto.
- Armar la consulta con texto exige cuidado: los valores siempre viajan como parámetros y nunca se concatenan.
- El contrato que consume service-turns está en specs/contrato-catalogo-para-turnos.md.