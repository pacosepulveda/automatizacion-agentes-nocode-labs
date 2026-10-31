# Aprobación manual de una ejecución pausada

Cuando la política de n8n determine que una recomendación requiere revisión humana, el workflow devolverá una respuesta similar a:

```json
{
  "status": "pending_approval",
  "alert_id": "ALT-7102",
  "priority": "P1",
  "action_type": "rollback_deployment",
  "recommended_next_step": "Proponer rollback de la versión desplegada",
  "reason": "...",
  "approval_url": "https://n8n-agentes.cloudsecurityinpractice.com/webhook-waiting/..."
}
```

`approval_url` es única para esa ejecución y debe tratarse como información sensible mientras la aprobación esté pendiente.

## Aprobar con curl

```bash
curl -X POST "<APPROVAL_URL>" \
  -H "Content-Type: application/json" \
  -d '{
    "decision": "approve",
    "comment": "Revisado por el operador durante el laboratorio"
  }'
```

## Rechazar con curl

```bash
curl -X POST "<APPROVAL_URL>" \
  -H "Content-Type: application/json" \
  -d '{
    "decision": "reject",
    "comment": "No existe evidencia suficiente para autorizar la acción"
  }'
```

## PowerShell

Aprobar:

```powershell
$body = @{
    decision = "approve"
    comment  = "Revisado por el operador durante el laboratorio"
} | ConvertTo-Json

Invoke-RestMethod \
    -Method Post \
    -Uri "<APPROVAL_URL>" \
    -ContentType "application/json" \
    -Body $body
```

Rechazar:

```powershell
$body = @{
    decision = "reject"
    comment  = "No existe evidencia suficiente para autorizar la acción"
} | ConvertTo-Json

Invoke-RestMethod \
    -Method Post \
    -Uri "<APPROVAL_URL>" \
    -ContentType "application/json" \
    -Body $body
```

## Importante

La aprobación **no ejecuta** un reinicio, rollback ni cambio real.

En este laboratorio significa:

```text
approve
→ la acción queda autorizada conceptualmente
→ action_status = authorized_not_executed
```

```text
reject
→ la acción queda rechazada
→ action_status = rejected
```

El objetivo es practicar el patrón Human-in-the-Loop sin proporcionar al agente ni al workflow herramientas destructivas.
