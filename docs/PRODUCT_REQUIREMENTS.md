# Product Requirements

## Propósito

Construir el sistema central de operación de Ferronik SRL para ventas presenciales y e-commerce, compras, inventario, producción, cuentas corrientes, finanzas, fiscalidad, CRM e integraciones. Es un producto greenfield; el ERP anterior no condiciona este modelo.

## Usuarios y objetivos

- Dueño: visión consolidada, rentabilidad, liquidez y alertas.
- Administración: operar ventas, compras, pagos, documentación y conciliaciones.
- Vendedor: cotizar y vender sin exposición de costos o márgenes no autorizados.
- Depósito: stock físico, reservas, preparación, producción y retiros.
- Contador: circuito fiscal, impuestos, ARCA y exportaciones.
- Superadmin: usuarios, roles, permisos, configuración y auditoría.

## Capacidades por dominio

1. Identity & Access: login, sesiones, recuperación, usuarios, roles y permisos.
2. Catálogo & Pricing: productos, variantes, unidades, conversiones, listas e historial.
3. Inventory: depósitos, movimientos, reservas, acopios y producción.
4. CRM & Sales: clientes, presupuestos, ventas, remitos, recibos y comisiones.
5. Procurement: proveedores, compras fiscales/mixtas/internas, NC y costos.
6. Ledgers & Treasury: cuentas corrientes, cajas, bancos, cheques, gastos y cash flow.
7. Fiscal & Channels: ARCA, Mercado Libre y mensajería Meta.
8. Governance: documentos, notificaciones, reportes, auditoría e idempotencia.

## Requisitos transversales

- Toda ruta privada exige autenticación.
- Autorización server-side en cada lectura/escritura sensible.
- Operaciones multi-entidad atómicas mediante transacciones PostgreSQL.
- Valores monetarios y cantidades con precisión decimal y moneda explícita.
- Snapshots inmutables de factores, costos, tasas, conversiones y comisiones.
- Ledger append-only para saldos; correcciones por contramovimiento, no edición destructiva.
- Auditoría antes/después y motivo obligatorio para cambios críticos.
- Paginación, filtros y ordenamiento server-side.
- Accesibilidad, responsive desktop-first, light/dark y design system común.

## No objetivos de FASE 0

- No implementar UI, autenticación, endpoints ni esquema Prisma ejecutable.
- No conectar servicios externos ni crear mocks que aparenten integraciones terminadas.
- No migrar ni analizar el ERP legacy.

## Definición de terminado

Cada fase requiere UI, backend, persistencia, autorización, validación, errores, auditoría, tests, responsive, temas, documentación, CI verde y compatibilidad Vercel. Una integración debe etiquetarse `MOCK`, `SANDBOX` o `PRODUCTION READY`.

## Decisiones de negocio pendientes no bloqueantes para FASE 0

- Política exacta de vencimiento de acopios y tratamiento contractual.
- Reglas de redondeo por lista/unidad y límites de descuentos por rol.
- Base y porcentaje de comisión por vendedor/canal.
- Imputación contable definitiva de NC y cargos comerciales.
- Proveedor de email, storage y observabilidad.

Se modelan como configuración versionada; se definen antes de la fase que las consume.
