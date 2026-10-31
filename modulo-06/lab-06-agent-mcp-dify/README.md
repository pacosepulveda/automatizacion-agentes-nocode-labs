# Laboratorio 6 — Agent + MCP en Dify

En este laboratorio evolucionarás la solución de M05 para que, además de consultar documentación mediante RAG, pueda **investigar estado operativo vivo** utilizando un Agent de Dify y herramientas MCP.

La aplicación seguirá siendo una Workflow App invocada desde n8n. El cambio principal será sustituir el LLM final por un **Agent** que reciba contexto documental y pueda decidir qué herramientas de `Telvora Ops Hub` necesita consultar.

## Resultado esperado

Al finalizar, la solución combinará:

```text
alerta + prioridad determinista
          |
          v
          RAG
          |
          v
   Agent + MCP tools
          |
          v
 investigación técnica
          |
          v
 respuesta de n8n
```

El Agent solo tendrá herramientas de consulta. No podrá reiniciar servidores, hacer rollback, cambiar firewall ni ejecutar remediaciones.
