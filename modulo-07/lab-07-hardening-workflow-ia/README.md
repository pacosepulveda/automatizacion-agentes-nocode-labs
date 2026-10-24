# Laboratorio 7 — Hardening de un workflow con IA, RAG y MCP

En este laboratorio endureceremos la automatización híbrida construida en M06. El objetivo ya no es añadir funcionalidad, sino reducir riesgo antes de pensar en un piloto.

Trabajaremos sobre cinco superficies:

```text
Webhook de entrada
Integraciones y secretos
RAG / LLM
Agent + MCP / tools
Fallback y supervisión humana
```

## Qué se hará

1. identificar trust boundaries y amenazas plausibles;
2. proteger el Webhook con autenticación de cabecera;
3. validar tamaño y contenido de entrada antes de procesar;
4. minimizar qué datos llegan a Dify y qué datos llegan específicamente al Agent MCP;
5. mantener credenciales y `MCP_TOKEN` fuera de prompts, payloads y workflows;
6. endurecer el prompt del Agent y el prompt del LLM final frente a instrucciones embebidas;
7. comprobar que la IA no puede sobrescribir la prioridad determinista;
8. limitar el Agent a cuatro tools MCP de solo lectura;
9. demostrar que una petición de restart/rollback no puede convertirse en una acción;
10. ampliar la revisión humana cuando la confianza, RAG o contexto MCP sean insuficientes/no disponibles;
11. probar degradación ante fallo de Dify y diseñar degradación ante fallo MCP;
12. documentar una política mínima de seguridad y operación.

## Archivos

- [`ENUNCIADO.md`](ENUNCIADO.md): instrucciones completas.
- [`materiales/casos_hardening.json`](materiales/casos_hardening.json): batería de pruebas normales y adversariales.
- [`materiales/matriz_riesgos.md`](materiales/matriz_riesgos.md): threat/risk review incluyendo MCP.
- [`materiales/prompt_agent_mcp_hardened.md`](materiales/prompt_agent_mcp_hardened.md): Instruction endurecida del Agent MCP.
- [`materiales/prompt_hardened.md`](materiales/prompt_hardened.md): prompt endurecido del LLM final.
- [`materiales/schema_analisis_hardened.json`](materiales/schema_analisis_hardened.json): Structured Output con estado MCP `unavailable`.
- [`materiales/documento_prompt_injection.txt`](materiales/documento_prompt_injection.txt): documento controlado para prompt injection indirecta mediante RAG.
- [`materiales/checklist_seguridad.md`](materiales/checklist_seguridad.md): checklist de controles y pruebas.
- [`materiales/politica_minima_template.md`](materiales/politica_minima_template.md): plantilla de política mínima para el prototipo.

## Prerrequisitos

- M06 funcional con RAG + Agent MCP + Structured Output.
- Cuenta personal de n8n.
- App Workflow de Dify de M06.
- Knowledge Base privada de M05.
- Integración compartida `Telvora Ops Hub` visible en Dify.
- Estrategia `FunctionCalling` disponible.
- Identificador de alumno.

La integración MCP y su token son administrados por el instructor a nivel de Workspace. El alumno usa las tools desde su App, pero no necesita conocer `MCP_TOKEN`.

Todos los payloads, documentos e instrucciones adversariales de este laboratorio son ficticios y se utilizan únicamente para formación.