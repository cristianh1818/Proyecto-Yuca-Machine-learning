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

## E2 – MobileNetV2 con fine-tuning

Imágenes en `results/E2_mobilenetv2_finetuning_errores.png`.

| # | Índice | Clase real | Predicción | Confianza | Observación y posible causa |
|---|---|---|---|---|---|
| 1 | 129 | cbsd | cbb | 0,94 | La misma imagen que falló en E1 con la misma confianza: hoja amarilla con manchas cafés necróticas y bordes secos, típicos de CBB. Error persistente entre modelos: candidato fuerte a **etiqueta errónea** o coinfección. |
| 2 | 1254 | cmd | cgm | 0,66 | Hojas con moteado claro difuso y deformación leve. Frontera mosaico/moteado; el fine-tuning movió esta frontera (CMD → CGM subió de 31 a 39). |
| 3 | 1582 | cmd | cbb | 0,39 | Hoja en primer plano desenfocada y suelo con hojarasca rojiza; las manchas amarillas del mosaico se ven como lesiones. Confianza baja: el modelo duda. |
| 4 | 1121 | cbb | cbsd | 0,75 | Hoja verde con amarillamiento en la punta de un lóbulo y suelo seco de fondo. El amarillamiento pesa más que la lesión de CBB. |
| 5 | 786 | cgm | cmd | 0,86 | También falló en E0: moteado claro intenso con hojas que parecen deformadas. Imagen muy cercana a CMD; posible coinfección o etiqueta dudosa. |
| 6 | 797 | cmd | cbsd | 0,52 | Hojas oscuras a la sombra con parches verde claro poco contrastados; la poca luz borra el patrón del mosaico. |
| 7 | 134 | cbb | cbsd | 0,92 | Toda la hoja amarilla con manchas cafés pequeñas en los bordes. El amarillamiento general domina y el modelo lo asocia a CBSD. |
| 8 | 1390 | healthy | cmd | 0,92 | Hoja sana a pleno sol, con el centro casi blanco por sobreexposición: el brillo se parece a los parches claros del mosaico. |

### Patrones observados en E2
- **Errores que se repiten entre modelos:** #129 (E1 y E2, 0,94 en ambos) y #786 (E0 y E2). Si modelos distintos fallan igual y con alta confianza, el problema probablemente está en la imagen o en su etiqueta, no en el modelo.
- **El amarillamiento domina sobre las manchas** (#134, #1121): CBB → CBSD sigue siendo la confusión principal (56 de 151).
- **Iluminación extrema** (#797 sombra, #1390 sobreexposición): alteran justamente los colores que definen los síntomas.
- **Mejoras frente a E1:** CGM → CMD bajó de 80 a 47 y CBSD → CMD de 84 a 37.

## E3 – ResNet50V2 congelado

Imágenes en `results/E3_resnet50v2_congelado_errores.png`.

| # | Índice | Clase real | Predicción | Confianza | Observación y posible causa |
|---|---|---|---|---|---|
| 1 | 148 | cbb | cbsd | 0,33 | Primer plano de hojas con muchos puntos cafés pequeños. Confianza muy baja: el modelo no reconoce el patrón de manchas de CBB. |
| 2 | 1310 | cgm | cmd | 0,81 | Planta densa con hojas moteadas y malezas; el moteado del ácaro se toma como mosaico (la confusión más frecuente de E3: 83 de 257). |
| 3 | 1606 | cbb | cbsd | 0,68 | Hoja grande con sombras fuertes y pocas manchas pequeñas; el síntoma ocupa muy poco de la foto. |
| 4 | 1204 | cbb | cmd | 0,64 | Lesión necrótica grande (típica de CBB) rodeada de hojas moteadas: posible coinfección; el modelo se guía por el moteado del resto de la imagen. |
| 5 | 846 | cbb | cbsd | 0,97 | Hoja verde con un lóbulo amarillo y borde seco. Muy alta confianza en la clase equivocada: el amarillo domina. |
| 6 | 852 | cbb | cbsd | 0,90 | Lóbulos amarillo intenso con manchas cafés; mismo patrón que #846 y que #134 de E2. |
| 7 | 149 | cgm | cmd | 0,67 | La misma imagen que falló en E0: varias hojas con moteado claro sobre suelo. |
| 8 | 1446 | cgm | cmd | 0,83 | Foto desenfocada y pálida, hoja con moteado amarillo difuso. |

### Patrones observados en E3
- **5 de 8 errores son de CBB**, casi todos predichos como CBSD: ResNet50V2 congelado distingue peor las manchas cafés cuando hay amarillamiento (recall de CBB 0,29, el peor de los modelos preentrenados).
- **CGM → CMD empeora** frente a E1 (83 frente a 80) y mucho frente a E2 (47).
- **Errores repetidos con otros modelos** (#149): misma conclusión que en E2.
- **Desenfoque y encuadre** (#1446, #1606) siguen siendo causas frecuentes.

## Generalización – mejor modelo en Mendeley (India)

Imágenes en `results/generalizacion_mendeley_errores.png`.

| # | Clase real | Predicción | Confianza | Observación y posible causa |
|---|---|---|---|---|
| 1 | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] |
| 2 | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] |
| 3 | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] |
| 4 | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] |
| 5 | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] |
| 6 | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] |
| 7 | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] |
| 8 | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] | [COMPLETAR TRAS EJECUTAR] |

### Patrones observados
- [COMPLETAR TRAS EJECUTAR] (diferencias de dominio: encuadre de una sola hoja, cámara, iluminación, variedades)
