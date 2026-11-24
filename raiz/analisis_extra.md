## Análisis del experimento adicional – Augmentation en E1

**Objetivo declarado:** comprobar si un augmentation moderado SOLO en entrenamiento reduce el sobreajuste de E1 y mejora el macro-F1.

**Configuración:** idéntica a E1 + `RandomFlip("horizontal_and_vertical")` + `RandomRotation(0.05)` (≈ ±18°) dentro del modelo (solo activas al entrenar). Sin cambios de color o brillo para no alterar los síntomas. Validación y prueba sin modificar. Notebook: `notebooks/06_extra_augmentation.ipynb`.

**Resultados (validación):** accuracy 0,767 · macro-F1 0,673 · 10 épocas (la mejor pérdida de validación fue la de la última) · 3,30 min. En prueba: accuracy 0,771 y macro-F1 0,667.

| Métrica | E1 | E1 + augmentation | Cambio |
|---|---|---|---|
| Accuracy | 0,765 | 0,767 | +0,002 |
| **Macro-F1 (validación)** | **0,672** | **0,673** | **+0,001** |
| Macro-F1 (prueba) | 0,671 | 0,667 | −0,003 |
| F1 CBB | 0,45 | 0,45 | +0,01 |
| F1 CBSD | 0,73 | 0,74 | +0,01 |
| F1 CGM | 0,64 | 0,65 | +0,01 |
| F1 CMD | 0,86 | 0,86 | +0,01 |
| F1 healthy | 0,68 | 0,65 | −0,03 |
| Recall healthy | 0,64 | 0,75 | +0,11 |
| Recall CMD | 0,92 | 0,88 | −0,04 |
| Accuracy entrenamiento / validación al final | ≈0,79 / ≈0,76 | 0,761 / 0,767 | brecha de ≈+0,03 a −0,01 |
| Tiempo de entrenamiento | 2,38 min | 3,30 min | +39 % |

**¿Se cumplió la hipótesis?** A medias. La brecha entre entrenamiento y validación desapareció (el modelo ya no se ajusta de más a las fotos de entrenamiento), como esperábamos. Pero el macro-F1 quedó igual (diferencias de ±0,003, dentro de la variación aleatoria), así que no hubo la leve mejora prevista.

**Curvas** (`results/E1_augmentation_curvas.png`): el entrenamiento arranca más lento y la pérdida de validación seguía bajando en la época 10. Con el mismo presupuesto, el augmentation no alcanzó a convertir la menor brecha en mejor desempeño; con más épocas podría hacerlo.

**Por clase:** el modelo predice menos CMD y más healthy (recall de healthy +0,11, de CMD −0,04). Los volteos y giros no cambian la clase, pero sí reducen la dependencia de la orientación del encuadre.

**Costo:** +39 % de tiempo de entrenamiento; ninguno en inferencia ni en tamaño (las capas de augmentation no hacen nada al predecir).

**Conclusión:** el augmentation moderado sirve como regularizador (quita el sobreajuste de E1), pero con 10 épocas no mejora el macro-F1. Para este problema, descongelar el último bloque (E2) aporta mucho más que aumentar los datos.
