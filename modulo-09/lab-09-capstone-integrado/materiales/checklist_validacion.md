# Checklist de validación — M09

## A. Arquitectura y contratos

- [ ] Existe diagrama end-to-end.
- [ ] Las trust boundaries están identificadas.
- [ ] La entrada cumple `contrato_capstone.md`.
- [ ] La salida final es estable y distingue negocio, IA y evidencia.
- [ ] `priority` tiene una única fuente autoritativa.
- [ ] `next_step` se calcula fuera del LLM.

## B. Funcionalidad

- [ ] CAP-01 produce P1 y análisis IA.
- [ ] CAP-02 produce P1 y no inventa causa raíz.
- [ ] CAP-03 produce P3 y análisis relevante.
- [ ] CAP-04 produce P2 y revisión humana si la documentación es insuficiente.
- [ ] CAP-05 se desvía a revisión manual sin llamar a Dify.
- [ ] CAP-06 se desvía por activo desconocido sin clasificar automáticamente.
- [ ] CAP-10 recupera el runbook de portal de clientes.

## C. Seguridad

- [ ] CAP-08 sin autenticación es rechazado.
- [ ] CAP-07 no modifica `priority` ni `requires_human_review`.
- [ ] CAP-07 no produce una falsa afirmación de remediación.
- [ ] Los secretos están en Credentials.
- [ ] Solo una allowlist de campos llega a Dify.
- [ ] Los documentos RAG se tratan como datos, no como instrucciones privilegiadas.

## D. Resiliencia

- [ ] CAP-09 conserva P1 cuando Dify no está disponible.
- [ ] CAP-09 marca `ai_status` explícitamente.
- [ ] CAP-09 fuerza revisión humana.
- [ ] Los retries son limitados.
- [ ] Un fallo de Evidence API no inventa `evidence_id`.

## E. RAG

- [ ] El nuevo `telvora_runbook_portal_cliente.txt` está indexado.
- [ ] CAP-10 recupera evidencia relevante del nuevo documento.
- [ ] El análisis distingue hechos de orientación documentada.
- [ ] La ausencia de documentación se refleja como contexto insuficiente.
- [ ] No se completa información interna utilizando conocimiento general no documentado.

## F. Trazabilidad

- [ ] `alert_id` se conserva de extremo a extremo.
- [ ] Puede localizarse la ejecución n8n.
- [ ] Se conserva `dify_run_id` cuando está disponible.
- [ ] Se conserva `evidence_id` cuando el registro ha tenido éxito.
- [ ] Se puede identificar la regla que produjo `priority`.
- [ ] Se puede saber si Dify fue llamado o no.

## G. Operación

- [ ] Existe una configuración conocida como buena.
- [ ] Existe fallback IA.
- [ ] Existe criterio de revisión humana.
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

## Validación final

```text
[ ] MVP construido
[ ] contratos cumplidos
[ ] casos normales ejecutados
[ ] casos adversariales ejecutados
[ ] fallback probado
[ ] trazabilidad comprobada
[ ] arquitectura documentada
[ ] presentación preparada
```
