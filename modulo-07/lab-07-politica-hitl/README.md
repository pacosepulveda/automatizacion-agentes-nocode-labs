# Laboratorio 7 — Decisiones, política y Human-in-the-Loop

En M06 el Agent ya podía investigar una incidencia utilizando documentación RAG y herramientas MCP. Sin embargo, su resultado seguía siendo principalmente texto para una persona.

En este laboratorio evolucionarás esa misma solución para que:

1. Dify convierta la recomendación del Agent en unos pocos campos estructurados;
2. n8n aplique una política determinista sobre esos campos;
3. las continuaciones seguras puedan avanzar automáticamente;
4. las acciones sensibles o la evidencia insuficiente requieran aprobación humana;
5. una ejecución que necesita aprobación quede realmente pausada hasta recibir `approve` o `reject`.

## Resultado esperado

```text
Agent + RAG + MCP
        |
        v
recomendación
        |
        v
Parameter Extractor
        |
        v
n8n POLICY
   /          \
AUTO         HUMANO
 |             |
 v             v
continúa     WAIT
               |
          approve/reject
```

No se ejecutarán reinicios, rollbacks ni cambios destructivos reales. El objetivo es construir el mecanismo de decisión y aprobación de forma segura.
