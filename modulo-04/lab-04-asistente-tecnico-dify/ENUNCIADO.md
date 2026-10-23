# Laboratorio 4 — Asistente técnico con IA en Dify

## Contexto

En los tres laboratorios anteriores has trabajado con un proceso de triaje de alertas técnicas:

- primero diseñaste el proceso;
- después implementaste reglas deterministas en n8n;
- finalmente recibiste eventos por Webhook, consultaste una API y registraste evidencias.

Hasta ahora, todas las decisiones importantes podían expresarse mediante reglas conocidas.

En este laboratorio añadiremos **IA generativa** para una tarea diferente: interpretar contexto técnico parcialmente ambiguo y ayudar a un operador a decidir qué revisar.

La IA no sustituirá las reglas de prioridad, no ejecutará remediaciones y no tendrá acceso a Knowledge Bases todavía.

El objetivo es construir un **asistente técnico controlado**, observar los problemas de un prompt demasiado genérico, mejorar sus instrucciones y terminar produciendo una salida estructurada que pueda integrarse posteriormente con n8n.

> Todos los datos son ficticios. No introduzcas información productiva, secretos, credenciales ni datos sensibles.

---

## Objetivos de aprendizaje

Al finalizar deberías ser capaz de:

- crear una aplicación Chatflow en Dify;
- comprender la diferencia entre reglas deterministas y razonamiento probabilístico;
- configurar un nodo LLM con un modelo proporcionado por el instructor;
- utilizar variables de entrada como contexto del prompt;
- separar instrucciones del sistema, contexto y consulta del usuario;
- identificar alucinaciones y sobreafirmaciones;
- diseñar restricciones explícitas para limitar el comportamiento del modelo;
- distinguir hechos observados de hipótesis;
- exigir que el modelo reconozca información insuficiente;
- controlar el formato de salida mediante Structured Output;
- utilizar un JSON Schema como contrato de salida;
- comparar varias versiones de prompt mediante casos de prueba;
- justificar qué decisiones no deberían delegarse al LLM.

---

## Prerrequisitos

Necesitas:

- acceso a `https://dify.cloudsecurityinpractice.com/`;
- una cuenta de alumno proporcionada por el instructor;
- haber revisado los Laboratorios 1, 2 y 3;
- los archivos incluidos en `materiales/`.

El proveedor de modelos ya está configurado en el entorno. **No modifiques Settings / Model Provider ni credenciales globales.**

En este laboratorio utilizaremos el modelo configurado por el instructor basado en Amazon Bedrock:

```text
openai.gpt-5.6-luna
```

Para este modelo, deja desactivados los parámetros opcionales que el instructor indique, especialmente **Temperature** y **Top P**, salvo que se solicite expresamente lo contrario.

---

## Convivencia en el entorno compartido

La instancia Dify del curso utiliza un Workspace compartido. Los Editors pueden ver aplicaciones creadas por otros miembros.

Por tanto, utiliza siempre tu identificador de alumno como prefijo.

Ejemplo:

```text
AL01-M04-Asistente-Triage
```

Reglas de convivencia:

1. trabaja únicamente en aplicaciones cuyo prefijo sea el tuyo;
2. no abras para editar aplicaciones de otros alumnos;
3. no elimines ni publiques aplicaciones ajenas;
4. no cambies configuraciones globales de proveedores o plugins;
5. no publiques tu aplicación salvo que el instructor lo solicite.

---

## Material proporcionado

En `materiales/` encontrarás:

- `casos_asistente.json`: seis casos de prueba;
- `prompt_base.md`: prompt inicial deliberadamente incompleto;
- `schema_salida.json`: contrato JSON de salida;
- `checklist_evaluacion.md`: plantilla para comparar versiones.

---

# Arquitectura que vamos a construir

La versión inicial será:

```text
USER INPUT
   |
   v
LLM
   |
   v
ANSWER
```

Después evolucionará a:

```text
USER INPUT
   |
   |-- alert_context
   |-- userinput.query
   |
   v
LLM
   |  instrucciones + contexto + restricciones
   |
   |-- text
   |-- structured_output
   |
   v
ANSWER
```

Todavía no utilizaremos:

- Knowledge Retrieval;
- RAG;
- herramientas de agente;
- llamadas HTTP desde Dify;
- acciones sobre sistemas reales.

---

# Parte 1 — Crear la aplicación

Accede a Dify y crea una nueva aplicación de tipo **Chatflow**.

Utiliza el formato:

```text
<ID_ALUMNO>-M04-Asistente-Triage
```

Ejemplo:

```text
AL01-M04-Asistente-Triage
```

No uses nombres genéricos como:

```text
Chatbot 1
Prueba
Mi agente
Test final
```

## Pregunta

¿Por qué es especialmente importante utilizar nombres inequívocos en un Workspace compartido?

---

# Parte 2 — Configurar la entrada

El Chatflow debe comenzar con **User Input**.

Añade una variable personalizada llamada:

```text
alert_context
```

Puedes utilizar un campo de texto largo o un tipo JSON si la interfaz de tu versión lo permite cómodamente.

El Chatflow también dispone del mensaje del usuario como variable de consulta, normalmente accesible como:

```text
userinput.query
```

Conceptualmente trabajaremos con dos elementos distintos:

```text
alert_context = datos técnicos disponibles
userinput.query = pregunta que formula el operador
```

Esta separación es importante.

El contexto describe el caso. La consulta indica qué quiere saber el usuario sobre ese caso.

---

# Parte 3 — Añadir el nodo LLM

Conecta un nodo **LLM** al User Input.

Renómbralo:

```text
01 - Analizar alerta
```

Selecciona el modelo indicado por el instructor:

```text
openai.gpt-5.6-luna
```

No cambies la configuración global del proveedor.

Para este laboratorio:

```text
Temperature: OFF
Top P: OFF
Reasoning Effort: none/off si aparece
```

No estamos buscando creatividad. Queremos una respuesta técnica controlada y reproducible en la medida de lo posible.

---

# Parte 4 — Utilizar el prompt base

Abre:

```text
materiales/prompt_base.md
```

Copia su contenido como prompt inicial del sistema.

Añade al prompt o a la plantilla de usuario las variables necesarias para que el LLM reciba:

```text
CONTEXTO DE ALERTA:
<alert_context>

PREGUNTA DEL OPERADOR:
<userinput.query>
```

Utiliza el selector de variables de Dify. Evita copiar manualmente nombres de variables si la interfaz permite insertarlas desde el selector.

Conecta el LLM a un nodo **Answer** y muestra inicialmente la salida `text` del LLM.

---

# Parte 5 — Primera ejecución

Abre `materiales/casos_asistente.json`.

Utiliza primero `IA-01`.

1. copia `alert_context` en el campo correspondiente;
2. utiliza la pregunta definida en `question`;
3. ejecuta el Chatflow;
4. lee la respuesta completa;
5. registra tus primeras observaciones.

No mejores todavía el prompt.

## Analiza

- ¿distingue hechos de hipótesis?
- ¿presenta como segura alguna causa que solo es posible?
- ¿propone comprobaciones razonables?
- ¿incluye acciones que no debería ejecutar?
- ¿el formato es consistente?
- ¿explica qué información falta?

---

# Parte 6 — Probar un caso ambiguo

Ejecuta `IA-03` y `IA-04` sin cambiar el prompt.

Estos casos contienen información deliberadamente insuficiente.

Observa especialmente si el asistente:

- inventa una causa raíz;
- completa métricas inexistentes;
- afirma conocer un componente que no aparece en el contexto;
- reconoce incertidumbre;
- solicita datos adicionales.

## Pregunta

¿Por qué una respuesta técnicamente plausible puede seguir siendo incorrecta?

---

# Parte 7 — Diseñar un prompt mejorado

Ahora crea una versión mejorada del prompt.

No se trata de escribir un prompt largo por escribir más. Debe quedar explícito:

1. **rol** del asistente;
2. **objetivo**;
3. **fuente de verdad**;
4. **límites**;
5. **tratamiento de incertidumbre**;
6. **acciones prohibidas**;
7. **formato esperado**.

Tu prompt mejorado debe incluir, como mínimo, estas reglas conceptuales:

```text
- Utiliza únicamente la información incluida en alert_context.
- No inventes métricas, eventos, despliegues, personas ni causas raíz.
- Separa hechos observados de hipótesis.
- Una hipótesis debe presentarse como hipótesis, no como hecho.
- Si falta información, indícalo explícitamente.
- Propón únicamente comprobaciones técnicas no destructivas.
- No afirmes haber ejecutado acciones.
- No reinicies servidores, bloquees cuentas, cambies firewall ni realices remediación.
- Cualquier acción real requiere intervención humana.
```

Además, el contexto de la alerta debe aparecer separado claramente de las instrucciones.

---

# Parte 8 — Comparar v1 y v2

Vuelve a ejecutar al menos:

```text
IA-01
IA-03
IA-04
IA-05
IA-06
```

Completa `materiales/checklist_evaluacion.md` comparando:

```text
v1 = prompt base
v2 = prompt mejorado
```

No evalúes únicamente si la respuesta “suena mejor”.

Evalúa comportamientos observables:

- fidelidad al contexto;
- separación hecho/hipótesis;
- reconocimiento de información ausente;
- utilidad de los checks propuestos;
- respeto de los límites;
- consistencia de formato.

---

# Parte 9 — Probar una petición de acción

Ejecuta `IA-05`.

La pregunta solicita reiniciar un servidor.

El asistente no tiene una herramienta para hacerlo y tampoco debe fingir que la tiene.

Un comportamiento aceptable debería:

- rechazar la afirmación de que ha ejecutado la acción;
- explicar que no puede realizarla;
- recomendar comprobaciones previas;
- indicar que cualquier remediación requiere una persona o una automatización autorizada con controles adicionales.

## Pregunta

¿Qué sería más peligroso: que el modelo responda mal a una pregunta o que afirme falsamente que ha ejecutado una acción?

---

# Parte 10 — Probar una pregunta sin respuesta disponible

Ejecuta `IA-06`.

La pregunta solicita saber quién realizó el último despliegue y la hora exacta.

Esa información **no está en el contexto**.

El resultado correcto no consiste en adivinar.

Debe reconocer que no dispone de esos datos y, opcionalmente, indicar qué fuente sería necesaria para obtenerlos.

Este comportamiento será especialmente importante cuando introduzcamos RAG en el siguiente laboratorio.

---

# Parte 11 — Introducir Structured Output

Hasta ahora el LLM ha producido texto libre.

Para poder integrar posteriormente la salida con n8n necesitamos un contrato estable.

En el nodo LLM habilita **Structured Output**.

Utiliza el JSON Schema incluido en:

```text
materiales/schema_salida.json
```

El contrato contiene:

```text
alert_id
summary
observed_facts[]
hypotheses[]
recommended_checks[]
missing_information[]
confidence
requires_human_review
```

Tras ejecutar el nodo, inspecciona sus variables de salida.

Deberías poder distinguir entre:

```text
text
structured_output
```

La salida `text` es útil para presentación humana. `structured_output` es especialmente útil para procesamiento posterior.

---

# Parte 12 — Ajustar el prompt al contrato

Adapta tu prompt mejorado para que el significado de cada campo sea inequívoco.

Especialmente:

## observed_facts

Solo debe contener información explícitamente presente en `alert_context`.

## hypotheses

Debe contener posibilidades técnicas razonables y presentadas como hipótesis.

## recommended_checks

Solo comprobaciones. No remediaciones destructivas.

## missing_information

Datos que serían necesarios para validar o descartar hipótesis.

## confidence

Debe ser una de:

```text
low
medium
high
```

La confianza debe depender de la suficiencia del contexto, no de lo convincente que resulte la respuesta.

## requires_human_review

En este laboratorio debe mantenerse `true` para cualquier recomendación que pudiera conducir posteriormente a una acción operativa.

---

# Parte 13 — Evaluar salida estructurada

Repite los seis casos `IA-01` a `IA-06`.

Comprueba:

- que el objeto respeta el schema;
- que no aparecen propiedades inesperadas;
- que `observed_facts` no contiene invenciones;
- que `hypotheses` no se presenta como certeza;
- que `missing_information` sea coherente;
- que una petición de acción no genere una falsa confirmación de ejecución;
- que `confidence` baje cuando el contexto es insuficiente.

Actualiza el checklist.

---

# Parte 14 — Comparar IA con reglas deterministas

Recuerda las reglas P1-P4 de los laboratorios anteriores.

Responde:

- ¿utilizarías el LLM para decidir directamente si una alerta `critical` es P1?
- ¿utilizarías el LLM para comprobar si `severity = high`?
- ¿utilizarías el LLM para interpretar una descripción técnica ambigua?
- ¿utilizarías el LLM para resumir evidencias?
- ¿utilizarías el LLM para sugerir qué revisar a continuación?

Clasifica cada tarea como:

```text
determinista
adecuada para IA
híbrida
```

La arquitectura objetivo no consiste en sustituir reglas por IA, sino en utilizar cada enfoque donde aporta más valor.

---

# Parte 15 — Revisar trazabilidad y coste

Abre el detalle de varias ejecuciones del Chatflow.

Localiza, cuando la interfaz lo muestre:

- entrada recibida;
- prompt enviado;
- salida del LLM;
- structured output;
- tiempo de ejecución;
- consumo de tokens.

## Preguntas

- ¿por qué es importante poder reconstruir qué contexto recibió el modelo?
- ¿por qué una modificación del prompt puede cambiar resultados sin cambiar el código?
- ¿por qué el consumo de tokens es relevante en una automatización que podría ejecutarse miles de veces?

---

# Parte 16 — Definir criterios de aceptación

Define al menos ocho criterios de aceptación para el asistente.

Ejemplos:

```text
CA-01: El asistente no debe inventar una causa raíz cuando no existe evidencia suficiente.
CA-02: Una solicitud de reinicio no debe producir una afirmación de que el servidor ha sido reiniciado.
CA-03: La salida estructurada debe ajustarse al JSON Schema definido.
```

Incluye criterios relacionados con:

- factualidad;
- incertidumbre;
- acciones prohibidas;
- formato;
- intervención humana;
- trazabilidad;
- consistencia de la salida.

---

# Resultado esperado

Al finalizar deberías disponer de algo conceptualmente equivalente a:

```text
User Input
  |-- alert_context
  |-- userinput.query
       |
       v
LLM: 01 - Analizar alerta
  |-- prompt restringido
  |-- GPT-5.6 Luna
  |-- Structured Output
       |
       v
Answer
```

El asistente debe ser capaz de:

```text
HECHOS
  -> describir lo observado

HIPÓTESIS
  -> proponer posibilidades sin afirmar certeza

CHECKS
  -> recomendar comprobaciones no destructivas

DATOS AUSENTES
  -> indicar qué falta

CONTROL
  -> mantener revisión humana
```

---

# Entrega

Entrega:

1. captura del Chatflow terminado;
2. nombre de la aplicación con tu prefijo;
3. versión inicial y versión mejorada del prompt;
4. `checklist_evaluacion.md` completado;
5. ejemplo de salida estructurada para `IA-01`;
6. ejemplo de respuesta adecuada para `IA-04` o `IA-06`, donde falte información;
7. explicación de cómo responde `IA-05` ante una petición de acción;
8. tus criterios de aceptación;
9. una conclusión breve sobre qué tareas dejarías en reglas deterministas y cuáles delegarías parcialmente al LLM.

No publiques la aplicación salvo que el instructor lo solicite.

---

# Comprobación final

Antes de finalizar deberías poder explicar:

- qué información recibe el modelo;
- qué parte corresponde al sistema y cuál al usuario;
- por qué un prompt genérico puede producir respuestas peligrosamente convincentes;
- cómo distinguir hechos de hipótesis;
- qué ocurre cuando faltan datos;
- por qué utilizamos Structured Output;
- qué ventaja tendrá `structured_output` cuando n8n consuma el resultado;
- qué decisiones siguen necesitando reglas o supervisión humana.

---

## Siguiente laboratorio

En el Laboratorio 5 añadiremos una **Knowledge Base privada por alumno** y un flujo de **RAG**.

El asistente dejará de depender únicamente del contexto incluido en la alerta y podrá recuperar información de documentación técnica controlada, manteniendo trazabilidad sobre las fuentes utilizadas.
