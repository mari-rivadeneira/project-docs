# Documentación del Proyecto - Sistema de Catálogo y Turnos
Rivadeneira Marina

Este repositorio centraliza la documentación técnica, arquitectónica y de diseño del sistema: las decisiones de
ingeniería, los diagramas, los modelos de datos, los contratos entre servicios, el plan de trabajo y las evidencias.
Es un caso de estudio académico con datos ficticios.

Última actualización: 23/11/2026

## Repositorios de código
- service-catalog: https://github.com/mari-rivadeneira/service-catalog
- service-turns: https://github.com/mari-rivadeneira/service-turns
- app-kmp: (pendiente, se desarrolla para el examen final)

## Estructura del repositorio
- `/adr`: registros de decisiones de arquitectura. Cada uno explica el contexto, la decisión, las alternativas
  descartadas y las consecuencias.
- `/diagrams`: diagramas de arquitectura, de secuencia y de estados.
- `/data-model`: esquemas de base de datos de cada servicio.
- `/specs`: contratos entre los servicios y colecciones de Postman.
- `/plan`: plan de trabajo con calendario, hitos y estado.
- `/evidence`: capturas y registros que demuestran el funcionamiento.

## Estado de la documentación frente al enunciado

| Requisito | Documento | Estado |
|---|---|---|
| README con requisitos, configuración y pasos para ejecutar | README de cada repositorio de código | service-catalog listo; service-turns y app-kmp pendientes |
| Diagrama de arquitectura | diagrams/arquitectura-general.md | Parcial (faltan service-turns y la aplicación) |
| Contratos entre los dos servicios | specs/contrato-catalogo-para-turnos.md, ADR 021 | Vigente |
| Modelo de datos de cada servicio | data-model/esquema-catalogo-v2.md, data-model/esquema-turnos-v1.md | Listo |
| Sincronización completa e incremental | ADR 004, 005 y 008 | Listo |
| Máquina de estados local de las reservas | ADR 018, diagrams/maquina-de-estados-reserva.md | Listo |
| Idempotencia y recuperación | ADR 006, 008 y 020 | service-catalog listo; service-turns en curso (falta el consumo de eventos y la recuperación) |
| Seguridad, autenticación y autorización | ADR 009 a 015, 021 y 022 | service-catalog listo; service-turns con la base lista (faltan los endpoints)|
| Instrucciones para ejecutar las pruebas | README de cada repositorio de código | service-catalog listo; service-turns y app-kmp pendientes |

## Índice de decisiones de arquitectura

| N.º | Decisión |
|---|---|
| 001 | Adopción de arquitectura hexagonal |
| 002 | PostgreSQL en Docker y migraciones con Flyway |
| 003 | Alineación del esquema y los modelos con el contrato de cátedra |
| 004 | Snapshot como reemplazo completo transaccional |
| 005 | Sincronización incremental con recuperación por snapshot |
| 006 | Idempotencia y consumo de eventos Kafka |
| 007 | Configuración externalizada, entorno local y estrategia de pruebas |
| 008 | Reconciliación de versiones al arrancar y de forma periódica |
| 009 | Usuarios finales y emisión del JWT en service-catalog |
| 010 | Modelo de usuario compatible con JHipster |
| 011 | Hash de contraseñas con Argon2id |
| 012 | JWT firmado con HS512 y secreto compartido |
| 013 | Autenticación como caso de uso y login que se confunde con email |
| 014 | Errores de la API en problem+json con un handler global |
| 015 | Alcance de la seguridad de la API |
| 016 | Definición del filtro de disponibilidad del catálogo |
| 017 | API de consulta del catálogo local |
| 018 | Máquina de estados del proceso de reserva |
| 019 | Modelo de datos de service-turns |
| 020 | Persistencia de service-turns: bloqueo optimista y registro atómico de eventos |
| 021 | Clientes REST: autenticación entre servicios, tiempos de espera y resultado incierto |
| 022 | Seguridad de service-turns: validación del JWT, identidad desde el token y autorización por dueño                                                                                   |
| 023 | Cálculo de la disponibilidad de turnos                                                                                                                                                                                 |
| 024 | Inicio de la reserva: escritura previa, resultado incierto e idempotencia                                                                                                                                                                                                                       |
| 025  | Eventos de la cátedra y envío del teléfono por Kafka                                                                                                                                                                                                                                                                                                |

## Plan de trabajo
El calendario, los hitos y el avance de cada servicio están en [plan/plan-de-trabajo.md](plan/plan-de-trabajo.md).
La aplicación KMP Android se desarrolla para el examen final; los dos servicios backend, para la regularización.

## Convenciones
- Las decisiones se documentan al terminar cada funcionalidad, para que coincidan con el código entregado.
- Un ADR no se reescribe: si una decisión cambia, se enmienda con una nota o se reemplaza por uno nuevo.
- Ninguna evidencia ni documento incluye tokens, contraseñas ni secretos.