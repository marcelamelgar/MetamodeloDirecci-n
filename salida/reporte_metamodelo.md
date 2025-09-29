# Informe — Metamodelo de dirección (4 semanas)

**Archivo**: `/Users/marcelamelgar/Downloads/Prueba Técnica - MDS/ReportDataALL_20250404 (1).xlsx`
**Hojas**: Real=`Real`, Predicted=`Predicted`
**Ventana**: 27–29 días
**Corte temporal 70/30**: 2024-12-05

## Baselines
- Clase mayoritaria (acc): **0.7255**
- Mejor modelo individual `pred_IPBG4J` (acc): **0.8235**

## Métricas del metamodelo (TEST)
- accuracy: **0.7333**
- precision: **0.6667**
- recall: **0.4000**
- f1: **0.5000**
- roc_auc: **0.8600**
- r2_consensus: **-0.7551**
- best_r2_model: **pred_HFWV8N**
- best_r2_value: **-2.1070**
- r² (consenso): **-0.7551**
- mejor r² individual: `pred_HFWV8N` = **-2.1070**

**Umbral óptimo (F1)**: 0.40 → F1=0.8000

## Artefactos
- `metrics_summary.json`
- `feature_importances.csv` y `r2_by_model_test.csv`
- `predicciones_test.csv`
- Figuras: `dist_precios_reales.png`, `dist_horizon_days.png`, `feature_importances_top20.png`,
           `cm_test.png`, `roc_test.png` (si aplica), `pr_test.png`, `threshold_f1_test.png`,
           `threshold_metrics.png`, `scatter_consensus_vs_real_test.png`, `lift_curve.png`, `missing_by_model.png`