# Comandos de prueba — M03

Sustituye `<WEBHOOK_URL>` por la **Test URL** o la **Production URL** de tu nodo Webhook.

## Linux / macOS / Git Bash

Ejemplo con EVT-01:

```bash
curl -i -X POST "<WEBHOOK_URL>" \
  -H "Content-Type: application/json" \
  -d '{
    "alert_id": "ALT-3001",
    "timestamp": "2026-11-04T09:00:00Z",
    "asset_id": "srv-web-01",
    "severity": "high",
    "source": "monitoring",
    "category": "availability",
    "summary": "HTTP 5xx rate above threshold",
    "details": "5xx responses exceeded 12% during the last 5 minutes"
  }'
```

## PowerShell

```powershell
$body = @{
  alert_id  = "ALT-3001"
  timestamp = "2026-11-04T09:00:00Z"
  asset_id  = "srv-web-01"
  severity  = "high"
  source    = "monitoring"
  category  = "availability"
  summary   = "HTTP 5xx rate above threshold"
  details   = "5xx responses exceeded 12% during the last 5 minutes"
} | ConvertTo-Json

Invoke-RestMethod `
  -Method POST `
  -Uri "<WEBHOOK_URL>" `
  -ContentType "application/json" `
  -Body $body
```

## Postman

1. Método: `POST`.
2. URL: tu Webhook URL.
3. `Body` → `raw` → `JSON`.
4. Copia únicamente el objeto `payload` del caso que quieras probar desde `alertas_webhook.json`.

## Probar otros casos

En `alertas_webhook.json`, cada entrada contiene:

```json
{
  "case_id": "EVT-XX",
  "description": "...",
  "payload": { }
}
```

Envía únicamente el contenido de `payload`, no el objeto exterior con `case_id` y `description`.