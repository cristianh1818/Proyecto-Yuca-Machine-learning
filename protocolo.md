# Protocolo experimental

## Datos
- Dataset: TFDS `cassava` (iCassava 2019), particiones oficiales: 5.656 entrenamiento / 1.889 validación / 1.885 prueba
- 5 clases: cbb, cbsd, cgm, cmd, healthy
- No se mezclan fuentes.
- Duplicados: el EDA encontró imágenes idénticas (hash MD5) repetidas dentro de las particiones (18 en train, 2 en validación, 1 en prueba) y compartidas entre ellas (11 train–validación, 10 train–prueba, 4 validación–prueba). Además, 18 imágenes repetidas tienen etiquetas distintas (ruido de etiquetado). Criterio: (1) se quitan de todas las particiones las imágenes con etiquetas contradictorias; (2) se recorre train → validación → prueba y se quita toda imagen ya vista. Índices excluidos en `results/indices_excluidos_final.json`.

## Configuración común a todos los experimentos
- Semilla: 42
- Tamaño de imagen: 224x224
- Normalización dentro de cada modelo: E0 → 0…1; MobileNetV2 → −1…1; ResNet → la de `resnet_v2.preprocess_input`
- Batch size: 32
- Optimizador: Adam, learning rate 1e-3 (E2: 1e-5 en el bloque descongelado)
- Presupuesto: máximo 10 épocas, EarlyStopping (val_loss, paciencia 3, restaura mejores pesos)
- Métrica principal: macro-F1. También: accuracy, F1 y recall por clase, matriz de confusión, tiempo de entrenamiento, tiempo de inferencia y tamaño del modelo
- El conjunto de prueba no se usa para escoger hiperparámetros ni para analizar errores (eso se hace en validación)
- Sin augmentation en validación ni prueba

## Arquitecturas
- Backbone A: MobileNetV2 (ImageNet)
- Backbone B: ResNet50V2 (ImageNet). ResNet18 no está disponible en `tf.keras.applications`; el enunciado permite usar un equivalente de la misma biblioteca.

## Hipótesis (escritas antes de ejecutar)
- **E0 (CNN pequeña desde cero):** creemos que aprenderá sobre todo la clase mayoritaria (CMD) y tendrá F1 bajo en las clases con pocas imágenes (healthy y CBB), porque 5.656 imágenes son pocas para aprender desde cero rasgos finos como manchas y moteados. Lo veremos si el accuracy queda claramente por encima del macro-F1 y si la matriz de confusión concentra predicciones en CMD.
- **E1 (MobileNetV2 congelado):** creemos que superará claramente a E0 en macro-F1 y convergerá en menos épocas, porque los filtros aprendidos en ImageNet (bordes, texturas, cambios de color) sirven para describir lesiones aunque nunca hayan visto yuca. La mejora debería ser mayor en las clases minoritarias; esperamos que sigan las confusiones entre CBSD, CMD y hojas sanas.
- **E2 (fine-tuning del último bloque de MobileNetV2):** creemos que mejorará algunos puntos de macro-F1 frente a E1, porque las últimas capas se adaptan a patrones propios de la yuca, pero con mayor tiempo por época y riesgo de sobreajuste (brecha creciente entre curvas de entrenamiento y validación) por el tamaño reducido del dataset.
- **E3 (ResNet50V2 congelado):** creemos que tendrá un macro-F1 similar o algo mayor que E1, pero con un modelo varias veces más grande y más lento en inferencia, por lo que MobileNetV2 ofrecería mejor equilibrio entre desempeño y costo.
- **Experimento adicional (augmentation moderado solo en el entrenamiento de E1):** volteos horizontales/verticales y rotaciones pequeñas. Creemos que reducirá la brecha entre entrenamiento y validación y mejorará levemente el macro-F1, porque una hoja girada conserva el mismo síntoma. No se alteran colores para no cambiar el significado de los síntomas.
