# Laboratorio 6 — Investigación agentic con MCP en Dify

## Contexto

En M05 la automatización ya puede combinar:

- una alerta recibida por n8n;
- datos del activo obtenidos mediante API REST;
- prioridad P1–P4 calculada de forma determinista;
- documentación interna recuperada mediante RAG.

Pero sigue existiendo una limitación importante: **RAG consulta documentación almacenada, no el estado operativo actual**.

Si queremos responder preguntas como:

> ¿Está degradado ahora el servicio?
>
> ¿Hay alguna incidencia abierta?
>
> ¿Se ha desplegado algo recientemente?
>
> ¿Existen otras alertas recientes del mismo activo?

necesitamos consultar sistemas vivos.

En este laboratorio añadiremos un **Agent de Dify** con acceso a las tools read-only del servidor MCP `Telvora Ops Hub`.

La diferencia fundamental es que no programaremos una secuencia fija de llamadas. El Agent decidirá **qué herramienta necesita, con qué argumentos y si debe realizar otra consulta después de ver el resultado**.

> Todos los datos son ficticios y se utilizan exclusivamente con fines formativos.

---

## Objetivos

Al finalizar deberías ser capaz de:

- explicar la diferencia entre RAG y herramientas operativas;
- configurar un nodo Agent en una Workflow App de Dify;
- utilizar una estrategia de Function Calling;
- asignar herramientas MCP a un Agent;
- proporcionar contexto RAG al Agent;
- observar qué tool elige el Agent y con qué argumentos;
- comprobar cómo una respuesta de una tool modifica la siguiente decisión;
- reconocer cuándo el Agent deja de investigar;
- diferenciar hechos documentales, hechos operativos e hipótesis;
- integrar el Workflow agentic con el flujo de n8n construido en módulos anteriores;
- mantener la prioridad P1–P4 fuera del control del Agent.

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

En el Workspace de Dify ya están configurados por el instructor:

```text
MCP Server: Telvora Ops Hub
Agent Strategy: Dify Agent Strategies
```

El servidor MCP utiliza:

```text
https://api.cloudsecurityinpractice.com/mcp
```

No necesitas conocer ni configurar el token MCP.

**No modifiques ni elimines la integración compartida.**

El modelo utilizado será el indicado por el instructor. En esta edición:

```text
openai.gpt-5.6-luna
```

---

## Material proporcionado

En `materiales/` encontrarás:

- `contrato_mcp_telvora_ops_hub.md`;
- `prompt_agent_mcp.md`;
- `casos_agent.json`;
- `eventos_webhook_m06.json`;
- `comportamiento_esperado.md`.

M06 reutiliza la Knowledge Base y los cuatro documentos de M05.

---

# Parte 1 — Duplicar la Workflow App de M05

No empieces desde cero.

Duplica tu aplicación M05 y renómbrala:

```text
<ID_ALUMNO>-M06-Agent-Ops
```

Ejemplo:

```text
AL01-M06-Agent-Ops
```

La aplicación de M05 tenía conceptualmente:

```text
START
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

En M06 evolucionará a:

```text
START
  |
  v
KNOWLEDGE RETRIEVAL
  |
  v
AGENT + MCP
  |
  v
END
```

El Agent será responsable de combinar contexto documental con información operativa actual.

---

# Parte 2 — Añadir `asset_id` como input

En el nodo **Start** conserva las variables de M05:

```text
alert_context
retrieval_query
question
```

y añade una nueva variable obligatoria:

```text
asset_id
```

Trabajaremos por tanto con cuatro inputs:

```text
alert_context   → contexto técnico ya validado por n8n
asset_id        → activo sobre el que puede consultar tools
retrieval_query → búsqueda sobre documentación interna
question        → objetivo concreto de la investigación
```

No obligaremos al Agent a extraer el identificador del activo desde un bloque de texto si n8n ya conoce ese dato.

---

# Parte 3 — Mantener Knowledge Retrieval

Conserva el nodo de recuperación de M05.

Nombre recomendado:

```text
01 - Recuperar conocimiento
```

Selecciona únicamente tu Knowledge Base:

```text
<ID_ALUMNO>-M05-KB-Ops
```

La query seguirá siendo:

```text
retrieval_query
```

RAG seguirá respondiendo a preguntas del tipo:

```text
¿Qué procedimiento existe?
¿Qué dice el runbook?
¿Qué información está documentada?
```

No lo confundas con estado operativo actual.

---

# Parte 4 — Sustituir el LLM por un Agent

Elimina o desconecta el LLM final de M05 y añade un nodo:

```text
02 - Investigar con Ops Hub
```

Tipo:

```text
Agent
```

Selecciona la estrategia agéntica disponible basada en:

```text
FunctionCalling
```

Selecciona el modelo indicado por el instructor.

Si aparecen estas opciones, utiliza:

```text
Memory: OFF
Maximum Iterations: 5
```

No necesitamos memoria conversacional para investigar una única incidencia.

---

# Parte 5 — Asignar las tools MCP

En la lista de herramientas del Agent añade únicamente las cuatro tools de:

```text
Telvora Ops Hub
```

Tools disponibles:

```text
get_service_health
get_recent_changes
get_recent_alerts
get_open_incidents
```

Consulta:

```text
materiales/contrato_mcp_telvora_ops_hub.md
```

para revisar sus parámetros.

No añadas herramientas de otros plugins o proveedores.

## Importante

La presencia de cuatro tools **no significa que el Agent deba utilizar siempre las cuatro**.

Queremos observar una decisión real:

```text
objetivo
  ↓
¿qué información necesito?
  ↓
tool adecuada
  ↓
resultado
  ↓
¿tengo suficiente evidencia?
  ├─ sí → responder
  └─ no → seleccionar otra tool
```

---

# Parte 6 — Configurar las instrucciones del Agent

Abre:

```text
materiales/prompt_agent_mcp.md
```

y utiliza su contenido como **Instruction** del Agent.

Las instrucciones establecen, entre otras cosas:

- no recalcular P1–P4;
- utilizar RAG como fuente documental;
- utilizar MCP como fuente de estado vivo;
- no inventar resultados;
- no llamar a todas las tools por defecto;
- detener la investigación cuando haya contexto suficiente;
- no ejecutar acciones de remediación.

---

# Parte 7 — Pasar el contexto RAG al Agent

En el campo **Context** del Agent selecciona mediante el selector de variables la salida de:

```text
01 - Recuperar conocimiento
```

De esta forma el Agent dispone de:

```text
documentación recuperada
+
tools de estado vivo
```

No copies manualmente los chunks dentro del prompt.

---

# Parte 8 — Construir la Query del Agent

En la Query utiliza las variables del nodo Start para construir algo equivalente a:

```text
ACTIVO
<asset_id>

CONTEXTO DE ALERTA
<alert_context>

OBJETIVO DE INVESTIGACIÓN
<question>
```

Inserta las variables mediante el selector de Dify.

No añadas instrucciones como:

```text
consulta las cuatro tools
```

porque eso eliminaría precisamente la decisión que queremos observar.

---

# Parte 9 — Configurar End

Conecta el Agent a **End**.

Crea una salida:

```text
analysis
```

Para asignarla, abre el selector de variables y selecciona la **salida textual final generada por el nodo Agent**.

> No escribas manualmente una referencia como `Agent.text`. El nombre exacto de la salida visible puede variar según la versión del nodo o del plugin. Selecciona la respuesta final disponible desde la interfaz.

No utilizaremos Structured Output en este módulo.

Primero queremos comprender el comportamiento del Agent y sus llamadas a tools.

---

# Parte 10 — Primera prueba: observar una investigación

Abre:

```text
materiales/casos_agent.json
```

Ejecuta `AG-01`.

Después abre el detalle de la ejecución del Agent y localiza la información de la estrategia agéntica / rounds / tool calls disponible en tu interfaz.

Debes poder identificar al menos:

```text
tool seleccionada
argumentos enviados
resultado recibido
siguiente decisión
```

Para `AG-01` es razonable que el Agent quiera conocer:

- el health actual;
- cambios recientes.

Pero no estamos evaluando una secuencia textual exacta.

---

# Parte 11 — Hacer que el Agent tome decisiones diferentes

Ejecuta:

```text
AG-02
AG-03
AG-04
AG-05
```

Antes de cada ejecución predice qué tools podrían ser útiles.

Después compara tu predicción con el comportamiento real.

Utiliza:

```text
materiales/comportamiento_esperado.md
```

como referencia.

## Observación especialmente importante — AG-03

La pregunta indica:

> Comprueba primero si existe una incidencia abierta relacionada.

Por tanto, esperamos que la primera tool elegida sea:

```text
get_open_incidents
```

Después el Agent debe decidir qué hacer a partir del resultado.

## Observación especialmente importante — AG-04

No buscamos que utilice todas las tools.

Un buen resultado puede terminar diciendo que la información disponible sigue siendo insuficiente.

---

# Parte 12 — Comparar RAG y MCP

Durante las pruebas identifica ejemplos de ambos tipos de información.

## RAG

Ejemplos:

```text
WEB-AVAIL-01
DB-HEALTH-01
política de reinicio
inventario documentado
```

## MCP

Ejemplos:

```text
health actual
incidentes abiertos actuales
alertas recientes
cambios recientes
```

Responde:

- ¿por qué no sería correcto utilizar un documento de hace semanas para afirmar que un servicio está `healthy` ahora?
- ¿por qué tampoco deberíamos utilizar una tool de estado vivo como sustituto de un procedimiento corporativo?

---

# Parte 13 — Comprobar el límite de actuación

Ejecuta `AG-05`.

El operador pregunta si debe reiniciar el servidor.

El Agent puede:

- consultar el runbook mediante el contexto RAG;
- consultar health u otras evidencias mediante MCP;
- recomendar comprobaciones;
- indicar que un reinicio podría requerir autorización.

Pero no puede:

```text
reiniciar
hacer rollback
cambiar configuración
```

porque esas tools no existen.

La ausencia de una tool también forma parte del diseño de un agente seguro.

---

# Parte 14 — Publicar la Workflow App

Cuando los cinco casos funcionen razonablemente:

1. guarda los cambios;
2. publica la aplicación;
3. crea una API key para la aplicación M06.

No subas la API key al repositorio ni la escribas directamente en un nodo.

---

# Parte 15 — Evolucionar el workflow de n8n

Duplica el workflow funcional de M05.

Nombre recomendado:

```text
M06-Triage-Agent-MCP
```

Mantén las partes ya construidas:

```text
Webhook
→ normalización
→ validación
→ API de activos
→ enriquecimiento
→ P1–P4
```

Las ramas de entrada inválida, activo desconocido o severity no válida deben seguir terminando sin llamar a Dify.

Para las alertas válidas conserva:

```text
alert_context
retrieval_query
question
```

y añade:

```text
asset_id
```

como input explícito para Dify.

---

# Parte 16 — Preparar el objetivo de investigación

En el nodo donde preparas los datos para Dify utiliza una pregunta general como:

```text
Investiga la situación actual del activo. Utiliza la documentación interna y las herramientas disponibles solo cuando aporten evidencia. Distingue hechos documentales, hechos operativos e hipótesis, e indica qué comprobaciones deberían hacerse a continuación.
```

Mantén `retrieval_query` orientada a encontrar documentación útil a partir de:

```text
asset_id
summary
details
category
```

Por ejemplo, conceptualmente:

```text
srv-web-01 HTTP 5xx availability procedimiento runbook
```

---

# Parte 17 — Actualizar la llamada a Dify

Utiliza la API key de la aplicación M06 mediante una credencial de n8n.

El endpoint sigue siendo:

```text
POST https://dify.cloudsecurityinpractice.com/v1/workflows/run
```

El body debe ser equivalente a:

```json
{
  "inputs": {
    "alert_context": "<alert_context>",
    "asset_id": "<asset_id>",
    "retrieval_query": "<retrieval_query>",
    "question": "<question>"
  },
  "response_mode": "blocking",
  "user": "<ID_ALUMNO>-m06"
}
```

Utiliza expresiones de n8n para insertar los cuatro valores.

Ejecuta primero una prueba y observa la respuesta completa de Dify.

La salida del nodo `End` aparecerá dentro de los outputs del Workflow. Localiza:

```text
analysis
```

En una respuesta típica estará dentro de una estructura equivalente a:

```text
data.outputs.analysis
```

Compruébalo en tu ejecución antes de construir la expresión definitiva.

---

# Parte 18 — Devolver el análisis agentic

Modifica la respuesta final del Webhook para incluir:

```text
priority
owner_team
requires_human_review
agent_analysis
```

Ejemplo conceptual:

```json
{
  "accepted": true,
  "alert_id": "ALT-6101",
  "priority": "P1",
  "owner_team": "WebOps",
  "requires_human_review": true,
  "agent_analysis": "RESUMEN ..."
}
```

Recuerda:

```text
priority       → n8n
agent_analysis → Dify Agent
```

El Agent no debe modificar la prioridad calculada por el workflow.

---

# Parte 19 — Prueba end-to-end

Utiliza:

```text
materiales/eventos_webhook_m06.json
```

Ejecuta los cuatro casos.

Para cada uno comprueba:

1. que n8n recibe y valida la alerta;
2. que consulta el activo;
3. que calcula P1–P4;
4. que envía los cuatro inputs a Dify;
5. que Knowledge Retrieval obtiene contexto documental;
6. que el Agent selecciona una o más tools MCP cuando las necesita;
7. que la respuesta final diferencia hechos de hipótesis;
8. que n8n devuelve la prioridad original junto al análisis agentic.

Para al menos **dos casos**, abre después la ejecución correspondiente en Dify y compara la secuencia de tools utilizada.

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
VALIDAR + API ACTIVO + P1-P4
      |
      v
DIFY WORKFLOW
      |
      +--> RAG / documentación
      |
      `--> AGENT
             |
             +--> get_service_health
             +--> get_recent_changes
             +--> get_recent_alerts
             `--> get_open_incidents
                    |
                    v
             investigación
                    |
                    v
              END / analysis
                    |
                    v
RESPUESTA DE n8n
```

La automatización ya no se limita a recuperar documentación y generar texto. El Agent puede **decidir qué información operativa necesita consultar para investigar una incidencia**.

## Limitación actual

El Agent puede investigar y recomendar, pero n8n todavía trata su resultado principalmente como texto.

Aún no hemos construido una política que permita distinguir, por ejemplo:

```text
recomendación informativa
→ continuar automáticamente

acción sensible
→ solicitar aprobación humana
```

Esa será la siguiente evolución de la solución.
