# Comportamiento esperado — Laboratorio 5

No compares frases exactas. Comprueba el comportamiento funcional.

## Casos Dify

| Caso | Comportamiento esperado |
|---|---|
| RAG-01 | Recupera inventario y responde `web-prod-01` y `eu-west-1`. |
| RAG-02 | Recupera el runbook y señala primero balanceador y después instancia asociada. |
| RAG-03 | Recupera cambios/despliegues y responde con `marta.rios`, `platform-release` y `18:42 UTC`. |
| RAG-04 | No inventa ni revela una contraseña; indica que no está disponible en la documentación. |
| RAG-05 | No deduce automáticamente el puerto 22; indica que el puerto SSH no está documentado. |
| RAG-06 | Recupera `DB-HEALTH-01` y propone comprobar conectividad, motor, disco/memoria, conexiones y replicación sin afirmar causa raíz. |

## Casos end-to-end

### M05-E2E-01

- n8n mantiene la prioridad calculada mediante reglas.
- RAG debe poder recuperar el runbook web y contexto del último despliegue de `web-frontend-01`.
- El análisis puede señalar el despliegue reciente como evidencia relevante o hipótesis, pero no como causa confirmada.
- Debe recomendar las comprobaciones del runbook antes de proponer remediación.

### M05-E2E-02

- Debe recuperar el procedimiento `DB-HEALTH-01`.
- Debe enumerar las comprobaciones documentadas.
- No debe inventar valores de disco, memoria, conexiones o replicación.
- No debe afirmar una causa raíz sin evidencias adicionales.

### M05-E2E-03

- Debe recuperar la política de backup de `telvora-prod-db`.
- Debe indicar que la copia completa está programada a las `02:30 UTC` y que la retención es de `14 días` si esos datos resultan relevantes al análisis.
- Para un fallo de backup debe recomendar revisar job, espacio disponible y estado del servicio de base de datos.
- No debe inventar el motivo concreto del fallo.

## Si el resultado es incorrecto

Comprueba en este orden:

1. qué valor recibió `retrieval_query`;
2. qué fragmentos recuperó Knowledge Retrieval;
3. si esos fragmentos contienen la evidencia necesaria;
4. si el LLM recibió realmente el contexto recuperado;
5. si el prompt obliga a reconocer información ausente.
