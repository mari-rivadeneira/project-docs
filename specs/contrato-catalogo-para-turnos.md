# Contrato: service-catalog consumido por service-turns

Estado: en elaboración (se confirma al implementar el consumo en service-turns)

## Autenticación
Todas las operaciones exigen la cabecera `Authorization: Bearer <JWT>`. Se prevé que service-turns reenvíe el JWT del
usuario, emitido por service-catalog y validado con el secreto compartido (ADR 012). Sin token, o con uno inválido o
vencido, la respuesta es 401 con code UNAUTHORIZED.

## Operaciones

| Método | Ruta | Uso para turnos | Respuesta |
|---|---|---|---|
| GET | /api/catalog/professionals/{id} | Validar que el profesional existe y está habilitado | Profesional |
| GET | /api/catalog/professionals/{id}/schedules | Obtener las reglas semanales para construir la disponibilidad | Lista de agendas |
| GET | /api/catalog/professionals | Búsqueda con filtros (para la aplicación) | Página de profesionales |
| GET | /api/catalog/categories | Lista de categorías (para la aplicación) | Lista de categorías |
| GET | /api/catalog/sync/status | Diagnóstico de la sincronización | Estado |

Filtros de la búsqueda: `categoryId`, `name`, `enabled`, `hasSchedules`, `dayOfWeek` (MONDAY a SUNDAY, en cualquier
combinación de mayúsculas), `page` (desde 0) y `size` (de 1 a 100, 20 por defecto).

## DTO

Profesional:
```json
{ "id": 15, "categoryId": 1, "firstName": "Ana", "lastName": "Gomez", "enabled": true,
  "createdAt": "2026-07-01T12:00:00Z", "updatedAt": "2026-09-10T13:40:00Z" }
```

Agenda semanal (incluye las deshabilitadas, ordenadas por día y hora de inicio):
```json
{ "id": 21, "professionalId": 15, "dayOfWeek": "MONDAY", "startTime": "09:00:00", "endTime": "12:00:00",
  "slotDurationMinutes": 30, "enabled": true,
  "createdAt": "2026-07-01T12:00:00Z", "updatedAt": "2026-09-10T13:40:00Z" }
```

Página: `{ "content": [ ... ], "page": 0, "size": 20, "totalElements": 3, "totalPages": 1 }`

Estado: `{ "version": 34, "lastSyncedAt": "2026-11-05T18:37:21Z", "lastMode": "SNAPSHOT", "lastError": null }`

## Errores
Formato `application/problem+json` con `status`, `title`, `detail`, `path` y `code`. Se decide por `status` y `code`,
nunca por `detail`.

| Status | code | Cuándo |
|---|---|---|
| 400 | VALIDATION_ERROR | Filtros o paginación inválidos (con `fieldErrors`) |
| 400 | BAD_REQUEST | Un parámetro con formato incorrecto (por ejemplo un id que no es número) |
| 401 | UNAUTHORIZED | Token ausente, inválido o vencido |
| 403 | FORBIDDEN | Sin permisos para la operación |
| 404 | NOT_FOUND | El profesional no existe |
| 500 | INTERNAL_ERROR | Error inesperado |

## Comportamiento ante indisponibilidad
Si service-catalog no responde, service-turns no inicia operaciones que requieran validar información vigente y responde 503; las operaciones que dependen solo de datos propios siguen funcionando.