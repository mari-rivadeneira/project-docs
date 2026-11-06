# 015 - Alcance de la seguridad de la API

## Contexto
La sección 9 del enunciado exige impedir el acceso anónimo, validar el JWT, no confiar en identificadores enviados por
el cliente, configurar CORS y documentar las decisiones y limitaciones.

## Decisión
- Sin sesión ni cookies: cada pedido trae su JWT en la cabecera Authorization.
- CSRF desactivado: protege a las aplicaciones que identifican al usuario con una cookie de sesión, que un sitio ajeno
  puede hacer enviar sin que el usuario lo note. Esta API no usa cookies y un sitio ajeno no puede agregar la cabecera
  Authorization. Es lo que recomienda Spring Security para APIs con token.
- Todo exige JWT salvo POST /api/register y POST /api/authenticate. El usuario sale siempre del claim userId, nunca
  del cuerpo ni de la URL.
- Seguridad por método habilitada (@EnableMethodSecurity), para pedir roles con @PreAuthorize cuando haga falta.
- Los 401 y 403 salen en problem+json y no revelan el motivo (token vencido, mal firmado o ausente); la cabecera
  WWW-Authenticate trae solo Bearer.
- CORS: ningún origen permitido por defecto, con una lista configurable (CORS_ALLOWED_ORIGINS). El único cliente es la
  aplicación Android, que no es un navegador.

## Alternativas descartadas
- Activar CSRF e ignorarlo para /api/**: no protege nada y agrega configuración sin uso.
- Mostrar el motivo del rechazo del token: el cliente actúa igual en todos los casos y se facilitaría probar tokens.

## Consecuencias y limitaciones conocidas
- Los tokens no se pueden revocar antes de vencer y no hay refresh tokens ni límite de intentos de ingreso.
- Hoy el servicio corre sobre HTTP en el entorno de desarrollo; proteger el canal entre la aplicación y los servicios
  (HTTPS) queda como pendiente de despliegue.
- Si se suma un cliente web, hay que cargar su origen en CORS_ALLOWED_ORIGINS.