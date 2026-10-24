# Laboratorio 8 — Preparación de piloto y operación

En este laboratorio convertiremos el workflow endurecido de M07 en un **candidato a piloto controlado**. No añadiremos una nueva capacidad de IA: trabajaremos sobre ownership, release gate, observabilidad, operación, rollback y decisión de readiness.

La novedad respecto al diseño original es que **MCP se trata como una dependencia operativa independiente**. Dify puede estar disponible mientras Telvora Ops Hub/MCP no lo está, por lo que debemos distinguir ambos estados.

## Qué se hará

1. definir owner, approver, rollback owner y escalation path;
2. crear un manifest de release del candidato, incluyendo Telvora Ops Hub/MCP;
3. establecer gates funcionales, de seguridad y resiliencia;
4. definir un rollout en shadow/limited pilot;
5. registrar métricas de n8n, Dify, MCP y Evidence API;
6. simular una jornada operativa con siete escenarios;
7. diferenciar degradación de Dify, MCP y Evidence API;
8. realizar un rollback drill;
9. completar un postmortem de un incidente controlado;
10. emitir una decisión final:

```text
READY
READY_WITH_CONDITIONS
NOT_READY
```

## Archivos

- [`ENUNCIADO.md`](ENUNCIADO.md): instrucciones completas.
- [`materiales/release_manifest_template.md`](materiales/release_manifest_template.md): inventario/versionado del candidato.
- [`materiales/checklist_readiness.md`](materiales/checklist_readiness.md): release gate y criterios de readiness.
- [`materiales/escenarios_operacion.json`](materiales/escenarios_operacion.json): jornada operativa simulada.
- [`materiales/registro_metricas.csv`](materiales/registro_metricas.csv): plantilla para registrar ejecuciones.
- [`materiales/runbook_operacion_template.md`](materiales/runbook_operacion_template.md): runbook mínimo del piloto.
- [`materiales/postmortem_template.md`](materiales/postmortem_template.md): postmortem del incidente controlado.
- [`materiales/decision_piloto_template.md`](materiales/decision_piloto_template.md): decisión final de readiness.

## Prerrequisitos

- M07 completado y funcional.
- Workflow n8n endurecido.
- Workflow Dify endurecido con Agent MCP.
- Knowledge Base privada de M05.
- Telvora Ops Hub configurado en el Workspace Dify.
- Casos SEC-01…SEC-10 ejecutados o preparados para su validación final.
- Matriz de riesgos y política mínima de M07.

Todos los datos son ficticios y el laboratorio se realiza sobre el entorno formativo.