# Implementation Plan

## Forma de trabajo obligatoria

Antes de cada fase se publicará: GOAL, ARCHITECTURE, DATABASE CHANGES, FILES TO CREATE, FILES TO MODIFY y RISKS. Después: IMPLEMENTED, TESTS, BUILD STATUS, VERCEL STATUS, KNOWN LIMITATIONS y NEXT PHASE.

## FASE 0 — Arquitectura

### GOAL

Definir stack, repositorio, límites de dominio, datos, seguridad, deployment, integraciones, testing y roadmap sin desarrollar módulos. La revisión final incorpora logística de ventas, rentabilidad total/provisional, regla Ferronik de compras mixtas y ledgers multimoneda.

### ARCHITECTURE

Monolito modular Next.js/Node en Vercel, PostgreSQL/Neon, Prisma, servicios de aplicación, dominio independiente e integraciones por adapters/outbox.

### DATABASE CHANGES

Ninguna. Se diseñó el modelo conceptual; no se creó DB ni migration.

### FILES CREATED

`README.md`, `.gitignore`, `.env.example` y diez documentos mínimos en `docs/`.

### RISKS

- Fiscalidad argentina y APIs externas cambian: validar documentación oficial en sus fases.
- Precisión/concurrencia de stock y ledgers requieren integración con PostgreSQL real.
- Vercel no es un worker persistente: eventos largos necesitan cola/proveedor compatible.
- Las reglas de comisión, vencimiento y redondeo aún requieren decisiones antes de implementarse.
- Fiscalidad del cargo de envío y catálogo de costos significativos deben quedar configurables y validarse antes de FASE 6.

### EXIT

Documentación coherente, trazable al requerimiento y sin afirmar funcionalidad inexistente.

## Fases de entrega

| Fase | Alcance | Dependencias / salida clave |
| --- | --- | --- |
| 1 Foundation | scaffold, CI, DB, auth, sesiones, users/RBAC, audit base, shell/theme, deploy | URL Vercel funcional y acceso seguro |
| 2 Catálogo | productos, variantes, unidades, listas, Pricing/Unit engines | snapshots y actualización masiva |
| 3 Inventory | depósitos, movimientos, reservas, acopios | stock concurrente consistente |
| 4 Customers | ficha, subledgers ARS/USD, pagos/imputaciones | saldos por moneda; consolidado sólo informativo |
| 5 Quotes/CRM | pipeline, timeline, PDF/envíos | conversión preparada |
| 6 Sales & Logistics | A/B/interna, pricing, comisión, pagos, entrega, cargo/costo logístico, rentabilidad provisional/final y remitos | operación transaccional y snapshots auditables |
| 7 Procurement | proveedores, compra mixta, NC, cost engine | subledgers fiscal/comercial |
| 8 Production | BOM, orden, consumos/output | costo snapshot |
| 9 Treasury | cajas, gastos, cheques, cross-transfer, cash flow | saldos derivados |
| 10 ARCA | adapter, homologación, CAE, PDF | primero SANDBOX, luego readiness |
| 11 Mercado Libre | OAuth, sync, fees, cargo/costo logístico por orden, rentabilidad provisional/final, conciliación | snapshots históricos |
| 12 Meta | WhatsApp/Instagram/Facebook oficiales | historial comunicación |
| 13 Command Center | KPIs, alertas y reportes | queries autorizadas/performantes |
| 14 Migration | ETL selectivo legacy | preview/reconcile; sin adaptar core |
| 15 Hardening | performance, seguridad, DR, observabilidad | production readiness review |

## FASE 1 — propuesta de primer corte

### GOAL

Aplicación mínima desplegada con login, logout, recuperación, sesión persistente, SUPERADMIN, usuarios/RBAC, auditoría base, shell privado y temas.

### DATABASE CHANGES

Identity, sessions, reset tokens, roles, permissions, settings y audit log; primera migration y seed idempotente de permisos/SUPERADMIN mediante secreto temporal.

### FILES TO CREATE

Scaffold Next.js, Prisma schema/migrations, módulos identity/audit, UI pública/privada, tests, workflow CI y health endpoint.

### RISKS

Proveedor de email y creación segura del primer SUPERADMIN; se resuelven antes de cerrar la fase, sin hardcodear credenciales.

## Gates transversales

- Migraciones compatibles y revisadas.
- Permisos negativos y no filtración de datos.
- Idempotencia y auditoría en writes críticos.
- Unit + integration + E2E proporcional al riesgo.
- Lint/typecheck/tests/build verdes.
- Preview aislada; adapters productivos bloqueados.
- Estado de integración declarado con honestidad.

## Estado al cerrar FASE 0

| Item | Estado |
| --- | --- |
| Arquitectura y documentación | DONE |
| Código funcional | NOT STARTED |
| DB/migrations | NOT STARTED |
| Tests/build | NOT APPLICABLE |
| Vercel | NOT DEPLOYED |
