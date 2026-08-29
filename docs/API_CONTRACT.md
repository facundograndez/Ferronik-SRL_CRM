# API Contract

## Autoridad

`Ferronik-SRL_Backend` es dueño del contrato OpenAPI 3.1. La especificación se genera desde schemas/rutas ejecutables, no desde un documento manual que pueda divergir. `Ferronik-SRL_Frontend` fija una versión/commit del artefacto y genera su cliente TypeScript.

## Flujo

1. Backend define request, response, error y autorización de cada operación.
2. CI genera y valida `openapi/openapi.json`.
3. CI compara con la última versión compatible y bloquea breaking changes accidentales.
4. El artefacto se publica como release/artifact descargable y, fuera de producción o con protección, como endpoint.
5. Frontend actualiza el artefacto fijado y regenera `src/api/generated`.
6. Typecheck, tests contractuales y E2E verifican la integración real.

No habrá copy/paste de DTOs ni un tercer repositorio inicialmente.

## Convenciones HTTP

- HTTPS y JSON; recursos plurales y endpoints de comandos cuando una transición de dominio lo requiera.
- Versionado mayor en path (`/v1`) o política equivalente fijada antes de publicar el primer contrato.
- Fechas ISO 8601; dinero como strings decimales + currency, nunca JSON number flotante.
- Paginación cursor preferida en colecciones grandes; filtros/orden server-side.
- Errores: `{ code, message, fieldErrors?, correlationId }`; sin stack traces ni datos sensibles.
- Header `Idempotency-Key` para ventas, pagos, webhooks/importaciones y operaciones críticas.
- ETag/version para concurrencia optimista donde corresponda.

## Autorización y datos sensibles

El backend aplica permisos antes de consultar/proyectar datos. Responses autorizadas pueden omitir campos sensibles; no se envía costo/margen para que el frontend lo oculte. OpenAPI documenta capacidades requeridas, pero la verificación ocurre en runtime.

## Compatibilidad

- Aditivo: nuevos endpoints o campos opcionales.
- Potencialmente incompatible: eliminar/renombrar, hacer obligatorio, cambiar tipo/semántica o restringir enum.
- Backend conserva durante una ventana la versión anterior cuando un frontend desplegado aún la consume.
- Deploy frontend y backend pueden realizarse en orden independiente sólo si el contrato es compatible.

## Testing

- Backend: schema validation, authz negativo, snapshots de OpenAPI y diff de breaking changes.
- Frontend: generación reproducible, typecheck contra contrato fijado y mocks generados sólo para tests.
- E2E: frontend desplegado contra backend/DB de preview reales; un mock nunca satisface el gate de FASE 1.
