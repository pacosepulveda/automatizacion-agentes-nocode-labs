# Laboratorio 2 — Primer workflow de triaje en n8n

## Contexto

En el Laboratorio 1 diseñaste el proceso de **triaje y enriquecimiento de alertas técnicas de infraestructura** sin implementarlo todavía en ninguna herramienta.

Ahora vamos a trasladar una parte de ese diseño a **n8n**.

El objetivo de este laboratorio no es conectar todavía sistemas reales. No utilizaremos webhooks, APIs externas ni Dify. Primero necesitamos comprender bien cómo funciona un workflow visual:

- cómo entra un item en el flujo;
- cómo se transforman sus datos;
- cómo se evalúan condiciones;
- cómo se crean rutas alternativas;
- cómo se inspecciona la ejecución;
- cómo se detecta un error lógico;
- cómo se comprueba que el resultado es correcto.

En el siguiente módulo sustituiremos la entrada simulada por un evento real de laboratorio y consultaremos la información del activo mediante una API.

> **Importante:** todos los datos son ficticios. No introduzcas credenciales reales, datos productivos ni información sensible.

---

# Objetivos de aprendizaje

Al terminar este laboratorio deberías ser capaz de:

- Crear y nombrar correctamente un workflow en n8n.
- Diferenciar entre trigger, nodo de transformación y nodo de decisión.
- Comprender que n8n transporta items JSON entre nodos.
- Introducir una entrada estructurada de prueba.
- Leer los datos de entrada y salida de cada nodo.
- Validar campos obligatorios sin programar.
- Utilizar un nodo **If** para tomar una decisión binaria.
- Utilizar un nodo **Switch** para crear varias rutas.
- Añadir y conservar campos mediante **Edit Fields (Set)**.
- Implementar reglas deterministas de clasificación.
- Diseñar una ruta de revisión manual para entradas problemáticas.
- Probar varios casos de entrada de forma sistemática.
- Revisar ejecuciones y localizar el punto donde aparece un error.
- Corregir un error y volver a ejecutar una prueba de regresión.

---

# Prerrequisitos

Necesitas:

- acceso al entorno n8n proporcionado para el curso;
- navegador actualizado;
- haber revisado el Laboratorio 1;
- los archivos incluidos en `materiales/`.

No necesitas conocimientos de programación.

La interfaz de n8n puede cambiar ligeramente entre versiones. Los nombres conceptuales de los nodos utilizados en este laboratorio son:

- **Manual Trigger**;
- **Edit Fields (Set)**;
- **If**;
- **Switch**.

Si alguna etiqueta de la interfaz difiere ligeramente, utiliza el nodo equivalente.

---

# Material proporcionado

En `materiales/` encontrarás:

- `casos_prueba.json`: ocho entradas con las que validarás el workflow;
- `checklist_pruebas.md`: plantilla para documentar resultados y evidencias.

El archivo de casos contiene tanto entradas correctas como entradas problemáticas.

No mires los casos únicamente como “datos para copiar”. Antes de ejecutar cada uno intenta predecir qué debería ocurrir. Esa predicción es parte del ejercicio.

---

# Arquitectura que vamos a construir

La versión simplificada de este módulo será:

```text
MANUAL TRIGGER
      |
      v
ENTRADA SIMULADA
      |
      v
VALIDACIÓN DE CAMPOS
      |
      +--------- inválido ---------> REVISIÓN MANUAL
      |
      v
SWITCH POR SEVERIDAD
      |
      +---- critical -------------> P1
      |
      +---- high ----> evaluar activo ----> P1 o P2
      |
      +---- medium ----------------> P3
      |
      +---- low -------------------> P4
      |
      +---- otro ------------------> REVISIÓN MANUAL
```

Todavía no consultaremos el catálogo de activos mediante una API. Para poder centrarnos en la lógica de n8n, cada caso de prueba incluye temporalmente:

- `environment`;
- `criticality`;
- `owner_team`.

En el siguiente laboratorio esos campos dejarán de venir en la entrada y deberán obtenerse desde otro sistema.

---

# Reglas del proceso

Utilizaremos las mismas reglas definidas en el Laboratorio 1.

## Campos obligatorios

Como mínimo deben existir y no estar vacíos:

- `alert_id`
- `timestamp`
- `asset_id`
- `severity`
- `summary`

Si falta cualquiera de ellos, la alerta debe ir a **revisión manual**.

## Valores admitidos de severidad

Solo aceptamos:

- `low`
- `medium`
- `high`
- `critical`

Cualquier otro valor debe enviarse a **revisión manual**.

## Prioridad P1

La alerta debe clasificarse como `P1` cuando:

- `severity = critical`; o
- `severity = high`, `environment = production` y `criticality = high`.

## Prioridad P2

La alerta debe clasificarse como `P2` cuando:

- `severity = high` y no se cumplen todas las condiciones de P1.

## Prioridad P3

La alerta debe clasificarse como `P3` cuando:

- `severity = medium`.

## Prioridad P4

La alerta debe clasificarse como `P4` cuando:

- `severity = low`.

## Supervisión humana

En este prototipo:

- las alertas P1 deben quedar marcadas para revisión humana;
- las alertas P2, P3 y P4 pueden quedar clasificadas automáticamente;
- cualquier error de validación debe ir a revisión manual;
- ninguna ruta realizará acciones de remediación.

---

# Parte 1 — Crear el workflow

Accede al entorno n8n facilitado para el curso y crea un workflow nuevo.

Utiliza un nombre descriptivo. Por ejemplo:

```text
M02-LAB02-Triage-Alertas
```

Evita nombres como:

```text
Workflow 1
Test
Nuevo workflow
Prueba final 2
```

En un entorno real pueden existir decenas o cientos de workflows. El nombre debe permitir identificar rápidamente:

- el módulo o proyecto;
- el propósito;
- el proceso que implementa.

## Pregunta

¿Por qué puede convertirse el nombre de un workflow en un problema operativo meses después de haberlo creado?

---

# Parte 2 — Añadir el trigger

Añade un nodo **Manual Trigger**.

Este nodo inicia el workflow únicamente cuando una persona solicita una ejecución desde el editor.

En este módulo utilizamos una ejecución manual porque queremos:

- controlar exactamente cuándo se ejecuta el flujo;
- probar entradas concretas;
- observar cada paso;
- evitar recibir eventos externos mientras todavía estamos construyendo.

Más adelante sustituiremos este trigger por uno basado en eventos.

Renombra el nodo como:

```text
00 - Inicio manual
```

## Pregunta

¿Qué diferencia conceptual existe entre un workflow manual y uno que se ejecuta automáticamente al recibir un webhook?

---

# Parte 3 — Crear una entrada simulada

Añade después del trigger un nodo **Edit Fields (Set)**.

Renómbralo:

```text
01 - Entrada simulada
```

Este nodo representará temporalmente al sistema de monitorización que, en una arquitectura real, enviaría la alerta.

Abre `materiales/casos_prueba.json` y localiza `CP-01`.

Introduce sus campos en el nodo.

Puedes hacerlo de dos formas dependiendo de la interfaz disponible:

1. **Manual Mapping**, creando los campos uno a uno; o
2. **JSON Output**, copiando un objeto JSON.

No copies el array completo del archivo. Trabajaremos con **un caso por ejecución**.

Ejecuta el nodo o el workflow y observa la salida JSON.

Deberías reconocer conceptos como:

```text
campo -> valor
severity -> high
environment -> production
criticality -> high
```

## Reflexión

En n8n los nodos no “se pasan pantallas”. Se pasan datos estructurados.

Comprueba que puedes responder:

- ¿qué JSON entra en el siguiente nodo?
- ¿qué tipo de dato tiene cada campo?
- ¿qué ocurriría si un campo no existiera?

---

# Parte 4 — Validar los campos obligatorios

Añade un nodo **If** después de `01 - Entrada simulada`.

Renómbralo:

```text
02 - Campos obligatorios presentes
```

Configura las condiciones necesarias para comprobar que los cinco campos obligatorios existen y no están vacíos.

La salida verdadera debe representar:

```text
entrada válida
```

La salida falsa debe representar:

```text
entrada incompleta
```

## Requisito

No debes comprobar únicamente `asset_id`.

El workflow debe validar como mínimo:

- `alert_id`
- `timestamp`
- `asset_id`
- `severity`
- `summary`

## Pregunta

¿Por qué debemos validar la entrada antes de aplicar reglas de clasificación?

---

# Parte 5 — Crear la ruta de error de validación

Conecta la salida falsa del nodo anterior a un nuevo **Edit Fields (Set)**.

Renómbralo:

```text
90 - Revisión manual - Entrada inválida
```

Añade al resultado, como mínimo:

```text
status = manual_review
reason = missing_required_field
requires_human_review = true
```

Conserva también los datos originales de la alerta siempre que sea posible.

Este último detalle es importante.

Si el flujo elimina la entrada original y conserva únicamente `status` y `reason`, el operador que reciba el caso perderá información necesaria para investigar el problema.

## Pregunta

¿Qué datos mínimos deberíamos conservar para poder investigar posteriormente por qué una alerta fue rechazada?

---

# Parte 6 — Separar las rutas por severidad

Desde la salida verdadera de `02 - Campos obligatorios presentes`, añade un nodo **Switch**.

Renómbralo:

```text
03 - Enrutar por severidad
```

Utiliza `severity` como valor de decisión.

Crea rutas para:

- `critical`
- `high`
- `medium`
- `low`

Además, configura una ruta de fallback para cualquier valor que no coincida con los anteriores.

La existencia del fallback es obligatoria.

Sin fallback, una entrada desconocida podría dejar de producir salida y ser más difícil de diagnosticar.

Renombra las salidas si tu versión de n8n lo permite.

## Pregunta

¿Por qué utilizamos un Switch y no encadenamos cuatro nodos If consecutivos?

---

# Parte 7 — Gestionar una severidad desconocida

Conecta el fallback del Switch a otro nodo **Edit Fields (Set)**.

Renómbralo:

```text
91 - Revisión manual - Severidad inválida
```

El resultado debe incluir al menos:

```text
status = manual_review
reason = invalid_severity
requires_human_review = true
```

Conserva los datos originales.

---

# Parte 8 — Clasificar critical

La ruta `critical` es directa.

Conéctala a un nodo **Edit Fields (Set)** llamado:

```text
10 - Asignar P1 Critical
```

Añade:

```text
priority = P1
status = pending_human_review
requires_human_review = true
```

Conserva los demás campos.

No ejecutes ninguna remediación.

---

# Parte 9 — Clasificar high

La ruta `high` necesita una decisión adicional.

Una alerta `high` será P1 únicamente si:

```text
environment = production
AND
criticality = high
```

Añade un nodo **If** y renómbralo:

```text
11 - High sobre activo crítico de producción
```

Configura las dos condiciones anteriores con operador lógico **AND**.

## Rama verdadera

Conéctala a:

```text
12 - Asignar P1 High
```

Añade:

```text
priority = P1
status = pending_human_review
requires_human_review = true
```

## Rama falsa

Conéctala a:

```text
13 - Asignar P2
```

Añade:

```text
priority = P2
status = classified
requires_human_review = false
```

---

# Parte 10 — Clasificar medium y low

## Medium

Conecta la ruta `medium` del Switch a:

```text
20 - Asignar P3
```

Añade:

```text
priority = P3
status = classified
requires_human_review = false
```

## Low

Conecta la ruta `low` del Switch a:

```text
30 - Asignar P4
```

Añade:

```text
priority = P4
status = classified
requires_human_review = false
```

---

# Parte 11 — Crear una salida estructurada

Queremos que todas las rutas de negocio produzcan un resultado comprensible.

Puedes conectar las rutas finales a un nodo común llamado:

```text
99 - Resultado
```

No utilices este nodo para recalcular la prioridad.

Su función debe ser únicamente normalizar o completar la salida final, por ejemplo añadiendo:

```text
workflow_stage = triage
workflow_version = m02-v1
```

Opcionalmente puedes añadir una marca temporal de procesamiento usando una expresión de n8n.

Si decides no utilizar un nodo final común, asegúrate de que todas las rutas producen una estructura coherente.

## Requisito de trazabilidad

La salida final debe permitir conocer al menos:

- qué alerta se procesó;
- qué activo estaba implicado;
- qué severidad tenía;
- qué prioridad se calculó cuando procede;
- si necesita revisión humana;
- por qué fue desviada a revisión manual cuando procede.

---

# Parte 12 — Revisar visualmente el diseño

Antes de probar todos los casos, comprueba que el workflow es legible.

Una posible disposición conceptual es:

```text
[00 Inicio manual]
        |
[01 Entrada simulada]
        |
[02 Campos presentes?]
    /           \
  no             sí
  |               |
[90 Manual]   [03 Switch severity]
                  |
     +------------+------------+-----------+
     |            |            |           |
 critical       high         medium       low
     |            |            |           |
   [P1]      [11 ¿prod+high?] [P3]        [P4]
                  /     \
                sí       no
                |         |
              [P1]      [P2]

Fallback del Switch -> [91 Revisión manual]
```

Evita cruces innecesarios de conexiones y nombres genéricos.

## Pregunta

¿Qué información aporta el diagrama del workflow que no aporta únicamente el código JSON de entrada y salida?

---

# Parte 13 — Predecir antes de ejecutar

Abre `materiales/checklist_pruebas.md`.

Para cada caso `CP-01` a `CP-08`, rellena **antes de ejecutar**:

- resultado esperado;
- ruta esperada.

No ejecutes primero y rellenes después.

Queremos comparar nuestro modelo mental con el comportamiento real.

Esta práctica es muy importante en automatización: una prueba solo es realmente útil cuando existe una expectativa previa contra la que comparar.

---

# Parte 14 — Ejecutar los ocho casos

Para probar un caso:

1. abre `casos_prueba.json`;
2. copia los datos del caso elegido;
3. sustituye los datos del nodo `01 - Entrada simulada`;
4. ejecuta el workflow manualmente;
5. observa qué nodos han sido recorridos;
6. inspecciona la salida de los nodos relevantes;
7. registra el resultado en el checklist.

Repite el proceso para los ocho casos.

## Durante las pruebas comprueba

- si el If de campos obligatorios se comporta correctamente;
- si el Switch selecciona exactamente una ruta útil;
- si el fallback captura valores desconocidos;
- si la lógica P1/P2 de `high` utiliza AND y no OR;
- si se conservan los datos originales;
- si `requires_human_review` es coherente;
- si las rutas problemáticas incluyen un motivo explícito.

---

# Parte 15 — Analizar los datos entre nodos

Selecciona al menos tres ejecuciones distintas.

En cada una, inspecciona:

1. salida de `01 - Entrada simulada`;
2. entrada y salida de `02 - Campos obligatorios presentes`;
3. entrada y salida de `03 - Enrutar por severidad`;
4. nodo que asigna la prioridad;
5. resultado final.

## Preguntas

- ¿El If modifica los datos o únicamente decide por qué salida continúan?
- ¿El Switch modifica los datos o únicamente enruta?
- ¿Qué nodo añade `priority`?
- ¿En qué punto aparece por primera vez `requires_human_review`?
- ¿Qué ocurre con un campo si un Edit Fields no conserva los datos de entrada?

---

# Parte 16 — Revisar las ejecuciones

Una ejecución es una instancia concreta del workflow.

Revisa la vista de ejecuciones disponible en tu entorno n8n.

Localiza varias ejecuciones del laboratorio y compara:

- estado;
- instante de ejecución;
- nodos recorridos;
- datos de entrada;
- datos de salida.

Si el entorno no conserva alguna ejecución manual, realiza la inspección inmediatamente después de ejecutar y documenta la evidencia en el checklist.

## Pregunta

¿Por qué es importante conservar trazabilidad de una automatización aunque el resultado final parezca correcto?

---

# Parte 17 — Introducir un error lógico de forma controlada

Ahora vamos a romper el workflow deliberadamente.

> Haz este cambio solo después de haber comprobado que todos los casos funcionan como esperabas.

En `03 - Enrutar por severidad`, cambia temporalmente la regla:

```text
high
```

por:

```text
HIGH
```

Mantén la comparación sensible a mayúsculas y minúsculas si tu versión permite configurarlo.

Ejecuta ahora `CP-01`.

No corrijas el error inmediatamente.

Primero diagnostícalo.

## Debes responder

- ¿Hasta qué nodo funciona correctamente la ejecución?
- ¿Qué valor llega al Switch?
- ¿Qué regla esperaba el Switch?
- ¿Qué salida tomó realmente?
- ¿La ejecución aparece como fallida técnicamente o como ejecución correcta con resultado funcional incorrecto?
- ¿Por qué este último caso puede ser más peligroso que un error que detiene el workflow?

Documenta el diagnóstico en `checklist_pruebas.md`.

---

# Parte 18 — Corregir y realizar una prueba de regresión

Restaura la regla original:

```text
high
```

Vuelve a ejecutar `CP-01`.

Después ejecuta también:

- un caso P1 por severidad `critical`;
- un caso P2;
- un caso inválido.

El objetivo es comprobar que la corrección no ha roto otras rutas.

Esto es una prueba de regresión básica.

---

# Parte 19 — Error técnico opcional

Si tu versión de n8n dispone del nodo **Stop And Error**, puedes realizar este ejercicio adicional.

1. Inserta temporalmente un nodo `Stop And Error` en una de las rutas válidas.
2. Configura un mensaje identificable, por ejemplo:

```text
ERROR_CONTROLADO_M02
```

3. Ejecuta un caso que recorra esa ruta.
4. Observa la diferencia entre:
   - una ejecución técnicamente fallida;
   - una ejecución técnicamente correcta que produce una clasificación equivocada.
5. Elimina después el nodo de error.

No dejes el workflow roto al terminar.

---

# Parte 20 — Revisar criterios de calidad

Antes de considerar terminado el laboratorio, evalúa el workflow con esta lista.

## Legibilidad

- ¿Todos los nodos tienen nombres significativos?
- ¿Las rutas son fáciles de seguir visualmente?
- ¿Los nodos están ordenados de izquierda a derecha o de arriba abajo de forma coherente?

## Datos

- ¿Se conservan los datos necesarios de la alerta?
- ¿Se añaden únicamente campos que tienen un significado claro?
- ¿Los booleanos se almacenan como booleanos y no como textos cuando sea posible?

## Decisiones

- ¿Las reglas están implementadas de forma determinista?
- ¿Existe una ruta de fallback?
- ¿Las entradas incompletas no continúan hacia la clasificación?

## Seguridad

- ¿El workflow evita ejecutar acciones destructivas?
- ¿P1 requiere revisión humana?
- ¿Los errores de entrada se desvían en lugar de ignorarse?

## Operación

- ¿Puedes explicar por qué una alerta tomó una ruta concreta?
- ¿Puedes localizar el nodo donde se produjo una desviación?
- ¿Puedes repetir una prueba después de corregir un error?

---

# Entrega

Entrega:

1. captura o exportación visual del workflow terminado;
2. `checklist_pruebas.md` completado;
3. una breve respuesta a estas preguntas:
   - ¿qué nodos modifican datos y cuáles solo enrutan?
   - ¿qué diferencia observaste entre error lógico y error técnico?
   - ¿por qué es necesaria una ruta de fallback?
   - ¿qué parte del workflow cambiará en el siguiente módulo cuando introduzcamos la API?

Si el instructor solicita una exportación de n8n, exporta únicamente este workflow de laboratorio y revisa antes que no contenga credenciales ni información sensible.

---

# Comprobación final

Deberías ser capaz de explicar de memoria el recorrido de una alerta:

```text
Trigger
  -> datos
  -> validación
  -> decisión por severidad
  -> decisión adicional si high
  -> asignación de prioridad
  -> resultado
```

Y también deberías poder responder:

- ¿qué nodo recibe inicialmente los datos?
- ¿cómo sabes qué datos tiene un nodo?
- ¿qué diferencia existe entre If y Switch?
- ¿dónde se asigna la prioridad?
- ¿qué ocurre con una severidad desconocida?
- ¿qué ocurre si falta `asset_id`?
- ¿por qué una P1 no ejecuta remediación automática?
- ¿cómo localizarías una clasificación incorrecta?

Si puedes responder esas preguntas y los ocho casos de prueba tienen un resultado coherente, el workflow está preparado para evolucionar.

---

# Qué ocurrirá en el siguiente laboratorio

Hasta ahora hemos simulado demasiado:

```text
alerta + environment + criticality + owner_team
```

En una arquitectura real, el sistema de monitorización no debería conocer necesariamente todos esos datos.

En el siguiente módulo evolucionaremos el workflow a algo más realista:

```text
Evento / webhook
      |
      v
Alerta básica
      |
      v
Consulta API de activos
      |
      v
Alerta enriquecida
      |
      v
Clasificación
```

De esta forma separaremos:

- el sistema que **genera el evento**;
- el sistema que **mantiene los datos del activo**;
- el workflow que **orquesta el proceso**.
