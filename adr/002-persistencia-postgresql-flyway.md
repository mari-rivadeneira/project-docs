002 - Uso de PostgreSQL en Docker y control de migraciones con Flyway

El catalogo requiere una base de datos relacional robusta con soporte para identificadores unicos (UUID), restricciones de integridad referencial, indices y control de tipos temporales. Ademas, el esquema debe poder reproducirse de manera automatica y consistente en cualquier entorno de desarrollo o integracion continua.

Decision:
    Se utiliza PostgreSQL como motor de base de datos relacional ejecutado en un contenedor via Docker Compose.
    Se utiliza Flyway para el versionado y ejecucion automatica de scripts SQL de migracion desde el arranque de Spring Boot.
    Se configura Hibernate con ddl-auto en modo validate, delegando la creacion y evolucion del esquema exclusivamente a los scripts de Flyway.

Consecuencias:
    Trazabilidad total de los cambios de esquema en el repositorio, eliminacion de inconsistencias manuales entre entornos y control estricto sobre las restricciones e indices en SQL nativo.

Enmienda: la referencia a identificadores UUID quedó obsoleta. Los IDs pasaron a BIGINT asignados
por la cátedra (ver ADR 003). El uso de PostgreSQL, Flyway y ddl-auto=validate se mantiene.
Migraciones vigentes: V1 (esquema inicial), V2 (alineación con el contrato) y V3 (eventos procesados).