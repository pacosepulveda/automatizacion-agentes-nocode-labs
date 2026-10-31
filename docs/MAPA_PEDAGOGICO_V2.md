# Mapa pedagógico V2

## Regla central

Cada módulo debe poder completar esta frase:

> **Antes de este módulo nuestra solución no podía ______. Después del laboratorio sí puede.**

La solución se construye de forma incremental. No se introducen minicasos independientes.

## Solución final

```text
SISTEMA DE MONITORIZACIÓN
        |
        | alerta JSON
        v
      n8n
        |
        |-- validar
        |-- enriquecer vía API
        |-- calcular prioridad
        v
      Dify
        |
        |-- RAG -> documentación interna
        |-- Agent -> tools MCP
        |             |-- health
        |             |-- incidentes
        |             |-- cambios
        |             `-- alertas
        |
        `-- síntesis
        v
 hechos + hipótesis + recomendación
        |
        v
      n8n
        |
        |-- política
        |-- acción segura o revisión humana
        `-- registro
```

## Progresión M01–M09

| Módulo | Problema | Capacidad añadida | Resultado |
|---|---|---|---|
| M01 | El proceso manual mezcla distintos tipos de decisiones | Diseñar qué automatizar y qué corresponde a workflow, IA, agente o humano | Arquitectura funcional inicial |
| M02 | El operador sigue clasificando alertas manualmente | Workflow determinista con n8n | Triaje automático básico |
| M03 | Entrada y datos todavía simulados | Webhook + REST + credenciales | Integración real entre sistemas |
| M04 | Las reglas no interpretan contexto ambiguo | LLM | Análisis contextual |
| M05 | El LLM desconoce procedimientos internos | RAG | Respuestas fundamentadas en conocimiento privado |
| M06 | RAG responde pero no investiga activamente | Agent + Function Calling + MCP | Agente que decide qué tools consultar |
| M07 | El agente aún no forma parte de una automatización completa | Workflow + agente + Human-in-the-Loop | Automatización agentic |
| M08 | La solución funciona en happy path | Errores, seguridad y situaciones realistas | Solución casi final |
| M09 | Aparece un nuevo requisito empresarial | Extensión dirigida de la solución | Adaptación y validación end-to-end |

## Criterios de diseño

1. El protagonista es siempre la automatización que estamos construyendo.
2. La ingeniería empresarial aparece cuando una necesidad real la hace necesaria.
3. Cada módulo debe dejar una nueva capacidad visible y funcional.
4. El alumno debe pasar progresivamente más tiempo construyendo y experimentando.
5. M09 no es un proyecto libre ni una reconstrucción desde cero.
6. M09 debe durar aproximadamente 1 h 45 min y nunca superar 2 horas.

## Distribución práctica orientativa

| Módulo | Construcción / experimentación |
|---|---:|
| M01 | 50–60% |
| M02 | ~70% |
| M03 | ~75% |
| M04 | ~70% |
| M05 | ~70% |
| M06 | ~80% |
| M07 | ~80% |
| M08 | ~80% |
| M09 | ~80% |

## Filosofía de robustez

Se mantienen:

- validación;
- credenciales;
- contratos mínimos;
- errores;
- mínimo privilegio;
- prompt injection;
- fallback;
- Human-in-the-Loop;
- trazabilidad.

Pero no se convierten en fines en sí mismos. Se introducen cuando la solución los necesita.
