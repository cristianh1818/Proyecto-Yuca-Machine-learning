<!-- Informe final. Para pasarlo a Word: copiar en un documento con formato APA 7 (Times New Roman 12 o Calibri 11, interlineado doble, márgenes de 2,54 cm, sangría de 1,27 cm). Máximo 10 páginas sin anexos. Los marcadores [COMPLETAR TRAS EJECUTAR] se llenan con las salidas reales de los notebooks 04–07. -->

# Comparación experimental de modelos preentrenados para reconocer enfermedades de la yuca mediante fotografías de hojas

Cristian Herrera Mendez, Kevin Vega Machado y Juan Elias Lopez Espitia

Ingeniería de Sistemas, Universidad de Córdoba

Aprendizaje Computacional

Profesor: Oswaldo Vélez Langs

[COMPLETAR: fecha de entrega]

---

## 1. Problema y pregunta de investigación

Las enfermedades de la yuca dejan marcas visibles en las hojas (manchas, rayas, moteados, deformaciones). Este trabajo compara redes neuronales preentrenadas para clasificar esas marcas en cinco categorías, sin pretender un diagnóstico agronómico: las etiquetas del conjunto de datos son categorías de referencia para un ejercicio académico.

**Pregunta:** ¿qué arquitectura de red neuronal preentrenada ofrece el mejor equilibrio entre desempeño por clase, generalización y costo computacional para clasificar enfermedades de la yuca, bajo un protocolo experimental común?

## 2. Procedencia y control de los datos

**Conjunto principal.** Se usó `cassava` de TensorFlow Datasets (versión 0.1.0), publicado para el reto iCassava 2019 (Mwebaze et al., 2019). Son fotos de campo tomadas con celulares en Uganda y etiquetadas por expertos del NaCRRI. El catálogo no especifica una licencia; se usó con fines académicos.

| Característica | Valor |
|---|---|
| URL | https://www.tensorflow.org/datasets/catalog/cassava |
| Imágenes | 9.430 (5.656 entrenamiento / 1.889 validación / 1.885 prueba) |
| Clases | CBB (tizón bacteriano), CBSD (rayado café), CGM (ácaro verde), CMD (mosaico), healthy (sana) |
| Distribución (entrenamiento) | CMD 47,0 %, CBSD 25,5 %, CGM 13,7 %, CBB 8,2 %, healthy 5,6 % (razón de desbalance 8,4) |
| Resolución | variable: lado corto ≈500 px, lado largo de 500 a 1.185 px; se lleva a 224×224 |
| Particiones | oficiales y estratificadas (las tres tienen las mismas proporciones) |

**Duplicados y fuga de datos.** Con una huella MD5 de los píxeles de cada imagen se encontraron fotos idénticas repetidas dentro de las particiones (18 en entrenamiento, 2 en validación, 1 en prueba) y compartidas entre ellas (11 entrenamiento–validación, 10 entrenamiento–prueba, 4 validación–prueba). Además, 18 imágenes aparecían con etiquetas distintas; en casi todos los casos la misma foto figuraba a la vez como CBB, CBSD y CGM.

**Criterio de exclusión.** Se quitaron de todas las particiones las imágenes con etiquetas contradictorias y, recorriendo entrenamiento → validación → prueba, toda imagen ya vista. Quedaron 5.620 / 1.876 / 1.874 imágenes. Los índices excluidos están en `results/indices_excluidos_final.json` y los usan todos los experimentos. Limitación: el hash solo detecta copias exactas, no fotos casi iguales de la misma planta.

**Observaciones del EDA.** La mayoría de las fotos muestran la planta completa con suelo, hojarasca y malezas de fondo, con iluminación muy variable; algunos síntomas son leves y se parecen a una hoja sana. Esto anticipa confusiones entre clases y el riesgo de que el modelo use el fondo como atajo visual.

**Conjunto externo (generalización).** Mendeley Data, *Cassava Leaf Disease Dataset* (Divyanth et al., 2021): 228 fotos de 512×512 tomadas con cámara de mano en cultivos de Tamil Nadu (India), en 3 clases: sana (91), CBB (49) y mosaico (88), mapeadas a `healthy`, `cbb` y `cmd`. No se mezcló con el entrenamiento: solo se usó para evaluar.

## 3. Hipótesis previas

Escritas en `protocolo.md` antes de ejecutar cada experimento.

| Exp. | Hipótesis |
|---|---|
| E0 | La CNN pequeña aprenderá sobre todo la clase mayoritaria (CMD) y tendrá F1 bajo en healthy y CBB; el accuracy quedará muy por encima del macro-F1. |
| E1 | MobileNetV2 congelado superará claramente a E0, sobre todo en las clases minoritarias, porque los filtros de ImageNet describen bordes, texturas y color. |
| E2 | El fine-tuning mejorará algunos puntos de macro-F1 frente a E1, con más tiempo por época y riesgo de sobreajuste. |
| E3 | ResNet50V2 tendrá macro-F1 similar o algo mayor que E1, pero será varias veces más grande y lento; MobileNetV2 daría mejor equilibrio. |
| Adicional | El augmentation moderado reducirá la brecha entrenamiento–validación y mejorará levemente el macro-F1. |

## 4. Protocolo experimental

- Semilla 42; mismas particiones e índices excluidos en todos los experimentos.
- Imágenes de 224×224; normalización dentro de cada modelo (E0: 0–1; MobileNetV2 y ResNet50V2: −1…1, equivalente a su `preprocess_input`).
- Batch 32; Adam; máximo 10 épocas con EarlyStopping (pérdida de validación, paciencia 3, se restauran los mejores pesos).
- Métrica principal: macro-F1. Se reportan además accuracy, F1 y recall por clase, matriz de confusión, tiempo de entrenamiento, tiempo de inferencia, parámetros y tamaño.
- El conjunto de prueba no se usa para escoger hiperparámetros ni para analizar errores (eso se hace en validación). Sin augmentation en validación ni prueba.
- Entorno: Google Colab con GPU T4, TensorFlow/Keras.

## 5. Experimentos

| Exp. | Diseño | Pregunta |
|---|---|---|
| E0 | CNN de 3 bloques convolucionales (32-64-128), GAP, Dropout 0,3; 93.893 parámetros; lr 1e-3 | ¿Qué se consigue sin una arquitectura sofisticada? |
| E1 | MobileNetV2 (ImageNet) congelado + cabeza GAP–Dropout 0,2–Dense 5; lr 1e-3 | ¿Qué aporta el aprendizaje por transferencia? |
| E2 | Desde E1: se descongelan `block_16` y `Conv_1` (BatchNormalization congelada); lr 1e-5 | ¿Mejora frente a E1 y a qué costo? |
| E3 | ResNet50V2 (ImageNet) congelado + la misma cabeza; lr 1e-3 (ResNet18 no está en Keras) | ¿Qué arquitectura ofrece mejor compromiso entre métricas y costo? |
| Adicional | E1 + volteos horizontal/vertical y rotación de ±0,05 vueltas, solo en entrenamiento | ¿Reduce el sobreajuste y mejora el macro-F1? |
| Generalización | Mejor modelo (por macro-F1 de validación) evaluado en Mendeley sin reentrenar | ¿Funciona con fotos de otro país? |

Comparación primaria: E1 frente a E3 (mismo régimen de extracción de características). Comparación secundaria: E1 frente a E2 (efecto del fine-tuning).

## 6. Resultados

**Tabla 1.** *Desempeño y recursos (validación, 1.876 imágenes; prueba, 1.874).*

| Exp. | Acc. val. | **Macro-F1 val.** | Macro-F1 prueba | Épocas | Entrenamiento (min) | Inferencia (ms/img) | Parámetros (entrenables) | Tamaño (MB) |
|---|---|---|---|---|---|---|---|---|
| E0 | 0,638 | **0,306** | 0,299 | 10 | 2,97 | 1,1 | 93.893 (todos) | 1,18 |
| E1 | 0,765 | **0,672** | 0,671 | 9 | 2,38 | 5,3 | 2.264.389 (6.405) | 9,70 |
| E2 | 0,799 | **0,718** | 0,723 | 10 | 2,77 | 4,9 | 2.264.389 (885.445) | 16,73* |
| E3 | 0,760 | **0,652** | 0,631 | 9 | 3,89 | 6,1 | 23.575.045 (10.245) | 95,07 |
| E1 + aug. | 0,767 | **0,673** | 0,667 | 10 | 3,30 | 2,8 | 2.264.389 (6.405) | 9,70 |

*Nota.* Inferencia medida en validación con la GPU de Colab; cada experimento corrió en una sesión distinta, así que los tiempos solo son comparables cuando la diferencia es grande. \*El archivo de E2 es más pesado porque guarda el estado del optimizador; su arquitectura es la de E1. Fuente: `results/tabla_consolidada.csv`.

**Tabla 2.** *F1 por clase (validación).*

| Exp. | CBB | CBSD | CGM | CMD | healthy |
|---|---|---|---|---|---|
| E0 | 0,10 | 0,65 | 0,00 | 0,78 | 0,00 |
| E1 | 0,45 | 0,73 | 0,64 | 0,86 | 0,68 |
| E2 | 0,52 | 0,78 | 0,70 | 0,89 | 0,70 |
| E3 | 0,41 | 0,73 | 0,61 | 0,86 | 0,65 |
| E1 + aug. | 0,45 | 0,74 | 0,65 | 0,86 | 0,65 |

**Figura 1.** *Macro-F1 por experimento en validación y prueba, con el tamaño de cada modelo* (`results/comparacion_macroF1.png`).

**Figura 2.** *Matrices de confusión de validación de E2 (mejor modelo) y E3* (`results/E2_mobilenetv2_finetuning_matriz_validacion.png`, `results/E3_resnet50v2_congelado_matriz_validacion.png`).

**Tabla 3.** *Generalización: mejor modelo (E2) sin reentrenar, en las 3 clases comunes.*

| Métrica | Uganda (test propio) | India (Mendeley) |
|---|---|---|
| Accuracy | [COMPLETAR] | [COMPLETAR] |
| Macro-F1 | [COMPLETAR] | [COMPLETAR] |
| % predicho como CBSD o CGM | [COMPLETAR] | [COMPLETAR] |

## 7. Análisis

**Aporte de la transferencia (E0 vs E1).** E0 alcanzó 0,638 de accuracy pero solo 0,306 de macro-F1: predijo casi exclusivamente CMD y CBSD y nunca CGM ni healthy (205 de 257 imágenes de CGM se clasificaron como CMD). Sus curvas muestran subajuste: el accuracy de entrenamiento se estancó en ≈0,61. E1 duplicó el macro-F1 (0,672) y reconoció las cinco clases; las mayores ganancias fueron justamente en CGM (0 → 0,64) y healthy (0 → 0,68). Entrenó en menos tiempo que E0 porque solo ajusta 6.405 parámetros, aunque es 8 veces más pesado y unas 5 veces más lento en inferencia. Al final de su entrenamiento la pérdida de validación empezó a subir (inicio de sobreajuste de la cabeza), y el EarlyStopping conservó los pesos de la época 6. Ambas hipótesis se cumplieron.

**Efecto del fine-tuning (E1 vs E2).** Descongelar el último bloque de MobileNetV2 (885.445 parámetros entrenables, learning rate 1e-5) subió el macro-F1 de 0,672 a 0,718 en validación y a 0,723 en prueba, con mejoras en las cinco clases, sobre todo en CBB (+0,07) y CGM (+0,06). Las confusiones grandes de E1 bajaron: CGM → CMD de 80 a 47 y CBSD → CMD de 84 a 37. El costo fue solo de entrenamiento (+16 % de tiempo); la inferencia es la misma porque la arquitectura no cambia. La brecha entre accuracy de entrenamiento y validación creció de 0,01 a 0,06, pero la pérdida de validación siguió bajando hasta la época 10: el presupuesto limitó a E2 antes de que apareciera un sobreajuste dañino. La hipótesis se cumplió. La mejora se atribuye al descongelamiento y al learning rate menor, porque arquitectura y datos son los mismos.

**Comparación de arquitecturas (E1 vs E3).** Con el mismo régimen (backbone congelado, misma cabeza y protocolo), ResNet50V2 obtuvo 0,652 de macro-F1 en validación y 0,631 en prueba, frente a 0,672 y 0,671 de MobileNetV2. Igualó a E1 en las clases grandes (CBSD, CMD) y perdió en las pequeñas (CBB 0,41; CGM 0,61; healthy 0,65), donde el macro-F1 pesa más; su recall de CBB fue 0,29. Todo ello con 10,4 veces más parámetros, 9,8 veces más tamaño (95 MB), 63 % más tiempo de entrenamiento y algo más de tiempo de inferencia. La hipótesis se cumplió en el costo pero no en el desempeño: esperábamos un macro-F1 igual o mayor. La diferencia en validación (−0,02) es pequeña y viene de una sola semilla, pero en ningún caso compensa el costo: MobileNetV2 ofrece el mejor compromiso.

**Experimento adicional.** El augmentation moderado (volteos y rotación de ≈±18°, solo en entrenamiento) eliminó la brecha entre entrenamiento y validación de E1 (de ≈+0,03 a −0,01), pero dejó el macro-F1 igual (0,673 frente a 0,672 en validación; 0,667 frente a 0,671 en prueba). Desplazó las predicciones de CMD hacia healthy (recall de healthy de 0,64 a 0,75). La pérdida de validación seguía bajando en la época 10, así que con este presupuesto la regularización no alcanzó a traducirse en mejor desempeño. La hipótesis se cumplió solo en la reducción del sobreajuste.

**Clases difíciles y desbalance.** En E1, CBB es la clase más difícil (recall 0,34; 60 de 151 se confunden con CBSD, que también produce amarillamiento y manchas cafés), y la frontera CMD ↔ CGM es la principal fuente de error en ambas direcciones (moteado frente a mosaico). E2 reduce ambas confusiones pero no las elimina (CBB → CBSD 56 de 151; CMD ↔ CGM 39 y 47), y E3 las empeora (CBB → CBSD 70; CGM → CMD 83). En todos los modelos, CBB es la clase con peor F1: tiene pocas imágenes (8 %) y su amarillamiento se parece al de CBSD. Seleccionar el modelo por accuracy habría ocultado que E0 ignoraba dos clases; por eso la decisión se toma por macro-F1.

## 8. Errores

Se inspeccionaron 8 errores reales de validación por modelo (`registro_errores.md`). Patrones:

- **Síntomas parecidos:** el moteado de CGM se confunde con el mosaico de CMD; las nervaduras claras de hojas sanas o con CGM leve se confunden con el rayado de CBSD.
- **Fondo y encuadre:** la mayoría de los errores tienen suelo, hojarasca o malezas ocupando buena parte de la imagen, o la planta se ve pequeña; al reducir a 224×224 se pierden lesiones pequeñas.
- **Iluminación:** sol directo, contraluz y sombras fuertes alteran el color de la hoja.
- **Posible ruido de etiquetado:** errores con confianza de 0,88–0,94 en imágenes que visualmente corresponden a la clase predicha (p. ej., #129: etiquetada CBSD, con manchas necróticas típicas de CBB), coherentes con las 18 imágenes con etiquetas contradictorias del EDA.
- **Errores que se repiten entre modelos:** entre las muestras revisadas, la imagen #129 falla igual en E1 y E2 (confianza 0,94 en ambos), la #786 en E0 y E2, la #149 en E0 y E3 y la #1155 en E0 y E1. Cuando modelos distintos coinciden en el error, el problema está probablemente en la imagen o en su etiqueta.
- **El amarillamiento domina sobre las manchas:** en E2 y sobre todo en E3, hojas con CBB y amarillamiento visible se predicen como CBSD con confianza de hasta 0,97 (#134, #846, #852).
- [COMPLETAR TRAS EJECUTAR: errores en el dataset externo.]

## 9. Límites y validez externa

- Las etiquetas son categorías de referencia y contienen ruido; no son una validación agronómica.
- Solo se detectaron duplicados exactos; fotos casi iguales de la misma planta pueden estar repartidas entre particiones.
- Redimensionar a 224×224 deforma imágenes no cuadradas y pierde detalle fino.
- El presupuesto fue corto (máximo 10 épocas) y no se ajustaron hiperparámetros; los resultados comparan configuraciones, no el máximo alcanzable.
- Los tiempos dependen de la GPU asignada por Colab.
- ResNet50V2 sustituye a ResNet18; la comparación de tamaño favorece a MobileNetV2 más de lo que lo haría con ResNet18.
- **Validez externa:** el entrenamiento usa fotos de Uganda. [COMPLETAR TRAS EJECUTAR: caída de desempeño en India.] Ninguno de estos resultados garantiza el desempeño en cultivos colombianos, con otras variedades, clima, cámaras e iluminación.

## 10. Conclusiones

1. El aprendizaje por transferencia es determinante: con la misma configuración, MobileNetV2 congelado duplica el macro-F1 del baseline (0,306 → 0,672) y es el único de los dos que reconoce las cinco clases.
2. El fine-tuning parcial del último bloque de MobileNetV2 (E2) es el mejor modelo: macro-F1 de 0,718 en validación y 0,723 en prueba, con mejoras en las cinco clases y un costo extra solo de entrenamiento.
3. **Respuesta a la pregunta de investigación:** MobileNetV2 ofrece el mejor equilibrio. Congelado, iguala o supera a ResNet50V2 con una décima parte del tamaño, y con fine-tuning parcial mejora todavía más. Una arquitectura más grande no significó mejor desempeño por clase.
4. El augmentation moderado reduce el sobreajuste de E1 pero no mejora el macro-F1 con 10 épocas. [COMPLETAR TRAS EJECUTAR: generalización a India.]
5. La calidad de los datos pesa tanto como el modelo: el control de duplicados evitó una fuga entre entrenamiento y prueba, y el ruido de etiquetado explica parte de los errores con alta confianza.

## Referencias

Deng, J., Dong, W., Socher, R., Li, L.-J., Li, K., & Fei-Fei, L. (2009). ImageNet: A large-scale hierarchical image database. En *2009 IEEE Conference on Computer Vision and Pattern Recognition* (pp. 248–255). IEEE. https://doi.org/10.1109/CVPR.2009.5206848

Divyanth, L. G., Soni, P., & Machavaram, R. (2021). *Cassava leaf disease dataset* (Versión 1) [Conjunto de datos]. Mendeley Data. https://doi.org/10.17632/3832tx2cb2.1

He, K., Zhang, X., Ren, S., & Sun, J. (2016). Identity mappings in deep residual networks. En *Computer Vision – ECCV 2016* (pp. 630–645). Springer. https://doi.org/10.1007/978-3-319-46493-0_38

Mwebaze, E., Gebru, T., Frome, A., Nsumba, S., & Tusubira, J. (2019). *iCassava 2019 fine-grained visual categorization challenge* (arXiv:1908.02900). arXiv. https://arxiv.org/abs/1908.02900

Sandler, M., Howard, A., Zhu, M., Zhmoginov, A., & Chen, L.-C. (2018). MobileNetV2: Inverted residuals and linear bottlenecks. En *Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition* (pp. 4510–4520). https://doi.org/10.1109/CVPR.2018.00474

TensorFlow Datasets. (s. f.). *cassava*. Recuperado el 24 de noviembre de 2026, de https://www.tensorflow.org/datasets/catalog/cassava

---

## Anexos (no cuentan en las 10 páginas)

- Anexo A. Bitácora de decisiones (`bitacora.md`).
- Anexo B. Registro de errores (`registro_errores.md`).
- Anexo C. Declaración de herramientas y uso de IA (`declaracion_IA.md`).
- Anexo D. Repositorio: https://github.com/cristianh1818/Proyecto-Yuca-Machine-learning
