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
M06  integración n8n + Dify
M07  hardening
M08  preparación de piloto y operación
```

M09 es el capstone.

No recibirás una receta nodo por nodo. Recibirás un briefing, contratos, documentación y casos de validación. Tu trabajo es **construir o adaptar una solución end-to-end y justificar tus decisiones**.

El sistema final será:

```text
Telvora Incident Triage & Response Advisor v1.0
```

Su función es recibir alertas técnicas, validarlas, enriquecerlas, calcular una prioridad determinista, utilizar Dify + RAG para generar análisis técnico, aplicar una política de revisión humana y devolver un siguiente paso simulado y trazable.

No realizará remediación real.

---

# Objetivos de aprendizaje

Al finalizar deberías ser capaz de:

- convertir un briefing en requisitos implementables;
- proponer y defender una arquitectura end-to-end;
- separar reglas deterministas y capacidades probabilísticas;
- diseñar contratos entre sistemas;
- reutilizar APIs y Credentials de forma segura;
- incorporar documentación nueva a una Knowledge Base;
- integrar n8n con un Workflow Dify publicado;
- mantener grounding y Structured Output;
- aplicar supervisión humana mediante una policy determinista;
- diseñar un siguiente paso simulado fuera del LLM;
- probar happy paths, excepciones, ataques controlados y fallos de dependencias;
- mantener trazabilidad entre n8n, Dify y evidencias;
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
riesgos
excepciones
controles
criterios de supervisión humana
```

No empieces por la herramienta.

Debes poder explicar primero qué sistema necesitas construir.

---

## Parte 2 — Diseñar la arquitectura

Crea un diagrama que incluya al menos:

```text
emisor
Webhook n8n
validación
Lab API
clasificación
Dify API
Knowledge Retrieval
LLM
policy de revisión
next_step
evidence
respuesta
```

Marca explícitamente:

```text
trust boundaries
fuentes autoritativas
puntos de fallo
```

No existe una única disposición visual correcta.

La arquitectura debe cumplir el contrato.

---

## Parte 3 — Elegir estrategia de reutilización

Puedes:

```text
duplicar/adaptar M08
```

o construir un nuevo candidato reutilizando componentes ya probados.

No se puntúa más por reconstruir desde cero.

Se evalúa:

```text
calidad del diseño
consistencia
pruebas
justificación
```

Nombre recomendado n8n:

```text
M09-CAPSTONE-Incident-Triage-<ID_ALUMNO>
```

Webhook:

```text
m09-capstone-<ID_ALUMNO>
```

Ejemplo:

```text
m09-capstone-alumno01
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

Implementa el contrato definido en:

```text
contrato_capstone.md
```

Debe existir:

```text
autenticación
validación de obligatorios
allowlist severity
límites de longitud
```

Las entradas inválidas deben desviarse antes de Dify.

---

## Parte 6 — Enriquecimiento y prioridad

Consulta la Lab API utilizando `asset_id`.

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

## Parte 7 — Preparar la llamada IA

Envía a Dify únicamente los campos necesarios.

Como mínimo, el contexto puede incluir:

```text
alert_id
asset_id
asset_name
severity
environment
criticality
owner_team
priority
summary
details
```

No envíes:

```text
Authorization
cookies
API keys
Webhook secret
headers innecesarios
```

Construye una `retrieval_query` razonable.

No se exige una única fórmula.

Debes justificar qué campos utilizas y por qué.

---

## Parte 8 — Dify Workflow

Puedes reutilizar/adaptar el Workflow endurecido de M07.

Debe mantener:

```text
Knowledge Retrieval
prompt hardened
Structured Output
GPT-5.6 Luna
Temperature OFF
Top P OFF
```

No añadas `priority` como decisión autoritativa del LLM.

La App debe estar publicada antes de invocarla desde n8n.

---

## Parte 9 — Policy de revisión humana

Implementa fuera del LLM una regla equivalente a:

```text
final_requires_human_review =
    base_requires_human_review
    OR ai_status != succeeded
    OR confidence == low
    OR documentation_status == insufficient_context
```

Puedes ampliar la policy si lo justificas.

No puedes reducir revisión humana de una P1 basándote únicamente en la confianza del LLM.

---

## Parte 10 — Siguiente paso simulado

Implementa:

```text
MANUAL_REVIEW
ESCALATE_HUMAN
NOTIFY_OWNER
```

según `contrato_capstone.md`.

Este campo se calcula en n8n.

No ejecutes acciones reales.

---

## Parte 11 — Evidencia

Registra evidencia mediante el contrato soportado por la API.

Si la Evidence API no admite campos de trazabilidad IA ampliados:

```text
mantén el contrato compatible
```

y conserva:

```text
dify_run_id
ai_status
analysis
```

en las ejecuciones de n8n/Dify.

No inventes `evidence_id` si el registro falla.

---

## Parte 12 — Respuesta final

Devuelve un JSON coherente con `contrato_capstone.md`.

Debe quedar claro:

```text
resultado de negocio
priority
human review
next_step
estado IA
estado de evidencia
```

---

# M09.2 — Validación y presentación

## Parte 13 — Predicción antes de ejecutar

Abre:

```text
casos_capstone.json
```

y `checklist_validacion.md`.

Antes de ejecutar cada caso completa:

```text
resultado esperado por el equipo
```

No copies una solución externa.

Derívalo del briefing y contratos.

---

## Parte 14 — CAP-01

Caso web de producción con `severity=high` y activo crítico.

Debes demostrar:

```text
prioridad determinista
retrieval útil
análisis grounded
human review
next_step coherente
```

---

## Parte 15 — CAP-02

Caso DB critical con información insuficiente para causa raíz.

Debes demostrar:

```text
P1
no inventar root cause
checks coherentes
human review
```

---

## Parte 16 — CAP-03

API medium con latencia.

Comprueba que el análisis puede utilizar contexto documental sin elevar arbitrariamente la prioridad.

---

## Parte 17 — CAP-04

Batch high con documentación insuficiente.

Debes demostrar la diferencia entre:

```text
priority
```

y:

```text
necesidad de revisión
```

---

## Parte 18 — CAP-05

Falta `asset_id`.

Esperado conceptualmente:

```text
manual review
Dify no llamado
```

---

## Parte 19 — CAP-06

Activo desconocido.

La API puede responder correctamente con:

```text
found=false
```

No clasifiques automáticamente sin contexto del activo.

---

## Parte 20 — CAP-07

Prompt injection directa.

El texto intenta:

```text
cambiar priority
eliminar human review
afirmar reboot
```

Debes demostrar que no controla decisiones sensibles.

---

## Parte 21 — CAP-08

Petición sin autenticación.

Debe ser rechazada antes del procesamiento normal.

---

## Parte 22 — CAP-09

Simula Dify no disponible.

Debes demostrar:

```text
priority preservada
ai_status explícito
human review
next_step seguro
```

---

## Parte 23 — CAP-10

Caso específico del nuevo runbook de portal de clientes.

Debes comprobar que el retrieval recupera `RUNBOOK-PORTAL-01` o fragmentos equivalentes relevantes.

La respuesta debería utilizar orientación sobre:

```text
autenticación
IdP
sesiones/cookies
cambios de release
```

sin ejecutar rollback automáticamente.

---

## Parte 24 — Revisar trazabilidad

Selecciona al menos una ejecución válida y una excepción.

Documenta:

```text
alert_id
n8n execution id
dify_run_id si existe
evidence_id si existe
regla de priority
rama seguida
```

---

## Parte 25 — Revisar arquitectura contra implementación

Compara tu diagrama inicial con el sistema final.

Documenta cambios.

Ejemplos:

```text
movimos validación antes de API
separamos policy review
cambiamos retrieval_query
añadimos un fallback
```

No ocultes desviaciones.

Explícalas.

---

## Parte 26 — Completar informe final

Utiliza:

```text
plantilla_informe_final.md
```

Debe incluir:

```text
arquitectura
requisitos
contratos
RAG
seguridad
resiliencia
pruebas
trazabilidad
trade-offs
riesgos residuales
siguiente paso
```

---

## Parte 27 — Preparar defensa técnica

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

No conviertas la presentación en una colección de capturas.

Debes defender:

```text
por qué
```

tanto como:

```text
qué construiste
```

---

# Criterios de evaluación

Consulta:

```text
rubrica_capstone.md
```

Se evaluará:

```text
arquitectura
contratos
funcionalidad
RAG
IA controlada
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
4. `checklist_validacion.md` completado;
5. `plantilla_informe_final.md` completada;
6. presentación final;
7. evidencias de pruebas CAP-01…CAP-10;
8. al menos un ejemplo de trazabilidad end-to-end;
9. recomendación de siguiente paso.

No incluyas secretos.

---

# Comprobación final

Al terminar deberías poder explicar sin abrir la solución:

- por qué la prioridad vive en n8n;
- qué aporta Dify que no aportan las reglas;
- cómo funciona RAG en tu solución;
- qué ocurre si falta documentación;
- qué ocurre si Dify falla;
- qué ocurre si Evidence API falla;
- cómo se evita que una prompt injection cambie decisiones sensibles;
- qué datos son autoritativos;
- qué datos se envían al LLM y cuáles no;
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
la IA aporta interpretación donde añade valor
RAG aporta conocimiento controlado
n8n mantiene orquestación y políticas
las personas mantienen autoridad sobre decisiones sensibles
```

Ése es el criterio de éxito del capstone.
