# 012 - JWT firmado con HS512 y secreto compartido

## Contexto
service-catalog emite el JWT de usuario y service-turns lo valida. Los dos deben coincidir en la clave sin que viaje
por la red ni se versione.

## Decisión
- Firma HS512 con una clave compartida: JWT_SECRET (Base64, al menos 64 bytes, generada con openssl rand -base64 64),
  en una variable de entorno y sin valor por defecto. Si falta, no es Base64 o es corta, la aplicación no arranca.
  service-turns usa el mismo valor.
- Librería: oauth2-resource-server de Spring Security (Nimbus), para emitir y validar. El decodificador solo acepta
  HS512: un token con otro algoritmo (HS256, none) se rechaza.
- Contenido: sub (login), userId, auth (autoridades separadas por espacio, formato de JHipster), iat y exp. Dura 24 horas,
  o 30 días si el ingreso pide rememberMe (propiedades jwt.token-validity y jwt.remember-me-validity).
- La identidad de cada pedido sale siempre del claim userId.

## Alternativas descartadas
- HS256: alcanza en seguridad, pero HS512 es el algoritmo por defecto de JHipster y con el que firma la cátedra.
- RSA con un par de claves: service-turns solo tendría la clave pública y no podría fabricar tokens, pero hay que generar
  y distribuir archivos PEM; con dos servicios del mismo dueño y entorno se prefirió la simplicidad.
- Una clave con valor por defecto: un despliegue olvidado firmaría con una clave pública.
- Nombre y apellido como claims: quedan fijos al emitir el token.

## Consecuencias
- Quien conozca la clave puede fabricar tokens: los .env de los dos servicios deben cuidarse igual.
- Un token no se puede revocar antes de que venza; la vigencia de 24 horas lo acota.
- Los tests usan un secreto distinto y descartable, en src/test/resources/config.
- Hay tests que rechazan tokens mal formados, vencidos, con otra clave, con HS256, sin firma y con el cuerpo alterado.