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
| PurchaseCost | 100% fiscal, 100% comercial, 50/50 sin cargo, 50/50 con cargo comercial, 30/70, NC esperada 5%, factor 1.105 y override administrativo |
| Units | cadenas, precisión, ciclo inválido y factor snapshot |
| Commissions | base neta A/B/interna y snapshot |
| Inventory | compra/venta, reserva, override, concurrencia y ajuste |
| Acopios | alta, retiros parciales, exceso rechazado y remito sin precios |
| Ledgers | venta, pago parcial, saldo a favor, reverso, transferencia cruzada, ARS/USD separados y consolidado informativo con FX snapshot |
| Cash flow | realizado/proyectado a 7/30/60/90 días |
| Sales revisions | edición con motivo, movimientos compensatorios y auditoría |
| Logistics | sin entrega, pickup, envío; charge mayor/menor/cero vs cost; dirección snapshot y transiciones válidas |
| Shipping cost | PENDING/ESTIMATED/FINAL, revisión antes/después, autorización, auditoría y recálculo |
| Profitability | producto + logística + otros conceptos; PROVISIONAL/FINAL y causas pendientes |
| Shipping authz | vendedor ve charge pero API omite cost/profit; edición sin permiso devuelve 403 |
| Mercado Libre | cargo vs costo por orden, Envíos/Flex, snapshot y rentabilidad provisional hasta liquidación |

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
- El tratamiento fiscal del cargo de envío usa la regla configurada (incluido un caso distinto de 21%) y preserva snapshot.
- Reportes logísticos agregan por moneda/carrier/zona sin mezclar saldos o importes incompatibles.
- PurchaseCost conserva `90.486425...` antes del redondeo para el ejemplo 50/50 y aplica la regla de redondeo versionada sólo en la frontera definida.

## Build status de FASE 0

No existe aplicación ni dependencias instaladas: lint/typecheck/tests/build son `NOT APPLICABLE` en esta fase documental, no “passing”. FASE 1 crea scripts y CI, y no se considera DONE sin los cuatro verdes.
