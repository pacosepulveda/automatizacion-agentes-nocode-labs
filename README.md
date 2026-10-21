# Automatización y agentes sin código — Laboratorios

Repositorio de laboratorios del curso **Automatización y agentes sin código para perfiles técnicos**.

El curso utiliza principalmente **n8n** para orquestación, eventos, webhooks e integraciones mediante APIs, y **Dify** para asistentes, workflows de IA, RAG y agentes con herramientas controladas.

## Enfoque de los laboratorios

Los laboratorios son progresivos: cada módulo amplía el mismo escenario técnico en lugar de utilizar ejercicios aislados. El hilo conductor inicial es el **triaje y enriquecimiento de alertas técnicas de infraestructura**.

A lo largo del curso el caso evolucionará desde el análisis y diseño del proceso hasta un prototipo integrado con validaciones, APIs, IA, conocimiento documental, trazabilidad y aprobación humana.

## Estructura

```text
modulo-01/
└── lab-01-diseno-automatizacion-tecnica/
    ├── README.md
    ├── ENUNCIADO.md
    └── materiales/
        ├── alertas_ejemplo.json
        ├── casos_candidatos.csv
        ├── catalogo_activos.csv
        └── plantilla_entrega.md

modulo-02/
└── lab-02-primer-workflow-n8n/
    ├── README.md
    ├── ENUNCIADO.md
    └── materiales/
        ├── casos_prueba.json
        └── checklist_pruebas.md

modulo-03/
└── lab-03-webhook-api/
    ├── README.md
    ├── ENUNCIADO.md
    └── materiales/
        ├── alertas_webhook.json
        ├── checklist_pruebas.md
        ├── contrato_api.md
        └── comandos_curl.md
```

## Laboratorios disponibles

### Módulo 1 — Diseño de la automatización

Análisis de procesos candidatos, modelado del workflow, reglas deterministas, riesgos e intervención humana.

### Módulo 2 — Primer workflow con n8n

Construcción manual del flujo de triaje, validación de entradas, rutas con If y Switch, clasificación P1–P4, fallback, pruebas y depuración de errores.

### Módulo 3 — Eventos, webhooks e integración mediante API

Sustitución de la entrada simulada por un Webhook, consulta del catálogo de activos mediante HTTP Request, uso de credenciales, enriquecimiento de datos, registro de evidencias y respuesta estructurada al emisor.

## Uso

Cada laboratorio contiene un `ENUNCIADO.md` con las instrucciones y una carpeta `materiales/` con los datos o archivos necesarios para realizarlo.

Todos los datos utilizados en este repositorio son ficticios y están destinados exclusivamente a formación. No deben sustituirse por credenciales, información productiva o datos corporativos sensibles.

## Soluciones

Las soluciones detalladas para instructor se mantienen fuera de este repositorio y no forman parte de los materiales entregados a los alumnos.
