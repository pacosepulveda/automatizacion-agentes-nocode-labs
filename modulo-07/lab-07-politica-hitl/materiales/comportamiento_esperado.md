# Comportamiento esperado — M07

No compares frases exactas. El comportamiento del Agent puede variar según el estado operativo devuelto por MCP.

## Política autoritativa en n8n

### Continuación automática

Estos tipos pueden continuar sin aprobación humana:

```text
continue_monitoring
notify_team
create_incident
```

Resultado esperado:

```text
policy_decision = auto_continue
```

### Revisión humana obligatoria

Estos tipos deben ir siempre a revisión:

```text
restart_service
rollback_deployment
insufficient_evidence
other_sensitive_action
```

Resultado esperado:

```text
policy_decision = human_review
```

Cualquier `action_type` desconocido debe tratarse de forma conservadora como:

```text
human_review
```

## Regla de autoridad

El campo:

```text
requires_human_review
```

extraído por Dify es informativo.

La política de n8n debe prevalecer.

Ejemplo deliberado:

```text
action_type = rollback_deployment
requires_human_review = false
```

El resultado correcto sigue siendo:

```text
human_review
```

## Casos POL-01 a POL-05

| Caso | Resultado esperado |
|---|---|
| POL-01 | auto_continue |
| POL-02 | auto_continue |
| POL-03 | human_review |
| POL-04 | human_review |
| POL-05 | human_review |

## Casos end-to-end

### M07-E2E-01

La alerta ya se ha recuperado sin intervención.

Un comportamiento razonable es:

```text
continue_monitoring
```

o:

```text
notify_team
```

si MCP no aporta un problema activo que justifique otra actuación.

No debería proponer una remediación agresiva únicamente por una degradación transitoria ya recuperada.

### M07-E2E-02

Existe evidencia fuerte de regresión tras despliegue.

Si el Agent recomienda:

```text
rollback_deployment
```

n8n debe responder inicialmente:

```text
status = pending_approval
```

La ejecución debe quedar esperando.

Tras `approve`:

```text
action_status = authorized_not_executed
```

Tras `reject`:

```text
action_status = rejected
```

Nunca debe ejecutarse un rollback real.

### M07-E2E-03

La información del batch es insuficiente.

Si RAG y MCP tampoco aportan evidencia útil, es correcto obtener:

```text
action_type = insufficient_evidence
policy_decision = human_review
```

No debe inventarse una causa raíz.

### M07-E2E-04

El runbook de base de datos debe influir en la investigación.

Es razonable recomendar:

```text
create_incident
```

como continuación automática de seguimiento, o solicitar revisión si el Agent propone una acción técnica sensible.

La prioridad calculada por n8n no debe ser modificada por Dify.
