# Rúbrica de evaluación — M09 Capstone

Puntuación sugerida: 100 puntos.

## 1. Arquitectura y separación de responsabilidades — 20 puntos

| Criterio | Puntos |
|---|---:|
| n8n conserva orquestación, validación y reglas | 5 |
| Dify se limita a retrieval/análisis | 5 |
| trust boundaries y fuentes autoritativas claras | 5 |
| diseño end-to-end legible y justificable | 5 |

## 2. Contratos y calidad funcional — 20 puntos

| Criterio | Puntos |
|---|---:|
| contrato de entrada validado | 5 |
| prioridad P1-P4 correcta | 5 |
| contrato de salida estable | 5 |
| `next_step` determinista y coherente | 5 |

## 3. RAG e IA controlada — 15 puntos

| Criterio | Puntos |
|---|---:|
| Knowledge Base y nuevo runbook integrados | 4 |
| grounding correcto | 4 |
| distingue hechos/orientación/hipótesis | 4 |
| reconoce contexto insuficiente | 3 |

## 4. Seguridad — 15 puntos

| Criterio | Puntos |
|---|---:|
| Webhook autenticado | 3 |
| secretos en Credentials | 3 |
| minimización de datos | 3 |
| prompt injection no altera autoridad | 4 |
| no simula acciones inexistentes | 2 |

## 5. Resiliencia y fallback — 10 puntos

| Criterio | Puntos |
|---|---:|
| Dify failure preserva decisión | 4 |
| human review en degraded mode | 2 |
| retries limitados | 2 |
| fallo Evidence API no se oculta | 2 |

## 6. Pruebas y trazabilidad — 10 puntos

| Criterio | Puntos |
|---|---:|
| batería CAP ejecutada con evidencia | 4 |
| alert_id/n8n/Dify/evidence correlacionables | 3 |
| explicación de errores y rutas | 3 |

## 7. Documentación y defensa — 10 puntos

| Criterio | Puntos |
|---|---:|
| informe técnico claro | 3 |
| decisiones/trade-offs justificados | 3 |
| presentación estructurada | 2 |
| responde preguntas técnicas con evidencia | 2 |

---

# Condiciones críticas

Independientemente de la puntuación, el capstone no puede considerarse satisfactorio si ocurre cualquiera de estos casos:

```text
- una petición sin autenticación procesa el workflow normal;
- el LLM puede cambiar la prioridad autoritativa;
- se expone un secreto real;
- la solución afirma ejecutar remediación inexistente;
- un fallo de Dify destruye la clasificación determinista;
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
