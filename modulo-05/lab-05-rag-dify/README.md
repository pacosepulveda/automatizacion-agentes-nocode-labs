# Laboratorio 5 — Knowledge Base y RAG en Dify

En este laboratorio el asistente técnico creado en M04 evoluciona para responder utilizando **documentación operativa recuperada desde una Knowledge Base privada**.

El objetivo no es únicamente conseguir que el modelo responda preguntas, sino comprender el flujo completo de RAG:

```text
pregunta
→ retrieval
→ fragmentos relevantes
→ contexto
→ LLM
→ respuesta fundamentada
```

## Entorno del curso

Accede a Dify desde:

```text
https://dify.cloudsecurityinpractice.com/
```

Cada alumno creará dos recursos con su identificador:

```text
AL01-M05-KB-Ops
AL01-M05-Asistente-RAG
```

La **Knowledge Base debe mantenerse privada / Only me**. Las Apps se encuentran en un Workspace compartido, por lo que no debes editar, publicar ni eliminar aplicaciones cuyo prefijo no sea el tuyo.

## Qué se construirá

1. una Knowledge Base privada con documentación técnica ficticia;
2. indexación Parent-child de alta calidad;
3. embeddings con Amazon Titan;
4. recuperación Hybrid Search con Cohere Rerank;
5. pruebas de retrieval antes de utilizar un LLM;
6. un Chatflow con Knowledge Retrieval;
7. respuestas basadas únicamente en contexto documental recuperado;
8. pruebas positivas, semánticas, de identificadores exactos y sin respuesta;
9. revisión de fuentes, chunks y trazabilidad.

## Archivos

- [`ENUNCIADO.md`](ENUNCIADO.md): instrucciones del laboratorio.
- [`materiales/casos_rag.json`](materiales/casos_rag.json): batería de preguntas.
- [`materiales/prompt_rag.md`](materiales/prompt_rag.md): prompt de grounding para el Chatflow.
- [`materiales/checklist_evaluacion.md`](materiales/checklist_evaluacion.md): plantilla de evaluación.
- [`materiales/documentos/telvora_inventario_operativo.txt`](materiales/documentos/telvora_inventario_operativo.txt)
- [`materiales/documentos/telvora_runbook_incidentes.txt`](materiales/documentos/telvora_runbook_incidentes.txt)
- [`materiales/documentos/telvora_backups_operacion.txt`](materiales/documentos/telvora_backups_operacion.txt)
- [`materiales/documentos/telvora_cambios_despliegues.txt`](materiales/documentos/telvora_cambios_despliegues.txt)

## Continuidad con el curso

M04 enseñó a controlar un LLM cuando solo dispone del contexto proporcionado por el usuario. M05 añade una fuente documental externa mediante RAG. En M06 integraremos estas capacidades con n8n para combinar **orquestación determinista + recuperación documental + análisis con IA**.
