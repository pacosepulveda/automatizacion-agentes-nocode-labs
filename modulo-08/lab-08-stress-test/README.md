# Laboratorio 8 — Stress test de la automatización agentic

En este laboratorio no construirás una arquitectura nueva. Tomarás la solución completa de M07 y comprobarás cómo se comporta ante situaciones menos ideales:

- evidencia insuficiente;
- dependencia MCP no disponible;
- fallo de la llamada a Dify;
- prompt injection en datos de entrada o documentación recuperada;
- recomendación de una acción sensible.

El objetivo es que la solución falle de forma segura y que el alumno pueda explicar qué parte del sistema tomó cada decisión.

## Resultado esperado

Al finalizar, la solución debe mantener estas propiedades:

```text
n8n conserva la autoridad sobre prioridad y política
Dify investiga y recomienda
RAG aporta documentación
MCP aporta estado operativo vivo
las acciones sensibles requieren persona
los fallos de IA/dependencias no provocan acciones automáticas
```

Trabajarás siempre sobre copias propias de la App de Dify y del workflow de n8n. No modifiques integraciones compartidas del Workspace.