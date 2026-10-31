# Comportamiento esperado — M09

El laboratorio amplía la solución de M08 sin sustituir sus controles existentes.

## M09-E2E-01 — 1 día restante

Esperado:

```text
category=certificate
severity=low

days_to_expiry=1
→ P1
```

La `severity=low` no debe reducir la prioridad.

RAG debería recuperar `TLS-CERT-01`.

El Agent debería reconocer que la renovación es urgente y puede consultar health o incidentes abiertos si aportan contexto.

La recomendación esperada pertenece a:

```text
renew_certificate
```

La política de n8n debe producir:

```text
human_review
```

incluso si el Agent indicara por error que no requiere revisión.

---

## M09-E2E-02 — 14 días restantes

Esperado:

```text
severity=critical

days_to_expiry=14
→ P3
```

La severity genérica no debe elevar la prioridad de una alerta de certificado.

El runbook indica planificación y notificación al equipo propietario.

Una recomendación razonable es:

```text
notify_team
```

que puede ser candidata a ruta automática según la política existente.

No es obligatorio consultar MCP si el Agent determina que no necesita estado vivo adicional.

---

## M09-E2E-03 — 45 días restantes

Esperado:

```text
severity=high

days_to_expiry=45
→ P4
```

El runbook no exige intervención urgente.

Una recomendación razonable es:

```text
continue_monitoring
```

El Agent no debería llamar a tools innecesarias solo porque estén disponibles.

---

## M09-E2E-04 — dato específico ausente

Falta:

```text
days_to_expiry
```

Esperado:

```text
status = manual_review
reason = missing_certificate_data
```

Dify no debe ser llamado.

La ausencia de un dato necesario para una regla determinista no debe resolverse mediante una estimación del LLM.

---

# Criterios finales

La solución es válida si:

1. las alertas que no sean `category=certificate` siguen funcionando como antes;
2. las alertas de certificado utilizan la nueva regla P1–P4;
3. `certificate_name` y `days_to_expiry` se validan antes de Dify;
4. RAG puede recuperar `TLS-CERT-01`;
5. el Agent conoce `renew_certificate`;
6. el Parameter Extractor puede devolver `renew_certificate` como `action_type`;
7. n8n fuerza HITL para `renew_certificate`;
8. el Agent no ejecuta una renovación;
9. el fallback seguro de M08 sigue funcionando;
10. la salida final mantiene la prioridad calculada por n8n.
