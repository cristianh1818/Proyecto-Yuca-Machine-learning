# Tabla consolidada

Métricas de **validación** (1.876 imágenes) salvo las columnas de prueba (1.874). Generada desde los JSON de `results/`. Mejor macro-F1 de validación en negrita.

## Desempeño

| Experimento | Acc. val. | Macro-F1 val. | Acc. prueba | Macro-F1 prueba |
|---|---|---|---|---|
| E0 | 0,6381 | 0,3062 | 0,6275 | 0,2994 |
| E1 | 0,7649 | 0,6722 | 0,7641 | 0,6706 |
| E2 | 0,799 | **0,7183** | 0,7972 | 0,7232 |
| E3 | 0,7596 | 0,6521 | 0,7567 | 0,631 |
| Extra (E1+aug) | 0,7671 | 0,6729 | 0,7711 | 0,6673 |
| Generalización (mejor modelo en Mendeley, 3 clases) | [COMPLETAR] | [COMPLETAR] | [COMPLETAR] | [COMPLETAR] |

## F1 por clase (validación)

| Experimento | CBB | CBSD | CGM | CMD | healthy |
|---|---|---|---|---|---|
| E0 | 0,097 | 0,65 | 0,0 | 0,7842 | 0,0 |
| E1 | 0,4483 | 0,734 | 0,636 | 0,8591 | 0,6834 |
| E2 | 0,5224 | 0,78 | 0,6994 | 0,8868 | 0,703 |
| E3 | 0,4055 | 0,734 | 0,6143 | 0,8611 | 0,6457 |
| Extra (E1+aug) | 0,4533 | 0,7441 | 0,6491 | 0,8646 | 0,6531 |
| Generalización (mejor modelo en Mendeley, 3 clases) | [COMPLETAR] | [COMPLETAR] | [COMPLETAR] | [COMPLETAR] | [COMPLETAR] |

## Recall por clase (validación)

| Experimento | CBB | CBSD | CGM | CMD | healthy |
|---|---|---|---|---|---|
| E0 | 0,053 | 0,7615 | 0,0 | 0,9333 | 0,0 |
| E1 | 0,3444 | 0,7448 | 0,5642 | 0,9208 | 0,6415 |
| E2 | 0,4238 | 0,8494 | 0,6654 | 0,8903 | 0,6698 |
| E3 | 0,2914 | 0,7678 | 0,5175 | 0,9152 | 0,6792 |
| Extra (E1+aug) | 0,3377 | 0,7971 | 0,5759 | 0,8812 | 0,7547 |
| Generalización (mejor modelo en Mendeley, 3 clases) | [COMPLETAR] | [COMPLETAR] | [COMPLETAR] | [COMPLETAR] | [COMPLETAR] |

## Recursos

| Experimento | Épocas | Entrenamiento (min) | Inferencia val. (ms/img) | Parámetros | Entrenables | Tamaño `.keras` (MB) |
|---|---|---|---|---|---|---|
| E0 | 10 | 2,97 | 1,118 | 93.893 | 93.893 | 1,18 |
| E1 | 9 | 2,38 | 5,307 | 2.264.389 | 6.405 | 9,7 |
| E2 | 10 | 2,77 | 4,866 | 2.264.389 | 885.445 | 16,73 |
| E3 | 9 | 3,89 | 6,128 | 23.575.045 | 10.245 | 95,07 |
| Extra (E1+aug) | 10 | 3,3 | 2,812 | 2.264.389 | 6.405 | 9,7 |
| Generalización (mejor modelo en Mendeley, 3 clases) | [COMPLETAR] | [COMPLETAR] | [COMPLETAR] | [COMPLETAR] | [COMPLETAR] | [COMPLETAR] |

> Los tiempos de inferencia se midieron en sesiones de Colab distintas y varían con la GPU asignada (E1 y E1+aug tienen la misma arquitectura y difieren en tiempo); solo las diferencias grandes son confiables. El `.keras` de E2 pesa más que E1 porque guarda el estado del optimizador de 885.445 parámetros entrenables; la arquitectura es la misma.
> La fila de generalización usa solo las 3 clases presentes en Mendeley (cbb, cmd, healthy); no es comparable directamente con las de 5 clases. Ver `analisis_generalizacion.md`.
