# Implementation Plan

## Forma de trabajo

Antes de cada fase: GOAL, ARCHITECTURE, DATABASE CHANGES, FILES TO CREATE, FILES TO MODIFY y RISKS. Después: IMPLEMENTED, TESTS, BUILD STATUS, FRONTEND STATUS, BACKEND STATUS, KNOWN LIMITATIONS y NEXT PHASE.

## FASE 0 — Arquitectura

### GOAL

Definir los dos repositorios, contrato API, límites de confianza, datos, seguridad, deployment, integraciones, testing y roadmap sin implementar aplicaciones.

### ARCHITECTURE

Frontend Next.js independiente en Vercel; backend TypeScript independiente inicialmente en Railway; PostgreSQL/Neon sólo accesible por backend; contrato OpenAPI con cliente generado.

### DATABASE CHANGES

Ninguna. Sólo modelo conceptual; no existe DB ni migration ejecutable.

### FILES CREATED/MODIFIED

Documentación de arquitectura, contrato, autenticación, datos, deployment, negocio, permisos, integraciones, testing y roadmap.

### RISKS

- Dos repositorios exigen coordinación de versiones, previews y despliegues compatibles.
- Cookies/CORS/CSRF dependen de dominios definitivos; deben cerrarse antes del login real.
- Región/latencia/costo de Railway y Neon deben medirse antes de producción.
- Jobs, webhooks e integraciones exigen cola, idempotencia y observabilidad desde el backend.
- El repositorio actual de FASE 0 no debe convertirse accidentalmente en un tercer runtime repository.

### EXIT

Documentación coherente, ADR monolítico marcado superseded y ninguna implementación de FASE 1.

## FASE 1A — Backend Foundation

### GOAL

Crear `Ferronik-SRL_Backend` como servicio real desplegado.

### SCOPE

- TypeScript Node.js y framework HTTP seleccionado.
- PostgreSQL/Neon, Prisma, migrations y seed idempotente.
- Login, logout, recuperación, sesiones, usuarios, roles y permisos.
- Auditoría base, rate limit, CORS/CSRF y secrets.
- OpenAPI 3.1, errores estables, health/readiness.
- Unit/integration/contract tests y CI.
- Deploy Railway de API; base para worker/cron sin implementar integraciones futuras.

### DONE GATE

API desplegada, DB real conectada, autenticación/autorización probadas, OpenAPI publicado y CI verde. Todavía no completa FASE 1 sin frontend.

## FASE 1B — Frontend Foundation

### GOAL

Crear `Ferronik-SRL_Frontend` y conectarlo a 1A.

### SCOPE

- Next.js, TypeScript strict, Tailwind y design system Ferronik.
- Login visual, manejo/expiración de sesión y route guards de UX.
- Cliente TypeScript generado desde OpenAPI.
- Layout autenticado, sidebar, header, light/dark.
- Administración de usuarios usando API real.
- Unit/E2E, CI y Preview/Production Vercel.

### DONE GATE

Frontend desplegado, login/session/logout y usuarios operando contra backend real; protección backend verificada; ambos builds/tests verdes. Sólo entonces FASE 1 es `DONE`.

## Fases posteriores

Cada fase coordina cambios backend compatibles, publicación OpenAPI, regeneración del cliente y UI. La lógica crítica permanece backend.

| Fase | Backend | Frontend / salida |
| --- | --- | --- |
| 2 Catálogo | productos, variantes, unidades, Pricing/Unit engines | productos/listas y previews validados por API |
| 3 Inventory | movimientos, reservas, acopios, concurrencia | operación de depósito |
| 4 Customers | clientes, subledgers ARS/USD e imputaciones | ficha y CC por moneda |
| 5 Quotes/CRM | pipeline, timeline y generación documental | presupuestos y seguimiento |
| 6 Sales & Logistics | venta, pricing, comisión, pagos, entrega y rentabilidad | flujo transaccional y snapshots |
| 7 Procurement | compra mixta, proveedores, NC y cost engine | breakdown/confirmación |
| 8 Production | BOM, consumos, outputs y costo | órdenes de producción |
| 9 Treasury | cajas, gastos, cheques, cross-transfer y cash flow | operación financiera autorizada |
| 10 ARCA | adapter, homologación, CAE y jobs | estados/PDF; SANDBOX antes de production readiness |
| 11 Mercado Libre | OAuth, webhooks, sync, conciliación y workers | operación/rentabilidad por orden |
| 12 Meta | adapters oficiales, webhooks y outbox | mensajería e historial |
| 13 Command Center | queries/KPIs/reportes autorizados | dashboards y reportes |
| 14 Migration | ETL selectivo/reconcile | preview de migración |
| 15 Hardening | performance, security, DR y observabilidad | production readiness review |

## Gates transversales

- OpenAPI compatible o versionado; cliente generado sincronizado.
- Backend nunca depende de DTOs/código frontend; frontend nunca accede a DB.
- Permisos negativos y ausencia de campos sensibles en responses.
- Migraciones revisadas y backward-compatible con deployments solapados.
- Idempotencia/auditoría en writes críticos.
- CI backend y frontend verdes; E2E contra servicios reales de preview.
- Estado de integración declarado `MOCK`, `SANDBOX` o `PRODUCTION READY`.

## Estado al cerrar esta revisión de FASE 0

| Item | Estado |
| --- | --- |
| Arquitectura separada y documentación | DONE |
| `Ferronik-SRL_Backend` | NOT CREATED |
| `Ferronik-SRL_Frontend` | NOT CREATED |
| DB/migrations | NOT STARTED |
| Tests/build | NOT APPLICABLE |
| Frontend Vercel | NOT DEPLOYED |
| Backend Railway | NOT DEPLOYED |
