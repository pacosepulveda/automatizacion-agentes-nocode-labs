# Laboratorio 7 — Decisiones, política y Human-in-the-Loop

## Contexto

En M06 construiste una solución capaz de:

- recibir una alerta mediante Webhook en n8n;
- validar y enriquecer el evento;
- calcular una prioridad P1–P4 mediante reglas deterministas;
- recuperar documentación interna mediante RAG;
- utilizar un Agent para decidir qué tools MCP consultar;
- combinar contexto documental y estado operativo vivo;
- devolver un análisis técnico.

La arquitectura ya investiga, pero todavía existe una limitación importante:

```text
Agent
  |
  v
texto de análisis
  |
  v
persona
```

El Agent puede recomendar un siguiente paso, pero n8n todavía no puede utilizar esa recomendación de forma controlada para decidir si el proceso puede continuar automáticamente o necesita una persona.

En este laboratorio añadiremos dos piezas:

```text
Parameter Extractor
```

para convertir la recomendación del Agent en unos pocos campos estructurados, y:

```text
POLÍTICA EN n8n
```

para que la decisión final siga siendo determinista.

Además implementaremos un **Human-in-the-Loop real**: cuando una acción necesite aprobación, la ejecución de n8n quedará pausada hasta recibir `approve` o `reject`.

> No se ejecutarán reinicios, rollbacks ni cambios destructivos reales. El laboratorio implementa decisión y aprobación, no remediación.

---

## Objetivos

Al finalizar deberías ser capaz de:

- transformar una recomendación en lenguaje natural en parámetros estructurados;
- utilizar `Parameter Extractor` en Dify;
- diferenciar una recomendación del Agent de una política empresarial;
- aplicar una allowlist de acciones automáticas en n8n;
- utilizar un criterio conservador para acciones desconocidas;
- implementar una rama de continuación automática;
- pausar una ejecución de n8n mediante `Wait`;
- generar una URL de reanudación única para una aprobación;
- reanudar la ejecución mediante una decisión humana;
- distinguir `approve` de `reject`;
- comprobar que una acción sensible nunca se ejecuta únicamente porque la IA la recomiende.

---

## Prerrequisitos

Necesitas:

- tu workflow funcional de M06 en n8n;
- tu Workflow App de M06 en Dify;
- tu Knowledge Base de M05;
- acceso al MCP compartido `Telvora Ops Hub`;
- tu identificador de alumno;
- los archivos de `materiales/`.

Infraestructura:

```text
n8n:  https://n8n-agentes.cloudsecurityinpractice.com/
Dify: https://dify.cloudsecurityinpractice.com/
```

No modifiques integraciones, proveedores ni aplicaciones de otros alumnos.

---

## Arquitectura objetivo

Al final del laboratorio tendrás algo equivalente a:

```text
SISTEMA EMISOR
      |
      v
WEBHOOK n8n
      |
      v
VALIDAR + API + P1-P4
      |
      v
DIFY
  +-------------------------+
  | RAG                     |
  |   |                     |
  | Agent + MCP             |
  |   |                     |
  | Parameter Extractor     |
  +-------------------------+
      |
      | action_type
      | recommended_next_step
      | confidence
      | requires_human_review
      v
POLÍTICA n8n
   /              \
  /                \
AUTO               HUMANO
 |                   |
 v                   v
RESPONDER        RESPONDER
                    |
                    v
                   WAIT
                    |
             approve / reject
```

La IA propone y clasifica.

**n8n decide.**

---

# Parte 1 — Duplicar la App de M06

Duplica únicamente tu propia Workflow App de M06.

Nombre recomendado:

```text
<ID_ALUMNO>-M07-Decisiones-HITL
```

Ejemplo:

```text
AL01-M07-Decisiones-HITL
```

No modifiques M06. Queremos conservar una versión funcional anterior.

La aplicación seguirá utilizando:

```text
alert_context
asset_id
retrieval_query
question
```

como inputs.

---

# Parte 2 — Ajustar el Agent para producir una recomendación explícita

Abre:

```text
materiales/prompt_agent_m07.md
```

Utilízalo como nueva versión de las instrucciones del Agent.

El cambio principal es que ahora el Agent debe terminar siempre indicando de forma explícita:

```text
Siguiente paso recomendado
Tipo de acción
Motivo
Confianza
Requiere revisión humana
```

El Agent sigue teniendo únicamente las tools read-only de `Telvora Ops Hub`.

No añadas ninguna tool de remediación.

---

# Parte 3 — Añadir Parameter Extractor

Después del Agent añade un nodo:

```text
02 - Extraer decisión
```

Tipo:

```text
Parameter Extractor
```

Consulta:

```text
materiales/parametros_decision.md
```

para configurar los cinco parámetros:

```text
recommended_next_step
action_type
requires_human_review
confidence
reason
```

## Input

Utiliza como entrada la **salida textual final del Agent**.

Selecciona la variable desde el selector de Dify.

No escribas manualmente:

```text
Agent.text
```

si esa variable no existe con ese nombre en tu interfaz.

## Modo de extracción

Si está disponible con el modelo del curso, utiliza:

```text
Function Call / Tool Call
```

Si no aparece o no es compatible, utiliza:

```text
Prompt-based
```

El objetivo no es practicar una opción concreta de interfaz, sino obtener cinco variables estructuradas a partir de la recomendación.

---

# Parte 4 — Exponer las salidas del Workflow

En el nodo **End** conserva una salida con el análisis completo del Agent y añade las variables extraídas.

La salida conceptual debe contener:

```text
analysis
recommended_next_step
action_type
requires_human_review
confidence
reason
```

No expongas una nueva prioridad.

La prioridad continúa siendo responsabilidad de n8n.

---

# Parte 5 — Probar Dify antes de continuar

Ejecuta manualmente al menos dos casos.

Puedes reutilizar contextos de M06 o construir los inputs a partir de `M07-E2E-01` y `M07-E2E-02`.

Comprueba dos cosas por separado.

## Agent

Abre el detalle de la estrategia agéntica y observa:

- qué tools consulta;
- en qué orden;
- con qué argumentos;
- si cambia de estrategia después de recibir un resultado;
- si se detiene cuando ya dispone de suficiente evidencia.

## Parameter Extractor

Comprueba que produce valores equivalentes a:

```json
{
  "recommended_next_step": "...",
  "action_type": "notify_team",
  "requires_human_review": false,
  "confidence": "medium",
  "reason": "..."
}
```

Si `action_type` contiene valores distintos de los permitidos, revisa la descripción del parámetro y las instrucciones del Agent.

---

# Parte 6 — Publicar la App M07

Cuando las pruebas sean correctas:

1. guarda la App;
2. publícala;
3. crea un API key específico para M07;
4. no copies el API key al repositorio ni a un nodo como texto plano.

En n8n utilizaremos una credencial.

---

# Parte 7 — Duplicar el workflow n8n de M06

Duplica tu workflow funcional de M06.

Nombre recomendado:

```text
M07-Policy-HITL
```

Utiliza un nuevo path de Webhook:

```text
m07-alert-<ID_ALUMNO>
```

Ejemplo:

```text
m07-alert-alumno01
```

Actualiza la credencial/API key de Dify para llamar a tu nueva App M07.

Mantén sin cambios la parte de:

```text
Webhook
→ normalización
→ validación
→ API de activos
→ enriquecimiento
→ P1-P4
→ preparación de inputs
→ Dify
```

---

# Parte 8 — Normalizar la decisión devuelta por Dify

Después de `HTTP Request` hacia Dify añade un nodo:

```text
10 - Normalizar decisión
```

Tipo:

```text
Edit Fields (Set)
```

Extrae desde la respuesta de Dify y deja disponibles en la raíz:

```text
analysis
recommended_next_step
action_type
requires_human_review
confidence
reason
```

Además recupera los datos que n8n ya conocía:

```text
alert_id
asset_id
priority
owner_team
```

Utiliza el selector de variables y las referencias a nodos anteriores.

No recalcules `priority` a partir del contenido de Dify.

---

# Parte 9 — La política que manda

Ahora vamos a aplicar una política determinista.

## Acciones que pueden continuar automáticamente

```text
continue_monitoring
notify_team
create_incident
```

## Acciones que requieren siempre una persona

```text
restart_service
rollback_deployment
insufficient_evidence
other_sensitive_action
```

## Regla por defecto

Si aparece cualquier otro valor:

```text
human_review
```

Nunca uses como única condición:

```text
requires_human_review == false
```

Ese valor ha sido generado por IA.

La allowlist de n8n es autoritativa.

---

# Parte 10 — Implementar la política con Switch

Añade:

```text
11 - Aplicar política
```

Tipo:

```text
Switch
```

Utiliza `action_type` como valor a evaluar.

Agrupa conceptualmente las salidas en:

```text
AUTO
HUMAN_REVIEW
```

Puedes crear reglas individuales para los valores permitidos y dirigir el resto a la rama de revisión.

Es importante que el fallback/default termine también en:

```text
HUMAN_REVIEW
```

Una categoría inesperada nunca debe convertirse automáticamente en una acción permitida.

---

# Parte 11 — Comprobar la política independientemente del Agent

Abre:

```text
materiales/decisiones_prueba.json
```

Revisa `POL-01` a `POL-05` y predice la rama de cada uno antes de mirar el resultado esperado.

El caso más importante es `POL-03`:

```text
action_type = rollback_deployment
requires_human_review = false
```

Aunque la IA diga `false`, la política debe producir:

```text
human_review
```

La razón es sencilla:

> La IA puede recomendar. No puede modificar la política que determina su propia autoridad.

Si quieres validar físicamente las rutas del `Switch`, puedes utilizar temporalmente un `Edit Fields` con los valores de cada caso y conectarlo al nodo `11 - Aplicar política`. Elimina o desconecta ese nodo de prueba al terminar.

---

# Parte 12 — Rama automática

En la salida `AUTO` añade:

```text
12A - Continuación automática
```

Prepara una respuesta equivalente a:

```json
{
  "status": "completed",
  "policy_decision": "auto_continue",
  "alert_id": "ALT-7101",
  "priority": "P4",
  "action_type": "continue_monitoring",
  "recommended_next_step": "...",
  "confidence": "medium",
  "action_status": "safe_path_completed",
  "requires_human_review": false
}
```

En esta práctica la rama automática **no envía emails ni modifica sistemas externos**.

El significado de `auto_continue` es:

```text
la política permite que el workflow termine sin detenerse a pedir aprobación
```

Conecta esta rama a un `Respond to Webhook`.

---

# Parte 13 — Preparar la aprobación humana

En la rama `HUMAN_REVIEW` añade un nodo:

```text
12B - Preparar aprobación
```

Incluye:

```text
status = pending_approval
policy_decision = human_review
alert_id
priority
action_type
recommended_next_step
reason
confidence
analysis
```

Y añade un campo especialmente importante:

```text
approval_url
```

con la expresión de n8n:

```text
{{ $execution.resumeUrl }}
```

Esa URL será única para la ejecución y permitirá reanudar el workflow después.

No la publiques ni la reutilices en otra ejecución.

---

# Parte 14 — Responder antes de esperar

Después de `12B - Preparar aprobación` añade un `Respond to Webhook`:

```text
13 - Responder pendiente de aprobación
```

La respuesta inicial debe ser equivalente a:

```json
{
  "status": "pending_approval",
  "alert_id": "ALT-7102",
  "priority": "P1",
  "action_type": "rollback_deployment",
  "recommended_next_step": "...",
  "reason": "...",
  "approval_url": "https://..."
}
```

El sistema que envió la alerta recibe inmediatamente la información necesaria para saber que el proceso está esperando una decisión.

La ejecución de n8n continuará después hacia el nodo `Wait`.

---

# Parte 15 — Pausar la ejecución con Wait

Añade:

```text
14 - Esperar aprobación
```

Tipo:

```text
Wait
```

Configura:

```text
Resume: On Webhook Call
HTTP Method: POST
Authentication: None
Respond: When Last Node Finishes
```

Para el laboratorio puedes establecer un límite de espera de aproximadamente:

```text
15 minutos
```

si tu versión de n8n permite configurar fácilmente `Limit Wait Time`.

## Sobre Authentication: None

La URL de reanudación es única y firmada para la ejecución, pero debe tratarse como un enlace sensible.

En un entorno productivo podríamos añadir autenticación adicional, controlar quién recibe el enlace y aplicar expiración más estricta.

No necesitamos convertir este laboratorio en un ejercicio de IAM.

---

# Parte 16 — Enviar approve o reject

Cuando una ejecución quede esperando, copia `approval_url` de la respuesta inicial.

Consulta:

```text
materiales/aprobacion_manual.md
```

Para aprobar conceptualmente:

```json
{
  "decision": "approve",
  "comment": "Revisado por el operador"
}
```

Para rechazar:

```json
{
  "decision": "reject",
  "comment": "No existe evidencia suficiente"
}
```

Puedes utilizar `curl` o PowerShell.

---

# Parte 17 — Evaluar la decisión humana

Después del `Wait`, inspecciona primero la salida real del nodo.

Localiza:

```text
decision
comment
```

Según la versión de n8n pueden aparecer dentro de una estructura tipo Webhook, por ejemplo dentro de `body`.

Utiliza el selector de variables; no copies una expresión de este documento sin comprobar primero la salida real del nodo.

Añade:

```text
15 - Evaluar aprobación
```

Tipo:

```text
Switch
```

Rutas:

```text
approve
reject
```

Cualquier valor inesperado debe tratarse como rechazo o revisión pendiente, nunca como aprobación automática.

---

# Parte 18 — Resultado de aprobación

## Approve

Añade:

```text
16A - Acción autorizada
```

con una salida equivalente a:

```json
{
  "status": "approved",
  "policy_decision": "human_review",
  "human_decision": "approve",
  "action_type": "rollback_deployment",
  "action_status": "authorized_not_executed",
  "comment": "Revisado por el operador"
}
```

`authorized_not_executed` es deliberado.

El laboratorio **no dispone de una tool para ejecutar el rollback**.

## Reject

Añade:

```text
16B - Acción rechazada
```

con una salida equivalente a:

```json
{
  "status": "rejected",
  "policy_decision": "human_review",
  "human_decision": "reject",
  "action_type": "rollback_deployment",
  "action_status": "rejected",
  "comment": "No existe evidencia suficiente"
}
```

Como el `Wait` está configurado para responder cuando termine el workflow, el cliente que llama a `approval_url` debería recibir el resultado final de esta segunda fase.

---

# Parte 19 — Probar una ruta automática

Abre:

```text
materiales/eventos_webhook_m07.json
```

Empieza con:

```text
M07-E2E-01
```

Ejecuta el flujo completo.

Comprueba:

1. prioridad calculada por n8n;
2. documentos recuperados;
3. tool calls del Agent;
4. `action_type` extraído;
5. rama elegida por la política.

Si el Agent concluye `continue_monitoring` o `notify_team`, la ejecución debe terminar sin aprobación.

---

# Parte 20 — Probar una ruta con aprobación

Ejecuta:

```text
M07-E2E-02
```

El caso contiene evidencia de una regresión después de un despliegue y pregunta implícitamente si debe proponerse rollback.

Si el Agent produce:

```text
action_type = rollback_deployment
```

comprueba:

1. que n8n no continúa por `AUTO`;
2. que la primera respuesta contiene `status=pending_approval`;
3. que recibes `approval_url`;
4. que la ejecución aparece en estado de espera;
5. que `approve` reanuda la ejecución;
6. que el resultado final es `authorized_not_executed`;
7. que ningún nodo ejecuta un rollback real.

Repite la prueba con `reject` si el tiempo lo permite.

---

# Parte 21 — Probar evidencia insuficiente

Ejecuta:

```text
M07-E2E-03
```

El contexto del batch es deliberadamente incompleto.

Si ni RAG ni MCP permiten justificar una actuación, un resultado correcto es:

```text
action_type = insufficient_evidence
```

La política debe enviarlo a revisión humana.

La falta de evidencia no debe transformarse en una acción automática para "hacer algo".

---

# Parte 22 — Probar escalado controlado

Ejecuta:

```text
M07-E2E-04
```

Observa cómo el Agent combina:

```text
alert_context
+
runbook DB-HEALTH-01
+
MCP
```

Una recomendación `create_incident` puede continuar automáticamente porque no modifica el sistema investigado.

Si en cambio propone una acción técnica sensible, la política debe detenerla.

---

# Resultado del laboratorio

Al finalizar tendrás una solución equivalente a:

```text
EVENTO
  |
  v
n8n: validar + enriquecer + prioridad
  |
  v
Dify
  |
  +--> RAG
  |
  +--> Agent + MCP
  |
  +--> Parameter Extractor
  |
  v
DECISIÓN ESTRUCTURADA
  |
  v
POLÍTICA n8n
  |
  +--> SAFE --------> continuar automáticamente
  |
  `--> SENSITIVE ---> responder pending_approval
                       |
                       v
                      WAIT
                       |
                 approve/reject
```

Ahora la solución ya no utiliza el Agent únicamente para producir texto.

La investigación del Agent **participa en una automatización**, pero su autoridad está limitada por una política externa y por una persona cuando la acción puede tener impacto.

---

## Limitación actual

El flujo ya funciona en el camino normal, pero todavía debemos comprobar cómo se comporta cuando aparecen situaciones menos cómodas:

- MCP no devuelve información útil;
- una dependencia falla;
- aparece contenido de prompt injection;
- la evidencia es contradictoria;
- el Agent intenta recomendar algo fuera del catálogo;
- una aprobación caduca o no llega.

En M08 someteremos la solución completa a ese tipo de situaciones y la dejaremos preparada para el cierre final del curso.
