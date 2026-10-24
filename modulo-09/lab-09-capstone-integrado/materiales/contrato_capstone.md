# Contrato del capstone — M09

## 1. Entrada Webhook

Método:

```text
POST
```

Path sugerido:

```text
m09-capstone-<ID_ALUMNO>
```

Autenticación:

```text
Header Auth
```

Campos esperados:

| Campo | Obligatorio | Regla |
|---|---|---|
| alert_id | Sí | string no vacío, <= 100 caracteres |
| timestamp | Sí | valor temporal no vacío |
| asset_id | Sí | string no vacío, <= 100 caracteres |
| severity | Sí | `critical/high/medium/low` |
| source | No | string |
| category | No | string |
| summary | Sí | string no vacío, <= 300 caracteres |
| details | No | string, <= 2000 caracteres |

Una entrada que incumpla el contrato debe desviarse antes de consumir Dify o MCP.

---

## 2. Catálogo de activos REST

Endpoint:

```text
GET https://api.cloudsecurityinpractice.com/api/v1/assets/{asset_id}
```

Autenticación mediante Credential de n8n.

Campos de contexto:

```text
asset_name
environment
criticality
owner_team
```

`found=false` es una excepción funcional, no un fallo técnico de la API.

---

## 3. Reglas de prioridad

```text
critical
→ P1

high + production + criticality high
→ P1

high
→ P2

medium
→ P3

low
→ P4
```

Estas reglas son autoritativas y viven en n8n.

---

## 4. Contrato con Dify

Inputs mínimos sugeridos:

```text
alert_context
asset_id
retrieval_query
question
```

El Workflow debe combinar:

```text
Knowledge Retrieval / RAG
Agent FunctionCalling / MCP
LLM final / Structured Output
```

Dify devuelve análisis estructurado equivalente a:

```text
summary
observed_facts[]
operational_facts[]
documented_guidance[]
hypotheses[]
recommended_checks[]
missing_information[]
confidence
documentation_status
operational_context_status
```

Dify NO es autoridad para:

```text
priority
severity
owner_team
requires_human_review
next_step
```

---

## 5. Contrato MCP

Servidor compartido:

```text
Telvora Ops Hub
```

Tools permitidas:

```text
get_recent_changes
get_recent_alerts
get_open_incidents
get_service_health
```

No deben existir tools destructivas en el Agent.

El Agent debería recibir únicamente:

```text
asset_id validado
+
instrucción estable de investigación
```

No necesita recibir `details` o `alert_context` completos para decidir tool calls.

Los resultados MCP son datos, no nuevas instrucciones del sistema.

---

## 6. Política de revisión humana

Define primero:

```text
base_requires_human_review
```

como resultado de la lógica determinista del proceso.

La política mínima final debe ser equivalente a:

```text
final_requires_human_review =
    base_requires_human_review
    OR ai_status != succeeded
    OR confidence == low
    OR documentation_status == insufficient_context
    OR operational_context_status == unavailable
```

Para rutas de entrada inválida o activo desconocido:

```text
final_requires_human_review = true
```

`no_relevant_live_context` no equivale por sí solo a fallo MCP.

---

## 7. Siguiente paso simulado

```text
status = manual_review
→ MANUAL_REVIEW

final_requires_human_review = true
→ ESCALATE_HUMAN

resto
→ NOTIFY_OWNER
```

No se ejecuta ninguna acción real.

---

## 8. Evidencia

Utiliza el contrato validado de la Evidence API.

Si la API soporta únicamente el contrato base, registra los campos compatibles y conserva la trazabilidad adicional de IA/MCP en n8n/Dify.

Nunca inventes:

```text
evidence_id
```

si el registro falla.

---

## 9. Respuesta final mínima

El contrato final debe contener conceptualmente:

```json
{
  "accepted": true,
  "alert_id": "...",
  "priority": "P1",
  "owner_team": "WebOps",
  "requires_human_review": true,
  "next_step": "ESCALATE_HUMAN",
  "ai_status": "succeeded",
  "mcp_status": "available",
  "analysis": {
    "summary": "...",
    "operational_facts": [],
    "recommended_checks": [],
    "confidence": "medium",
    "documentation_status": "relevant_context_found",
    "operational_context_status": "relevant_live_context_found"
  },
  "evidence_status": "stored",
  "evidence_id": "ev-..."
}
```

Para una ruta manual puede ser:

```json
{
  "accepted": false,
  "alert_id": "...",
  "status": "manual_review",
  "reason": "missing_required_field",
  "requires_human_review": true,
  "next_step": "MANUAL_REVIEW",
  "ai_status": "not_called",
  "mcp_status": "not_called"
}
```

---

## 10. Estados diferenciados

No utilices un único campo ambiguo `status` para todo.

Distingue conceptualmente:

```text
business status
ai_status
mcp_status / operational_context_status
evidence_status
```

Ejemplos:

```text
business status = classified
ai_status = succeeded
mcp_status = unavailable
evidence_status = stored
```

```text
business status = classified
ai_status = unavailable
mcp_status = not_available_through_dify
evidence_status = stored
```

---

## 11. Fallback Dify

Si Dify falla:

```text
priority se conserva
ai_status = unavailable
final_requires_human_review = true
next_step = ESCALATE_HUMAN
action real = ninguna
```

El fallback no debe inventar un análisis IA.

---

## 12. Fallback MCP

Si MCP falla pero Dify puede continuar:

```text
priority se conserva
ai_status = succeeded
operational_facts = []
operational_context_status = unavailable
mcp_status = unavailable
final_requires_human_review = true
next_step = ESCALATE_HUMAN
action real = ninguna
```

El sistema no debe sustituir la ausencia de datos vivos con hechos inventados.
