# Laboratorio 5 — RAG y conocimiento interno en Dify

En este laboratorio evolucionarás la solución de M04 para que el análisis de incidencias pueda utilizar documentación interna de Telvora mediante **RAG (Retrieval-Augmented Generation)**.

Partirás de la misma Workflow App de Dify y añadirás una Knowledge Base, un nodo `Knowledge Retrieval` y una nueva entrada `retrieval_query`.

Después actualizarás el workflow de n8n para que construya esa consulta de recuperación y siga invocando Dify como parte del flujo end-to-end.

## Resultado esperado

Al finalizar, la solución podrá combinar:

```text
alerta + datos del activo + prioridad determinista
                    |
                    v
          documentación interna
                    |
                    v
            análisis fundamentado
```

La documentación de este laboratorio es ficticia y se utiliza exclusivamente con fines formativos.
