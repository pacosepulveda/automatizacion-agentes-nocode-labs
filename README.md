# Automatización y agentes sin código — Laboratorios

Repositorio de laboratorios del curso **Automatización y agentes sin código para perfiles técnicos**.

El curso utiliza principalmente **n8n** para orquestación, eventos, webhooks e integraciones mediante APIs, y **Dify** para asistentes, workflows de IA, RAG y agentes con herramientas controladas.

## Enfoque de los laboratorios

Los laboratorios son progresivos: cada módulo amplía el mismo escenario técnico en lugar de utilizar ejercicios aislados. El hilo conductor inicial es el **triaje y enriquecimiento de alertas técnicas de infraestructura**.

A lo largo del curso el caso evoluciona desde el análisis y diseño del proceso hasta un prototipo integrado con validaciones, APIs, IA, conocimiento documental, trazabilidad y aprobación humana.

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

modulo-04/
└── lab-04-asistente-tecnico-dify/
    ├── README.md
    ├── ENUNCIADO.md
    └── materiales/
        ├── casos_asistente.json
        ├── prompt_base.md
        ├── schema_salida.json
        └── checklist_evaluacion.md

modulo-05/
└── lab-05-rag-dify/
    ├── README.md
    ├── ENUNCIADO.md
    └── materiales/
        ├── casos_rag.json
        ├── prompt_rag.md
        ├── checklist_evaluacion.md
        └── documentos/
            ├── telvora_inventario_operativo.txt
            ├── telvora_runbook_incidentes.txt
            ├── telvora_backups_operacion.txt
            └── telvora_cambios_despliegues.txt

modulo-06/
└── lab-06-integracion-n8n-dify/
    ├── README.md
    ├── ENUNCIADO.md
    └── materiales/
        ├── casos_integracion.json
        ├── schema_analisis_dify.json
        ├── prompt_workflow_dify.md
        ├── contrato_dify_api.md
        └── checklist_pruebas.md
```

## Laboratorios disponibles

### Módulo 1 — Diseño de la automatización

Análisis de procesos candidatos, modelado del workflow, reglas deterministas, riesgos e intervención humana.

### Módulo 2 — Primer workflow con n8n

Construcción manual del flujo de triaje, validación de entradas, rutas con If y Switch, clasificación P1–P4, fallback, pruebas y depuración de errores.

### Módulo 3 — Eventos, webhooks e integración mediante API

Sustitución de la entrada simulada por un Webhook, consulta del catálogo de activos mediante HTTP Request, uso de credenciales, enriquecimiento de datos, registro de evidencias y respuesta estructurada al emisor.

### Módulo 4 — Asistente técnico con IA en Dify

Construcción de un Chatflow para interpretar el contexto de una alerta, comparar prompts, controlar incertidumbre y acciones prohibidas, generar salida estructurada mediante JSON Schema y evaluar resultados de forma sistemática.

### Módulo 5 — Knowledge Base y RAG en Dify

Creación de una Knowledge Base privada, indexación Parent-child, embeddings con Amazon Titan, Hybrid Search con Cohere Rerank, pruebas de retrieval y construcción de un Chatflow que responda utilizando evidencia documental recuperada y reconozca cuándo la documentación no contiene la respuesta.

### Módulo 6 — Integración n8n + Dify

Construcción de una automatización híbrida donde n8n mantiene validación, APIs y prioridad determinista, invoca mediante API un Workflow publicado de Dify con RAG y Structured Output, valida su respuesta, combina el análisis IA con los datos autoritativos y mantiene trazabilidad y fallback ante fallos del componente de IA.

## Uso

Cada laboratorio contiene un `ENUNCIADO.md` con las instrucciones y una carpeta `materiales/` con los datos o archivos necesarios para realizarlo.

Todos los datos utilizados en este repositorio son ficticios y están destinados exclusivamente a formación. No deben sustituirse por credenciales, información productiva o datos corporativos sensibles.

## Soluciones

Las soluciones detalladas para instructor se mantienen fuera de este repositorio y no forman parte de los materiales entregados a los alumnos.
