# Envío de eventos con curl

Sustituye `<WEBHOOK_URL>` por la URL que te muestre n8n.

> En desarrollo usa la **Test URL** mientras el Webhook está escuchando.  
> Con el workflow **publicado** usa la **Production URL**.

## Path único por alumno

El Webhook del laboratorio debe usar un path distinto para cada alumno:

```text
m03-alert-<ID_ALUMNO>
```

Ejemplo:

```text
m03-alert-alumno01
```

No construyas manualmente la URL completa si no es necesario: copia la Test URL o la Production URL que muestra n8n y verifica que contiene tu identificador.

## EVT-01

```bash
curl -X POST "<WEBHOOK_URL>" \
  -H "Content-Type: application/json" \
  -d '{"alert_id":"ALT-2001","timestamp":"2026-10-21T09:00:00Z","asset_id":"srv-web-01","severity":"high","source":"monitoring","category":"availability","summary":"HTTP 5xx rate above threshold","details":"5xx responses exceeded 12% during the last 5 minutes"}'
```

## EVT-04 — activo desconocido

```bash
curl -X POST "<WEBHOOK_URL>" \
  -H "Content-Type: application/json" \
  -d '{"alert_id":"ALT-2004","timestamp":"2026-10-21T09:06:00Z","asset_id":"srv-unknown-99","severity":"high","source":"monitoring","category":"security","summary":"Unexpected outbound connection","details":"Connection detected to an unclassified external destination"}'
```

## EVT-05 — falta asset_id

```bash
curl -X POST "<WEBHOOK_URL>" \
  -H "Content-Type: application/json" \
  -d '{"alert_id":"ALT-2005","timestamp":"2026-10-21T09:08:00Z","severity":"low","source":"monitoring","category":"capacity","summary":"Filesystem usage above 70%","details":"The alert intentionally omits asset_id"}'
```

El resto de payloads está en `alertas_webhook.json`.

En Windows 10/11 puedes usar `curl.exe` desde PowerShell.
