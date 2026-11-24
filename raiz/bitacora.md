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

## 2026-11-22 — Planificación de E2
- Problema: definir cómo hacer el fine-tuning sin romper la comparación con E1.
- Alternativas: descongelar todo MobileNetV2 / descongelar solo el último bloque.
- Decisión: descongelar solo el último bloque, partiendo de los pesos de E1, con learning rate 1e-5.
- Motivo: con ≈5.600 imágenes, descongelar toda la red tiene alto riesgo de sobreajuste; un learning rate bajo evita borrar lo aprendido en ImageNet.
- Nota: esta entrada se había guardado por error dentro del README; se movió aquí.

## 2026-11-24 — Mismas fotos en todos los experimentos
- Problema: E2, E3 y el experimento adicional se corren en notebooks y sesiones de Colab distintas; hay que garantizar que usen exactamente las mismas particiones que E0 y E1.
- Alternativas: (a) recalcular los hashes en cada notebook; (b) leer la lista de índices excluidos guardada en el EDA.
- Decisión: (b) `results/indices_excluidos_final.json`, descargado del repositorio, con una verificación (`assert`) de que quedan 1.876 fotos de validación y 1.874 de prueba.
- Motivo: es más rápido y asegura que el filtro sea idéntico.
- Prueba / resultado: la verificación pasó en los notebooks 04, 05 y 06 (1.876 / 1.874) (si no, el notebook se habría detenido antes de entrenar).

## 2026-11-24 — E2: qué capas descongelar
- Problema: el enunciado pide descongelar "un bloque final" de MobileNetV2.
- Alternativas: solo `block_16`; `block_16` + `Conv_1`; los últimos 3 bloques.
- Decisión: `block_16` y la convolución final `Conv_1`, con las BatchNormalization congeladas y learning rate 1e-5, partiendo de los pesos de E1.
- Motivo: es el bloque más cercano a la salida (rasgos más específicos); congelar las BatchNormalization evita que sus estadísticas de ImageNet se desajusten con lotes pequeños.
- Prueba / resultado: 885.445 parámetros entrenables. Validación: accuracy 0,799 | macro-F1 0,718 (E1: 0,672) | 10 épocas sin activar el EarlyStopping | 2,77 min. Prueba: macro-F1 0,723. Mejoraron las cinco clases; CGM → CMD bajó de 80 a 47 y CBSD → CMD de 84 a 37.
- Revisión de la decisión: se mantiene. La pérdida de validación seguía bajando en la época 10, así que el presupuesto limitó a E2; la brecha entre entrenamiento y validación creció de 0,01 a 0,06, por lo que más épocas tendrían que vigilarse.

## 2026-11-24 — E3: normalización y tamaño del modelo
- Problema 1: ResNet50V2 tiene su propia normalización (`resnet_v2.preprocess_input`).
- Decisión: usar `Rescaling(1/127.5, offset=-1)`, que hace la misma operación, y comprobarlo numéricamente en el notebook.
- Motivo: una capa estándar se guarda sin problemas dentro del `.keras`; una capa `Lambda` puede fallar al cargar.
- Problema 2: el `.keras` de ResNet50V2 pesa ≈95 MB y la web de GitHub no acepta archivos de más de 25 MB.
- Decisión: guardar además solo la cabeza entrenada (`E3_resnet50v2_congelado_cabeza.npz`) y rearmar el modelo con los pesos de ImageNet cuando haga falta.
- Prueba / resultado: diferencia ≈0 entre `Rescaling` y `preprocess_input`. Validación: accuracy 0,760 | macro-F1 0,652 (E1: 0,672) | 9 épocas | 3,89 min | 23,6 M parámetros | 95,1 MB. Prueba: macro-F1 0,631.
- Revisión: la hipótesis se cumplió solo en el costo; esperábamos macro-F1 igual o mayor y fue menor, sobre todo en CBB, CGM y healthy. La cabeza (`.npz`) permitió subir el modelo sin el archivo de 95 MB.

## 2026-11-24 — Experimento adicional
- Problema: el enunciado pide un experimento breve y controlado: augmentation solo en entrenamiento o prueba de brillo.
- Alternativas: augmentation en E1; augmentation en E3; perturbación de brillo sobre una copia del test.
- Decisión: augmentation moderado en E1 (volteos horizontal y vertical, rotación de ±0,05 vueltas), sin cambios de color. La prueba de brillo queda en el notebook de demo como modificación en vivo.
- Motivo: E1 es el más barato de reentrenar y mostró inicio de sobreajuste; los cambios de color podrían alterar los síntomas (amarillamiento, manchas cafés).
- Prueba / resultado: validación macro-F1 0,673 (E1: 0,672); prueba 0,667 (E1: 0,671). La brecha entre accuracy de entrenamiento y validación pasó de ≈+0,03 a −0,01. +39 % de tiempo de entrenamiento.
- Revisión: la hipótesis se cumplió en la brecha (menos sobreajuste) pero no en el macro-F1, que quedó igual. La pérdida de validación seguía bajando en la época 10: con el mismo presupuesto, el augmentation no alcanzó a traducirse en mejor desempeño.

## 2026-11-24 — Tiempos de inferencia entre sesiones
- Problema: E1 y E1 + augmentation tienen la misma arquitectura, pero sus tiempos de inferencia salieron distintos (5,3 frente a 2,8 ms por imagen en validación).
- Causa: cada notebook corrió en una sesión de Colab distinta y la GPU o su carga cambia entre sesiones.
- Decisión: comparar tamaños y parámetros (exactos) con confianza, y los tiempos solo cuando la diferencia es grande; se declara como limitación.

## 2026-11-24 — Archivos subidos en una carpeta repetida
- Problema: los resultados de E2 y E3 quedaron en `results/results/` porque se subió la carpeta del zip en vez de sus archivos; el notebook 07 no los encontraba.
- Decisión: volver a subir los archivos directamente a `results/` y borrar la carpeta repetida.

## 2026-11-24 — Generalización con otro dataset
- Problema: el profesor sugirió probar el mismo modelo con otros datos.
- Alternativas: Kaggle Cassava 2020 (5 clases, Uganda) / Mendeley 2021 (3 clases, India).
- Decisión: Mendeley como fuente principal; Kaggle queda como opción en el notebook.
- Motivo: Mendeley es un dominio realmente distinto (otro país, otra cámara, fotos de una hoja) y se descarga sin cuenta. Kaggle 2020 proviene del mismo programa de recolección en Uganda que iCassava 2019, así que no mide validez externa y podría compartir fotos con nuestro entrenamiento.
- Dificultad: la API pública de Mendeley devuelve una lista vacía al pedir los archivos de la carpeta raíz; las fotos están en 3 subcarpetas ("Cassava CB (Cassava Blight)", "Cassava CM (Cassava Mosaic)", "Cassava Healthy leaf"). Se resolvió listando las carpetas y descargando carpeta por carpeta: 49 CBB, 88 mosaico y 91 sanas (228 fotos de 512×512).
- Limitación: solo 3 de nuestras 5 clases; se reporta además el porcentaje de fotos que el modelo asigna a CBSD o CGM, que no existen en ese dataset.
- Prueba / resultado: [COMPLETAR TRAS EJECUTAR]

## 2026-11-24 — Notebook de demostración
- Problema: la revisión en vivo debe ser rápida y no depender de reentrenar.
- Decisión: `08_demo_control.ipynb` carga solo validación, usa los `.keras` y `.npy` del repositorio y tiene celdas para cambiar brillo o rotación y explicar un error concreto.
- Prueba / resultado: [COMPLETAR TRAS EJECUTAR]

## Declaración de uso de IA
Ver `declaracion_IA.md`.
