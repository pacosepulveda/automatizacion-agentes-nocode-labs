# Laboratorio 1 — Diseñando la automatización

## Contexto

Telvora opera servicios técnicos y recibe alertas generadas por sus sistemas de monitorización. En la actualidad, un operador debe revisar cada alerta, consultar información del activo, valorar su prioridad, investigar posibles causas y decidir el siguiente paso.

Durante el curso iremos automatizando progresivamente este proceso.

En este primer laboratorio no utilizarás todavía n8n ni Dify. Primero realizarás manualmente una parte del trabajo para entender qué estamos automatizando y decidir qué tareas deben corresponder a un workflow, a un LLM, a un agente o a una persona.

> Todos los datos utilizados son ficticios.

## Objetivos

Al finalizar deberías ser capaz de:

- identificar las principales etapas del proceso de triaje e investigación;
- distinguir tareas deterministas de tareas que requieren interpretación;
- identificar qué información debe consultarse en otros sistemas;
- diferenciar el papel de un workflow, un LLM, RAG, un agente y una persona;
- aplicar unas reglas iniciales de prioridad;
- proponer la arquitectura funcional que construiremos durante el curso.

## Material proporcionado

En `materiales/` encontrarás:

- `alertas_iniciales.json`;
- `catalogo_activos.csv`;
- `mapa_solucion.md`.

---

# Parte 1 — Analizar una alerta como operador

Abre `alertas_iniciales.json` y comienza por `ALT-1001`.

Antes de consultar ningún otro archivo, responde:

1. ¿Qué sabes con certeza a partir de la alerta?
2. ¿Qué información adicional necesitarías?
3. ¿Qué comprobarías antes de decidir una acción?
4. ¿Qué partes del análisis podrían automatizarse?

No busques todavía una solución tecnológica. Piensa como un operador que recibe la alerta.

---

# Parte 2 — Enriquecer manualmente la alerta

Abre `catalogo_activos.csv`.

Utiliza `asset_id` para localizar el activo correspondiente y completa para cada alerta válida:

- `asset_name`;
- `environment`;
- `criticality`;
- `owner_team`.

Anota también qué ocurre cuando el activo no existe en el catálogo.

## Preguntas

- ¿Por qué no sería buena idea copiar permanentemente toda la información del activo dentro de cada alerta?
- ¿Qué ventaja tendría consultar el catálogo automáticamente?
- ¿Qué debería ocurrir si no encontramos el activo?

---

# Parte 3 — Aplicar las reglas iniciales de prioridad

Utilizaremos estas reglas durante los primeros módulos:

## P1

Asignar `P1` cuando:

- `severity = critical`; o
- `severity = high`, `environment = production` y `criticality = high`.

## P2

Asignar `P2` cuando:

- `severity = high` y no se cumplen las condiciones de P1.

## P3

Asignar `P3` cuando:

- `severity = medium`.

## P4

Asignar `P4` cuando:

- `severity = low`.

Enviar a revisión cuando:

- falte un campo obligatorio;
- el activo no exista;
- `severity` no sea `low`, `medium`, `high` o `critical`.

Completa en `mapa_solucion.md` el resultado para las alertas proporcionadas.

---

# Parte 4 — ¿Workflow, IA, agente o humano?

Clasifica estas tareas indicando el mecanismo que utilizarías principalmente:

- comprobar campos obligatorios;
- consultar los datos de un activo;
- calcular P1–P4;
- interpretar un resumen técnico ambiguo;
- localizar un procedimiento interno;
- decidir qué fuentes consultar durante una investigación;
- comprobar el estado actual de un servicio;
- consultar cambios recientes;
- formular una hipótesis;
- reiniciar un servicio de producción;
- autorizar un rollback.

Utiliza estas categorías:

```text
WORKFLOW
API / TOOL
LLM
RAG
AGENT
HUMANO
```

No todas las tareas tienen una única respuesta posible. Justifica especialmente aquellas en las que dudes.

---

# Parte 5 — Workflow frente a agente

Compara estas dos tareas:

### Caso A

```text
Si severity = critical → priority = P1
```

### Caso B

```text
Investiga qué puede estar provocando los errores 502 de srv-web-01.
```

Responde:

- ¿en cuál conocemos de antemano todos los pasos?
- ¿en cuál puede ser necesario decidir dinámicamente qué consultar?
- ¿cuál resolverías con un workflow?
- ¿en cuál podría aportar valor un agente?

---

# Parte 6 — Diseñar la solución que construiremos

En `mapa_solucion.md` organiza estas piezas:

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

La solución futura deberá ser capaz de:

1. recibir una alerta;
2. validarla;
3. consultar información del activo;
4. calcular prioridad;
5. interpretar contexto técnico;
6. consultar documentación interna;
7. investigar información operativa mediante herramientas;
8. generar una recomendación;
9. mantener las acciones sensibles bajo control humano.

No necesitas conocer todavía cómo se implementará cada componente.

---

# Resultado del laboratorio

Al terminar deberías poder explicar por qué:

- las reglas P1–P4 deben ser deterministas;
- el LLM puede ayudar a interpretar información ambigua;
- RAG permitirá utilizar documentación interna;
- un agente podrá decidir qué herramientas consultar durante una investigación;
- una recomendación del agente no equivale automáticamente a una autorización para ejecutar una acción sensible.

En el siguiente módulo automatizaremos con n8n la primera parte del proceso que acabas de realizar manualmente.
