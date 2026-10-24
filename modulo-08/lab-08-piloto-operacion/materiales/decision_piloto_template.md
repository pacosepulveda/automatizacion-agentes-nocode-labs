# Decisión de piloto — Plantilla M08

## 1. Candidato evaluado

```text
Release ID:
Fecha:
Workflow owner:
Pilot approver:
```

## 2. Resultado de gates

| Área | Resultado | Evidencia / observaciones |
|---|---|---|
| Funcionalidad | PASS / FAIL |  |
| Seguridad | PASS / FAIL |  |
| Resiliencia | PASS / FAIL |  |
| MCP / tool safety | PASS / FAIL |  |
| Observabilidad | PASS / FAIL |  |
| Operación | PASS / FAIL |  |
| Rollback | PASS / FAIL |  |
| Known issues | PASS / CONDITIONAL / FAIL |  |

## 3. Métricas observadas

```text
Número de ejecuciones:
Éxitos funcionales:
Fallos esperados/controlados:
Fallos inesperados:
Baseline de latencia end-to-end:
Baseline de tiempo Dify:
Tokens observados:
Casos con MCP disponible:
Casos con MCP unavailable:
Casos con human review:
```

> La latencia se utiliza como baseline del entorno de laboratorio, no como blocker salvo degradación evidente o timeout.

## 4. Riesgos residuales

| Riesgo | Severidad | Mitigación / control compensatorio | Owner |
|---|---|---|---|
|  |  |  |  |

## 5. Condiciones pendientes

| Condición | Owner | Control compensatorio | Criterio de cierre |
|---|---|---|---|
|  |  |  |  |

## 6. Decisión

Selecciona exactamente una:

```text
[ ] READY
[ ] READY_WITH_CONDITIONS
[ ] NOT_READY
```

### Reglas

`READY`

```text
Todos los blockers pasan y no existen condiciones relevantes pendientes para el alcance del piloto.
```

`READY_WITH_CONDITIONS`

```text
Todos los blockers de seguridad/autoridad/fallback/tool safety/rollback pasan, pero queda una o más condiciones no bloqueantes con owner, mitigación y criterio de cierre.
```

`NOT_READY`

```text
Falla cualquier blocker obligatorio o no existe evidencia suficiente para demostrar readiness.
```

## 7. Justificación

```text

```

## 8. Alcance del piloto autorizado

Si la decisión no es `NOT_READY`, define un alcance limitado:

```text
Modo: shadow / limited pilot
Tipos de alertas incluidos:
Usuarios/equipos incluidos:
Human review:
MCP tools permitidas:
Acciones automáticas permitidas:
Acciones automáticas prohibidas:
Duración / criterio de revisión:
```

## 9. Stop conditions

Incluye como mínimo:

```text
bypass de autenticación
secreto expuesto
priority alterada por IA
aparición de tool destructiva no aprobada
MCP unavailable con hechos operativos inventados
fallback/rollback no funcional
```

## 10. Aprobación

```text
Workflow owner:     [ ]
Security reviewer:  [ ]
Pilot approver:     [ ]
Rollback owner:     [ ]
```
