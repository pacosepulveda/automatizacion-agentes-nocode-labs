# Checklist de evaluación — Laboratorio 5

Completa la tabla después de probar primero el **retrieval** y después el Chatflow completo.

| Caso | ¿Recupera documento relevante? | Documento / fragmento | ¿Respuesta fundamentada? | ¿Evita inventar? | ¿Reconoce ausencia de información? | ¿Fuente/cita visible? | Observaciones |
|---|---|---|---|---|---|---|---|
| RAG-01 |  |  |  |  |  |  |  |
| RAG-02 |  |  |  |  |  |  |  |
| RAG-03 |  |  |  |  |  |  |  |
| RAG-04 |  |  |  |  |  |  |  |
| RAG-05 |  |  |  |  |  |  |  |
| RAG-06 |  |  |  |  |  |  |  |
| RAG-07 |  |  |  |  |  |  |  |
| RAG-08 |  |  |  |  |  |  |  |

## Inspección del retrieval

Para al menos cuatro casos anota:

- consulta enviada;
- documento recuperado;
- fragmento o chunk relevante;
- posición del resultado;
- score si la interfaz lo muestra;
- si el fragmento contiene realmente la evidencia necesaria.

## Preguntas de cierre

1. ¿Qué diferencia hay entre que el retrieval encuentre un fragmento y que el LLM responda correctamente?
2. ¿Por qué RAG-05 es útil para observar la combinación de búsqueda semántica y coincidencia exacta?
3. ¿Qué debería ocurrir en RAG-06 y RAG-07 aunque el modelo conozca respuestas plausibles por entrenamiento previo?
4. ¿Qué aporta Parent-child frente a enviar únicamente fragmentos muy pequeños?
5. ¿Qué función cumple el modelo de embeddings?
6. ¿Qué función cumple el reranker?
7. ¿Por qué Top K demasiado alto puede introducir ruido?
8. ¿Por qué un documento recuperado no debe considerarse automáticamente correcto o actualizado?
9. ¿Qué diferencia existe entre una fuente documental y una instrucción del usuario?
10. ¿Qué información conservarías para auditar una respuesta RAG?

## Experimento opcional

Selecciona uno de los casos semánticos (`RAG-02` o `RAG-08`):

1. observa los resultados con `Top K = 3`;
2. cambia temporalmente a `Top K = 1`;
3. repite la consulta;
4. compara el contexto disponible;
5. restaura `Top K = 3`.

Documenta si el cambio mejora, empeora o no altera la respuesta y explica por qué.
