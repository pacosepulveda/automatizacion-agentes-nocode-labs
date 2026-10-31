# Resultados esperados — M02

Utiliza esta tabla para comprobar las pruebas después de haber predicho primero el resultado.

| Caso | Resultado esperado | Revisión humana | Motivo principal |
|---|---|---|---|
| M02-01 | P1 | Sí | `severity = critical` |
| M02-02 | P1 | Sí | `high + production + criticality high` |
| M02-03 | P2 | No | `severity = high`, fuera de la condición P1 |
| M02-04 | P3 | No | `severity = medium` |
| M02-05 | P4 | No | `severity = low` |
| M02-06 | Revisión manual | Sí | `severity` no admitida |
| M02-07 | Revisión manual | Sí | falta `asset_id` |
| M02-08 | Revisión manual | Sí | falta `summary` |

## Observación

Si un caso termina en una ruta diferente, inspecciona el input y output de los nodos recorridos antes de modificar el workflow.
