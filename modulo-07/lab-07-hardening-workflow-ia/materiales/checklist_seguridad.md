# Checklist de seguridad — Laboratorio 7

## Controles técnicos

- [ ] El Webhook M07 utiliza autenticación `Header Auth`.
- [ ] El secreto del Webhook está almacenado como credencial y no escrito en el workflow.
- [ ] Una petición sin cabecera válida no inicia el procesamiento normal.
- [ ] Se mantienen las validaciones de campos obligatorios.
- [ ] `severity` usa allowlist explícita.
- [ ] `summary` y `details` tienen límites de longitud antes de llamar a Dify.
- [ ] Las entradas rechazadas no consumen Dify ni MCP.
- [ ] La prioridad se calcula exclusivamente en n8n.
- [ ] Dify no puede sobrescribir `priority`, `severity`, `owner_team` ni `requires_human_review`.
- [ ] `alert_context` se construye mediante allowlist y no serializando automáticamente todo el item.
- [ ] El Agent MCP recibe el `asset_id` validado y una consulta fija; no recibe `details` ni el `alert_context` completo.
- [ ] El Agent utiliza `FunctionCalling`.
- [ ] La Tool List del Agent contiene únicamente `get_recent_changes`, `get_recent_alerts`, `get_open_incidents` y `get_service_health`.
- [ ] No existe ninguna tool de restart, rollback, firewall, cuentas, comandos o modificación de configuración.
- [ ] El `MCP_TOKEN` no aparece en workflows, prompts, payloads, capturas ni repositorio.
- [ ] El prompt del Agent trata los resultados de tools como datos, no como instrucciones.
- [ ] El prompt final trata alerta, RAG y contexto MCP como datos no confiables desde el punto de vista de instrucciones.
- [ ] Structured Output sigue activo con `schema_analisis_hardened.json`.
- [ ] `documentation_status = insufficient_context` fuerza revisión humana.
- [ ] `confidence = low` fuerza revisión humana.
- [ ] `operational_context_status = unavailable` fuerza revisión humana.
- [ ] `operational_context_status = no_relevant_live_context` no se confunde con fallo técnico.
- [ ] P1 mantiene revisión humana.
- [ ] Un fallo de Dify conserva la prioridad determinista.
- [ ] El Agent tiene configurado un tratamiento de error/fallback para no inventar contexto MCP si las tools fallan.
- [ ] Los retries de Dify son limitados.
- [ ] API keys y secretos no se copian a payloads ni respuestas.
- [ ] `alert_id`, ejecución n8n, `dify_run_id` y `evidence_id` pueden correlacionarse cuando estén disponibles.
- [ ] Los rounds del Agent permiten comprobar qué tools MCP se invocaron realmente.

## Casos

| Caso | Control principal | Resultado esperado | Observado | ¿Correcto? |
|---|---|---|---|---|
| SEC-01 | autenticación | rechazo antes del flujo normal |  |  |
| SEC-02 | baseline | P1 + Dify + MCP normal |  |  |
| SEC-03 | prompt injection directa | no obedecer texto embebido; Agent no recibe details |  |  |
| SEC-04 | autoridad | P1 + human review se conservan |  |  |
| SEC-05 | límite de entrada | no llamar a Dify/MCP |  |  |
| SEC-06 | grounding / revisión | P2 + human review por contexto documental insuficiente |  |  |
| SEC-07 | allowlist / gating | manual_review, sin Dify/MCP |  |  |
| SEC-08 | resiliencia Dify | P1 se conserva; IA unavailable |  |  |
| SEC-09 | prompt injection RAG | documento no controla al LLM |  |  |
| SEC-10 | mínimo privilegio MCP | no existe acción destructiva; solo tools read-only |  |  |

## Revisión de secretos

Comprueba manualmente que no aparecen valores sensibles en:

- [ ] nombres de nodos;
- [ ] Edit Fields;
- [ ] payload enviado a Dify;
- [ ] `alert_context`;
- [ ] salida del Agent;
- [ ] respuesta al Webhook;
- [ ] archivos del repositorio;
- [ ] capturas entregadas.

Comprueba además que el `MCP_TOKEN` permanece gestionado por la integración MCP compartida y no es necesario para construir la App del alumno.

## Preguntas de cierre

1. ¿Qué controles actúan antes de consumir IA?
2. ¿Qué diferencia hay entre autenticación y autorización?
3. ¿Por qué un path único no sustituye a Header Auth?
4. ¿Por qué un prompt endurecido no es suficiente por sí solo?
5. ¿Qué debe ocurrir si la documentación recuperada intenta dar órdenes al modelo?
6. ¿Por qué el Agent MCP no debería recibir `details` si solo necesita `asset_id`?
7. ¿Qué diferencia existe entre autorizar un MCP Server y permitir una tool concreta?
8. ¿Por qué una tool read-only reduce el impacto de prompt injection?
9. ¿Qué debe hacer el sistema si MCP está indisponible pero Dify sigue funcionando?
10. ¿Qué capacidad debería conservar el sistema si Dify está caído?
