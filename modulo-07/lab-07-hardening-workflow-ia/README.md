# Laboratorio 7 — Hardening de un workflow con IA

En este laboratorio endureceremos la automatización híbrida construida en M06. El objetivo ya no es añadir funcionalidad, sino reducir riesgo antes de pensar en un piloto.

Trabajaremos sobre cuatro superficies:

```text
Webhook de entrada
Integraciones y secretos
RAG / LLM
Fallback y supervisión humana
```

## Qué se hará

1. identificar amenazas y fallos plausibles;
2. proteger el Webhook con autenticación de cabecera;
3. validar tamaño y contenido de entrada antes de procesar;
4. mantener las credenciales fuera de los datos del workflow;
5. endurecer el prompt frente a instrucciones embebidas en datos o documentos;
6. comprobar que la IA no puede sobrescribir la prioridad determinista;
7. ampliar la revisión humana cuando la confianza o el contexto documental sean insuficientes;
8. probar degradación ante fallo de Dify;
9. ejecutar casos adversariales controlados;
10. documentar una política mínima de seguridad y operación.

## Archivos

- [`ENUNCIADO.md`](ENUNCIADO.md): instrucciones completas.
- [`materiales/casos_hardening.json`](materiales/casos_hardening.json): batería de pruebas normales y adversariales.
- [`materiales/matriz_riesgos.md`](materiales/matriz_riesgos.md): plantilla inicial de threat/risk review.
- [`materiales/prompt_hardened.md`](materiales/prompt_hardened.md): versión endurecida del prompt de Dify.
- [`materiales/documento_prompt_injection.txt`](materiales/documento_prompt_injection.txt): documento controlado para probar prompt injection indirecta mediante RAG.
- [`materiales/checklist_seguridad.md`](materiales/checklist_seguridad.md): checklist de controles y pruebas.
- [`materiales/politica_minima_template.md`](materiales/politica_minima_template.md): plantilla de política mínima para el prototipo.

## Prerrequisitos

- M06 funcional.
- Cuenta personal de n8n.
- App Workflow de Dify de M06.
- Knowledge Base privada de M05.
- Identificador de alumno.

Todos los payloads, documentos e instrucciones adversariales de este laboratorio son ficticios y se utilizan únicamente para formación.