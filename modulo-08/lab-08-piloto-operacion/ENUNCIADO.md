# Laboratorio 8 — Preparación de piloto y operación

## Contexto

En M07 endureciste la automatización híbrida y aplicaste controles sobre autenticación, validación, prompt injection, RAG, Agent MCP, mínimo privilegio, revisión humana y fallback.

Ahora debemos responder otra pregunta:

> **¿Tenemos suficiente evidencia para permitir un piloto controlado y sabemos operar, degradar y revertir el sistema si algo falla?**

M08 no añade una nueva capacidad de IA. Trabaja sobre:

```text
ownership
release gate
observabilidad
runbook
rollout
rollback
incidentes
readiness
```

Una diferencia importante respecto a módulos anteriores es que ahora tratamos **Dify**, **MCP/Telvora Ops Hub** y **Evidence API** como dependencias diferenciadas. Pueden fallar de forma independiente.

El resultado final será exactamente una de estas decisiones:

```text
READY
READY_WITH_CONDITIONS
NOT_READY
```

> El entorno sigue siendo formativo. No se realizará remediación real ni se utilizarán datos productivos.

---

# Objetivos de aprendizaje

Al finalizar deberías ser capaz de:

- definir ownership y decision rights mínimos;
- identificar exactamente qué versión/configuración se quiere pilotar;
- construir un release manifest que incluya n8n, Dify, RAG y MCP;
- definir gates de funcionalidad, seguridad, tool safety y resiliencia;
- diferenciar blocker de condición no bloqueante;
- establecer un rollout progresivo en shadow/limited pilot;
- utilizar el historial de ejecuciones para observabilidad básica;
- registrar latencia, estado Dify, estado MCP, tokens, human review y evidencia;
- escribir un runbook operativo mínimo;
- simular una jornada de producción;
- distinguir fallo de negocio, Dify, MCP y Evidence API;
- ejecutar un rollback drill;
- redactar un postmortem;
- justificar una decisión `READY`, `READY_WITH_CONDITIONS` o `NOT_READY`.

---

# Prerrequisitos

Necesitas:

- `M07-LAB07-Hardened-AI-Triage` funcional o preparado para validación;
- tu App Dify M07 publicada;
- Agent `FunctionCalling` con las cuatro tools read-only de Telvora Ops Hub;
- tu Knowledge Base privada M05;
- matriz de riesgos y política mínima de M07;
- casos SEC-01…SEC-10;
- acceso al historial de ejecuciones de n8n y Dify.

---

# Material proporcionado

```text
materiales/
├── release_manifest_template.md
├── checklist_readiness.md
├── escenarios_operacion.json
├── registro_metricas.csv
├── runbook_operacion_template.md
├── postmortem_template.md
└── decision_piloto_template.md
```

---

# Parte 1 — Crear el candidato de release

Duplica el workflow n8n endurecido de M07.

Nombre:

```text
M08-LAB08-Pilot-Candidate
```

Webhook:

```text
m08-alert-<ID_ALUMNO>
```

No destruyas M07. M07 representa la **configuración conocida como buena** para rollback.

M08 puede reutilizar la App Dify endurecida de M07 si no vas a modificar su comportamiento.

---

# Parte 2 — Definir ownership y decision rights

Completa `materiales/release_manifest_template.md`.

Asigna al menos:

```text
Workflow Owner
Security Reviewer
Pilot Approver
Rollback Owner
Escalation Contact
```

En un entorno real pueden ser personas distintas.

## Workflow Owner

Responsable de configuración, cambios, funcionamiento y runbook.

## Security Reviewer

Puede bloquear la release si falla un control crítico.

## Pilot Approver

Emite la decisión final de paso a piloto.

## Rollback Owner

Tiene autoridad para restaurar la configuración conocida como buena.

## Escalation Contact

Recibe incidentes que no pueden resolverse siguiendo el runbook.

---

# Parte 3 — Completar el Release Manifest

Registra:

```text
release_id
workflow n8n
App Dify
Agent Strategy
Telvora Ops Hub / MCP
Knowledge Base
prompt final
prompt Agent
schema
APIs
Credentials requeridas por nombre
known issues
rollback target
```

No copies valores de secretos.

Ejemplo:

```text
M08-AL01-v1
```

## Por qué MCP debe aparecer en el manifest

El Agent depende de una integración compartida y de un servicio remoto concreto.

Si una respuesta cambia, debemos poder responder:

```text
¿Qué Agent Strategy estaba activa?
¿Qué servidor MCP estaba configurado?
¿Qué tools estaban permitidas?
¿Qué prompt del Agent estaba publicado?
```

---

# Parte 4 — Known issues

Revisa los issues conocidos que puedan afectar a la release.

Clasifica cada uno como:

```text
blocker
non-blocking with mitigation
accepted risk
```

Para cualquier condición no bloqueante indica:

```text
owner
control compensatorio
criterio de cierre
```

Un preflight pendiente de MCP puede ser no bloqueante mientras el piloto no se autorice todavía, pero debe cerrarse antes de declarar `READY`.

---

# Parte 5 — Definir el release gate

Abre `materiales/checklist_readiness.md`.

Los gates se agrupan en:

```text
funcionalidad
seguridad
resiliencia
MCP / tool safety
observabilidad
operación
rollback
known issues
```

## Blockers obligatorios

El candidato será `NOT_READY` si ocurre cualquiera de estos casos:

```text
Webhook acepta tráfico sin autenticación
prompt injection altera priority
se expone un secreto
caída de Dify destruye la decisión determinista
MCP failure provoca hechos operativos inventados
el Agent dispone de una tool destructiva no autorizada
no existe rollback probado
un documento RAG controla una decisión sensible
```

---

# Parte 6 — Reejecutar regresión

Antes de simular producción ejecuta como mínimo:

```text
INT-01
INT-02
INT-04
INT-05
INT-08

SEC-01
SEC-03
SEC-04
SEC-05
SEC-08
SEC-09
SEC-10
```

Marca PASS/FAIL.

Para SEC-10 verifica los **rounds del Agent**, no solo el texto final.

---

# Parte 7 — Definir rollout

No pasaremos directamente de laboratorio a autonomía plena.

## Fase 0 — Shadow

El workflow procesa eventos de prueba/replay o una copia controlada.

Objetivos:

```text
medir
observar
comparar
corregir
```

## Fase 1 — Limited Pilot

Alcance reducido:

```text
pocos tipos de alerta
usuarios/equipo limitado
human review obligatoria
MCP read-only
sin remediación automática
```

## Fase 2 — Expansión

Fuera del laboratorio. Requiere nueva revisión de métricas y riesgos.

---

# Parte 8 — Construir baseline de observabilidad

Utiliza `materiales/registro_metricas.csv`.

Para cada ejecución registra cuando esté disponible:

```text
scenario_id
n8n status
end-to-end time
dify status
mcp status
operational_context_status
dify elapsed time
dify tokens
priority
human review
evidence status
notes
```

No fijaremos un SLO arbitrario de latencia. Primero obtendremos un baseline real.

Sí existen gates binarios que deben pasar al 100%:

```text
autenticación
fuente autoritativa de priority
fallback
MCP tool safety
no exposición de secretos
rollback
```

---

# Parte 9 — Dónde observar

## n8n

En **Executions** revisa:

```text
success / failed
duración
nodos ejecutados
rama seguida
error técnico
```

## Dify

Revisa:

```text
workflow run
Knowledge Retrieval
Agent
Agent rounds/tool calls
LLM final
outputs
elapsed time
tokens
error
```

## Telvora Ops Hub / MCP

No necesitas acceso al token MCP.

La evidencia principal es:

```text
Agent → Detalle → rounds
→ tool_call_name
→ tool_call_input
→ tool_response
```

## Evidence API

Cuando funcione, conserva `evidence_id`.

La correlación mínima es:

```text
alert_id
↕
n8n execution
↕
dify_run_id
↕
evidence_id
```

---

# Parte 10 — Escribir el runbook

Completa `materiales/runbook_operacion_template.md`.

Debe responder al menos:

```text
¿Cómo compruebo salud?
¿Qué hago si Dify cae?
¿Qué hago si MCP/Telvora Ops Hub cae?
¿Qué hago si veo auth failures?
¿Qué hago si falta documentación?
¿Qué hago si falla Evidence API?
¿Cómo hago rollback?
¿Cómo detengo el piloto?
¿Qué datos necesito para diagnosticar?
```

---

# Parte 11 — Jornada operativa

Ejecuta `materiales/escenarios_operacion.json`.

Tenemos siete escenarios.

## OPS-01 — Baseline P1 saludable

Ejecuta un caso equivalente a INT-01.

Esperado:

```text
priority = P1
ai_status = succeeded
mcp_status = available
human review = true
```

Registra latencia, tokens e IDs de trazabilidad.

## OPS-02 — Documentación insuficiente

Ejecuta un batch equivalente a SEC-06.

Esperado:

```text
priority = P2
documentation_status = insufficient_context
human review = true
```

## OPS-03 — Auth failures

Realiza tres peticiones sin autenticación y después una correcta.

Esperado:

```text
3 rechazadas
1 procesada
```

No desactives autenticación para "resolver" el incidente.

## OPS-04 — Dify no disponible

Rompe temporalmente la URL Dify como en M07.

Esperado:

```text
priority = P1
ai_status = unavailable
human review = true
```

Se pierde toda la capacidad Dify de esa ejecución.

## OPS-05 — MCP no disponible

Utiliza el mecanismo de fallo indicado por el instructor.

Esperado conceptualmente:

```text
priority preservada
Dify workflow puede seguir completándose
RAG puede seguir disponible
operational_facts = []
operational_context_status = unavailable
mcp_status = unavailable
human review = true
```

Esta prueba demuestra que:

```text
Dify unavailable
!=
MCP unavailable
```

No inventes estado vivo para compensar la ausencia de MCP.

## OPS-06 — Evidence API no disponible

Cambia temporalmente la ruta de evidencia por una inexistente o utiliza el método indicado por el instructor.

Esperado:

```text
decisión preservada
evidence_status = failed/unavailable
no evidence_id inventado
human review / escalado según policy
```

## OPS-07 — Rollback drill

Introduce un cambio controlado fácil de detectar, por ejemplo:

```text
límite details: 2000 → 20
```

Ejecuta OPS-01 y verifica que el gate falla.

Restaura:

```text
details <= 2000
```

y repite el smoke test.

Esperado:

```text
PASS después del rollback
```

---

# Parte 12 — Postmortem

Utiliza `materiales/postmortem_template.md`.

Puedes basarlo en OPS-04, OPS-05 o el rollback drill.

Distingue:

```text
causa técnica
factores contribuyentes
impacto
controles que funcionaron
controles que faltaron
acciones de mejora
```

---

# Parte 13 — Readiness Review

Completa `checklist_readiness.md` con evidencia real.

No marques PASS porque:

```text
"debería funcionar"
```

Marca PASS cuando:

```text
lo has ejecutado y observado
```

---

# Parte 14 — Decisión final

Completa `decision_piloto_template.md`.

Selecciona exactamente una:

```text
READY
READY_WITH_CONDITIONS
NOT_READY
```

## READY

Todos los blockers pasan y no quedan condiciones relevantes para el alcance definido.

## READY_WITH_CONDITIONS

Todos los blockers críticos pasan, pero existen condiciones no bloqueantes con:

```text
owner
mitigación
criterio de cierre
```

## NOT_READY

Falla un blocker o no existe evidencia suficiente.

---

# Parte 15 — Stop conditions

Define condiciones que detendrían inmediatamente el piloto.

Incluye como mínimo:

```text
bypass de autenticación
secreto expuesto
priority alterada por IA
tool destructiva no aprobada
MCP unavailable con operational facts inventados
fallback Dify/MCP no funcional
rollback no ejecutable
```

---

# Entrega

Entrega:

1. release manifest;
2. checklist readiness;
3. registro de métricas con OPS-01…OPS-07;
4. runbook operativo;
5. evidencia de rollback drill;
6. postmortem;
7. decisión de piloto;
8. capturas/IDs suficientes para demostrar Dify y MCP observables;
9. listado de known issues/condiciones pendientes.

---

# Comprobación final

Debes poder explicar:

- qué componentes forman la release;
- qué diferencia existe entre Dify y MCP como dependencias;
- cómo sabes si una tool MCP se ejecutó;
- qué se conserva si Dify falla;
- qué se conserva si MCP falla;
- qué se conserva si Evidence API falla;
- qué métricas registrarías;
- qué condiciones detienen el piloto;
- cómo haces rollback;
- por qué un sistema puede estar `READY_WITH_CONDITIONS` sin ser `READY`.

---

# Resultado esperado

M08 no busca "más IA".

Busca demostrar que sabes pasar de:

```text
prototipo funcional
```

a:

```text
servicio observable,
operable,
degradable,
reversible
y gobernado.
```
