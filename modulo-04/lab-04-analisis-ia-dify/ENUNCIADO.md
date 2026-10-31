# Laboratorio 4 — Análisis de alertas con IA en Dify

## Contexto

En M03 tu automatización ya puede:

- recibir una alerta mediante Webhook;
- validar el payload;
- consultar el catálogo de activos mediante API REST;
- enriquecer la alerta;
- calcular una prioridad P1–P4;
- devolver una respuesta al sistema emisor.

Sin embargo, sigue existiendo una limitación importante: el workflow puede aplicar reglas, pero no sabe interpretar bien información técnica ambigua.

Por ejemplo, puede saber que una alerta es `P1`, pero no responder bien a preguntas como:

> ¿Qué puede estar ocurriendo y qué debería comprobar primero el equipo técnico?

En este laboratorio añadiremos **IA generativa mediante Dify** para realizar ese análisis.

La prioridad seguirá siendo responsabilidad de n8n. El LLM no debe sustituir las reglas deterministas ni ejecutar acciones sobre sistemas.

> Todos los datos utilizados son ficticios y se emplean exclusivamente con fines formativos.

---

## Objetivos

Al finalizar deberías ser capaz de:

- crear una Workflow App en Dify;
- pasar contexto técnico a un nodo LLM;
- observar las limitaciones de un prompt demasiado genérico;
- separar hechos de hipótesis;
- exigir que el modelo reconozca información insuficiente;
- impedir que el modelo afirme haber ejecutado acciones;
- generar una respuesta con formato consistente;
- publicar una Workflow App;
- invocar la API de Dify desde n8n;
- incorporar el análisis del LLM a la respuesta final de la automatización.

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

El proveedor de modelos ya está configurado en Dify.

Utiliza el modelo indicado por el instructor. En esta edición el modelo configurado para la práctica es:

```text
openai.gpt-5.6-luna
```

No modifiques proveedores, credenciales globales ni aplicaciones de otros alumnos.

### Nombres en el Workspace compartido

Utiliza siempre tu identificador como prefijo.

Ejemplo:

```text
AL01-M04-Analisis-Incidentes
```

---

## Material proporcionado

En `materiales/` encontrarás:

- `prompt_base.md`;
- `casos_dify.json`;
- `eventos_webhook_m04.json`;
- `comportamiento_esperado.md`.

---

# Parte 1 — Crear la Workflow App en Dify

Crea una nueva aplicación de tipo **Workflow**.

Nombre recomendado:

```text
<ID_ALUMNO>-M04-Analisis-Incidentes
```

Ejemplo:

```text
AL01-M04-Analisis-Incidentes
```

La primera versión tendrá esta estructura:

```text
START
  |
  v
 LLM
  |
  v
 END
```

No añadiremos todavía Knowledge Retrieval, RAG ni Agent.

---

# Parte 2 — Configurar las entradas

En el nodo **Start** crea dos variables obligatorias:

```text
alert_context
question
```

Utiliza tipos de texto. Para `alert_context` resulta conveniente un campo de texto largo/paragraph.

Su significado será:

```text
alert_context = contexto técnico ya obtenido por la automatización
question      = objetivo de análisis que pedimos al modelo
```

Esta separación será útil más adelante cuando n8n invoque el workflow mediante API.

---

# Parte 3 — Añadir el LLM

Añade un nodo **LLM** y renómbralo:

```text
01 - Analizar alerta
```

Selecciona el modelo indicado por el instructor.

No necesitamos creatividad. Si la interfaz permite configurar parámetros como `Temperature` o `Top P`, utiliza los valores recomendados por el instructor o déjalos sin personalizar.

Abre:

```text
materiales/prompt_base.md
```

y utiliza ese prompt como primera versión.

A continuación añade al prompt el contexto y la pregunta usando el selector de variables de Dify:

```text
CONTEXTO DE ALERTA:
<alert_context>

OBJETIVO DEL ANÁLISIS:
<question>
```

No escribas manualmente valores de un caso concreto dentro del prompt.

---

# Parte 4 — Configurar la salida

Añade un nodo **End**.

Crea una salida:

```text
analysis
```

asignada a la salida de texto del nodo LLM.

La aplicación debe quedar conceptualmente así:

```text
alert_context ─┐
               ├─> LLM ─> analysis
question ──────┘
```

---

# Parte 5 — Ejecutar el primer caso

Abre:

```text
materiales/casos_dify.json
```

Ejecuta `IA-01`.

En `alert_context` introduce el objeto de contexto del caso y en `question` utiliza la pregunta indicada.

Lee la respuesta completa.

No modifiques todavía el prompt.

Analiza:

- ¿qué considera hechos?
- ¿qué presenta como hipótesis?
- ¿afirma una causa con demasiada seguridad?
- ¿propone comprobaciones útiles?
- ¿el formato sería fácil de leer para un operador?

---

# Parte 6 — Comprobar qué ocurre con información insuficiente

Ejecuta ahora:

```text
IA-02
IA-03
```

Estos casos están diseñados para que el contexto no permita confirmar una causa raíz.

Comprueba si el modelo:

- inventa métricas;
- inventa componentes;
- presenta una posibilidad como certeza;
- reconoce explícitamente qué información falta.

Una respuesta técnicamente plausible no es necesariamente una respuesta respaldada por la evidencia.

---

# Parte 7 — Comprobar los límites de actuación

Ejecuta:

```text
IA-04
```

La pregunta solicita reiniciar un servidor.

El workflow de Dify no dispone de ninguna herramienta de remediación y el modelo no debe fingir que puede realizarla.

Después ejecuta:

```text
IA-05
```

La pregunta solicita datos sobre el último despliegue que no aparecen en el contexto.

El comportamiento correcto consiste en reconocer la falta de información, no en adivinar.

---

# Parte 8 — Mejorar el prompt

Ahora modifica el prompt para imponer estas reglas:

```text
- Utiliza únicamente la información disponible en alert_context.
- Considera la prioridad recibida como un dato ya calculado por n8n; no la recalcules ni la sustituyas.
- Separa siempre hechos observados de hipótesis.
- No presentes una hipótesis como causa confirmada.
- No inventes métricas, personas, cambios, eventos ni resultados.
- Si falta información, indícalo explícitamente.
- Propón comprobaciones técnicas no destructivas.
- No afirmes haber ejecutado acciones.
- Reinicios, rollbacks, bloqueos o cambios operativos requieren autorización externa al modelo.
```

Además, exige siempre este formato:

```text
RESUMEN

HECHOS OBSERVADOS
- ...

HIPÓTESIS
- ...

COMPROBACIONES RECOMENDADAS
- ...

INFORMACIÓN FALTANTE
- ...

REVISIÓN HUMANA
- ...
```

No necesitas utilizar JSON ni Structured Output en este módulo.

El objetivo es obtener una respuesta útil para una persona y suficientemente consistente para incorporarla al resultado de la automatización.

---

# Parte 9 — Repetir las pruebas

Vuelve a ejecutar los cinco casos.

Utiliza `materiales/comportamiento_esperado.md` para comprobar el comportamiento.

No compares palabras exactas. Comprueba que:

- no se inventan hechos;
- la prioridad permanece sin cambios;
- las hipótesis aparecen como hipótesis;
- la información ausente se reconoce;
- una petición de acción no produce una falsa confirmación de ejecución;
- las comprobaciones propuestas son razonables y no destructivas.

Cuando el resultado sea satisfactorio, guarda y publica la aplicación.

---

# Parte 10 — Preparar acceso API a Dify

Abre la sección de acceso API de tu aplicación y crea una API key para esta práctica.

No compartas ni subas la clave al repositorio.

La API de la instancia del curso utiliza como base:

```text
https://dify.cloudsecurityinpractice.com/v1
```

Para ejecutar una Workflow App utilizaremos:

```text
POST /workflows/run
```

con autenticación:

```http
Authorization: Bearer <DIFY_APP_API_KEY>
Content-Type: application/json
```

El body tendrá esta forma:

```json
{
  "inputs": {
    "alert_context": "<contexto construido por n8n>",
    "question": "Analiza la alerta y recomienda qué debería comprobar el equipo técnico."
  },
  "response_mode": "blocking",
  "user": "alumno01-m04"
}
```

El valor de `user` debe identificar tu ejecución. Sustituye `alumno01` por tu identificador.

---

# Parte 11 — Evolucionar el workflow de n8n

Duplica el workflow funcional de M03.

Renómbralo, por ejemplo:

```text
M04-Triage-IA
```

Mantén intacta la versión anterior.

El nuevo flujo será conceptualmente:

```text
WEBHOOK
  |
  v
VALIDAR
  |
  v
API DE ACTIVOS
  |
  v
ENRIQUECER
  |
  v
P1–P4
  |
  v
PREPARAR CONTEXTO IA
  |
  v
DIFY
  |
  v
RESPONDER
```

Las ramas de entrada inválida, activo desconocido o severity no válida pueden seguir terminando en revisión manual sin llamar al LLM.

---

# Parte 12 — Preparar el contexto para Dify

Después de las rutas válidas P1–P4 añade un nodo `Edit Fields (Set)`:

```text
08 - Preparar contexto IA
```

Construye un campo de texto llamado:

```text
alert_context
```

con información equivalente a:

```text
Alert ID: <alert_id>
Asset ID: <asset_id>
Asset name: <asset_name>
Severity: <severity>
Priority: <priority>
Environment: <environment>
Criticality: <criticality>
Owner team: <owner_team>
Summary: <summary>
Details: <details>
```

No recalcules aquí la prioridad. Utiliza la que ya produjo el workflow.

Si alguno de estos campos no está disponible en el item actual, recupéralo del nodo anterior correspondiente mediante una expresión de n8n.

Añade también:

```text
question = Analiza la alerta y recomienda qué debería comprobar primero el equipo técnico.
```

---

# Parte 13 — Crear la credencial de Dify en n8n

Crea una credencial de tipo **Header Auth** con un nombre como:

```text
M04 - Dify API
```

Debe enviar:

```http
Authorization: Bearer <DIFY_APP_API_KEY>
```

No escribas la API key directamente en el nodo ni en un `Edit Fields`.

---

# Parte 14 — Invocar Dify desde n8n

Añade un nodo **HTTP Request**:

```text
09 - Analizar en Dify
```

Configura:

```text
Method: POST
URL: https://dify.cloudsecurityinpractice.com/v1/workflows/run
Authentication: M04 - Dify API
Body Content Type: JSON
```

El body debe ser equivalente a:

```json
{
  "inputs": {
    "alert_context": "<alert_context>",
    "question": "<question>"
  },
  "response_mode": "blocking",
  "user": "<ID_ALUMNO>-m04"
}
```

Utiliza expresiones para insertar los valores de `alert_context` y `question`.

Ejecuta una prueba e inspecciona la respuesta completa de Dify.

En una ejecución correcta, el resultado del nodo `End` aparecerá dentro de la respuesta de la Workflow App. Localiza el valor de:

```text
analysis
```

antes de continuar.

---

# Parte 15 — Incorporar el análisis a la respuesta del Webhook

Modifica la respuesta final del workflow para incluir el análisis de IA junto con la clasificación determinista.

Debe quedar equivalente a:

```json
{
  "accepted": true,
  "alert_id": "ALT-4101",
  "priority": "P1",
  "owner_team": "WebOps",
  "requires_human_review": true,
  "ai_analysis": "RESUMEN ..."
}
```

La prioridad procede de n8n.

`ai_analysis` procede de Dify.

Son responsabilidades diferentes.

---

# Parte 16 — Prueba end-to-end

Utiliza:

```text
materiales/eventos_webhook_m04.json
```

Envía los tres casos al Webhook de tu workflow.

Para cada ejecución comprueba:

1. que n8n recibe el evento;
2. que consulta correctamente el activo;
3. que calcula la prioridad;
4. que construye `alert_context`;
5. que llama a Dify;
6. que Dify devuelve un análisis coherente;
7. que la respuesta HTTP contiene tanto la prioridad como el análisis.

En especial:

- `M04-E2E-01` puede sugerir una relación con el despliegue, pero no debe afirmar una causa confirmada;
- `M04-E2E-02` debe reconocer la falta de métricas para determinar una causa raíz;
- `M04-E2E-03` debe reconocer que no puede identificar exactamente el componente que provoca el retraso.

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
VALIDAR
      |
      v
API ACTIVOS
      |
      v
ENRIQUECER
      |
      v
PRIORIDAD P1–P4
      |
      v
DIFY / LLM
      |
      v
ANÁLISIS TÉCNICO
      |
      v
RESPUESTA HTTP
```

La automatización ya combina dos mecanismos distintos:

```text
reglas deterministas → prioridad
IA generativa        → interpretación y comprobaciones sugeridas
```

## Limitación actual

El LLM solo conoce la información que le enviamos en `alert_context`.

No conoce los procedimientos internos de Telvora, runbooks, políticas de backup ni documentación de cambios.

En M05 añadiremos **RAG** para que el análisis pueda utilizar conocimiento interno de la organización sin incluirlo manualmente en cada alerta.
