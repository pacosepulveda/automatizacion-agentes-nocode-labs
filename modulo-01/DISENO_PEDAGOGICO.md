# M01 — Diseñando nuestra automatización inteligente

## Objetivo funcional

**Antes de M01:** Telvora tiene un proceso manual de análisis de incidencias.

**Después de M01:** queda definido qué partes automatizaremos, qué corresponde a workflow, IA, agente o humano y cuál es la arquitectura inicial que empezará a implementarse en M02.

## Duración objetivo

Entre **1 h 50 min y 2 h**, con aproximadamente 60–70 minutos de trabajo práctico.

## Escenario inicial

Una alerta de ejemplo:

```json
{
  "alert_id": "ALT-1001",
  "timestamp": "2026-11-04T08:17:13Z",
  "asset_id": "srv-web-01",
  "severity": "high",
  "summary": "Intermittent HTTP 502 responses detected after deployment"
}
```

Pregunta inicial:

> ¿Qué haría ahora un operador?

La discusión debe hacer aparecer naturalmente necesidades como:

- identificar el activo;
- determinar si es producción;
- conocer criticidad;
- revisar cambios;
- comprobar estado;
- consultar incidencias;
- consultar procedimientos;
- formular hipótesis;
- decidir siguiente paso;
- identificar acciones que requieren autorización.

## Práctica 1 — Resolver manualmente el caso

Materiales previstos:

```text
alertas_iniciales.json
catalogo_activos.csv
```

El alumno realiza manualmente:

```text
alerta
  ↓
comprobar datos
  ↓
buscar asset_id
  ↓
obtener environment / criticality / owner_team
  ↓
determinar prioridad
```

Esto permitirá que en M02 comprenda qué trabajo está automatizando.

## Clasificación conceptual

El alumno clasifica tareas según el mecanismo más apropiado:

| Tarea | Enfoque esperado |
|---|---|
| Validar campos | Workflow |
| Consultar activo | API |
| Calcular P1–P4 | Workflow |
| Interpretar texto ambiguo | LLM |
| Buscar procedimiento interno | RAG |
| Decidir qué fuentes investigar | Agent |
| Consultar estado/incidentes/cambios | Tools |
| Formular hipótesis | LLM / Agent |
| Aprobar acción sensible | Humano |

Conceptos que se introducen brevemente:

```text
WORKFLOW
LLM
RAG
AGENT
HUMANO
```

## Workflow vs Agent

Ejemplo determinista:

```text
severity = critical
→ P1
```

Ejemplo agentic:

```text
Investiga la causa probable de los errores 502
→ comprobar health
→ decidir si necesita revisar cambios
→ decidir si necesita revisar incidentes
→ detener investigación cuando haya suficiente contexto
```

## Reglas iniciales P1–P4

```text
critical
→ P1

high + production + criticality high
→ P1

high
→ P2

medium
→ P3

low
→ P4
```

Excepciones mínimas:

```text
campo obligatorio ausente
→ revisión

asset desconocido
→ revisión

severity desconocida
→ revisión
```

## Autoridad de las decisiones

```text
P1–P4                  → workflow determinista
datos del activo       → sistema/API
procedimientos         → documentación/RAG
hipótesis              → IA/Agent
acción sensible        → humano
```

Pregunta clave:

> Si el LLM propone P3 pero las reglas deterministas indican P1, ¿qué resultado debe prevalecer?

## Práctica 2 — Arquitectura funcional

Piezas:

```text
MONITORING
WORKFLOW
ASSET API
LLM
DOCUMENTACIÓN
AGENT
TOOLS
HUMANO
```

Arquitectura objetivo conceptual:

```text
MONITORING
    |
    v
AUTOMATION
    |
    +--> ASSET DATA
    |
    +--> PRIORITY
    |
    v
   IA
  /  \
RAG  AGENT
 |     |
docs  tools
       |
       +-- health
       +-- changes
       `-- incidents
       |
       v
recommendation
       |
       v
    HUMANO
```

Todavía no se introducen detalles de implementación como n8n, Dify, MCP o Function Calling.

## Resultado final de M01

El alumno debe poder explicar:

> Las reglas y las integraciones deterministas irán en el workflow. Utilizaremos IA donde haya que interpretar información. El agente se utilizará cuando necesitemos decidir qué herramientas consultar para investigar. Las acciones sensibles seguirán requiriendo intervención humana.

## Conexión directa con M02

M01 termina en:

```text
ALERTA
  ↓
¿válida?
  ↓
buscar activo
  ↓
calcular prioridad
```

Cierre:

> Ya sabemos hacer esta parte manualmente. En M02 vamos a conseguir que n8n la haga por nosotros.
