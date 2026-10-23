# Matriz de riesgos — Laboratorio 7

Completa esta matriz antes y después de aplicar controles.

Escala sugerida:

```text
Probabilidad: Baja / Media / Alta
Impacto: Bajo / Medio / Alto
Riesgo residual: Bajo / Medio / Alto
```

| ID | Superficie | Riesgo | Probabilidad inicial | Impacto | Control previsto | Evidencia de prueba | Riesgo residual |
|---|---|---|---|---|---|---|---|
| R-01 | Webhook | Invocación por emisor no autorizado | Alta | Alto | Header Auth | SEC-01 / SEC-02 |  |
| R-02 | Entrada | Campos o valores inválidos | Media | Medio | Validación + allowlist + fallback | SEC-07 |  |
| R-03 | Entrada | Payload excesivo / consumo innecesario | Media | Medio | Límites de longitud antes de Dify | SEC-05 |  |
| R-04 | Datos | Texto de alerta intenta dar instrucciones al LLM | Media | Alto | Treat data as untrusted + prompt hardened | SEC-03 |  |
| R-05 | Autoridad | Datos no autoritativos intentan modificar prioridad | Media | Alto | Prioridad solo en n8n | SEC-04 |  |
| R-06 | RAG | Documento contiene instrucciones maliciosas | Media | Alto | Contexto documental como datos no confiables | SEC-09 |  |
| R-07 | RAG | Contexto insuficiente se interpreta como evidencia | Media | Alto | documentation_status + revisión humana | SEC-06 |  |
| R-08 | Integración | Dify no disponible | Media | Medio/Alto | Retry limitado + degradación | SEC-08 |  |
| R-09 | Secretos | API keys expuestas en datos/logs/repositorio | Baja/Media | Alto | Credentials + minimización | revisión manual |  |
| R-10 | Operación | Recomendación IA se trata como acción autorizada | Media | Alto | Human review + no tools destructivas | SEC-03/SEC-09 |  |
| R-11 | Trazabilidad | No se puede correlacionar n8n, Dify y evidencia | Media | Medio | alert_id + dify_run_id + evidence_id | revisión de ejecución |  |
| R-12 | Concurrencia/coste | IA se invoca para entradas ya rechazables | Media | Medio | gating antes de Dify | SEC-05/SEC-07 |  |

## Preguntas

1. ¿Qué riesgo disminuye con autenticación pero no desaparece con un path difícil de adivinar?
2. ¿Qué riesgos no se solucionan con prompting?
3. ¿Qué control impide que Dify sea fuente de verdad de `priority`?
4. ¿Qué riesgo persiste incluso si el JSON estructurado es válido?
5. ¿Qué controles dependen de personas y cuáles son técnicos?
6. ¿Qué riesgos aceptarías temporalmente para un piloto y con qué compensaciones?
