# Laboratorio 9 — Change Request: alertas de certificados TLS

## Contexto

La solución construida hasta M08 ya puede:

- recibir alertas mediante Webhook;
- validar y enriquecer datos;
- calcular prioridad P1–P4;
- recuperar documentación mediante RAG;
- investigar estado operativo con un Agent y tools MCP;
- transformar la recomendación a parámetros estructurados;
- aplicar una política determinista en n8n;
- solicitar aprobación humana mediante HITL;
- degradar de forma segura si Dify no está disponible.

Ahora Telvora solicita un cambio funcional concreto:

> El sistema debe aceptar alertas de expiración de certificados TLS y tratarlas mediante reglas específicas.

No vas a reconstruir la solución.

Vas a **adaptar la que ya tienes**.

Tiempo objetivo del laboratorio:

```text
aprox. 1 h 45 min
```

No se realizará ninguna renovación real de certificados.

---

# Objetivos

Al terminar deberías ser capaz de:

- incorporar un requisito nuevo a una automatización existente;
- añadir una regla determinista específica sin romper las reglas anteriores;
- extender la validación de entrada;
- incorporar nuevo conocimiento a RAG;
- ampliar el catálogo de acciones de un Agent;
- modificar un Parameter Extractor existente;
- actualizar una política HITL;
- validar regresión y compatibilidad con casos anteriores.

---

# Material proporcionado

En `materiales/` encontrarás:

```text
change_request_telvora.md
comportamiento_esperado.md
eventos_webhook_m09.json
prompt_agent_m09.md
telvora_runbook_certificados_tls.txt
```

Empieza leyendo:

```text
change_request_telvora.md
```

---

# Arquitectura

La arquitectura general no cambia:

```text
ALERTA
  |
  v
n8n
  |
  +-- validar
  +-- consultar activo
  +-- calcular prioridad
  |
  v
Dify
  |
  +-- RAG
  +-- Agent + MCP
  +-- Parameter Extractor
  |
  v
n8n POLICY
  |
  +-- AUTO
  +-- HITL
  `-- DEGRADED / MANUAL REVIEW
```

El cambio de M09 afecta únicamente a algunas reglas y datos.

---

# Parte 1 — Crear una copia de trabajo

Duplica tu versión funcional de M08.

Nombre recomendado en n8n:

```text
M09-Certificados-TLS-<ID_ALUMNO>
```

Webhook recomendado:

```text
m09-certificados-<ID_ALUMNO>
```

Duplica también tu Workflow App de Dify de M08.

Nombre recomendado:

```text
<ID_ALUMNO>-M09-Certificados-TLS
```

No modifiques M08.

---

# Parte 2 — Incorporar el nuevo runbook a RAG

Añade a tu Knowledge Base privada:

```text
telvora_runbook_certificados_tls.txt
```

Espera a que el documento quede procesado.

Realiza al menos una prueba de recuperación con:

```text
TLS certificate expires in 1 day production web
```

Debes poder recuperar contenido relacionado con:

```text
TLS-CERT-01
renovación
aprobación humana
```

No continúes si el documento no se recupera de forma razonable.

---

# Parte 3 — Extender la validación de entrada en n8n

Las alertas existentes deben seguir funcionando igual.

Solo debes añadir comportamiento especial cuando:

```text
category == certificate
```

Para ese caso son obligatorios:

```text
certificate_name
days_to_expiry
```

`days_to_expiry` debe ser numérico.

Si falta cualquiera de estos datos, la alerta debe terminar en:

```text
status = manual_review
reason = missing_certificate_data
```

No llames a Dify en ese caso.

## Importante

No intentes pedir al LLM que estime los días restantes.

Si no existe el dato necesario para aplicar la regla determinista, se requiere revisión manual.

---

# Parte 4 — Añadir la prioridad específica para certificados

La clasificación actual P1–P4 sigue siendo válida para alertas normales.

Pero para:

```text
category == certificate
```

aplica esta regla **antes** de la lógica genérica basada en severity:

```text
days_to_expiry <= 2
→ P1

days_to_expiry <= 7
→ P2

days_to_expiry <= 30
→ P3

days_to_expiry > 30
→ P4
```

## Consecuencia importante

Para una alerta de certificado:

```text
severity
```

ya no decide la prioridad.

Ejemplo:

```text
severity = low
days_to_expiry = 1
```

resultado:

```text
P1
```

Otro ejemplo:

```text
severity = critical
days_to_expiry = 14
```

resultado:

```text
P3
```

La nueva regla es autoritativa para este tipo de evento.

---

# Parte 5 — Incluir los datos TLS en alert_context

Cuando construyas `alert_context`, asegúrate de conservar para las alertas de certificado:

```text
category
certificate_name
days_to_expiry
priority
asset_id
environment
criticality
owner_team
summary
details
```

No necesitas crear nuevas variables de entrada en Dify.

Los nuevos campos pueden viajar dentro de:

```text
alert_context
```

Actualiza también `retrieval_query` para que las alertas de certificado incluyan términos como:

```text
certificate
certificate_name
days_to_expiry
summary
```

El objetivo es que RAG pueda encontrar `TLS-CERT-01`.

---

# Parte 6 — Actualizar el Agent en Dify

Utiliza:

```text
materiales/prompt_agent_m09.md
```

como evolución del prompt de M08.

El catálogo de acciones ahora incluye:

```text
renew_certificate
```

El Agent sigue teniendo únicamente tools read-only:

```text
get_service_health
get_recent_changes
get_recent_alerts
get_open_incidents
```

No añadas ninguna tool de renovación.

## Qué significa renew_certificate

Significa:

```text
recomendar iniciar el proceso de renovación
```

No significa:

```text
ejecutar la renovación
```

---

# Parte 7 — Observar el comportamiento del Agent

Antes de integrar el cambio completo en n8n, ejecuta manualmente en Dify un caso equivalente a:

```text
asset_id = srv-web-01
certificate_name = portal.telvora.example
days_to_expiry = 1
priority = P1
```

Comprueba:

1. que RAG recupera `TLS-CERT-01`;
2. que el Agent no recalcula P1;
3. que puede consultar MCP si necesita saber si existe impacto actual;
4. que no llama necesariamente a todas las tools;
5. que no afirma haber renovado nada;
6. que puede recomendar `renew_certificate`.

Un servicio puede estar:

```text
healthy
```

y seguir teniendo un certificado próximo a expirar.

No confundas ambas fuentes de información.

---

# Parte 8 — Extender Parameter Extractor

En el Parameter Extractor existente, amplía los valores permitidos de:

```text
action_type
```

para incluir:

```text
renew_certificate
```

El resto de campos permanece igual:

```text
recommended_next_step
action_type
requires_human_review
confidence
reason
```

No necesitas crear un schema nuevo ni rediseñar la salida.

---

# Parte 9 — Actualizar la política en n8n

La allowlist automática existente no cambia:

```text
continue_monitoring
notify_team
create_incident
```

La lista de acciones que requieren revisión humana debe ampliarse con:

```text
renew_certificate
```

Por tanto:

```text
restart_service
rollback_deployment
renew_certificate
insufficient_evidence
other_sensitive_action
```

siempre deben terminar en:

```text
HUMAN_REVIEW
```

## Regla importante

La política de n8n debe prevalecer incluso si Dify devuelve:

```text
action_type = renew_certificate
requires_human_review = false
```

El resultado debe seguir siendo:

```text
human_review
```

La IA recomienda.

n8n aplica la política.

---

# Parte 10 — Reutilizar HITL

No construyas otro mecanismo de aprobación.

Reutiliza el patrón de M07/M08:

```text
HUMAN_REVIEW
   |
   v
pending_approval
   |
   v
WAIT
   |
approve / reject
```

Para `renew_certificate`, una aprobación produce únicamente una autorización conceptual, por ejemplo:

```text
authorized_not_executed
```

No se ejecuta ninguna renovación real.

---

# Parte 11 — Mantener el fallback de M08

El cambio de M09 no debe romper el comportamiento cuando Dify falla.

Si la llamada a Dify no responde correctamente:

```text
Dify ERROR
   |
   v
degraded_manual_review
```

La prioridad ya calculada en n8n debe conservarse.

No elimines este fallback al modificar el workflow.

---

# Parte 12 — Ejecutar M09-E2E-01

Carga desde:

```text
materiales/eventos_webhook_m09.json
```

el caso:

```text
M09-E2E-01
```

Características:

```text
severity = low
days_to_expiry = 1
```

Predice el resultado antes de ejecutar.

Debes comprobar al menos:

```text
priority = P1
```

Además:

- RAG debe recuperar el runbook TLS;
- el Agent debe producir una recomendación coherente;
- `renew_certificate` es el tipo esperado;
- n8n debe forzar Human-in-the-Loop.

Si aparece una URL de aprobación, prueba `approve` o `reject` como hiciste en M07.

---

# Parte 13 — Ejecutar M09-E2E-02

Caso:

```text
severity = critical
days_to_expiry = 14
```

Esperado:

```text
priority = P3
```

Éste es un caso importante de regresión lógica.

Si obtienes P1, significa que la lógica genérica de `severity` sigue ejecutándose antes o en lugar de la regla específica de certificados.

La recomendación esperada es equivalente a:

```text
notify_team
```

No es obligatorio que el Agent consulte MCP si la documentación y la alerta ya son suficientes.

---

# Parte 14 — Ejecutar M09-E2E-03

Caso:

```text
severity = high
days_to_expiry = 45
```

Esperado:

```text
priority = P4
```

Una salida razonable es:

```text
action_type = continue_monitoring
```

Este caso permite comprobar que el Agent no necesita utilizar herramientas solo porque estén disponibles.

---

# Parte 15 — Ejecutar M09-E2E-04

En este caso falta:

```text
days_to_expiry
```

Esperado:

```text
status = manual_review
reason = missing_certificate_data
```

Comprueba en la ejecución de n8n que:

```text
Dify no fue llamado
```

---

# Parte 16 — Prueba de regresión

Ejecuta una alerta normal de M08 cuya categoría **no** sea `certificate`.

Por ejemplo, reutiliza un caso de disponibilidad o performance.

Comprueba que:

- sigue utilizando la lógica P1–P4 anterior;
- sigue llamando a RAG/Agent cuando corresponde;
- la política HITL sigue funcionando;
- el nuevo código de certificados no afecta al resto de alertas.

---

# Resultado final

Al terminar, tu solución debe implementar:

```text
ALERTA NORMAL
→ reglas existentes
→ RAG + Agent + MCP
→ policy
→ AUTO / HITL

ALERTA CERTIFICATE
→ validar datos TLS
→ prioridad por days_to_expiry
→ RAG TLS-CERT-01
→ Agent + MCP si aporta valor
→ action_type
→ policy
→ AUTO / HITL

DIFY ERROR
→ degraded_manual_review
```

---

# Validación final

Antes de dar el laboratorio por terminado verifica:

```text
[ ] M08 permanece intacto
[ ] TLS-CERT-01 está indexado
[ ] certificate_name se valida
[ ] days_to_expiry se valida
[ ] P1/P2/P3/P4 de certificados se calcula en n8n
[ ] severity no modifica esa prioridad específica
[ ] alert_context conserva los datos TLS
[ ] retrieval_query recupera el runbook TLS
[ ] Agent conoce renew_certificate
[ ] Parameter Extractor acepta renew_certificate
[ ] n8n fuerza HITL para renew_certificate
[ ] no existe ninguna tool de renovación
[ ] Dify failure conserva fallback seguro
[ ] una alerta normal de M08 sigue funcionando
```

Consulta:

```text
materiales/comportamiento_esperado.md
```

si necesitas contrastar los cuatro casos de prueba.

---

# Cierre

Has partido de una solución ya operativa y has incorporado un requisito nuevo sin reconstruirla.

El resultado representa la evolución completa del curso:

```text
workflow determinista
        +
integración REST
        +
LLM
        +
RAG
        +
Agent + MCP
        +
política
        +
Human-in-the-Loop
        +
degradación segura
        +
cambio de requisito
```

Éste es el comportamiento esperado de una automatización técnica que debe poder evolucionar sin entregar toda la autoridad a la IA.
