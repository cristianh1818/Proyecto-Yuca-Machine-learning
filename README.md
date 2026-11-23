# Clasificación de enfermedades de la yuca con modelos preentrenados

Proyecto de la asignatura Aprendizaje Computacional – Ingeniería de Sistemas.

## Objetivo
Comparar, bajo un mismo protocolo experimental, modelos preentrenados (MobileNetV2 y ResNet50V2) frente a un baseline para clasificar enfermedades de la yuca a partir de fotos de hojas, evaluando desempeño por clase (macro-F1), generalización y costo computacional, y analizando el efecto del fine-tuning.

## Datos
- Fuente: TensorFlow Datasets – `cassava` (iCassava 2019)
- 9.430 imágenes, 5 clases: CBB, CBSD, CGM, CMD y hoja sana
- Particiones oficiales: 5.656 entrenamiento / 1.889 validación / 1.885 prueba
- Después de quitar duplicados y etiquetas contradictorias: 5.620 / 1.876 / 1.874

## Experimentos
| Exp. | Descripción |
|------|-------------|
| E0 | Baseline: CNN pequeña |
| E1 | MobileNetV2 congelado (feature extraction) |
| E2 | MobileNetV2 con fine-tuning del último bloque |
| E3 | ResNet50V2 congelado, mismo protocolo que E1 |
| Extra | Augmentation en entrenamiento o robustez ante brillo |

Métrica principal: **macro-F1**. También se reportan accuracy, F1 y recall por clase, matriz de confusión, tiempo de entrenamiento, tiempo de inferencia y tamaño del modelo.

## Estructura
- `notebooks/` – notebooks de cada experimento
- `results/` – métricas, matrices de confusión y gráficas
- `protocolo.md` – protocolo experimental e hipótesis
- `bitacora.md` – registro fechado de decisiones
- `analisis_E0.md` y `analisis_E1.md` – análisis de resultados
- `registro_errores.md` – errores revisados
- `requirements.txt` – dependencias

## Ejecución
1. Abrir el notebook en Google Colab (con GPU).
2. Instalar las dependencias: `pip install -r requirements.txt`
3. Ejecutar las celdas en orden. Semilla fija: 42.


## 2026-11-22 — Planificación de E2
- Problema: definir cómo hacer el fine-tuning sin romper la comparación con E1.
- Alternativas: descongelar todo MobileNetV2 / descongelar solo el último bloque.
- Decisión: descongelar solo el último bloque, partiendo de los pesos de E1, con learning rate 1e-5.
- Motivo: con ≈5.600 imágenes, descongelar toda la red tiene alto riesgo de sobreajuste; un learning rate bajo evita borrar lo aprendido en ImageNet.
- Próximo paso: implementar E2 con el mismo protocolo (semilla 42, 224×224, máx. 10 épocas, EarlyStopping) y comparar contra E1.

## Integrantes
- Cristian Herrera Mendez
- Kevin Vega Machado
- Juan Elias Lopez Espitia

> Proyecto académico. Las predicciones no constituyen un diagnóstico agronómico.
