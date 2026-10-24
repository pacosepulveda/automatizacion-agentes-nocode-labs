# Contrato de integración con Dify — M06

Base URL del entorno del curso:

```text
https://dify.cloudsecurityinpractice.com
```

El laboratorio utiliza una aplicación de tipo **Workflow** publicada en Dify.

Dentro del Workflow, Dify combina:

```text
Knowledge Retrieval / RAG
Agent + MCP / Telvora Ops Hub
LLM final / Structured Output
```

## Endpoint

```http
POST /v1/workflows/run
Authorization: Bearer <DIFY_APP_API_KEY>
Content-Type: application/json
```

El API key es específico de la aplicación y debe almacenarse en una credencial de n8n. No debe escribirse directamente en el nodo HTTP Request ni incluirse en el repositorio.

El token MCP no forma parte de esta llamada. La conexión con Telvora Ops Hub está configurada por el instructor dentro del Workspace de Dify.

## Request

El Workflow de Dify define cuatro inputs:

```text
alert_context
asset_id
retrieval_query
question
```

Ejemplo conceptual:

```json
{
  "inputs": {
    "alert_context": "{\"alert_id\":\"alumno01-ALT-6001\",\"asset_id\":\"srv-web-01\",\"severity\":\"high\",\"priority\":\"P1\",\"asset_name\":\"web-frontend-01\",\"environment\":\"production\",\"criticality\":\"high\",\"summary\":\"HTTP 5xx rate above threshold\",\"details\":\"5xx responses increased shortly after a deployment\"}",
    "asset_id": "srv-web-01",
    "retrieval_query": "srv-web-01 web-frontend-01 HTTP 5xx rate above threshold deployment runbook",
    "question": "Analiza la alerta combinando hechos validados, documentación operativa y estado operativo vivo."
  },
  "response_mode": "blocking",
  "user": "alumno01-m06"
}
```

`user` identifica al consumidor de la API y debe mantenerse estable para el laboratorio.

## Qué ocurre dentro de Dify

La llamada anterior no incluye manualmente los datos MCP. El Workflow realiza internamente:

```text
asset_id
   ↓
Agent FunctionCalling
   ↓
Telvora Ops Hub / MCP
   ↓
resultados de tools
```

El Agent puede invocar:

```text
get_recent_changes
get_recent_alerts
get_open_incidents
get_service_health
```

Los resultados vivos pueden variar con el tiempo.

## Response

En modo `blocking`, la respuesta contiene información de la ejecución y los outputs del Workflow. La estructura esperada es conceptualmente equivalente a:

```json
{
  "workflow_run_id": "...",
  "task_id": "...",
  "data": {
    "id": "...",
    "workflow_id": "...",
    "status": "succeeded",
    "outputs": {
      "analysis": {
        "summary": "...",
        "observed_facts": [],
        "operational_facts": [],
        "documented_guidance": [],
        "hypotheses": [],
        "recommended_checks": [],
        "missing_information": [],
        "confidence": "medium",
        "documentation_status": "relevant_context_found",
        "operational_context_status": "relevant_live_context_found"
      }
    },
    "error": null,
    "elapsed_time": 0.0,
    "total_tokens": 0,
    "total_steps": 0
  }
}
```

Durante el laboratorio inspecciona la respuesta real de tu versión de Dify y mapea los campos desde esa respuesta, no desde valores escritos manualmente.

## Criterio de éxito

n8n debe considerar correcta la llamada cuando, como mínimo:

```text
HTTP correcto
AND
data.status = succeeded
AND
data.outputs.analysis existe
```

El hecho de que Dify responda HTTP 200 no sustituye la validación del estado funcional del Workflow.

## Trazabilidad MCP

El output `analysis` no contiene necesariamente el detalle de todas las llamadas MCP. Para comprobar qué tools utilizó el Agent, inspecciona la ejecución en Dify:

```text
Agent
→ Detalle
→ Estrategia agéntica
→ Round
```

Ahí podrás observar campos equivalentes a:

```text
tool_call_name
tool_call_input
tool_response
```

## Errores relevantes

- `401`: API key de la App Dify inválida o ausente.
- `400`: inputs inválidos, aplicación no publicada u otro problema de la solicitud/workflow.
- `429`: límite de concurrencia o rate limit.
- `500`: error interno o de ejecución aguas arriba.
- timeout: n8n no obtiene respuesta dentro del tiempo configurado.

En M07 se ampliará el diseño para tratar explícitamente la indisponibilidad del componente MCP y otros fallos parciales.

## Responsabilidad

Dify devuelve **análisis**, no prioridad. n8n conserva como autoritativos:

```text
severity
priority
status
requires_human_review
owner_team
```

Dentro del análisis debes diferenciar:

```text
observed_facts       → alert_context / n8n
operational_facts    → Agent + MCP
documented_guidance → RAG
hypotheses           → interpretación IA
```
