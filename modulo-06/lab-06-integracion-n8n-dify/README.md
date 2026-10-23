# Laboratorio 6 — Integración n8n + Dify

En este laboratorio se integran por primera vez las dos plataformas principales del curso en un único flujo: **n8n** como capa de orquestación y control determinista, y **Dify** como componente de análisis con IA y RAG.

## Qué se construirá

La automatización recibirá una alerta mediante Webhook, validará y enriquecerá los datos con la API del laboratorio, calculará la prioridad con reglas deterministas y, solo para alertas válidas, invocará un **Workflow publicado de Dify** mediante su API de servicio.

Dify utilizará la Knowledge Base privada creada en M05 para recuperar documentación relevante y devolverá un análisis estructurado. n8n conservará la autoridad sobre la prioridad, registrará la evidencia y responderá al sistema emisor.

```text
Webhook
  ↓
Validación
  ↓
API de activos
  ↓
Enriquecimiento
  ↓
Clasificación determinista
  ↓
Dify Workflow + RAG
  ↓
Validación de respuesta IA
  ↓
Evidencia
  ↓
Respuesta HTTP
```

## Principio de arquitectura

```text
n8n  → orquestación, contratos, reglas, credenciales y control
Dify → retrieval, interpretación y análisis técnico
Humano → decisión sensible y autorización de acciones
```

Dify **no decide la prioridad** y no ejecuta remediaciones.

## Archivos

- [`ENUNCIADO.md`](ENUNCIADO.md): instrucciones completas.
- [`materiales/casos_integracion.json`](materiales/casos_integracion.json): casos de prueba.
- [`materiales/schema_analisis_dify.json`](materiales/schema_analisis_dify.json): contrato de salida estructurada del Workflow de Dify.
- [`materiales/prompt_workflow_dify.md`](materiales/prompt_workflow_dify.md): prompt del nodo LLM.
- [`materiales/contrato_dify_api.md`](materiales/contrato_dify_api.md): contrato de invocación del Workflow desde n8n.
- [`materiales/checklist_pruebas.md`](materiales/checklist_pruebas.md): checklist de integración y resiliencia.

## Prerrequisitos

- M03 completado o un workflow equivalente de Webhook + API.
- M05 completado y una Knowledge Base privada funcional.
- Cuenta personal de n8n.
- Cuenta de Dify.
- Identificador de alumno (`alumno01`, `alumno02`, ...).

Todos los datos son ficticios y están destinados exclusivamente a formación.
