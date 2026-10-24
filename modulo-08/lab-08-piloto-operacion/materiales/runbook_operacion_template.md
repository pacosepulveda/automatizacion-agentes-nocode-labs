# Runbook de operación — Plantilla M08

## 1. Servicio

```text
Nombre:
Release ID:
Workflow n8n:
Workflow Dify:
Knowledge Base:
MCP Server: Telvora Ops Hub
Owner:
Escalation contact:
```

## 2. Qué hace el sistema

Describe en 5-10 líneas:

```text

```

## 3. Qué NO hace

```text
- no ejecuta remediación automática;
- no permite que Dify cambie priority;
- no permite tools MCP destructivas;
- no elimina human review cuando la policy la exige;
- ...
```

## 4. Smoke test

Pasos mínimos para comprobar salud:

1. comprobar FQDN de n8n;
2. comprobar FQDN de Dify;
3. comprobar `https://api.cloudsecurityinpractice.com/health`;
4. verificar que Telvora Ops Hub está autorizado en Dify y expone 4 tools read-only;
5. ejecutar un caso baseline autenticado;
6. verificar prioridad;
7. verificar `ai_status`;
8. verificar `operational_context_status`;
9. verificar ejecución n8n;
10. verificar `dify_run_id` cuando exista;
11. verificar evidencia cuando aplique.

Resultado esperado:

```text

```

## 5. Dify no disponible

Síntomas:

```text

```

Comprobaciones:

```text

```

Comportamiento esperado del workflow:

```text
priority preservada
ai_status = unavailable
human review = true
```

Escalado:

```text

```

## 6. MCP / Telvora Ops Hub no disponible

Síntomas:

```text
Agent error / fallback MCP_UNAVAILABLE
operational_context_status = unavailable
```

Comprobaciones:

```text
- salud de api.cloudsecurityinpractice.com;
- estado de la integración MCP en Dify;
- tool list del Agent;
- logs/rounds del Agent;
```

Comportamiento esperado:

```text
priority preservada
Dify puede seguir aportando RAG/LLM si el fallback funciona
operational_facts no se inventan
operational_context_status = unavailable
human review = true
```

Qué NO hacer:

```text
no añadir tools destructivas como solución temporal
no copiar MCP_TOKEN a la App o a n8n
```

## 7. Fallos de autenticación en Webhook

Señal:

```text

```

Qué comprobar:

```text

```

Qué NO hacer:

```text
no desactivar autenticación para restaurar servicio
```

## 8. Documentación insuficiente / confidence low

Comportamiento esperado:

```text

```

Responsable de revisión:

```text

```

## 9. Evidence API no disponible

Comportamiento esperado:

```text

```

¿La decisión principal puede continuar?:

```text

```

Cómo registrar temporalmente el incidente:

```text

```

## 10. Rollback

Trigger de rollback:

```text

```

Configuración conocida como buena:

```text

```

Pasos:

```text
1.
2.
3.
4.
```

Smoke test posterior:

```text

```

## 11. Detener el piloto

Condiciones de stop inmediato:

```text
- bypass de autenticación;
- exposición de secretos;
- IA altera una decisión autoritativa;
- una tool MCP destructiva aparece sin aprobación;
- MCP failure provoca hechos operativos inventados;
- no funciona fallback;
- ...
```

Procedimiento:

```text

```

## 12. Datos para diagnóstico

Conservar:

```text
alert_id
n8n execution id
dify_run_id
Agent rounds/tool calls cuando sea necesario
evidence_id
ai_status
operational_context_status
status/errores
latencias
```

No registrar:

```text
API keys
tokens
MCP_TOKEN
Authorization headers
passwords
```
