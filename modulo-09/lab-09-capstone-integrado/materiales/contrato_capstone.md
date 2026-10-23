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

Una entrada que incumpla el contrato debe desviarse antes de consumir Dify.

---

## 2. Catálogo de activos

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
retrieval_query
question
```

Dify devuelve análisis estructurado equivalente a:

```text
summary
observed_facts[]
documented_guidance[]
hypotheses[]
recommended_checks[]
missing_information[]
confidence
documentation_status
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

## 5. Política de revisión humana

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
```

Para rutas de entrada inválida o activo desconocido:

```text
final_requires_human_review = true
```

---

## 6. Siguiente paso simulado

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

## 7. Evidencia

Utiliza el contrato validado de la Evidence API.

Si la API soporta únicamente el contrato base de M03, registra los campos compatibles y conserva la trazabilidad adicional de IA en n8n/Dify.

Nunca inventes:

```text
evidence_id
```

si el registro falla.

---

## 8. Respuesta final mínima

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
  "analysis": {
    "summary": "...",
    "recommended_checks": [],
    "confidence": "medium",
    "documentation_status": "relevant_context_found"
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
  "ai_status": "not_called"
}
```

---

## 9. Estados diferenciados

No utilices un único campo ambiguo `status` para todo.

Distingue conceptualmente:

```text
business status
ai_status
evidence_status
```

Ejemplo:

```text
business status = classified
ai_status = unavailable
evidence_status = stored
```

---

## 10. Fallback Dify

Si Dify falla:

```text
priority se conserva
ai_status = unavailable
final_requires_human_review = true
next_step = ESCALATE_HUMAN
action real = ninguna
```

El fallback no debe inventar un análisis IA.
