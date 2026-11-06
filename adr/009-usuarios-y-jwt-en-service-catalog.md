# 009 - Usuarios finales y emisión del JWT en service-catalog

## Contexto
El enunciado exige que los usuarios finales se registren e inicien sesión "contra el backend desarrollado por el
alumno", que se emita un JWT y que los dos servicios lo validen (secciones 3.2 y 9). No dice en cuál de los dos
servicios viven los usuarios.

## Decisión
service-catalog registra y autentica a los usuarios y emite el JWT. service-turns solo lo valida con el mismo secreto y
toma la identidad del claim userId para asociar y aislar las reservas; no guarda usuarios.

## Alternativas descartadas
- Usuarios en service-turns: es defendible, porque las reservas pertenecen al usuario, pero la API de búsqueda del
  catálogo también es protegida y habría quedado con un token que nadie emite hasta terminar service-turns, sin poder
  probarla completa antes.
- Un tercer servicio de identidad: el enunciado exige exactamente dos servicios backend.

## Consecuencias
- service-catalog combina catálogo y autenticación; se mitiga separando lo técnico en infrastructure/security y los
  casos de uso en application.
- Si service-catalog no está disponible nadie puede iniciar sesión, pero los tokens ya emitidos siguen validándose en
  service-turns, porque la validación es local y sin estado.
- Los dos servicios dependen de la misma clave (ver ADR 012).