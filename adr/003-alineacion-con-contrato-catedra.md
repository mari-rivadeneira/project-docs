# 003 - Alineación del esquema y los modelos con el contrato de cátedra v1

## Contexto
El modelo inicial usaba UUID, `active`, `email` y `phone`. El contrato de cátedra define IDs enteros (int64),
un campo `enabled` en las tres entidades (las deshabilitadas son bajas lógicas), `createdAt` y `updatedAt`,
y no incluye datos de contacto.

## Decisión
- IDs BIGINT asignados por la cátedra, nunca generados localmente.
- `enabled` en las tres tablas y timestamps provistos por la cátedra, sin valor por defecto.
- Se eliminan los campos que el contrato no define y las restricciones UNIQUE sobre nombre y email.
- `sync_state` con una única fila `CATALOG` (la versión de cátedra es global), con `last_mode` y `last_error`.
- La migración V2 recrea las tablas. La V1 no se edita: Flyway valida su checksum.
- En JPA las claves foráneas son columnas simples (sin @ManyToOne): menos consultas y sin proxies; la FK real queda en la base.

## Alternativas descartadas
- Editar la V1: rompe las bases ya migradas.
- Mantener UUID y traducir a Long: agrega una capa de traducción sin beneficio.

## Consecuencias
La copia local es fiel al contrato y el historial de migraciones queda intacto. La V2 descarta los datos
existentes, aceptable porque la copia se reconstruye con un snapshot.