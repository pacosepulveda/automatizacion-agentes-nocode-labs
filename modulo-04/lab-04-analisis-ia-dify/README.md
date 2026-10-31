# Laboratorio 4 — Análisis de alertas con IA en Dify

En este laboratorio ampliarás la automatización construida en M03 para añadir análisis semántico mediante IA generativa.

Primero construirás una Workflow App en Dify y observarás las limitaciones de un prompt demasiado genérico. Después mejorarás sus instrucciones y, por último, conectarás Dify con tu workflow de n8n mediante la API.

Al terminar, una alerta válida podrá recorrer esta cadena:

```text
Webhook → API de activos → prioridad P1–P4 → Dify/LLM → respuesta HTTP
```

La IA no sustituye las reglas de prioridad ni ejecuta remediaciones.

## Materiales

- `ENUNCIADO.md`
- `materiales/prompt_base.md`
- `materiales/casos_dify.json`
- `materiales/eventos_webhook_m04.json`
- `materiales/comportamiento_esperado.md`
