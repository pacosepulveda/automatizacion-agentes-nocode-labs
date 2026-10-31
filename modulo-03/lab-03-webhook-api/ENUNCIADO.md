# Laboratorio 3 — Eventos, Webhooks e integración con sistemas

## Contexto

En M02 construiste un workflow que clasificaba alertas en P1–P4, pero todavía tenía dos simplificaciones:

1. la ejecución comenzaba manualmente;
2. `environment`, `criticality` y `owner_team` ya venían incluidos en la entrada.

En este laboratorio eliminaremos esas dos simplificaciones.

La alerta llegará desde fuera de n8n mediante un **Webhook** y el workflow consultará la API de laboratorio para obtener los datos reales del activo.

A partir de este punto el flujo ya se comporta como una integración entre sistemas.

> Todos los datos son ficticios y se utilizan exclusivamente con fines formativos.

## Objetivos

Al finalizar deberías ser capaz de:

- recibir un evento HTTP en n8n mediante Webhook;
- diferenciar Test URL y Production URL;
- localizar el payload dentro de `body`;
- normalizar una entrada externa;
- consumir una API REST mediante `HTTP Request`;
- utilizar una credencial en lugar de incrustar un token;
- construir una URL dinámica con `asset_id`;
- enriquecer una alerta con información de otro sistema;
- reutilizar las reglas P1–P4 de M02;
- distinguir un activo inexistente de un fallo técnico de la API;
- devolver una respuesta JSON al sistema emisor.

## Infraestructura

n8n:

```text
https://n8n-agentes.cloudsecurityinpractice.com/
```

API:

```text
https://api.cloudsecurityinpractice.com
```

El instructor proporcionará el token de acceso a la API.

Cada alumno debe utilizar un path de Webhook único.

Ejemplo:

```text
m03-alert-alumno01
```

## Material proporcionado

En `materiales/` encontrarás:

- `alertas_webhook.json`;
- `contrato_api.md`;
- `comandos_prueba.md`;
- `resultados_esperados.md`.

---

# Parte 1 — Crear la nueva versión del workflow

Duplica el workflow de M02 para conservar una versión funcional.

Renómbralo, por ejemplo:

```text
M03-Webhook-API-Triage
```

El objetivo es evolucionar lo que ya construiste, no empezar de cero.

---

# Parte 2 — Sustituir Manual Trigger por Webhook

Añade un nodo:

```text
00 - Recibir alerta
```

Tipo:

```text
Webhook
```

Configura:

```text
HTTP Method: POST
Path: m03-alert-<ID_ALUMNO>
Response: Using Respond to Webhook
```

Por ejemplo:

```text
m03-alert-alumno01
```

No utilices un path genérico compartido con otros alumnos.

Pon el nodo en modo de escucha y copia la **Test URL**.

Envía el caso `EVT-01` utilizando `materiales/comandos_prueba.md`.

## Observa la entrada

El Webhook entregará una estructura similar a:

```json
{
  "headers": {},
  "params": {},
  "query": {},
  "body": {
    "alert_id": "ALT-3001",
    "asset_id": "srv-web-01",
    "severity": "high"
  }
}
```

Los campos de la alerta están dentro de `body`.

---

# Parte 3 — Normalizar la alerta

Añade un nodo `Edit Fields (Set)`:

```text
01 - Normalizar alerta
```

Extrae desde `body` y deja en la raíz:

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

A partir de este nodo, el resto del workflow debe trabajar con una estructura estable, independientemente de cómo llegase el payload por HTTP.

---

# Parte 4 — Validar campos obligatorios

Reutiliza la lógica de M02 para validar:

```text
alert_id
timestamp
asset_id
severity
summary
```

Si falta alguno, genera una salida equivalente a:

```json
{
  "accepted": false,
  "status": "manual_review",
  "reason": "missing_required_field",
  "requires_human_review": true
}
```

Esta rama no debe consultar la API.

---

# Parte 5 — Crear la credencial de la API

No escribas el token directamente en el nodo `HTTP Request`.

Crea una credencial de tipo **Header Auth** con un nombre como:

```text
M03 - Lab API
```

Debe enviar:

```http
Authorization: Bearer <LAB_API_TOKEN>
```

El token será facilitado por el instructor.

---

# Parte 6 — Consultar el activo

Desde la rama de entrada válida, añade:

```text
03 - Consultar activo
```

Tipo:

```text
HTTP Request
```

Configura:

```text
Method: GET
URL: https://api.cloudsecurityinpractice.com/api/v1/assets/<asset_id>
Authentication: M03 - Lab API
Response: JSON
```

`asset_id` debe obtenerse mediante una expresión.

Para `srv-web-01`, la URL final será:

```text
https://api.cloudsecurityinpractice.com/api/v1/assets/srv-web-01
```

Consulta `materiales/contrato_api.md` para ver la respuesta esperada.

---

# Parte 7 — Comprobar si el activo existe

La API devolverá:

```json
{
  "found": true,
  ...
}
```

cuando encuentre el activo.

Si devuelve:

```json
{
  "found": false,
  "error": "asset_not_found"
}
```

el workflow debe terminar en revisión manual con:

```text
reason = asset_not_found
requires_human_review = true
```

Ten en cuenta que `found=false` significa que **la API ha funcionado correctamente**, pero el activo no existe en el catálogo.

---

# Parte 8 — Construir la alerta enriquecida

Si el activo existe, crea:

```text
05 - Alerta enriquecida
```

Combina los campos de la alerta original con:

```text
asset_name
environment
criticality
owner_team
```

procedentes de la API.

Después de `HTTP Request`, `$json` contiene principalmente la respuesta de la API. Para recuperar datos de la alerta puedes referenciar explícitamente el nodo de normalización.

Ejemplo:

```text
{{ $('01 - Normalizar alerta').item.json.alert_id }}
```

Los datos devueltos por la API pueden obtenerse desde `$json`.

No copies manualmente información del catálogo.

---

# Parte 9 — Reutilizar la clasificación P1–P4

Aplica las mismas reglas de M02, ahora sobre la alerta enriquecida.

## P1

```text
severity = critical
```

O:

```text
severity = high
AND environment = production
AND criticality = high
```

## P2

```text
severity = high
```

cuando no sea P1.

## P3

```text
severity = medium
```

## P4

```text
severity = low
```

Cualquier otro valor de `severity` debe terminar en revisión manual con:

```text
reason = invalid_severity
```

---

# Parte 10 — Preparar una respuesta útil

Para una alerta clasificada, prepara una salida equivalente a:

```json
{
  "accepted": true,
  "alert_id": "ALT-3001",
  "asset_id": "srv-web-01",
  "asset_name": "web-frontend-01",
  "environment": "production",
  "priority": "P1",
  "owner_team": "WebOps",
  "requires_human_review": true
}
```

Las ramas de revisión manual deben mantener una estructura coherente, por ejemplo:

```json
{
  "accepted": false,
  "alert_id": "ALT-3005",
  "status": "manual_review",
  "reason": "asset_not_found",
  "requires_human_review": true
}
```

---

# Parte 11 — Responder al sistema emisor

Añade un nodo:

```text
99 - Responder
```

Tipo:

```text
Respond to Webhook
```

Haz que las distintas rutas terminen entregando su resultado a este nodo.

Vuelve a enviar `EVT-01` y comprueba que el cliente recibe el JSON generado por tu workflow.

---

# Parte 12 — Probar todos los casos

Ejecuta los casos de `alertas_webhook.json`.

Antes de cada prueba, predice el resultado.

Comprueba al menos:

- P1 por severidad alta sobre activo crítico de producción;
- P3;
- P1 por severidad critical;
- activo desconocido;
- campo obligatorio ausente;
- severity desconocida;
- P2;
- P4.

Compara el resultado con `resultados_esperados.md`.

---

# Parte 13 — Observar un fallo técnico real

Cambia temporalmente la URL del nodo `HTTP Request` a:

```text
https://api.cloudsecurityinpractice.com/api/v1/assetss/srv-web-01
```

Ejecuta una alerta válida.

Observa:

- qué nodo falla;
- el código HTTP;
- el mensaje de error;
- qué nodos posteriores dejan de ejecutarse.

Después restaura la URL correcta.

Compara estos dos casos:

```text
HTTP 200 + found=false
```

frente a:

```text
HTTP 404 por endpoint incorrecto
```

El primero es un resultado de negocio; el segundo es un fallo técnico de integración.

No implementaremos todavía una estrategia completa de reintentos o fallback.

---

# Parte 14 — Publicar y usar la Production URL

Cuando el flujo funcione:

1. guarda los cambios;
2. publica/activa el workflow;
3. copia la **Production URL**;
4. envía de nuevo `EVT-01`;
5. comprueba la ejecución en el historial de n8n.

## Preguntas

- ¿Qué diferencia existe entre Test URL y Production URL?
- ¿Por qué cada alumno necesita un path distinto?
- ¿Qué datos proceden del sistema emisor y cuáles de la API?

---

# Resultado del laboratorio

Al terminar deberías tener una solución equivalente a:

```text
SISTEMA EMISOR
     |
     | POST JSON
     v
WEBHOOK n8n
     |
     v
NORMALIZAR
     |
     v
VALIDAR
  /      \
 no       sí
 |         |
 v         v
REVISIÓN  API DE ACTIVOS
             |
             v
        ENRIQUECER
             |
             v
        CLASIFICAR
          P1–P4
             |
             v
       RESPONDER JSON
```

La automatización ya recibe eventos reales y obtiene contexto de otro sistema sin intervención manual.

## Siguiente evolución

Aun así, la solución sigue teniendo una limitación importante: puede aplicar reglas, pero no sabe interpretar qué puede significar el contenido de una alerta compleja.

En M04 añadiremos IA generativa para analizar ese contexto.