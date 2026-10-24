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
| R-04 | Datos | Texto de alerta intenta dar instrucciones al LLM | Media | Alto | Datos no confiables + prompt hardened | SEC-03 |  |
| R-05 | Autoridad | Datos no autoritativos intentan modificar prioridad | Media | Alto | Prioridad solo en n8n | SEC-04 |  |
| R-06 | RAG | Documento contiene instrucciones maliciosas | Media | Alto | Contexto documental como dato no confiable | SEC-09 |  |
| R-07 | RAG | Contexto insuficiente se interpreta como evidencia | Media | Alto | documentation_status + revisión humana | SEC-06 |  |
| R-08 | Integración | Dify no disponible | Media | Medio/Alto | Retry limitado + degradación | SEC-08 |  |
| R-09 | Secretos | API keys expuestas en datos/logs/repositorio | Baja/Media | Alto | Credentials + minimización | revisión manual |  |
| R-10 | Operación | Recomendación IA se trata como acción autorizada | Media | Alto | Human review + no tools destructivas | SEC-03 / SEC-10 |  |
| R-11 | Trazabilidad | No se puede correlacionar n8n, Dify y evidencia | Media | Medio | alert_id + dify_run_id + evidence_id | revisión de ejecución |  |
| R-12 | Concurrencia/coste | IA se invoca para entradas ya rechazables | Media | Medio | gating antes de Dify | SEC-05 / SEC-07 |  |
| R-13 | MCP / permisos | Agent dispone de tools con más privilegios de los necesarios | Media | Alto | Tool List mínima y read-only | SEC-10 + inspección de Agent |  |
| R-14 | MCP / prompt injection | Texto de alerta intenta controlar qué acción ejecuta el Agent | Media | Alto | Agent recibe solo asset_id + consulta fija; no recibe details | SEC-03 / SEC-10 |  |
| R-15 | MCP / resultados | Texto devuelto por una tool intenta comportarse como instrucción | Baja/Media | Alto | Tool output tratado como dato no confiable en Agent y LLM final | revisión de prompts |  |
| R-16 | MCP / disponibilidad | Estado vivo no disponible pero el sistema inventa contexto operativo | Media | Medio/Alto | error handling del Agent + operational_context_status=unavailable | revisión de configuración / fault test instructor |  |
| R-17 | MCP / secretos | MCP_TOKEN se expone a alumnos, prompts o logs | Baja | Alto | Integración MCP compartida administrada por instructor; token no entra en workflows | revisión manual |  |

## Preguntas

1. ¿Qué riesgo disminuye con autenticación pero no desaparece con un path difícil de adivinar?
2. ¿Qué riesgos no se solucionan con prompting?
3. ¿Qué control impide que Dify sea fuente de verdad de `priority`?
4. ¿Qué riesgo persiste incluso si el JSON estructurado es válido?
5. ¿Qué controles dependen de personas y cuáles son técnicos?
6. ¿Por qué autorizar un servidor MCP no implica que todas sus tools deban estar disponibles para el Agent?
7. ¿Por qué una respuesta de una tool autorizada sigue tratándose como dato desde el punto de vista de instrucciones?
8. ¿Qué diferencia existe entre `no_relevant_live_context` y `unavailable`?
9. ¿Qué riesgos aceptarías temporalmente para un piloto y con qué compensaciones?
