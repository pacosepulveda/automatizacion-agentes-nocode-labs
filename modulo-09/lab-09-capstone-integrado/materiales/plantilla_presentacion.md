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
→ Lab API REST
→ reglas
→ Dify
   ├─ RAG
   ├─ Agent + MCP
   └─ LLM final
→ policy
→ evidence
→ response
```

Indica claramente qué decisiones son deterministas y cuáles utilizan IA.

## Slide / bloque 3 — REST vs RAG vs MCP

Explica en una sola vista:

```text
REST → datos autoritativos mediante contrato conocido
RAG  → conocimiento documental
MCP  → herramientas y contexto operativo vivo para el Agent
```

Muestra las cuatro tools MCP permitidas y por qué son read-only.

## Slide / bloque 4 — Datos y contratos

Muestra:

```text
entrada
salida
Structured Output Dify
fuentes autoritativas
operational_context_status
```

## Slide / bloque 5 — Seguridad y resiliencia

Selecciona los controles más importantes:

```text
Header Auth
Credentials
input validation
minimización
Agent context minimization
prompt injection controls
tool allowlist
fallback Dify
fallback MCP
human review
```

## Slide / bloque 6 — Pruebas

Presenta al menos:

```text
un happy path con tool call MCP
una excepción funcional
CAP-07 adversarial
CAP-09 Dify unavailable
CAP-10 nuevo runbook
CAP-11 MCP unavailable
```

No muestres únicamente capturas; explica qué demuestra cada prueba.

## Slide / bloque 7 — Resultado y siguiente paso

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
2. ¿Por qué el catálogo se consulta por REST y el estado vivo mediante MCP?
3. ¿Qué aporta RAG que MCP no aporta?
4. ¿Cómo sabes que el Agent realmente llamó a una tool?
5. ¿Qué ocurre si Dify está caído?
6. ¿Qué ocurre si Dify funciona pero MCP está caído?
7. ¿Qué diferencia existe entre `no_relevant_live_context` y `unavailable`?
8. ¿Qué ocurre si un documento RAG o una tool response contiene instrucciones?
9. ¿Por qué no hay tools de restart/rollback?
10. ¿Cómo sabes qué dato es autoritativo?
11. ¿Qué sucede si Evidence API falla?
12. ¿Cómo demostrarías por qué se tomó una decisión concreta?
13. ¿Qué cambiarías antes de un piloto real?

---

# Criterio de una buena defensa

No buscamos una demo sin errores.

Buscamos que puedas explicar:

```text
por qué está diseñado así
qué trade-offs existen
qué controles protegen cada frontera
cómo sabes que funciona
qué ocurre cuando falla cada dependencia
```
