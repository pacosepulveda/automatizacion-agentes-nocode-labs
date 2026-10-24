# Release Manifest — Plantilla M08

## Identificación

```text
release_id: M08-<ID_ALUMNO>-v1
fecha:
owner:
approver:
rollback_owner:
escalation_contact:
```

## Componentes

| Componente | Recurso | Versión / referencia | Estado |
|---|---|---|---|
| n8n | M07-LAB07-Hardened-AI-Triage | baseline M07 |  |
| Dify | ALxx-M07-Analisis-RAG-MCP-Hardened | versión publicada |  |
| Agent Strategy | Dify Agent Strategies / FunctionCalling | Workspace |  |
| MCP | Telvora Ops Hub | integración compartida / 4 tools read-only |  |
| Knowledge Base | ALxx-M05-KB-Ops | documentos vigentes |  |
| Prompt final | prompt_hardened.md | M07 |  |
| Prompt Agent | prompt_agent_mcp_hardened.md | M07 |  |
| Schema | schema_analisis_hardened.json | M07 |  |
| Lab API | api.cloudsecurityinpractice.com | entorno curso |  |
| Evidence API | POST /api/v1/evidence | contrato validado / pendiente |  |

## Contratos autoritativos

```text
severity -> evento validado
asset metadata -> Lab API
priority -> reglas n8n
requires_human_review -> política n8n
analysis -> Dify
runbook guidance -> Knowledge Base
operational live context -> Telvora Ops Hub / MCP
```

## Secretos / credenciales requeridos

Enumera únicamente el **nombre de las Credentials**, nunca sus valores.

```text
Webhook Auth:
Lab API:
Dify Workflow API:
MCP integration: gestionada en el Workspace por el instructor
```

## Dependencias

```text
n8n:
Dify:
Agent Strategy plugin:
Telvora Ops Hub / MCP:
Postgres/Redis/Weaviate: gestionados por la plataforma
Amazon Bedrock:
Cohere Rerank:
Lab API:
Evidence API:
```

## Estados degradados que debe soportar

```text
Dify unavailable
MCP unavailable
Evidence API unavailable
```

Describe para cada uno qué capacidad se pierde y qué decisión debe preservarse.

## Known issues

| ID | Descripción | Severidad | Control compensatorio | Owner | ¿Bloquea piloto? |
|---|---|---|---|---|---|
|  |  |  |  |  |  |

## Rollback target

Define la última configuración conocida como buena:

```text
n8n baseline:
Dify baseline:
prompt final baseline:
prompt Agent baseline:
schema baseline:
KB baseline:
MCP server/integration baseline:
```

## Cambios de esta release

```text

```

## Aprobaciones

```text
Workflow owner:            [ ]
Security reviewer:         [ ]
Pilot approver:            [ ]
Rollback owner informado:  [ ]
```
