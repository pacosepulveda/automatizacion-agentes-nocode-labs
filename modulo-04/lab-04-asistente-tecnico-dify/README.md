# Laboratorio 4 — Asistente técnico con IA en Dify

En este laboratorio se introduce por primera vez **IA generativa** en el caso de triaje de alertas técnicas.

Hasta ahora, n8n ha aplicado reglas deterministas sobre datos estructurados. En este módulo se añade un asistente en **Dify** para interpretar el contexto de una alerta, resumir la situación y proponer comprobaciones técnicas sin ejecutar acciones ni sustituir las reglas de clasificación.

## Entorno del curso

Accede a Dify desde:

```text
https://dify.cloudsecurityinpractice.com/
```

El entorno Dify del curso es compartido. Utiliza siempre el identificador asignado por el instructor como prefijo de la aplicación, por ejemplo:

```text
AL01-M04-Asistente-Triage
```

No abras, edites, publiques ni elimines aplicaciones cuyo prefijo no sea el tuyo.

> En este laboratorio no utilizaremos Knowledge Bases. El conocimiento documental y RAG se introducirán en el Laboratorio 5.

## Qué se construirá

Un Chatflow que:

1. recibe el contexto de una alerta técnica;
2. permite formular una pregunta sobre esa alerta;
3. utiliza un modelo LLM configurado por el instructor;
4. genera un análisis técnico controlado;
5. distingue hechos, hipótesis y datos que faltan;
6. propone comprobaciones, pero no acciones destructivas;
7. produce una salida estructurada para facilitar una futura integración con n8n;
8. se evalúa con varios casos de prueba.

## Archivos

- [`ENUNCIADO.md`](ENUNCIADO.md): instrucciones completas del laboratorio.
- [`materiales/casos_asistente.json`](materiales/casos_asistente.json): casos de prueba.
- [`materiales/prompt_base.md`](materiales/prompt_base.md): prompt deliberadamente simple que se utilizará como punto de partida.
- [`materiales/schema_salida.json`](materiales/schema_salida.json): esquema de salida estructurada.
- [`materiales/checklist_evaluacion.md`](materiales/checklist_evaluacion.md): plantilla para comparar variantes.

## Continuidad con el curso

En el siguiente laboratorio se incorporará una **Knowledge Base privada** y recuperación documental mediante **RAG**. En módulos posteriores, Dify y n8n se integrarán para combinar interpretación con IA, reglas deterministas, APIs, trazabilidad y aprobación humana.
