# Laboratorio 2 — Primer workflow de triaje en n8n

En este laboratorio se implementa por primera vez en **n8n** el proceso diseñado en el Laboratorio 1.

El objetivo no es integrar todavía sistemas externos, sino aprender a construir, ejecutar, probar y depurar un workflow visual con datos estructurados.

## Entorno del curso

Accede a n8n desde:

```text
https://n8n-agentes.cloudsecurityinpractice.com/
```

Trabaja siempre con la **cuenta de alumno facilitada por el instructor**. Cada alumno dispone de su propio espacio personal de trabajo en n8n, por lo que los workflows y credenciales de otros alumnos no deben aparecer en tu sesión.

Este laboratorio utiliza **Manual Trigger**, por lo que no es necesario publicar el workflow para realizar las pruebas. La publicación se introducirá en el Laboratorio 3 cuando utilicemos una Production URL de Webhook.

## Qué se construirá

Un workflow que:

1. se ejecuta manualmente;
2. recibe una alerta técnica simulada;
3. valida los campos obligatorios;
4. comprueba la severidad;
5. clasifica la alerta como P1, P2, P3 o P4;
6. deriva los casos problemáticos a revisión manual;
7. produce un resultado estructurado y trazable;
8. permite revisar ejecuciones y diagnosticar errores.

## Archivos

- [`ENUNCIADO.md`](ENUNCIADO.md): instrucciones del laboratorio.
- [`materiales/casos_prueba.json`](materiales/casos_prueba.json): entradas que se utilizarán para validar el workflow.
- [`materiales/checklist_pruebas.md`](materiales/checklist_pruebas.md): plantilla para registrar las pruebas.

## Solución

- [`solucion/M02-LAB01-Triage-Alertas.json`](solucion/M02-LAB01-Triage-Alertas.json): workflow completo exportado desde n8n, listo para importar en otra instancia.

## Continuidad con el curso

En el siguiente laboratorio la entrada manual será sustituida por un **webhook** y los datos de activo dejarán de estar simulados para obtenerse mediante una **API de laboratorio**.