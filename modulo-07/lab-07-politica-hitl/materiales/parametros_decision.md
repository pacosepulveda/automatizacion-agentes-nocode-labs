# Parameter Extractor — contrato mínimo de decisión

Añade un nodo **Parameter Extractor** después del Agent.

Nombre recomendado:

```text
02 - Extraer decisión
```

## Input

Selecciona mediante el selector de variables la **salida textual final del Agent**.

No escribas manualmente un nombre como `Agent.text` si la interfaz no lo muestra. Selecciona la salida disponible en tu versión de Dify.

## Modelo

Utiliza el modelo indicado por el instructor.

Si la interfaz ofrece varios modos de extracción, utiliza preferentemente:

```text
Function Call / Tool Call
```

Si ese modo no está disponible con el modelo configurado, utiliza:

```text
Prompt-based
```

## Parámetros

Define exactamente estos cinco parámetros.

### `recommended_next_step`

```text
Type: string
Required: yes
```

Descripción:

```text
Siguiente paso operativo que el Agent recomienda. Debe ser una frase breve y concreta. No debe afirmar que la acción ya se ha ejecutado.
```

### `action_type`

```text
Type: string
Required: yes
```

Descripción:

```text
Clasifica la recomendación en exactamente uno de estos valores:
continue_monitoring
notify_team
create_incident
restart_service
rollback_deployment
insufficient_evidence
other_sensitive_action

No inventes otras categorías. Si la recomendación implica modificar sistemas, configuración, accesos o infraestructura y no encaja en restart_service o rollback_deployment, utiliza other_sensitive_action.
```

### `requires_human_review`

```text
Type: boolean
Required: yes
```

Descripción:

```text
true si la recomendación implica restart_service, rollback_deployment, other_sensitive_action o si no existe evidencia suficiente. false para recomendaciones puramente informativas o de seguimiento. Este campo es orientativo: n8n aplicará después la política autoritativa.
```

### `confidence`

```text
Type: string
Required: yes
```

Descripción:

```text
Confianza de la recomendación. Utiliza únicamente low, medium o high. La confianza debe depender de la evidencia disponible, no del estilo de la respuesta.
```

### `reason`

```text
Type: string
Required: yes
```

Descripción:

```text
Justificación breve de por qué se recomienda ese siguiente paso, basada en hechos observados, contexto RAG o resultados MCP.
```

## Salidas esperadas

El nodo debe exponer variables equivalentes a:

```text
recommended_next_step
action_type
requires_human_review
confidence
reason
```

También puede exponer variables internas de estado como `__is_success` y `__reason` dependiendo de la versión.

## Regla importante

`requires_human_review` no es la política final.

El Agent y el Parameter Extractor pueden equivocarse. La decisión autoritativa se tomará en n8n utilizando `action_type` y una lista explícita de categorías permitidas.
