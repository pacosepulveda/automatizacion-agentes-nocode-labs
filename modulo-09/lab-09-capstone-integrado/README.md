# Laboratorio 9 — Caso integrado final

M09 es el **capstone** del curso. El objetivo no es aprender otro nodo ni otra función de IA, sino demostrar que puedes diseñar, construir, validar y defender una automatización técnica completa utilizando los patrones aprendidos en M01–M08.

El caso final será **Telvora Incident Triage & Response Advisor v1.0**.

## Qué debe integrar

```text
Webhook autenticado
→ validación y límites
→ API REST de activos
→ prioridad determinista
→ Dify Workflow
    ├─ RAG / documentación
    ├─ Agent FunctionCalling / MCP / estado operativo vivo
    └─ LLM final / Structured Output
→ política de revisión humana
→ siguiente paso simulado
→ evidencia
→ respuesta estructurada
→ trazabilidad y fallback
```

El capstone debe demostrar que entiendes la diferencia entre:

```text
REST → contrato conocido y datos autoritativos
RAG  → conocimiento documental
MCP  → herramientas y estado operativo vivo para el Agent
LLM  → interpretación y síntesis
n8n  → orquestación, reglas y políticas
```

## Dos fases

### M09.1 — Construcción

A partir de un briefing y contratos, construirás o adaptarás un MVP end-to-end. El enunciado define requisitos y criterios, pero no prescribe cada nodo.

### M09.2 — Validación y presentación

Ejecutarás la batería de casos, revisarás seguridad y resiliencia, documentarás decisiones y presentarás la solución justificando arquitectura, trade-offs y siguiente paso.

## Archivos

- [`ENUNCIADO.md`](ENUNCIADO.md): instrucciones del capstone.
- [`materiales/brief_capstone.md`](materiales/brief_capstone.md): necesidad del cliente y requisitos.
- [`materiales/contrato_capstone.md`](materiales/contrato_capstone.md): contratos de entrada, salida, IA/MCP y políticas autoritativas.
- [`materiales/casos_capstone.json`](materiales/casos_capstone.json): batería de validación CAP-01…CAP-11.
- [`materiales/telvora_runbook_portal_cliente.txt`](materiales/telvora_runbook_portal_cliente.txt): nuevo documento operativo para incorporar a RAG.
- [`materiales/checklist_validacion.md`](materiales/checklist_validacion.md): validación funcional, seguridad, MCP, resiliencia y trazabilidad.
- [`materiales/rubrica_capstone.md`](materiales/rubrica_capstone.md): criterios de evaluación.
- [`materiales/plantilla_informe_final.md`](materiales/plantilla_informe_final.md): memoria técnica final.
- [`materiales/plantilla_presentacion.md`](materiales/plantilla_presentacion.md): guion de defensa técnica.

## Restricción principal

La IA puede interpretar, recuperar documentación, consultar tools read-only y proponer comprobaciones, pero **no es autoridad sobre `priority`, `owner_team`, autenticación, aprobación, `next_step` ni ejecución de acciones**.

Todos los datos son ficticios y el siguiente paso operativo es simulado; no se realiza remediación real.