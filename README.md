# Ferronik ERP

ERP greenfield para Ferronik SRL. El repositorio se encuentra en **FASE 0 — Arquitectura**.

En esta fase no hay módulos de negocio implementados ni una aplicación desplegable. La fuente de verdad del diseño está en [`docs/`](docs/IMPLEMENTATION_PLAN.md).

## Decisiones principales

- Next.js App Router + React + TypeScript estricto.
- PostgreSQL administrado en Neon, con Prisma ORM y runtime Node.js en Vercel.
- Monolito modular con servicios de dominio y transacciones ACID.
- Auth propia con sesiones opacas persistidas; autorización RBAC + permisos efectivos.
- Dinero, cantidades, factores y tipos de cambio en `numeric`; nunca `float`.
- Integraciones detrás de puertos/adapters y procesamiento idempotente.

## Estado

| Fase | Estado |
| --- | --- |
| FASE 0 — Arquitectura | DONE (documentación) |
| FASE 1 — Foundation | NOT STARTED |
| FASE 2–15 | NOT STARTED |

No hay URL de Vercel todavía: crearla pertenece a FASE 1.
