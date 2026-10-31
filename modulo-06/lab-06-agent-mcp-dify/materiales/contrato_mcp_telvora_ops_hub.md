# Telvora Ops Hub — Tools MCP disponibles

El Workspace compartido de Dify tiene configurado por el instructor un servidor MCP llamado:

```text
Telvora Ops Hub
```

Endpoint remoto:

```text
https://api.cloudsecurityinpractice.com/mcp
```

La autenticación está gestionada por el instructor a nivel del Workspace. No necesitas conocer ni introducir el token MCP.

El servidor es deliberadamente **read-only**.

## `get_service_health`

Devuelve el último estado operativo conocido de un activo.

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

## `get_recent_changes`

Devuelve cambios o despliegues recientes de un activo.

Parámetros:

```json
{
  "asset_id": "srv-web-01",
  "hours": 24
}
```

Rango permitido:

```text
hours: 1..168
```

## `get_recent_alerts`

Devuelve alertas recientes del activo.

Parámetros:

```json
{
  "asset_id": "srv-web-01",
  "minutes": 240
}
```

Rango permitido:

```text
minutes: 1..1440
```

Una lista vacía es un resultado válido.

## `get_open_incidents`

Devuelve incidentes actualmente abiertos para el activo.

Parámetros:

```json
{
  "asset_id": "srv-web-01"
}
```

## Herramientas que NO existen

El MCP no expone herramientas como:

```text
restart_server
rollback_deployment
change_firewall
disable_account
modify_configuration
```

Por tanto, el Agent puede **investigar y recomendar**, pero no remediar.

## Importante

Los resultados operativos pueden cambiar con el tiempo. El criterio de éxito no es obtener siempre el mismo JSON, sino comprobar que:

1. el Agent selecciona herramientas relevantes;
2. utiliza argumentos correctos;
3. interpreta los resultados recibidos;
4. diferencia hechos de hipótesis;
5. no inventa datos ni acciones.
