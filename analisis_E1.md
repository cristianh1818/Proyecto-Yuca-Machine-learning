## Análisis E1

**Resultados (validación, 1.876 imágenes):** accuracy 0,765 · macro-F1 0,672 · 9 épocas (EarlyStopping; mejores pesos en la época 6) · 2,38 min · 2.264.389 parámetros, de los cuales solo 6.405 se entrenan · 9,7 MB · 5,3 ms por imagen. En prueba: accuracy 0,764 y macro-F1 0,671, coherentes con validación.

**Comparación con E0 (validación):**

| Métrica | E0 | E1 | Cambio |
|---|---|---|---|
| Accuracy | 0,638 | 0,765 | +0,127 |
| **Macro-F1** | **0,306** | **0,672** | **+0,366** |
| F1 CBB | 0,10 | 0,45 | +0,35 |
| F1 CBSD | 0,65 | 0,73 | +0,08 |
| F1 CGM | 0,00 | 0,64 | +0,64 |
| F1 CMD | 0,78 | 0,86 | +0,08 |
| F1 healthy | 0,00 | 0,68 | +0,68 |
| Tiempo de entrenamiento | 2,97 min | 2,38 min | menor |
| Tamaño | 1,18 MB | 9,7 MB | ×8 |
| Inferencia | 1,1 ms/img | 5,3 ms/img | ×5 |

**¿Se cumplió la hipótesis?** Sí. El macro-F1 más que se duplica y la mejora se concentra en las clases que E0 ignoraba (CGM y healthy pasan de 0 a ≈0,65). El modelo ahora predice las cinco clases. La brecha entre accuracy y macro-F1 se reduce de 0,33 a 0,09: ya no depende solo de la clase mayoritaria.

**Por qué funciona:** los filtros aprendidos en ImageNet ya detectan bordes, texturas y cambios de color, que sirven para describir moteados y manchas aunque la red nunca haya visto yuca. Solo hay que aprender a combinarlos (6.405 parámetros), lo que es posible con ≈5.600 imágenes.

**Curvas:** el accuracy de entrenamiento sigue subiendo (≈0,79) mientras la validación se estabiliza en ≈0,76 y su pérdida sube en la última época: la cabeza empieza a sobreajustarse. El EarlyStopping lo detuvo y restauró los pesos de la época 6.

**Errores (ver `registro_errores.md`):** la confusión principal sigue siendo CMD ↔ CGM, ahora en ambas direcciones (80 y 31). CBB es la clase más difícil (recall 0,34; 60 de 151 se confunden con CBSD). Algunos errores con confianza muy alta (0,88–0,94) sugieren etiquetas erróneas.

**Costo:** el modelo es más pesado y lento en inferencia, pero entrena más rápido que E0 porque casi todo está congelado. 5,3 ms por imagen sigue siendo muy rápido para el uso previsto.

**Conclusión preliminar:** el aprendizaje por transferencia aporta una mejora grande y clara. Queda por ver si el fine-tuning (E2) mejora CBB y la frontera CMD/CGM, y si otra arquitectura (E3) ofrece mejor equilibrio entre desempeño y costo.
