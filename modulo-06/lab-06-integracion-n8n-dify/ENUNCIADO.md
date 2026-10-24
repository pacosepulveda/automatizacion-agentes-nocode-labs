# Laboratorio 6 — Integración n8n + Dify + RAG + MCP

## Contexto

Hasta ahora hemos construido piezas que resolvían problemas distintos:

- **n8n** recibe alertas, valida contratos, consulta APIs, enriquece datos, aplica reglas deterministas y registra evidencias;
- **Dify + RAG** interpreta contexto técnico y recupera documentación operativa;
- en este módulo añadiremos **MCP (Model Context Protocol)** para que un agente pueda consultar **estado operativo vivo** mediante herramientas controladas.

El objetivo no es convertir el LLM en el dueño del proceso. La arquitectura mantiene una separación explícita de responsabilidades:

```text
n8n
→ entrada
→ validación
→ APIs deterministas
→ reglas
→ prioridad
→ credenciales
→ trazabilidad

Dify / RAG
→ documentación relativamente estable
→ runbooks
→ procedimientos
→ orientación interna

Dify / Agent + MCP
→ herramientas descubiertas dinámicamente
→ cambios recientes
→ alertas recientes
→ incidentes abiertos
→ health actual

LLM final
→ síntesis
→ separación de hechos e hipótesis
→ comprobaciones recomendadas
→ salida estructurada

Humano
→ autorización
→ juicio
→ acciones sensibles
```

Dify **no calculará ni modificará la prioridad**. Las reglas P1–P4 siguen siendo responsabilidad de n8n.

MCP tampoco ejecutará remediaciones. El servidor del laboratorio solo expone herramientas **read-only** de observación.

> Todos los datos son ficticios. No utilices secretos, credenciales o información productiva.

---

## Objetivos de aprendizaje

Al finalizar deberías ser capaz de:

- explicar qué responsabilidad corresponde a n8n, RAG, MCP, LLM y humano;
- crear una aplicación **Workflow** en Dify para uso máquina a máquina;
- reutilizar una Knowledge Base RAG desde un Workflow;
- utilizar una integración MCP compartida sin gestionar su secreto;
- configurar un nodo **Agent** con estrategia `FunctionCalling`;
- limitar el Agent a un conjunto concreto de herramientas MCP;
- inspeccionar qué tools decidió invocar el agente y con qué argumentos;
- diferenciar **documentación** de **estado operativo vivo**;
- sintetizar alerta validada + RAG + MCP mediante un LLM final;
- generar Structured Output sin permitir que la IA sustituya reglas deterministas;
- publicar un Workflow de Dify;
- crear un API key de aplicación;
- almacenar ese API key mediante Credentials en n8n;
- invocar Dify mediante `HTTP Request`;
- utilizar `response_mode=blocking`;
- validar tanto el HTTP como el estado funcional devuelto por Dify;
- conservar datos de n8n después de una llamada HTTP;
- registrar trazabilidad entre n8n, Dify y evidencia;
- razonar sobre qué debería ocurrir si Dify o MCP no están disponibles.

---

# Prerrequisitos

Necesitas:

- acceso a `https://n8n-agentes.cloudsecurityinpractice.com/`;
- acceso a `https://dify.cloudsecurityinpractice.com/`;
- M03 completado o un workflow equivalente;
- la Knowledge Base privada de M05, por ejemplo:

```text
AL01-M05-KB-Ops
```

- los cuatro documentos de M05 procesados;
- acceso a la API del laboratorio;
- tu identificador de alumno.

En los ejemplos utilizaremos:

```text
alumno01
AL01
```

Sustitúyelos por tu identificador real.

## Integraciones compartidas preparadas por el instructor

En el Workspace de Dify ya deben estar disponibles:

```text
MCP Server: Telvora Ops Hub
Agent Strategy: Dify Agent Strategies
```

No necesitas conocer el token MCP ni crear otra integración.

**No modifiques, elimines ni vuelvas a instalar estas integraciones compartidas.**

El Workspace de Dify es compartido. Las aplicaciones pueden ser visibles para otros alumnos. Edita únicamente Apps cuyo nombre comience por tu prefijo (`AL01-`, `AL02-`, etc.).

La Knowledge Base de M05 sí es privada y debes seleccionar únicamente la tuya.

---

# Material proporcionado

En `materiales/` encontrarás:

- `casos_integracion.json`;
- `schema_analisis_dify.json`;
- `prompt_agent_mcp.md`;
- `prompt_workflow_dify.md`;
- `contrato_mcp_telvora_ops_hub.md`;
- `contrato_dify_api.md`;
- `checklist_pruebas.md`.

M06 reutiliza la documentación indexada en M05.

---

# Arquitectura objetivo

```text
SISTEMA EMISOR
     |
     | POST
     v
n8n WEBHOOK
     |
     v
NORMALIZAR
     |
     v
VALIDAR
  /      \
no        sí
|          |
v          v
MANUAL   API ACTIVOS (REST)
              |
              v
         ¿ACTIVO EXISTE?
          /         \
        no           sí
        |             |
        v             v
     MANUAL       ENRIQUECER
                       |
                       v
                 CLASIFICAR P1-P4
                       |
                       v
                PREPARAR INPUT IA
                       |
                       v
                DIFY WORKFLOW API
                       |
                       v
             +----------------------+
             | WORKFLOW DE DIFY     |
             |                      |
             |  RAG                 |
             |   +                  |
             |  AGENT + MCP         |
             |   +                  |
             |  LLM FINAL           |
             |   +                  |
             |  STRUCTURED OUTPUT   |
             +----------------------+
                       |
                       v
                VALIDAR RESPUESTA
                       |
                       v
             COMBINAR DECISIÓN + IA
                       |
                       v
                REGISTRAR EVIDENCIA
                       |
                       v
                 RESPONDER HTTP
```

Las excepciones de entrada, activo desconocido o severidad inválida **no deben llamar a Dify** y, por tanto, tampoco deben consumir RAG ni MCP.

---

# Parte 1 — Crear una App Workflow en Dify

Crea una nueva aplicación de tipo:

```text
Workflow
```

Nombre:

```text
<ID_ALUMNO>-M06-Analisis-RAG-MCP-API
```

Ejemplo:

```text
AL01-M06-Analisis-RAG-MCP-API
```

No edites aplicaciones cuyo prefijo pertenezca a otro alumno.

## ¿Por qué Workflow y no Chatflow?

En M05 el usuario interactuaba directamente con un Chatflow.

Ahora Dify será invocado por n8n. Queremos una interfaz máquina a máquina basada en inputs y outputs, no una conversación persistente.

---

# Parte 2 — Definir inputs del Workflow

En el nodo **Start** crea cuatro variables de texto:

```text
alert_context
asset_id
retrieval_query
question
```

## alert_context

Contendrá la alerta ya validada, enriquecida y clasificada por n8n, serializada como texto JSON.

## asset_id

Contendrá únicamente el identificador del activo validado, por ejemplo:

```text
srv-web-01
```

Lo utilizaremos para construir la consulta del Agent MCP sin obligarle a inferir el identificador desde un bloque JSON.

## retrieval_query

Contendrá una consulta preparada por n8n con términos útiles para recuperar documentación.

## question

Describe el objetivo de análisis que esperamos del LLM final.

---

# Parte 3 — Añadir Knowledge Retrieval

Añade:

```text
01 - Recuperar conocimiento
```

Selecciona únicamente tu Knowledge Base de M05:

```text
<ID_ALUMNO>-M05-KB-Ops
```

Usa como query:

```text
retrieval_query
```

No selecciones la Knowledge Base de otro alumno.

La configuración de búsqueda permanece como en M05:

```text
Hybrid Search
Reranker: cohere.rerank-v3-5:0
Top K: 3
Score Threshold: OFF
```

## Qué representa esta rama

RAG responde a preguntas del tipo:

```text
¿Qué dice la documentación interna?
¿Qué procedimiento existe?
¿Qué comprobaciones prescribe el runbook?
```

No debe utilizarse como fuente de estado operativo actual.

---

# Parte 4 — Añadir el Agent MCP

Añade un nodo:

```text
02 - Investigar Ops Hub (MCP)
```

Estrategia agéntica:

```text
FunctionCalling
```

Modelo:

```text
openai.gpt-5.6-luna
```

Configuración:

```text
Temperature: OFF
Top P: OFF
Reasoning Effort: none/off si aparece
Maximum Iterations: 5
Memory: OFF
```

Deja `Allowed Tools` vacío.

## Tool List

Pulsa `+` y selecciona las cuatro herramientas de:

```text
Telvora Ops Hub
```

Habilita:

```text
get_recent_changes
get_recent_alerts
get_open_incidents
get_service_health
```

No añadas herramientas de otros proveedores.

## Instruction

Copia el contenido de:

```text
materiales/prompt_agent_mcp.md
```

## Query

Construye mediante variables de Dify una consulta equivalente a:

```text
Investiga el estado operativo actual de <asset_id>.
Consulta cambios recientes, alertas recientes, incidentes abiertos y el estado actual del servicio.
Distingue hechos obtenidos mediante herramientas de hipótesis.
```

Inserta `asset_id` desde **Start** utilizando el selector de variables.

## Context

Déjalo vacío.

El Agent consulta **estado vivo mediante tools**. La documentación RAG se incorpora después en el LLM final.

---

# Parte 5 — Comprender qué está haciendo MCP

En este laboratorio el servidor MCP se denomina:

```text
Telvora Ops Hub
```

Dify ya tiene configurada la conexión a nivel del Workspace.

El Agent no recibe manualmente una lista JSON de datos operativos. En tiempo de ejecución:

1. conoce las tools que tiene permitidas;
2. el modelo decide qué tool invocar;
3. genera los argumentos;
4. Dify llama al servidor MCP;
5. recibe el resultado;
6. puede llamar a otras tools;
7. genera una investigación final.

Consulta:

```text
materiales/contrato_mcp_telvora_ops_hub.md
```

para revisar el contrato de las cuatro tools.

---

# Parte 6 — Probar el Agent antes de continuar

Ejecuta el nodo Agent con:

```text
asset_id = srv-web-01
```

Inspecciona **Detalle / Estrategia agéntica**.

Debes poder identificar `tool_calls` reales.

No es obligatorio que el contenido vivo coincida exactamente con los ejemplos del enunciado. El Ops Hub mantiene estado operativo y las ventanas temporales cambian.

Lo que sí debes comprobar es:

- que el Agent utiliza tools de `Telvora Ops Hub`;
- que `asset_id` se envía correctamente;
- que los resultados proceden de las tools;
- que no inventa una acción destructiva;
- que distingue hechos de hipótesis.

---

# Parte 7 — Añadir el LLM final

Añade:

```text
03 - Sintetizar RAG + MCP
```

Modelo:

```text
openai.gpt-5.6-luna
```

Configuración:

```text
Temperature: OFF
Top P: OFF
Reasoning Effort: none/off si aparece
```

Conecta como **Context** la salida de:

```text
01 - Recuperar conocimiento
```

Copia el prompt de:

```text
materiales/prompt_workflow_dify.md
```

Inserta mediante el selector de variables:

- `alert_context` desde Start;
- `question` desde Start;
- `text` desde `02 - Investigar Ops Hub (MCP)`.

El último LLM es el encargado de combinar tres fuentes conceptualmente distintas:

```text
alert_context  → hechos validados por n8n
RAG            → documentación interna
Agent + MCP    → estado operativo vivo
```

---

# Parte 8 — Structured Output

En `03 - Sintetizar RAG + MCP` habilita:

```text
Structured Output
```

Carga:

```text
materiales/schema_analisis_dify.json
```

El objeto generado contiene:

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

Observa deliberadamente lo que **no** contiene:

```text
priority
severity
owner_team
```

Dify no es la fuente autoritativa de esos campos.

---

# Parte 9 — Añadir End

Añade un nodo **End**.

Expón una salida llamada:

```text
analysis
```

que apunte al `structured_output` del nodo:

```text
03 - Sintetizar RAG + MCP
```

Prueba el Workflow desde Dify con inputs manuales antes de integrarlo con n8n.

---

# Parte 10 — Inspeccionar la ejecución Dify

Antes de publicar, ejecuta un caso manual con:

```text
asset_id = srv-web-01
```

Revisa:

```text
01 - Recuperar conocimiento
02 - Investigar Ops Hub (MCP)
03 - Sintetizar RAG + MCP
```

En el Agent abre:

```text
Detalle
→ Estrategia agéntica
→ Round
```

Comprueba qué tools se ejecutaron y con qué argumentos.

Ésta es una evidencia importante del laboratorio: no basta con que la respuesta final mencione datos del Ops Hub; debes poder observar una llamada MCP.

---

# Parte 11 — Publicar el Workflow

Guarda y **publica** el Workflow.

La API de servicio utiliza la versión publicada de la aplicación.

No llames a la API antes de comprobar que una ejecución manual termina correctamente.

---

# Parte 12 — Crear API key de la aplicación

Desde **Acceso API** de la App crea un API key para el laboratorio.

No copies el API key en:

- el repositorio;
- una captura;
- el nodo HTTP Request;
- una nota compartida.

Lo utilizaremos únicamente para crear una credencial en n8n.

El token MCP **no lo gestiona el alumno**. Está asociado a la integración compartida del Workspace.

---

# Parte 13 — Preparar una nueva versión del workflow n8n

Duplica el workflow funcional de M03.

Nombre:

```text
M06-LAB06-n8n-Dify-RAG-MCP
```

No destruyas M03.

Cambia el Webhook a un path nuevo:

```text
m06-alert-<ID_ALUMNO>
```

Ejemplo:

```text
m06-alert-alumno01
```

Esto evita colisionar con el Webhook M03 publicado.

---

# Parte 14 — Mantener la lógica determinista

Conserva de M03:

```text
Webhook
Normalización
Validación
API de activos
found
Enriquecimiento
Switch de severity
If de high + production + criticality high
P1/P2/P3/P4
fallbacks
```

No muevas la decisión de prioridad a Dify.

Antes de integrar IA, vuelve a probar al menos una alerta P1 y una P2/P3/P4.

---

# Parte 15 — No llamar a Dify para entradas inválidas

Las ramas:

```text
missing_required_field
asset_not_found
invalid_severity
```

no deben ejecutar la llamada de Dify.

Por tanto tampoco deben:

```text
consultar RAG
invocar Agent
consumir MCP
consumir tokens LLM
```

Estas situaciones ya tienen una decisión determinista suficiente:

```text
manual_review
```

---

# Parte 16 — Unificar las rutas P1-P4 antes de Dify

Conecta las rutas válidas P1, P2, P3 y P4 a un nodo común:

```text
60 - Preparar análisis Dify
```

Debe conservar los datos de la alerta enriquecida y la prioridad ya calculada.

Añade cuatro campos auxiliares:

```text
alert_context
asset_id
retrieval_query
question
```

## alert_context

Serializa el item actual como JSON.

Expresión equivalente:

```text
{{ JSON.stringify($json) }}
```

## asset_id

Conserva el `asset_id` ya validado.

## retrieval_query

Construye una cadena con datos útiles para retrieval:

```text
asset_id + asset_name + summary + details
```

No incluyas secretos.

## question

Usa una instrucción estable:

```text
Analiza esta alerta combinando los hechos validados, la documentación operativa disponible y el estado operativo vivo. Recomienda comprobaciones técnicas no destructivas. Distingue hechos, orientación documentada e hipótesis.
```

---

# Parte 17 — Crear la credencial de Dify en n8n

Crea una credencial genérica de cabecera HTTP.

Nombre sugerido:

```text
M06 - Dify Workflow API
```

Configuración equivalente:

```text
Name: Authorization
Value: Bearer <DIFY_APP_API_KEY>
```

El secreto debe quedar almacenado en Credentials.

---

# Parte 18 — Invocar Dify

Añade **HTTP Request**:

```text
61 - Ejecutar Dify Workflow
```

Configura:

```text
Method: POST
URL: https://dify.cloudsecurityinpractice.com/v1/workflows/run
Authentication: M06 - Dify Workflow API
Content-Type: application/json
Response: JSON
```

Body:

```json
{
  "inputs": {
    "alert_context": "<desde 60>",
    "asset_id": "<desde 60>",
    "retrieval_query": "<desde 60>",
    "question": "<desde 60>"
  },
  "response_mode": "blocking",
  "user": "<ID_ALUMNO>-m06"
}
```

Utiliza expresiones de n8n. No copies datos de un caso concreto.

---

# Parte 19 — Inspeccionar la respuesta

Ejecuta INT-01 e inspecciona la salida completa de:

```text
61 - Ejecutar Dify Workflow
```

Localiza:

```text
workflow_run_id o identificador equivalente
data.status
data.outputs.analysis
data.elapsed_time
data.total_tokens
```

No asumas la estructura: compruébala en la ejecución real.

Además abre la ejecución en Dify y localiza el Agent para comprobar los `tool_calls` MCP.

---

# Parte 20 — Validar el estado funcional de Dify

Añade:

```text
62 - Dify completado correctamente
```

Comprueba al menos:

```text
data.status = succeeded
```

y que exista:

```text
data.outputs.analysis
```

## ¿Por qué no basta HTTP 200?

Porque HTTP indica que la petición fue aceptada y procesada por el servicio, pero la propia ejecución del Workflow tiene un estado funcional que debemos inspeccionar.

---

# Parte 21 — Combinar datos deterministas e IA

En TRUE añade:

```text
63 - Combinar decisión y análisis IA
```

Después de `61 - Ejecutar Dify Workflow`, `$json` representa la respuesta de Dify.

Recupera desde `60 - Preparar análisis Dify`:

```text
alert_id
asset_id
severity
priority
status
requires_human_review
owner_team
```

Y desde la respuesta Dify:

```text
analysis
workflow_run_id / data.id
elapsed_time
total_tokens
```

Añade:

```text
ai_status = succeeded
```

No sustituyas `priority` por ningún dato del modelo.

---

# Parte 22 — Evidencia final

Amplía la evidencia de M03 para registrar, como mínimo:

```text
alert_id
asset_id
severity
priority
status
requires_human_review
owner_team
workflow_version = m06-v2
ai_status
dify_run_id
```

La API de evidencias permite campos adicionales. Puedes incluir:

```text
ai_confidence
documentation_status
operational_context_status
```

No es necesario almacenar toda la respuesta del LLM.

No registres:

```text
DIFY_APP_API_KEY
MCP_TOKEN
headers Authorization
```

---

# Parte 23 — Respuesta al emisor

La respuesta puede tener una estructura conceptual como:

```json
{
  "accepted": true,
  "alert_id": "alumno01-ALT-6001",
  "priority": "P1",
  "status": "pending_human_review",
  "requires_human_review": true,
  "owner_team": "WebOps",
  "ai": {
    "status": "succeeded",
    "confidence": "medium",
    "summary": "...",
    "operational_facts": ["..."],
    "recommended_checks": ["..."],
    "documentation_status": "relevant_context_found",
    "operational_context_status": "relevant_live_context_found",
    "dify_run_id": "..."
  },
  "evidence_id": "ev-..."
}
```

Separa visualmente datos de proceso y datos de IA.

---

# Parte 24 — Sustituir `alumnoXX` en los casos

`casos_integracion.json` utiliza IDs como:

```text
alumnoXX-ALT-6001
```

Antes de enviar los eventos sustituye `alumnoXX` por tu identificador real.

Ejemplo:

```text
alumno01-ALT-6001
```

Esto reduce colisiones entre evidencias generadas por alumnos distintos.

---

# Parte 25 — Ejecutar INT-01

Esperado:

```text
asset srv-web-01
→ found=true
→ high + production + criticality high
→ P1
→ Dify sí
→ RAG sí
→ Agent/MCP sí
```

RAG debería aportar orientación documental del entorno web.

MCP debe aportar el estado vivo disponible en ese momento: cambios, alertas, incidentes y/o health.

**No esperes valores temporales idénticos a los de otro compañero.**

Dify no debe afirmar que un despliegue causó la degradación únicamente por proximidad temporal.

---

# Parte 26 — Ejecutar INT-02

Esperado:

```text
srv-db-01
critical
→ P1
→ Dify sí
→ RAG sí
→ Agent/MCP sí
```

RAG debería recuperar `DB-HEALTH-01`.

El Ops Hub puede devolver o no eventos vivos relacionados, según el escenario activo.

La ausencia de un evento MCP no autoriza a inventarlo.

---

# Parte 27 — Ejecutar INT-03

Esperado:

```text
srv-api-01
medium
→ P3
→ Dify sí
→ Agent/MCP sí
```

La Knowledge Base contiene inventario y cambios documentados.

MCP aporta el estado vivo disponible.

El análisis debe diferenciar:

```text
cambio documentado en RAG
cambio reciente devuelto por MCP
```

si ambos aparecen.

---

# Parte 28 — Ejecutar INT-04

Esperado:

```text
srv-batch-01
high + production + criticality medium
→ P2
→ Dify sí
```

La base documental identifica el activo, pero no contiene un runbook específico de retrasos de facturación.

Si MCP tampoco aporta datos relevantes, un buen resultado debe reflejar:

```text
documentation_status = insufficient_context
operational_context_status = no_relevant_live_context
```

en lugar de inventar un procedimiento o un incidente.

---

# Parte 29 — Ejecutar INT-05, INT-06 e INT-07

## INT-05

Falta `asset_id`.

```text
manual_review
reason = missing_required_field
Dify = NO
MCP = NO
```

## INT-06

Activo desconocido:

```text
srv-unknown-99
```

Esperado:

```text
API activos → found=false
manual_review
reason = asset_not_found
Dify = NO
MCP = NO
```

## INT-07

Severity:

```text
urgent
```

Esperado:

```text
invalid_severity
manual_review
Dify = NO
MCP = NO
```

Comprueba en n8n que `61 - Ejecutar Dify Workflow` no se ejecutó.

Si Dify no se ejecuta, el Agent MCP tampoco puede ejecutarse.

---

# Parte 30 — Ejecutar INT-08

Esperado:

```text
severity low
→ P4
→ Dify sí
→ RAG sí
→ MCP sí
```

Dify puede recuperar documentación y estado vivo, pero no debe cambiar P4 ni convertir un episodio transitorio en una causa raíz confirmada.

---

# Parte 31 — Prueba explícita de observabilidad MCP

Elige una ejecución válida, preferentemente INT-01.

En Dify:

```text
Registros / ejecución
→ Agent
→ Detalle
→ Estrategia agéntica
→ Round
```

Documenta al menos una llamada:

```text
tool_call_name
tool_call_input
tool_response
```

Ejemplo conceptual:

```text
tool_call_name = get_service_health
tool_call_input.asset_id = srv-web-01
```

No copies tokens ni headers.

Esta prueba demuestra que el agente **consultó realmente MCP** y no generó la respuesta únicamente con conocimiento del modelo.

---

# Parte 32 — Prueba de fallo controlado de Dify

Cambia temporalmente la URL de:

```text
/v1/workflows/run
```

a:

```text
/v1/workflows/runn
```

Ejecuta INT-01.

Observa:

- la prioridad P1 ya se calculó antes;
- el error ocurre en la integración con Dify;
- RAG y MCP dejan de aportar análisis porque no se ejecuta el Workflow;
- la clasificación determinista no se ha vuelto incorrecta.

Restaura la URL después de la prueba.

La implementación completa del fallback se endurecerá en M07.

---

# Parte 33 — Diferenciar cuatro tipos de información

## 1. Estado determinista

```text
severity
priority
status
requires_human_review
owner_team
```

Procede del proceso controlado por n8n.

## 2. Conocimiento documental

```text
documented_guidance
documentation_status
```

Procede del RAG.

## 3. Estado operativo vivo

```text
operational_facts
operational_context_status
```

Procede del Agent que consulta Telvora Ops Hub mediante MCP.

## 4. Interpretación IA

```text
summary
hypotheses
recommended_checks
confidence
missing_information
```

Es información de apoyo generada por el modelo.

No mezcles estos cuatro conceptos.

---

# Parte 34 — Trazabilidad de extremo a extremo

Para una ejecución válida deberías poder correlacionar:

```text
alert_id
n8n execution
asset API request
priority
Dify workflow_run_id
Dify Agent rounds
MCP tool calls
Dify outputs
Dify token/latency metadata
evidence_id
HTTP response
```

No necesitas implementar una plataforma de observabilidad. Debes ser capaz de reconstruir qué ocurrió.

---

# Parte 35 — Seguridad

Analiza:

1. ¿qué ocurriría si se filtrase el API key de la App Dify?;
2. ¿por qué debe almacenarse en Credentials de n8n?;
3. ¿por qué el token MCP no se entrega a cada alumno?;
4. ¿por qué el servidor MCP solo expone tools read-only?;
5. ¿qué riesgo tendría exponer una tool `restart_server` al mismo Agent?;
6. ¿qué datos del `alert_context` podrían ser sensibles en un entorno real?;
7. ¿por qué Dify no debería recibir eventos inválidos?;
8. ¿qué permisos deberían existir sobre la Knowledge Base?;
9. ¿qué riesgo supone que las Apps estén en un Workspace compartido?;
10. ¿por qué una respuesta del LLM no debe convertirse directamente en una acción destructiva?

---

# Parte 36 — Publicar n8n

Cuando las pruebas con Test URL hayan terminado:

1. guarda;
2. pulsa **Publish**;
3. copia la Production URL;
4. comprueba que utiliza `m06-alert-<ID_ALUMNO>`;
5. ejecuta INT-01;
6. localiza la ejecución en el historial de n8n;
7. localiza el `workflow_run_id` en Dify;
8. abre el detalle del Agent y observa los rounds MCP.

---

# Entrega

Entrega:

1. captura del Workflow Dify completo;
2. captura del workflow n8n;
3. nombre de la Knowledge Base utilizada;
4. evidencia de una llamada Dify exitosa;
5. evidencia de al menos una llamada MCP (`tool_call_name` + argumentos, sin secretos);
6. respuesta final de INT-01;
7. prueba de que INT-05, INT-06 e INT-07 no llaman a Dify/MCP;
8. resultado de INT-04 mostrando cómo se maneja contexto insuficiente;
9. resultado de la prueba de fallo controlado de Dify;
10. checklist completado;
11. explicación breve de qué información controla n8n, cuál procede de RAG, cuál de MCP y cuál es interpretación del LLM.

---

# Resultado esperado

Al finalizar dispondrás de una arquitectura híbrida equivalente a:

```text
n8n
  ├─ recibe
  ├─ valida
  ├─ consulta REST
  ├─ enriquece
  ├─ calcula prioridad
  └─ decide si merece invocar IA
         |
         v
      Dify Workflow
         |
         +--> Knowledge Retrieval
         |      └─ documentación / RAG
         |
         +--> Agent FunctionCalling
         |      └─ Telvora Ops Hub / MCP
         |           ├─ recent changes
         |           ├─ recent alerts
         |           ├─ open incidents
         |           └─ service health
         |
         +--> LLM final
         |      └─ Structured Output
         |
         v
        n8n
         ├─ valida respuesta IA
         ├─ conserva autoridad determinista
         ├─ registra evidencia
         └─ responde
```

---

# Comprobación final

Antes de terminar debes poder explicar:

- por qué n8n y Dify tienen responsabilidades distintas;
- por qué usamos Workflow en Dify;
- qué diferencia existe entre REST, RAG y MCP;
- por qué MCP resulta apropiado para consultar estado operativo vivo;
- cómo descubre el Agent las tools que puede utilizar;
- por qué utilizamos `FunctionCalling`;
- por qué no entregamos el token MCP a cada alumno;
- por qué el Agent MCP no controla `priority`;
- por qué RAG no debe representar el health actual del servicio;
- por qué se utiliza un LLM final después del Agent;
- qué contiene `inputs`;
- qué significa `response_mode=blocking`;
- por qué se valida `data.status`;
- cómo se recuperan datos anteriores después de HTTP Request;
- por qué algunas alertas no deben consumir IA;
- qué ocurre si Dify falla;
- qué identificadores permiten correlacionar una ejecución completa.
