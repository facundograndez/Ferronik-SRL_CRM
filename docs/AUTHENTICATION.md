# Authentication

## Propiedad y confianza

La autenticación y autorización pertenecen exclusivamente a `Ferronik-SRL_Backend`. El frontend muestra formularios, envía credenciales, consulta sesión y renderiza capacidades, pero no valida passwords, no emite sesiones y no decide acceso a datos.

## Topología recomendada

- Frontend: `app.erp.ferronik.com.ar` en Vercel.
- Backend: `api.erp.ferronik.com.ar` en Railway.
- Ambos comparten el mismo sitio registrable (`ferronik.com.ar`) pero mantienen origins diferentes.

El backend usa una cookie de sesión opaca, `HttpOnly`, `Secure`, `SameSite=Lax`, con alcance host-only para `api.erp.ferronik.com.ar`. El navegador realiza requests a la API con `credentials: include`. JavaScript nunca lee el token; obtiene identidad/capacidades desde `GET /v1/session`.

Esta estrategia debe validarse con los dominios definitivos en FASE 1A/1B. Si frontend y API no pueden compartir sitio registrable, se reevaluará `SameSite=None; Secure`, CSRF reforzado o un proxy/BFF de transporte sin lógica de negocio.

## CORS y CSRF

- CORS allowlist exacta por ambiente; nunca `*` con credenciales.
- Sólo origins frontend conocidos; métodos/headers mínimos y preflight cacheado prudentemente.
- Toda mutación valida `Origin`/`Referer` y token CSRF ligado a sesión cuando corresponda.
- `SameSite` es defensa adicional, no sustituto de verificación CSRF.
- Webhooks externos usan rutas y autenticación/firma separadas, nunca cookies de usuario.

## Credenciales

- Argon2id con parámetros calibrados en backend.
- Email/username normalizados y comparación segura.
- Hashes y tokens nunca salen del backend.
- Contraseña temporal implica `must_change_password=true`.

## Sesiones

- Token aleatorio opaco; DB almacena sólo su hash, usuario, expiraciones, metadata mínima y revocación.
- Expiración absoluta y ociosa; “recordarme” selecciona una política más larga pero finita.
- Rotación al autenticar, cambiar password o privilegios.
- Logout, bloqueo y cambio de contraseña revocan sesiones.
- Permisos no viven como verdad en JWT cliente; backend los resuelve y puede invalidarlos de inmediato.

## Flujos

- Login: frontend envía credenciales por HTTPS; backend rate-limita, valida, crea cookie y audita.
- Recuperación: respuesta no enumerable, token de alta entropía hasheado, expiración corta y uso único.
- Primer acceso: backend impide otras operaciones privadas hasta cambiar password.
- Sesión: frontend hidrata usuario/capabilities desde API y muestra estado de carga/error/expiración.
- Protección de ruta frontend mejora UX, pero cada endpoint backend repite autenticación y permiso.

## Seguridad operacional

- Rate limit distribuido por IP + identidad para login/recuperación.
- CSP/headers en frontend y backend; HSTS en dominios productivos.
- Logs sin password, cookie o token; eventos de autenticación auditados.
- Credenciales ARCA/MeLi/Meta nunca se comparten con Vercel/frontend.

## Decisión pendiente antes de implementación

Confirmar dominio productivo y dominios de preview. Esta decisión fija atributos finales de cookie, CORS y CSRF; no se hardcodearán hasta entonces.
