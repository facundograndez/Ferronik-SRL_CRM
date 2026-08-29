# Authentication

## Enfoque

Autenticación propia para empleados, separada de la autorización. Se implementará con primitivas auditadas (Auth.js cuando su versión estable vigente cubra el flujo Credentials) y servicios propios para alta, bloqueo, primer acceso y recuperación. No se inventará criptografía.

## Credenciales

- Password hash Argon2id con parámetros calibrados en el runtime; fallback sólo si la plataforma lo exige y queda documentado.
- Reglas de longitud y contraseñas comprometidas; no reglas cosméticas rígidas.
- Hash y tokens jamás salen de server/infrastructure.
- Contraseña temporal implica `must_change_password=true`.

## Sesiones

- Token opaco aleatorio en cookie `HttpOnly`, `Secure` en producción, `SameSite=Lax`, path `/`.
- En DB sólo se guarda hash del token, usuario, expiración, última actividad, metadata mínima y revocación.
- Expiración absoluta y ociosa; “recordarme” selecciona política más larga, no elimina expiración.
- Rotación al autenticar/cambiar privilegios/contraseña; logout y bloqueo revocan sesiones.
- No se usa JWT auto-contenido para permisos: revocación y cambios deben ser inmediatos.

## Flujos

### Login

Normalizar identificador, rate-limit por IP + identidad, verificar usuario activo y hash en tiempo constante, crear sesión, auditar éxito/fallo relevante y redirigir sólo a rutas internas validadas.

### Recuperación

Respuesta indistinguible exista o no la cuenta. Token de alta entropía, hash en DB, uso único, expiración corta. Al restablecer: invalidar sesiones, marcar token usado, auditar y notificar.

### Cambio/primer acceso

Requiere sesión reciente y contraseña actual salvo token de primer acceso. Tras cambio se rota la sesión. Rutas privadas distintas del cambio quedan bloqueadas mientras `must_change_password` sea verdadero.

## Protección de rutas

Un proxy/middleware hace redirección temprana para UX, pero cada Server Component, Route Handler y Server Action vuelve a verificar sesión y permiso. Rutas públicas allowlisted: `/login`, solicitud/confirmación de recuperación y assets necesarios.

## CSRF y abuso

- Cookies SameSite, validación de Origin para mutaciones y tokens CSRF cuando el mecanismo de framework no sea suficiente.
- Rate limit distribuido para login/recuperación y lockout progresivo sin permitir denegación permanente por atacante.
- Mensajes genéricos; logs sin password/token.
- Headers de seguridad, CSP progresiva, HSTS en producción y protección clickjacking.

## Auditoría

Eventos: login exitoso, fallo relevante, logout, recuperación solicitada/consumida, password cambiado, sesión revocada, usuario bloqueado y privilegios modificados. IP/user-agent se minimizan y retienen según política a definir.
