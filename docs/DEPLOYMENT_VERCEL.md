# Deployment on Vercel

## 1. Crear proyecto

Conectar el repositorio Git a Vercel y seleccionar Next.js. `main` será Production Branch; cada PR/branch genera Preview. No se requiere `vercel.json` mientras los defaults sean suficientes.

## 2. Ambientes y variables

Configurar Development, Preview y Production por separado. Los secretos de Preview/Production deben marcarse sensitive. Nunca asignar a Preview credenciales productivas de ARCA, Mercado Libre o Meta. `APP_ENV` se valida contra `VERCEL_ENV` al iniciar.

## 3. Base de datos

Neon es la opción inicial por PostgreSQL serverless, pooling y branching. Producción tiene proyecto/branch protegido. Preview usa branch/database aislada y datos sintéticos; si no hay automatización por PR, usa una DB preview compartida sin datos reales y con migraciones compatibles.

Runtime recibe `DATABASE_URL` pooled. CI/migrations recibe `DIRECT_DATABASE_URL` con alcance restringido. Las funciones corren en Node.js y reutilizan el cliente Prisma a nivel de módulo.

## 4. Migrations

- PR: generar SQL, revisar cambios destructivos y probar sobre DB efímera/preview.
- Producción: backup/checkpoint, ejecutar `prisma migrate deploy` una vez mediante job CI protegido, luego desplegar/promover app.
- Cambios incompatibles usan expand/migrate/contract en múltiples releases.
- Nunca ejecutar `migrate dev` ni `db push` en producción.

## 5. Build

Pipeline obligatorio: install locked, lint, typecheck, unit/integration tests y `next build`. El build no debe necesitar conectarse a producción ni ejecutar migrations.

## 6. Deployment

Push a branch crea Preview; merge a `main` crea candidato de Production. Promoción sólo después de health smoke test y migrations compatibles. Las regiones de funciones se acercan a la región primaria de DB.

## 7. Preview deployments

- URL única por PR.
- Base aislada o sintética, email sink, storage de preview.
- `ARCA_MODE=disabled|homologation`; `MELI_MODE=disabled|sandbox`; `META_MODE=disabled|sandbox`.
- Banner visible de no producción y bloqueo server-side de adapters production.

## 8. Producción

Dominio, HSTS, backups/PITR, alertas de error/latencia, presupuesto y logs estructurados. `/api/health` comprueba proceso/configuración sin filtrar secretos; `/api/ready` puede comprobar DB con rate limit y protección operativa.

## 9. Rollback

El código puede volver mediante Instant Rollback/promote. La DB no se “desmigra” automáticamente: migrations son forward-compatible y se corrigen con una migration nueva. Antes de rollback verificar compatibilidad entre versión anterior y esquema actual.

## 10. Secretos

- Guardar sólo en Vercel/env manager; rotar periódicamente y ante incidentes.
- Separar credenciales por ambiente y mínimo privilegio.
- Certificados ARCA en base64/secret store sólo si el adapter lo exige; jamás logs.
- No exponer variables server con prefijo público.
- Acceso a Production limitado y auditado.

## Checklist inicial de FASE 1

1. Crear Neon dev/preview/prod y confirmar región/pooling.
2. Importar variables desde `.env.example` con valores separados.
3. Conectar Git y activar previews.
4. Agregar CI y migration job protegido.
5. Desplegar, ejecutar smoke de login/privacidad/logout/health.
6. Registrar URL y estado real; hasta entonces Vercel Status es `NOT DEPLOYED`.

## Referencias verificadas en FASE 0

- [Next.js App Router](https://nextjs.org/docs/app)
- [Vercel environment variables](https://vercel.com/docs/environment-variables)
- [Vercel Git deployments](https://vercel.com/docs/git)
- [Vercel Instant Rollback](https://vercel.com/docs/instant-rollback)
- [Prisma serverless deployment](https://www.prisma.io/docs/orm/prisma-client/deployment/serverless)
