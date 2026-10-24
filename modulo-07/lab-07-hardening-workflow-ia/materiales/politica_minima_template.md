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
| estado operativo vivo | Telvora Ops Hub mediante MCP | Solo como contexto factual observado |
| hipótesis | Dify/LLM | Sí, pero no como hecho |

Añade o modifica filas cuando sea necesario.

## 3. Autenticación y secretos

- Webhook de entrada:
- Método de autenticación:
- Dónde se almacena el secreto:
- API key de Dify:
- Token de Lab API:
- Integración MCP utilizada:
- Quién administra `MCP_TOKEN`:
- ¿Necesitan los alumnos conocer `MCP_TOKEN`?:
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

## 7. MCP y herramientas

Servidor MCP autorizado:

```text
Telvora Ops Hub
```

Tools permitidas para este caso de uso:

```text

```

Tools o clases de acciones expresamente prohibidas:

```text

```

Datos que recibe el Agent desde el workflow:

```text

```

Tratamiento de texto/instrucciones embebidas en respuestas de tools:

```text

```

Regla ante lista vacía / ausencia de contexto vivo:

```text

```

Regla ante MCP no disponible:

```text

```

## 8. Resiliencia

Timeout / error Dify:

```text

```

Número máximo de reintentos Dify:

```text

```

Comportamiento degradado si Dify falla:

```text

```

Comportamiento degradado si MCP falla pero Dify sigue disponible:

```text

```

## 9. Trazabilidad

Identificadores a conservar:

```text
alert_id
n8n execution id
dify_run_id
evidence_id
```

Evidencia adicional útil:

```text
Agent rounds / tool calls MCP cuando sea necesario para diagnóstico
```

Información que no debe registrarse:

```text

```

## 10. Criterio de paso a piloto

El workflow no puede considerarse candidato a piloto hasta que:

- [ ] pruebas normales pasen;
- [ ] casos adversariales críticos pasen;
- [ ] exista fallback sin IA;
- [ ] exista política de degradación sin MCP;
- [ ] Tool List MCP cumpla mínimo privilegio;
- [ ] no existan tools destructivas en el Agent;
- [ ] secretos estén en Credentials/integraciones administradas;
- [ ] revisión humana esté definida;
- [ ] riesgos residuales estén documentados;
- [ ] exista un owner operativo;
- [ ] exista procedimiento de rollback/despublicación.

## 11. Riesgos aceptados temporalmente

| Riesgo | Motivo de aceptación | Control compensatorio | Fecha/condición de revisión |
|---|---|---|---|
|  |  |  |  |
