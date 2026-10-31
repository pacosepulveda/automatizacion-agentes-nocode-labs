# Fallos controlados — M08

Estas pruebas se realizan **solo sobre tus copias de M08**. No modifiques integraciones compartidas ni aplicaciones de otros alumnos.

## A. Simular MCP no disponible

1. Abre tu App de Dify de M08.
2. En el nodo Agent anota qué tools de `Telvora Ops Hub` están habilitadas.
3. Deshabilita o elimina temporalmente las cuatro tools MCP del nodo Agent de tu copia:

```text
get_service_health
get_recent_changes
get_recent_alerts
get_open_incidents
```

4. No elimines ni modifiques el servidor MCP compartido del Workspace.
5. Ejecuta `M08-03` desde Dify.
6. Comprueba que el Agent no inventa estado operativo vivo.
7. Restaura las cuatro tools antes de continuar.
8. Ejecuta otra vez el mismo caso y compara el resultado.

La prueba no busca provocar un error global, sino comprobar cómo cambia el comportamiento cuando el Agent no dispone de su fuente de estado vivo.

---

## B. Simular Dify no disponible desde n8n

Trabaja sobre la copia M08 de tu workflow.

1. En el nodo HTTP Request que invoca Dify, configura el manejo de errores para que el workflow pueda continuar por una salida de error o equivalente. El nombre exacto de la opción puede variar según la versión de n8n.
2. Añade después una comprobación que distinga entre:

```text
respuesta Dify válida
```

y:

```text
error / ausencia de los campos esperados
```

3. Para la prueba, cambia temporalmente la URL:

```text
https://dify.cloudsecurityinpractice.com/v1/workflows/run
```

por una ruta inexistente, por ejemplo:

```text
https://dify.cloudsecurityinpractice.com/v1/workflows/run-unavailable-test
```

4. Ejecuta `M08-03`.
5. La rama de fallback debe producir un resultado equivalente a:

```json
{
  "status": "degraded_manual_review",
  "reason": "dify_unavailable",
  "priority": "<prioridad ya calculada por n8n>"
}
```

6. No inventes un `action_type` ni continúes automáticamente.
7. Restaura inmediatamente la URL correcta y vuelve a probar.

## Regla de seguridad

Cuando la dependencia que realiza la investigación inteligente no está disponible:

```text
fallback seguro > suposición automática
```
