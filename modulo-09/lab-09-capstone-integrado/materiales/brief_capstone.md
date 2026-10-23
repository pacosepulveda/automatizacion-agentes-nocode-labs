# Briefing — Telvora Incident Triage & Response Advisor v1.0

## Contexto

Telvora quiere pilotar un asistente técnico para ayudar al equipo de operaciones a clasificar y analizar alertas de infraestructura. El sistema debe combinar reglas deterministas, datos del catálogo de activos, documentación interna y análisis con IA, manteniendo supervisión humana y trazabilidad.

La organización no autoriza remediación automática en esta versión.

## Problema

Hoy el operador debe:

1. validar la alerta;
2. identificar el activo;
3. consultar entorno, criticidad y owner;
4. determinar prioridad;
5. revisar runbooks y cambios recientes;
6. decidir qué comprobar;
7. escalar cuando corresponda;
8. registrar la decisión.

Se busca reducir trabajo repetitivo sin delegar decisiones sensibles al LLM.

## Objetivo del MVP

Construir una automatización que:

```text
reciba una alerta autenticada
→ valide el contrato
→ enriquezca el activo
→ calcule prioridad
→ utilice RAG + LLM para análisis técnico
→ aplique una policy determinista de revisión humana
→ determine un siguiente paso simulado
→ registre evidencia cuando sea posible
→ responda con un contrato estable
```

## Requisitos funcionales

RF-01. El sistema debe recibir alertas mediante Webhook autenticado.

RF-02. Debe validar campos obligatorios y límites antes de consumir dependencias externas innecesarias.

RF-03. Debe consultar el catálogo de activos mediante la Lab API.

RF-04. Debe calcular `priority` mediante las reglas P1–P4 del curso.

RF-05. No debe calcular prioridad con el LLM.

RF-06. Para entradas válidas debe invocar un Workflow Dify con RAG y Structured Output.

RF-07. Debe utilizar la Knowledge Base del alumno e incorporar el nuevo runbook de portal de clientes.

RF-08. Debe diferenciar hechos, orientación documentada, hipótesis, checks y datos ausentes.

RF-09. Debe aplicar una policy de revisión humana fuera del LLM.

RF-10. Debe calcular un `next_step` simulado mediante reglas deterministas.

RF-11. Debe registrar evidencia con el contrato soportado por la Evidence API cuando ésta esté disponible.

RF-12. Debe devolver una respuesta JSON estable al emisor.

## Requisitos de seguridad

RS-01. El Webhook debe requerir autenticación.

RS-02. Los secretos deben almacenarse mediante Credentials.

RS-03. Solo los campos necesarios deben enviarse a Dify.

RS-04. Texto de alertas y documentos RAG deben tratarse como datos no confiables desde el punto de vista de instrucciones.

RS-05. Un intento de prompt injection no debe cambiar `priority`, `owner_team` ni la política de revisión humana.

RS-06. No se deben exponer prompts de sistema, tokens, API keys ni credenciales.

RS-07. El sistema no debe afirmar haber realizado acciones que no ejecuta.

## Requisitos de resiliencia

RR-01. Una entrada inválida no debe consumir Dify.

RR-02. Si Dify no está disponible, debe conservarse la prioridad determinista.

RR-03. El fallo de Dify debe producir estado IA explícito y revisión humana.

RR-04. El fallo de Evidence API no debe falsificar un `evidence_id`.

RR-05. Los retries deben ser limitados.

## Requisitos de trazabilidad

RT-01. Debe conservarse `alert_id` durante todo el flujo.

RT-02. Cuando estén disponibles, deben poder correlacionarse:

```text
alert_id
n8n execution
dify_run_id
evidence_id
```

RT-03. Debe poder reconstruirse la regla que produjo `priority`.

RT-04. Debe ser posible distinguir:

```text
estado de negocio
estado IA
estado de evidencia
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
- No bloqueo de cuentas.
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
