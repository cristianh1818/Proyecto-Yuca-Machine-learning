## Análisis E2 – Fine-tuning del último bloque de MobileNetV2

**Configuración:** parte de los pesos de E1; se descongelan `block_16` y `Conv_1` de MobileNetV2 (BatchNormalization congelada); Adam con learning rate 1e-5; máx. 10 épocas con EarlyStopping (val_loss, paciencia 3). Notebook: `notebooks/04_E2_mobilenetv2_finetuning.ipynb`.

**Resultados (validación, 1.876 imágenes):** accuracy 0,799 · macro-F1 0,718 · 10 épocas (el EarlyStopping no se activó; la mejor pérdida de validación fue la de la última época) · 2,77 min · 885.445 parámetros entrenables de 2.264.389 · 16,7 MB · 4,9 ms por imagen. En prueba: accuracy 0,797 y macro-F1 0,723, coherentes con validación.

**Comparación con E1 (comparación secundaria: efecto del fine-tuning):**

| Métrica | E1 | E2 | Cambio |
|---|---|---|---|
| Accuracy | 0,765 | 0,799 | +0,034 |
| **Macro-F1** | **0,672** | **0,718** | **+0,046** |
| F1 CBB | 0,45 | 0,52 | +0,07 |
| F1 CBSD | 0,73 | 0,78 | +0,05 |
| F1 CGM | 0,64 | 0,70 | +0,06 |
| F1 CMD | 0,86 | 0,89 | +0,03 |
| F1 healthy | 0,68 | 0,70 | +0,02 |
| Parámetros entrenables | 6.405 | 885.445 | ×138 |
| Tiempo de entrenamiento | 2,38 min (9 épocas) | 2,77 min (10 épocas) | +16 % |
| Tamaño del archivo `.keras` | 9,7 MB | 16,7 MB | +7 MB* |

\*La arquitectura es idéntica; el archivo crece porque guarda el estado del optimizador Adam (dos valores por cada parámetro entrenable). Los pesos para usar el modelo ocupan lo mismo que E1.

**¿Se cumplió la hipótesis?** Sí en lo principal: el macro-F1 subió 4,6 puntos y mejoraron las cinco clases. El costo de entrenamiento fue menor de lo esperado (+16 %), porque solo se ajusta un bloque. El riesgo de sobreajuste se vio a medias: la brecha entre accuracy de entrenamiento y de validación creció de 0,01 en la primera época a 0,06 en la última (0,863 frente a 0,799), pero la pérdida de validación siguió bajando, así que todavía no hay sobreajuste dañino.

**Por clase:** las mayores ganancias fueron en las clases difíciles: CBB (+0,07) y CGM (+0,06). CBB sigue siendo la peor (recall 0,42).

**Matriz de confusión** (`results/E2_mobilenetv2_finetuning_matriz_validacion.png`): bajaron las confusiones grandes de E1. CGM → CMD pasa de 80 a 47 y CBSD → CMD de 84 a 37. Persisten CBB → CBSD (56 de 151) y aumenta un poco CMD → CGM (31 → 39): la frontera entre moteado y mosaico se movió, no desapareció. Healthy → CBSD sigue en 24.

**Curvas** (`results/E2_mobilenetv2_finetuning_curvas.png`): la pérdida de validación bajó lentamente (0,634 → 0,599) y la mejor época fue la última. El presupuesto de 10 épocas limitó a E2: con más épocas podría mejorar algo más, aunque la brecha creciente indica que habría que vigilar el sobreajuste.

**Errores** (`registro_errores.md`): algunos errores se repiten con otros modelos (#129 con E1, #786 con E0), lo que apunta a imágenes ambiguas o mal etiquetadas más que al modelo. Aparecen errores por amarillamiento generalizado (CBB → CBSD) y por sobreexposición (sana → CMD).

**Costo:** el tiempo de inferencia es prácticamente el mismo que E1 (misma arquitectura); el costo extra es solo de entrenamiento.

**Conclusión:** el fine-tuning parcial mejora de forma consistente (validación y prueba) a un costo bajo. Es el mejor modelo del estudio por macro-F1 de validación. La mejora se atribuye solo al descongelamiento y al learning rate menor, porque la arquitectura y los datos son los mismos que en E1.
