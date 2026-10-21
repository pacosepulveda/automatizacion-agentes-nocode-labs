# Checklist de pruebas — Laboratorio 3

Completa la predicción **antes** de enviar cada evento y registra el resultado real después.

| Caso | Predicción | HTTP recibido | Resultado real | ¿Coincide? | Evidencia / observaciones |
|---|---|---:|---|---|---|
| EVT-01 |  |  |  |  |  |
| EVT-02 |  |  |  |  |  |
| EVT-03 |  |  |  |  |  |
| EVT-04 |  |  |  |  |  |
| EVT-05 |  |  |  |  |  |
| EVT-06 |  |  |  |  |  |
| EVT-07 |  |  |  |  |  |
| EVT-08 |  |  |  |  |  |

## Comprobaciones adicionales

- [ ] Distingo entre Test URL y Production URL del Webhook.
- [ ] El payload externo queda normalizado antes de usarlo.
- [ ] Ninguna credencial aparece escrita en un nodo como texto plano.
- [ ] Una entrada sin `asset_id` no llama a la API de activos.
- [ ] Un activo desconocido termina en revisión manual.
- [ ] La clasificación utiliza los datos devueltos por la API.
- [ ] Una severidad no permitida termina en revisión manual.
- [ ] Las alertas P1 quedan marcadas para revisión humana.
- [ ] Se registra una evidencia para resultados válidos y excepciones.
- [ ] El emisor recibe una respuesta JSON comprensible.
- [ ] Puedo localizar una ejecución concreta en el historial.
- [ ] Puedo explicar error HTTP vs decisión de negocio.

## Preguntas de cierre

1. ¿Qué información aporta ahora la API que antes estaba simulada?
2. ¿Qué ventajas tiene desacoplar el catálogo de activos del workflow?
3. ¿Qué ocurriría si la API estuviera caída?
4. ¿Qué datos incluirías en un log de auditoría?
5. ¿Qué harías para impedir que cualquiera invoque el webhook?
