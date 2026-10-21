# Módulo 01 — Laboratorio 01

## Diseño de una automatización técnica

Este laboratorio integra las tres prácticas previstas del módulo:

- **Lab 1.1:** identificación de procesos técnicos candidatos a automatización.
- **Lab 1.2:** diseño del mapa de entrada, procesamiento, decisión y salida.
- **Lab 1.3:** clasificación según riesgo, impacto y nivel de intervención humana.

## Objetivo

Antes de construir un workflow en n8n o introducir IA con Dify, analizaremos si el proceso está suficientemente definido, qué partes pueden automatizarse de forma determinista y dónde debe mantenerse la intervención humana.

El caso utilizado como hilo conductor es el **triaje y enriquecimiento de alertas técnicas de infraestructura**, que se reutilizará y ampliará en los módulos siguientes.

## Archivos

```text
lab-01-diseno-automatizacion-tecnica/
├── README.md
├── ENUNCIADO.md
└── materiales/
    ├── alertas_ejemplo.json
    ├── casos_candidatos.csv
    ├── catalogo_activos.csv
    └── plantilla_entrega.md
```

Empieza por [`ENUNCIADO.md`](./ENUNCIADO.md) y utiliza la plantilla de entrega incluida en `materiales/` para documentar el ejercicio.
