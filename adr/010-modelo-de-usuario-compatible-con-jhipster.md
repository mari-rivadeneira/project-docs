# 010 - Modelo de usuario compatible con JHipster

## Contexto
El enunciado exige que el modelo y el contrato de usuarios sean compatibles con el usuario de JHipster: el registro
contempla login, password, firstName, lastName, email, imageUrl y langKey; el id, la activación, las autoridades y la
auditoría los administra el backend; el usuario queda activo al registrarse y la contraseña nunca se guarda en texto plano.

## Decisión
- Tablas users, authorities y user_authorities (migración V4) con los campos de JHipster y auditoría. Se cargan
  ROLE_USER y ROLE_ADMIN; el administrador es opcional y todavía no se usa.
- Validaciones de la cátedra en el registro: login de 3 a 50 caracteres (letras, números y _ . @ -), contraseña de 4 a 100,
  email de 5 a 254 con formato válido, firstName y lastName de hasta 50, imageUrl de hasta 256 y langKey de 2 a 10.
- El cliente nunca elige el id, la activación ni las autoridades: el usuario queda activo y con ROLE_USER.
- login y email se guardan en minúscula y son únicos con restricciones con nombre (uk_users_login y uk_users_email),
  que el adaptador traduce a errores de negocio.
- POST /api/register responde 201 sin cuerpo, como JHipster. GET /api/account devuelve la cuenta sin la contraseña ni su hash.
- El email, firstName y lastName son obligatorios (el enunciado marca solo imageUrl como opcional). langKey es opcional
  y toma "es" por defecto.

## Alternativas descartadas
- Campos propios adicionales: el enunciado pide el modelo de JHipster y nada más.
- Verificación por correo antes de activar: el enunciado la excluye.
- Los nombres de tabla de JHipster (jhi_user): la compatibilidad pedida es la de los campos y el comportamiento observable.
- firstName y lastName opcionales, como en JHipster: el enunciado marca solo imageUrl como opcional.

## Consecuencias
La base no puede garantizar con una restricción única que un valor no se repita entre dos columnas distintas
(login y email): ver ADR 013.