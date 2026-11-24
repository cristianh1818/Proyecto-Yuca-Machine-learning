## Análisis de generalización – el mismo modelo con otros datos

**Pregunta:** ¿el modelo entrenado con fotos de Uganda (iCassava 2019) funciona con fotos de otro lugar, sin reentrenar?

**Datos externos:** Mendeley Data, *Cassava Leaf Disease Dataset* (Divyanth et al., 2021; DOI 10.17632/3832tx2cb2.1). 228 fotos de campo de Tamil Nadu (India), 512×512 px, 3 clases: sana (91), tizón bacteriano CBB (49) y mosaico CMV (88). Se mapean a nuestras clases `healthy`, `cbb` y `cmd`. No contiene CBSD ni CGM. Notebook: `notebooks/07_generalizacion_otro_dataset.ipynb`.

**Modelo evaluado:** [COMPLETAR TRAS EJECUTAR] (el de mayor macro-F1 de validación entre E1, E2, E3 y E1+aug).

**Cómo se mide:** mismo tamaño (224×224) y la normalización que el modelo lleva adentro; sin reentrenar. Para comparar de forma justa, el mismo modelo se evalúa en nuestro test de Uganda restringido a las mismas 3 clases.

| Métrica (3 clases) | Uganda (test propio) | India (Mendeley) | Cambio |
|---|---|---|---|
| Accuracy | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] |
| **Macro-F1** (predicción entre las 5 clases) | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] |
| Macro-F1 restringido (solo entre cbb, cmd, healthy) | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] |
| F1 CBB | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] |
| F1 CMD | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] |
| F1 healthy | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] |
| % de fotos predichas como CBSD o CGM (clases que no existen en India) | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] |

**Matriz de confusión** (`results/generalizacion_mendeley_matriz.png`): [COMPLETAR TRAS EJECUTAR]

**Errores** (`results/generalizacion_mendeley_errores.png`): [COMPLETAR TRAS EJECUTAR] (diferencias de cámara, encuadre de una sola hoja, fondo, iluminación, variedades)

**Interpretación:** [COMPLETAR TRAS EJECUTAR]

**Límites:** solo 228 fotos y 3 clases; no permite evaluar CBSD ni CGM. Las etiquetas de Mendeley no se verificaron de forma independiente. Kaggle 2020 tiene las 5 clases, pero proviene del mismo programa de recolección en Uganda que iCassava 2019, así que no sería un dominio externo y podría compartir fotos con nuestro entrenamiento.

**Conclusión sobre validez externa:** [COMPLETAR TRAS EJECUTAR] (los resultados de Uganda o India no garantizan el desempeño en cultivos colombianos).
