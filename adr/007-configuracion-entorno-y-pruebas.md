# 007 - Configuración externalizada, entorno local y estrategia de pruebas

## Contexto
El enunciado exige no publicar secretos, direcciones ni tokens, y probar contra REST, Redis y Kafka.

## Decisión
- Toda la configuración de cátedra (URL, JWT, Redis, Kafka) se lee de variables de entorno; el repositorio no contiene valores reales.
- El acceso al servidor de la cátedra es por la red ZeroTier.
- Docker Compose levanta PostgreSQL, Redis y Kafka (KRaft) para desarrollo y pruebas locales.
- Las pruebas de integración corren contra ese compose, sin Testcontainers. El test de Redis usa un prefijo propio (test:catedra:sync) para no tocar nunca las claves reales.
- La lógica de orquestación se prueba con tests unitarios con Mockito; la atomicidad, el rollback y la concurrencia, con tests de integración.

## Consecuencias
Los tests de integración requieren el compose activo y reescriben las tablas de catálogo de la base local.