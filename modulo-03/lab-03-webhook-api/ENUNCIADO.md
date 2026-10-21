# Laboratorio 3 — Eventos, webhooks e integración con sistemas

## Contexto

En el Laboratorio 2 construiste un workflow de n8n que clasificaba alertas, pero la entrada y los datos del activo estaban simulados dentro del propio flujo.

En este laboratorio eliminaremos esas dos simplificaciones:

1. la alerta llegará desde fuera de n8n mediante un **Webhook**;
2. los datos del activo se consultarán mediante una **API REST de laboratorio**.

A partir de este punto el workflow empieza a comportarse como una integración técnica real.

El flujo deberá recibir una alerta externa, validar su estructura, consultar el catálogo de activos mediante HTTP, enriquecer la alerta, aplicar las mismas reglas de prioridad, desviar excepciones a revisión manual, registrar una evidencia y devolver al emisor una respuesta JSON estructurada.

No utilizaremos todavía IA ni Dify. Primero debemos dominar la integración determinista entre sistemas.

> Todos los datos son ficticios. No utilices credenciales reales, datos productivos ni información sensible.

---

## Objetivos de aprendizaje

Al finalizar deberías ser capaz de:

- explicar ejecución manual, programada y disparada por evento;
- crear un Webhook en n8n;
- diferenciar Test URL y Production URL;
- inspeccionar headers, query y body;
- normalizar un payload externo;
- validar antes de llamar a otro sistema;
- consumir una API REST con HTTP Request;
- utilizar una credencial de n8n en lugar de incrustar un token;
- construir URLs dinámicas;
- enriquecer un item con datos devueltos por una API;
- separar errores de integración de excepciones de negocio;
- registrar evidencias;
- responder mediante Respond to Webhook;
- utilizar el historial de ejecuciones para diagnosticar problemas.

---

## Prerrequisitos

Necesitas acceso al n8n del curso, haber revisado M01 y M02, acceso HTTPS a:

```text
https://api.cloudsecurityinpractice.com
https://events.cloudsecurityinpractice.com
```

y un token de API proporcionado por el instructor.

Para enviar peticiones puedes usar `curl`, PowerShell, Postman o equivalente.

---

## Material proporcionado

- `materiales/alertas_webhook.json`
- `materiales/checklist_pruebas.md`
- `materiales/contrato_api.md`
- `materiales/comandos_curl.md`

---

# Arquitectura

```text
SISTEMA EMISOR
     |
     | HTTP POST
     v
n8n WEBHOOK
     |
     v
NORMALIZAR
     |
     v
VALIDAR
  /      \
no        sí
|          |
v          v
MANUAL   HTTP REQUEST -> API ACTIVOS
               |
               v
          ¿ACTIVO EXISTE?
           /         \
         no           sí
         |             |
         v             v
      MANUAL       CLASIFICAR
                       |
                       v
               REGISTRAR EVIDENCIA
                       |
                       v
                RESPONDER HTTP
```

---

# Parte 1 — Comprobar servicios

Abre:

```text
https://api.cloudsecurityinpractice.com/health
```

y:

```text
https://events.cloudsecurityinpractice.com/health
```

Ambos deben responder correctamente.

Opcionalmente observa SSE:

```bash
curl -N https://events.cloudsecurityinpractice.com/test/sse
```

Un webhook y SSE están relacionados con eventos, pero no son iguales: el webhook recibe una petición puntual; SSE mantiene una conexión abierta para recibir eventos.

---

# Parte 2 — Crear una nueva versión

Duplica M02 o crea un workflow nuevo:

```text
M03-LAB03-Webhook-API-Triage
```

No destruyas la versión anterior.

**Pregunta:** ¿por qué conviene conservar una versión funcional antes de añadir una integración?

---

# Parte 3 — Sustituir Manual Trigger por Webhook

Añade **Webhook**:

```text
00 - Recibir alerta
```

Configura:

```text
HTTP Method: POST
Path: m03-alert
Response: Using Respond to Webhook
```

n8n mostrará normalmente una **Test URL** y una **Production URL**.

La Test URL se usa mientras desarrollas y el nodo está escuchando. La Production URL se usa con el workflow activo.

1. Haz que n8n escuche un evento de prueba.
2. Copia la Test URL.
3. Envía EVT-01 usando `materiales/comandos_curl.md`.
4. Inspecciona la salida.

---

# Parte 4 — Comprender la entrada real

El Webhook suele entregar algo similar a:

```json
{
  "headers": {},
  "params": {},
  "query": {},
  "body": {
    "alert_id": "ALT-2001",
    "asset_id": "srv-web-01",
    "severity": "high"
  }
}
```

La alerta puede estar bajo `body`, no en la raíz.

Responde:

- ¿dónde está el body?
- ¿qué aporta `Content-Type`?
- ¿qué diferencia hay entre query string y body?
- ¿qué cabeceras ha recibido n8n?

---

# Parte 5 — Normalizar

Añade **Edit Fields (Set)**:

```text
01 - Normalizar alerta
```

Lleva a la raíz:

```text
alert_id
timestamp
asset_id
severity
source
category
summary
details
```

usando los valores del body.

A partir de aquí el resto del flujo trabajará contra un modelo interno uniforme.

---

# Parte 6 — Validar

Añade **If**:

```text
02 - Campos obligatorios presentes
```

Valida que existan y no estén vacíos:

```text
alert_id
timestamp
asset_id
severity
summary
```

## FALSE

Conecta a:

```text
90 - Revisión manual - Entrada inválida
```

Añade:

```text
status = manual_review
reason = missing_required_field
requires_human_review = true
workflow_version = m03-v1
```

Esta rama **no debe llamar a la API**.

---

# Parte 7 — Crear la credencial

El instructor facilitará un token.

No lo escribas en un Set, URL, nota, repositorio o captura.

Crea una credencial HTTP de n8n, por ejemplo:

```text
M03 - Lab API
```

que envíe:

```http
Authorization: Bearer <token>
```

---

# Parte 8 — Consultar el activo

Desde TRUE de validación añade **HTTP Request**:

```text
03 - Consultar activo
```

Configura:

```text
Method: GET
URL: https://api.cloudsecurityinpractice.com/api/v1/assets/<asset_id>
Authentication: M03 - Lab API
Response: JSON
```

`asset_id` debe insertarse con una expresión.

Para `srv-web-01` la URL final debe ser:

```text
https://api.cloudsecurityinpractice.com/api/v1/assets/srv-web-01
```

La respuesta esperada está documentada en `contrato_api.md`.

---

# Parte 9 — Gestionar activo desconocido

Añade **If**:

```text
04 - Activo encontrado
```

Evalúa `found`.

## FALSE

Conecta a:

```text
91 - Revisión manual - Activo desconocido
```

y genera:

```text
status = manual_review
reason = asset_not_found
requires_human_review = true
workflow_version = m03-v1
```

Conserva los datos originales.

**Importante:** `found=false` no significa que la API esté caída. La API respondió correctamente.

---

# Parte 10 — Construir alerta enriquecida

En TRUE añade:

```text
05 - Alerta enriquecida
```

Combina los datos originales:

```text
alert_id
timestamp
asset_id
severity
source
category
summary
details
```

con los datos devueltos por la API:

```text
asset_name
environment
criticality
owner_team
```

No vuelvas a escribir manualmente valores del catálogo.

---

# Parte 11 — Clasificar

Añade **Switch**:

```text
06 - Enrutar por severidad
```

Rutas:

```text
critical
high
medium
low
fallback
```

Reglas:

- `critical` → P1, revisión humana.
- `high` + `production` + `criticality=high` → P1, revisión humana.
- resto de `high` → P2.
- `medium` → P3.
- `low` → P4.
- fallback → revisión manual por `invalid_severity`.

Para high utiliza un **If**:

```text
07 - High sobre activo crítico de producción
```

con condiciones unidas por **AND**.

---

# Parte 12 — Preparar evidencia

Todas las rutas deben generar una estructura coherente con, al menos:

```text
alert_id
asset_id
severity
priority
status
reason
requires_human_review
owner_team
workflow_version = m03-v1
processed_at
```

No registres tokens, passwords ni cabeceras de autenticación.

---

# Parte 13 — Registrar evidencia

Añade **HTTP Request**:

```text
80 - Registrar evidencia
```

Configura:

```text
Method: POST
URL: https://api.cloudsecurityinpractice.com/api/v1/evidence
Authentication: M03 - Lab API
Body: JSON
```

La API devolverá un `evidence_id`.

**Pregunta:** ¿por qué es útil registrar una decisión fuera del propio workflow?

---

# Parte 14 — Responder al emisor

Añade **Respond to Webhook**:

```text
99 - Responder al emisor
```

Ejemplo de respuesta:

```json
{
  "accepted": true,
  "alert_id": "ALT-2001",
  "status": "pending_human_review",
  "priority": "P1",
  "requires_human_review": true,
  "evidence_id": "ev-..."
}
```

Una entrada inválida puede devolver:

```json
{
  "accepted": false,
  "alert_id": "ALT-2005",
  "status": "manual_review",
  "reason": "missing_required_field",
  "requires_human_review": true,
  "evidence_id": "ev-..."
}
```

`accepted=false` es una decisión funcional, no necesariamente un error HTTP.

---

# Parte 15 — Probar ocho casos

Completa primero la predicción en `checklist_pruebas.md`.

Después ejecuta EVT-01 a EVT-08.

Comprueba para cada uno:

- respuesta recibida por el cliente;
- ruta tomada en n8n;
- llamada o no a la API;
- datos enriquecidos;
- prioridad o motivo de revisión;
- evidencia registrada.

---

# Parte 16 — Historial de ejecuciones

Localiza al menos:

- una P1;
- una P3/P4;
- un activo desconocido;
- una entrada inválida;
- una severidad desconocida.

Identifica en cada ejecución:

1. payload original;
2. normalización;
3. validación;
4. llamada API cuando proceda;
5. respuesta API;
6. regla aplicada;
7. evidencia;
8. respuesta HTTP final.

---

# Parte 17 — Provocar un fallo técnico

Cambia temporalmente la URL a:

```text
https://api.cloudsecurityinpractice.com/api/v1/assetss/srv-web-01
```

Ejecuta un caso válido.

Observa:

- nodo que falla;
- HTTP status;
- mensaje;
- nodos posteriores no ejecutados.

Restaura la URL.

Explica por qué:

```text
HTTP 404 por endpoint incorrecto
```

es distinto de:

```text
HTTP 200 + found=false
```

---

# Parte 18 — Diseñar fallback

Describe qué debería ocurrir si la API:

- tiene timeout;
- devuelve 500;
- devuelve 401;
- no está disponible;
- devuelve JSON inesperado.

Decide:

- reintentos máximos;
- cuándo pasar a revisión manual;
- qué registrar;
- cuándo notificar.

---

# Parte 19 — Pasar a Production URL

1. Guarda el workflow.
2. Actívalo.
3. Copia Production URL.
4. Envía EVT-01.
5. Comprueba historial.

Responde:

- ¿qué ocurre con Test URL si n8n no escucha?
- ¿por qué no usarla como endpoint estable?
- ¿qué cambia al activar el workflow?

---

# Parte 20 — Revisión de seguridad

Analiza:

1. ¿puede cualquiera que conozca la URL enviar alertas?
2. ¿cómo autenticarías al emisor?
3. ¿aceptarías payloads de cualquier tamaño?
4. ¿dónde guardarías tokens?
5. ¿qué no debería aparecer en logs?
6. ¿qué riesgo existe si el catálogo es incorrecto?
7. ¿ejecutarías remediación automática desde este flujo?

No implementaremos todos los controles todavía.

---

# Resultado esperado

```text
00 Webhook
   |
01 Normalizar
   |
02 Validar
 | false -----------------------> 90 Manual
 | true
 v
03 Consultar activo
   |
04 Activo encontrado
 | false -----------------------> 91 Manual
 | true
 v
05 Alerta enriquecida
   |
06 Switch severity
 | critical -> P1
 | high -> 07 If -> P1/P2
 | medium -> P3
 | low -> P4
 | fallback -> 92 Manual
            |
      70 Preparar evidencia
            |
      80 Registrar evidencia
            |
      99 Respond to Webhook
```

Las rutas de excepción también deben registrar evidencia y responder.

---

# Entrega

Entrega:

1. captura o export visual del workflow;
2. checklist completado;
3. ejemplo de respuesta P1;
4. ejemplo de respuesta manual;
5. evidencia de llamada correcta a API;
6. evidencia del fallo HTTP provocado;
7. explicación de webhook vs polling, Test vs Production, error técnico vs excepción funcional y gestión de credenciales;
8. propuesta de fallback.

---

## Siguiente módulo

En M04 añadiremos Dify para tareas donde las reglas simples no bastan: interpretar texto, extraer información y producir recomendaciones controladas.

La IA se añadirá sobre esta base determinista; no sustituirá la validación ni el control de errores.
