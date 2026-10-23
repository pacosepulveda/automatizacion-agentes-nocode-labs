# Laboratorio 8 — Preparación de piloto y operación

## Contexto

En M07 endureciste la automatización híbrida y validaste controles frente a autenticación ausente, inputs excesivos, prompt injection, documentación manipulada y caída de Dify.

Ahora debemos responder una pregunta diferente:

> **¿Tenemos suficiente evidencia para permitir un piloto controlado y sabemos operar y revertir el sistema si algo falla?**

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

El resultado final del laboratorio será exactamente una de estas decisiones:

```text
READY
READY_WITH_CONDITIONS
NOT_READY
```

> El entorno sigue siendo formativo. No se realizará ninguna remediación real ni se utilizarán datos productivos.

---

# Objetivos de aprendizaje

Al finalizar deberías ser capaz de:

- definir ownership y decision rights mínimos;
- identificar exactamente qué versión/configuración se quiere pilotar;
- construir un release manifest;
- definir gates de funcionalidad, seguridad y resiliencia;
- diferenciar blocker de condición no bloqueante;
- establecer un rollout progresivo en shadow/limited pilot;
- utilizar el historial de ejecuciones para observabilidad básica;
- registrar latencia, estado Dify, tokens, human review y evidencia;
- escribir un runbook operativo mínimo;
- simular una jornada de producción;
- distinguir fallo de IA, fallo de evidencia y fallo de negocio;
- ejecutar un rollback drill;
- redactar un postmortem;
- justificar una decisión `READY`, `READY_WITH_CONDITIONS` o `NOT_READY`.

---

# Prerrequisitos

Necesitas:

- `M07-LAB07-Hardened-AI-Triage` funcional;
- tu App Dify M07 publicada;
- tu Knowledge Base privada M05;
- `matriz_riesgos.md` y política mínima de M07;
- resultados de SEC-01…SEC-09;
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

Utiliza un Webhook específico:

```text
m08-alert-<ID_ALUMNO>
```

Ejemplo:

```text
m08-alert-alumno01
```

No destruyas M07. M07 representa la **configuración conocida como buena** que utilizaremos como referencia para rollback.

M08 puede reutilizar la App Dify endurecida de M07. No es necesario duplicar Dify si no hay cambios en su comportamiento.

---

# Parte 2 — Definir ownership y decision rights

Antes de ejecutar pruebas abre:

```text
materiales/release_manifest_template.md
```

Asigna al menos estos roles:

```text
Workflow Owner
Security Reviewer
Pilot Approver
Rollback Owner
Escalation Contact
```

En un entorno real pueden ser personas diferentes.

En el laboratorio puedes asignar roles al alumno/instructor según corresponda, pero debes mantener separadas las responsabilidades conceptuales.

## Qué decide cada rol

### Workflow Owner

Responsable de:

```text
configuración
cambios
funcionamiento
runbook
```

### Security Reviewer

Puede bloquear la release si falla un control crítico.

### Pilot Approver

Emite la decisión final de paso a piloto.

### Rollback Owner

Tiene autoridad para restaurar la última configuración conocida como buena.

### Escalation Contact

Recibe incidentes que no pueden resolverse siguiendo el runbook.

---

# Parte 3 — Completar el Release Manifest

Registra:

```text
release_id
workflow n8n
App Dify
Knowledge Base
prompt
schema
APIs
Credentials requeridas por nombre
known issues
rollback target
```

No copies valores de secretos.

Un ejemplo de release ID:

```text
M08-AL01-v1
```

## Por qué necesitamos un manifest

Si dentro de dos días una respuesta cambia, queremos poder responder:

```text
¿Qué workflow estaba activo?
¿Qué App Dify utilizaba?
¿Qué Knowledge Base?
¿Qué prompt?
¿Qué issue estaba abierto?
```

Sin una fotografía de la release, investigar regresiones es mucho más difícil.

---

# Parte 4 — Known issues

Revisa los issues conocidos del curso que puedan afectar a la release.

Ejemplo relevante:

```text
validación de campos ampliados de POST /api/v1/evidence
```

No todos los issues bloquean un piloto.

Clasifica cada uno como:

```text
blocker
non-blocking with mitigation
accepted risk
```

Para cualquier condición no bloqueante debes indicar:

```text
owner
control compensatorio
criterio de cierre
```

---

# Parte 5 — Definir el release gate

Abre:

```text
materiales/checklist_readiness.md
```

Los gates se dividen en:

```text
funcionalidad
seguridad
resiliencia
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
no existe rollback probado
un documento RAG puede controlar una decisión sensible
```

No se permite compensar un blocker crítico diciendo simplemente:

```text
"lo vigilaremos durante el piloto"
```

---

# Parte 6 — Reejecutar un subconjunto de regresión

Antes de simular producción ejecuta como mínimo:

```text
INT-01 / baseline P1 web
INT-02 / DB critical
INT-04 / batch con docs insuficientes
INT-05 / campo obligatorio ausente
INT-08 / P4 que IA no debe elevar

SEC-01 / auth
SEC-03 / prompt injection
SEC-04 / autoridad
SEC-05 / límite de input
SEC-08 / degraded mode
SEC-09 / RAG injection
```

No necesitas conservar el documento adversarial de M07 permanentemente. Para SEC-09 puedes volver a cargarlo temporalmente y retirarlo al terminar.

Marca PASS/FAIL en el checklist.

---

# Parte 7 — Definir rollout

No pasaremos directamente de:

```text
laboratorio
```

a:

```text
automatización plenamente autónoma
```

Diseña el rollout en fases.

## Fase 0 — Shadow

El workflow procesa eventos de prueba/replay o una copia controlada, pero sus recomendaciones no cambian operación real.

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
sin remediación automática
```

## Fase 2 — Expansión

Queda fuera del laboratorio y solo sería posible tras nueva revisión de métricas y riesgos.

En `decision_piloto_template.md` define qué alcance autorizarías.

---

# Parte 8 — Construir baseline de observabilidad

Utiliza:

```text
materiales/registro_metricas.csv
```

Para cada ejecución registra cuando esté disponible:

```text
scenario_id
n8n status
end-to-end time
dify status
dify elapsed time
dify tokens
priority
human review
evidence status
notes
```

## Importante

No fijaremos un SLO de latencia arbitrario en este laboratorio.

Primero queremos obtener un **baseline real del entorno**.

Sí existen gates binarios que deben pasar al 100%:

```text
autenticación
fuente autoritativa de priority
fallback
no exposición de secretos
rollback
```

---

# Parte 9 — Dónde observar

## n8n

Utiliza el historial de **Executions** para comprobar:

```text
success / failed
inicio
fin / duración
nodos ejecutados
rama seguida
error técnico
```

## Dify

En la ejecución/log de la App revisa:

```text
workflow run
inputs
retrieval
outputs
elapsed time
tokens
error
```

## Evidence API

Cuando funcione, conserva:

```text
evidence_id
```

La observabilidad debe permitir correlacionar:

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

Completa:

```text
materiales/runbook_operacion_template.md
```

Debe responder como mínimo:

```text
¿Cómo compruebo salud?
¿Qué hago si Dify cae?
¿Qué hago si veo varios auth failures?
¿Qué hago si falta documentación?
¿Qué hago si falla Evidence API?
¿Cómo hago rollback?
¿Cómo detengo el piloto?
¿Qué datos necesito para diagnosticar?
```

El runbook debe poder seguirlo otra persona sin conocer todos los detalles internos del laboratorio.

---

# Parte 11 — Jornada operativa

Ejecuta:

```text
materiales/escenarios_operacion.json
```

Tenemos seis escenarios.

---

## OPS-01 — Baseline P1 saludable

Ejecuta un caso equivalente a INT-01.

Esperado:

```text
priority = P1
ai_status = succeeded
human review = true
```

Registra:

```text
latencia total
elapsed time Dify
tokens
IDs de trazabilidad
```

Éste será nuestro baseline.

---

## OPS-02 — Documentación insuficiente

Ejecuta un batch equivalente a SEC-06.

Esperado:

```text
priority = P2
documentation_status = insufficient_context
human review = true
```

El objetivo es validar la política, no conseguir que el LLM invente un runbook.

---

## OPS-03 — Auth failures

Realiza tres peticiones sin autenticación y después una con la cabecera correcta.

Esperado:

```text
3 rechazadas
1 procesada
```

Documenta en el runbook qué harías si ese patrón apareciera repetidamente en un piloto.

No desactives autenticación para "resolver" el incidente.

---

## OPS-04 — Dify no disponible

Rompe temporalmente la URL Dify como en M07.

Ejecuta un P1.

Esperado:

```text
priority = P1
ai_status = unavailable
human review = true
```

Registra qué capacidades se pierden y cuáles permanecen.

Después restaura la URL.

Este escenario será una buena base para el postmortem.

---

## OPS-05 — Evidence API no disponible

Si tu workflow registra evidencia, cambia temporalmente la ruta de Evidence API por una inexistente o utiliza el mecanismo de simulación indicado por el instructor.

Queremos distinguir:

```text
decisión calculada
```

de:

```text
persistencia de evidencia
```

El workflow no debe afirmar que la evidencia se registró si la API falló.

Una política razonable para el piloto es:

```text
preservar la decisión en la ejecución n8n
marcar evidence_status = unavailable/failed
forzar revisión humana
escalar el problema de auditoría
```

No es obligatorio modificar la API para completar M08.

---

# Parte 12 — Rollback drill

Rollback no es una frase en un documento.

Debemos probar que sabemos volver a un estado conocido.

## Preparación

Tu referencia conocida como buena es:

```text
M07-LAB07-Hardened-AI-Triage
```

y la configuración registrada en `release_manifest_template.md`.

## Cambio controlado

En el candidato M08 introduce deliberadamente un cambio fácil de detectar y de revertir. Por ejemplo:

```text
cambiar temporalmente el límite de details de 2000 a 20 caracteres
```

No cambies credenciales ni controles de autenticación para esta prueba.

Ejecuta OPS-01.

El gate funcional debe fallar porque una alerta normal será desviada por `input_too_large`.

## Rollback

Restaura:

```text
details <= 2000
```

vuelve a publicar/activar si fuera necesario y repite OPS-01.

Esperado:

```text
smoke test PASS
```

Registra:

```text
qué falló
cómo lo detectaste
qué restauraste
tiempo aproximado de recuperación
resultado posterior
```

---

# Parte 13 — Postmortem

Utiliza:

```text
materiales/postmortem_template.md
```

Puedes usar OPS-04 o el fallo controlado del rollback drill.

El postmortem debe distinguir:

```text
causa técnica
factores contribuyentes
impacto
controles que funcionaron
controles que faltaron
acciones de mejora
```

Evita frases centradas en culpabilidad personal.

Queremos mejorar el sistema.

---

# Parte 14 — Readiness Review

Vuelve a:

```text
checklist_readiness.md
```

Completa todos los apartados con evidencia real del laboratorio.

No marques PASS porque:

```text
"debería funcionar"
```

Marca PASS cuando:

```text
lo has ejecutado y observado
```

---

# Parte 15 — Decisión final

Completa:

```text
materiales/decision_piloto_template.md
```

Elige exactamente una:

```text
READY
READY_WITH_CONDITIONS
NOT_READY
```

## READY

Todos los blockers pasan y no quedan condiciones relevantes pendientes para el alcance definido.

## READY_WITH_CONDITIONS

Todos los blockers críticos pasan, pero existen condiciones no bloqueantes que tienen:

```text
owner
mitigación
criterio de cierre
```

## NOT_READY

Falla un blocker o no existe evidencia suficiente para demostrar que pasa.

---

# Parte 16 — Ejemplo de razonamiento sobre un issue abierto

Supongamos que todavía no está resuelta la ampliación de campos de Evidence API.

Eso no implica automáticamente:

```text
NOT_READY
```

Podría ser:

```text
READY_WITH_CONDITIONS
```

si:

```text
el contrato M03 sigue funcionando
la trazabilidad IA permanece en n8n/Dify
no se pierde la decisión determinista
hay owner y criterio de cierre
```

Pero si la Evidence API no registra ninguna evidencia y el alcance del piloto exige auditoría persistente obligatoria, la misma incidencia sí podría ser blocker.

La clasificación depende del **alcance y la política**, no solo del nombre del bug.

---

# Parte 17 — Stop conditions del piloto

Define al menos:

```text
bypass de autenticación
exposición de secreto
priority alterada por IA
fallback no funciona
RAG injection controla decisiones
rollback no recupera estado conocido
```

Ante cualquiera de ellas:

```text
detener piloto
restaurar configuración conocida
analizar incidente
reevaluar readiness
```

---

# Resultado final esperado

Al terminar tendrás:

```text
Release Manifest
       |
       v
Regression + Security Gates
       |
       v
Shadow / Limited Pilot Plan
       |
       v
Metrics Baseline
       |
       v
Operational Runbook
       |
       v
Production-Day Simulation
       |
       v
Rollback Drill
       |
       v
Postmortem
       |
       v
Readiness Review
       |
       v
READY / READY_WITH_CONDITIONS / NOT_READY
```

---

# Entrega

Entrega:

1. `release_manifest_template.md` completado;
2. `checklist_readiness.md` completado;
3. `registro_metricas.csv` con los escenarios ejecutados;
4. `runbook_operacion_template.md` completado;
5. `postmortem_template.md` completado;
6. `decision_piloto_template.md` completado;
7. captura del workflow candidato M08;
8. evidencia del rollback drill;
9. justificación de la decisión final.

No incluyas secretos.

---

# Comprobación final

Deberías poder explicar:

- qué diferencia hay entre que algo funcione y que esté ready;
- quién puede aprobar, bloquear y hacer rollback;
- qué es un release manifest;
- qué diferencia hay entre blocker y condición;
- por qué primero medimos baseline antes de fijar un SLO de latencia;
- qué señales observamos en n8n y Dify;
- qué capacidad mantiene el sistema cuando Dify cae;
- por qué un rollback debe ensayarse;
- qué aporta un postmortem;
- cuándo elegirías READY, READY_WITH_CONDITIONS o NOT_READY.
