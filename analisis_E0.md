## Análisis E0

**Resultados (validación, 1.876 imágenes):** accuracy 0,638 · macro-F1 0,306 · 2,97 min de entrenamiento · 93.893 parámetros (1,18 MB) · 1,1 ms por imagen. En prueba: accuracy 0,628 y macro-F1 0,299, coherentes con validación.

**¿Se cumplió la hipótesis?** Sí. El accuracy (0,64) duplica el macro-F1 (0,31): el modelo acierta sobre todo la clase mayoritaria. Un modelo que dijera siempre CMD tendría ≈0,47 de accuracy, así que E0 sí aprendió algo, pero solo para dos clases.

**Por clase:** CMD tiene F1 0,78 (recall 0,93) y CBSD 0,65; CBB apenas 0,10, y CGM y healthy 0,00: **el modelo nunca predice esas dos clases**. De 1.876 predicciones, 1.220 son CMD y 642 CBSD. Las clases que fallan son las de menos imágenes (healthy 5,6 %, CBB 8,2 %), pero CGM (13,7 %) también falla por parecerse visualmente a CMD.

**Matriz de confusión:** la confusión dominante es **CGM → CMD** (205 de 257). También CBB → CBSD (103 de 151) y healthy → CBSD/CMD (65 y 41).

**Curvas:** la pérdida de entrenamiento y la de validación bajan juntas y el accuracy de entrenamiento se estanca en ≈0,61. **No hay sobreajuste sino subajuste**: la red pequeña no tiene capacidad para aprender rasgos finos (moteados, manchas pequeñas) desde cero con ≈5.600 imágenes. La validación oscila entre épocas; los mejores pesos (época 8) se restauraron al final.

**Errores revisados:** ver `registro_errores.md`. Patrones: el moteado de CGM se confunde con el mosaico; nervaduras claras se confunden con el rayado de CBSD; casi todas las fotos tienen suelo y malezas de fondo; varios errores ocurren con confianza alta (≈0,8).

**Conclusión:** E0 marca el punto de partida: macro-F1 ≈0,31. Para justificar la transferencia, E1 debe mejorar sobre todo en CGM, CBB y healthy, no solo en accuracy.
