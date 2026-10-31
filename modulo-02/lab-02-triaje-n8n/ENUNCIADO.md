# Laboratorio 2 — Primer workflow de triaje en n8n

## Contexto

En M01 analizaste manualmente varias alertas de Telvora y aplicaste las reglas iniciales de prioridad.

Ahora automatizarás esa primera parte del trabajo con **n8n**.

Todavía no recibiremos eventos reales ni consultaremos APIs externas. Para concentrarnos en cómo funciona un workflow, la alerta de prueba incluirá temporalmente también los datos del activo (`environment`, `criticality` y `owner_team`). En M03 esos campos dejarán de venir en la entrada y n8n tendrá que obtenerlos desde otro sistema.

> Todos los datos utilizados son ficticios.

## Objetivos

Al finalizar deberías ser capaz de:

- crear un workflow en n8n;
- comprender cómo pasan los datos JSON de un nodo a otro;
- utilizar `Manual Trigger` y `Edit Fields (Set)`;
- leer campos mediante expresiones;
- validar datos obligatorios;
- utilizar `If` y `Switch` para crear rutas;
- implementar reglas deterministas P1–P4;
- generar una salida estructurada;
- inspeccionar ejecuciones y localizar un error lógico sencillo.

## Material proporcionado

En `materiales/` encontrarás:

- `casos_prueba.json`;
- `resultados_esperados.md`.

---

# Parte 1 — Crear el workflow

Crea un workflow nuevo con un nombre descriptivo, por ejemplo:

```text
M02-Triage-Alertas
```

Añade:

```text
Manual Trigger
      ↓
Edit Fields (Set)
```

Renombra el segundo nodo como:

```text
01 - Entrada de prueba
```

Introduce inicialmente el caso `M02-01` de `casos_prueba.json`.

Ejecuta el workflow e inspecciona el output del nodo.

## Pregunta

¿Qué estructura reciben los nodos de n8n cuando trabajamos con una alerta en formato JSON?

---

# Parte 2 — Validar campos obligatorios

Los campos mínimos son:

```text
alert_id
timestamp
asset_id
severity
summary
```

Añade una validación que separe:

```text
entrada válida
```

de:

```text
revisión manual
```

Puedes resolverlo con uno o varios nodos `If`, según la estrategia que prefieras.

Para la ruta de revisión manual crea un resultado similar a:

```json
{
  "status": "manual_review",
  "requires_human_review": true,
  "reason": "missing_required_field"
}
```

No necesitas construir un sistema completo de validación. El objetivo es impedir que una alerta incompleta continúe hacia la clasificación automática.

---

# Parte 3 — Validar severity

Solo se admiten estos valores:

```text
low
medium
high
critical
```

Cualquier otro valor debe ir a revisión manual.

Resultado orientativo:

```json
{
  "status": "manual_review",
  "requires_human_review": true,
  "reason": "invalid_severity"
}
```

---

# Parte 4 — Implementar las reglas P1–P4

Utiliza las reglas definidas en M01.

## P1

Asignar `P1` cuando:

```text
severity = critical
```

O cuando:

```text
severity = high
AND environment = production
AND criticality = high
```

## P2

Asignar `P2` cuando:

```text
severity = high
```

y no se cumpla la condición de P1.

## P3

```text
severity = medium
→ P3
```

## P4

```text
severity = low
→ P4
```

Puedes utilizar una combinación de `If`, `Switch` y `Edit Fields (Set)`.

No utilices código si puedes resolverlo con nodos visuales.

---

# Parte 5 — Generar un resultado coherente

Todas las rutas válidas deben dejar campos equivalentes a estos:

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

Para este prototipo:

```text
P1 → requires_human_review = true
P2/P3/P4 → requires_human_review = false
```

Las rutas de error también deben marcar revisión humana.

---

# Parte 6 — Probar todas las rutas

Abre `casos_prueba.json`.

Antes de ejecutar cada caso, intenta predecir el resultado.

Prueba como mínimo:

- `M02-01` — critical;
- `M02-02` — high + producción + criticidad alta;
- `M02-03` — high fuera de P1;
- `M02-04` — medium;
- `M02-05` — low;
- `M02-06` — severity no válida;
- `M02-07` — falta `asset_id`;
- `M02-08` — falta `summary`.

Compara después con `resultados_esperados.md`.

---

# Parte 7 — Depurar una clasificación incorrecta

Introduce temporalmente un error sencillo en la condición que distingue P1 de P2 para alertas `high`.

Por ejemplo, configura la condición de modo que `M02-02` termine incorrectamente en `P2`.

Después:

1. ejecuta `M02-02`;
2. comprueba que el resultado no coincide con lo esperado;
3. inspecciona el input y output de los nodos recorridos;
4. localiza la condición incorrecta;
5. corrígela;
6. vuelve a ejecutar el caso.

El objetivo es comprobar que inspeccionar las ejecuciones forma parte natural de construir automatizaciones.

---

# Resultado del laboratorio

Al terminar deberías tener un workflow equivalente a:

```text
MANUAL TRIGGER
      |
      v
ENTRADA DE PRUEBA
      |
      v
VALIDAR
  /       \
no         sí
|           |
v           v
REVISIÓN   VALIDAR SEVERITY
MANUAL       /       \
           no         sí
           |           |
           v           v
        REVISIÓN    CLASIFICAR
        MANUAL       P1–P4
                       |
                       v
                   RESULTADO
```

Has convertido una regla manual repetitiva en una automatización reproducible.

## Limitación actual

El workflow funciona, pero todavía existen dos artificios:

1. alguien debe iniciar manualmente la ejecución;
2. `environment`, `criticality` y `owner_team` ya vienen incluidos en la entrada.

En M03 eliminaremos esas simplificaciones: la alerta llegará mediante un **Webhook** y n8n consultará los datos del activo mediante una **API REST**.
