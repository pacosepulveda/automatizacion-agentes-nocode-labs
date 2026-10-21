# Contrato de la API de laboratorio — M03

Base URL:

```text
https://api.cloudsecurityinpractice.com
```

La API contiene exclusivamente datos ficticios.

## Health check

```http
GET /health
```

No requiere autenticación.

## Consulta de activo

```http
GET /api/v1/assets/{asset_id}
Authorization: Bearer <LAB_API_TOKEN>
```

El token será proporcionado por el instructor. **No debe incluirse en el repositorio ni escribirse directamente en un nodo.**

### Activo encontrado

```json
{
  "found": true,
  "asset_id": "srv-web-01",
  "asset_name": "web-frontend-01",
  "environment": "production",
  "criticality": "high",
  "owner_team": "WebOps"
}
```

### Activo no encontrado

```json
{
  "found": false,
  "asset_id": "srv-unknown-99",
  "error": "asset_not_found"
}
```

El endpoint devuelve HTTP 200 para este caso de negocio. Así distinguimos un **resultado funcional** de un **fallo de transporte o integración**.

## Registro de evidencia

```http
POST /api/v1/evidence
Authorization: Bearer <LAB_API_TOKEN>
Content-Type: application/json
```

Ejemplo de body:

```json
{
  "alert_id": "ALT-2001",
  "asset_id": "srv-web-01",
  "severity": "high",
  "priority": "P1",
  "status": "pending_human_review",
  "requires_human_review": true,
  "workflow_version": "m03-v1"
}
```

Respuesta:

```json
{
  "stored": true,
  "evidence_id": "ev-...",
  "received_at": "..."
}
```

## Consulta opcional

```http
GET /api/v1/evidence/{alert_id}
Authorization: Bearer <LAB_API_TOKEN>
```

La persistencia es únicamente de laboratorio y puede reiniciarse al redeplegar.
