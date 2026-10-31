# Hoja de trabajo — M01

## 1. Resultado manual de las alertas

| alert_id | activo encontrado | environment | criticality | owner_team | prioridad / revisión | justificación |
|---|---|---|---|---|---|---|
| ALT-1001 | | | | | | |
| ALT-1002 | | | | | | |
| ALT-1003 | | | | | | |
| ALT-1004 | | | | | | |

## 2. Clasificación de tareas

Utiliza principalmente una de estas categorías:

`WORKFLOW`, `API / TOOL`, `LLM`, `RAG`, `AGENT`, `HUMANO`.

| Tarea | Mecanismo principal | Justificación |
|---|---|---|
| Comprobar campos obligatorios | | |
| Consultar datos del activo | | |
| Calcular P1–P4 | | |
| Interpretar texto ambiguo | | |
| Buscar procedimiento interno | | |
| Decidir qué fuentes investigar | | |
| Consultar estado actual | | |
| Consultar cambios recientes | | |
| Formular hipótesis | | |
| Reiniciar servicio de producción | | |
| Autorizar rollback | | |

## 3. Workflow frente a agente

### Regla determinista

`severity = critical → P1`

¿Por qué es apropiada para un workflow?

**Respuesta:**


### Investigación

`Investiga qué puede estar provocando los errores 502 de srv-web-01.`

¿Por qué podría requerir un agente?

**Respuesta:**


## 4. Arquitectura propuesta

Organiza conceptualmente estas piezas:

```text
MONITORING
WORKFLOW
ASSET API
LLM
DOCUMENTACIÓN
AGENT
TOOLS
HUMANO
```

Dibuja aquí tu propuesta:

```text



```

## 5. Decisiones que no debe tomar libremente el modelo

Anota al menos tres decisiones o acciones que deberían estar controladas por reglas o por una persona.

1.
2.
3.
