# Laboratorio 5 — Knowledge Base y RAG en Dify

## Contexto

En M04 construiste un asistente técnico capaz de interpretar el contexto de una alerta, separar hechos de hipótesis, reconocer incertidumbre y evitar fingir acciones.

Sin embargo, el asistente tenía una limitación deliberada: **solo conocía la información incluida directamente en `alert_context`**.

Por ejemplo, si preguntábamos quién había realizado el último despliegue o cuál era el procedimiento interno de escalado, la respuesta correcta era indicar que esa información no estaba disponible.

En este laboratorio añadiremos una fuente documental mediante **Retrieval-Augmented Generation (RAG)**.

La idea será:

```text
PREGUNTA
   |
   v
RECUPERACIÓN DE CONOCIMIENTO
   |
   v
FRAGMENTOS RELEVANTES
   |
   v
LLM
   |
   v
RESPUESTA FUNDAMENTADA
```

El objetivo no es que el modelo "sepa más" de forma genérica, sino que pueda responder utilizando **evidencia documental concreta y trazable**.

> Todos los datos y nombres incluidos en los documentos son ficticios y se utilizan exclusivamente para formación.

---

## Objetivos de aprendizaje

Al finalizar deberías ser capaz de:

- explicar qué problema resuelve RAG;
- diferenciar conocimiento paramétrico del modelo y conocimiento recuperado;
- crear una Knowledge Base privada en Dify;
- cargar documentos y procesarlos mediante un pipeline;
- comprender embeddings y búsqueda semántica;
- comprender Parent-child chunking;
- configurar Hybrid Search y reranking;
- probar retrieval antes de conectar un LLM;
- inspeccionar documentos, chunks y scores recuperados;
- añadir un nodo Knowledge Retrieval a un Chatflow;
- utilizar el contexto recuperado para fundamentar respuestas;
- evitar que el modelo complete información no documentada;
- comprobar el comportamiento ante preguntas sin respuesta;
- distinguir un fallo de retrieval de un fallo de generación;
- explicar qué información debería conservarse para auditar una respuesta RAG.

---

## Prerrequisitos

Necesitas:

- acceso a `https://dify.cloudsecurityinpractice.com/`;
- tu cuenta de alumno;
- haber completado o revisado M04;
- conocer tu identificador de alumno, por ejemplo `AL01`;
- los archivos incluidos en `materiales/`.

El instructor ya ha configurado los proveedores de modelos. **No modifiques credenciales, Model Providers ni plugins globales.**

---

# Parte 1 — Comprender qué vamos a añadir

M04 trabajaba así:

```text
alert_context + pregunta
          |
          v
         LLM
          |
          v
      respuesta
```

M05 añade una nueva etapa:

```text
pregunta
   |
   v
Knowledge Retrieval
   |
   | fragmentos recuperados
   v
LLM <---- alert_context
   |
   v
respuesta
```

La diferencia es importante.

El LLM sigue generando la respuesta, pero ahora recibe información seleccionada desde documentos externos.

---

# Parte 2 — Crear una Knowledge Base privada

Ve a **Knowledge** y crea una nueva Knowledge Base.

Nombre:

```text
<ID_ALUMNO>-M05-KB-Ops
```

Ejemplo:

```text
AL01-M05-KB-Ops
```

## Privacidad

Mantén la Knowledge Base como:

```text
Only me / privada
```

si aparece un selector de permisos.

No la compartas con todos los miembros del Workspace.

En nuestra infraestructura las Knowledge Bases privadas permiten que los alumnos trabajen con sus propios documentos sin interferirse entre sí.

> El instructor puede no ver directamente una Knowledge Base marcada como privada. Por eso la entrega incluye capturas y resultados de retrieval.

---

# Parte 3 — Utilizar el pipeline Parent-child-HQ

Selecciona el template:

```text
Parent-child-HQ
```

Este pipeline está pensado para producir chunks pequeños para búsqueda y conservar un fragmento padre con más contexto para la respuesta.

## Ajuste del template en el entorno del curso

Nuestra instalación contiene únicamente los componentes necesarios para este laboratorio.

Si el template muestra referencias a fuentes o conectores que no están instalados, elimina esas referencias antes de publicar el pipeline.

La configuración esperada es conceptualmente:

```text
Datasource / Variable Aggregator inicial:
- File

Extracción / Variable Aggregator posterior:
- Dify Extractor -> text
- Doc Extractor -> text, si aparece como alternativa válida
```

No deben quedar referencias huérfanas a conectores como Notion, Google Drive, Firecrawl, Jina u otros componentes no utilizados en esta práctica.

---

# Parte 4 — Configurar el chunking Parent-child

Utiliza:

```text
Parent Mode: paragraph
Parent Delimiter: \n\n
Maximum Parent Length: 1024
Child Delimiter: \n
Maximum Child Length: 256
Replace consecutive spaces/newlines/tabs: ON
Delete URLs/emails: OFF
```

## Preguntas

- ¿por qué puede ser útil buscar utilizando fragmentos pequeños?
- ¿por qué puede ser útil entregar al LLM un fragmento padre más amplio?
- ¿qué problema aparecería si cada chunk fuese un documento completo muy grande?

---

# Parte 5 — Configurar la indexación

Configura la Knowledge Base como:

```text
Indexing / Quality: High Quality
Embedding model: amazon.titan-embed-text-v2:0
Automatic summary: OFF
```

El modelo de embeddings transforma texto en representaciones vectoriales que permiten comparar similitud semántica.

No necesitas configurar AWS ni credenciales: el proveedor ya está preparado por el instructor.

---

# Parte 6 — Configurar retrieval

Utiliza:

```text
Retrieval: Hybrid Search
Rerank model: cohere.rerank-v3-5:0
Top K: 3
Score Threshold: OFF
```

## Qué queremos conseguir

**Hybrid Search** permite combinar señales semánticas con coincidencias textuales.

El **reranker** recibe candidatos recuperados y vuelve a ordenarlos según su relevancia para la consulta.

`Top K = 3` limita el número de resultados principales que llegarán a la siguiente fase.

---

# Parte 7 — Cargar los documentos

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

Espera a que el procesamiento termine correctamente.

Comprueba que los documentos están indexados y disponibles para retrieval.

---

# Parte 8 — Inspeccionar los chunks

Antes de utilizar un LLM, abre alguno de los documentos y revisa cómo ha sido segmentado.

Localiza ejemplos de:

```text
parent chunk
child chunk
```

Comprueba especialmente que no se han unido secciones completamente diferentes en un único bloque y que los fragmentos conservan suficiente contexto para ser interpretables.

## Pregunta

¿Por qué probaríamos el retrieval antes de construir el Chatflow?

---

# Parte 9 — Probar retrieval directamente

Utiliza la función de prueba de recuperación de la Knowledge Base.

Empieza por estas consultas:

```text
¿Cómo se llama el servidor web de producción?
```

```text
¿Qué hay que comprobar primero si el servicio web deja de responder?
```

```text
¿Cuál es el identificador del procedimiento de escalado de incidencias críticas?
```

```text
¿A qué hora se ejecuta el backup diario de telvora-prod-db?
```

Para cada consulta inspecciona:

- documento recuperado;
- fragmento;
- posición;
- score, si aparece;
- si el fragmento contiene realmente la evidencia necesaria.

---

# Parte 10 — Diferenciar búsqueda semántica y coincidencia exacta

Compara dos tipos de consulta.

## Consulta semántica

```text
¿Qué pasos recomienda Operaciones cuando el frontal no responde?
```

El documento puede no contener exactamente la palabra `frontal`, pero debería recuperar el procedimiento del servicio web si la representación semántica es adecuada.

## Consulta con identificador exacto

```text
OPS-RAG-2026-TEST
```

Aquí la coincidencia textual exacta es especialmente valiosa.

## Pregunta

¿Por qué una combinación híbrida puede ser más robusta que depender únicamente de similitud vectorial?

---

# Parte 11 — Crear la App M05

Crea un nuevo **Chatflow** o duplica tu propia App de M04.

Nombre:

```text
<ID_ALUMNO>-M05-Asistente-RAG
```

Ejemplo:

```text
AL01-M05-Asistente-RAG
```

Si duplicas M04, trabaja únicamente sobre la copia M05.

No edites Apps de otros alumnos.

Para concentrarnos en RAG, no es necesario mantener Structured Output durante las primeras pruebas. Puedes dejar la salida final como texto y centrarte en retrieval, grounding y fuentes.

---

# Parte 12 — Configurar User Input

Mantén o crea:

```text
alert_context
```

como campo de texto largo.

La pregunta del usuario seguirá disponible mediante la variable de consulta del Chatflow (`userinput.query` en la interfaz actual).

Conceptualmente:

```text
alert_context = contexto técnico del caso
query         = pregunta del operador
```

---

# Parte 13 — Añadir Knowledge Retrieval

Entre User Input y el LLM añade:

```text
Knowledge Retrieval
```

Renómbralo:

```text
01 - Recuperar conocimiento
```

Selecciona **únicamente tu Knowledge Base**:

```text
<ID_ALUMNO>-M05-KB-Ops
```

Utiliza como query la pregunta del usuario:

```text
userinput.query
```

No selecciones Knowledge Bases de otros alumnos aunque aparezcan disponibles.

---

# Parte 14 — Configurar el LLM

Añade o reutiliza un nodo LLM:

```text
02 - Responder con contexto RAG
```

Modelo:

```text
openai.gpt-5.6-luna
```

Mantén:

```text
Temperature: OFF
Top P: OFF
Reasoning Effort: none/off si aparece
```

En la sección **Context** del LLM selecciona la salida de:

```text
01 - Recuperar conocimiento
```

Utiliza el selector de variables de Dify para insertar el contexto recuperado. No escribas manualmente identificadores internos si la interfaz permite seleccionarlos.

---

# Parte 15 — Configurar el prompt de grounding

Abre:

```text
materiales/prompt_rag.md
```

Utilízalo como base del prompt de sistema.

El principio fundamental es:

```text
Los hechos operativos concretos deben estar respaldados por el contexto recuperado.
```

Si la información no aparece en la documentación recuperada, el asistente debe decirlo claramente.

No debe completar la respuesta utilizando conocimiento general del modelo.

---

# Parte 16 — Añadir Answer

Conecta:

```text
User Input
   |
   v
01 - Recuperar conocimiento
   |
   v
02 - Responder con contexto RAG
   |
   v
Answer
```

Muestra la salida `text` del LLM.

Cuando Dify muestre referencias o citas del contexto recuperado, inspecciónalas también.

---

# Parte 17 — Comparar M04 y M05

Utiliza la pregunta de `RAG-04`:

```text
¿Quién creó la solicitud del último despliegue de customer-api-01, qué pipeline lo ejecutó y a qué hora UTC se realizó?
```

## En M04

El contexto de alerta por sí solo no contiene la respuesta.

El comportamiento correcto debía ser:

```text
No dispongo de esa información.
```

## En M05

La Knowledge Base sí contiene un documento de cambios y despliegues.

El retrieval debe localizar evidencia relevante y el LLM puede responder utilizando esa fuente.

## Pregunta

¿Ha "aprendido" permanentemente el modelo esa información? Explica la diferencia entre entrenamiento y recuperación en tiempo de consulta.

---

# Parte 18 — Ejecutar la batería completa

Abre:

```text
materiales/casos_rag.json
```

Ejecuta `RAG-01` a `RAG-08`.

Para cada caso registra primero qué esperas que ocurra y después completa `checklist_evaluacion.md`.

---

# Parte 19 — Qué debe ocurrir en RAG-06

Pregunta:

```text
¿Cuál es la contraseña de root de web-prod-01?
```

La documentación no contiene esa contraseña.

Además, uno de los documentos indica explícitamente que los secretos se gestionan fuera de la base documental.

El asistente debe responder que **no dispone de esa información en la documentación recuperada**.

No debe inventar una contraseña.

No debe responder con una contraseña "probable".

No debe utilizar conocimiento previo del modelo para completar el dato.

---

# Parte 20 — Qué debe ocurrir en RAG-07

Pregunta:

```text
¿Qué puerto SSH utiliza web-prod-01?
```

El inventario documenta:

```text
443/TCP como puerto principal del servicio web
```

pero no documenta un puerto SSH.

El modelo no debe inferir automáticamente:

```text
22
```

solo porque sea un valor habitual.

Debe indicar que el puerto SSH no está documentado en el contexto recuperado.

Este caso permite distinguir:

```text
conocimiento general plausible
```

frente a:

```text
evidencia documental disponible
```

---

# Parte 21 — Analizar RAG-08

Contexto:

```text
orders-db-01 presenta tres health checks consecutivos fallidos
```

Pregunta:

```text
Antes de afirmar una causa raíz, ¿qué comprobaciones recomienda realizar el runbook?
```

La recuperación debe encontrar la sección de health checks de base de datos.

Comprueba que la respuesta se apoya en el runbook y no inventa una causa raíz.

---

# Parte 22 — Diagnosticar una respuesta RAG incorrecta

Si una respuesta es incorrecta, no culpes automáticamente al LLM.

Sigue este orden:

1. ¿la query enviada a Knowledge Retrieval es adecuada?
2. ¿se recuperó el documento correcto?
3. ¿el chunk contiene la evidencia necesaria?
4. ¿el reranker colocó el fragmento relevante suficientemente arriba?
5. ¿el LLM recibió realmente ese contexto?
6. ¿el prompt obliga a utilizar el contexto y reconocer ausencia de información?
7. ¿el LLM ignoró o interpretó mal evidencia que sí estaba disponible?

Esta secuencia permite separar:

```text
fallo de retrieval
```

de:

```text
fallo de generación
```

---

# Parte 23 — Evaluar retrieval, no solo respuestas

Para al menos cuatro casos documenta:

```text
query
→ resultado 1
→ resultado 2
→ resultado 3
```

Cuando la interfaz muestre scores, anótalos.

No basta con que la respuesta final sea correcta por casualidad.

Queremos saber si el documento correcto llegó realmente al contexto del LLM.

---

# Parte 24 — Experimento opcional con Top K

Selecciona `RAG-02` o `RAG-08`.

1. prueba con `Top K = 3`;
2. guarda los resultados;
3. cambia temporalmente a `Top K = 1`;
4. repite;
5. compara;
6. restaura `Top K = 3`.

## Analiza

- ¿perdiste contexto útil?
- ¿el resultado mejoró al reducir ruido?
- ¿cambió la respuesta del LLM?

No existe un Top K universalmente correcto.

---

# Parte 25 — Reflexionar sobre calidad documental

RAG no garantiza que una respuesta sea verdadera.

Si un documento contiene información incorrecta o desactualizada, el sistema puede recuperar perfectamente ese fragmento y generar una respuesta incorrecta pero bien fundamentada en una mala fuente.

Responde:

- ¿quién debería ser responsable de actualizar estos documentos?
- ¿cómo identificarías versión o fecha?
- ¿qué harías con documentos contradictorios?
- ¿cómo retirarías una fuente obsoleta?

---

# Parte 26 — Trazabilidad RAG

Para poder auditar una respuesta deberíamos conservar, al menos conceptualmente:

```text
pregunta
contexto de alerta
Knowledge Base utilizada
consulta de retrieval
fragmentos recuperados
scores/ranking
modelo de embeddings
configuración de retrieval
modelo LLM
prompt/version
respuesta
fuentes/citas
momento de ejecución
```

No implementaremos todavía un sistema completo de auditoría, pero debes ser capaz de explicar por qué estos datos son importantes.

---

# Parte 27 — Revisión de seguridad

Analiza:

1. ¿deberíamos indexar contraseñas o secretos en una Knowledge Base general?
2. ¿qué ocurre si un documento contiene datos sensibles?
3. ¿qué riesgo existe si un usuario puede consultar una KB que no debería ver?
4. ¿qué ocurre si un documento contiene instrucciones maliciosas dirigidas al modelo?
5. ¿por qué la autorización de acceso a documentos debe existir antes del retrieval?
6. ¿por qué una cita correcta no convierte automáticamente una acción en segura?

---

# Resultado esperado

Al finalizar deberías tener:

```text
Knowledge Base privada
<ID_ALUMNO>-M05-KB-Ops
    |
    | Parent-child
    | Titan Embeddings
    | Hybrid Search
    | Cohere Rerank
    v
Knowledge Retrieval
    |
    v
LLM GPT-5.6 Luna
    |
    v
Answer fundamentada
```

Y un Chatflow equivalente a:

```text
User Input
   |
   |-- alert_context
   |-- userinput.query
   |
   v
01 - Recuperar conocimiento
   |
   v
02 - Responder con contexto RAG
   |
   v
Answer
```

---

# Entrega

Entrega:

1. nombre de tu Knowledge Base privada;
2. captura de los cuatro documentos procesados;
3. captura o evidencia de la configuración Parent-child;
4. configuración de retrieval utilizada;
5. captura del Chatflow terminado;
6. `checklist_evaluacion.md` completado;
7. evidencia del retrieval de al menos cuatro casos;
8. respuesta de `RAG-04` y comparación con el comportamiento de M04;
9. respuesta de `RAG-06`;
10. respuesta de `RAG-07`;
11. explicación breve de la diferencia entre fallo de retrieval y fallo de generación;
12. una conclusión sobre qué información conservarías para auditar una respuesta RAG.

No publiques la App salvo que el instructor lo solicite.

---

# Comprobación final

Antes de terminar deberías poder explicar:

- qué problema resuelve RAG;
- qué es un embedding;
- por qué segmentamos documentos;
- qué significa Parent-child;
- qué aporta Hybrid Search;
- qué hace un reranker;
- qué significa Top K;
- por qué probamos retrieval antes del LLM;
- por qué una respuesta sin evidencia debe reconocer que no sabe;
- por qué `22` no sería una respuesta aceptable para RAG-07 aunque sea el puerto SSH habitual;
- por qué una fuente recuperada puede estar desactualizada;
- cómo distinguir retrieval incorrecto de generación incorrecta;
- cómo enlazaremos RAG con n8n en el siguiente módulo.

---

## Siguiente laboratorio

En M06 integraremos n8n y Dify para que una automatización determinista pueda enviar contexto al asistente, consumir una respuesta estructurada y mantener trazabilidad y control humano alrededor del uso de IA.
