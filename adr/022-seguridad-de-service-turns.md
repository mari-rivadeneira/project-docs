# 022 - Seguridad de service-turns: validación del JWT, identidad desde el token y autorización por dueño

## Contexto
La sección 9 del enunciado exige validar el JWT en las APIs protegidas y en la comunicación entre los dos servicios,
autorizar cada operación según la identidad autenticada, impedir que un usuario acceda a procesos y reservas de otro, no confiar
en identificadores enviados por el cliente y configurar CORS. service-catalog registra a los usuarios y emite los tokens (ADR 009).

## Decisión
- service-turns no emite tokens: los valida con el mismo secreto y el mismo algoritmo que service-catalog (HS512, ADR 012).
  El decodificador solo acepta HS512, y el secreto es obligatorio, sin valor por defecto.
- No hay rutas públicas: todo exige un JWT válido. El registro y el ingreso viven en service-catalog.
- La identidad sale siempre del claim userId (AuthenticatedUser), nunca del cuerpo, la URL ni un parámetro. Un token con
  firma válida pero sin userId se rechaza con 401. El objeto no expone el token en su representación de texto, para que
  no llegue a los logs. El token se conserva para reenviarlo a service-catalog (ADR 021).
- La propiedad por usuario se aplica en la consulta: el proceso se busca por id y por el usuario del token. El proceso de
  otro usuario responde 404 con el mismo cuerpo que uno inexistente, así que no se revela que existe.
- Seguridad por método habilitada para pedir roles con @PreAuthorize cuando haga falta. El rol administrador del
  enunciado es opcional y no se implementa.
- Los 401 y 403 salen en problem+json sin el motivo del rechazo; los errores de integración tienen un code estable:
  409 INVALID_STATE_TRANSITION y CONCURRENT_UPDATE, 503 CATALOG_UNAVAILABLE y CATEDRA_UNAVAILABLE, y 502 CATALOG_ERROR y
  CATEDRA_ERROR. Un error inesperado responde 500 con un texto genérico.
- Sin sesión ni cookies y con CSRF desactivado, con la misma justificación que en service-catalog (ADR 015). CORS no permite
  ningún origen por defecto, con una lista configurable.
- El transporte es HTTP, según lo acordado con el profesor (ADR 015).

## Alternativas descartadas
- Que service-turns le pregunte a service-catalog si el token es válido: acopla su disponibilidad a la de service-catalog, y
  el enunciado pide validar firma, vigencia y permisos localmente.
- Responder 403 ante el proceso de otro usuario: confirma que existe.
- Tomar el id de usuario de un parámetro o del cuerpo: el enunciado lo prohíbe.
- Declarar rutas públicas por conveniencia: no hay ninguna necesidad.

## Consecuencias
- El secreto es compartido: quien lo conozca puede fabricar tokens, y los dos .env deben cuidarse igual.
- Si service-catalog cae, los tokens ya emitidos siguen validándose en service-turns, pero nadie puede iniciar sesión nueva.
- Los tokens no se pueden revocar antes de vencer; Spring tolera por defecto 60 segundos de diferencia de reloj al validar el vencimiento.
- Cada endpoint nuevo debe buscar siempre por el usuario del token; un test del conjunto lo verifica.