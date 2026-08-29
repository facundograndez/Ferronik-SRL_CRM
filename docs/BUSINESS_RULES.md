# Business Rules

## PricingEngine

Entrada: tipo de venta, precio ingresado, alícuota/factor fiscal versionado, factor comercial, moneda, FX, descuentos y regla de redondeo. Salida: neto, impuestos, total, base de rentabilidad, costo neto, margen y snapshots.

- Factura A: `net = inputNet`; `tax = net * taxRate`; `total = net + tax`.
- Factura B: `total = inputGross`; `net = total / (1 + taxRate)`.
- Operación interna: `total = inputGross`; `commercialNet = total / commercialAdjustmentFactor`.
- Rentabilidad de producto: `commercial/fiscal product net - product net cost`, según tipo. La rentabilidad total se calcula por separado e incorpora logística y otros conceptos.

`commercialAdjustmentFactor` vive en settings versionados, default `1.105`. Es regla interna, nunca normativa fiscal. El resultado persiste factor, fuente y versión.

## CommissionEngine

Calcula siempre sobre el neto snapshot producido por PricingEngine. La política de porcentaje puede variar por usuario, producto o canal; se resuelve antes de calcular y se guarda completa. Cambios posteriores no alteran comisiones históricas.

## UnitConversionEngine

Las conversiones forman un grafo hacia la unidad base. Para una operación se resuelve una ruta sin ciclos, se multiplican factores Decimal y se guarda cantidad comercial, unidad, ruta/factores y cantidad base. Una conversión modificada crea nueva versión.

## PurchaseCostEngine

Cada línea contiene porcentaje fiscal/documentado, porcentaje comercial, descuento o NC esperada, impuestos recuperables/no recuperables, `commercialAdjustmentFactor` y cargo configurable del componente comercial. Los porcentajes fiscal + comercial suman 1.

Regla Ferronik actual, expresada como configuración comercial y no como normativa fiscal:

```text
baseNetCost = 100
expectedCreditNoteDiscount = 5%
fiscalAdjustedCost = 100 × (1 - 0.05) = 95
commercialAdjustedBase = 95 / 1.105
commercialAdjustedCost = commercialAdjustedBase + configurableCommercialCharge
effectiveCost =
  fiscalAdjustedCost × fiscalWeight
  + commercialAdjustedCost × commercialWeight
  + otherAllocableCharges
  - recoverableTaxes
```

El cargo comercial se configura como porcentaje o importe según la regla seleccionada. Valores operativos pueden ser 0%, 10,5% u otro valor manual, pero 10,5% jamás se hardcodea. Debe guardarse su base de aplicación, modalidad y snapshot.

Ejemplo 50/50 sin cargo: `(95 × 0.50) + ((95 / 1.105) × 0.50) = 90.486425...`; el importe presentado será `90.49` con redondeo estándar a dos decimales o el que determine la regla versionada. Ejemplo 30/70: `(95 × 0.30) + ((95 / 1.105) × 0.70)`. Nunca se divide por dos salvo que los pesos sean exactamente 50/50.

Resultado general:

`effectiveCost = Σ(segmentEffectiveCost × segmentWeight) + allocableCharges - recoverableTaxes`

El motor soporta 100/0, 0/100, 50/50, 30/70 o cualquier mezcla válida y devuelve breakdown paso a paso antes de confirmar. Un override exige permiso, motivo y auditoría; se guardan cálculo original y final, descuento/NC esperada, factor y cargo snapshots.

## Inventory

- El movimiento es append-only y expresa cantidad base con signo, tipo, origen y costo snapshot.
- Stock físico suma movimientos efectivizados.
- Reserva/acopio suma reservas activas pendientes.
- Disponible es físico menos reservado.
- Ajustes y venta de reservado requieren permisos específicos y motivo.
- Concurrencia: lock de filas/proyección de stock dentro de la transacción y revalidación al confirmar.

## Acopios

Una venta crea reserva y saldo acopiado. Cada retiro parcial reduce reserva, genera salida física, remito sin precios y trazabilidad. No puede superar saldo. Vencimiento alerta; no libera automáticamente hasta definir política contractual y autorización.

## Ventas

Estados propuestos: `DRAFT`, `CONFIRMED`, `FISCAL_PENDING`, `COMPLETED`, `CANCELLED`. Confirmar congela snapshots, movimientos, ledger y outbox. Editar una venta confirmada crea revisión con motivo; diferencias se contabilizan mediante movimientos compensatorios. ARCA fallida no elimina la venta.

## Entrega y costo logístico

Una venta elige: sin entrega, retiro del cliente o envío. Métodos iniciales extensibles: `CUSTOMER_PICKUP`, `OWN_FREIGHT`, `THIRD_PARTY_FREIGHT`, `EXPRESS`, `MERCADO_ENVIOS`, `MERCADO_ENVIOS_FLEX`, `OTHER`.

Se mantienen separados:

```text
shippingProfit = shippingChargeNet - shippingCost
productProfit = productRevenueNet - productCost
totalProfit = productProfit + shippingProfit + otherIncome - otherCosts
```

`shippingCharge` es lo cobrado; `shippingCost` es el costo real Ferronik. Ambos pueden ser cero o diferir. El tratamiento fiscal de `shippingCharge` se resuelve por tipo de comprobante y configuración vigente; no se presupone IVA 21%. Se guardan neto, impuesto, total, alícuota/regla, moneda y versiones.

Estado de entrega (`NOT_REQUIRED`, `PENDING`, `SCHEDULED`, `PREPARING`, `DISPATCHED`, `DELIVERED`, `CANCELLED`) es independiente de venta y ARCA. Estado del costo (`PENDING`, `ESTIMATED`, `FINAL`) permite confirmar la venta sin costo definitivo.

Cada entrega conserva carrier/provider, dirección snapshot y notas. Modificar al cliente no altera historia. Corregir costo o dirección exige permiso/motivo, crea revisión con antes/después, recalcula un nuevo snapshot de rentabilidad y genera AuditLog.

## Rentabilidad total y estado

`PROVISIONAL` significa que falta costo significativo: envío real, liquidación/costos Mercado Libre u otro concepto pendiente/estimado. `FINAL` requiere que todos estén confirmados. UI y APIs exponen el estado y causas pendientes de forma inequívoca; nunca presentan provisional como definitiva.

Los reportes podrán segmentar cargo, costo y resultado logístico; envío bonificado; período; carrier; zona; promedios cobrados/reales. Las agregaciones respetan moneda o muestran una conversión informativa explícita.

## Compras

Separa valor comercial de parte documentada y no documentada. Las cuentas del proveedor pueden tener subcuentas fiscal y comercial dentro del mismo proveedor. NC esperada no se trata como documento emitido; NC recibida se concilia y genera sus efectos explícitos.

## Cuentas corrientes

Saldo derivado de ledger y separado por moneda para clientes y proveedores. Pagos parciales, saldos a favor y no imputados son entries y allocations. Correcciones revierten y vuelven a registrar. Transferencia cruzada genera una transacción correlacionada en CC cliente y proveedor, sin caja. `ARS 10.000.000` y `USD 5.000` permanecen dos saldos; cualquier equivalente consolidado informa tasa snapshot y no los reemplaza.

## Producción

Al confirmar: validar BOM vigente y stock, bloquear proyección, consumir componentes, producir terminado, asignar costo y auditar en una transacción. BOM y costos se guardan como snapshot.

## Moneda

Monedas iniciales ARS/USD. Cada conversión guarda tasa, fecha, fuente y override. La fuente de dólar divisa es adapter configurable. Ningún cambio de tasa recalcula historia.

## Documentos fiscales e internos

Remitos nunca muestran precios. Documentos internos se identifican como no fiscales. Sólo ARCAProvider emite/autoriza comprobantes oficiales; los sistemas internos no simulan CAE ni NC de proveedor.
