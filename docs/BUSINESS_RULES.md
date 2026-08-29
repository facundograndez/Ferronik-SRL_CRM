# Business Rules

## PricingEngine

Entrada: tipo de venta, precio ingresado, alícuota/factor fiscal versionado, factor comercial, moneda, FX, descuentos y regla de redondeo. Salida: neto, impuestos, total, base de rentabilidad, costo neto, margen y snapshots.

- Factura A: `net = inputNet`; `tax = net * taxRate`; `total = net + tax`.
- Factura B: `total = inputGross`; `net = total / (1 + taxRate)`.
- Operación interna: `total = inputGross`; `commercialNet = total / commercialAdjustmentFactor`.
- Rentabilidad: `commercial/fiscal net - net cost`, según tipo.

`commercialAdjustmentFactor` vive en settings versionados, default `1.105`. Es regla interna, nunca normativa fiscal. El resultado persiste factor, fuente y versión.

## CommissionEngine

Calcula siempre sobre el neto snapshot producido por PricingEngine. La política de porcentaje puede variar por usuario, producto o canal; se resuelve antes de calcular y se guarda completa. Cambios posteriores no alteran comisiones históricas.

## UnitConversionEngine

Las conversiones forman un grafo hacia la unidad base. Para una operación se resuelve una ruta sin ciclos, se multiplican factores Decimal y se guarda cantidad comercial, unidad, ruta/factores y cantidad base. Una conversión modificada crea nueva versión.

## PurchaseCostEngine

Cada línea contiene segmentos con `weight`, costos, descuentos, impuestos recuperables/no recuperables y cargos. Resultado:

`effectiveCost = Σ(segmentEffectiveCost × segmentWeight) + allocableCharges - recoverableTaxes`

Los pesos suman 1. El motor soporta 100/0, 50/50, 30/70 o cualquier mezcla válida. Un override exige permiso, motivo y auditoría; se guardan cálculo original y final.

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

## Compras

Separa valor comercial de parte documentada y no documentada. Las cuentas del proveedor pueden tener subcuentas fiscal y comercial dentro del mismo proveedor. NC esperada no se trata como documento emitido; NC recibida se concilia y genera sus efectos explícitos.

## Cuentas corrientes

Saldo derivado de ledger. Pagos parciales, saldos a favor y no imputados son entries y allocations. Correcciones revierten y vuelven a registrar. Transferencia cruzada genera una transacción correlacionada en CC cliente y proveedor, sin caja.

## Producción

Al confirmar: validar BOM vigente y stock, bloquear proyección, consumir componentes, producir terminado, asignar costo y auditar en una transacción. BOM y costos se guardan como snapshot.

## Moneda

Monedas iniciales ARS/USD. Cada conversión guarda tasa, fecha, fuente y override. La fuente de dólar divisa es adapter configurable. Ningún cambio de tasa recalcula historia.

## Documentos fiscales e internos

Remitos nunca muestran precios. Documentos internos se identifican como no fiscales. Sólo ARCAProvider emite/autoriza comprobantes oficiales; los sistemas internos no simulan CAE ni NC de proveedor.
