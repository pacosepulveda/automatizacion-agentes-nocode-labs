# Resultados esperados — M03

Predice el resultado antes de ejecutar cada caso.

| Caso | Resultado esperado | Revisión humana |
|---|---|---:|
| EVT-01 | P1 — high + production + criticality high | Sí |
| EVT-02 | P3 — medium | No |
| EVT-03 | P1 — critical | Sí |
| EVT-04 | `manual_review` — `asset_not_found` | Sí |
| EVT-05 | `manual_review` — `missing_required_field` | Sí |
| EVT-06 | `manual_review` — `invalid_severity` | Sí |
| EVT-07 | P2 — high sin condición de P1 | No |
| EVT-08 | P4 — low | No |

## Comprobaciones adicionales

- EVT-05 no debe llamar a la API porque falta `asset_id`.
- EVT-04 sí debe llamar a la API y recibir `found=false`.
- EVT-01 debe obtener `environment`, `criticality` y `owner_team` desde la API, no desde el payload del Webhook.
- Una URL incorrecta del endpoint debe provocar un fallo técnico distinto de `asset_not_found`.