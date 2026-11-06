# 013 - Autenticación como caso de uso y login que se confunde con email

## Contexto
El ingreso se hace con login o email y contraseña. El patrón de login de JHipster admite arrobas, así que un mismo
texto podía ser el login de un usuario y el email de otro.

## Decisión
- AuthenticateUserUseCase busca por login y por email (sin distinguir mayúsculas ni espacios), exige la cuenta activa,
  compara la contraseña con el puerto PasswordHasher y pide el token al puerto TokenIssuer (JwtTokenIssuer en
  infraestructura). No usa el AuthenticationManager de Spring Security.
- Usuario inexistente, contraseña incorrecta y cuenta inactiva dan el mismo 401 INVALID_CREDENTIALS.
- Las entradas vacías o enormes se rechazan sin consultar la base ni ejecutar el hash.
- Colisión: el registro rechaza un login igual al email de otro usuario y un email igual al login de otro. Como respaldo,
  al ingresar se prueba la contraseña contra las dos cuentas (login primero) y entra la que acierte; un mismo usuario
  hallado por las dos vías se verifica una sola vez, porque el hash es costoso.

## Alternativas descartadas
- El flujo estándar de Spring (UserDetailsService y AuthenticationManager): resuelve lo mismo, pero lleva clases del
  framework al dominio o duplica al usuario.
- Prohibir la arroba en el login: es lo más simple, pero se aparta del patrón de login de JHipster.
- Resolverlo solo al ingresar: deja abierto el caso de la cuenta equivocada.

## Consecuencias
- Dos registros opuestos en el mismo instante podrían colarse, porque la base no garantiza unicidad entre columnas
  distintas; para ese caso queda el respaldo del ingreso.
- Limitaciones conocidas: un usuario inexistente responde más rápido que una contraseña incorrecta (no se ejecuta el
  hash), por lo que el tiempo podría revelar si existe, y no hay límite de intentos de ingreso. Quedan como mejoras futuras.