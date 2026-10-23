# Runbook de operación — Plantilla M08

## 1. Servicio

```text
Nombre:
Release ID:
Workflow n8n:
Workflow Dify:
Knowledge Base:
Owner:
Escalation contact:
```

## 2. Qué hace el sistema

Describe en 5-10 líneas:

```text

```

## 3. Qué NO hace

```text
- no ejecuta remediación automática;
- no permite que Dify cambie priority;
- no elimina human review cuando la policy la exige;
- ...
```

## 4. Smoke test

Pasos mínimos para comprobar salud:

1. comprobar FQDN de n8n;
2. comprobar FQDN de Dify;
3. ejecutar un caso baseline autenticado;
4. verificar prioridad;
5. verificar `ai_status`;
6. verificar ejecución n8n;
7. verificar `dify_run_id` cuando exista;
8. verificar evidencia cuando aplique.

Resultado esperado:

```text

```

## 5. Dify no disponible

Síntomas:

```text

```

Comprobaciones:

```text

```

Comportamiento esperado del workflow:

```text
priority preservada
ai_status = unavailable
human review = true
```

Escalado:

```text

```

## 6. Fallos de autenticación en Webhook

Señal:

```text

```

Qué comprobar:

```text

```

Qué NO hacer:

```text
no desactivar autenticación para restaurar servicio
```

## 7. Documentación insuficiente / confidence low

Comportamiento esperado:

```text

```

Responsable de revisión:

```text

```

## 8. Evidence API no disponible

Comportamiento esperado:

```text

```

¿La decisión principal puede continuar?:

```text

```

Cómo registrar temporalmente el incidente:

```text

```

## 9. Rollback

Trigger de rollback:

```text

```

Configuración conocida como buena:

```text

```

Pasos:

```text
1.
2.
3.
4.
```

Smoke test posterior:

```text

```

## 10. Detener el piloto

Condiciones de stop inmediato:

```text
- bypass de autenticación;
- exposición de secretos;
- IA altera una decisión autoritativa;
- no funciona fallback;
- ...
```

Procedimiento:

```text

```

## 11. Datos para diagnóstico

Conservar:

```text
alert_id
n8n execution id
dify_run_id
evidence_id
status/errores
latencias
```

No registrar:

```text
API keys
tokens
Authorization headers
passwords
```
