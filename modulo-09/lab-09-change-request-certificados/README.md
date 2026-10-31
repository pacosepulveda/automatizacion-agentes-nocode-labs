# Laboratorio 9 — Cambio dirigido: certificados TLS

En este laboratorio adaptarás la solución casi final construida hasta M08 a un nuevo requisito de Telvora: gestionar alertas de expiración de certificados TLS.

No construirás una solución nueva. Trabajarás sobre copias de tus componentes de M08 y realizarás cambios pequeños pero coordinados en:

- reglas deterministas de prioridad en n8n;
- documentación RAG;
- catálogo de acciones del Agent;
- Parameter Extractor;
- política Human-in-the-Loop.

## Resultado esperado

La solución deberá aceptar alertas `category=certificate`, calcular una prioridad específica a partir de `days_to_expiry`, utilizar el runbook `TLS-CERT-01`, recomendar una acción coherente y garantizar que cualquier renovación de certificado requiere aprobación humana.

No se realizará ninguna renovación real.
