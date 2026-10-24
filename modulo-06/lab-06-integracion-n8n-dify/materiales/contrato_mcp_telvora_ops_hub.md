# Contrato MCP — Telvora Ops Hub

El Workspace compartido de Dify tiene configurado por el instructor un servidor MCP llamado:

```text
Telvora Ops Hub
```

Endpoint remoto:

```text
https://api.cloudsecurityinpractice.com/mcp
```

La autenticación está gestionada a nivel del Workspace. **No necesitas conocer ni introducir el token MCP.**

El servidor es deliberadamente **read-only**. No expone herramientas de remediación.

## Tools disponibles

### `get_recent_changes`

Devuelve cambios o despliegues recientes de un activo.

Parámetros:

```json
{
  "asset_id": "srv-web-01",
  "hours": 24
}
```

Restricción del laboratorio:

```text
hours: 1..168
```

Salida conceptual:

```json
{
  "asset_id": "srv-web-01",
  "window_hours": 24,
  "count": 1,
  "changes": [
    {
      "change_id": "...",
      "deployed_at": "...",
      "version": "...",
      "status": "...",
      "actor": "...",
      "pipeline": "...",
      "description": "..."
    }
  ]
}
```

### `get_recent_alerts`

Devuelve alertas recientes del activo.

Parámetros:

```json
{
  "asset_id": "srv-web-01",
  "minutes": 240
}
```

Restricción del laboratorio:

```text
minutes: 1..1440
```

Una lista vacía es un resultado válido y significa que no existen alertas en la ventana consultada.

### `get_open_incidents`

Devuelve incidentes operativos actualmente abiertos para el activo.

Parámetros:

```json
{
  "asset_id": "srv-web-01"
}
```

Salida conceptual:

```json
{
  "asset_id": "srv-web-01",
  "count": 1,
  "incidents": [
    {
      "incident_id": "...",
      "opened_at": "...",
      "severity": "...",
      "title": "...",
      "status": "..."
    }
  ]
}
```

### `get_service_health`

Devuelve el último estado operativo conocido del activo.

Parámetros:

```json
{
  "asset_id": "srv-web-01"
}
```

Salida conceptual:

```json
{
  "found": true,
  "asset_id": "srv-web-01",
  "status": "degraded",
  "checked_at": "...",
  "details": "..."
}
```

## Qué NO proporciona este MCP

No existen tools para:

```text
restart_server
rollback_deployment
change_firewall
disable_account
modify_configuration
```

La ausencia de estas tools es una decisión de seguridad y de diseño del laboratorio.

## Estado vivo y reproducibilidad

Los resultados de MCP pueden cambiar con el tiempo:

- las ventanas de cambios y alertas son temporales;
- el instructor puede activar escenarios operativos;
- una lista vacía puede ser correcta;
- dos alumnos pueden ejecutar la misma consulta en momentos diferentes y observar resultados distintos.

Por ello, el criterio de éxito del laboratorio no es memorizar un JSON exacto, sino demostrar:

1. que Dify descubre las tools;
2. que el Agent decide e invoca tools;
3. que los argumentos son correctos;
4. que la respuesta final distingue hechos de hipótesis;
5. que no se inventan datos ausentes.
