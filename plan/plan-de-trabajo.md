# Plan de trabajo - Backend

Objetivo: tener **service-catalog** y **service-turns** funcionando para regularizar (límite: 30/11/2026).
La app KMP Android no es requerida para regularizar: se desarrolla para el examen final.

Última actualización: 08/11/2026

## Calendario

| Período | service-catalog | service-turns |
|---|---|---|
| 04/11 - 05/11 | Reconciliación al arrancar y periódica; primera corrida real contra la cátedra | |
| 06/11 - 08/11 | Usuarios, login y JWT | |
| 09/11 - 10/11 | API de consulta y filtros locales | |
| 11/11 | Dockerfile, README y contrato que consume turns | |
| 12/11 - 13/11 | | Base del servicio, dominio y migraciones, persistencia JPA |
| 14/11 - 15/11 | | Clientes: REST de la cátedra y de service-catalog |
| 16/11 | | Seguridad: JWT de usuario y entre servicios |
| 17/11 - 18/11 | | Disponibilidad; iniciar reserva (hold y confirmación inicial) |
| 19/11 - 21/11 | | Kafka: eventos de acciones y envío del teléfono |
| 22/11 - 23/11 | | Mis reservas, consulta del proceso y cancelación |
| 24/11 - 25/11 | | Recuperación: expiración y reinicio con procesos pendientes |
| 26/11 - 28/11 | Evidencias, documentación, correcciones y demo contra el servidor (ambos servicios) | |
| 29/11 - 30/11 | Colchón | |

## service-catalog

- [x] Proyecto Spring Boot
- [x] Docker Compose y persistencia (PostgreSQL, Flyway)
- [x] Modelos de dominio y esquema inicial
- [x] Adaptadores de persistencia JPA y puertos de salida
- [x] Cliente REST del snapshot
- [x] Sincronización completa por snapshot
- [x] Alineación del modelo y el esquema con el contrato de la cátedra
- [x] Adaptador Redis (historial incremental)
- [x] Consumidor Kafka y sincronización incremental con idempotencia
- [x] Reconciliación al arrancar y periódica
- [x] Usuarios finales, autenticación y JWT
- [x] API de consulta y filtros locales
- [ ] Empaquetado (Dockerfile) y README

## service-turns

- [ ] 1. Base del servicio: Compose (PostgreSQL propio y Kafka local), dependencias, estructura hexagonal
- [ ] 2. Dominio y migraciones: proceso de reserva y reserva con dueño, máquina de estados con estados finales que no retroceden, eventos procesados
- [ ] 3. Persistencia JPA y tests de integración
- [ ] 4. Clientes: REST de la cátedra (holds, confirmación, ocupaciones, reservas, cancelación) y de service-catalog
- [ ] 5. Seguridad: JWT de usuario, comunicación entre servicios, autorización por dueño
- [ ] 6. Disponibilidad: horarios semanales menos ocupaciones
- [ ] 7. Iniciar reserva: hold, persistencia del proceso y confirmación inicial
- [ ] 8. Kafka: consumo de eventos de acciones (idempotente) y envío del teléfono
- [ ] 9. Mis reservas, consulta del proceso y cancelación
- [ ] 10. Recuperación: expiración de procesos y reinicio con pendientes

## Hitos de control

| Fecha | Debe estar listo |
|---|---|
| 11/11 | service-catalog completo, probado contra la cátedra |
| 18/11 | service-turns puede crear y confirmar inicialmente un hold |
| 21/11 | Reserva completa por REST y Kafka contra el servidor |
| 25/11 | Todas las funcionalidades de service-turns |
| 28/11 | Evidencias y documentación al día |

