# Laboratorio 6 — Integración n8n + Dify

## Contexto

Hasta ahora hemos construido dos piezas separadas:

- **n8n** recibe alertas, valida datos, consulta APIs, enriquece información, aplica reglas deterministas y registra evidencias;
- **Dify** interpreta contexto técnico, recupera documentación mediante RAG y genera análisis controlados.

En este laboratorio uniremos ambas plataformas.

El objetivo no es convertir Dify en el cerebro de toda la automatización. La arquitectura mantendrá una división clara de responsabilidades:

```text
n8n
→ entrada
→ validación
→ integración
→ reglas
→ prioridad
→ credenciales
→ trazabilidad

Dify
→ retrieval
→ interpretación
→ hipótesis
→ orientación documental
→ comprobaciones recomendadas

Humano
→ autorización
→ juicio
→ acciones sensibles
```

Dify **no calculará ni modificará la prioridad**. Las reglas P1-P4 siguen siendo responsabilidad de n8n.

> Todos los datos son ficticios. No utilices secretos, credenciales o información productiva.

---

## Objetivos de aprendizaje

Al finalizar deberías ser capaz de:

- explicar cómo dividir responsabilidades entre un orquestador y un componente de IA;
- crear una aplicación **Workflow** en Dify para uso máquina a máquina;
- reutilizar una Knowledge Base RAG desde un Workflow;
- publicar un Workflow de Dify;
- crear un API key de aplicación;
- almacenar ese API key mediante Credentials en n8n;
- invocar Dify mediante `HTTP Request`;
- construir inputs dinámicos desde una alerta enriquecida;
- utilizar `response_mode=blocking`;
- validar tanto el HTTP como el estado funcional devuelto por Dify;
- recuperar outputs estructurados;
- conservar datos de n8n después de una llamada HTTP;
- integrar análisis IA sin permitir que sustituya reglas deterministas;
- diseñar fallback ante caída o error del servicio de IA;
- correlacionar trazas de n8n y Dify.

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

---

# Material proporcionado

En `materiales/` encontrarás:

- `casos_integracion.json`;
- `schema_analisis_dify.json`;
- `prompt_workflow_dify.md`;
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
MANUAL   API ACTIVOS
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

Las excepciones de entrada, activo desconocido o severidad inválida **no necesitan llamar al LLM**.

---

# Parte 1 — Crear una App Workflow en Dify

Crea una nueva aplicación de tipo:

```text
Workflow
```

Nombre:

```text
<ID_ALUMNO>-M06-Analisis-RAG-API
```

Ejemplo:

```text
AL01-M06-Analisis-RAG-API
```

No edites aplicaciones cuyo prefijo pertenezca a otro alumno.

## ¿Por qué Workflow y no Chatflow?

En M05 el usuario interactuaba directamente con un Chatflow.

Ahora Dify será invocado por n8n. Queremos una interfaz máquina a máquina basada en inputs y outputs, no una conversación persistente.

---

# Parte 2 — Definir inputs del Workflow

En el nodo **Start** crea tres variables de texto:

```text
alert_context
retrieval_query
question
```

## alert_context

Contendrá la alerta ya validada, enriquecida y clasificada por n8n, serializada como texto JSON.

## retrieval_query

Contendrá una consulta preparada por n8n con términos relevantes para buscar documentación.

## question

Describe el objetivo de análisis que esperamos del LLM.

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

---

# Parte 4 — Añadir el LLM

Añade:

```text
02 - Analizar con RAG
```

Modelo:

```text
openai.gpt-5.6-luna
```

Configuración del laboratorio:

```text
Temperature: OFF
Top P: OFF
Reasoning Effort: none/off si aparece
```

Conecta como **Context** la salida de `01 - Recuperar conocimiento`.

Copia el prompt de:

```text
materiales/prompt_workflow_dify.md
```

Inserta `alert_context` y `question` mediante el selector de variables de Dify.

---

# Parte 5 — Structured Output

En el nodo LLM habilita:

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
documented_guidance[]
hypotheses[]
recommended_checks[]
missing_information[]
confidence
documentation_status
```

Observa deliberadamente lo que **no** contiene:

```text
priority
severity
owner_team
```

Dify no es la fuente autoritativa de esos campos.

---

# Parte 6 — Añadir End

Añade un nodo **End**.

Expón una salida llamada:

```text
analysis
```

que apunte al `structured_output` del nodo LLM.

Si tu interfaz permite añadir información adicional, puedes exponer también el texto libre, pero `analysis` es la salida necesaria para la integración.

Prueba el Workflow desde Dify con inputs manuales antes de integrarlo con n8n.

---

# Parte 7 — Publicar el Workflow

Guarda y **publica** el Workflow.

La API de servicio utiliza la versión publicada de la aplicación.

No llames a la API antes de comprobar que una ejecución manual del Workflow termina correctamente.

---

# Parte 8 — Crear API key de la aplicación

Desde el acceso API de la App crea un API key para el laboratorio.

No copies el API key en:

- el repositorio;
- una captura;
- el nodo HTTP Request;
- una nota compartida.

Lo utilizaremos únicamente para crear una credencial en n8n.

---

# Parte 9 — Preparar una nueva versión del workflow n8n

Duplica el workflow funcional de M03.

Nombre:

```text
M06-LAB06-n8n-Dify-RAG
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

# Parte 10 — Mantener la lógica determinista

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

# Parte 11 — No llamar a Dify para entradas que no pasan los controles

Las ramas:

```text
missing_required_field
asset_not_found
invalid_severity
```

no deben ejecutar la llamada de Dify.

Estas situaciones ya tienen una decisión determinista suficiente:

```text
manual_review
```

Además evitamos consumir tokens en entradas que no han superado los controles básicos.

---

# Parte 12 — Unificar las rutas P1-P4 antes de Dify

Conecta las rutas válidas P1, P2, P3 y P4 a un nodo común:

```text
60 - Preparar análisis Dify
```

Debe conservar los datos de la alerta enriquecida y la prioridad ya calculada.

Añade tres campos auxiliares:

```text
alert_context
retrieval_query
question
```

## alert_context

Serializa el item actual como JSON.

Una expresión equivalente puede utilizar:

```text
{{ JSON.stringify($json) }}
```

## retrieval_query

Construye una cadena con datos útiles para retrieval, por ejemplo:

```text
asset_id + asset_name + summary + details
```

No incluyas secretos.

## question

Usa una instrucción estable, por ejemplo:

```text
Analiza esta alerta, utiliza la documentación operativa disponible y recomienda qué comprobaciones técnicas no destructivas deberían realizarse a continuación. Distingue hechos, orientación documentada e hipótesis.
```

---

# Parte 13 — Crear la credencial de Dify en n8n

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

# Parte 14 — Invocar Dify

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
    "retrieval_query": "<desde 60>",
    "question": "<desde 60>"
  },
  "response_mode": "blocking",
  "user": "<ID_ALUMNO>-m06"
}
```

Utiliza expresiones de n8n. No copies datos de un caso concreto.

---

# Parte 15 — Inspeccionar la respuesta

Antes de continuar, ejecuta INT-01 e inspecciona la salida completa de `61 - Ejecutar Dify Workflow`.

Localiza:

```text
workflow_run_id o identificador de ejecución equivalente
data.status
data.outputs.analysis
data.elapsed_time
data.total_tokens
```

No asumas la estructura: compruébala en la ejecución real.

---

# Parte 16 — Validar el estado funcional de Dify

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

Porque HTTP indica que la petición fue procesada por el servicio, pero la propia ejecución del Workflow tiene un estado funcional que debemos inspeccionar.

---

# Parte 17 — Combinar datos deterministas e IA

En TRUE añade:

```text
63 - Combinar decisión y análisis IA
```

Después de `61 - Ejecutar Dify Workflow`, `$json` representa la respuesta de Dify.

Recupera desde `60 - Preparar análisis Dify` los campos deterministas:

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

# Parte 18 — Evidencia final

Amplía la evidencia de M03 para registrar, como mínimo:

```text
alert_id
asset_id
severity
priority
status
requires_human_review
owner_team
workflow_version = m06-v1
ai_status
dify_run_id
```

Si la API de evidencias permite campos adicionales, puedes incluir un resumen reducido del análisis. No es necesario almacenar toda la respuesta del LLM para completar el laboratorio.

No registres el API key de Dify.

---

# Parte 19 — Respuesta al emisor

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
    "recommended_checks": ["..."],
    "documentation_status": "relevant_context_found",
    "dify_run_id": "..."
  },
  "evidence_id": "ev-..."
}
```

Separa visualmente datos de proceso y datos de IA.

---

# Parte 20 — Sustituir `alumnoXX` en los casos

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

# Parte 21 — Ejecutar INT-01

Esperado:

```text
asset srv-web-01
→ found=true
→ high + production + criticality high
→ P1
→ Dify sí
```

La documentación recuperada debería aportar orientación del runbook web y puede aportar información del historial de despliegues.

Dify no debe afirmar que el despliegue sea la causa únicamente por proximidad temporal.

---

# Parte 22 — Ejecutar INT-02

Esperado:

```text
srv-db-01
critical
→ P1
→ Dify sí
```

El análisis debería recuperar `DB-HEALTH-01` y recomendar comprobar:

- conectividad;
- motor;
- disco;
- memoria;
- conexiones;
- replicación.

No debe inventar una causa raíz.

---

# Parte 23 — Ejecutar INT-03

Esperado:

```text
srv-api-01
medium
→ P3
→ Dify sí
```

La Knowledge Base contiene inventario de `customer-api-01` y un despliegue documentado.

El análisis puede utilizarlo como contexto, pero no debe afirmar que el despliegue causó la latencia.

---

# Parte 24 — Ejecutar INT-04

Esperado:

```text
srv-batch-01
high + production + criticality medium
→ P2
→ Dify sí
```

La base documental identifica el activo, pero no contiene un runbook específico de retrasos de facturación.

Un buen resultado debe reconocer que la orientación documental es insuficiente en lugar de inventar un procedimiento interno.

---

# Parte 25 — Ejecutar INT-05

Falta `asset_id`.

Esperado:

```text
manual_review
reason = missing_required_field
Dify = NO
```

Comprueba en el historial que `61 - Ejecutar Dify Workflow` no se ejecutó.

---

# Parte 26 — Ejecutar INT-06

Activo:

```text
srv-unknown-99
```

Esperado:

```text
API activos → found=false
manual_review
reason = asset_not_found
Dify = NO
```

---

# Parte 27 — Ejecutar INT-07

Severity:

```text
urgent
```

Esperado:

```text
invalid_severity
manual_review
Dify = NO
```

El activo puede existir y la API puede funcionar correctamente; aun así no llamamos al LLM porque la entrada no supera el contrato de clasificación.

---

# Parte 28 — Ejecutar INT-08

Esperado:

```text
severity low
→ P4
→ Dify sí
```

Dify puede recuperar contexto de `customer-api-01`, pero no debe cambiar P4 ni convertir un episodio transitorio en una causa raíz confirmada.

---

# Parte 29 — Probar un fallo controlado de Dify

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
- la clasificación determinista no se ha vuelto incorrecta.

Restaura la URL después de la prueba.

---

# Parte 30 — Diseñar fallback ante caída de Dify

No es obligatorio implementar toda la estrategia, pero debes diseñarla.

Una política razonable podría ser:

```text
Dify disponible
→ análisis IA
→ evidencia normal

Dify no disponible tras reintentos limitados
→ conservar priority
→ ai_status = unavailable
→ registrar error técnico
→ responder sin análisis IA
→ opcionalmente elevar revisión humana según política
```

La caída de la IA no debe convertir:

```text
P1
```

en:

```text
P3
```

o viceversa.

---

# Parte 31 — Diferenciar tres estados

## Estado determinista

```text
priority
status
requires_human_review
```

procede de n8n.

## Estado de integración IA

```text
ai_status
Dify HTTP
data.status
```

indica si el componente IA funcionó.

## Contenido IA

```text
summary
recommended_checks
hypotheses
confidence
```

es información de apoyo.

No mezcles los tres conceptos.

---

# Parte 32 — Trazabilidad de extremo a extremo

Para una ejecución válida deberías poder correlacionar:

```text
alert_id
n8n execution
asset API request
priority
Dify workflow_run_id
Dify outputs
Dify token/latency metadata
evidence_id
HTTP response
```

No necesitas implementar una plataforma de observabilidad; debes ser capaz de explicar qué identificadores guardarías.

---

# Parte 33 — Seguridad

Analiza:

1. ¿qué ocurriría si se filtrase el API key de Dify?;
2. ¿por qué debe ser específico de la App?;
3. ¿por qué no debemos enviarlo al endpoint de evidencias?;
4. ¿qué datos del alert_context podrían ser sensibles en un entorno real?;
5. ¿por qué Dify no debería recibir eventos inválidos?;
6. ¿qué permisos deberían existir sobre la Knowledge Base?;
7. ¿qué riesgo supone que las Apps de Dify estén en un Workspace compartido durante esta formación?;
8. ¿por qué un output de IA no debe convertirse directamente en una acción destructiva?

---

# Parte 34 — Publicar n8n

Cuando las pruebas con Test URL hayan terminado:

1. guarda;
2. pulsa **Publish**;
3. copia la Production URL;
4. comprueba que utiliza `m06-alert-<ID_ALUMNO>`;
5. ejecuta INT-01;
6. localiza la ejecución en el historial de n8n y en Dify.

---

# Entrega

Entrega:

1. captura del Workflow Dify;
2. captura del workflow n8n;
3. nombre de la Knowledge Base utilizada;
4. evidencia de una llamada Dify exitosa;
5. respuesta final de INT-01;
6. prueba de que INT-05, INT-06 e INT-07 no llaman a Dify;
7. resultado de INT-04 mostrando cómo se maneja documentación insuficiente;
8. resultado de la prueba de fallo controlado;
9. checklist completado;
10. explicación de qué datos controla n8n y cuáles aporta Dify.

---

# Resultado esperado

Al finalizar dispondrás de una arquitectura híbrida equivalente a:

```text
n8n
  ├─ recibe
  ├─ valida
  ├─ consulta API
  ├─ enriquece
  ├─ calcula prioridad
  ├─ decide si merece invocar IA
  |
  └─→ Dify Workflow
          ├─ retrieval
          ├─ RAG
          ├─ LLM
          └─ structured output
          |
          v
      n8n
       ├─ valida respuesta IA
       ├─ combina sin perder autoridad determinista
       ├─ registra evidencia
       └─ responde
```

---

# Comprobación final

Antes de terminar debes poder explicar:

- por qué n8n y Dify tienen responsabilidades distintas;
- por qué usamos Workflow en Dify;
- por qué la App debe estar publicada;
- cómo se autentica n8n frente a Dify;
- qué contiene `inputs`;
- qué significa `response_mode=blocking`;
- por qué se valida `data.status`;
- cómo se recuperan datos anteriores después de HTTP Request;
- por qué Dify no controla priority;
- por qué algunas alertas no deben consumir IA;
- qué ocurre si Dify falla;
- qué identificadores permiten correlacionar una ejecución completa.
