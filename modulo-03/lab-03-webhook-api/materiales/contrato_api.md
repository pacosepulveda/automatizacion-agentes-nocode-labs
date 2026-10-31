# API de laboratorio — consulta de activos

Base URL:

```text
https://api.cloudsecurityinpractice.com
```

Todos los datos son ficticios.

## Health check

```http
GET /health
```

No requiere autenticación.

## Consultar un activo

```http
GET /api/v1/assets/{asset_id}
Authorization: Bearer <LAB_API_TOKEN>
```

El token será proporcionado por el instructor y debe almacenarse en una credencial de n8n.

No lo escribas directamente en el workflow ni lo incluyas en el repositorio.

### Activo encontrado

Ejemplo:

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

Este segundo caso devuelve una respuesta válida de la API: el sistema ha respondido correctamente, pero el activo no existe en el catálogo.

Eso es diferente de un error HTTP provocado, por ejemplo, por una URL incorrecta o una credencial no válida.