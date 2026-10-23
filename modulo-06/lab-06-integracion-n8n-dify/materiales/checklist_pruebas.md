# Checklist de pruebas — Laboratorio 6

Completa la predicción antes de ejecutar cada caso.

| Caso | Prioridad esperada | ¿Debe llamarse Dify? | Resultado Dify esperado | Resultado observado | ¿Correcto? | Observaciones |
|---|---|---|---|---|---|---|
| INT-01 | P1 | Sí | análisis RAG web |  |  |  |
| INT-02 | P1 | Sí | análisis RAG DB |  |  |  |
| INT-03 | P3 | Sí | contexto API/cambios |  |  |  |
| INT-04 | P2 | Sí | contexto documental insuficiente para runbook batch |  |  |  |
| INT-05 | — | No | — |  |  |  |
| INT-06 | — | No | — |  |  |  |
| INT-07 | — | No | — |  |  |  |
| INT-08 | P4 | Sí | análisis sin cambiar prioridad |  |  |  |

## Validación de arquitectura

- [ ] n8n calcula la prioridad antes de consumir la respuesta de Dify.
- [ ] Dify no devuelve ni modifica `priority` como dato autoritativo.
- [ ] Solo las alertas válidas y enriquecidas llaman a Dify.
- [ ] INT-05 no ejecuta el HTTP Request de Dify.
- [ ] INT-06 no ejecuta el HTTP Request de Dify.
- [ ] INT-07 no ejecuta el HTTP Request de Dify.
- [ ] La API key de Dify está almacenada en Credentials de n8n.
- [ ] El Workflow de Dify está publicado antes de invocarlo.
- [ ] La respuesta de Dify se valida mediante `data.status` y la existencia de `data.outputs.analysis`.
- [ ] Se conserva el `workflow_run_id` o identificador equivalente para trazabilidad.
- [ ] La evidencia final conserva prioridad determinista y resultado del análisis IA.
- [ ] La respuesta HTTP al emisor diferencia datos deterministas de análisis IA.

## Prueba de fallo controlado de Dify

Provoca temporalmente un fallo de integración cambiando la URL del nodo de Dify a una ruta inexistente, por ejemplo:

```text
https://dify.cloudsecurityinpractice.com/v1/workflows/runn
```

Ejecuta INT-01 y documenta:

- [ ] La prioridad P1 ya estaba calculada antes del fallo.
- [ ] El nodo de Dify muestra el error técnico esperado.
- [ ] Has identificado dónde implantarías una rama de fallback.
- [ ] La política propuesta no recalcula la prioridad con IA.
- [ ] El fallback registra que el análisis IA no está disponible.
- [ ] Tras restaurar la URL, INT-01 vuelve a funcionar.

## Preguntas de cierre

1. ¿Por qué Dify se invoca después de validación, enriquecimiento y clasificación?
2. ¿Qué ventaja tiene utilizar una App Workflow en Dify para una integración máquina a máquina?
3. ¿Por qué la API key de Dify no debe estar escrita en el nodo HTTP Request?
4. ¿Qué información perderíamos si después de `HTTP Request` solo utilizáramos `$json` sin referenciar nodos anteriores?
5. ¿Qué diferencia existe entre `HTTP 200` y `data.status = succeeded`?
6. ¿Qué debería ocurrir si Dify no está disponible?
7. ¿Qué datos son autoritativos de n8n y cuáles son recomendaciones de IA?
8. ¿Qué identificadores guardarías para reconstruir una ejecución de extremo a extremo?
