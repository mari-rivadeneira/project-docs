# API de service-turns (consumida por la aplicación)

Estado: en elaboración (se completa con cada funcionalidad)

## Autenticación
Todas las operaciones exigen `Authorization: Bearer <JWT>` con el token que emite service-catalog (ADR 012 y 022). Sin token, con
uno inválido o vencido, la respuesta es 401 con code UNAUTHORIZED.

## Disponibilidad

`GET /api/availability?professionalId=15&date=2026-12-07`

Turnos libres de un profesional en una fecha: agendas vigentes de service-catalog menos las ocupaciones de la cátedra. Es
informativo: al crear el hold todavía se puede perder la carrera por un turno.

Respuesta 200:
```json
{ "professionalId": 15, "date": "2026-12-07",
  "slots": [ { "startTime": "09:30:00", "endTime": "10:00:00" }, { "startTime": "10:30:00", "endTime": "11:00:00" } ] }
```

| Status | code | Cuándo |
|---|---|---|
| 400 | VALIDATION_ERROR | Fecha anterior a hoy o profesionalId inválido (con `fieldErrors`) |
| 400 | BAD_REQUEST | Parámetro faltante o con formato incorrecto |
| 401 | UNAUTHORIZED | Token ausente, inválido o vencido |
| 404 | PROFESSIONAL_NOT_FOUND | El profesional no existe |
| 409 | PROFESSIONAL_DISABLED | El profesional está deshabilitado |
| 503 | CATALOG_UNAVAILABLE | service-catalog no responde |
| 503 | CATEDRA_UNAVAILABLE | La cátedra no responde |
| 502 | CATEDRA_ERROR, CATALOG_ERROR | Rechazo inesperado de la cátedra o de service-catalog |

## Reservas
(pendiente: se documenta junto con su implementación)