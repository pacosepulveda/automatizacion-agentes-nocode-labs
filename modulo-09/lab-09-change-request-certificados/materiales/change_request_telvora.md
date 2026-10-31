# Change Request — Gestión de expiración de certificados TLS

Telvora quiere ampliar el Incident Investigation & Response Advisor para aceptar alertas del sistema de monitorización de certificados.

## Nuevo tipo de alerta

Las alertas de certificado utilizarán:

```text
category = certificate
```

y añadirán:

```text
certificate_name
days_to_expiry
```

Ejemplo:

```json
{
  "alert_id": "ALT-9001",
  "timestamp": "2026-11-12T09:00:00Z",
  "asset_id": "srv-web-01",
  "severity": "low",
  "source": "certificate-monitoring",
  "category": "certificate",
  "summary": "TLS certificate expires in 1 day",
  "details": "The certificate presented by the production web endpoint expires within 24 hours.",
  "certificate_name": "portal.telvora.example",
  "days_to_expiry": 1
}
```

## Regla nueva de prioridad

Para `category=certificate`, la prioridad genérica basada en `severity` deja de aplicarse.

La prioridad será:

```text
days_to_expiry <= 2   → P1
days_to_expiry <= 7   → P2
days_to_expiry <= 30  → P3
days_to_expiry > 30   → P4
```

Estas reglas son deterministas y deben ejecutarse en n8n antes de llamar a Dify.

## Validación adicional

Cuando `category=certificate`:

- `certificate_name` es obligatorio;
- `days_to_expiry` es obligatorio;
- `days_to_expiry` debe ser numérico.

Si falta alguno de estos datos, la alerta debe pasar a revisión manual y Dify no debe ser llamado.

## Nuevo conocimiento

Debe incorporarse a la Knowledge Base:

```text
telvora_runbook_certificados_tls.txt
```

El Agent debe utilizar ese procedimiento cuando sea relevante.

## Nuevo tipo de acción

El catálogo de acciones se amplía con:

```text
renew_certificate
```

Esta acción representa recomendar el inicio de una renovación de certificado.

No significa que el Agent pueda ejecutarla.

## Política

`renew_certificate` requiere siempre Human-in-the-Loop.

Incluso si el Agent devuelve:

```text
requires_human_review = false
```

n8n debe forzar revisión humana para `renew_certificate`.

## Restricciones

- No crear nueva infraestructura.
- No añadir tools destructivas al Agent.
- No permitir que Dify calcule o modifique la prioridad.
- No ejecutar renovaciones reales.
- Mantener el fallback seguro de M08 si Dify no responde.
