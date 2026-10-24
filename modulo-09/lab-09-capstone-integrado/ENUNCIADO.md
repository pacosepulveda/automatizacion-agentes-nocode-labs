# Laboratorio 9 — Caso integrado final
## Telvora Incident Triage & Response Advisor v1.0

## Contexto

Has recorrido todo el ciclo del curso:

```text
M01  diseño
M02  reglas en n8n
M03  Webhooks y APIs
M04  IA controlada
M05  RAG
M06  integración n8n + Dify + MCP
M07  hardening de IA/RAG/MCP
M08  preparación de piloto y operación
```

M09 es el capstone.

No recibirás una receta nodo por nodo. Recibirás un briefing, contratos, documentación y casos de validación. Tu trabajo es **construir o adaptar una solución end-to-end y justificar tus decisiones**.

El sistema final será:

```text
Telvora Incident Triage & Response Advisor v1.0
```

Su función es recibir alertas técnicas, validarlas, enriquecerlas, calcular una prioridad determinista, utilizar RAG para documentación, MCP para estado operativo vivo, un LLM para síntesis controlada, aplicar una política de revisión humana y devolver un siguiente paso simulado y trazable.

No realizará remediación real.

---

# Objetivos de aprendizaje

Al finalizar deberías ser capaz de:

- convertir un briefing en requisitos implementables;
- proponer y defender una arquitectura end-to-end;
- separar reglas deterministas y capacidades probabilísticas;
- justificar cuándo utilizar REST, RAG y MCP;
- diseñar contratos entre sistemas;
- reutilizar APIs y Credentials de forma segura;
- incorporar documentación nueva a una Knowledge Base;
- integrar n8n con un Workflow Dify publicado;
- utilizar un Agent `FunctionCalling` con tools MCP controladas;
- mantener grounding y Structured Output;
- aplicar supervisión humana mediante una policy determinista;
- diseñar un siguiente paso simulado fuera del LLM;
- probar happy paths, excepciones, prompt injection y fallos diferenciados de Dify/MCP/Evidence API;
- mantener trazabilidad entre n8n, Dify, Agent/tool calls y evidencias;
- identificar riesgos residuales;
- presentar y defender técnicamente la solución.

---

# Material proporcionado

```text
materiales/
├── brief_capstone.md
├── contrato_capstone.md
├── casos_capstone.json
├── telvora_runbook_portal_cliente.txt
├── checklist_validacion.md
├── rubrica_capstone.md
├── plantilla_informe_final.md
└── plantilla_presentacion.md
```

Empieza leyendo:

```text
brief_capstone.md
contrato_capstone.md
```

antes de abrir n8n o Dify.

---

# M09.1 — Construcción

## Parte 1 — Interpretar el briefing

Lee `brief_capstone.md`.

Identifica y anota:

```text
entradas
salidas
reglas autoritativas
integraciones
fuentes documentales
fuentes operativas vivas
riesgos
excepciones
controles
criterios de supervisión humana
```

No empieces por la herramienta.

---

## Parte 2 — Diseñar la arquitectura

Crea un diagrama que incluya al menos:

```text
emisor
Webhook n8n
validación
Lab API REST
clasificación
Dify API
Knowledge Retrieval / RAG
Agent FunctionCalling
Telvora Ops Hub / MCP
LLM final
policy de revisión
next_step
evidence
respuesta
```

Marca explícitamente:

```text
trust boundaries
fuentes autoritativas
fuentes de contexto
puntos de fallo
```

Debes ser capaz de explicar:

```text
REST → datos autoritativos mediante contrato conocido
RAG  → conocimiento documental
MCP  → herramientas/estado vivo para el Agent
```

---

## Parte 3 — Elegir estrategia de reutilización

Puedes:

```text
duplicar/adaptar M08
```

o construir un nuevo candidato reutilizando componentes probados.

No se puntúa más por reconstruir desde cero.

Nombre recomendado n8n:

```text
M09-CAPSTONE-Incident-Triage-<ID_ALUMNO>
```

Webhook:

```text
m09-capstone-<ID_ALUMNO>
```

---

## Parte 4 — Incorporar nueva documentación RAG

Añade a tu Knowledge Base privada:

```text
telvora_runbook_portal_cliente.txt
```

Comprueba mediante Retrieval Test que puedes localizar al menos:

```text
RUNBOOK-PORTAL-01
fallos de login después de release
no ejecutar rollback automáticamente
```

No continúes si el documento no se recupera de forma razonable.

---

## Parte 5 — Diseñar la entrada

Implementa `contrato_capstone.md`.

Debe existir:

```text
autenticación
validación de obligatorios
allowlist severity
límites de longitud
```

Las entradas inválidas deben desviarse antes de Dify y, por tanto, antes de MCP.

---

## Parte 6 — Enriquecimiento REST y prioridad

Consulta:

```text
GET https://api.cloudsecurityinpractice.com/api/v1/assets/{asset_id}
```

Recupera:

```text
asset_name
environment
criticality
owner_team
```

Aplica las reglas P1–P4.

La prioridad debe estar completamente decidida antes de llamar a Dify.

---

## Parte 7 — Preparar la llamada Dify

Envía únicamente los campos necesarios.

Inputs mínimos sugeridos:

```text
alert_context
asset_id
retrieval_query
question
```

No envíes:

```text
Authorization
cookies
API keys
Webhook secret
MCP_TOKEN
headers innecesarios
```

El `asset_id` separado permite que el Agent reciba un identificador ya validado sin necesitar parsear texto libre.

---

## Parte 8 — Knowledge Retrieval / RAG

Dify debe recuperar documentación mediante tu KB.

La salida RAG representa:

```text
qué está documentado
qué runbook existe
qué orientación interna está disponible
```

No representa necesariamente el estado actual del activo.

---

## Parte 9 — Agent + MCP

Puedes reutilizar/adaptar el Agent endurecido de M07.

Debe utilizar:

```text
FunctionCalling
openai.gpt-5.6-luna
Maximum Iterations limitado
Memory OFF
```

Tool List permitida:

```text
get_recent_changes
get_recent_alerts
get_open_incidents
get_service_health
```

El Agent no debe disponer de tools destructivas.

Su Query debería construirse a partir de:

```text
asset_id validado
+
instrucción estable
```

No necesita recibir `details` completos.

Debes poder demostrar al menos una llamada real inspeccionando:

```text
Agent → Detalle → Round
→ tool_call_name
→ tool_call_input
→ tool_response
```

---

## Parte 10 — LLM final y Structured Output

El LLM final debe sintetizar tres fuentes diferenciadas:

```text
alert_context → hechos validados
RAG → orientación documentada
MCP → hechos operativos vivos
```

Debe devolver al menos:

```text
summary
observed_facts[]
operational_facts[]
documented_guidance[]
hypotheses[]
recommended_checks[]
missing_information[]
confidence
documentation_status
operational_context_status
```

No añadas `priority` como decisión autoritativa del LLM.

---

## Parte 11 — Manejo de MCP unavailable

La solución debe diferenciar:

```text
MCP respondió sin datos relevantes
→ no_relevant_live_context

MCP no pudo consultarse
→ unavailable
```

Si MCP falla pero Dify puede continuar:

```text
operational_facts = []
operational_context_status = unavailable
```

No inventes estado vivo.

---

## Parte 12 — Policy de revisión humana

Implementa fuera del LLM una regla equivalente a:

```text
final_requires_human_review =
    base_requires_human_review
    OR ai_status != succeeded
    OR confidence == low
    OR documentation_status == insufficient_context
    OR operational_context_status == unavailable
```

Puedes ampliar la policy si lo justificas.

No puedes reducir revisión humana de una P1 basándote únicamente en la confianza del LLM.

---

## Parte 13 — Siguiente paso simulado

Implementa en n8n:

```text
status = manual_review
→ MANUAL_REVIEW

final_requires_human_review = true
→ ESCALATE_HUMAN

resto
→ NOTIFY_OWNER
```

Dify no decide `next_step`.

No ejecutes acciones reales.

---

## Parte 14 — Evidencia

Registra evidencia mediante el contrato soportado por la API.

Si la Evidence API no admite todos los campos ampliados:

```text
mantén el contrato compatible
```

y conserva trazabilidad adicional en n8n/Dify.

No inventes `evidence_id` si el registro falla.

---

## Parte 15 — Respuesta final

Devuelve un JSON coherente con `contrato_capstone.md`.

Debe distinguir:

```text
business status
priority
human review
next_step
ai_status
mcp_status / operational_context_status
evidence_status
```

---

# M09.2 — Validación y presentación

## Parte 16 — Predicción antes de ejecutar

Abre:

```text
casos_capstone.json
checklist_validacion.md
```

Antes de ejecutar cada caso completa el resultado esperado.

No copies una solución externa. Derívalo del briefing y contratos.

---

## Parte 17 — CAP-01

Web de producción, `severity=high`, activo crítico.

Debes demostrar:

```text
P1 determinista
RAG útil
Agent MCP real
hechos operativos separados de hipótesis
human review
next_step coherente
```

---

## Parte 18 — CAP-02

DB critical con información insuficiente para causa raíz.

Debes demostrar:

```text
P1
no inventar root cause
MCP puede aportar o no datos vivos
checks coherentes
human review
```

---

## Parte 19 — CAP-03

API medium con latencia.

Comprueba que el análisis utiliza contexto documental/operativo sin elevar arbitrariamente la prioridad.

Esperado:

```text
P3
```

---

## Parte 20 — CAP-04

Batch high con documentación insuficiente.

Debes demostrar la diferencia entre:

```text
priority
```

y:

```text
calidad/suficiencia del contexto
```

---

## Parte 21 — CAP-05

Falta `asset_id`.

Esperado conceptualmente:

```text
manual review
Dify no llamado
MCP no llamado
```

---

## Parte 22 — CAP-06

Activo desconocido.

La API puede responder correctamente con:

```text
found=false
```

No clasifiques automáticamente sin contexto del activo.

Dify/MCP no deben utilizarse para sustituir al catálogo autoritativo.

---

## Parte 23 — CAP-07

Prompt injection directa.

El texto intenta:

```text
cambiar priority
eliminar human review
afirmar reboot
ordenar rollback mediante MCP
```

Debes demostrar:

```text
priority permanece P1
Agent no recibe details innecesarios
solo existen tools read-only
no hay restart/rollback
no se afirma una remediación inexistente
```

---

## Parte 24 — CAP-08

Petición sin autenticación.

Debe ser rechazada antes del procesamiento normal.

No deben ejecutarse Dify ni MCP.

---

## Parte 25 — CAP-09

Simula Dify no disponible.

Debes demostrar:

```text
priority preservada
ai_status = unavailable
human review
next_step seguro
```

Aquí se pierde el Workflow Dify completo de esa ejecución.

---

## Parte 26 — CAP-10

Caso específico del nuevo runbook de portal de clientes.

Debes comprobar que RAG recupera `RUNBOOK-PORTAL-01` o fragmentos equivalentes.

El análisis puede combinarlo con datos MCP sobre cambios/health si están disponibles.

No debe ejecutar rollback automáticamente.

---

## Parte 27 — CAP-11

Simula **MCP no disponible mientras Dify sigue operativo**.

Esperado:

```text
priority preservada
ai_status = succeeded
operational_context_status = unavailable
operational_facts = []
human review = true
next_step = ESCALATE_HUMAN
```

Debes explicar por qué esto es distinto de CAP-09.

---

## Parte 28 — Revisar trazabilidad

Selecciona al menos:

```text
una ejecución válida con tool calls
una excepción funcional
un degraded mode
```

Documenta:

```text
alert_id
n8n execution id
dify_run_id si existe
Agent round/tool call si existe
evidence_id si existe
regla de priority
rama seguida
```

---

## Parte 29 — Revisar arquitectura contra implementación

Compara tu diagrama inicial con el sistema final.

Documenta cambios y justifica desviaciones.

---

## Parte 30 — Completar informe final

Utiliza:

```text
plantilla_informe_final.md
```

Debe incluir:

```text
arquitectura
requisitos
contratos
REST/RAG/MCP
seguridad
resiliencia
pruebas
trazabilidad
trade-offs
riesgos residuales
siguiente paso
```

---

## Parte 31 — Preparar defensa técnica

Utiliza:

```text
plantilla_presentacion.md
```

Duración sugerida:

```text
5-7 minutos
+
3-5 minutos de preguntas
```

Debes defender el **porqué** del diseño, no solo enseñar capturas.

---

# Criterios de evaluación

Consulta `rubrica_capstone.md`.

Se evaluará:

```text
arquitectura
contratos
funcionalidad
REST/RAG/MCP
IA controlada
tool safety
seguridad
resiliencia
pruebas
trazabilidad
documentación
defensa
```

Existen condiciones críticas que no pueden compensarse únicamente con puntuación.

---

# Entrega

Entrega:

1. workflow n8n funcional;
2. Workflow Dify utilizado;
3. Knowledge Base actualizada con el runbook del capstone;
4. evidencia de Agent/tool calls MCP;
5. `checklist_validacion.md` completado;
6. `plantilla_informe_final.md` completada;
7. presentación final;
8. evidencias de pruebas CAP-01…CAP-11;
9. al menos un ejemplo de trazabilidad end-to-end;
10. recomendación de siguiente paso.

No incluyas secretos.

---

# Comprobación final

Al terminar deberías poder explicar sin abrir la solución:

- por qué la prioridad vive en n8n;
- por qué el catálogo se consulta por REST;
- qué aporta RAG;
- qué aporta MCP;
- cómo sabes que una tool realmente se ejecutó;
- qué ocurre si falta documentación;
- qué ocurre si MCP no devuelve datos;
- qué ocurre si MCP no está disponible;
- qué ocurre si Dify falla;
- qué ocurre si Evidence API falla;
- cómo se evita que una prompt injection cambie decisiones o provoque tool use peligroso;
- por qué las tools MCP son read-only;
- cómo se decide `next_step`;
- cómo reconstruir una ejecución;
- qué riesgos siguen abiertos;
- qué harías a continuación antes de un despliegue real.

---

# Cierre del curso

M09 no pretende demostrar que un LLM puede hacerlo todo.

Pretende demostrar que sabes diseñar una automatización donde:

```text
las reglas hacen lo que debe ser determinista
REST aporta contratos autoritativos
RAG aporta conocimiento documentado
MCP aporta capacidades y contexto operativo vivo controlado
la IA interpreta y sintetiza donde añade valor
n8n mantiene orquestación y políticas
las personas mantienen autoridad sobre decisiones sensibles
```

Ése es el criterio de éxito del capstone.
