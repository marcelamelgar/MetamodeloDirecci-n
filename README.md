# Informe — Metamodelo de Dirección (4 semanas)

**Autora:** Marcela Melgar  
**Archivo analizado:** `ReportDataALL_20250404 (1).xlsx`  
**Hojas:** Real=`Real`, Predicted=`Predicted`  
**Horizonte considerado:** 27–29 días (≈ 4 semanas con tolerancia ±2)  
**Corte temporal (train/test 70/30):** 2024-12-05  

---

## Baselines (comparación inicial)
- **Clase mayoritaria (accuracy):** 0.7255  
- **Mejor modelo individual:** `pred_IPBG4J` (accuracy = 0.8235)

*Interpretación:*  
El baseline muestra que incluso sin metamodelo, el mejor modelo individual logra un 82% de acierto. El metamodelo debe superar estos valores para ser competitivo.

---

## Métricas del Metamodelo (TEST)
- **Accuracy:** 0.7333  
- **Precision:** 0.6667  
- **Recall:** 0.4000  
- **F1:** 0.5000  
- **ROC-AUC:** 0.8600  
- **R² (consenso):** -0.7551  
- **Mejor R² individual:** `pred_HFWV8N` = -2.1070  

*Interpretación:*  
- El modelo acierta un 73% de las veces, apenas por encima de la clase mayoritaria, pero por debajo del mejor modelo individual.  
- **Precision (0.67)**: cuando predice subida, acierta 2 de cada 3 veces.  
- **Recall (0.40)**: solo detecta 4 de cada 10 subidas - el modelo tiende a ser conservador.  
- **F1 (0.50)**: equilibrio moderado.  
- **ROC-AUC (0.86)**: muy bueno - las probabilidades sí separan bien positivos y negativos.  
- **R² negativo**: ni el consenso ni el mejor modelo individual logran explicar bien la variabilidad de los precios - los valores numéricos predichos no son confiables como regresión.  

---

## Resumen, Limitaciones y Próximos pasos

### Resumen
- El **metamodelo sí logra mejorar un poco el resultado más simple (predecir siempre la clase mayoritaria)**, lo que demuestra que está aprendiendo patrones reales de los datos.  
  Sin embargo, todavía **no logra superar al mejor modelo individual en exactitud (accuracy)**, lo que indica que la combinación actual de modelos podría aprovecharse mejor.  

- El **ROC-AUC de 0.86** nos dice que el metamodelo **distingue bastante bien entre subidas y bajadas**.  
  El problema está en que, con el umbral estándar (0.5), el balance entre aciertos y errores no es tan bueno, y por eso el **F1 inicial queda en 0.5**.  
  Esto refleja que **el modelo necesita calibrarse mejor**.  

- Cuando vemos la parte de **predicción de precios exactos (R²)**, los resultados salen negativos.  
  Esto significa que los modelos **no logran predecir bien los valores exactos**, aunque sí sirven para identificar la dirección (si sube o baja).  
  En pocas palabras: **son útiles para clasificar la tendencia, pero no para dar un número exacto de precio**.  

- Al **ajustar el umbral de decisión a 0.40**, el **F1 sube hasta 0.8**, lo que es una mejora grande.  
  Esto muestra que **con una buena calibración, el metamodelo puede tener un rendimiento mucho mejor**, especialmente en aplicaciones donde importa más identificar bien la dirección que acertar el valor exacto.  


### Limitaciones identificadas
- **Tamaño reducido del dataset**: solo 66 observaciones en el set de prueba, lo que genera métricas poco estables y sensibles a valores atípicos.  
- **Diversidad limitada de algoritmos**: únicamente se probó Random Forest; no se exploraron alternativas como XGBoost, regresión logística regularizada o ensamblajes más sofisticados (stacking, blending).  
- **Predicción de precios poco efectiva**: el análisis de regresión (R² negativo) indica que los modelos, aunque capturan la dirección, no explican bien la variabilidad numérica de los precios.  
- **Problemas de cobertura en algunos modelos individuales**: columnas como `pred_LFHXNV` presentan valores faltantes, reduciendo la robustez y el valor agregado del consenso.  
- **Poca calibración en las probabilidades**: las métricas (F1, precision/recall) cambian bastante según el umbral, lo que revela que el modelo aún no está bien calibrado.  

### Próximos pasos
- **Probar algoritmos adicionales y más avanzados**: incluir XGBoost, LightGBM, regresión logística regularizada y ensamblajes (stacking con meta-learner) para comparar mejoras frente al baseline.  
- **Calibrar las probabilidades de salida**: aplicar técnicas como *Platt scaling* o *isotonic regression* para mejorar la estabilidad en precision/recall y facilitar decisiones de negocio.  
- **Ampliar y enriquecer el dataset**: extender el horizonte temporal, incorporar más commodities y añadir variables externas (indicadores de mercado, shocks macroeconómicos, estacionalidad, etc.).  
- **Mejorar el tratamiento de datos faltantes y calidad del consenso**: evaluar imputación inteligente de predicciones faltantes o descartar modelos poco confiables.  
- **Explorar selección y reducción de variables**: aplicar *feature selection* o PCA para reducir ruido y mejorar la interpretabilidad del metamodelo.  
- **Validación más robusta**: usar validación cruzada temporal (time series CV) para obtener métricas más estables y evitar sobreajuste al split 70/30.  


---

## Análisis de Resultados y Gráficas

### 1. Distribución de precios reales — `dist_precios_reales.png`
- Muestra precios entre 200–500, con asimetría y dispersión alta.  
- Esto explica por qué el R² es negativo: la variabilidad de precios es difícil de capturar.  

### 2. Distribución de horizontes — `dist_horizon_days.png`
- La mayoría de las predicciones se concentran en el rango esperado (27–29 días).  
- Validación: los datos cumplen con la ventana de análisis definida.  

### 3. Importancia de variables — `feature_importances_top20.png`
- Predominan variables de consenso (`consensus_mean`, `consensus_std`) y métricas históricas de error.  
- El metamodelo sí está **aprendiendo de la diversidad entre modelos individuales**.  

### 4. Matriz de confusión — `cm_test.png`
- Mayoría de aciertos en la clase negativa.  
- Falsos negativos frecuentes - bajo recall (0.40).  
- El modelo falla en detectar subidas.  

### 5. Curva ROC — `roc_test.png`
- Área bajo la curva (AUC = 0.86).  
- Buen desempeño como clasificador probabilístico - las probabilidades son útiles.  

### 6. Curva Precision–Recall — `pr_test.png`
- Área promedio AP = 0.754.  
- Alta precisión inicial, pero recall cae rápido.  
- Confirma que el modelo es más conservador en detectar subidas.  

### 7. Tuning de umbral (F1) — `threshold_f1_test.png`
- F1 máximo = **0.8** en umbral 0.40.  
- Ajustando el umbral, el modelo se vuelve más equilibrado.  

### 8. Barrido de métricas por umbral — `threshold_metrics.png`
- Aumentar umbral - mayor precisión pero menor recall.  
- Se observa el trade-off clásico - decisión depende del costo de falsos positivos vs falsos negativos.  

### 9. Consenso vs Real — `scatter_consensus_vs_real_test.png`
- Puntos dispersos, alejados de la diagonal.  
- Confirma que el consenso **no predice bien valores absolutos** - explica el R² negativo.  

### 10. Serie temporal de un commodity — `ts_1.png`
- Predicciones de modelos individuales dispersas y ruidosas.  
- El consenso suaviza, pero **sigue sin capturar la tendencia real**.  

### 11. Curva Lift — `lift_curve.png`
- La curva está por encima de la línea base - el modelo ayuda a priorizar casos positivos.  
- Útil si el objetivo es ranking o priorización.  

### 12. Faltantes por modelo — `missing_by_model.png`
- Modelos con alta proporción de NaN (ej. `pred_LFHXNV`).  
- Esto limita la confiabilidad del consenso y del metamodelo.  

---

## Artefactos generados
- **Reportes y tablas:**  
  - `metrics_summary.json`  
  - `feature_importances.csv`  
  - `r2_by_model_test.csv`  
  - `predicciones_test.csv`  
  - Archivos EDA: `real_*`, `predicted_*`, `count_models.csv`, `count_commodities_by_type.csv`  

- **Figuras:**  
  - `dist_precios_reales.png`, `dist_horizon_days.png`  
  - `feature_importances_top20.png`  
  - `cm_test.png`, `roc_test.png`, `pr_test.png`  
  - `threshold_f1_test.png`, `threshold_metrics.png`  
  - `scatter_consensus_vs_real_test.png`  
  - `ts_1.png`  
  - `lift_curve.png`, `missing_by_model.png`  

---

## Conclusiones Generales

1. **Capacidad de discriminación mejorada**:  
   El metamodelo logra un **ROC-AUC de 0.86** y una curva Lift favorable, lo que indica que sabe **diferenciar casos de subida vs bajada mejor que los baselines simples**. Esto muestra que tiene valor como clasificador probabilístico, aunque la exactitud global no sea la mejor.

2. **Limitaciones en exactitud absoluta**:  
   En términos de **accuracy**, el metamodelo no supera al mejor modelo individual (`pred_HFWV8N`). Esto sugiere que, si el objetivo es únicamente “adivinar correcto o incorrecto”, algunos modelos individuales siguen siendo más confiables.

3. **Regresión poco útil en su estado actual**:  
   Los resultados de R² fueron negativos, tanto en consenso como en el mejor modelo individual, lo que significa que **no explican bien la variabilidad de los precios reales**. En otras palabras, el metamodelo sí predice dirección, pero no magnitud.

4. **Valor del ajuste de umbral**:  
   Ajustando el umbral de decisión a **0.40**, se logra un **F1 de 0.8**, mucho mayor que el baseline. Esto muestra que el verdadero potencial está en usar el metamodelo como **clasificador calibrado**, no como predictor crudo.

5. **Robustez limitada por dataset**:  
   El conjunto de prueba solo tiene **66 observaciones**, lo que genera métricas inestables y susceptibles a variación por pocos casos. Se requiere **más datos para conclusiones sólidas**.

6. **Modelos con información faltante**:  
   Algunos modelos, como `pred_LFHXNV`, muestran **valores vacíos en todo el horizonte**. Esto afecta la robustez del consenso y debería revisarse si se incluyen o excluyen en la versión final del metamodelo.

7. **Potencial de mejora clara**:  
   - Explorar algoritmos más potentes como **XGBoost, LightGBM o stacking**.  
   - Ampliar el dataset con más commodities y periodos.  
   - Incorporar **variables externas** (indicadores de mercado, shocks de oferta/demanda, tipo de cambio).  
   - Evaluar técnicas de **reducción de dimensionalidad** para evitar ruido en features.  
   - Reforzar la **calibración de probabilidades** para que las salidas puedan usarse directamente en decisiones de negocio.  

---

El metamodelo **aporta valor como clasificador probabilístico** y supera los baselines en discriminación, pero **no reemplaza aún al mejor modelo individual en accuracy** ni es útil en regresión. Su mayor fortaleza está en la flexibilidad de calibración (ej. ajuste de umbral) y su principal debilidad es la falta de datos y la ausencia de técnicas más avanzadas que podrían estabilizar y mejorar los resultados.


