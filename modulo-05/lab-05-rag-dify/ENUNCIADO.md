# Laboratorio 5 — RAG y conocimiento interno en Dify

## Contexto

En M04 la automatización ya podía:

- recibir una alerta mediante Webhook en n8n;
- consultar el activo mediante API;
- calcular una prioridad P1–P4 de forma determinista;
- enviar el contexto a una Workflow App de Dify;
- utilizar un LLM para separar hechos, hipótesis, comprobaciones e información faltante.

Sin embargo, el LLM solo conoce la información incluida en `alert_context`.

Si preguntamos, por ejemplo:

> ¿Qué procedimiento interno debe seguirse ante este fallo?

> ¿Cuándo se realizó el último despliegue de este servicio?

> ¿Qué política de backup aplica a esta base de datos?

la respuesta no puede obtenerse a partir de la alerta.

En este laboratorio añadiremos **RAG (Retrieval-Augmented Generation)** para que el análisis pueda utilizar documentación interna de Telvora.

La prioridad seguirá siendo responsabilidad de n8n. RAG aporta conocimiento documental; no sustituye las reglas deterministas ni autoriza acciones sobre sistemas.

> Todos los datos y documentos son ficticios y se utilizan exclusivamente con fines formativos.

---

## Objetivos

Al finalizar deberías ser capaz de:

- crear una Knowledge Base en Dify;
- cargar y segmentar documentación operativa;
- comprender qué información recupera una búsqueda semántica/híbrida;
- inspeccionar fragmentos recuperados antes de involucrar al LLM;
- comprobar el efecto de `Top K` sobre la recuperación;
- añadir `Knowledge Retrieval` a una Workflow App existente;
- separar la consulta de recuperación (`retrieval_query`) del objetivo de análisis (`question`);
- obligar al LLM a fundamentar hechos operativos en documentación recuperada;
- reconocer cuándo la documentación no contiene una respuesta;
- integrar RAG dentro del mismo flujo n8n → Dify construido en módulos anteriores.

---

## Infraestructura

Dify:

```text
https://dify.cloudsecurityinpractice.com/
```

n8n:

```text
https://n8n-agentes.cloudsecurityinpractice.com/
```

El proveedor de modelos y los modelos de embeddings/reranking ya están configurados por el instructor.

No modifiques proveedores, credenciales globales, plugins ni aplicaciones de otros alumnos.

Utiliza siempre tu identificador como prefijo, por ejemplo:

```text
AL01-M05-KB-Ops
AL01-M05-Incident-Advisor-RAG
```

---

## Material proporcionado

En `materiales/` encontrarás:

- `prompt_rag.md`;
- `casos_rag.json`;
- `eventos_webhook_m05.json`;
- `comportamiento_esperado.md`;
- `documentos/` con cuatro documentos internos ficticios de Telvora.

---

# Parte 1 — Crear la Knowledge Base

En Dify abre **Knowledge** y crea una Knowledge Base nueva.

Nombre recomendado:

```text
<ID_ALUMNO>-M05-KB-Ops
```

Ejemplo:

```text
AL01-M05-KB-Ops
```

Si la interfaz permite elegir visibilidad, mantenla privada/personal para no interferir con el trabajo de otros alumnos.

---

# Parte 2 — Configurar el pipeline de ingestión

Utiliza el template/pipeline:

```text
Parent-child-HQ
```

En el entorno del curso queremos trabajar únicamente con archivos subidos por el alumno.

Si el template muestra fuentes o conectores que no se utilizan, deja como fuente efectiva únicamente el archivo cargado y elimina referencias no necesarias.

## Chunking

Utiliza esta configuración:

```text
Parent Mode: paragraph
Parent Delimiter: \n\n
Maximum Parent Length: 1024
Child Delimiter: \n
Maximum Child Length: 256
Replace consecutive spaces/newlines/tabs: ON
Delete URLs/emails: OFF
```

La idea es utilizar fragmentos pequeños para localizar información relevante, conservando suficiente contexto alrededor de la evidencia recuperada.

## Indexación

Configura:

```text
Indexing / Quality: High Quality
Embedding model: amazon.titan-embed-text-v2:0
Automatic summary: OFF
```

## Retrieval

Configura inicialmente:

```text
Retrieval: Hybrid Search
Rerank model: cohere.rerank-v3-5:0
Top K: 3
Score Threshold: OFF
```

No necesitas configurar credenciales de AWS ni del reranker.

---

# Parte 3 — Cargar los documentos

Carga los cuatro archivos de:

```text
materiales/documentos/
```

Documentos:

```text
telvora_inventario_operativo.txt
telvora_runbook_incidentes.txt
telvora_backups_operacion.txt
telvora_cambios_despliegues.txt
```

Espera a que todos aparezcan correctamente procesados e indexados.

---

# Parte 4 — Inspeccionar los chunks

Antes de conectar ningún LLM, abre uno o dos documentos y revisa cómo han sido segmentados.

Busca ejemplos donde un fragmento contenga información como:

```text
WEB-AVAIL-01
```

```text
DB-HEALTH-01
```

```text
CHG-4836
```

Comprueba que el fragmento recuperable conserva suficiente contexto para entender qué significa el dato.

La primera pregunta que debemos hacernos cuando una respuesta RAG es incorrecta no es siempre "¿qué hizo mal el LLM?". Primero debemos comprobar si la evidencia adecuada llegó a recuperarse.

---

# Parte 5 — Probar retrieval directamente

Utiliza la función de prueba de recuperación de la Knowledge Base.

Prueba estas consultas:

```text
¿Cómo se llama el servidor web de producción y en qué región está desplegado?
```

```text
¿Qué debe comprobar Operaciones primero si el servicio web de producción deja de responder?
```

```text
customer-api-01 último despliegue pipeline cambio
```

```text
orders-db-01 health checks fallidos comprobaciones
```

Para cada consulta observa:

- qué documento se recupera;
- qué fragmento contiene la evidencia;
- el orden de los resultados;
- el score, si la interfaz lo muestra.

Todavía no estamos evaluando la redacción del LLM. Estamos comprobando si el sistema encuentra la información adecuada.

---

# Parte 6 — Experimentar con Top K

Utiliza temporalmente una consulta que necesite información de más de un documento:

```text
Para srv-web-01 con errores después de un cambio, ¿qué recomienda el runbook y qué despliegue reciente existe para el frontend?
```

1. configura temporalmente `Top K = 1`;
2. ejecuta la búsqueda;
3. anota qué evidencia aparece;
4. cambia a `Top K = 3`;
5. repite la búsqueda.

Comprueba si con `Top K = 3` aparecen fragmentos de más de una fuente relevante.

No concluyas que un valor mayor siempre es mejor: recuperar más texto también puede introducir información irrelevante.

Al terminar deja:

```text
Top K = 3
```

---

# Parte 7 — Evolucionar la Workflow App de M04

Duplica tu Workflow App funcional de M04 para conservar la versión anterior.

Renombra la copia:

```text
<ID_ALUMNO>-M05-Incident-Advisor-RAG
```

Ejemplo:

```text
AL01-M05-Incident-Advisor-RAG
```

En el nodo **Start** conserva:

```text
alert_context
question
```

y añade una tercera variable obligatoria:

```text
retrieval_query
```

Las tres variables tienen responsabilidades diferentes:

```text
alert_context
→ datos técnicos de la incidencia

retrieval_query
→ texto utilizado para encontrar documentación relevante

question
→ resultado que queremos que produzca el LLM
```

---

# Parte 8 — Añadir Knowledge Retrieval

Añade un nodo:

```text
01 - Recuperar conocimiento
```

Tipo:

```text
Knowledge Retrieval
```

Selecciona únicamente tu Knowledge Base:

```text
<ID_ALUMNO>-M05-KB-Ops
```

Utiliza como consulta la variable:

```text
retrieval_query
```

La aplicación debe empezar a parecerse a:

```text
START
  |
  |-- alert_context
  |-- retrieval_query
  `-- question
          |
          v
  KNOWLEDGE RETRIEVAL
          |
          v
         LLM
          |
          v
         END
```

---

# Parte 9 — Conectar el contexto recuperado al LLM

Mantén el nodo LLM utilizado en M04.

Abre:

```text
materiales/prompt_rag.md
```

Utilízalo como base del nuevo prompt.

Mediante el selector de variables de Dify, inserta:

- `alert_context`;
- `question`;
- la salida/contexto del nodo `01 - Recuperar conocimiento`.

No copies manualmente resultados concretos de la Knowledge Base dentro del prompt.

Mantén en el nodo **End** la salida:

```text
analysis
```

asociada al texto generado por el LLM.

---

# Parte 10 — Probar la Workflow App directamente

Abre:

```text
materiales/casos_rag.json
```

Ejecuta `RAG-01` a `RAG-06`.

Para cada caso introduce:

```text
alert_context
retrieval_query
question
```

Comprueba el resultado con `materiales/comportamiento_esperado.md`.

## Dos pruebas especialmente importantes

### RAG-04

Pregunta por una contraseña que no está documentada.

El modelo no debe inventarla ni proponer una contraseña probable.

### RAG-05

Pregunta por el puerto SSH de `web-prod-01`.

El inventario indica que el puerto principal del servicio web es `443/TCP`, pero no documenta el puerto SSH.

El modelo no debe responder `22` únicamente porque sea un valor habitual.

---

# Parte 11 — Observar qué ha cambiado respecto a M04

Utiliza `RAG-03`.

En M04, si preguntábamos por el último despliegue de `customer-api-01`, el comportamiento correcto era reconocer que esa información no estaba en `alert_context`.

Ahora la Knowledge Base contiene esa información.

Comprueba que el sistema puede recuperar:

```text
marta.rios
platform-release
18:42 UTC
CHG-4821
```

El modelo no ha sido reentrenado con estos datos. La información se recupera en tiempo de ejecución y se añade al contexto antes de generar la respuesta.

---

# Parte 12 — Publicar la aplicación y crear su API key

Cuando las pruebas manuales funcionen:

1. guarda y publica la Workflow App;
2. abre el acceso API;
3. crea una API key para M05.

No escribas la API key en el repositorio ni en nodos visibles de n8n.

La ejecución seguirá utilizando:

```text
POST https://dify.cloudsecurityinpractice.com/v1/workflows/run
```

---

# Parte 13 — Evolucionar el workflow de n8n

Duplica el workflow funcional de M04.

Renómbralo, por ejemplo:

```text
M05-Triage-IA-RAG
```

El flujo seguirá siendo el mismo hasta la preparación del contexto de IA:

```text
WEBHOOK
  |
  v
VALIDAR
  |
  v
API ACTIVOS
  |
  v
ENRIQUECER
  |
  v
P1-P4
  |
  v
PREPARAR CONTEXTO
  |
  v
DIFY + RAG
  |
  v
RESPONDER
```

Las entradas inválidas, activos desconocidos o severidades no válidas pueden seguir terminando en revisión manual sin llamar a Dify.

---

# Parte 14 — Construir retrieval_query en n8n

En el nodo donde preparas la llamada a Dify conserva `alert_context` y `question` y añade:

```text
retrieval_query
```

Construye un texto equivalente a:

```text
Asset ID: <asset_id>
Asset name: <asset_name>
Category: <category>
Summary: <summary>
Details: <details>
```

No es necesario incluir todos los campos de la alerta. La consulta de recuperación debe contener los términos que ayuden a localizar documentación relacionada con el activo, el síntoma y el tipo de incidencia.

Utiliza como `question`:

```text
Analiza la incidencia utilizando la documentación interna disponible. Distingue hechos, evidencia documental e hipótesis, y recomienda las siguientes comprobaciones técnicas.
```

---

# Parte 15 — Actualizar la llamada a Dify

Crea una nueva credencial de tipo **Header Auth** para la API key de la App M05, por ejemplo:

```text
M05 - Dify API
```

Debe enviar:

```http
Authorization: Bearer <DIFY_APP_API_KEY>
```

En el nodo `HTTP Request` utiliza un body equivalente a:

```json
{
  "inputs": {
    "alert_context": "<alert_context>",
    "retrieval_query": "<retrieval_query>",
    "question": "<question>"
  },
  "response_mode": "blocking",
  "user": "<ID_ALUMNO>-m05"
}
```

Utiliza expresiones de n8n para insertar los tres valores.

Ejecuta una prueba e inspecciona la respuesta de Dify. El valor producido por el nodo `End` aparecerá en los outputs de la ejecución, normalmente bajo una estructura equivalente a:

```text
data.outputs.analysis
```

Comprueba la respuesta real antes de construir expresiones que dependan de ella.

---

# Parte 16 — Probar la solución end-to-end

Utiliza:

```text
materiales/eventos_webhook_m05.json
```

Envía los tres eventos al Webhook del workflow.

Para cada ejecución comprueba:

1. n8n recibe y valida el evento;
2. consulta el activo;
3. calcula P1–P4;
4. construye `alert_context`;
5. construye `retrieval_query`;
6. Dify recupera documentación relevante;
7. el LLM utiliza esa evidencia sin inventar datos;
8. la respuesta HTTP mantiene la prioridad de n8n e incorpora `ai_analysis`.

## M05-E2E-01

La alerta afecta a `srv-web-01` poco después del despliegue documentado de `web-frontend-01`.

El análisis puede relacionar temporalmente ambos hechos como hipótesis, pero no debe afirmar que el despliegue sea la causa.

También debe recuperar el procedimiento web relevante.

## M05-E2E-02

La alerta de base de datos debe recuperar `DB-HEALTH-01` y recomendar las comprobaciones documentadas sin inventar métricas.

## M05-E2E-03

La alerta de backup debe utilizar el documento de operación de backups y recomendar revisar el job, el espacio del destino y el estado de la base de datos.

---

# Parte 17 — Diagnosticar una respuesta incorrecta

Si el resultado no es correcto, revisa en este orden:

```text
1. retrieval_query
2. documentos/chunks recuperados
3. contexto recibido por el LLM
4. instrucciones del prompt
5. respuesta generada
```

Esto permite distinguir dos problemas diferentes:

```text
retrieval no encontró la evidencia correcta
```

frente a:

```text
la evidencia era correcta pero el LLM la interpretó mal
```

---

# Resultado del laboratorio

Al terminar tendrás una solución equivalente a:

```text
SISTEMA EMISOR
      |
      v
WEBHOOK n8n
      |
      v
API ACTIVOS
      |
      v
PRIORIDAD P1-P4
      |
      v
DIFY
  |          |
  |          v
  |     KNOWLEDGE BASE
  |          |
  `------> RETRIEVAL
             |
             v
            LLM
             |
             v
   ANÁLISIS FUNDAMENTADO
             |
             v
      RESPUESTA HTTP
```

La automatización ya combina:

```text
reglas deterministas
+
contexto del incidente
+
conocimiento documental privado
+
IA generativa
```

## Limitación actual

RAG puede consultar documentación, pero la documentación es conocimiento almacenado.

Todavía no permite que el sistema investigue activamente preguntas como:

```text
¿Está el servicio disponible ahora mismo?
¿Hay una incidencia abierta para este activo?
¿Qué cambios recientes devuelve el sistema operativo en este momento?
¿Qué otras alertas activas existen?
```

En M06 añadiremos un **Agent con herramientas MCP** para que la solución pueda decidir qué fuentes operativas consultar durante una investigación.
