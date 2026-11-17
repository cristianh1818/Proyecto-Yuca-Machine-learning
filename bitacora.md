# Bitácora

## 2026-11-16 — Fuente de datos
- Problema: elegir fuente de datos.
- Alternativas: TFDS cassava, Kaggle 2020, Mendeley.
- Decisión: TFDS cassava.
- Motivo: tiene particiones oficiales ya documentadas y un tamaño manejable en Colab; Kaggle exige partición propia y Mendeley solo tiene 3 clases.
- Próximo paso: probar descarga y contar imágenes por clase.

## 2026-11-17 — Hipótesis y presupuesto de entrenamiento
- Problema: fijar condiciones comparables antes de entrenar.
- Alternativas: número fijo de épocas / épocas máximas con parada temprana.
- Decisión: máximo 10 épocas con EarlyStopping (val_loss, paciencia 3) para todos los experimentos.
- Motivo: evita entrenar de más y usa solo validación para decidir cuándo parar; el presupuesto es el mismo para todos.
- Resultado: hipótesis E0–E3 y experimento adicional registradas en `protocolo.md` antes de ejecutar.

## 2026-11-17 — Backbone B
- Problema: ResNet18 no existe en `tf.keras.applications`.
- Alternativas: cambiar a PyTorch solo para E3 / usar un ResNet disponible en Keras.
- Decisión: ResNet50V2 de Keras.
- Motivo: el enunciado permite equivalentes de la misma biblioteca; mezclar bibliotecas rompería la comparabilidad (preprocesamiento, entrenamiento, medición de tiempos).
- Revisión: al comparar tamaños, recordar que ResNet50V2 es más grande que ResNet18.

## 2026-11-17 — Normalización
- Problema: cada backbone espera los píxeles en una escala distinta.
- Decisión: guardar imágenes en 0–255 y normalizar dentro de cada modelo (capa Rescaling).
- Motivo: el mismo pipeline de datos sirve para todos y no se puede olvidar el paso al predecir una imagen nueva.

## 2026-11-17 — EDA
- Prueba: conteo por clase, resoluciones, ejemplos y duplicados exactos (hash MD5).
- Resultado: CMD concentra el 47,0 % del entrenamiento (2.658) y healthy el 5,6 % (316); razón de desbalance 8,4. Las tres particiones tienen las mismas proporciones (estratificadas).
- Resoluciones variables: lado corto ≈500 px, lado largo entre 500 y 1.185 px (fotos verticales, horizontales y cuadradas). Redimensionar a 224×224 deforma algunas imágenes y pierde detalle fino.
- Ejemplos: fotos de plantas completas con suelo y malezas de fondo, iluminación muy variable, síntomas leves que se parecen a hojas sanas.
- Decisión derivada: macro-F1 como métrica principal por el desbalance.

## 2026-11-17 — Fuga de datos por duplicados
- Problema: se encontraron imágenes idénticas repetidas dentro de las particiones (18 train, 2 validación, 1 prueba) y compartidas entre ellas (11 train–validación, 10 train–prueba, 4 validación–prueba).
- Alternativas: (a) ignorarlo por ser pocas; (b) eliminar de validación y prueba lo que ya está en entrenamiento; (c) rehacer las particiones.
- Decisión: (b) extendida a todas: recorrer train → validación → prueba y quitar toda imagen ya vista.
- Motivo: el enunciado pide evitar que imágenes de entrenamiento aparezcan en prueba; son pocas, así que se conservan las particiones oficiales casi intactas y la comparación sigue siendo reproducible.
- Resultado: con el primer criterio se excluían 18 de train, 13 de validación y 11 de prueba (`results/indices_excluidos.json`).
- Hallazgo adicional: 18 imágenes repetidas tienen ETIQUETAS DISTINTAS. En casi todos los casos la misma foto aparece a la vez como CBB, CBSD y CGM. Es ruido de etiquetado.
- Revisión de la decisión: quedarse con la primera aparición dejaría en entrenamiento una etiqueta elegida al azar. Se decide quitar esas imágenes de todas las particiones y luego aplicar el filtro de duplicados. Lista final en `results/indices_excluidos_final.json`.
- Limitación: solo se detectan copias exactas; fotos casi iguales (otra toma de la misma planta) no se detectan.

## 2026-11-17 — Error de dependencias en Colab
- Problema: `tfds.load` falló en Colab por una librería faltante (`importlib_resources`).
- Decisión: instalarla al inicio de la celda de carga con `!pip install -q importlib_resources`.
- Resultado: el dataset se descargó (1.291 MB, 9.430 imágenes) y la carga funcionó en los tres notebooks.
- Datos finales tras el filtro: 5.620 entrenamiento, 1.876 validación, 1.874 prueba.

## 2026-11-17 — E0 baseline
- Alternativas: clasificador lineal sobre píxeles / CNN pequeña.
- Decisión: CNN de 3 bloques convolucionales (32-64-128 filtros), GAP, Dropout 0,3 (93.893 parámetros).
- Motivo: un clasificador lineal sobre 224×224×3 píxeles no aprovecha la estructura espacial; la CNN pequeña es sencilla pero razonable.
- Datos tras el filtro: se excluyeron 36 de entrenamiento, 13 de validación y 11 de prueba (validación final: 1.876 imágenes).
- Resultado (validación): accuracy 0,638 | macro-F1 0,306 | 10 épocas (EarlyStopping no se activó; mejores pesos en la época 8) | 2,97 min | 1,18 MB | 1,1 ms por imagen.
- Resultado (prueba): accuracy 0,628 | macro-F1 0,299 (coherente con validación).
- F1 por clase (validación): CMD 0,78 | CBSD 0,65 | CBB 0,10 | CGM 0,00 | healthy 0,00.
- Revisión: la hipótesis se cumplió. El modelo predice casi solo CMD y CBSD; nunca predice CGM ni healthy. 205 de 257 CGM se clasifican como CMD. Las curvas muestran subajuste (accuracy de entrenamiento estancado en ≈0,61), no sobreajuste: la red es demasiado simple para estas imágenes.

## 2026-11-17 — E1 MobileNetV2 congelado
- Prueba: MobileNetV2 (ImageNet) congelado, solo se entrena la cabeza (6.405 de 2.264.389 parámetros). Mismo presupuesto que E0.
- Resultado (validación): accuracy 0,765 | macro-F1 0,672 | 9 épocas (EarlyStopping; mejores pesos en la época 6) | 2,38 min | 9,7 MB | 5,3 ms por imagen.
- Resultado (prueba): accuracy 0,764 | macro-F1 0,671.
- F1 por clase (validación): CMD 0,86 | CBSD 0,73 | healthy 0,68 | CGM 0,64 | CBB 0,45.
- Revisión: la hipótesis se cumplió. El macro-F1 pasa de 0,31 a 0,67 y las mayores mejoras están en las clases que E0 nunca predecía (CGM 0 → 0,64; healthy 0 → 0,68). Persisten las confusiones CMD ↔ CGM y CBB → CBSD. Las curvas muestran el inicio de sobreajuste de la cabeza al final (la pérdida de validación sube en la época 9), que el EarlyStopping detuvo.
- Costo: el modelo es 8 veces más grande y 5 veces más lento en inferencia que E0, pero entrenó en menos tiempo porque solo ajusta 6.405 parámetros.
- Próximo paso: E2 (fine-tuning del último bloque) y E3 (ResNet50V2 congelado).

## Declaración de uso de IA
- Herramienta: asistente de IA generativa (Claude).
- Usos: explicación de conceptos, estructura del repositorio, redacción del protocolo e hipótesis, generación de los notebooks base y apoyo en el análisis.
- Verificación del grupo: los notebooks fueron ejecutados en Google Colab por el grupo; los resultados provienen de esas ejecuciones.
