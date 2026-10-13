# Customer Segmentation – Unsupervised Learning Project

> Notas de diseño del proyecto. La entrega final será en inglés; las notas pueden estar en español o inglés.

---

## 1. Project Brief (original assignment)

### Business Context

Understanding customer personality and behavior is pivotal for businesses to enhance customer satisfaction and increase revenue. Segmentation based on a customer's personality, demographics, and purchasing behavior allows companies to create tailored marketing campaigns, improve customer retention, and optimize product offerings.

A leading retail company with a rapidly growing customer base seeks to gain deeper insights into their customers' profiles. The company recognizes that understanding customer personalities, lifestyles, and purchasing habits can unlock significant opportunities for personalizing marketing strategies and creating loyalty programs. These insights can help address critical business challenges, such as improving the effectiveness of marketing campaigns, identifying high-value customer groups, and fostering long-term relationships with customers.

With the competition intensifying in the retail space, moving away from generic strategies to more targeted and personalized approaches is essential for sustaining a competitive edge.

### Objective

In an effort to optimize marketing efficiency and enhance customer experience, the company has embarked on a mission to identify distinct customer segments. By understanding the characteristics, preferences, and behaviors of each group, the company aims to:

1. Develop personalized marketing campaigns to increase conversion rates.
2. Create effective retention strategies for high-value customers.
3. Optimize resource allocation, such as inventory management, pricing strategies, and store layouts.

As a data scientist tasked with this project, your responsibility is to analyze the given customer data, apply machine learning techniques to segment the customer base and provide actionable insights into the characteristics of each segment.

### Data Dictionary

The dataset includes historical data on customer demographics, personality traits, and purchasing behaviors. Key attributes are:

**Customer Information**

- `ID`: Unique identifier for each customer.
- `Year_Birth`: Customer's year of birth.
- `Education`: Education level of the customer.
- `Marital_Status`: Marital status of the customer.
- `Income`: Yearly household income (in dollars).
- `Kidhome`: Number of children in the household.
- `Teenhome`: Number of teenagers in the household.
- `Dt_Customer`: Date when the customer enrolled with the company.
- `Recency`: Number of days since the customer's last purchase.
- `Complain`: Whether the customer complained in the last 2 years (1 for yes, 0 for no).

**Spending Information (Last 2 Years)**

- `MntWines`: Amount spent on wine.
- `MntFruits`: Amount spent on fruits.
- `MntMeatProducts`: Amount spent on meat.
- `MntFishProducts`: Amount spent on fish.
- `MntSweetProducts`: Amount spent on sweets.
- `MntGoldProds`: Amount spent on gold products.

**Purchase and Campaign Interaction**

- `NumDealsPurchases`: Number of purchases made using a discount.
- `AcceptedCmp1`: Response to the 1st campaign (1 for yes, 0 for no).
- `AcceptedCmp2`: Response to the 2nd campaign (1 for yes, 0 for no).
- `AcceptedCmp3`: Response to the 3rd campaign (1 for yes, 0 for no).
- `AcceptedCmp4`: Response to the 4th campaign (1 for yes, 0 for no).
- `AcceptedCmp5`: Response to the 5th campaign (1 for yes, 0 for no).
- `Response`: Response to the last campaign (1 for yes, 0 for no).

**Shopping Behavior**

- `NumWebPurchases`: Number of purchases made through the company's website.
- `NumCatalogPurchases`: Number of purchases made using catalogs.
- `NumStorePurchases`: Number of purchases made directly in stores.
- `NumWebVisitsMonth`: Number of visits to the company's website in the last month.

---

## 2. Grading Rubric (60 points total)

| # | Criteria | Points | Requirements |
|---|----------|--------|--------------|
| 1 | **Exploratory Data Analysis** | 15 | Add the problem definition. Add a data description covering the background of the data and what it contains. Write observations from the data description (shape of the data, data types of attributes, statistical summary). Perform univariate and bivariate analysis of variable distributions and interactions between variables. Provide observations and insights on key patterns in the data, what they mean for the problem formulation, and whether any data treatments or pre-processing are required. |
| 2 | **Data Pre-processing** | 12 | Check duplicate values and missing values and add observations. Write initial observations on the range of attributes and outliers of various attributes. Scale the data before clustering. |
| 3 | **Applying K-means Clustering** | 20 | Create the Elbow curve and add observations. Compute Silhouette scores for different values of k. Select the appropriate number of clusters with reasons for the choice. |
| 4 | **Actionable Insights & Recommendations** | 8 | Add insights for each cluster identified. Provide recommendations that can be acted upon to improve business outcomes (may include additional data sources for further analysis, real-world model implementation, or potential business benefits from improving the model). |
| 5 | **Presentation / Notebook – Overall Quality** | 5 | Clear structure and flow so everything fits into a coherent story. Crisp and focused, with only enough text to support key points. Visual appeal through effective use of charts, colors, diagrams, format and symmetry, with informative and easy-to-interpret visualizations — or a well-structured notebook with logical flow and clean, well-commented code. |

### Checklist de entregables (derivado de la rúbrica)

- [ ] EDA: problem definition
- [ ] EDA: data description (background + contenido)
- [ ] EDA: shape, dtypes, statistical summary + observaciones
- [ ] EDA: análisis univariado
- [ ] EDA: análisis bivariado
- [ ] EDA: insights clave + necesidades de pre-processing
- [ ] Pre-processing: duplicados + observaciones
- [ ] Pre-processing: missing values + observaciones
- [ ] Pre-processing: rangos y outliers + observaciones
- [ ] Pre-processing: escalado de datos
- [ ] K-means: Elbow curve + observaciones
- [ ] K-means: Silhouette scores para varios k
- [ ] K-means: elección de k justificada
- [ ] Insights por cluster
- [ ] Recomendaciones accionables
- [ ] Calidad general: estructura, storytelling, visualizaciones, código comentado

---

## 3. Dataset – primer vistazo

**Archivo:** `Customer_Personality_Segmentation.csv` (~220 KB)

- ⚠️ A pesar de la extensión `.csv`, el separador es **tabulación** → leer con `pd.read_csv(path, sep="\t")`. Fin de línea CRLF.
- **Shape:** 2240 filas × 29 columnas (25 int, 1 float, 3 texto).
- **Columnas extra no documentadas en el Data Dictionary:** `Z_CostContact` (siempre 3) y `Z_Revenue` (siempre 11). Constantes → sin información, candidatas a eliminar.
- **Missing values:** solo `Income`, 24 nulos (~1.1%).
- **Duplicados:** 0 filas duplicadas exactas e IDs únicos, pero **182 filas duplicadas si se excluye `ID`** → mismo cliente registrado con distinto ID. Hay que decidir si eliminarlas.
- **`Dt_Customer`:** texto en formato `dd-mm-yyyy`; rango 2012-07-30 → 2014-06-29. Convertir a datetime (útil para antigüedad del cliente).
- **`Education`:** Graduation 1127, PhD 486, Master 370, 2n Cycle 203, Basic 54.
- **`Marital_Status`:** Married 864, Together 580, Single 480, Divorced 232, Widow 77, y categorías raras: Alone 3, Absurd 2, YOLO 2 → limpiar/agrupar.
- **Valores sospechosos / outliers a revisar:**
  - `Year_Birth`: mínimos 1893, 1899, 1900 (edades > 110 años), el siguiente es 1940.
  - `Income`: máximo 666666 (claramente anómalo); el siguiente es ~162k. Mediana ~51k.
  - Gastos (`Mnt*`) muy asimétricos: media bastante mayor que mediana (ej. `MntWines` mediana 174, media 304, máx 1493).
- **Campañas:** `AcceptedCmp1-5`, `Response` y `Complain` son binarias con tasas bajas.

---

## 4. Notas de diseño

### Enfoque metodológico (hipótesis inicial)

- Tipo de problema: aprendizaje no supervisado → **clustering**.
- Algoritmo probable: **K-means** (alineado con lo visto en el curso).
- Selección de K: mediante **método del codo (Elbow)** y/o **Silhouette score**.

### Entorno de ejecución

- Ejecución **local** en el conda env `mit-env`, con **Python 3.11.17** y kernel de Jupyter `Python 3.11 (mit-env)`.
- **¿Por qué 3.11?** El notebook usa `yellowbrick` (la última versión es la 1.5, de 2022, y ya no se mantiene). 3.11 es la versión con menos riesgo de incompatibilidades.
- Versiones fijadas en `requirements.txt` (en esta carpeta). Instalación:

```bash
conda create -n mit-env python=3.11 -y
conda activate mit-env
pip install -r requirements.txt
python -m ipykernel install --user --name mit-env --display-name "Python 3.11 (mit-env)"
```

- ⚠️ **Pins obligatorios por `yellowbrick` 1.5** (verificados con una prueba de `KElbowVisualizer` + `SilhouetteVisualizer`):
  - `scikit-learn==1.5.2`: a partir de 1.6, sklearn eliminó `_estimator_type`, y yellowbrick rechaza `KMeans` con *"The supplied model is not a clustering estimator"*.
  - `matplotlib==3.8.4`: a partir de 3.9 se eliminó `matplotlib.cm.get_cmap`, y `SilhouetteVisualizer` falla.
- El env usa **pandas 3.x** (Copy-on-Write y dtype `str` por defecto para el texto). Hay que evitar las asignaciones encadenadas (`df[col][mask] = ...`) y usar `.loc`.
- Notebook: la ruta de carga se cambió de `/content/...` (Colab) a la relativa `Customer_Personality_Segmentation.csv`, y el kernelspec apunta a `mit-env`.

---

## 5. Log de decisiones

| Fecha | Decisión / nota |
|-------|-----------------|
| 2026-10-05 | Se crea el documento con la letra del proyecto. Hipótesis inicial: clustering con K-means, K vía Elbow/Silhouette. |
| 2026-10-05 | Se agrega la rúbrica de evaluación (60 pts) y un checklist de entregables derivado. |
| 2026-10-05 | Se recibe el dataset y se registra un primer vistazo (estructura, nulos, duplicados, anomalías). |
| 2026-10-06 | Se decide ejecutar localmente en el conda env `mit-env` con Python 3.11, por compatibilidad con `yellowbrick`. Se fijan `scikit-learn==1.5.2` y `matplotlib==3.8.4`, porque las versiones más nuevas rompen yellowbrick 1.5. Las versiones quedan en `requirements.txt`. Ver "Entorno de ejecución" en la sección 4. |
