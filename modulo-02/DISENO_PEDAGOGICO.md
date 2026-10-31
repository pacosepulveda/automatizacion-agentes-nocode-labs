# M02 — Primera automatización real con n8n

## Objetivo funcional

**Antes de M02:** el alumno sabe cómo se clasifica una alerta, pero el proceso sigue siendo manual.

**Después de M02:** n8n recibe una alerta de prueba, valida los datos mínimos, aplica las reglas P1–P4 y devuelve un resultado estructurado indicando prioridad y si requiere revisión manual.

El propósito no es aprender n8n en abstracto. El propósito es automatizar la primera parte del trabajo manual definido en M01.

## Duración objetivo

Entre **1 h 50 min y 2 h**, con aproximadamente un **70 % de construcción y prueba**.

Distribución orientativa:

| Bloque | Tiempo |
|---|---:|
| Retomar el proceso manual de M01 | 10 min |
| Crear el primer workflow y comprender items/JSON | 15 min |
| Validar entrada | 20 min |
| Implementar clasificación P1–P4 | 30 min |
| Probar casos y depurar | 25 min |
| Consolidar resultado y preparar M03 | 15 min |

## Problema que resolvemos

En M01 el alumno realizó manualmente:

```text
alerta
  ↓
comprobar datos
  ↓
consultar contexto del activo
  ↓
calcular prioridad
```

En M02 automatizamos esa lógica con n8n.

Todavía no utilizamos Webhooks ni APIs externas. Para concentrarnos en la lógica del workflow, la entrada de prueba incluirá temporalmente los datos de contexto que en M03 obtendremos mediante una API.

## Arquitectura de M02

```text
MANUAL TRIGGER
      |
      v
ENTRADA DE PRUEBA
      |
      v
VALIDAR CAMPOS
   /       \
 no         sí
 |           |
 v           v
REVISIÓN    CLASIFICAR P1–P4
MANUAL           |
                 v
            RESULTADO
```

## Datos de entrada

Durante M02 cada caso puede incluir:

```json
{
  "alert_id": "ALT-2001",
  "timestamp": "2026-11-04T08:17:13Z",
  "asset_id": "srv-web-01",
  "severity": "high",
  "summary": "Intermittent HTTP 502 responses detected after deployment",
  "environment": "production",
  "criticality": "high",
  "owner_team": "WebOps"
}
```

Los campos `environment`, `criticality` y `owner_team` son deliberadamente temporales.

En M03 desaparecerán de la entrada y se recuperarán desde la API de activos.

## Capacidades de n8n que se introducen

Solo las necesarias para construir la solución:

- Manual Trigger;
- Edit Fields / Set;
- lectura de JSON;
- expresiones;
- If;
- Switch;
- ramas;
- inspección de inputs y outputs;
- historial / ejecución del workflow;
- depuración básica.

No se introduce código ni nodos avanzados si no son necesarios.

## Parte 1 — Crear el workflow

Nombre recomendado:

```text
M02-Triage-Alertas
```

Primero se crea:

```text
Manual Trigger
      ↓
Edit Fields / Set
```

El alumno introduce un único caso de prueba y ejecuta el flujo.

El objetivo inicial es comprender una idea simple:

> n8n transporta datos estructurados entre nodos y cada nodo recibe un input y produce un output.

No se dedica una explicación larga previa a la arquitectura interna de n8n.

## Parte 2 — Validar antes de clasificar

Campos obligatorios mínimos:

```text
alert_id
timestamp
asset_id
severity
summary
```

Se construye una decisión que separe:

```text
entrada válida
```

de:

```text
revisión manual
```

La idea que se refuerza es:

> no automatizamos una decisión si la entrada no contiene la información mínima necesaria.

No es necesario diseñar un framework de validación completo.

## Parte 3 — Valores admitidos de severity

Valores válidos:

```text
low
medium
high
critical
```

Cualquier otro valor debe ir a revisión manual.

Esto permite introducir una rama de excepción sin convertir el módulo en un laboratorio de gestión de errores.

## Parte 4 — Implementar P1–P4

Reglas heredadas de M01:

### P1

```text
severity = critical
```

O:

```text
severity = high
AND environment = production
AND criticality = high
```

### P2

```text
severity = high
```

cuando no se cumpla la condición de P1.

### P3

```text
severity = medium
```

### P4

```text
severity = low
```

El alumno debe construir estas rutas con nodos visuales y expresiones, no copiando código.

## Parte 5 — Resultado estructurado

Todas las rutas válidas deben terminar dejando una salida coherente, por ejemplo:

```json
{
  "alert_id": "ALT-2001",
  "asset_id": "srv-web-01",
  "severity": "high",
  "priority": "P1",
  "owner_team": "WebOps",
  "requires_human_review": true,
  "classification_reason": "High severity on a high-criticality production asset"
}
```

No se busca todavía un contrato exhaustivo.

El objetivo es que el alumno vea que el workflow no solo ejecuta ramas: **produce un resultado útil para otra etapa del proceso**.

## Revisión humana mínima

En M02 basta con estas reglas:

```text
P1
→ requires_human_review = true

entrada inválida
→ requires_human_review = true

severity desconocida
→ requires_human_review = true
```

El resto puede quedar clasificado automáticamente.

No se introduce todavía una política sofisticada de Human-in-the-Loop.

## Casos de prueba

El material debe incluir unos 7–8 casos que obliguen a recorrer las distintas rutas.

Ejemplos:

| Caso | Situación | Resultado esperado |
|---|---|---|
| M02-01 | critical | P1 |
| M02-02 | high + production + criticality high | P1 |
| M02-03 | high + development | P2 |
| M02-04 | medium | P3 |
| M02-05 | low | P4 |
| M02-06 | severity desconocida | revisión manual |
| M02-07 | falta asset_id | revisión manual |
| M02-08 | falta summary | revisión manual |

Antes de ejecutar cada caso, el alumno debe predecir qué debería ocurrir.

Esto evita que la práctica se convierta en pulsar `Execute` sin razonar.

## Depuración integrada

La depuración debe aparecer como una necesidad normal de construir workflows, no como una disciplina separada.

Se puede introducir un error deliberado sencillo, por ejemplo:

```text
high + production + criticality high
```

termina inicialmente en P2 por una condición mal configurada.

El alumno debe:

1. ejecutar el caso;
2. observar que el resultado no coincide con lo esperado;
3. inspeccionar input/output de los nodos;
4. localizar la condición incorrecta;
5. corregirla;
6. volver a ejecutar el caso.

Objetivo:

> aprender que probar y observar ejecuciones forma parte natural de construir una automatización.

No se introduce testing formal ni regresión extensa en este módulo.

## Qué no añadimos todavía

M02 no debe incluir:

- Webhooks;
- HTTP Request;
- APIs externas;
- credenciales;
- Dify;
- LLM;
- RAG;
- agentes;
- MCP;
- autenticación;
- observabilidad avanzada;
- contratos extensos;
- gestión compleja de errores.

Todo eso aparecerá cuando la evolución funcional de la solución lo requiera.

## Materiales previstos

Estructura provisional:

```text
modulo-02/
├── DISENO_PEDAGOGICO.md
└── lab-02-triaje-n8n/
    ├── ENUNCIADO.md
    └── materiales/
        ├── casos_prueba.json
        └── resultados_esperados.md
```

`resultados_esperados.md` puede utilizarse durante la corrección o liberarse después de la práctica. No es necesario añadir una checklist extensa.

## Qué queda funcionando al terminar M02

```text
ALERTA SIMULADA
      ↓
     n8n
      ↓
validación
      ↓
P1 / P2 / P3 / P4
      ↓
revisión humana cuando proceda
      ↓
resultado estructurado
```

El alumno ha automatizado realmente una parte del trabajo que realizó manualmente en M01.

## Conexión directa con M03

El punto débil queda visible:

> El workflow funciona, pero alguien sigue introduciendo la alerta manualmente y además le estamos proporcionando `environment`, `criticality` y `owner_team` como si ya los conociéramos.

M03 parte exactamente de esa limitación.

La pregunta de cierre es:

> ¿Cómo hacemos que una alerta llegue automáticamente desde otro sistema y que n8n consulte por sí mismo los datos del activo?

Respuesta que desencadena M03:

```text
Webhook + API REST
```

## Criterio de éxito de M02

El alumno debe ser capaz de mirar una ejecución y explicar:

- qué datos entraron;
- qué validación se realizó;
- por qué se tomó una rama concreta;
- por qué se asignó P1, P2, P3 o P4;
- qué resultado estructurado produjo el workflow;
- por qué ciertos casos requieren revisión humana.

No se evalúa la cantidad de nodos ni la complejidad visual del workflow.

La idea fundamental es:

> **una regla manual repetitiva ya se ha convertido en una automatización reproducible.**