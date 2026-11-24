# Clasificación de enfermedades de la yuca con modelos preentrenados

Proyecto de la asignatura Aprendizaje Computacional – Ingeniería de Sistemas, Universidad de Córdoba.

## Objetivo
Comparar, bajo un mismo protocolo experimental, modelos preentrenados (MobileNetV2 y ResNet50V2) frente a un baseline para clasificar enfermedades de la yuca a partir de fotos de hojas, evaluando desempeño por clase (macro-F1), generalización y costo computacional, y analizando el efecto del fine-tuning.

## Datos
- Fuente: TensorFlow Datasets – `cassava` (iCassava 2019), versión 0.1.0
- 9.430 imágenes, 5 clases: CBB, CBSD, CGM, CMD y hoja sana
- Particiones oficiales: 5.656 entrenamiento / 1.889 validación / 1.885 prueba
- Después de quitar duplicados y etiquetas contradictorias: 5.620 / 1.876 / 1.874 (índices en `results/indices_excluidos_final.json`)
- Datos externos para generalización: Mendeley Data, *Cassava Leaf Disease Dataset* (India, 228 fotos, 3 clases)

## Experimentos
| Exp. | Descripción | Notebook |
|------|-------------|----------|
| E0 | Baseline: CNN pequeña | `02_E0_baseline.ipynb` |
| E1 | MobileNetV2 congelado (feature extraction) | `03_E1_mobilenetv2.ipynb` |
| E2 | MobileNetV2 con fine-tuning del último bloque | `04_E2_mobilenetv2_finetuning.ipynb` |
| E3 | ResNet50V2 congelado, mismo protocolo que E1 | `05_E3_resnet50v2.ipynb` |
| Extra | Augmentation solo en el entrenamiento de E1 | `06_extra_augmentation.ipynb` |
| Generalización | Mejor modelo evaluado en otro dataset sin reentrenar | `07_generalizacion_otro_dataset.ipynb` |

Métrica principal: **macro-F1**. También se reportan accuracy, F1 y recall por clase, matriz de confusión, tiempo de entrenamiento, tiempo de inferencia y tamaño del modelo. Resumen en `results/tabla_consolidada.md`.

## Estructura
- `notebooks/` – notebooks de cada experimento (01 EDA … 07 generalización) y `08_demo_control.ipynb` para la revisión en vivo
- `results/` – métricas (`.json`), matrices de confusión, curvas, errores, predicciones (`.npy`) y modelos (`.keras`)
- `protocolo.md` – protocolo experimental e hipótesis
- `bitacora.md` – registro fechado de decisiones
- `analisis_E0.md` … `analisis_generalizacion.md` – análisis de resultados
- `registro_errores.md` – errores revisados
- `declaracion_IA.md` – herramientas y ayudas externas usadas
- `informe/informe_final.md` – informe final
- `presentacion/` – guion de diapositivas y guía de defensa
- `requirements.txt` – dependencias

## Ejecución
1. Abrir el notebook en Google Colab con GPU (Entorno de ejecución → T4 GPU).
2. Ejecutar las celdas en orden (*Ejecutar todas*). Cada notebook instala `importlib_resources` y descarga del repositorio lo que necesita (índices excluidos, pesos de E1, etc.).
3. Orden: 01 → 02 → 03 → 04 → 05 → 06 → 07. El 08 solo usa lo que ya está guardado.
4. Al final de cada notebook se descarga un `.zip` con los resultados para subir a `results/`.

Reproducibilidad: semilla 42, imágenes de 224×224, batch 32, Adam, máximo 10 épocas con EarlyStopping (val_loss, paciencia 3). Los pesos de cada modelo quedan en `results/<experimento>.keras`; el de ResNet50V2 (≈95 MB) no cabe en GitHub web, así que se guarda su cabeza (`E3_resnet50v2_congelado_cabeza.npz`) y el notebook lo rearma con los pesos de ImageNet.

## Integrantes
- Cristian Herrera Mendez
- Kevin Vega Machado
- Juan Elias Lopez Espitia

> Proyecto académico. Las predicciones no constituyen un diagnóstico agronómico.
