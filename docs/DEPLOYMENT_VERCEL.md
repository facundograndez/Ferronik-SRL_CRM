# Deployment — Frontend on Vercel, Backend on Railway

## Decisión

- `Ferronik-SRL_Frontend`: Vercel, por integración nativa con Next.js, previews por branch y rollback de deployments.
- `Ferronik-SRL_Backend`: Railway inicialmente, como servicio Node.js persistente más worker(s), cron y cola privada.
- PostgreSQL: Neon administrado si su evaluación de región, backups, pooling y branching continúa satisfaciendo el proyecto.

El nombre histórico de este documento se conserva por el requisito Vercel del frontend; ya no afirma que todo el ERP se despliega allí.

## Comparación backend verificada en FASE 0

| Plataforma | Fortalezas | Riesgos/limitaciones para Ferronik | Decisión |
| --- | --- | --- | --- |
| Vercel | Functions, Cron, Queues/Workflow administrados, observabilidad integrada | Modelo orientado a invocaciones, límites de duración; Cron comparte esos límites y no reintenta fallos por sí solo | No seleccionada inicialmente para backend; sí frontend |
| Railway | Servicio persistente, workers siempre activos, cron, Redis/RabbitMQ y red privada por ambiente | Cron puede omitir una ejecución solapada; observabilidad avanzada puede requerir proveedor externo | **Recomendada** por equilibrio entre capacidad y operación |
| Render | Web services, background workers, cron, private network, health checks y zero-downtime deploy | Algunos features/pre-deploy dependen del plan; separar servicios incrementa costo | Segunda opción administrada |
| Fly.io | Machines, process groups, red privada, escalado/regiones y gran control | Mayor carga de Docker, red, escalado y operación; Postgres no administrado requiere responsabilidad propia | Alternativa si región/control lo justifican |

Railway evita diseñar ARCA, conciliaciones y outbox alrededor de ejecuciones HTTP limitadas, y permite evolucionar `api + worker + cron + queue` dentro de un mismo proyecto y red privada. La selección se revalida al comenzar FASE 1A con precios, región cercana a Neon/Argentina, SLA y requisitos ARCA vigentes.

## Ambientes

### Development

Frontend local contra backend local/dev. DB y storage sintéticos. `.env` separados por repositorio; frontend sólo contiene URL pública de API y flags no secretos.

### Preview

- Vercel crea preview del frontend por PR.
- Backend requiere ambiente/deployment preview compatible y URL conocida por el frontend.
- Neon branch o DB preview aislada; datos sintéticos.
- ARCA `disabled|homologation`; MeLi/Meta `disabled|sandbox`.
- CORS incluye únicamente origins preview autorizados. Como URLs Vercel son dinámicas, usar integración que registre origins o un dominio preview controlado; nunca regex abierta a dominios ajenos.

### Production

Frontend Vercel en dominio `app...`; backend Railway en `api...`; Neon productivo protegido. Secretos separados y acceso mínimo.

## Pipeline frontend

1. Install locked.
2. Obtener/fijar OpenAPI publicado por backend y regenerar cliente.
3. Verificar que generated client no tenga diff inesperado.
4. Lint, typecheck, unit, E2E/contract y `next build`.
5. Preview Vercel; smoke contra backend preview real.
6. Merge/promoción y rollback independiente.

## Pipeline backend

1. Install locked; lint, typecheck, unit/integration/contract.
2. Generar OpenAPI y ejecutar breaking-change diff.
3. Construir una imagen reproducible para API/worker/jobs.
4. Probar migrations sobre DB efímera/preview.
5. Producción: backup/checkpoint y `prisma migrate deploy` mediante pre-deploy/job único protegido.
6. Desplegar API y worker, ejecutar health/readiness y smoke.

## Orden y compatibilidad

Frontend y backend se despliegan independientemente. Cambios API siguen expand/migrate/contract: desplegar primero soporte backend compatible, luego frontend consumidor y sólo después retirar el contrato anterior. Rollback de código nunca intenta revertir automáticamente DB.

## Health, jobs y observabilidad

- Backend expone `/health` (proceso) y `/ready` (dependencias críticas) sin secretos.
- API no ejecuta trabajos largos; encola y responde con identificador.
- Worker procesa outbox con retries, idempotencia y dead-letter.
- Cron dispara jobs reentrantes con lock; el código contempla ejecuciones omitidas/duplicadas.
- Logs JSON, correlation ID extremo a extremo, error reporting y métricas de API/worker/cola.

## Secretos

- Vercel: sólo configuración frontend; nunca DB, Prisma, ARCA, MeLi, Meta ni storage credentials.
- Railway: DB, session secret, certificados y tokens backend como variables/secretos por ambiente.
- Neon y proveedores usan mínimo privilegio, rotación y credenciales distintas por ambiente.
- `NEXT_PUBLIC_*` jamás contiene secretos.

## Rollback

- Frontend: Instant Rollback/promote en Vercel.
- Backend: rollback/redeploy del artefacto anterior sólo si es compatible con el esquema actual.
- DB: forward fix con migration nueva; backups/PITR para incidentes de datos, no como rutina de deploy.

## Estado actual

Frontend Vercel: `NOT DEPLOYED`. Backend Railway: `NOT DEPLOYED`. DB: `NOT CREATED`. Ninguna plataforma queda `PRODUCTION READY` durante FASE 0.

## Referencias oficiales

- [Vercel Function duration](https://vercel.com/docs/functions/configuring-functions/duration)
- [Vercel Cron](https://vercel.com/docs/cron-jobs/manage-cron-jobs)
- [Vercel Queues](https://vercel.com/docs/queues)
- [Railway workers, queues and cron](https://docs.railway.com/guides/cron-workers-queues)
- [Railway private networking](https://docs.railway.com/networking/private-networking)
- [Render background workers](https://render.com/docs/background-workers)
- [Render deploys](https://render.com/docs/deploys)
- [Fly.io process groups](https://fly.io/docs/launch/processes/)
