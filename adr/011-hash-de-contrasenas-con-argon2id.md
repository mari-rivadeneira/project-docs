# 011 - Hash de contraseñas con Argon2id

## Contexto
El enunciado exige guardar las contraseñas con un mecanismo criptográfico adecuado, sin exponerlas en respuestas ni
logs, y la cátedra define contraseñas de 4 a 100 caracteres.

## Decisión
- Argon2id, que OWASP recomienda en primer lugar para guardar contraseñas, con el Argon2PasswordEncoder de Spring
  Security (usa BouncyCastle). Va detrás del puerto PasswordHasher: el dominio no conoce el algoritmo y cambiarlo
  es tocar una clase.
- Parámetros: los mínimos de OWASP (19 MiB de memoria, 2 iteraciones, paralelismo 1), con sal de 16 bytes y hash de 32.
  Se configuran con PASSWORD_ARGON2_MEMORY_KIB, PASSWORD_ARGON2_ITERATIONS y PASSWORD_ARGON2_PARALLELISM; si un valor
  queda por debajo del mínimo, el servicio no arranca.
- El hash guarda sus propios parámetros, así que cambiar el costo más adelante no invalida los hashes existentes.
- password_hash es VARCHAR(255): un hash Argon2id mide unos 97 caracteres.

## Alternativas descartadas
- BCrypt: es la generación anterior y solo procesa los primeros 72 bytes; con contraseñas de hasta 100 caracteres
  (y caracteres como la ñ, que pesan 2 bytes) habría requerido un paso previo.
- Los valores por defecto de Spring (16 MiB): quedan por debajo del mínimo recomendado por OWASP.
- Costos fijos en el código: cambiarlos exigiría recompilar.

## Consecuencias
- Cada registro o ingreso ocupa unos 19 MiB durante unas decenas de milisegundos: con muchos ingresos simultáneos la
  memoria temporal crece. Por eso se limita el largo de la entrada (contraseña de hasta 100, login y email de hasta 254).
- BouncyCastle se suma como dependencia.
- Un hash de otro formato (por ejemplo BCrypt) no se verifica: matches devuelve false sin lanzar excepción; un test lo comprueba.