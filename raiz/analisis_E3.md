## Análisis E3 – ResNet50V2 congelado

**Configuración:** ResNet50V2 de ImageNet congelado, misma cabeza que E1 (GAP → Dropout 0,2 → Dense 5), normalización a [-1, 1] equivalente a `resnet_v2.preprocess_input`, Adam 1e-3, máx. 10 épocas, EarlyStopping. Notebook: `notebooks/05_E3_resnet50v2.ipynb`. Se usa ResNet50V2 porque ResNet18 no está en `tf.keras.applications`.

**Resultados (validación, 1.876 imágenes):** accuracy 0,760 · macro-F1 0,652 · 9 épocas (EarlyStopping; mejores pesos en la época 6) · 3,89 min · 23.575.045 parámetros (10.245 entrenables) · 95,1 MB · 6,1 ms por imagen. En prueba: accuracy 0,757 y macro-F1 0,631.

**Comparación primaria E1 vs E3 (mismo régimen: backbone congelado):**

| Métrica | E1 MobileNetV2 | E3 ResNet50V2 | Cambio |
|---|---|---|---|
| Accuracy | 0,765 | 0,760 | −0,005 |
| **Macro-F1 (validación)** | **0,672** | **0,652** | **−0,020** |
| Macro-F1 (prueba) | 0,671 | 0,631 | −0,040 |
| F1 CBB | 0,45 | 0,41 | −0,04 |
| F1 CBSD | 0,73 | 0,73 | 0,00 |
| F1 CGM | 0,64 | 0,61 | −0,02 |
| F1 CMD | 0,86 | 0,86 | 0,00 |
| F1 healthy | 0,68 | 0,65 | −0,04 |
| Parámetros totales | 2,26 M | 23,58 M | ×10,4 |
| Tamaño | 9,7 MB | 95,1 MB | ×9,8 |
| Tiempo de entrenamiento | 2,38 min | 3,89 min | +63 % |
| Inferencia (validación) | 5,3 ms/img | 6,1 ms/img | +15 % |

**¿Se cumplió la hipótesis?** Solo en parte. Acertamos en que sería mucho más grande y lento, pero esperábamos un macro-F1 igual o algo mayor, y fue menor: −2 puntos en validación y −4 en prueba. La diferencia en validación es pequeña y viene de una sola semilla, así que podría estar dentro de la variación aleatoria; lo que sí es claro es que ResNet50V2 no aporta ninguna mejora que justifique 10 veces más tamaño.

**Por clase:** iguala a E1 en las clases grandes (CBSD, CMD) y pierde en las pequeñas (CBB, CGM, healthy), justo donde el macro-F1 pesa más.

**Matriz de confusión** (`results/E3_resnet50v2_congelado_matriz_validacion.png`): CGM → CMD es la peor de todos los modelos preentrenados (83 de 257) y CBB → CBSD llega a 70 de 151 (recall de CBB 0,29). CBSD → CMD sube a 70.

**Curvas** (`results/E3_resnet50v2_congelado_curvas.png`): la pérdida de validación se estanca alrededor de 0,71 desde la época 5, por encima de la de E1. La brecha con entrenamiento es pequeña (0,78 frente a 0,76 en la mejor época): no es sobreajuste, sino que las características congeladas de ResNet50V2, con esta cabeza y este presupuesto, separan peor las clases minoritarias.

**Errores** (`registro_errores.md`): 5 de los 8 errores revisados son CBB → CBSD, casi siempre con hojas amarillas en las que las manchas cafés existen pero ocupan poco. Repite el error #149 (CGM → CMD) que ya tenía E0.

**Costo y compromiso:** E3 cuesta 10 veces más en tamaño, 63 % más en entrenamiento y algo más en inferencia, y rinde menos. MobileNetV2 ofrece mejor compromiso. Ojo: los tiempos de inferencia dependen de la GPU asignada por Colab en cada sesión, así que solo las diferencias grandes son confiables; la de tamaño sí es exacta.

**Conclusión:** en extracción de características (backbone congelado), MobileNetV2 es mejor opción que ResNet50V2 para este problema: igual o mejor macro-F1 con una décima parte del tamaño. Respuesta a la pregunta de investigación: MobileNetV2, y con fine-tuning parcial (E2) mejora todavía más.

> Nota: el .keras de ResNet50V2 pesa 95 MB y no se puede subir por la web de GitHub; en el repositorio se guarda la cabeza (`E3_resnet50v2_congelado_cabeza.npz`) y el notebook rearma el modelo con los pesos de ImageNet.
