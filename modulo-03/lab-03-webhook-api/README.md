# Laboratorio 3 — Eventos, webhooks e integración con API

En este laboratorio el workflow de triaje construido en el módulo anterior deja de depender de una entrada simulada y pasa a recibir alertas mediante un **webhook**.

Además, el workflow consulta una **API REST de laboratorio** para obtener la información del activo, clasifica la alerta con las mismas reglas de los laboratorios anteriores, registra una evidencia y devuelve una respuesta JSON al emisor.

## Objetivos principales

- Recibir eventos mediante un Webhook de n8n.
- Diferenciar Test URL y Production URL.
- Normalizar un payload externo.
- Validar campos obligatorios.
- Consumir una API REST mediante HTTP Request.
- Utilizar una credencial HTTP sin incrustar secretos en el workflow.
- Enriquecer la alerta con datos de otro sistema.
- Gestionar activos desconocidos y severidades inválidas.
- Registrar una evidencia de ejecución.
- Devolver una respuesta estructurada al sistema emisor.
- Analizar errores de integración y distinguirlos de errores de negocio.

Consulta [ENUNCIADO.md](ENUNCIADO.md) para realizar el laboratorio.

## Materiales

- `materiales/alertas_webhook.json`: casos de prueba.
- `materiales/checklist_pruebas.md`: tabla para documentar pruebas.
- `materiales/contrato_api.md`: contrato funcional de la API de laboratorio.
- `materiales/comandos_curl.md`: ejemplos para enviar eventos al webhook.

> Todos los datos son ficticios. No utilices información real, productiva o sensible.
