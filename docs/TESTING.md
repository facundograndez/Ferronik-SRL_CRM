# Testing Strategy

## Pirámide

- Unit: motores puros, value objects, permisos y máquinas de estado.
- Integration: repositorios y casos de uso contra PostgreSQL real, incluyendo locks/transactions.
- Contract: adapters externos con fixtures sanitizados y schemas.
- E2E: Playwright para flujos de alto valor y aislamiento por rol.
- Security: authz negativo, CSRF/origin, rate limit, exposición de campos y uploads.

## Matriz mínima solicitada

| Área | Casos |
| --- | --- |
| Pricing | Factura A, B, interno 1.105, descuentos, redondeo, USD |
| PurchaseCost | 100% fiscal, interna, 50/50, 30/70, descuentos, cargas y override |
| Units | cadenas, precisión, ciclo inválido y factor snapshot |
| Commissions | base neta A/B/interna y snapshot |
| Inventory | compra/venta, reserva, override, concurrencia y ajuste |
| Acopios | alta, retiros parciales, exceso rechazado y remito sin precios |
| Ledgers | venta, pago parcial, saldo a favor, reverso y transferencia cruzada |
| Cash flow | realizado/proyectado a 7/30/60/90 días |
| Sales revisions | edición con motivo, movimientos compensatorios y auditoría |

## Datos y aislamiento

Factories deterministas, reloj/UUID/provider inyectables. Cada test integration usa schema/base aislada y rollback o limpieza explícita. Nunca se usan datos o credenciales productivas.

## CI

En cada PR: `lint`, `typecheck`, unit, integration y build. E2E smoke corre sobre Preview; suite completa en merge/nightly según duración. Coverage es señal, no sustituto: umbral alto para engines y authz, más mutation testing opcional en cálculos críticos.

## Criterios de aceptación

- Tests verifican resultados y asientos/movimientos/auditoría producidos.
- Casos de permiso confirman que datos sensibles ni siquiera aparecen en respuesta.
- Tests de idempotencia repiten el mismo request/webhook y observan un solo efecto.
- Concurrencia intenta vender/retirar el mismo stock simultáneamente.
- Snapshots siguen iguales tras modificar configuración actual.

## Build status de FASE 0

No existe aplicación ni dependencias instaladas: lint/typecheck/tests/build son `NOT APPLICABLE` en esta fase documental, no “passing”. FASE 1 crea scripts y CI, y no se considera DONE sin los cuatro verdes.
