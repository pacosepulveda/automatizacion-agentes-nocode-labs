# Política mínima del prototipo — Plantilla

Completa esta política al finalizar el laboratorio.

## 1. Alcance

Sistema / workflow:

```text
M07-LAB07-Hardened-AI-Triage
```

Finalidad permitida:

```text

```

Acciones expresamente fuera de alcance:

```text

```

## 2. Fuentes autoritativas

| Dato / decisión | Fuente autoritativa | ¿Puede IA modificarlo? |
|---|---|---|
| severity | evento validado por n8n | No |
| priority | reglas n8n | No |
| owner_team | API de activos | No |
| requires_human_review | política n8n | No |
| orientación operativa | documentación RAG válida | Solo como recomendación |
| hipótesis | Dify/LLM | Sí, pero no como hecho |

Añade o modifica filas cuando sea necesario.

## 3. Autenticación y secretos

- Webhook de entrada:
- Método de autenticación:
- Dónde se almacena el secreto:
- API key de Dify:
- Token de Lab API:
- Datos que nunca deben enviarse al LLM:

## 4. Validación de entrada

Campos obligatorios:

```text

```

Valores permitidos de severity:

```text

```

Límite de `summary`:

```text

```

Límite de `details`:

```text

```

Tratamiento de payload inválido:

```text

```

## 5. Uso de IA

La IA puede:

```text

```

La IA no puede:

```text

```

Condiciones que obligan a revisión humana:

```text

```

## 6. RAG

Fuentes documentales permitidas:

```text

```

Regla para documentación insuficiente:

```text

```

Tratamiento de instrucciones embebidas en documentos:

```text

```

## 7. Resiliencia

Timeout / error Dify:

```text

```

Número máximo de reintentos:

```text

```

Comportamiento degradado:

```text

```

## 8. Trazabilidad

Identificadores a conservar:

```text
alert_id
n8n execution id
dify_run_id
evidence_id
```

Información que no debe registrarse:

```text

```

## 9. Criterio de paso a piloto

El workflow no puede considerarse candidato a piloto hasta que:

- [ ] pruebas normales pasen;
- [ ] casos adversariales críticos pasen;
- [ ] exista fallback sin IA;
- [ ] secretos estén en Credentials;
- [ ] revisión humana esté definida;
- [ ] riesgos residuales estén documentados;
- [ ] exista un owner operativo;
- [ ] exista procedimiento de rollback/despublicación.

## 10. Riesgos aceptados temporalmente

| Riesgo | Motivo de aceptación | Control compensatorio | Fecha/condición de revisión |
|---|---|---|---|
|  |  |  |  |
