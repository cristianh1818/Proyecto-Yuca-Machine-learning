# Registro de errores

Errores revisados en el conjunto de VALIDACIÓN (no en prueba).
Imágenes en `results/<experimento>_errores.png`. Índices referidos a la validación ya filtrada (1.876 imágenes).

## E0 – Baseline (CNN pequeña)

| # | Índice | Clase real | Predicción | Confianza | Observación y posible causa |
|---|---|---|---|---|---|
| 1 | 149 | cgm | cmd | 0,77 | Varias hojas con moteado blanco-amarillento sobre suelo. El moteado del ácaro verde se parece al mosaico; el modelo lo asigna a la clase mayoritaria. |
| 2 | 1263 | cgm | cmd | 0,59 | Hoja con puntos amarillos finos en toda la lámina y fondo oscuro. Al reducir a 224×224 el punteado fino se vuelve un patrón difuso parecido al mosaico. |
| 3 | 1597 | cbb | cbsd | 0,65 | Hoja algo desenfocada sin lesiones evidentes a esta resolución. El síntoma puede ser pequeño (perdido al redimensionar) o estar en otra parte de la planta (la etiqueta es de la planta, no de la hoja visible). |
| 4 | 1155 | cgm | cbsd | 0,42 | Nervaduras claras y amarillamiento leve a lo largo de las venas, que es justo el patrón de CBSD. Confianza baja: el modelo duda. |
| 5 | 786 | cgm | cmd | 0,80 | Moteado claro intenso con hojas que parecen deformadas. Visualmente es muy cercano a CMD; posible coinfección o etiqueta dudosa. Error con confianza alta. |
| 6 | 798 | cgm | cmd | 0,81 | Varias hojas con moteado amarillo y muchas malezas de fondo. Mismo patrón que #149 y #786: el modelo no distingue moteado (CGM) de mosaico (CMD). |
| 7 | 151 | cbb | cmd | 0,45 | Manchas cafés circulares con amarillamiento, síntoma visible de CBB, pero con fondo de pastos y malezas. CBB es solo el 8 % del entrenamiento y el modelo casi nunca la predice. |
| 8 | 1393 | healthy | cbsd | 0,46 | Hoja sana con nervaduras claras resaltadas por la iluminación. Las venas claras se confunden con el rayado amarillo de CBSD. |

### Patrones observados en E0
- **CGM → CMD es la confusión dominante:** 205 de 257 imágenes de CGM en validación se predicen como CMD. El moteado del ácaro y el mosaico son patrones parecidos de manchas claras, y CMD es la clase mayoritaria.
- **El modelo solo usa dos clases:** de 1.876 predicciones, 1.220 son CMD, 642 CBSD, 14 CBB y ninguna CGM ni healthy. Ante la duda, cae en las clases con más imágenes.
- **Nervaduras claras → CBSD:** hojas sanas o con CGM leve que muestran venas claras se asignan a CBSD.
- **Lesiones visibles no bastan:** incluso con manchas claras de CBB (#151) el modelo falla. No aprendió a reconocer esa clase con tan pocos ejemplos.
- **Fondo y varias hojas:** casi todos los errores tienen suelo, hojarasca o malezas ocupando buena parte de la imagen.
- **Errores con confianza alta (0,77–0,81):** el modelo se equivoca "seguro", sobre todo en CGM → CMD.

## E1 – MobileNetV2 congelado

| # | Índice | Clase real | Predicción | Confianza | Observación y posible causa |
|---|---|---|---|---|---|
| 1 | 128 | cbsd | cmd | 0,46 | Hoja verde con venas algo claras, sombras fuertes y malezas de fondo. El síntoma de CBSD es leve y casi no se ve; el modelo duda (confianza baja) y cae en la clase mayoritaria. |
| 2 | 1251 | cgm | cmd | 0,54 | Hojas con moteado claro y algo arrugadas, con luz intensa. El moteado del ácaro sigue confundiéndose con el mosaico, como en E0. |
| 3 | 1560 | cmd | cgm | 0,44 | Plantas pequeñas sobre suelo rojo; la planta ocupa poca parte de la imagen y el suelo domina. Al reducir a 224×224 se pierde la deformación típica de CMD. |
| 4 | 1155 | cgm | healthy | 0,26 | Misma imagen que E0 clasificó como CBSD. Nervaduras claras y síntomas muy leves; confianza muy baja (0,26): caso genuinamente ambiguo. |
| 5 | 772 | cmd | cgm | 0,88 | Muchas hojas pequeñas con puntos amarillos dispersos (patrón de moteado) y un objeto azul en la esquina. Error con confianza alta: posible coinfección o etiqueta dudosa. |
| 6 | 787 | cbb | cbsd | 0,70 | Hoja grande verde oscura a contraluz con una mancha café pequeña en el borde. El síntoma ocupa muy poco de la imagen. |
| 7 | 129 | cbsd | cbb | 0,94 | Hoja amarilla con manchas cafés necróticas y bordes secos, rasgos que corresponden más a CBB. Confianza 0,94: candidato fuerte a **etiqueta errónea** o coinfección. |
| 8 | 1367 | cmd | cgm | 0,50 | Hojas con moteado verde-amarillo y deformación leve, foto cercana. Frontera CMD/CGM. |

### Patrones observados en E1
- **CMD ↔ CGM ahora es en ambas direcciones:** 80 CGM → CMD y 31 CMD → CGM. En E0 el modelo nunca predecía CGM; ahora la reconoce, pero la frontera entre moteado y mosaico sigue siendo la mayor fuente de error.
- **CBB sigue siendo la clase más difícil** (recall 0,34): 60 de 151 se predicen como CBSD. Ambas producen amarillamiento y manchas cafés.
- **CBSD → CMD** (84 de 478) se mantiene.
- **Errores con confianza muy alta (#129, #772)** donde la imagen parece de otra clase: apuntan a ruido de etiquetado, coherente con las 18 imágenes con etiquetas contradictorias del EDA.
- **Síntomas pequeños o planta lejana (#787, #1560):** la reducción a 224×224 y el fondo de suelo dominan la imagen.
- **Comparación con E0:** los errores de E1 son más "razonables": ocurren entre clases visualmente parecidas y con confianza más baja, en lugar de asignar todo a CMD.
