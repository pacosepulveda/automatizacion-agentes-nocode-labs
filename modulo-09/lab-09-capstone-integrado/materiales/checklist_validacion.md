# Checklist de validación — M09

## A. Arquitectura y contratos

- [ ] Existe diagrama end-to-end.
- [ ] Las trust boundaries están identificadas.
- [ ] La entrada cumple `contrato_capstone.md`.
- [ ] La salida final es estable y distingue negocio, IA, MCP y evidencia.
- [ ] `priority` tiene una única fuente autoritativa.
- [ ] `next_step` se calcula fuera del LLM.
- [ ] Se justifica por qué se usa REST, RAG y MCP en partes distintas.

## B. Funcionalidad

- [ ] CAP-01 produce P1, análisis IA y consulta MCP real.
- [ ] CAP-02 produce P1 y no inventa causa raíz.
- [ ] CAP-03 produce P3 y análisis relevante.
- [ ] CAP-04 produce P2 y revisión humana si la documentación es insuficiente.
- [ ] CAP-05 se desvía a revisión manual sin llamar a Dify/MCP.
- [ ] CAP-06 se desvía por activo desconocido sin clasificar automáticamente.
- [ ] CAP-10 recupera el runbook de portal de clientes.

## C. MCP / herramientas

- [ ] El Agent utiliza `FunctionCalling`.
- [ ] El Agent expone exactamente las cuatro tools read-only de Telvora Ops Hub.
- [ ] El Agent recibe únicamente el contexto mínimo necesario para investigar el activo.
- [ ] Puede demostrarse al menos una tool call real mediante los rounds del Agent.
- [ ] `operational_facts` procede únicamente del contexto MCP reportado como disponible.
- [ ] `no_relevant_live_context` se distingue de `unavailable`.
- [ ] CAP-07 no puede provocar restart/rollback ni otras acciones no expuestas.
- [ ] CAP-11 degrada MCP sin inventar operational facts.

## D. Seguridad

- [ ] CAP-08 sin autenticación es rechazado.
- [ ] CAP-07 no modifica `priority` ni `requires_human_review`.
- [ ] CAP-07 no produce una falsa afirmación de remediación.
- [ ] Los secretos están en Credentials/integraciones gestionadas.
- [ ] Solo una allowlist de campos llega a Dify.
- [ ] Los documentos RAG y resultados MCP se tratan como datos, no como instrucciones privilegiadas.
- [ ] `MCP_TOKEN` no aparece en workflow, payloads, capturas ni repositorio.

## E. Resiliencia

- [ ] CAP-09 conserva P1 cuando Dify no está disponible.
- [ ] CAP-09 marca `ai_status` explícitamente.
- [ ] CAP-09 fuerza revisión humana.
- [ ] CAP-11 conserva P1 cuando MCP no está disponible.
- [ ] CAP-11 mantiene Dify/RAG disponibles si el fallback MCP funciona.
- [ ] CAP-11 marca `operational_context_status=unavailable`.
- [ ] CAP-11 fuerza revisión humana.
- [ ] Los retries son limitados.
- [ ] Un fallo de Evidence API no inventa `evidence_id`.

## F. RAG

- [ ] El nuevo `telvora_runbook_portal_cliente.txt` está indexado.
- [ ] CAP-10 recupera evidencia relevante del nuevo documento.
- [ ] El análisis distingue hechos validados, hechos operativos, orientación documentada e hipótesis.
- [ ] La ausencia de documentación se refleja como contexto insuficiente.
- [ ] No se completa información interna utilizando conocimiento general no documentado.

## G. Trazabilidad

- [ ] `alert_id` se conserva de extremo a extremo.
- [ ] Puede localizarse la ejecución n8n.
- [ ] Se conserva `dify_run_id` cuando está disponible.
- [ ] Pueden inspeccionarse Agent rounds/tool calls para una ejecución válida.
- [ ] Se conserva `evidence_id` cuando el registro ha tenido éxito.
- [ ] Se puede identificar la regla que produjo `priority`.
- [ ] Se puede saber si Dify y MCP fueron llamados o no.

## H. Operación

- [ ] Existe una configuración conocida como buena.
- [ ] Existe fallback Dify.
- [ ] Existe fallback MCP.
- [ ] Existe criterio determinista de revisión humana.
- [ ] La solución no ejecuta remediación real.
- [ ] Los known issues relevantes están documentados.

---

# Registro de casos

| Caso | Resultado esperado por el equipo | Resultado observado | PASS/FAIL | Evidencia / execution ID |
|---|---|---|---|---|
| CAP-01 |  |  |  |  |
| CAP-02 |  |  |  |  |
| CAP-03 |  |  |  |  |
| CAP-04 |  |  |  |  |
| CAP-05 |  |  |  |  |
| CAP-06 |  |  |  |  |
| CAP-07 |  |  |  |  |
| CAP-08 |  |  |  |  |
| CAP-09 |  |  |  |  |
| CAP-10 |  |  |  |  |
| CAP-11 |  |  |  |  |

## Validación final

```text
[ ] MVP construido
[ ] contratos cumplidos
[ ] casos normales ejecutados
[ ] casos adversariales ejecutados
[ ] fallback Dify probado
[ ] fallback MCP probado
[ ] trazabilidad comprobada
[ ] arquitectura documentada
[ ] presentación preparada
```
