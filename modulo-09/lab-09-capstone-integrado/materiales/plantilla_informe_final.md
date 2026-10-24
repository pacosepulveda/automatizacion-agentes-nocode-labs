# Informe final — M09 Capstone

## 1. Resumen ejecutivo

Describe en 8-12 líneas:

```text
qué problema resuelve
qué arquitectura has elegido
qué papel tiene n8n
qué papel tienen REST, RAG y MCP
qué papel tiene Dify
qué controles principales existen
resultado global de validación
```

## 2. Requisitos interpretados

Lista los requisitos más importantes del briefing y cómo los has implementado.

| Requisito | Implementación / decisión | Evidencia |
|---|---|---|
|  |  |  |

## 3. Arquitectura

Incluye un diagrama end-to-end.

Debe mostrar al menos:

```text
emisor
Webhook n8n
validación
Lab API REST
prioridad
Dify API
Knowledge Retrieval / RAG
Agent FunctionCalling / MCP
LLM final
policy review
next_step
evidence
respuesta
```

Marca trust boundaries y puntos de fallo.

## 4. Fuentes autoritativas y fuentes de contexto

| Dato / decisión | Fuente | Tipo / autoridad | Justificación |
|---|---|---|---|
| priority |  |  |  |
| owner_team |  |  |  |
| documented_guidance |  |  |  |
| operational_facts |  |  |  |
| analysis |  |  |  |
| next_step |  |  |  |
| human review |  |  |  |

## 5. Contratos

### Entrada

```text

```

### REST / Lab API

```text

```

### Dify

```text

```

### MCP

```text
Servidor:
Tools permitidas:
Contexto que recibe el Agent:
Qué acciones NO puede ejecutar:
```

### Salida

```text

```

## 6. RAG

Documentos utilizados:

```text

```

Nuevo documento añadido:

```text
telvora_runbook_portal_cliente.txt
```

Explica:

- estrategia de retrieval;
- cómo verificas grounding;
- qué ocurre si falta documentación.

## 7. MCP y Agent

Explica:

```text
por qué MCP aporta valor frente a llamar REST directamente en esa fase
qué tools expones
por qué son read-only
cómo verificas tool calls reales
cómo distingues no results de MCP unavailable
```

Incluye un ejemplo de round/tool call si está disponible.

## 8. Seguridad

Describe controles para:

```text
autenticación
secretos
input validation
input limits
data minimization
Agent context minimization
prompt injection
RAG injection
tool output como dato no confiable
autoridad de decisiones
tool allowlist / mínimo privilegio
```

## 9. Resiliencia

Describe por separado:

```text
fallback Dify
fallback MCP
retries
human review
Evidence API failure
```

Explica qué capacidad se conserva en cada degradación.

## 10. Resultados de pruebas

| Caso | Esperado | Observado | PASS/FAIL | Evidencia |
|---|---|---|---|---|
| CAP-01 |  |  |  |  |
| CAP-02 |  |  |  |  |
| CAP-03 |  |  |  |  |
| CAP-04 |  |  |  |  |
| CAP-05 |  |  |  |  |
| CAP-06 |  |  |  |  |
| CAP-07 |  |  |  |  |
| CAP-08 |  |  |  |  |
| CAP-09 |  |  |  |  |
| CAP-10 |  |  |  |  |
| CAP-11 |  |  |  |  |

## 11. Trazabilidad

Ejemplo de correlación de una ejecución:

```text
alert_id:
n8n execution:
dify_run_id:
Agent round/tool call relevante:
evidence_id:
```

## 12. Trade-offs

Explica al menos tres decisiones donde existía más de una alternativa.

Ejemplos:

```text
REST directo vs MCP
qué procesa IA y qué no
cuándo forzar human review
qué hacer si falla MCP
qué hacer si falla evidence
qué datos enviar a Dify/Agent
```

## 13. Riesgos residuales

| Riesgo | Severidad | Control actual | Mejora futura |
|---|---|---|---|
|  |  |  |  |

## 14. Resultado del MVP

Selecciona:

```text
[ ] Cumple el brief
[ ] Cumple parcialmente
[ ] No cumple todavía
```

Justificación:

```text

```

## 15. Siguiente paso recomendado

Elige y justifica una de estas opciones o una equivalente:

```text
mantener en shadow
limited pilot
resolver blockers antes de pilotar
ampliar documentación
añadir observabilidad
ampliar herramientas MCP read-only
preparar aprobación humana para futuras tools de acción
```

## 16. Lecciones aprendidas

```text

```
