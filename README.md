# Ferronik ERP — Arquitectura de FASE 0

Este repositorio contiene la documentación arquitectónica greenfield de Ferronik SRL. **No es un tercer componente desplegable** y no contendrá código funcional del ERP.

La arquitectura oficial se implementará en dos repositorios:

| Repositorio | Responsabilidad | Deployment inicial |
| --- | --- | --- |
| `Ferronik-SRL_Frontend` | Next.js, experiencia de usuario y cliente OpenAPI | Vercel |
| `Ferronik-SRL_Backend` | API, autenticación, dominio, PostgreSQL e integraciones | Railway (recomendado) |

```text
Ferronik-SRL_Frontend (Vercel)
        |
        | HTTPS / OpenAPI
        v
Ferronik-SRL_Backend (Railway)
        |
        +--> PostgreSQL / Neon
        +--> cola y workers
        +--> ARCA, Mercado Libre, Meta, FX y object storage
```

El frontend nunca accede directamente a PostgreSQL ni es fuente de verdad para cálculos financieros, stock, impuestos o permisos. El backend recalcula, valida y autoriza toda operación.

## Documentación

La fuente de verdad de FASE 0 está en [`docs/`](docs/IMPLEMENTATION_PLAN.md), incluyendo [arquitectura](docs/ARCHITECTURE.md), [contrato API](docs/API_CONTRACT.md), [autenticación](docs/AUTHENTICATION.md) y [deployment](docs/DEPLOYMENT_VERCEL.md).

## Estado

| Fase | Estado |
| --- | --- |
| FASE 0 — Arquitectura | DONE (documentación revisada) |
| FASE 1A — Backend Foundation | NOT STARTED |
| FASE 1B — Frontend Foundation | NOT STARTED |
| FASE 2–15 | NOT STARTED |

FASE 1 sólo será `DONE` cuando ambos repositorios estén desplegados y conectados mediante la API real. Todavía no existen URLs de producción.
