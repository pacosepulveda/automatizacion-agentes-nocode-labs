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
| Dify | ALxx-M07-Analisis-RAG-Hardened | versión publicada |  |
| Knowledge Base | ALxx-M05-KB-Ops | documentos vigentes |  |
| Prompt | prompt_hardened.md | M07 |  |
| Schema | schema_analisis_dify.json | M06 |  |
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
```

## Secretos / credenciales requeridos

Enumera únicamente el **nombre de las Credentials**, nunca sus valores.

```text
Webhook Auth:
Lab API:
Dify Workflow API:
```

## Dependencias

```text
n8n:
Dify:
Postgres/Redis/Weaviate: gestionados por la plataforma
Amazon Bedrock:
Cohere Rerank:
Lab API:
Evidence API:
```

## Known issues

| ID | Descripción | Severidad | Control compensatorio | Owner | ¿Bloquea piloto? |
|---|---|---|---|---|---|
|  |  |  |  |  |  |

## Rollback target

Define la última configuración conocida como buena:

```text
n8n baseline:
Dify baseline:
prompt baseline:
KB baseline:
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
