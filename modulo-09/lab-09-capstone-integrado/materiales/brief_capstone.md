# Briefing — Telvora Incident Triage & Response Advisor v1.0

## Contexto

Telvora quiere pilotar un asistente técnico para ayudar al equipo de operaciones a clasificar y analizar alertas de infraestructura. El sistema debe combinar reglas deterministas, datos autoritativos del catálogo de activos, documentación interna, estado operativo vivo y análisis con IA, manteniendo supervisión humana y trazabilidad.

La organización no autoriza remediación automática en esta versión.

## Problema

Hoy el operador debe:

1. validar la alerta;
2. identificar el activo;
3. consultar entorno, criticidad y owner;
4. determinar prioridad;
5. revisar runbooks;
6. consultar cambios, alertas, incidentes y health recientes;
7. decidir qué comprobar;
8. escalar cuando corresponda;
9. registrar la decisión.

Se busca reducir trabajo repetitivo sin delegar decisiones sensibles al LLM.

## Objetivo del MVP

Construir una automatización que:

```text
reciba una alerta autenticada
→ valide el contrato
→ enriquezca el activo mediante REST
→ calcule prioridad
→ recupere documentación mediante RAG
→ permita a un Agent consultar estado vivo mediante MCP
→ sintetice ambas fuentes con un LLM y Structured Output
→ aplique una policy determinista de revisión humana
→ determine un siguiente paso simulado
→ registre evidencia cuando sea posible
→ responda con un contrato estable
```

## Requisitos funcionales

RF-01. El sistema debe recibir alertas mediante Webhook autenticado.

RF-02. Debe validar campos obligatorios y límites antes de consumir dependencias externas innecesarias.

RF-03. Debe consultar el catálogo de activos mediante la Lab API REST.

RF-04. Debe calcular `priority` mediante las reglas P1–P4 del curso.

RF-05. No debe calcular prioridad con el LLM.

RF-06. Para entradas válidas debe invocar un Workflow Dify con RAG, Agent MCP y Structured Output.

RF-07. Debe utilizar la Knowledge Base del alumno e incorporar el nuevo runbook de portal de clientes.

RF-08. El Agent debe poder consultar Telvora Ops Hub mediante las cuatro tools read-only autorizadas:

```text
get_recent_changes
get_recent_alerts
get_open_incidents
get_service_health
```

RF-09. Debe diferenciar hechos validados, hechos operativos vivos, orientación documentada, hipótesis, checks y datos ausentes.

RF-10. Debe aplicar una policy de revisión humana fuera del LLM.

RF-11. Debe calcular un `next_step` simulado mediante reglas deterministas.

RF-12. Debe registrar evidencia con el contrato soportado por la Evidence API cuando ésta esté disponible.

RF-13. Debe devolver una respuesta JSON estable al emisor.

## Requisitos de seguridad

RS-01. El Webhook debe requerir autenticación.

RS-02. Los secretos deben almacenarse mediante Credentials o integraciones gestionadas.

RS-03. Solo los campos necesarios deben enviarse a Dify.

RS-04. El Agent MCP debe recibir únicamente el contexto necesario para seleccionar tools, preferiblemente el `asset_id` validado y una instrucción estable.

RS-05. Texto de alertas, documentos RAG y strings devueltos por tools deben tratarse como datos no confiables desde el punto de vista de instrucciones.

RS-06. Un intento de prompt injection no debe cambiar `priority`, `owner_team`, `next_step` ni la política de revisión humana.

RS-07. El Agent no debe disponer de tools destructivas o de remediación.

RS-08. No se deben exponer prompts de sistema, tokens, API keys, `MCP_TOKEN` ni credenciales.

RS-09. El sistema no debe afirmar haber realizado acciones que no ejecuta.

## Requisitos de resiliencia

RR-01. Una entrada inválida no debe consumir Dify ni MCP.

RR-02. Si Dify no está disponible, debe conservarse la prioridad determinista.

RR-03. El fallo de Dify debe producir estado IA explícito y revisión humana.

RR-04. Si MCP no está disponible pero Dify puede continuar, debe conservarse la decisión determinista, no inventarse `operational_facts` y debe quedar `operational_context_status = unavailable`.

RR-05. `no_relevant_live_context` y `unavailable` deben distinguirse.

RR-06. El fallo de Evidence API no debe falsificar un `evidence_id`.

RR-07. Los retries deben ser limitados.

## Requisitos de trazabilidad

RT-01. Debe conservarse `alert_id` durante todo el flujo.

RT-02. Cuando estén disponibles, deben poder correlacionarse:

```text
alert_id
n8n execution
dify_run_id
Agent rounds / tool calls
evidence_id
```

RT-03. Debe poder reconstruirse la regla que produjo `priority`.

RT-04. Debe ser posible distinguir:

```text
business status
ai_status
operational_context_status / mcp_status
evidence_status
```

## Política de siguiente paso simulado

El MVP no ejecutará acciones reales.

n8n debe producir uno de:

```text
MANUAL_REVIEW
ESCALATE_HUMAN
NOTIFY_OWNER
```

Regla mínima:

```text
si status = manual_review
→ MANUAL_REVIEW

si final_requires_human_review = true
→ ESCALATE_HUMAN

si no
→ NOTIFY_OWNER
```

Dify no decide este campo.

## Restricciones

- No remediación automática.
- No SSH ni comandos sobre servidores.
- No cambios de firewall.
- No rollback automático.
- No bloqueo de cuentas.
- No tools MCP destructivas.
- No datos productivos.
- No secretos en documentos o payloads.

## Entregable esperado

Un MVP funcional y defendible técnicamente, acompañado de:

```text
arquitectura
contratos
decisiones
pruebas
riesgos
trazabilidad
resultado de validación
siguiente paso recomendado
```
