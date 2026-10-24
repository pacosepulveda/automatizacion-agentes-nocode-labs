# Rúbrica de evaluación — M09 Capstone

Puntuación sugerida: 100 puntos.

## 1. Arquitectura y separación de responsabilidades — 20 puntos

| Criterio | Puntos |
|---|---:|
| n8n conserva orquestación, validación y reglas | 4 |
| REST, RAG y MCP tienen responsabilidades diferenciadas y justificadas | 5 |
| Dify se limita a retrieval, tool use controlado y análisis | 4 |
| trust boundaries y fuentes autoritativas claras | 4 |
| diseño end-to-end legible y justificable | 3 |

## 2. Contratos y calidad funcional — 20 puntos

| Criterio | Puntos |
|---|---:|
| contrato de entrada validado | 5 |
| prioridad P1-P4 correcta | 5 |
| contrato de salida estable | 5 |
| `next_step` determinista y coherente | 5 |

## 3. RAG, MCP e IA controlada — 20 puntos

| Criterio | Puntos |
|---|---:|
| Knowledge Base y nuevo runbook integrados | 4 |
| Agent `FunctionCalling` usa tools MCP read-only justificadas | 4 |
| puede demostrar tool calls reales y uso correcto de resultados | 4 |
| distingue hechos validados / operativos / documentados / hipótesis | 4 |
| reconoce contexto documental u operativo insuficiente/no disponible | 4 |

## 4. Seguridad — 15 puntos

| Criterio | Puntos |
|---|---:|
| Webhook autenticado | 3 |
| secretos en Credentials/integraciones gestionadas | 3 |
| minimización de datos y del contexto del Agent | 3 |
| prompt injection no altera autoridad ni tool safety | 4 |
| no dispone de tools destructivas ni simula acciones inexistentes | 2 |

## 5. Resiliencia y fallback — 10 puntos

| Criterio | Puntos |
|---|---:|
| Dify failure preserva decisión | 3 |
| MCP failure se distingue y no inventa operational facts | 3 |
| human review en degraded mode | 2 |
| fallo Evidence API no se oculta / retries limitados | 2 |

## 6. Pruebas y trazabilidad — 10 puntos

| Criterio | Puntos |
|---|---:|
| batería CAP-01…CAP-11 ejecutada con evidencia | 4 |
| alert_id/n8n/Dify/Agent/evidence correlacionables | 3 |
| explicación de errores, tool calls y rutas | 3 |

## 7. Documentación y defensa — 5 puntos

| Criterio | Puntos |
|---|---:|
| informe técnico claro y trade-offs justificados | 2 |
| presentación estructurada | 1 |
| responde preguntas técnicas con evidencia | 2 |

---

# Condiciones críticas

Independientemente de la puntuación, el capstone no puede considerarse satisfactorio si ocurre cualquiera de estos casos:

```text
- una petición sin autenticación procesa el workflow normal;
- el LLM puede cambiar la prioridad autoritativa;
- se expone un secreto real o MCP_TOKEN;
- el Agent dispone de una tool destructiva no autorizada;
- la solución afirma ejecutar remediación inexistente;
- un fallo de Dify destruye la clasificación determinista;
- un fallo MCP produce operational facts inventados o una acción sensible;
- no se ejecuta ninguna prueba de excepción/adversarial;
```

---

# Interpretación orientativa

```text
90–100  Excelente: solución consistente, defendible y preparada para siguiente fase.
75–89   Correcta: cumple el objetivo con mejoras menores identificadas.
60–74   Parcial: funciona, pero presenta debilidades importantes de diseño/control.
<60     Insuficiente: faltan elementos esenciales del caso integrado.
```

La puntuación no sustituye la revisión de condiciones críticas.
