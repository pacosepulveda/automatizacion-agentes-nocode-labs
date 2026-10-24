# Checklist de pruebas — Laboratorio 6

Completa la predicción antes de ejecutar cada caso.

> El contenido devuelto por MCP es estado vivo. No se evalúa que dos alumnos obtengan exactamente el mismo JSON, sino que la tool adecuada se invoque y que el análisis trate correctamente el resultado observado.

| Caso | Prioridad esperada | ¿Dify? | ¿MCP? | RAG / contexto esperado | Resultado observado | ¿Correcto? | Observaciones |
|---|---|---|---|---|---|---|---|
| INT-01 | P1 | Sí | Sí | runbook web + estado vivo |  |  |  |
| INT-02 | P1 | Sí | Sí | DB-HEALTH-01 + estado vivo DB |  |  |  |
| INT-03 | P3 | Sí | Sí | inventario/cambios + estado vivo API |  |  |  |
| INT-04 | P2 | Sí | Sí | runbook batch insuficiente; MCP puede ser vacío |  |  |  |
| INT-05 | — | No | No | — |  |  |  |
| INT-06 | — | No | No | — |  |  |  |
| INT-07 | — | No | No | — |  |  |  |
| INT-08 | P4 | Sí | Sí | análisis sin cambiar prioridad |  |  |  |

## Validación de arquitectura

- [ ] n8n calcula la prioridad antes de consumir la respuesta de Dify.
- [ ] Dify no devuelve ni modifica `priority` como dato autoritativo.
- [ ] Solo las alertas válidas y enriquecidas llaman a Dify.
- [ ] INT-05 no ejecuta el HTTP Request de Dify.
- [ ] INT-06 no ejecuta el HTTP Request de Dify.
- [ ] INT-07 no ejecuta el HTTP Request de Dify.
- [ ] La API key de Dify está almacenada en Credentials de n8n.
- [ ] El token MCP no está almacenado en el workflow del alumno.
- [ ] Se utiliza el MCP compartido `Telvora Ops Hub` configurado por el instructor.
- [ ] El nodo Agent utiliza la estrategia `FunctionCalling`.
- [ ] El Tool List del Agent contiene únicamente las cuatro tools de Telvora Ops Hub.
- [ ] La Knowledge Base seleccionada pertenece al alumno.
- [ ] El Workflow de Dify está publicado antes de invocarlo desde n8n.
- [ ] La respuesta de Dify se valida mediante `data.status` y la existencia de `data.outputs.analysis`.
- [ ] Se conserva el `workflow_run_id` o identificador equivalente para trazabilidad.
- [ ] La evidencia final conserva prioridad determinista y resultado del análisis IA.
- [ ] La respuesta HTTP al emisor diferencia datos deterministas de análisis IA.

## Validación explícita de MCP

Selecciona una ejecución válida, preferentemente INT-01.

En Dify abre:

```text
Agent
→ Detalle
→ Estrategia agéntica
→ Round
```

Comprueba:

- [ ] aparece al menos un `tool_call_name` de Telvora Ops Hub;
- [ ] el `asset_id` enviado coincide con el activo de la alerta;
- [ ] puedes distinguir `tool_call_input` y `tool_response`;
- [ ] una lista vacía de la tool se trata como un resultado válido;
- [ ] la respuesta final no presenta una hipótesis como un hecho obtenido por MCP;
- [ ] la proximidad temporal entre un cambio y un incidente no se presenta como causalidad demostrada;
- [ ] no se ha invocado ninguna acción destructiva porque el MCP no expone ese tipo de tools.

Anota una llamada observada:

```text
tool_call_name:
tool_call_input:
resumen del tool_response:
```

No copies tokens ni headers.

## Validación de las fuentes

- [ ] `observed_facts` procede de `alert_context`.
- [ ] `operational_facts` procede de la investigación MCP.
- [ ] `documented_guidance` procede del contexto RAG.
- [ ] `hypotheses` se expresa como posibilidad.
- [ ] si RAG no aporta contexto relevante, `documentation_status = insufficient_context`.
- [ ] si MCP no aporta hechos vivos relevantes, `operational_context_status = no_relevant_live_context`.
- [ ] si MCP aporta hechos vivos relevantes, `operational_context_status = relevant_live_context_found`.

## Prueba de fallo controlado de Dify

Provoca temporalmente un fallo de integración cambiando la URL del nodo de Dify a una ruta inexistente:

```text
https://dify.cloudsecurityinpractice.com/v1/workflows/runn
```

Ejecuta INT-01 y documenta:

- [ ] La prioridad P1 ya estaba calculada antes del fallo.
- [ ] El nodo de Dify muestra el error técnico esperado.
- [ ] Al no ejecutarse Dify, tampoco se ejecutan RAG ni MCP.
- [ ] Has identificado dónde implantarías una rama de fallback.
- [ ] La política propuesta no recalcula la prioridad con IA.
- [ ] El fallback registra que el análisis IA no está disponible.
- [ ] Tras restaurar la URL, INT-01 vuelve a funcionar.

> La indisponibilidad específica del servidor MCP y su fallback se trabajará con más detalle en M07.

## Preguntas de cierre

1. ¿Por qué Dify se invoca después de validación, enriquecimiento y clasificación?
2. ¿Qué diferencia conceptual existe entre REST, RAG y MCP en este laboratorio?
3. ¿Por qué el catálogo de activos se consulta de forma determinista por REST y el estado operativo vivo mediante MCP?
4. ¿Qué ventaja tiene utilizar una App Workflow en Dify para integración máquina a máquina?
5. ¿Qué hace `FunctionCalling` en el nodo Agent?
6. ¿Por qué el token MCP no se entrega a cada alumno?
7. ¿Por qué una lista vacía de `get_recent_alerts` no es un error?
8. ¿Qué diferencia existe entre un hecho MCP y una hipótesis del LLM?
9. ¿Por qué la API key de Dify no debe estar escrita en el nodo HTTP Request?
10. ¿Qué información perderíamos si después de `HTTP Request` solo utilizáramos `$json` sin referenciar nodos anteriores?
11. ¿Qué diferencia existe entre `HTTP 200` y `data.status = succeeded`?
12. ¿Qué debería ocurrir si Dify no está disponible?
13. ¿Qué identificadores guardarías para reconstruir una ejecución de extremo a extremo?
