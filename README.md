# Clasificación de enfermedades de la yuca con modelos preentrenados

Proyecto de la asignatura Aprendizaje Computacional – Ingeniería de Sistemas.

## Objetivo
Comparar, bajo un mismo protocolo experimental, modelos preentrenados (MobileNetV2 y ResNet18) frente a un baseline para clasificar enfermedades de la yuca a partir de fotos de hojas, evaluando desempeño por clase (macro-F1), generalización y costo computacional, y analizando el efecto del fine-tuning.

## Datos
- Fuente: TensorFlow Datasets – `cassava` (iCassava 2019)
- 9.430 imágenes, 5 clases: CBB, CBSD, CGM, CMD y hoja sana
- Particiones oficiales: 5.656 entrenamiento / 1.889 validación / 1.885 prueba

## Experimentos
| Exp. | Descripción |
|------|-------------|
| E0 | Baseline: CNN pequeña |
| E1 | MobileNetV2 congelado (feature extraction) |
| E2 | MobileNetV2 con fine-tuning del último bloque |
| E3 | ResNet18 congelado, mismo protocolo que E1 |
| Extra | Augmentation en entrenamiento o robustez ante brillo |

Métrica principal: **macro-F1**. También se reportan accuracy, F1 y recall por clase, matriz de confusión, tiempo de entrenamiento, tiempo de inferencia y tamaño del modelo.

## Estructura
- `notebooks/` – notebooks de cada experimento
- `results/` – métricas, matrices de confusión y gráficas
- `protocolo.md` – protocolo experimental e hipótesis
- `bitacora.md` – registro fechado de decisiones
- `requirements.txt` – dependencias

## Ejecución
1. Abrir el notebook en Google Colab (con GPU).
2. Instalar las dependencias: `pip install -r requirements.txt`
3. Ejecutar las celdas en orden. Semilla fija: 42.

## Integrantes
- Cristian Herrera Mendez
- Kevin Vega Machado
- Juan Elias Lopez Espitia

> Proyecto académico. Las predicciones no constituyen un diagnóstico agronómico.
