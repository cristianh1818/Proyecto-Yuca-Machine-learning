# Declaración de herramientas y ayudas externas

El profesor autorizó el uso de IA generativa en este proyecto, con la condición de declararlo. Aquí se registra qué herramientas se usaron, para qué, qué se consultó, qué respuesta se obtuvo y qué decidió el grupo.

## Herramientas

| Herramienta | Tipo | Uso |
|---|---|---|
| Claude (Anthropic) | Asistente de IA generativa | Explicación de conceptos, redacción de documentos, generación del código base de los notebooks, apoyo en el análisis de resultados y en la preparación de la sustentación |
| Google Colab (GPU T4) | Entorno de ejecución | Ejecución de todos los notebooks; todos los resultados provienen de estas ejecuciones |
| GitHub | Control de versiones | Repositorio, historial de commits y entrega |
| TensorFlow / Keras, TensorFlow Datasets, scikit-learn | Bibliotecas | Carga de datos, modelos preentrenados (ImageNet) y métricas |

## Trazabilidad del uso de IA (Claude)

| Fecha | Consulta del grupo | Respuesta recibida | Decisión del grupo |
|---|---|---|---|
| 2026-11-10 | Cuál de los tres proyectos propuestos (yuca, mamografías, frutas) era más abordable | Comparación de dificultad: la yuca tiene datos listos en TFDS, clasificación simple y no exige interfaz | Se eligió el proyecto de la yuca |
| 2026-11-16 | Qué pide el trabajo y qué llevar a la revisión del día 7 | Explicación del aprendizaje por transferencia, congelado vs. fine-tuning, macro-F1 y lista de evidencias para el control | Se organizó el trabajo según esa lista |
| 2026-11-16 | Descripción, README, protocolo y bitácora iniciales | Borradores de `README.md`, `protocolo.md`, `bitacora.md` y `requirements.txt` | Se subieron al repositorio; el README se ajustó a la estructura original del grupo |
| 2026-11-17 | Código para carga de datos, EDA, E0 y E1 | Notebooks 01–03 comentados | El grupo los ejecutó en Colab; se agregó `pip install importlib_resources` para resolver un error de dependencias |
| 2026-11-17 | Interpretación de los duplicados encontrados en el EDA | Propuesta de eliminar duplicados entre particiones y, luego, las 18 imágenes con etiquetas contradictorias | Se adoptó el criterio; queda en `results/indices_excluidos_final.json` |
| 2026-11-17 | Análisis de E0 y E1 a partir de las salidas reales | Borradores de `analisis_E0.md`, `analisis_E1.md` y `registro_errores.md` (8 errores por modelo) | Se revisaron y subieron |
| 2026-11-17 / 19 | Preparación de la sustentación | Guías de estudio, preguntas probables y cambios que podía pedir el profesor | Usadas para preparar la defensa individual |
| 2026-11-24 | Notebooks de E2, E3, experimento adicional, generalización y demo; plantillas de análisis, informe y guion | Notebooks 04–08 y documentos con marcadores `[COMPLETAR TRAS EJECUTAR]` | [COMPLETAR: qué se ejecutó, qué se modificó] |
| 2026-11-24 | Cómo descargar el dataset de Mendeley | Las fotos están en 3 subcarpetas; se descargan listando las carpetas con la API pública | Se usó Mendeley como dataset externo principal |
| [COMPLETAR] | Análisis de E2, E3, adicional y generalización | [COMPLETAR] | [COMPLETAR] |

## Lo que hizo el grupo

- Ejecutar todos los notebooks en Google Colab y revisar sus salidas.
- Subir el código, los resultados y los documentos al repositorio en commits de días distintos.
- [COMPLETAR por el grupo: decisiones propias, cambios hechos al código o a los textos, revisiones de los análisis]

## Lo que NO hizo la IA

- No generó ni modificó resultados: todas las métricas, matrices y curvas salen de las ejecuciones en Colab (archivos de `results/`).
- No tuvo acceso al repositorio para hacer commits.

> Ninguna cifra del informe se escribió sin una salida real que la respalde; los espacios sin ejecutar están marcados como `[COMPLETAR TRAS EJECUTAR]`.
