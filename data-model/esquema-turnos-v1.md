# Esquema de Base de Datos - service-turns (Versión 1)

Base PostgreSQL propia de service-turns, con su propio usuario y sus propias migraciones (Flyway). service-catalog no
accede a ella ni ella a la de service-catalog.

```mermaid
erDiagram
    reservation_processes {
        UUID id PK
        BIGINT version
        BIGINT user_id
        VARCHAR external_patient_id
        BIGINT professional_id
        VARCHAR professional_first_name
        VARCHAR professional_last_name
        DATE appointment_date
        TIME start_time
        TIME end_time
        VARCHAR patient_first_name
        VARCHAR patient_last_name
        VARCHAR patient_phone
        VARCHAR catedra_process_id UK
        VARCHAR hold_id
        VARCHAR request_event_id
        BIGINT reservation_id UK
        VARCHAR status
        VARCHAR status_reason
        TIMESTAMPTZ expires_at
        TIMESTAMPTZ created_at
        TIMESTAMPTZ updated_at
        TIMESTAMPTZ confirmed_at
        TIMESTAMPTZ cancelled_at
    }
    processed_events {
        VARCHAR event_id PK
        TIMESTAMPTZ processed_at
    }
```

## Tablas
- `reservation_processes`: el proceso de reserva de un usuario, desde que inicia hasta que queda confirmado, cancelado,
  vencido, inválido o fallido. Una reserva confirmada es una fila en estado CONFIRMED.
- `processed_events`: eventos de Kafka ya procesados, para descartar los repetidos.

## Reglas
- `user_id` es el dueño y sale del token. Las consultas siempre filtran por él.
- Los datos del profesional (nombre y apellido) son históricos, no el catálogo vigente.
- Restricciones: estados válidos, `end_time` posterior a `start_time`, identificadores de la cátedra pasado el estado STARTED,
  `reservation_id` en CONFIRMED y CANCELLED, y teléfono y `request_event_id` en PHONE_SUBMITTED.
- Índice único parcial: un solo CONFIRMED por profesional, fecha y hora de inicio.
- `version` implementa el bloqueo optimista.