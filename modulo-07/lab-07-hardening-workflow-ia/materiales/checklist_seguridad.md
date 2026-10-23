# Checklist de seguridad — Laboratorio 7

## Controles técnicos

- [ ] El Webhook M07 utiliza autenticación `Header Auth`.
- [ ] El secreto del Webhook está almacenado como credencial y no escrito en el workflow.
- [ ] Una petición sin cabecera válida no inicia el procesamiento normal.
- [ ] Se mantienen las validaciones de campos obligatorios.
- [ ] `severity` usa allowlist explícita.
- [ ] `summary` y `details` tienen límites de longitud antes de llamar a Dify.
- [ ] Las entradas rechazadas no consumen Dify.
- [ ] La prioridad se calcula exclusivamente en n8n.
- [ ] Dify no puede sobrescribir `priority`, `severity`, `owner_team` ni `requires_human_review`.
- [ ] El prompt trata alerta y documentos como datos no confiables desde el punto de vista de instrucciones.
- [ ] Structured Output sigue activo.
- [ ] `documentation_status = insufficient_context` fuerza revisión humana.
- [ ] `confidence = low` fuerza revisión humana.
- [ ] P1 mantiene revisión humana.
- [ ] Un fallo de Dify conserva la prioridad determinista.
- [ ] Los retries de Dify son limitados.
- [ ] API keys y secretos no se copian a payloads ni respuestas.
- [ ] `alert_id`, ejecución n8n, `dify_run_id` y `evidence_id` pueden correlacionarse cuando estén disponibles.

## Casos

| Caso | Control principal | Resultado esperado | Observado | ¿Correcto? |
|---|---|---|---|---|
| SEC-01 | autenticación | rechazo antes del flujo normal |  |  |
| SEC-02 | baseline | P1 + IA normal |  |  |
| SEC-03 | prompt injection directa | no obedecer texto embebido |  |  |
| SEC-04 | autoridad | P1 + human review se conservan |  |  |
| SEC-05 | límite de entrada | no llamar a Dify |  |  |
| SEC-06 | grounding / revisión | P2 + human review por contexto insuficiente |  |  |
| SEC-07 | allowlist / gating | manual_review, sin Dify |  |  |
| SEC-08 | resiliencia | P1 se conserva; IA unavailable |  |  |
| SEC-09 | prompt injection RAG | documento no controla al LLM |  |  |

## Revisión de secretos

Comprueba manualmente que no aparecen valores sensibles en:

- [ ] nombres de nodos;
- [ ] Edit Fields;
- [ ] payload enviado a Dify;
- [ ] `alert_context`;
- [ ] respuesta al Webhook;
- [ ] archivos del repositorio;
- [ ] capturas entregadas.

## Preguntas de cierre

1. ¿Qué controles actúan antes de consumir IA?
2. ¿Qué diferencia hay entre autenticación y autorización?
3. ¿Por qué un path único no sustituye a Header Auth?
4. ¿Por qué un prompt endurecido no es suficiente por sí solo?
5. ¿Qué debe ocurrir si la documentación recuperada intenta dar órdenes al modelo?
6. ¿Por qué `confidence=low` no debe convertirse automáticamente en una acción?
7. ¿Qué capacidad debería conservar el sistema si Dify está caído?
