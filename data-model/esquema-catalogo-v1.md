# Esquema de Base de Datos - Módulo Catálogo (Versión 1)

## Modelo de Datos - Catálogo (v1)

```mermaid
erDiagram
    professional_categories ||--o{ professionals : "ON DELETE RESTRICT"
    professionals ||--o{ weekly_schedules : "ON DELETE CASCADE"

    professional_categories {
        UUID id PK
        VARCHAR name "UNIQUE"
        TEXT description
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    professionals {
        UUID id PK
        VARCHAR first_name
        VARCHAR last_name
        VARCHAR email "UNIQUE"
        VARCHAR phone
        UUID category_id FK
        BOOLEAN active
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    weekly_schedules {
        UUID id PK
        UUID professional_id FK
        VARCHAR day_of_week
        TIME start_time
        TIME end_time
        INT slot_duration_minutes
    }

    sync_state {
        VARCHAR entity_name PK
        BIGINT version
        TIMESTAMP last_synced_at
    }

   
      
```

---

## Tablas principales

* **`professional_categories`:** Almacena las distintas categorías y especialidades que agrupan a los profesionales.
* **`professionals`:** Registra la información personal, estado y vías de contacto de cada profesional, vinculándose a una categoría.
* **`weekly_schedules`:** Define las franjas horarias semanales y la duración de turnos asignadas a cada profesional.
* **`sync_state`:** Lleva el control de versiones y marcas temporales de sincronización con otros módulos.
*ay para versionar y auditar la ejecución de las migraciones SQL.

