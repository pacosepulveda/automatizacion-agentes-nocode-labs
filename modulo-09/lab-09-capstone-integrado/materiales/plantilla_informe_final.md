# Informe final — M09 Capstone

## 1. Resumen ejecutivo

Describe en 8-12 líneas:

```text
qué problema resuelve
qué arquitectura has elegido
qué papel tiene n8n
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
Lab API
prioridad
Dify API
Knowledge Retrieval
LLM
policy review
next_step
evidence
respuesta
```

## 4. Fuentes autoritativas

| Dato / decisión | Fuente autoritativa | Justificación |
|---|---|---|
| priority |  |  |
| owner_team |  |  |
| analysis |  |  |
| next_step |  |  |
| human review |  |  |

## 5. Contratos

### Entrada

```text

```

### Dify

```text

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

## 7. Seguridad

Describe controles para:

```text
autenticación
secretos
input validation
input limits
data minimization
prompt injection
autoridad de decisiones
```

## 8. Resiliencia

Describe:

```text
fallback Dify
retries
human review
Evidence API failure
```

## 9. Resultados de pruebas

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

## 10. Trazabilidad

Ejemplo de correlación de una ejecución:

```text
alert_id:
n8n execution:
dify_run_id:
evidence_id:
```

## 11. Trade-offs

Explica al menos tres decisiones donde existía más de una alternativa.

Ejemplos:

```text
qué procesa IA y qué no
cuándo forzar human review
qué hacer si falla evidence
qué datos enviar a Dify
```

## 12. Riesgos residuales

| Riesgo | Severidad | Control actual | Mejora futura |
|---|---|---|---|
|  |  |  |  |

## 13. Resultado del MVP

Selecciona:

```text
[ ] Cumple el brief
[ ] Cumple parcialmente
[ ] No cumple todavía
```

Justificación:

```text

```

## 14. Siguiente paso recomendado

Elige y justifica una de estas opciones o una equivalente:

```text
mantener en shadow
limited pilot
resolver blockers antes de pilotar
ampliar documentación
añadir observabilidad
añadir una integración no destructiva
```

## 15. Lecciones aprendidas

```text

```
