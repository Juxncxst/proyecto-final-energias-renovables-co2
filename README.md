# 🌎 Energías Renovables y Emisiones de CO₂ en Colombia (1990–2021)

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue?logo=python)](https://www.python.org/)
[![scikit-learn](https://img.shields.io/badge/scikit--learn-ML-orange?logo=scikitlearn)](https://scikit-learn.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter)](https://jupyter.org/)
[![License](https://img.shields.io/badge/License-Academic-lightgrey)]()

Proyecto Final · Curso de Ciencia de Datos · **Universidad EAN**

---

## 👥 Integrantes

- Juan Castillo
- Maira Martínez
- Jorge Bernal
- Samuel Solano

**Docente:** Camila Silva Gómez

---

## 📌 Descripción del proyecto

Este repositorio contiene el desarrollo completo del proyecto final del curso de Ciencia de Datos, cuyo objetivo es analizar la evolución de las **emisiones de CO₂** en Colombia entre 1990 y 2021, en relación con el crecimiento de la generación de energía eléctrica a partir de **fuentes renovables**.

> ### ❓ Pregunta de investigación
> ¿Qué relación existe entre el crecimiento de las energías renovables y las emisiones de CO₂ en Colombia entre 1990 y 2021?

### 🎯 Objetivo

Determinar el tipo y la magnitud de la relación entre la participación de energías renovables en la matriz eléctrica y las emisiones de CO₂ per cápita en Colombia, a partir de un análisis estadístico y de modelos de aprendizaje automático (*machine learning*) aplicados a series de tiempo del periodo 1990–2021.

---

## 🗂️ Fuente de los datos

La información utilizada proviene de dos conjuntos de datos públicos publicados por **Our World in Data (OWID)**, alojados en su repositorio oficial de GitHub:

| Dataset | Enlace |
|---|---|
| CO2 and Greenhouse Gas Emissions Dataset | [github.com/owid/co2-data](https://github.com/owid/co2-data) |
| Energy Dataset | [github.com/owid/energy-data](https://github.com/owid/energy-data) |

Se seleccionaron estas fuentes por tratarse de bases de datos abiertas, actualizadas periódicamente y ampliamente utilizadas en investigación académica sobre energía y cambio climático, lo que garantiza trazabilidad y confiabilidad de la información.

De ambos conjuntos se extrajeron únicamente los registros correspondientes a **Colombia** en el rango de años **1990–2021**.

---

## 📊 Descripción de las variables

Tras el proceso de limpieza, la tabla analítica final (`df_clean`) quedó conformada por:

| Variable | Descripción |
|---|---|
| `Year` | Año de observación (1990–2021) |
| `CO2_Mt` | Emisiones totales de CO₂ de Colombia, en millones de toneladas (Mt) |
| `CO2_per_capita` | Emisiones de CO₂ por habitante, en toneladas por persona |
| `Renewables_pct` | Porcentaje de electricidad generada a partir de fuentes renovables |
| `Energy_per_capita` | Consumo de energía primaria por habitante |
| `population` | Población total de Colombia para el año correspondiente |

**Unidad de análisis:** año-país (Colombia), con **32 observaciones** (1990–2021), sin valores nulos ni duplicados.

---

## 🧹 Proceso de limpieza y preprocesamiento

1. **Carga de datos** — lectura directa de `owid-co2-data.csv` y `owid-energy-data.csv` desde el repositorio oficial de OWID.
2. **Filtrado** — se conservaron solo los registros de Colombia entre 1990 y 2021.
3. **Selección y renombrado de columnas** relevantes de cada fuente.
4. **Unión de tablas** mediante `merge` por la columna `year`.
5. **Cálculo de variables per cápita** (`CO2_per_capita`, `Energy_per_capita`).
6. **Estandarización de nombres** de columnas.
7. **Consolidación final** en la tabla `df_clean`.
8. **Exportación** a `datos_limpios_colombia_1990_2021.xlsx`.

<details>
<summary>💻 Ver fragmento de código (filtrado, unión y cálculo per cápita)</summary>

```python
# Filtrado por país y periodo (1990-2021)
col_co2 = df_co2_raw[(df_co2_raw['country'] == 'Colombia') &
                      (df_co2_raw['year'].between(1990, 2021))].copy()

# Selección y renombrado de columnas relevantes
col_co2 = col_co2[['year', 'co2', 'population']].rename(
    columns={'co2': 'CO2_Mt'})
col_energy = col_energy[['year', 'renewables_share_elec',
                          'primary_energy_consumption']].rename(
    columns={'renewables_share_elec': 'Renewables_pct',
             'primary_energy_consumption': 'Energy_TWh'})

# Unión de las dos fuentes de datos por año
df_colombia = pd.merge(col_co2, col_energy, on='year')

# Cálculo de variables per cápita
df_colombia['CO2_per_capita'] = (df_colombia['CO2_Mt'] * 1e6) / df_colombia['population']
df_colombia['Energy_per_capita'] = (df_colombia['Energy_TWh'] * 1e9) / df_colombia['population']
```

</details>

---

## 📈 Resultados principales

### Tendencias generales (1990–2021)

- Las emisiones de CO₂ pasaron de **56.9 Mt (1990)** a **103.7 Mt (2021)** → **+82 %**
- La electricidad renovable se mantuvo estable: de **72.6 % (1990)** a **75.1 % (2021)**

### Correlaciones (variables per cápita)

| Relación | Coeficiente de Pearson (r) | Interpretación |
|---|---|---|
| Electricidad renovable vs. CO₂ per cápita | **-0.601** | Negativa moderada |
| Energía per cápita vs. CO₂ per cápita | **+0.690** | Positiva moderada-alta |
| Energía per cápita vs. CO₂ total | **+0.926** | Positiva muy fuerte |

### Comparación de modelos (validación Leave-One-Out, n = 32)

| Modelo | R² | RMSE | MAE |
|---|---|---|---|
| 🏆 **KNN (k=6)** | **0.728** | 0.107 | 0.076 |
| Ridge (L2) | 0.692 | 0.114 | 0.093 |
| Regresión Lineal | 0.692 | 0.114 | 0.093 |
| Lasso (L1) | 0.688 | 0.115 | 0.094 |
| Random Forest | 0.574 | 0.134 | 0.090 |
| Árbol de Decisión | 0.419 | 0.156 | 0.114 |

El modelo con mejor desempeño fue **KNN (k=6)**, con un R² de 0.728 bajo validación Leave-One-Out, seguido de cerca por Ridge y la Regresión Lineal. Los modelos basados en árboles tuvieron el desempeño más bajo, probablemente por el tamaño reducido de la muestra (n=32 años).

### 🔑 Conclusión clave

> El principal factor asociado al crecimiento de las emisiones es el **aumento del consumo de energía per cápita**, no la composición de la matriz eléctrica. Es un problema de **volumen de consumo**, más que de **composición renovable**.

---

## 📁 Estructura del repositorio

```
├── README.md                                   # Este documento
├── Informe_Final.pdf                           # Informe completo del proyecto
├── Renovables_CO2_Colombia_pptx.pdf            # Diapositivas de la sustentación
├── Codigo_proyecto_Data.ipynb                  # Cuaderno con el código completo
└── datos_limpios_colombia_1990_2021.xlsx       # Base de datos procesada
```

---

## ▶️ Instrucciones para ejecutar el código

1. Abrir el archivo `Codigo_proyecto_Data.ipynb` en Google Colab o Jupyter.
2. Ejecutar las celdas de manera secuencial (*Entorno de ejecución → Ejecutar todas*).
3. El cuaderno instala/importa automáticamente las librerías necesarias: `pandas`, `numpy`, `matplotlib`, `seaborn`, `scikit-learn`.
4. Los datos se descargan automáticamente desde los repositorios de GitHub de OWID, por lo que no se requieren archivos locales adicionales.
5. Al finalizar la ejecución, se generará y descargará el archivo `datos_limpios_colombia_1990_2021.xlsx` con la base de datos ya procesada.

---

## 📚 Referencias

- Pedregosa, F., et al. (2011). *Scikit-learn: Machine learning in Python*. Journal of Machine Learning Research, 12, 2825–2830.
- Ritchie, H., Rosado, P., & Roser, M. (2023). *CO2 and Greenhouse Gas Emissions* [Conjunto de datos]. Our World in Data. https://github.com/owid/co2-data
- Ritchie, H., Rosado, P., & Roser, M. (2023). *Energy* [Conjunto de datos]. Our World in Data. https://github.com/owid/energy-data
