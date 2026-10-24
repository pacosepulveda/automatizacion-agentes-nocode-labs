# Laboratorio 6 — Integración n8n + Dify + RAG + MCP

En este laboratorio se integran las piezas principales del curso en un único flujo: **n8n** como capa de orquestación y control determinista, **Dify + RAG** como fuente de conocimiento documental y un **Agent con MCP** para consultar estado operativo vivo en Telvora Ops Hub.

## Qué se construirá

La automatización recibirá una alerta mediante Webhook, validará y enriquecerá los datos con la API del laboratorio, calculará la prioridad con reglas deterministas y, solo para alertas válidas, invocará un **Workflow publicado de Dify** mediante su API de servicio.

Dentro de Dify se combinarán tres fuentes:

```text
alert_context            → hechos validados por n8n
Knowledge Retrieval/RAG → documentación y runbooks
Agent + MCP             → cambios, alertas, incidentes y health actual
```

Un LLM final sintetizará esas fuentes mediante Structured Output. n8n conservará la autoridad sobre la prioridad, registrará la evidencia y responderá al sistema emisor.

```text
Webhook
  ↓
Validación
  ↓
API de activos / REST
  ↓
Enriquecimiento
  ↓
Clasificación determinista
  ↓
Dify Workflow
  ├─ Knowledge Retrieval / RAG
  ├─ Agent FunctionCalling / MCP
  └─ LLM final / Structured Output
  ↓
Validación de respuesta IA
  ↓
Evidencia
  ↓
Respuesta HTTP
```

## Principio de arquitectura

```text
n8n  → orquestación, contratos, APIs deterministas, reglas, credenciales y control
RAG  → conocimiento documentado relativamente estable
MCP  → herramientas y estado operativo vivo
LLM  → interpretación y síntesis
Humano → decisión sensible y autorización de acciones
```

Dify **no decide la prioridad** y el servidor MCP del laboratorio **no expone remediaciones destructivas**.

## Integraciones compartidas

El instructor deja preparadas en el Workspace de Dify:

```text
Telvora Ops Hub     → servidor MCP compartido
Dify Agent Strategies → FunctionCalling / ReAct
```

Los alumnos no deben reinstalarlas ni gestionar el token MCP. Cada alumno crea únicamente su App con su prefijo y selecciona las tools necesarias.

## Archivos

- [`ENUNCIADO.md`](ENUNCIADO.md): instrucciones completas.
- [`materiales/casos_integracion.json`](materiales/casos_integracion.json): casos de prueba.
- [`materiales/schema_analisis_dify.json`](materiales/schema_analisis_dify.json): contrato de salida estructurada.
- [`materiales/prompt_agent_mcp.md`](materiales/prompt_agent_mcp.md): instrucciones del Agent que consulta MCP.
- [`materiales/prompt_workflow_dify.md`](materiales/prompt_workflow_dify.md): prompt del LLM final que sintetiza alerta + RAG + MCP.
- [`materiales/contrato_mcp_telvora_ops_hub.md`](materiales/contrato_mcp_telvora_ops_hub.md): contrato de las tools MCP.
- [`materiales/contrato_dify_api.md`](materiales/contrato_dify_api.md): contrato de invocación del Workflow desde n8n.
- [`materiales/checklist_pruebas.md`](materiales/checklist_pruebas.md): checklist de integración, MCP y resiliencia.

## Prerrequisitos

- M03 completado o un workflow equivalente de Webhook + API.
- M05 completado y una Knowledge Base privada funcional.
- Cuenta personal de n8n.
- Cuenta de Dify en el Workspace del curso.
- Identificador de alumno (`alumno01`, `alumno02`, ...).

Todos los datos son ficticios y están destinados exclusivamente a formación.
