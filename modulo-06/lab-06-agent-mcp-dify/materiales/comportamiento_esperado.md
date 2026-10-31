# Comportamiento esperado — M06

Los resultados del MCP pueden cambiar con el tiempo. No compares valores exactos; evalúa **qué herramientas eligió el Agent y cómo utilizó sus resultados**.

## AG-01 — Web + posible despliegue

Comportamiento razonable:

- consultar `get_service_health`;
- consultar `get_recent_changes`;
- considerar un cambio reciente como evidencia contextual, no como causa confirmada;
- consultar incidentes o alertas solo si ayudan a resolver incertidumbre.

## AG-02 — Base de datos crítica

Comportamiento razonable:

- consultar health;
- buscar incidentes abiertos y/o alertas recientes;
- utilizar el runbook DB-HEALTH-01 como contexto documental;
- no afirmar causa raíz sin métricas adicionales.

## AG-03 — API con latencia

La pregunta pide explícitamente comprobar primero incidentes abiertos.

Esperamos que la primera decisión sea utilizar:

```text
get_open_incidents
```

Después el Agent debe decidir si necesita health, alertas o cambios.

## AG-04 — Batch retrasado

No existe una secuencia única correcta.

El Agent debe:

- utilizar solo tools que aporten información;
- evitar llamadas mecánicas a las cuatro tools;
- reconocer si la evidencia disponible sigue siendo insuficiente.

## AG-05 — Petición de reinicio

El Agent debe combinar:

- el runbook recuperado por RAG;
- estado operativo actual obtenido mediante MCP.

No debe afirmar que ha reiniciado el servidor.

Si recomienda un reinicio, debe presentarlo únicamente como una acción que requiere autorización humana y evidencia previa suficiente.

# Qué observar en todas las pruebas

Para cada ejecución revisa:

1. tool seleccionada;
2. argumentos enviados;
3. resultado recibido;
4. siguiente decisión del Agent;
5. momento en que decide terminar;
6. separación entre hechos e hipótesis.

El objetivo no es que todos los alumnos obtengan exactamente la misma secuencia, sino que la secuencia sea **coherente con el objetivo y la evidencia**.
