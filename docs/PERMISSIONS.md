# Permissions

## Modelo

Autorización híbrida:

1. Roles otorgan conjuntos base.
2. Overrides por usuario permiten `ALLOW` o `DENY` explícito.
3. `DENY` prevalece, salvo SUPERADMIN de sistema.
4. El permiso se evalúa server-side en cada caso de uso y consulta.
5. La UI consume capacidades efectivas sólo para experiencia; nunca es control de seguridad.

Los permisos usan `recurso.acción`, se registran en catálogo y se referencian por clave estable.

## Roles iniciales

- `SUPERADMIN`: acceso total, gestión de identidad y configuración crítica.
- `OWNER`: visibilidad empresarial completa, sin capacidades técnicas reservadas si se decide separarlas.
- `ADMINISTRATION`: operación administrativa configurable.
- `SELLER`: clientes, presupuestos, ventas, disponibilidad y acopios; sin costos/margen por defecto.
- `WAREHOUSE`: inventario, retiros, movimientos y producción.
- `ACCOUNTANT`: fiscal, impuestos, ARCA, compras fiscales, reportes y exportaciones.

## Catálogo inicial

```text
dashboard.view
sales.view/create/edit/cancel/viewCost/viewMargin/viewProfitability/overrideReservedStock
quotes.view/create/edit/send/convert
customers.view/create/edit/viewLedger/editLedger
products.view/create/edit/viewCost/managePrices
inventory.view/adjust/reserve/release/withdraw
stockpiles.view/create/withdraw/override
purchases.view/create/edit/cancel/viewCosts/overrideCost
suppliers.view/create/edit/viewLedger/editLedger
production.view/create/confirm/cancel
finance.view/manageCash/manageExpenses/viewCashFlow
checks.view/create/update/deliver/cancel
fiscal.view/issue/retry/export
marketplace.view/sync/reconcile/viewProfitability
reports.sales/stock/finance/profitability/tax
documents.view/upload/delete
users.view/administer/disable/resetPassword
roles.view/administer
settings.view/administer
audit.view
```

## Enforcement

- `requirePermission(actor, permission, scope?)` en application layer.
- Repositorios reciben un query policy para columnas sensibles; no se recuperan y luego se ocultan.
- Descargas usan URLs firmadas sólo tras autorización.
- Cambios de rol/permisos incrementan `authz_version`; sesiones con versión anterior recalculan o se invalidan.
- Acciones masivas reevalúan permisos por operación y registran resultado.
- Tests de contrato verifican 401, 403 y ausencia de campos sensibles.

## Scopes futuros

La interfaz acepta contexto (`warehouseId`, `sellerId`, canal) para limitar acceso sin rediseñar el catálogo. FASE 1 implementa scope global; scopes por depósito/vendedor se activan cuando exista una regla de negocio definida.
