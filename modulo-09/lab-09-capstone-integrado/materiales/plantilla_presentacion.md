# Presentación final — Guion M09

Duración sugerida:

```text
5-7 minutos de presentación
+
3-5 minutos de preguntas
```

## Slide / bloque 1 — Problema y alcance

Explica:

```text
qué problema resuelve el MVP
qué no hace
por qué mantiene supervisión humana
```

## Slide / bloque 2 — Arquitectura

Muestra el flujo end-to-end:

```text
Webhook
→ n8n
→ Lab API
→ reglas
→ Dify/RAG
→ policy
→ evidence
→ response
```

Indica claramente qué decisiones son deterministas y cuáles utilizan IA.

## Slide / bloque 3 — Datos y contratos

Muestra:

```text
entrada
salida
Structured Output Dify
fuentes autoritativas
```

## Slide / bloque 4 — Seguridad y resiliencia

Selecciona los controles más importantes:

```text
Header Auth
Credentials
input validation
minimización
prompt injection controls
fallback Dify
human review
```

## Slide / bloque 5 — Pruebas

Presenta al menos:

```text
un happy path
una excepción funcional
un caso adversarial
un fallo de Dify
CAP-10 con nuevo runbook
```

No muestres únicamente capturas; explica qué demuestra cada prueba.

## Slide / bloque 6 — Resultado y siguiente paso

Indica:

```text
qué funciona
qué riesgo queda
qué mejorarías
qué siguiente paso recomiendas
```

---

# Preguntas que debes estar preparado para responder

1. ¿Por qué no utiliza el LLM para decidir P1-P4?
2. ¿Qué ocurre si Dify está caído?
3. ¿Qué ocurre si un documento RAG contiene instrucciones?
4. ¿Cómo sabes qué dato es autoritativo?
5. ¿Por qué `confidence=low` no cambia directamente la prioridad?
6. ¿Qué sucede si Evidence API falla?
7. ¿Qué información no envías a Dify?
8. ¿Cómo evitarías que otro sistema invoque el Webhook sin permiso?
9. ¿Cómo demostrarías por qué se tomó una decisión concreta?
10. ¿Qué cambiarías antes de un piloto real?

---

# Criterio de una buena defensa

No buscamos una demo sin errores.

Buscamos que puedas explicar:

```text
por qué está diseñado así
qué trade-offs existen
qué controles protegen cada frontera
cómo sabes que funciona
qué ocurre cuando falla
```
