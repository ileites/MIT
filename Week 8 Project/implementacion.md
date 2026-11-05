# EasyVisa – Clasificación de visas (Week 8 Project 2)

> Notas de implementación y checklist de avance. La entrega final va en **inglés**, en primera persona y con observaciones concisas.
> Leyenda: `[x]` hecho · `[ ]` pendiente · `[~]` en progreso · `[-]` no aplica

---

## 1. Resumen del proyecto

- **Contexto:** la OFLC procesa cientos de miles de solicitudes de certificación laboral por año. EasyVisa quiere un modelo de ML que ayude a preseleccionar las solicitudes con mayor probabilidad de aprobación.
- **Objetivos:**
  1. Facilitar el proceso de aprobación de visas.
  2. Recomendar un perfil de solicitante que debería ser certificado o denegado, según los factores que más influyen en `case_status`.
- **Tipo de problema:** clasificación binaria (`Certified` = 1, `Denied` = 0).
- **Modelos requeridos:** Logistic Regression (datos escalados), Decision Tree y Random Forest (datos codificados, sin escalar).

## 2. Archivos

| Archivo | Rol |
|---|---|
| `EasyVisa.csv` | Dataset (25,480 filas, 12 columnas) |
| `Ignacio Leites - Week 8 project.ipynb` | **Notebook de trabajo / entrega** (copia del Full Code) |
| `AI&DS_Week8_Project2_Full_Code_Notebook_EasyVisa_.ipynb` | Plantilla original (celdas de código vacías) |
| `AI&DS_Week8_Project2_Low_Code_Notebook_EasyVisa.ipynb` | Guía / solución de referencia con blanks `_______` |
| `Week9_Project2_Presentation_Template_EasyVisa.pptx` | Plantilla de la presentación final |
| `implementacion.md` | Este archivo |

**Enfoque:** seguir la estructura y el código del Low Code notebook, manteniendo las respuestas simples.

## 3. Diccionario de datos

| Columna | Descripción |
|---|---|
| `case_id` | ID de la solicitud (se elimina, no aporta) |
| `continent` | Continente del empleado |
| `education_of_employee` | Nivel educativo del empleado |
| `has_job_experience` | ¿Tiene experiencia laboral? (Y/N) |
| `requires_job_training` | ¿Requiere capacitación? (Y/N) |
| `no_of_employees` | Cantidad de empleados de la empresa (hay 33 valores negativos → `abs()`) |
| `yr_of_estab` | Año de fundación de la empresa |
| `region_of_employment` | Región de EE.UU. donde trabajará |
| `prevailing_wage` | Salario de referencia para el puesto en la zona |
| `unit_of_wage` | Unidad del salario (Hour, Week, Month, Year) |
| `full_time_position` | ¿Es tiempo completo? (Y/N) |
| `case_status` | **Target**: Certified / Denied |

---

## 4. Checklist del notebook (Full Code)

### 4.1 Setup
- [x] Quitar celda `%pip install` (uso mi env `mit-env`, ya tiene las versiones requeridas)
- [x] Imports (pandas, numpy, matplotlib, seaborn, sklearn: `train_test_split`, `StandardScaler`, `LogisticRegression`, `DecisionTreeClassifier`, `RandomForestClassifier`, métricas)
- [x] Cargar `EasyVisa.csv` y copiar a `data`

### 4.2 Overview del dataset
- [x] `head()` / `tail()`
- [x] `shape`
- [x] `info()` (tipos de datos)
- [x] Duplicados (exactos, por `case_id` e ignorando `case_id`) → 0
- [x] Nulos (cantidad y %) → 0
- [x] Observaciones

### 4.3 EDA
- [x] `describe()` – resumen estadístico (numéricas y categóricas)
- [x] Corregir negativos en `no_of_employees` (33 filas, -26 a -11 → valor absoluto)
- [x] Chequeo de `yr_of_estab` (2,108 empresas antes de 1900, se mantienen) y `prevailing_wage` por `unit_of_wage` (salarios por hora muy altos)
- [x] `value_counts()` de cada variable categórica (con %)
- [x] Verificar `case_id` único y eliminarlo → 25,480 × 11
- [x] Funciones univariadas: `labeled_barplot`, `histogram_boxplot`
- [x] Funciones bivariadas: `stacked_barplot`, `distribution_plot_wrt_target`, `certification_rate_by_quartile` (estilo propio)

**Univariado**
- [x] `education_of_employee`
- [x] `region_of_employment`
- [x] `has_job_experience`
- [x] `no_of_employees`
- [x] `yr_of_estab`
- [x] `full_time_position`
- [x] `case_status` (≈66.8% Certified → clases algo desbalanceadas)
- [x] (extra) `continent`, `requires_job_training`, `prevailing_wage`, `unit_of_wage`
- [x] (extra) `prevailing_wage` por `unit_of_wage`: el problema afecta a toda la unidad (Hour 92% > 100 USD/h; Week/Month ≈ anuales; 1,038 Year < 10k) → no se ajusta, se mantiene `unit_of_wage` como feature
- [x] Gráficos con estilo propio (labels más chicos, orden por frecuencia, media/mediana con valor en la leyenda)

**Bivariado**
- [x] Heatmap de correlación de variables numéricas
- [x] Education vs case_status
- [x] Continent vs case_status
- [x] Job experience vs case_status
- [x] Prevailing wage vs region (boxplot)
- [x] Region vs case_status
- [x] Prevailing wage vs case_status
- [x] No. of employees vs case_status
- [x] Year of establishment vs case_status
- [x] Unit of wage vs case_status
- [x] Full time position vs case_status
- [x] (extra) `requires_job_training` vs case_status, u otro análisis “beyond the basics”

**Multivariado**
- [x] Heatmap tasa de aprobación: educación × experiencia
- [x] Heatmap tasa de aprobación: continente × educación
- [x] Colores del target en todos los gráficos bivariados: Certified `forestgreen`, Denied `firebrick`

**Leading questions (responder explícitamente)**
- [x] 1. Distribución de case_status (certified vs denied) → 66.8% / 33.2%
- [x] 2. Impacto del nivel educativo en la aprobación
- [x] 3. Diferencia por experiencia laboral previa
- [x] 4. Efecto del prevailing wage (¿más salario → más aprobación?)
- [x] 5. Diferencias de aprobación por región de EE.UU.
- [x] 6. Influencia del tamaño de la empresa
- [x] 7. Diferencias por continente (mayor y menor tasa)

> ⚠️ El rubric pide un análisis que vaya **más allá** de estas 7 preguntas.

### 4.4 Data Pre-processing
- [x] Tratamiento de nulos → no hay, no hace falta
- [x] Detección de outliers (boxplots + conteo IQR) → no se tratan (valores reales; árboles robustos; LR con scaling)
- [x] Feature engineering → no se crean features (salario anualizado no es posible; edad de empresa = misma info que yr_of_estab)
- [x] Codificar target: Certified → 1, Denied → 0 (sin modificar `data`)
- [x] Separar `X` / `y`
- [x] `pd.get_dummies(X, columns=categorical_features, drop_first=True, dtype=int)` → 21 features
- [x] `train_test_split(test_size=0.30, random_state=1, stratify=y)` → 17,836 / 7,644
- [x] Verificar shapes y proporción de clases en train/test (66.8% / 33.2% en ambos)
- [x] `StandardScaler` (fit solo en train) → `X_train_lr`, `X_test_lr` (solo para Logistic Regression)

### 4.5 Model Building
- [x] Definir criterio de evaluación (costo de FP vs FN) → **métrica principal: F1-score** (+ nota sobre desbalance: stratify + F1, sin remuestreo)
- [x] Función `metrics_score` (classification report + matriz de confusión con % + dict de métricas)
- [x] Logistic Regression – train (sobre `X_train_lr`)
- [x] Logistic Regression – test
- [x] Decision Tree – train
- [x] Decision Tree – test
- [x] Random Forest – train
- [x] Random Forest – test
- [x] Comentar overfitting / gap de generalización de cada modelo

> ✅ **Resuelto (2026-11-04) – criterio de evaluación.**
> - Uso del modelo: **primer filtro**. Preselecciona las solicitudes con alta probabilidad de certificación y después un oficial de OFLC revisa solo esas (menos solicitudes, la decisión final siempre es humana).
> - **FN es el error más costoso:** un candidato calificado marcado como Denied queda afuera y nadie lo vuelve a revisar. **FP es menos costoso:** el oficial lo revisa igual y lo puede denegar.
> - **Contraparte (escrita en el notebook):** para OFLC (proteger a los trabajadores de EE.UU.) el FP es más grave → por eso el modelo nunca aprueba solo; la revisión humana controla los FP.
> - **Métrica principal: F1** (Recall sola no sirve: predecir todo Certified da Recall = 1). **Métrica secundaria: Recall** (desempate).
> - Con este criterio, LR gana en los dos (F1 0.817, Recall 0.881). FN de LR en test: 608 de 5,105.
> - Actualicé en el notebook el markdown del criterio y la observación de LR.

### 4.6 Comparación y selección del modelo final
- [x] Tabla comparativa (F1, Recall, Accuracy, Precision) train y test + gap de F1, ordenada por F1 y Recall de test + gráfico de barras de test
- [x] Identificar el modelo que más sobreajusta (DT, gap 0.253) y el que mejor generaliza (LR, gap 0.001)
- [x] Seleccionar modelo final con justificación basada en métricas y negocio → **Logistic Regression**
- [x] Reportar performance del modelo final en test (matriz de confusión + tabla; lectura como primer filtro: preaprueba 5,898 de 7,644 → ~23% menos revisiones)
- [x] Coeficientes de LR (datos escalados) → drivers de certificación
- [x] (extra) Histograma de probabilidad predicha por clase real (umbral 0.5) + tabla por rango: >0.9 → 95.2% certificados; 0.5–0.7 → 63.1% (zona gris); <0.5 → ~1/3 certificados (los FN)
- [x] (extra) Análisis de error: tasas de FN/FP por educación, unit_of_wage y experiencia (tabla + barras)

### 4.7 Actionable Insights & Recommendations
- [x] Insights clave del EDA
- [x] Drivers principales de `case_status`
- [x] Perfil recomendado para **certificar**
- [x] Perfil recomendado para **denegar** / revisar
- [x] Recomendaciones de negocio para OFLC / EasyVisa (6: primer filtro, priorizar por probabilidad, no descartar High School/Hour, foco en factores clave, calidad del dato de salario, asesorar a empleadores/solicitantes)
- [x] Cómo el modelo apoya el proceso en adelante (velocidad, menos carga, explicable, próximos pasos: reentrenar, monitorear FN por grupo, probar umbral / class_weight)
- Perfiles: certificar → Master's/Doctorate + experiencia + salario anual = 90.5% (6,529); denegar → High School + hora = 24.8% (395), High School sin experiencia = 31.7% (1,396)

---

## 5. Checklist de la presentación (plantilla .pptx) — no aplica

> No aplica: en la versión Full Code no se entrega presentación, el notebook cubre el criterio 6 de la rúbrica.

- [-] 1. Título (proyecto, curso, fecha)
- [-] 2. Agenda
- [-] 3. Executive Summary – insights y recomendaciones accionables
- [-] 4. Business Problem Overview and Solution Approach
- [-] 5. EDA Results – respuestas a las leading questions (+ link al apéndice de datos)
- [-] 6. Data Preprocessing – duplicados, nulos, outliers, feature engineering, split, encoding, scaling
- [-] 7. Model Performance Summary & Final Model Selection – comparación, modelo final, performance, drivers
- [-] 8. Appendix
- [-] 9. Data Background and Contents
- [-] 10. Model Building – métricas de cada modelo y comentario de overfitting
- [-] 11. Slide extra (si hace falta)

---

## 6. Resultados / hallazgos (completar a medida que avanzo)

### Overview
- Filas / columnas: 25,480 × 12 (9 object, 2 int64, 1 float64)
- Duplicados: 0
- Nulos: 0

### EDA
- Q1: 66.8% Certified, 33.2% Denied (desbalanceado ~2:1)
- Q2: a más educación, más aprobación (Doctorate 87.2%, Master's 78.6%, Bachelor's 62.2%, High School 34.0%)
- Q3: con experiencia 74.5% vs sin experiencia 56.1%
- Q4: en conjunto los certificados tienen salario mayor, pero es efecto de Hour. Solo anuales: sin diferencia (68.7–71.6% por cuartil). Importa la unidad (Hour 34.6% certificado), no el monto
- Q5: Midwest 75.5%, South 70.0%, Northeast 62.9%, West 62.3%, Island 60.3%
- Q6: efecto débil; solo el cuartil más grande (>3,500 empleados) sube a 69.6% vs ~66%
- Q7: mayor Europe 79.2%, menor South America 57.9%
- Otros hallazgos: requires_job_training y full_time_position casi sin efecto; yr_of_estab casi sin efecto; numéricas sin correlación entre sí

### Modelos (test)

| Modelo | Accuracy | Precision | Recall | F1 | Gap train-test |
|---|---|---|---|---|---|
| Logistic Regression | 0.737 | 0.762 | 0.881 | 0.817 | 0.001 (no overfit) |
| Decision Tree | 0.665 | 0.752 | 0.743 | 0.748 | 0.253 (overfit fuerte) |
| Random Forest | 0.723 | 0.771 | 0.833 | 0.801 | 0.199 (overfit) |

- FN en test: LR 608 · RF 853 · DT 1,313
- Modelo final: **Logistic Regression**
- Justificación: mejor F1 (principal) y mejor Recall (secundaria, menos FN); mejor generalización (gap 0.001); RF tiene Precision apenas mayor (0.771 vs 0.762) pero los FP los filtra la revisión humana; coeficientes interpretables.
- Drivers (coeficientes, vs categoría base Bachelor's / Africa / Island / Hour): + experiencia, Master's/Doctorate, Midwest/South, salario anual, Europe · − High School (el más fuerte), Asia/North America/South America · ≈0 numéricas, job training, full time.
- Análisis de error: el modelo se apoya mucho en educación. FN concentrados en **High School** (82% de sus certificados perdidos; 13% del test pero 47% de los FN, 288/608) y **salario por hora** (63%, 134 FN); juntos 397/608 (65%). Sin experiencia FN 24% vs 5%. FP casi todos Bachelor's/Master's (1,224/1,401). → Recomendación: no descartar automáticamente High School / Hour predichos Denied; mandarlos a revisión o bajar el umbral. Usar la probabilidad para priorizar la revisión.

---

## 7. Log de avance

| Fecha | Qué hice |
|---|---|
| 2026-11-02 | Creé `implementacion.md` con el plan y los checklists. Agregué la rúbrica. Saqué el `%pip install`. Imports, carga y Overview (con observaciones). |
| 2026-11-02/03 | EDA completo: inicio (resumen, negativos, chequeos, `value_counts`, drop `case_id`), análisis de `prevailing_wage` por unidad, univariado, bivariado (Q1–Q7), multivariado (2 heatmaps). Gráficos con estilo propio (Certified `forestgreen`, Denied `firebrick`). Preprocesamiento completo (sin tratar nulos, outliers ni feature engineering, con justificación; dummies explícitas; split 70/30 estratificado; scaling → `X_train_lr` / `X_test_lr`). Model Building: criterio de evaluación + LR, DT y RF en train y test, con observaciones. |
| 2026-11-04 | Revisé el criterio: el modelo funciona como primer filtro + revisión humana; FN es el error más costoso; F1 principal, Recall secundaria; agregué la contraparte de OFLC. Actualicé el markdown del criterio y la observación de LR. |
| 2026-11-04 | 4.6 completa: tabla train/test con gap, gráfico de test, selección de LR con justificación, performance final en test (lectura como primer filtro), coeficientes de LR como drivers. Notebook re-ejecutado completo sin errores. |
| 2026-11-04 | Extra en 4.6: histograma de probabilidad predicha por clase + tabla por rango, y análisis de error (FN/FP por educación, unidad de salario, experiencia) con observaciones. Re-ejecutado sin errores. |
| 2026-11-04 | 4.7 completa: drivers (EDA + modelo), perfiles certificar/denegar, 6 recomendaciones para OFLC/EasyVisa, cómo el modelo apoya el proceso. |

### ▶️ Próxima sesión – retomar acá
1. ~~Revisar el criterio de evaluación~~ → resuelto: F1 principal + Recall secundaria (ver 4.5).
2. ~~4.6 Comparación y selección del modelo final~~ → hecho (LR).
3. ~~4.7 Actionable Insights & Recommendations~~ → hecho.
4. Revisar los puntos de la rúbrica sin tildar (sección 8) y seguir con la **presentación** (.pptx).

---

## 8. Rúbrica de evaluación (60 puntos)

| # | Criterio | Puntos |
|---|---|---|
| 1 | Exploratory Data Analysis | 12 |
| 2 | Data Pre-processing | 12 |
| 3 | Model Building: Logistic Regression, Decision Tree, and Random Forest | 14 |
| 4 | Model Performance Comparison and Final Model Selection | 8 |
| 5 | Actionable Insights & Recommendations | 6 |
| 6 | Presentation / Notebook – Overall quality | 8 |
| | **Total** | **60** |

### 1. Exploratory Data Analysis (12 pts)
> Problem definition; univariate analysis; bivariate analysis; multivariate analysis; use appropriate visualizations to identify the patterns and insights; provide key meaningful observations on individual variables and the relationship between variables.

### 2. Data Pre-processing (12 pts)
> Perform missing value detection and treatment; outlier detection and treatment; feature engineering; prepare data for modeling, including encoding, the train-test split, and scaling where needed. Perform each step only if needed, and clearly explain your reasoning, including why you did or did not apply it.

### 3. Model Building: Logistic Regression, Decision Tree, and Random Forest (14 pts)
> Define the model evaluation criterion by identifying the cost of false positives and false negatives for this business problem, and choose an appropriate primary metric. Build a Logistic Regression model on the scaled data. Build a Decision Tree model on the encoded data. Build a Random Forest model on the encoded data. Evaluate each model on both the training and test data, and comment on overfitting or the generalization gap.

### 4. Model Performance Comparison and Final Model Selection (8 pts)
> Compare the training and test performance of all three models on the chosen primary metric, plus Accuracy, Precision, and Recall. Identify which model overfits the most and which model generalizes best. Select a final model with a clear, metric-based justification tied to the business problem. Report the final model's performance on the test data.

### 5. Actionable Insights & Recommendations (6 pts)
> Summarize the key drivers of visa certification and denial based on the EDA and the model. Recommend a profile of applicants likely to be certified and a profile likely to be denied. Provide clear, practical recommendations for OFLC or EasyVisa based on the analysis. State how the chosen model can support the certification process going forward.

### 6. Presentation / Notebook – Overall quality (8 pts)
> Structure and flow; crispness; visual appeal; conclusion and business recommendations OR structure and flow; well commented code; conclusion and business recommendations.

### Puntos de la rúbrica que no están explícitos en el Low Code (no olvidar)
- [x] **Análisis multivariado** en el EDA: la rúbrica lo nombra pero no dice qué hacer. El heatmap de correlación del Low Code probablemente alcanza; si quiero reforzarlo, puedo sumar un pairplot con `hue="case_status"` (opcional).
- [x] **Definición del problema** al inicio del EDA.
- [x] Preprocessing: explicar **por qué sí o por qué no** se aplica cada paso (nulos, outliers, feature engineering).
- [x] Comparación: incluir métricas de **train y test** de los 3 modelos (el Low Code solo arma la tabla de test).
- [x] Insights: explicar **cómo el modelo elegido apoya el proceso de certificación** en adelante.
- [x] Código bien comentado.
